---
id: navigation-actions
title: CommonActions reference
sidebar_label: CommonActions
---

A navigation action is an object containing at least a `type` property. The action can be handled by routers with the `getStateForAction` method to return a new state from an existing navigation state.

Each navigation actions can contain at least the following properties:

- `type` (required) - A string which represents the name of the action.
- `payload` (options) - An object containing additional information about the action. For example, it will contain `name` and `params` for `navigate`.
- `source` (optional) - The key of the route which should be considered as the source of the action. This is used by some routers to determine which route to apply the action on. By default, `navigation.dispatch` adds the key of the route that dispatched the action.
- `target` (optional) - The key of the navigation state the action should be applied on.

It's important to highlight that dispatching a navigation action doesn't throw any error when the action is unhandled (similar to when you dispatch an action that isn't handled by a reducer in redux and nothing happens).

## Common actions

The library exports several action creators under the `CommonActions` namespace. You should use these action creators instead of writing action objects manually.

### navigate

The `navigate` action allows to navigate to a specific route. It takes the following arguments:

- `name` - _string_ - A destination name of the route that has been registered somewhere..
- `key` - _string_ - The identifier for the route to navigate to. Navigate back to this route if it already exists..
- `params` - _object_ - Params to merge into the destination route..

The options object passed should at least contain a `key` or `name` property, and optionally `params`. If both `key` and `name` are passed, stack navigator will create a new route with the specified key if no matches were found.

<samp id="common-actions" />

```js
import { CommonActions } from '@react-navigation/native';

navigation.dispatch(
  CommonActions.navigate({
    name: 'Profile',
    params: {
      user: 'jane',
    },
  })
);
```

The `navigate` action may also take on a different function signature:

`CommonActions.navigate(name, navigateParams)`

- `name` - _string_ - A destination name of the route that has been defined somewhere
- `navigateParams` - _object_ - An object which may contain any of the following fields:
  - `screen` - _string_ - The screen of a nested navigator to route to
  - `params` - _object_ - Params to merge into the destination route
  - `initial` - _boolean_ - <!-- HELP WANTED: Still didn't figure out what this property does - I see it's used in packages/core/src/useNavigationBuilder.tsx but I didn't yet understand what it's purpose is. Could someone explain? I'll then replace this comment. -->

With the `screen` prop, for example, you can navigate to screen of a [nested navigator](nesting-navigators.md) right after navigating to its parent. This is necessary if you want to route to a screen of a nested navigator which is not the initial screen.

Here we want to navigate to the `Profile` screen (which is a non-initial screen inside `SettingsStack`):

<samp id="common-actions" />

```js
import { CommonActions } from '@react-navigation/native';

navigation.dispatch(
  CommonActions.navigate('SettingsStack', {
    screen: 'Profile'
    params: {
      user: 'jane',
    },
  })
);
```

**Note**: This function signature is different from the [`navigate` method of the `navigation` prop](navigation-prop.md#navigate) where the `params` argument directly contains the `params` you want to merge into the destination route whereas here the `navigateParams` argument accepts an object which may contain a `params` property.

### reset

The `reset` action allows to reset the navigation state to the given state. It takes the following arguments:

- `state` - _object_ - The new navigation state object to use.

<samp id="common-actions" />

```js
import { CommonActions } from '@react-navigation/native';

navigation.dispatch(
  CommonActions.reset({
    index: 1,
    routes: [
      { name: 'Home' },
      {
        name: 'Profile',
        params: { user: 'jane' },
      },
    ],
  })
);
```

The state object specified in `reset` replaces the existing navigation state with the new one. This means that if you provide new route objects without a key, or route objects with a different key, it'll remove the existing screens for those routes and add new screens.

If you want to preserve the existing screens but only want to modify the state, you can pass a function to `dispatch` where you can get the existing state. Then you can change it as you like (make sure not to mutate the existing state, but create new state object for your changes). and return a `reset` action with the desired state:

```js
import { CommonActions } from '@react-navigation/native';

navigation.dispatch(state => {
  // Remove the home route from the stack
  const routes = state.routes.filter(r => r.name !== 'Home');

  return CommonActions.reset({
    ...state,
    routes,
    index: routes.length - 1,
  });
});
```

> Note: Consider the navigator's state object to be internal and subject to change in a minor release. Avoid using properties from the navigation state object except `index` and `routes`, unless you really need it. If there is some functionality you cannot achieve without relying on the structure of the state object, please open an issue.

### goBack

The `goBack` action creator allows to go back to the previous route in history. It doesn't take any arguments.

<samp id="common-actions" />

```js
import { CommonActions } from '@react-navigation/native';

navigation.dispatch(CommonActions.goBack());
```

If you want to go back from a particular route, you can add a `source` property referring to the route key and a `target` property referring to the `key` of the navigator which contains the route:

<samp id="common-actions" />

```js
import { CommonActions } from '@react-navigation/native';

navigation.dispatch({
  ...CommonActions.goBack(),
  source: route.key,
  target: state.key,
});
```

By default, the key of the route which dispatched the action is passed as the `source` property and the `target` property is `undefined`.

### setParams

The `setParams` action allows to update params for a certain route. It takes the following arguments:

- `params` - _object_ - required - New params to be merged into existing route params.

<samp id="common-actions" />

```js
import { CommonActions } from '@react-navigation/native';

navigation.dispatch(CommonActions.setParams({ user: 'Wojtek' }));
```

If you want to set params for a particular route, you can add a `source` property referring to the route key:

<samp id="common-actions" />

```js
import { CommonActions } from '@react-navigation/native';

navigation.dispatch({
  ...CommonActions.setParams({ user: 'Wojtek' }),
  source: route.key,
});
```

If the `source` property is explicitly set to `undefined`, it'll set the params for the focused route.
