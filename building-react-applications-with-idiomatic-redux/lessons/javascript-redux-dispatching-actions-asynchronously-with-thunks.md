
**VisibleTodoList.js**
```javascript
fetchData() {
  const { filter, fetchTodos, requestTodos } = this.props;
  requestTodos(filter);
  fetchTodos(filter);
}
```
To show the loading indicator, I dispatch the `requestTodos` action before I `fetchTodos`. It would be great if I could make `requestTodos` dispatched automatically when I fetch the todos because I never want to fire them separately.

**VisibleTodoList.js**
```javascript
fetchData() {
  const { filter, fetchTodos } = this.props;
  fetchTodos(filter);
}
```
I'm removing the explicit `requestTodos` dispatch from the component, and in the file where I define the action creators, I'm no longer exporting `requestTodos` action creator.

**index.js**
```javascript
const requestTodos = (filter) => ({
  type: 'REQUEST_TODOS',
  filter,
});

export const receiveTodos = (filter) => ({
  type: 'RECEIVE_TODOS',
  filter,
  response,
});
```
I want to dispatch `requestTodos` when they start fetching, and `receiveTodos` when they finish fetching, but the `fetchTodos` action creator only resolves through the `receiveTodos` action.

**index.js**
```javascript
export const fetchTodos = (filter) =>
  api.fetchTodos(filter).then(response =>
  receiveTodos(filter, response)
  );
```
An action promise resolves through a single action at the end, but we want an abstraction that represents multiple actions dispatched over the period of time. This is why rather than return a **promise**, I want to return a **function** that accepts a **dispatch callback argument**.

This lets me call dispatch as many times as I like at any point of time during the async operation. I can dispatch the `requestTodos` action in the beginning, and when the promise resolves, I can explicitly dispatch another `receiveTodos` action at the end.

**index.js**
```javascript
export const fetchTodos = (filter) => (dispatch) => {
  dispatch(requestTodos(filter));

  return api.fetchTodos(filter).then(response => {
    dispatch(receiveTodos(filter, response));
  });
}
```
This means more typing than returning a promise, but it also gives me more flexibility. A promise can only express one async value, so `fetchTodos` now returns a function with a callback argument so that it can call it multiple times during the async operation.

Such functions returned from other functions are often called **thunks**, so we're going to implement a thunk middleware to support them. I'm opening the `configureStore` file where I define the middleware, and I will remove the promise middleware, and I will replace it with a different middleware that I'm going to write now.

The new middleware I'm writing is called the `thunk` middleware because it supports the dispatching of thunks, and it takes the `store`, the `next` middleware, and the `action` as curried arguments, just like any other middleware.

If the action is not an `action`, but rather a `function`, we're going to assume that this is a thunk that wants the dispatch function to be injected into it, so I'm calling the action with `store.dispatch`. Otherwise, I'm just going to return the result of passing the action to the `next` middleware in chain.

**configureStore.js**
```javascript
const thunk = (store) => (next) => (action) =>
  typeof action === 'function' ?
    action(store.dispatch) :
    next(action);

const configureStore = () => {
  const middlewares = [thunk];

}
```
As a final step, I need to add the `thunk` middleware I just wrote to my array of `middlewares` so that it gets applied to the store. Let's recap how `thunk` middleware lets us dispatch multiple actions asynchronously.

The `thunk` middleware has the same signature as all Redux middlewares. It accepts the `store`, the `next` dispatch function, and the `action` as curried arguments. It returns a new dispatch function that checks if the `action` is actually a `function` itself.

We can see that this is the case with `fetchTodos` action creator. Its return value is a function that wants `dispatch` as its argument so that it can `dispatch` multiple times.

**index.js**
```javascript
export const fetchTodos = (filter) => (dispatch) => {
  dispatch(requestTodos(filter));

  return api.fetchTodos(filter).then(response => {
    dispatch(receiveTodos(filter, response));
  });
}
```
This is why when we see an action that is really a function, a `thunk`, we call it with `store.dispatch` as an argument so that it can dispatch other actions by itself. The `store.dispatch` function becomes available as the `dispatch` argument inside the `thunk`.

Now, the `thunk` can use the `dispatch` argument to dispatch the `requestTodos` action in the very beginning, and `receiveTodos` action at the very end. The dispatch function injected into thunks is already wrapped with the whole middleware chain, so thunks can dispatch both plain object actions and other thunks.

No matter what gets dispatched, it will go through the middleware chain again, and if its type is a `function`, it will be called like a thunk, but otherwise, it will be passed on to the next middleware in chain.

In this case, it's the `logger`. Only plain object actions reach the logger, and then reach the reducers. 

**configureStore.js**
```javascript
cosnt configureStore = () => {
  const middlewares = [thunk];
  if (process.env.NODE_ENV !== 'production') {
    middlewares.push(creatLogger());
  }
}
```
The thunk middleware is a powerful, composable way to express async action creators that want to emit several actions during the course of an async operation.

This lets the components specify the intention to start an async operation without worrying which actions get dispatched and when.