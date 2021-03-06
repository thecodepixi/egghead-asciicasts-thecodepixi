In the current implementation, we keep all `todos` in memory at once. We have an array of `allIDs` ever and we get the array of todos that we can filter according to the `filter` passed from React router.

However, this only works correctly if all the data from the server is already available in the client, which is usually not the case with applications that fetch something. If we have thousands of todos on the server, it would be impractical to fetch them all and filter them on the client.

Rather than keep a big list of IDs, we'll keep a list of IDs for every tab so that they can be stored separately and filled according with the `actions` with the fetch data on the corresponding tab.

I'm removing the `getAllTodos` selector because we won't have access to `allTodos`. We also don't need to filter on the client anymore because we will use the list of todos provided by the server.

**todos.js**
```javascript
export const getVisibleTodos = (state, filter) => {
  const ids = state.idsByFilter[filter];
  return ids.map(id => state.byId[id]);
};
```
Rather than read from `state` all `ids`, I will read the IDs from state IDs by filter. Then I can use the same technique as before and map the IDs to `state.byId[id]` look up table to get the actual todos.

My selector now expects `byId` and `idsByFilter` to be part of the combined state of the `todos` reducer. The `todos` reducer used to combine the look of table and a list of all IDs which I'm replacing with the list of `idsByFilter`. `idsByFilter` is a new combined reducer that I'm creating.

**todos.js**
```javascript
const todos = combineReducers({
  byId,
  idsByFilter,
});
```
It combines a separate list of Ids for every filter. So it's `allIds` for the `all` filter, `activeIds` for `active` filter, and `completedIds` for the `completed` filter.

**todos.js**
```javascript
const idsByFilter = combineReducers({
  all: allIds,
  active: activeIds,
  completed: completedIds,
});
```
The existing `allIds` reducer manages an array of Ids, but it handles `ADD_TODO` action which we don't want to handle anymore, at least not yet, because for now we want to teach it to respond to the data fetched from the server.

To handle the `RECEIVE_TODOS` action, we want to return a new array of todos that we get from the server response. I map the response which is an array of todos to a function that just selects an Id from the todo. 

**todos.js**
```javascript
const allIds = (state = [], action) => {
  switch (action.type) {
    case `RECEIVE_TODOS`:
      return action.response.map(todo => todo.id);
    default:
      return state;
  }
}
```
We decided to keep `allIds` separate from `activeIds` and `completedIds`, so they are fetched completely independently.

I'm adding a new reducer called `activeIds` that also keeps track of an array of IDs, but for todos on the active tab. If we think about which actions it needs to handle, we'll realize that it also needs to handle `RECEIVE_TODOS` action in exactly the same way as all the IDs before it. Both `activeIds` and `allIds` need to return a new state when the `RECEIVE_TODOS` action fires.

**todos.js**
```javascript
const activeIds = (state = [], action) => {
  switch (action.type) {
    case `RECEIVE_TODOS`:
      return action.response.map(todo => todo.id);
    default:
      return state;
  }
}
```
However, we need to have a way of telling which ID array we should update.

If you recall the `RECEIVE_TODOS` action, you might remember that we passed the `filter` as part of the action. This lets me compare the `filter` inside the action with a `filter` that corresponds to the reducer. The `allIds` reducer is only interested in the actions with the `all` filter.

**todos.js**
```javascript
const allIds = (state = [], action) => {
  if (action.filter !== 'all') {
    return state;
  }
  switch (action.type) { ... }
}
```
Similarly, the `activeIds` reducer only wants to replace the state when the filter in the action is equal to `active`. 

**todos.js**
```javascript
const activeIds = (state = [], action) => {
  if (action.filter !== 'active') {
    return state;
  }
  switch (action.type) { ... }
}
```
Finally, I can copy and paste my `activeIds` reducer to get a stub for my `completedIds` reducer. I'll just place the word `active` with the word `completed`.

**todos.js**
```javascript
const completed = (state = [], action) => {
  if (action.filter !== 'completed') {
    return state;
  }
  switch (action.type) {
    case `RECEIVE_TODOS`:
      return action.response.map(todo => todo.id);
    default:
      return state;
  }
}
```
Now that I wrote all the reducers managing the IDs, I need to update the `byId` reducer to actually handle the new todos from the response. I'm removing the existing cases because the data does not live locally anymore. Instead, I'll handle the `RECEIVE_TODOS` action just like I do in the other reducers.

I'm creating a shallow copy of the state object which corresponds to the lookup table. For every `todo` object in the response, I want to take it and put it into the next version of the lookup table. I'm replacing whatever is in the next state by `todo.id` with the new `todo` I just fetched. Finally, I can return the next version of the lookup table from my reducer. 

