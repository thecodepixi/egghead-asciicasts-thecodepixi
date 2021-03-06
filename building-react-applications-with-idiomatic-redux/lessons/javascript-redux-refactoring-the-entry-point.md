I'm extracting the logic necessary to create the store, and to subscribe to it to persist the state into a separate file.

I'm calling this file `configureStore`, and so I'm creating a function called `configureStore` that is going to contain this logic so that the app doesn't have to know exactly how the store is created and if we have any **subscribe handlers** on it. It can just use the `return store` in the index.js file.

**configureStore.js**
```javascript
const configureStore = () => {
  const persistedState = loadState();
  const store = createStore(
    todoApp,
    persistedState
  );

  store.subscribe(throttle() => {
    saveState({
      todos: store.getState().todos,
    });
  }, 1000))

  return store;
};
```

It is useful to export `configureStore` rather than `store` itself, so that if you later write some tests, you can easily create as many store instances as you want. Now I can import `configureStore` from my application's entry point, and I can use it right away without having to specify the reducer or the subscribers, because `configureStore` takes care of that.

**index.js**
```javascript
import configure from './configureStore';

const store = configureStore();
```

I also want to extract the root rendered element into a separate component that I'm going to call `Root`. It's going to accept the `store` as a **prop**, and it's going to be defined in a separate file in the components folder.

```javascript
render(
  <Root store={store} />
  document.getElementById('root')
);
```

I'm creating a new file called `Root`. It needs to import **React** to use the **JSX** syntax, and I define a **stateless functional component** that just takes the `store` as a **prop** and returns an app inside a **React Redux Provider**.

**Root.js**
```javascript
import React from 'react';

const Root = ({ store }) => (
  <Provider store={store}>
    <App />
  </Provider>
);
```

I need to import `Provider` from **React Redux**, and also import the `App` component they use, which is in the same folder. I can export the `Root` component, and I can go back to my entry point, remove the outdated imports I no longer use, and instead import the `Root` component from the components folder.

**index.js**
```javascript
import Root from './components/Root'
```

Let's recap our refactoring. I moved all the code that is necessary for creating the `store` into a separate module. This module exports a function called `configureStore`. It takes care of creating the store with the right reducer and the right initial state, and setting up any kinds of subscriptions, so that it returns a store that is ready for consumption by my application.

**configureStore.js**
```javascript
const configureStore = () => {
  const persistedState = loadState();
  const store = createStore(
    todoApp,
    persistedState
  );

  store.subscribe(throttle() => {
    saveState({
      todos: store.getState().todos,
    });
  }, 1000))

  return store;
};
```

I also moved the `Root` component into a separate file. Currently, it is very simple. It just wraps the app into **React Redux Provider**. In the future, we can add something like a **router** here.