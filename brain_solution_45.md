<!-- Brain solution for: suggestion: add info about dispatched action types
<!-- Approach: Added a new section to the guide on declaring dispatched action types using the `typeof` operator.

# Declaring Dispatched Action Types
=====================================

When working with Redux and TypeScript, it's essential to accurately declare the types of dispatched actions. This ensures that your code is type-safe and helps catch errors early. In this section, we'll explore how to declare dispatched action types using the `typeof` operator.

## Using `typeof` to Declare Dispatched Action Types
---------------------------------------------------

When creating a connected component's property types, you can use the `typeof` operator to declare dispatched function types. This matches the action's declaration and provides a clear understanding of the expected action type.

```tsx
const actions = {
  ping: createAction('ping/PING', (arg: number) => ({
    type: 'ping/PING',
    arg,
  })),
};

interface Props {
  ping: typeof actions.ping;
}
```

In the example above, we define an `actions` object with a `ping` property. The `ping` property is a function that returns an object with a `type` property set to `'ping/PING'` and an `arg` property set to the provided `number` argument. We then declare the `Props` interface with a `ping` property of type `typeof actions.ping`, which matches the type of the `ping` action.

## Benefits of Using `typeof`
---------------------------

Using `typeof` to declare dispatched action types offers several benefits:

*   **Type Safety**: By matching the action's declaration, you ensure that your code is type-safe and helps catch errors early.
*   **Code Clarity**: The `typeof` operator provides a clear understanding of the expected action type, making your code easier to read and maintain.
*   **Improved Code Completion**: When using the `typeof` operator, your IDE or code editor can provide more accurate code completion suggestions, reducing the likelihood of typos and errors.

## Example Use Case
-------------------

Let's consider an example use case where we have a connected component that dispatches an action to update a user's profile. We can use the `typeof` operator to declare the dispatched action type as follows:

```tsx
const actions = {
  updateUserProfile: createAction('user/UPDATE_PROFILE', (user: User) => ({
    type: 'user/UPDATE_PROFILE',
    user,
  })),
};

interface Props {
  updateUserProfile: typeof actions.updateUserProfile;
}

const UserProfileComponent: React.SFC<Props> = ({ updateUserProfile }) => {
  return (
    <Button onPress={() => updateUserProfile({ name: 'John Doe', email: 'john.doe@example.com' })} title="Update Profile"/>
  );
};
```

In this example, we define an `actions` object with an `updateUserProfile` property. We then declare the `Props` interface with an `updateUserProfile` property of type `typeof actions.updateUserProfile`, which matches the type of the `updateUserProfile` action.

By using the `typeof` operator to declare dispatched action types, you can ensure that your code is type-safe, clear, and maintainable. This approach also improves code completion and reduces the likelihood of typos and errors.