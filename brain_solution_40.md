<!-- Brain solution for: [Section] Modelling async data with ADT  
<!-- Approach: Utilize Algebraic Data Types (ADTs) to constrain data structures and prevent incorrect state representations. -->

# Modelling Async Data with Algebraic Data Types (ADTs)
=====================================================

In this section, we will explore how to model data structures using static-typing to make the incorrect state unrepresentable. By constraining our data structure to only represent correct state, we can prevent a multitude of edge cases and display bugs.

## Why ADTs?

Algebraic Data Types (ADTs) are a powerful tool for modelling complex data structures in a type-safe manner. They allow us to define a set of possible values that a type can take, and ensure that only those values are represented. This is particularly useful when working with async data, where the state of the application can change rapidly.

## Pattern 1: Using a Generic Union Type

One common pattern for modelling async data is to use a generic union type. This type can be designed specifically for our use case, and will ensure that only the correct state is represented.

```typescript
type AsyncData<T> =
  | { type: 'loading' }
  | { type: 'loaded'; data: T }
  | { type: 'error'; error: string }
```

In this example, the `AsyncData` type is a union of three possible values: `loading`, `loaded` with a value of type `T`, and `error` with a string error message. This ensures that the type can only represent the correct state, and prevents incorrect state representations.

## Pattern 2: Using a Discriminated Union Type

Another pattern for modelling async data is to use a discriminated union type. This type has a discriminant property that determines which variant of the type is being represented.

```typescript
type AsyncData<T> =
  | { status: 'loading' }
  | { status: 'loaded'; data: T }
  | { status: 'error'; error: string }
```

In this example, the `AsyncData` type is a union of three possible values, each with a `status` property that determines which variant is being represented. This ensures that the type can only represent the correct state, and prevents incorrect state representations.

## Pattern 3: Using a Type Guard

A type guard is a function that takes a value of a certain type and returns a boolean indicating whether the value conforms to that type. We can use a type guard to ensure that only the correct state is represented.

```typescript
function isLoaded<T>(data: AsyncData<T>): data is { type: 'loaded'; data: T } {
  return data.type === 'loaded';
}

const data: AsyncData<string> = { type: 'loaded', data: 'Hello, World!' };
if (isLoaded(data)) {
  console.log(data.data); // string
}
```

In this example, the `isLoaded` function is a type guard that takes a value of type `AsyncData<T>` and returns a boolean indicating whether the value conforms to the `loaded` variant. We can use this type guard to ensure that only the correct state is represented.

## Conclusion

In this section, we have explored how to model data structures using static-typing to make the incorrect state unrepresentable. By utilizing Algebraic Data Types (ADTs), we can constrain our data structure to only represent correct state, and prevent a multitude of edge cases and display bugs. We have also seen how to use generic union types, discriminated union types, and type guards to achieve this goal. By following these patterns, we can write more robust and maintainable code.