<!-- Brain solution for: @connect decorator
<!-- Approach: Provide a detailed example of using the `@connect` decorator with `react-redux` and TypeScript for class components, including `mapStateToProps` and `mapDispatchToProps` type safety.

## Using `@connect` Decorator

While functional components with hooks are now the recommended approach for `react-redux`, the `@connect` decorator remains a valid and type-safe option for class components. This section will guide you through setting up and using the `@connect` decorator with TypeScript.

### Installation

First, ensure you have the necessary packages installed:

```bash
npm install react-redux @types/react-redux
# or
yarn add react-redux @types/react-redux
```

### Basic Usage with TypeScript

Let's assume you have a Redux store with a simple state and an action.

**`src/store/types.ts`**

```typescript
// Define your RootState
export interface RootState {
  counter: {
    value: number;
  };
}

// Define action types
export const INCREMENT = 'INCREMENT';

interface IncrementAction {
  type: typeof INCREMENT;
}

export type CounterActionTypes = IncrementAction;
```

**`src/store/reducers/counter.ts`**

```typescript
import { INCREMENT, CounterActionTypes } from '../types';

interface CounterState {
  value: number;
}

const initialState: CounterState = {
  value: 0,
};

export function counterReducer(
  state = initialState,
  action: CounterActionTypes
): CounterState {
  switch (action.type) {
    case INCREMENT:
      return { ...state, value: state.value + 1 };
    default:
      return state;
  }
}
```

**`src/store/actions/counter.ts`**

```typescript
import { INCREMENT, CounterActionTypes } from '../types';

export function increment(): CounterActionTypes {
  return {
    type: INCREMENT,
  };
}
```

**`src/store/index.ts`**

```typescript
import { createStore, combineReducers } from 'redux';
import { counterReducer } from './reducers/counter';
import { RootState } from './types';

const rootReducer = combineReducers<RootState>({
  counter: counterReducer,
});

export const store = createStore(rootReducer);
```

Now, let's create a class component and connect it using the `@connect` decorator.

**`src/components/CounterDecorator.tsx`**

```typescript
import * as React from 'react';
import { connect } from 'react-redux';
import { RootState } from '../store/types';
import { increment } from '../store/actions/counter';
import { Dispatch } from 'redux';

// 1. Define the props that the component *receives* from its parent
interface OwnProps {
  initialValue?: number;
}

// 2. Define the props that the component *receives* from Redux state
interface StateProps {
  count: number;
}

// 3. Define the props that the component *receives* as Redux actions (dispatch functions)
interface DispatchProps {
  increment: () => void;
}

// 4. Combine all props into a single interface for the component
type Props = OwnProps & StateProps & DispatchProps;

class CounterDecorator extends React.Component<Props> {
  // Use the initialValue from ownProps if provided, otherwise fallback to connected count
  private get displayValue(): number {
    return this.props.initialValue !== undefined
      ? this.props.initialValue
      : this.props.count;
  }

  public render() {
    return (
      <div>
        <h2>Counter (Decorator)</h2>
        <p>Current Count: {this.displayValue}</p>
        <button onClick={this.props.increment}>Increment</button>
      </div>
    );
  }
}

// mapStateToProps: Maps Redux state to component props
const mapStateToProps = (state: RootState, ownProps: OwnProps): StateProps => ({
  count: state.counter.value,
});

// mapDispatchToProps: Maps Redux actions to component props
const mapDispatchToProps = (dispatch: Dispatch): DispatchProps => ({
  increment: () => dispatch(increment()),
});

// Use the @connect decorator.
// It automatically infers the types for mapStateToProps and mapDispatchToProps.
// For the decorator to work, make sure 'experimentalDecorators' is true in your tsconfig.json.
export default connect(mapStateToProps, mapDispatchToProps)(CounterDecorator);
```

**`tsconfig.json`**

Ensure you have `experimentalDecorators` enabled in your `tsconfig.json`:

```json
{
  "compilerOptions": {
    // ... other options
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true // Often useful with experimentalDecorators
  }
}
```

### Explanation

1.  **`OwnProps`**: These are the props passed directly to the `CounterDecorator` component from its parent.
2.  **`StateProps`**: These are the props that will be injected into the component from the Redux store's state via `mapStateToProps`.
3.  **`DispatchProps`**: These are the props that represent Redux action dispatchers, injected via `mapDispatchToProps`.
4.  **`Props`**: This is a union of `OwnProps`, `StateProps`, and `DispatchProps`, representing the complete set of props the component expects to receive.
5.  **`mapStateToProps`**:
    *   It takes `RootState` (your entire Redux state) and `OwnProps` as arguments.
    *   It returns an object of type `StateProps`, mapping specific parts of the `RootState` to the component's props.
    *   TypeScript ensures that `state.counter.value` matches the type expected by `StateProps['count']`.
6.  **`mapDispatchToProps`**:
    *   It takes a `Dispatch` function from Redux as an argument.
    *   It returns an object of type `DispatchProps`, mapping action creators to dispatch calls.
    *   TypeScript ensures that `increment()` returns an action compatible with `dispatch` and that the returned object matches `DispatchProps`.
7.  **`@connect` Decorator**:
    *   The `connect` function from `react-redux` is used as a decorator by placing `@connect(mapStateToProps, mapDispatchToProps)` directly above the class definition.
    *   It takes `mapStateToProps` and `mapDispatchToProps` as arguments.
    *   `react-redux`'s TypeScript typings are robust enough to correctly infer the resulting component props based on these functions, providing strong type safety.
    *   The final connected component is then exported.

This setup provides a fully type-safe way to connect your class components to Redux using the `@connect` decorator, ensuring that your state and action props are correctly typed throughout your application.