**todos.js**
```javascript
const byId = (state = {}, action) => {
  switch (action.type) {
    case `RECEIVE_TODOS`:
      const nextState = { ...state };
      action.response.forEach(todo => {
        nextState[todo.id] = todo;
      });
      return nextState;
    default:
      return state;
  }
};
```
Normally the assignment operation is a mutation. However it's fine because `nextState` is a shallow copy, and we're only assigning one level deep. So we're not modifying any of the original state objects. The function itself stays [pure](https://egghead.io/lessons/javascript-redux-pure-and-impure-functions).

As the last step, I can remove the existing `todo` reducer because the logic for adding and toggling todos will be implemented as API calls to the server in the future lessons.

Let's run the app and inspect how the state changes. Before the action, the original state object contains the todos object with an empty lookup table and empty arrays of `Ids` for every filter. 

![Original State](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1553542112/transcript-images/javascript-redux-updating-the-state-with-the-fetched-data-original-state.jpg)

The action object contains the filter and the server response corresponding to this filter with todo objects inside of it.

![Action Object](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1553542111/transcript-images/javascript-redux-updating-the-state-with-the-fetched-data-action-object.jpg)

After handling the action, the todos object contains the updated lookup table that has every todo by its `Id`, as well as an updated list of Ids by filter where we only have fetched the old todos so far, so we have the IDs for `all` todos, but the other filters are amped in.

![Next State Object](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1553542111/transcript-images/javascript-redux-updating-the-state-with-the-fetched-data-next-state-object.jpg)

If I change the tab now, it will make another API request and store these IDs separately from the `all` IDs. Inside the `action` object, the filter is `active` now. The response contains only one `active` todo.

![Next State Object](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1553542111/transcript-images/javascript-redux-updating-the-state-with-the-fetched-data-active-todo.jpg)

Inside the `next state` object, the todos `byId` lookup table is essentially the same because we have not received any new todos. However, the IDs by filter object now contains an array of IDs for `all` filter and a separate array of IDs for the `active` filter.

We have **filter and logic to the server**, so switching a tab for the first time will take some time for the answers to load. However the **next switch** is filled instantaneous because we have **cached the array** of the fetched IDs. Even though we refresh them by fetching them again, the UI can already render the previous version of the array.

Let's recap how we updated this state in response to the action with the fetch data. The state shape of the `byId` reducer stayed the same. It's still a mapping of todo IDs to the todo objects. However, now it handles the `RECEIVE_TODOS` action when they are fetched from the server.

**todos.js**
```javascript
const byId = (state = {}, action) => {
  switch (action.type) {
    case `RECEIVE_TODOS`:
      const nextState = { ...state };
      action.response.forEach(todo => {
        nextState[todo.id] = todo;
      });
      return nextState;
    default:
      return state;
  }
};
```
It creates a shallow copy of the current mapping so that it can efficiently reassign its fellows inside a loop. It is a shallow copy. The assignment is only one level deep. So while this is a mutation, it does not modify any of the original state objects. So the reducer function stays pure.

The `action.response` field is an array of `todos` fetched from the server, so they get merged into the lookup table managed by `byId`.

Next, I created a separate reducer for every collection of IDs. They all handle the `RECEIVE_TODOS` action, but to make sure that they handle only the IDs they're interested in, we check the `filter`.

The `action.response` field is an array of todos, so we map them to their `Ids`. The reducers handling todo IDs handle `RECEIVE_TODOS` action in the same way but check for different filters. We have the all IDs for the `all` filter, the `active` IDs reducer for the active filter, and the `completed` IDs reducer for the completed filter.

**todos.js**
```javascript
const allIds = (state = {}, action) => {
  if (action.filter !== 'all') {
    return state;
  }
  switch (action.type) { ... }
};

const activeIds = (state = {}, action) => {
  if (action.filter !== 'active') {
    return state;
  }
  switch (action.type) { ... }
};

const completedIds = (state = {}, action) => {
  if (action.filter !== 'complete') {
    return state;
  }
  switch (action.type) { ... }
};
```
Finally, I'm using `combineReducers` to combine all of them into single `idsByFilter`. I use the `filter` values as the keys so that the corresponding state appears under those keys.

**todos.js**
```javascript
const idsByFilter = combineReducers({
  all: allIds,
  active: activeIds,
  completed: completedIds,
});
```
For example, to read the IDs corresponding to `all` filter, I can read state IDs by filter and pass all as a dynamic key. 

**todos.js**
```javascript
export const getVisibleTodos = (state, filter) => {
  const ids = state.idsByFilter[filter];
  return ids.map(id => state.byId[id]);
}
```
In order for `idsByFilter` to be in the state, I combine the todos reducer from by ID and `idsByFilter` combined reducer.

**todos.js**
```javascript
const todos = combineReducers({
  byId,
  idsByFilter,
})
```
This last my `getVisibleTodos` selector read the IDs for the corresponding `filter` and map them to the lookup table to get an array of todo objects that we can return to the components.