<!-- Brain solution for: How to use forwardRef?
<!-- Approach: Provide a comprehensive guide on correctly typing components using `forwardRef` in React with TypeScript, covering basic usage, custom props, and integration with `memo` and `connect`.

# How to use `forwardRef`?

Using `forwardRef` with TypeScript can be a bit tricky due to its generic nature. React's `forwardRef` allows your component to receive a ref and forward it to a child component. Here's a comprehensive guide on how to correctly type components that use `forwardRef`, including common scenarios and best practices.

## Basic Usage

The `forwardRef` function expects a render function as its argument. This render function receives `props` and `ref` as parameters. TypeScript infers the types if you provide the correct generics to `forwardRef`.

The general signature for `forwardRef` is `forwardRef<RefType, PropsType>(...)`.

Let's start with a simple example:

```tsx
import React, { forwardRef, Ref } from 'react';

// 1. Define the props for your component
interface MyButtonProps {
  label: string;
  onClick?: () => void;
  // Add any other props your component needs
}

// 2. Define the type of the ref that will be forwarded
// In this case, we're forwarding a ref to a native HTMLButtonElement
type MyButtonRef = HTMLButtonElement;

// 3. Create the component using forwardRef
const MyButton = forwardRef<MyButtonRef, MyButtonProps>((props, ref: Ref<MyButtonRef>) => {
  const { label, onClick, ...rest } = props;
  return (
    <button ref={ref} onClick={onClick} {...rest}>
      {label}
    </button>
  );
});

// 4. Usage example
function App() {
  const buttonRef = React.useRef<MyButtonRef>(null);

  React.useEffect(() => {
    if (buttonRef.current) {
      console.log('Button element:', buttonRef.current);
      buttonRef.current.focus(); // Example: focusing the button
    }
  }, []);

  return (
    <div>
      <MyButton label="Click Me" onClick={() => alert('Clicked!')} ref={buttonRef} />
    </div>
  );
}
```

**Explanation:**
- `MyButtonProps`: Defines the props that users of `MyButton` will pass.
- `MyButtonRef`: Specifies the type of the ref that this component will actually hold (e.g., `HTMLButtonElement` for a button, `HTMLDivElement` for a div, or an instance of a class component).
- `forwardRef<MyButtonRef, MyButtonProps>`: This is crucial.
  - The first generic argument (`MyButtonRef`) is the type of the `ref` that will be passed *into* the render function.
  - The second generic argument (`MyButtonProps`) is the type of the `props` that will be passed *into* the render function.
- `(props, ref: Ref<MyButtonRef>)`: The `ref` parameter inside the render function needs to be explicitly typed as `Ref<MyButtonRef>` to ensure correct type inference and usage.

## Forwarding Ref to a Custom Component

You can also forward a ref to another custom component that itself accepts a ref. The key is to ensure the types match.

```tsx
import React, { forwardRef, Ref, ComponentPropsWithoutRef } from 'react';

// Inner component that actually receives the ref
interface FancyInputProps extends ComponentPropsWithoutRef<'input'> {
  // We extend ComponentPropsWithoutRef<'input'> to include all standard input props
  // without 'ref' because it's handled by forwardRef
  customStyle?: React.CSSProperties;
}

const FancyInput = forwardRef<HTMLInputElement, FancyInputProps>((props, ref) => {
  const { customStyle, ...rest } = props;
  return <input ref={ref} style={{ border: '2px solid blue', ...customStyle }} {...rest} />;
});


// Outer component that forwards the ref to FancyInput
interface LabeledInputProps {
  label: string;
  // We still want to allow users to pass standard input props like placeholder, value etc.
  // but explicitly exclude `ref` since we are handling it.
  inputProps?: Omit<FancyInputProps, 'ref'>;
}

type LabeledInputRef = HTMLInputElement; // The ref is still for the underlying HTMLInputElement

const LabeledInput = forwardRef<LabeledInputRef, LabeledInputProps>((props, ref: Ref<LabeledInputRef>) => {
  const { label, inputProps } = props;
  return (
    <div>
      <label>{label}: </label>
      <FancyInput ref={ref} {...inputProps} />
    </div>
  );
});

// Usage
function AppWithLabeledInput() {
  const inputRef = React.useRef<HTMLInputElement>(null);

  React.useEffect(() => {
    if (inputRef.current) {
      inputRef.current.focus();
    }
  }, []);

  return (
    <div>
      <LabeledInput label="Your Name" ref={inputRef} inputProps={{ placeholder: 'Enter your name' }} />
    </div>
  );
}
```

Here, `LabeledInput` receives a ref, and then explicitly passes it down to `FancyInput`, which in turn passes it to the native `<input>` element. The `LabeledInputRef` remains `HTMLInputElement` because that's what the ref ultimately points to.

