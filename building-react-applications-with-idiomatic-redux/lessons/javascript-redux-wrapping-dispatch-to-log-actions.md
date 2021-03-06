Now that my state shape is more complex, I want to override the `store.dispatch` function to add some console logs there so that I can see how the state is affected by the actions.

**configureStore**
```javascript
const configureStore = () => {
  const persistedState = loadState();
  const store = createStore(todoApp, persistedState);

  store.dispatch = addLoggingToDispatch(store);

  store.subscribe(throttle(() => { ... }
```

I'm creating the new function called `addLoggingToDispatch`, that accepts `store` as an argument. It's going to wrap the dispatch provided by Redux, so it reads the raw dispatch from `store.dispatch`.

**configureStore**
```javascript
const addLoggingToDispatch = (store) => {
  const rawDispatch = store.dispatch;

}
```

It will return another function with the same signature as `dispatch`, which is a single `action` argument. Some browsers like Chrome support using **console group API** to group several log statements under a single title, and I'm passing the `action.type` in order to group several logs under the `action.type`.

**configureStore**
```javascript
const addLoggingToDispatch = (store) => {
  const rawDispatch = store.dispatch;
  return (action) => {
    console.group(action.type);
    console.groupEnd(action.type);
  };
};
```

I will log the previous state before dispatching the action by calling `store.getState()`. Next, I will log the action itself so that I can see which action causes the change.

**configureStore**
```javascript
const addLoggingToDispatch = (store) => {
  const rawDispatch = store.dispatch;
  return (action) => {
    console.group(action.type);
    console.log('prev state', store.getState());
    console.log('action', action); 
    console.groupEnd(action.type);
  };
};
```

To preserve the contract of the dispatch function exactly, I am declaring the `returnValue`, and I'm calling the `rawDispatch()` function with the `action`. Now the calling code will not be able to distinguish between my function and the function provided by Redux, except that I also log some information.

**configureStore.js**
```javascript
const addLoggingToDispatch = (store) => {
  const rawDispatch = store.dispatch;
  return (action) => {
    console.group(action.type);
    console.log('prev state', store.getState());
    console.log('action', action); 
    const returnValue = rawDispatch(action);
    console.groupEnd(action.type);
    return returnValue;
  };
};
```

I want to log the next state as well, because after the dispatch is called, the state is guaranteed to be updated, so I can use `store.getState()` in order to receive the next state after the dispatch.

**configureStore**
```javascript
const addLoggingToDispatch = (store) => {
  const rawDispatch = store.dispatch;
  return (action) => {
    ...
    const returnValue = rawDispatch(action);
    console.log('next state', store.getState());
    ... 
  }
}
```

Some browsers, such as Google Chrome, also offer an API to style console logs, and I'm going to add some colors to my logs. I'm painting the previous state gray. The action is blue, and the next state is green.

**configureStore.js**
```javascript
const addLoggingToDispatch = (store) => {
  const rawDispatch = store.dispatch;
  return (action) => {
    console.group(action.type);
    console.log('%c prev state', 'color: gray', store.getState());
    console.log('%c action', 'color: blue', action); 
    const returnValue = rawDispatch(action);
    console.log('%c next state', 'color: green', store.getState());
    console.groupEnd(action.type);
    return returnValue;
  };
};
```

A few final touches. Since the console group API is not available in all browsers, if it's not supported, I'll just return the raw dispatch as is.

**configureStore.js**
```javascript
const addLoggingToDispatch = (store) => {
  const rawDispatch = store.dispatch;
  if (!console.group) {
    return rawDispatch;
  }

  return (action) => {...}

```

Finally, it's not a good idea to log everything in production, so I'll add a gate saying that `if (process.env.NODE_ENV ...)` is not production, then I'm going to run this code. Otherwise, I'm just going to leave the store as is.

**configureStore.js**
```javascript
const configureStore = () => {
  const persistedState = loadState();
  const store = createStore(todoApp, persistedState);

  if (process.env.NODE_ENV !== 'production') {
    store.dispatch = addLoggingToDispatch(store);
  }

  store.subscribe(throttle(() => {...}

```

Let's have a look at how our app runs with the wrapped dispatch function. Now, every time I dispatch an action such as `ADD_TODO`, I can see it in the log. In the previous state, I can see that there are no IDs, and the `byId` look-up table is empty.

![console output](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1553542111/transcript-images/javascript-redux-wrapping-dispatch-to-log-actions-output-1.jpg)

Then I see the action with the type `ADD_TODO`, and the ID and text of the todo. Finally, in the state after dispatching the action, I can see both the ID in all IDs and the corresponding todo in the `byId` mapping.

![console output](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1553542111/transcript-images/javascript-redux-wrapping-dispatch-to-log-actions-output-2.jpg)

The only way to change the Redux store state is by dispatching an action, and now we log every action along with the state before and after it, so it's really easy to find, if there is a mistake in the state, which action caused it.

Let's recap how this works. I added a new function that takes the `store`, grabs dispatch from it, and if your browser doesn't support the console group API, it'll just return the original dispatch function.

**configureStore.js**
```javascript
const addLoggingToDispatch = (store) => {
  const rawDispatch = store.dispatch;
  if (!console.group) {
    return rawDispatch;
  }

  return (action) => {...}

```

Otherwise, I will return a new function that looks like dispatch. It accepts the `action` argument, and it even calls the original dispatch function inside. However, it also places a few logs into a group with the action type.

**configureStore.js**
```javascript
const addLoggingToDispatch = (store) => {
  const rawDispatch = store.dispatch;
  return (action) => {
    console.group(action.type);
    console.log('%c prev state', 'color: gray', store.getState());
    console.log('%c action', 'color: blue', action); 
    const returnValue = rawDispatch(action);
    console.log('%c next state', 'color: green', store.getState());
    console.groupEnd(action.type);
    return returnValue;
  };
};
```

I log the previous state by calling store get state before dispatching. I also log the action that caused the change, and finally, because I know that the store dispatch function is synchronous, I can call `store.getState()` again right after it to log the next state of the store.

Finally, to keep complete compatibility with the underlying dispatch function, I return whatever it returned. Usually, this is the action object.

My function returns a wrapped dispatch function, so I will use it to return value to override the `store.dispatch` method. I don't want to run this code in production, so wrap it in an environment check.

**configureStore.js**
```javascript
const configureStore = () => {
  const persistedState = loadState();
  const store = createStore(todoApp, persistedState);

  if (process.env.NODE_ENV !== 'production') {
    store.dispatch = addLoggingToDispatch(store);
  }

  store.subscribe(throttle(() => {...}

```

This check by itself is not enough, and it will only work correctly if in the production build you use **Define plugin** for Webpack or transform for **Browserify**.