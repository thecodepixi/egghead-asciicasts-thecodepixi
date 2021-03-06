We currently represent the todos in the state tree as an array of todo objects. However, in the real app, we probably have more than a single array and then todos with the same IDs in different arrays might get out of sync.

This is why I wanted to **treat my state as a database**, and I'm going to keep my todos in an object indexed by the IDs of the todos. I've renamed the reducer to `byId` and rather than add a new item at the end or map over every item, now it's going to change the value in the lookup table.

Both `TOGGLE_TODO` and `ADD_TODO` object is now going to be the same. I want to return a new **lookup table** where the value unto the ID in the action is going to be the result of calling the reducer on the previous value under reducer ID and the action.

**todos.js**
```javascript
const byId = (state = {}, action) => {
  switch (action.type) {
    case 'ADD_TODO':
    case 'TOGGLE_TODO':
      return {
        ...state,
        [action.id]: todo(state[action.id], action),
    };
    default:
      return state;
  }
};
```

This is to reduce a composition but with an object instead of an array. I'm also using the **object spread operator** here. It is **not** a part of ES6 so you need to enable `transform-object-rest-spread` in babel reducer file, and you need to install the `babel-plugin-transform-object-rest-spread` for this to work.

**.babelrc**
```
{
  "presets": ["es2015", "react"],
  "plugins": ["transform-object-rest-operator"]
}
```
Anytime the `byId` reducer receives an action, it's going to return the copy of its mapping between the IDs and the actual todos with updated todo for the current action. I will let another reducer that keeps track of all the added IDs.

We keep the todos themselves in the `byId` map now, so the state of this reducer is an array of IDs rather than array of todos. I `switch` on the action type and the only action I care about is `ADD_TODO` because if a new todo is added, I want to return a new array of IDs with that ID as the last item.

**todos.js**
```javascript
const allIds = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.id];
  }
};
```

For any other actions, I just need to return the current state, that is, the current array of IDs. Finally, I still need to export the single reducer from the todos file, so I'm going to use `combinedReducers` again to combine the `byId` and the `allIds` reducers.

**todos.js**
```javascript
const allIds = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.id];
    default: 
      return state;
  }
};

const todos = combineReducers({
  byId,
  allIds,
});
```

You can use `combinedReducers` as many times as you like. You don't have to only use it on the top-level reducer. In fact, it's very common that as your app grows, you'll use combine reducers in several places.

**todos.js**
```javascript
import { combineReducers } from 'redux';
```

Now that we have changed the state shape in reducers, we also need to update the selectors that rely on it. The `state` object then `getVisibleTodos` is now going to contain `byId` and `allIds` fields, because it corresponds to the C of the combined reducer.

Since I don't have the array of todos anymore, I will write the selector that is going to calculate it for me. I won't export it because I only plan to use it in the current file and it takes the current state and returns all the todos by mapping AllIDs to the state ByID lookup table.

**todos.js**
```javascript
const getAllTodos = (state) => 
  state.allIds.map(id => state.byId[id]);
```

I will use this name selector inside my `getVisibleTodos` selector to obtain an array of todos that I can filter. `allTodos` is an array of todos just like the components I expect so I can return it from the selector and not worry about change in my component code.

**todos.js**
```javascript
export const getVisibleTodos = (state, filter) => {
  const allTodos = getAllTodos(state);
  switch (filter) {
    case 'all':
      return allTodos;
    case 'completed':
      return allTodos.filter(t => t.completed);
    case 'active':
      return allTodos.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};
```

My todos file has grown quite a bit so it's a good time to extract the `todo` reducer that manages just a single todo into a separate file of its own. I created a file called todo in the same folder and I will paste my implementation right there so that I can import it from the todos file.

**todos.js**
```javascript
import todo from './todo';
```



Let's recap how we change those state structures to be normalized and more like a database. I just extracted the `todo` reducer into a separate file but it hasn't changed in the state.

**/reducers/todo.js** 
```javascript
const todo = (state, action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        id: action.id,
        text: action.text,
        completed: false,
      };
    case 'TOGGLE_TODO':
      if (state.id !== action.id) {
        return state;
      }
      return {
        ...state,
        completed: !state.completed,
      };
    default:
      return state;
  }
};

export default todo;
```

It's still a reducer that handles updates to a single todo item. I'm using this reducer inside the new reducer I wrote today, which is called `byId` and its state shape is an object. It reads the ID of the todo to update from the action and it calls the `todo` reducer with the previous state of this ID and the action.

**todos.js**
```javascript
const byId = (state = {}, action) => {
  switch (action.type) {
    case 'ADD_TODO':
    case 'TOGGLE_TODO':
      return {
        ...state,
        [action.id]: todo(state[action.id], action),
    };
    default:
      return state;
  }
};
```

For the `ADD_TODO` action, the corresponding todo will not exist in the lookup table yet. We're calling the `todo` reducer with `undefined` as the first argument. The `todo` reducer would then return a new todo object when handling `ADD_TODO` so this object will get assigned under the `action.id` key inside the next version of the lookup table.

Notice how we're mixing the **objects spread operator** with the **computed property syntax**, which lets us specify a value at a dynamic key inside `action.id`. Also, remember that the objects spread operator is experimental and you need a special babel plugin to enable it.

I also wrote a second reducer called `addIds` that manages just the array of IDs of the todos. Every time a todo is added, it returns a new array with this ID of the new todo at the very end.

**todos.js**
```javascript
const allIds = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.id];
    default: 
      return state;
  }
};
```

It uses the **array-spread operator**, which is part of ES6, to produce a new array with the `allIds` followed by the new ID. The two reducers now handle the same action. This is fine and very common in the Redux apps.

I'm combining the two reducers I wrote into a single reducer by calling `combineReducers` provided by Redux. You may use `combineReducers` at multiple levels in your reducer hierarchy.

**todos.js**
```javascript
const todos = combineReducers({
  byId,
  allIds,
});

Since the state shape changed, I needed to update the selectors that depend on it. I wrote the private selector called `getAllTodos` that just assembles all the todos objects from the state by mapping the IDs to the lookup table.

**todos.js**
```javascript
const getAllTodos = (state) => 
  state.allIds.map(id => state.byId[id]);
```

For every ID, we get the todo from `state.byId`. I'm returning the array of todos with exactly the same shape as the state inside `getVisibleTodos` used to be before this change.

**todos.js**
```javascript
export const getVisibleTodos = (state, filter) => {
  const allTodos = getAllTodos(state);
  switch (filter) {
    case 'all':
      return allTodos;
    case 'completed':
      return allTodos.filter(t => t.completed);
    case 'active':
      return allTodos.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};
```

I can get `allTodos` and now I can use `allTodos` for filtering and I can return it to the new ID that doesn't need to be changed, because all the state shape knowledge is encapsulated in the selector.