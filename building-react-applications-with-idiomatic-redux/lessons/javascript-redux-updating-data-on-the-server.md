I am changing the `toggleTodo` action creator to be a `thunk` action creator, so I'm adding `dispatch` as a **curried** argument. I'm calling the `TOGGLE_TODO` API endpoint, and waiting for the response to come back.

**index.js**
```javascript
export const toggleTodo = (id) => (dispatch) =>
  api.toggleTodo(id).then(response => {
    dispatch({
      type: 'TOGGLE_TODO_SUCCESS',
      response: normalize(response, schema.todo)
    });
  });
```
When the response is available, I will dispatch an action with the type `TOGGLE_TODO_SUCCESS`, and the response. However, I will normalize the response first by passing the original `response` as the first argument, and the `schema.todo` as the second argument.

Now let's run the app. I will clear the console and toggle the todo. The UI updates when the `TOGGLE_TODO_SUCCESS` action is dispatched because the action contains a normalized response in which the corresponding `todo` has a completed field set to `true`, 

![Toggle Todo Action](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1553542111/transcript-images/javascript-redux-updating-data-on-the-server-toggle-todo-action.jpg)

so the `byId` reducer merges the new version of the `todo` into the current state of the lookup table.

![Toggle Todo Next State](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1553542113/transcript-images/javascript-redux-updating-data-on-the-server-next-state.jpg)

This is the benefit of normalizing the API responses. Updates to the entities get **merged automatically**. I can switch to the `completed` tab to verify that the server state has indeed been updated.

I will clear the console, and then toggle the `todo` while I'm on the `completed` tab. After the `TOGGLE_TODO_SUCCESS` action, the corresponding `todo` in the `byId` lookup table has been updated, and now has completed set to `false`. However, it is still visible in the completed list because we have not refetched it, and so its ID is still there.

![Toggle Completed Todo](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1553542113/transcript-images/javascript-redux-updating-data-on-the-server-toggle-completed-todo.jpg)

I can force refetching by going to the `active` tab, and then going to the `completed` tab back again, and it will disappear. However, I would prefer if it disappeared immediately, so I'm opening the `ids` reducer in `createlist.js`.

I will add a new case for the `TOGGLE_TODO_SUCCESS` action. I will extract the code for this case into a separate function called handle toggle, to which I pass the state and the action.

**createList.js**
```javascript
case 'TOGGLE_TODO_SUCCESS':
  return handleToggle(state, action);
```
I'm declaring a `handleToggle` function above the `ids` reducer. It accepts the `state` and array of `ids`, and the toggle todo success `action`.

I'm [destructuring](https://egghead.io/lessons/ecmascript-6-destructuring-assignment?course=learn-es6-ecmascript-2015) the result as the `toggledId` and the `entities` from the normalized response. Next, I am reading the `completed` value from the `todo`, which I get by referencing `entities.todos` by the `toggledId`.

**createList.js**
```javascript
const createList = (filter) => {
  const handleToggle = (state, actions) => {
    const { result: toggledId, entities } = action.response;
    const { completed } = entities.todos[toggledId];

  }
}
```
There are two cases in which I want to remove the `todo` from the list. The first case is when the `completed` field is `true` but the filter is `active`, so it shouldn't be there, and the other situation is when the `completed` is `false` but the `filter` is `completed`, so we want it to disappear.

**createList.js**
```javascript
const createList = (filter) => {
  const handleToggle = (state, actions) => {
    const { result: toggledId, entities } = action.response;
    const { completed } = entities.todos[toggledId];
    const shouldRemove = (
      (completed && filter === 'active') ||
      (!completed && filter === 'completed')
    );
  }
}
```
When `shouldRemove` is `true`, I want to return a copy of the list that does not contain the `id` of the `todo` that was just toggled, so I filter the list by `id` and only leave the ones that have a different id. Otherwise, I return the original array.

**createList.js**
```javascript
const createList = (filter) => {
  const handleToggle = (state, actions) => {
    const { result: toggledId, entities } = action.response;
    const { completed } = entities.todos[toggledId];
    const shouldRemove = (
      (completed && filter === 'active') ||
      (!completed && filter === 'completed')
    );
    return shouldRemove ?
      state.filter(id => id !== toggledID) :
      state;
  };
}
```
If I run the app now and toggle the `todo` while I'm on the completed tab, it will disappear right after `TOGGLE_TODO_SUCCESS` action, even though we have not refetched the `todos` since the last time.

Similarly, I can open the `active` tab and toggle a `todo` there. It disappears after the `TOGGLE_TODO_SUCCESS` action even though the `active todos` have not been refetched.

![Toggle Todo Correctly Works](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1553542111/transcript-images/javascript-redux-updating-data-on-the-server-toggle-todo-correctly-works.jpg)

Finally, I can open the `all` tab and toggle the `todos` back and forth, and since it doesn't have any special logic, they all stay there.

Let's recap how we handle the `TOGGLE_TODO_SUCCESS` action. I extracted a function called `handleToggle`, and it makes sure that the toggle `todo` is immediately removed from the lists that shouldn't contain it.

**createList.js**
```javascript
const createList = (filter) => {
  const handleToggle = (state, actions) => {
    const { result: toggledId, entities } = action.response;
    const { completed } = entities.todos[toggledId];
    const shouldRemove = (
      (completed && filter === 'active') ||
      (!completed && filter === 'completed')
    );
    return shouldRemove ?
      state.filter(id => id !== toggledID) :
      state;
  };
}
```
I'm reading the ID of the toggle `todo` from the `result` field populated by a normalizer. Next, I'm reading the `completed` field from the updated `todo` inside the todos dictionary in the normalized response.

I want to remove the `todo` in two cases, if it's `completed` but we're looking at the list of `active todos`, or if it's not completed but we're looking at the list of the `completed todos`.

The `handleToggle` function is called from the `TOGGLE_TODO_SUCCESS` case inside the `ids` reducer. 

**createList.js**
```javascript
case 'TOGGLE_TODO_SUCCESS':
  return handleToggle(state, action);
```
I did not need to change any code inside the `byId` reducer because any updated `todos` get merged into the new version of the lookup table automatically.

**byid.js**
```javascript
const byId = (state = {}, action) => {
  if (action.response) {
    return {
      ...state,
      ...action.response.entities.todos,
    };
  }
  return state;
}
```
The only other place I needed to change is the `toggleTodo` action creator. I change it to be a thunk action creator by adding a curried `dispatch` argument, and I made it call the `toggleTodo` API endpoint.

**index.js**
```javascript
export const toggleTodo = (id) => (dispatch) =>
  api.toggleTodo(id).then(response => {
    dispatch({
      type: 'TOGGLE_TODO_SUCCESS',
      response: normalize(response, schema.todo)
    });
  });
```
When the response is available, it dispatches the `TOGGLE_TODO_SUCCESS` action, with the normalized response that I get by calling `normalize` with the response, and a single `schema.todo`.