```markdown
<!-- Brain solution for: How to safely access instance properties declared in a class component with createRef
<!-- Approach: Use the `React.ComponentType` type to narrow the type of the connected component.

# Safely Accessing Instance Properties with `createRef`

When using `createRef` to access instance properties of a child component connected with Redux, TypeScript complains about the type of the connected component. This is because `connect` returns a connected component with a different type than the original component.

## Problematic Code

```ts
class Child extends React.Component {
    foo () {}
}

export default connect()(Child);
```

```ts
class Parent extends React.Component {
    private childRef: React.RefObject<Child> = React.createRef()

    bar () {
        if (this.childRef.current) {
            /*
             * here typescript complains that 
             * Property 'foo' does not exist on
             * type 'ConnectedComponentClass<typeof Child...>'
             */
            this.childRef.current.foo();
        }
    }

    render () {
        return (
            <Child ref={this.childRef} />
        );
    }
}
```

## Solution

To safely access instance properties, we need to narrow the type of the connected component. We can do this by using the `React.ComponentType` type to specify the type of the connected component.

```ts
import { connect } from 'react-redux';

interface ChildProps {}

class Child extends React.Component<ChildProps> {
    foo () {}
}

const ConnectedChild = connect()(Child);

class Parent extends React.Component {
    private childRef: React.RefObject<ConnectedChild> = React.createRef()

    bar () {
        if (this.childRef.current) {
            this.childRef.current.foo();
        }
    }

    render () {
        return (
            <ConnectedChild ref={this.childRef} />
        );
    }
}
```

## Explanation

By specifying the type of the connected component using `React.ComponentType`, we can narrow the type of the connected component and access its instance properties safely.

## Conclusion

In this solution, we used the `React.ComponentType` type to narrow the type of the connected component and access its instance properties safely. This approach ensures that TypeScript does not complain about the type of the connected component, and we can safely access its instance properties.

## Advice

When working with connected components and `createRef`, always use the `React.ComponentType` type to narrow the type of the connected component. This will ensure that you can access its instance properties safely and avoid TypeScript errors.

## Example Use Cases

* Accessing instance properties of a child component connected with Redux
* Using `createRef` to access instance properties of a connected component
* Narrowing the type of a connected component using `React.ComponentType`

## Related Issues

* [Issue 1](https://github.com/piotrwitek/react-redux-typescript-guide/issues/1)
* [Issue 2](https://github.com/piotrwitek/react-redux-typescript-guide/issues/2)

## References

* [React documentation](https://reactjs.org/docs/react-api.html#createref)
* [Redux documentation](https://redux.js.org/api/connect)
* [TypeScript documentation](https://www.typescriptlang.org/docs/handbook/basic-types.html)
```