## Integration with `React.memo`

When combining `forwardRef` with `React.memo` for performance optimization, the `memo` call should wrap the `forwardRef` component.

```tsx
import React, { forwardRef, memo, Ref } from 'react';

interface MemoButtonProps {
  label: string;
  count: number;
  onClick: () => void;
}

type MemoButtonRef = HTMLButtonElement;

// Define the component with forwardRef first
const MyMemoButton = forwardRef<MemoButtonRef, MemoButtonProps>((props, ref: Ref<MemoButtonRef>) => {
  console.log('MemoButton rendered');
  const { label, count, onClick } = props;
  return (
    <button ref={ref} onClick={onClick}>
      {label} - Count: {count}
    </button>
  );
});

// Then wrap it with memo
const MemoizedMyButton = memo(MyMemoButton);

// Usage example
function AppWithMemo() {
  const [count, setCount] = React.useState(0);
  const buttonRef = React.useRef<MemoButtonRef>(null);

  return (
    <div>
      <MemoizedMyButton label="Increment" count={count} onClick={() => setCount(prev => prev + 1)} ref={buttonRef} />
      <p>Parent count: {count}</p>
    </div>
  );
}
```

The order matters: `memo(forwardRef(...))` not `forwardRef(memo(...))`.

## Integration with `redux` `connect`

When using `react-redux`'s `connect` HOC with `forwardRef`, you need to set the `forwardRef` option to `true` in the `connect` configuration. This allows the connected component to receive a ref.

```tsx
import React, { forwardRef, Ref, memo } from 'react';
import { connect, ConnectedProps } from 'react-redux';

// 1. Define the component's own props (props that are not from Redux)
interface MyConnectedButtonOwnProps {
  label: string;
  // Any other props passed directly to the component
}

// 2. Define Redux state and dispatch types
interface RootState {
  counter: { value: number };
}

interface MyDispatchProps {
  increment: () => void;
}

const mapStateToProps = (state: RootState) => ({
  count: state.counter.value,
});

const mapDispatchToProps = (dispatch: any): MyDispatchProps => ({
  increment: () => dispatch({ type: 'INCREMENT' }),
});

// 3. Create the connector
const connector = connect(mapStateToProps, mapDispatchToProps, null, { forwardRef: true });

// 4. Infer props type from the connector
type PropsFromRedux = ConnectedProps<typeof connector>;

// 5. Combine own props and Redux props
type MyConnectedButtonProps = MyConnectedButtonOwnProps & PropsFromRedux;

// 6. Define the ref type
type MyConnectedButtonRef = HTMLButtonElement; // If forwarding to a button

// 7. Create the component, accepting ref
const MyConnectedButton = forwardRef<MyConnectedButtonRef, MyConnectedButtonProps>((props, ref: Ref<MyConnectedButtonRef>) => {
  const { label, count, increment } = props;
  console.log('MyConnectedButton rendered');
  return (
    <button ref={ref} onClick={increment}>
      {label} - Redux Count: {count}
    </button>
  );
});

// 8. Connect the component
const ConnectedAndForwardedButton = connector(MyConnectedButton);

// Optional: Wrap with memo for performance if Redux props change frequently
// const ConnectedAndMemoizedButton = memo(ConnectedAndForwardedButton);

// Usage example (assuming a Redux store is provided higher up)
function AppWithReduxForwardRef() {
  const buttonRef = React.useRef<MyConnectedButtonRef>(null);

  React.useEffect(() => {
    if (buttonRef.current) {
      console.log('Connected Button element:', buttonRef.current);
    }
  }, []);

  return (
    <div>
      <ConnectedAndForwardedButton label="Increment Redux" ref={buttonRef} />
      {/* You'd typically display the Redux state elsewhere or inside the component */}
    </div>
  );
}

// Minimal Redux store setup for demonstration
// This would typically be in a separate file (e.g., store.ts)
const initialState = {
  counter: { value: 0 },
};

function rootReducer(state = initialState, action: any) {
  switch (action.type) {
    case 'INCREMENT':
      return {
        ...state,
        counter: { value: state.counter.value + 1 },
      };
    default:
      return state;
  }
}

import { createStore } from 'redux';
import { Provider } from 'react-redux';

const store = createStore(rootReducer);

function AppRoot() {
  return (
    <Provider store={store}>
      <AppWithReduxForwardRef />
    </Provider>
  );
}
```

**Key takeaways for `connect` with `forwardRef`:**
- Pass `{ forwardRef: true }` as the fourth argument to `connect`.
- The component passed to `connect` should be the one wrapped with `forwardRef`.
- The ref type will be the type of the underlying DOM element or component instance that the ref eventually points to.

By following these patterns, you can effectively use `forwardRef` with TypeScript in various React component structures, ensuring type safety and correct behavior.