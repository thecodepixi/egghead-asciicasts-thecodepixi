My `mapStateToProps` function uses the `getVisibleTodos` function, and it passes the slice of the state corresponding to the todos. However, if I ever change the state structure, I'll have to remember to update this whole side.

**VisibleTodoList**
```javascript
const mapStateToProps = (state, { params }) => ({
  todos: getVisibleTodos(state.todos, params.filter || 'all')
});
```

Alternatively, I can move the `getVisibleTodos` function out of my view layer and place it in the file that knows best about the `state.todos` internal structure.

The file that determines the internal structure of todos is the file that contains the `todos` reducer. This is why I'm placing my `getVisibleTodos` implementation right into the file with reducers, and I'm making it a **named export**.

**todos.js**
```javascript
export const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'all';
      return todos;
    case 'completed';
      return todos.filter(t => t.completed);
    case 'active':
      return todos.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};
```

The convention I follow is simple. **The default export is always the reducer function**, but any named export starting with `get` is a function that prepares the data to be displayed by the UI. We usually call these functions **selectors** because they select something from the current state.

In the reducers, the `state` argument corresponds to the state of this particular reducer. I'll follow the same convention in selectors, where the state argument will correspond to the state of the exported reducer in this file.

**todos.js**
```javascript
export const getVisibleTodos = (state, filter) => {
  switch (filter) {
    case 'all';
      return state;
    case 'completed';
      return state.filter(t => t.completed);
    case 'active':
      return state.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};
```

Going back to my component, I still depend on the state structure because I read the todos from the state. The actual method of reading todos may change in the future.

**VisibleTodoList**
```javascript
const mapStateToProps = (state, { params }) => ({
  todos: getVisibleTodos(state.todos, params.filter || 'all')
});
```

I am opening the file that contains the root reducer, and I add a named selector export there, as well. It is also called `getVisibleTodos`, and it also accepts the `state` and the `filter`, but the state corresponds to the state of the combined reducer.

**index.js**
```javascript
export const getVisibleTodos = (state, filter) =>
```

Now I want to call the `getVisibleTodos` function defined in the todos file alongside the reducer, but I can't use a named import because I have function with exactly the same name in the scope.

This is why I'm using the **namespace import syntax** that puts all the exports on an object, called `fromTodos` in this case, so I can use `fromTodos.getVisibleTodos` to call this function I defined in the other file, and pass the slice of the state corresponding to the todos.

**index.js**
```javascript
export const getVisibleTodos = (state, filter) =>
  fromTodos.getVisibleTodos(state.todos, filter);
```

Now I can go back to my component, and I can import `getVisibleTodos` from the root reducer file. It encapsulates all the knowledge about the application state shape, so I can just pass it the whole state of my application, and it will figure out how to select the visible todos according to the logic described in selectors.

**VisibleTodoList**
```javascript
const mapStateToProps = (state, { params }) => ({
  todos: getVisibleTodos(state.todos, params.filter || 'all')
});
```

Let's recap how I co-locate the selectors with the reducers. Inside `mapStateToProps`, I call `getVisibleTodos`, and I pass it the whole application state.

The fresh value of the `state` will be passed anytime it changes to the `mapStateToProps` function. I import `getVisibleTodos` selector, and notice the curly brace. This is a named import from the file that defines the root reducer.

**VisibleTodoList**
```javascript
import { getVisibleTodos } from '../reducers';
```

In the reducer files, I started adding new methods with `get` prefix that are exported as named exports, as opposed to the reducer, which is exported as a `default` export. It's easy to tell a named export because it doesn't use the `default` keyword after the `export` keyword.

**index.js**
```javascript
export const getVisibleTodos = (state, filter) =>
  fromTodos.getVisibleTodos(state.todos, filter);
```

I want to limit the knowledge about the exact state shape to the files contained in the reducers that manage this state. `todos` is one of the reducers from which the root reducer is combined, so we know that its state is available as a `state.todos` under the todos key.

However, the state shape of the todos reducer should be encapsulated in the file where it's defined. This is why I'm delegating the further selection to the `getVisibleTodos` I get from the from `todos` object that I import as a namespace import from the todos file, so I get all the named exports from it.

**index.js**
```javascript
import todos, * as fromTodos from './todos';
```

In the todos file, I define my reducer as the `default` export. But I also define the `getVisibleTodos` implementation as a named export, and this time, the `state` refers to the state of just the corresponding reducer, which in this case is an array of todos.

**todos.js**
```javascript
export const getVisibleTodos = (state, filter) => {
  switch (filter) {
    case 'all':
      return state;
    case 'completed':
      return state.filter(t => t.completed);
    case 'active':
      return state.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};
```

**todos.js**
```javascript
const todos = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        todo(undefined, action),
      ];
    case 'TOGGLE_TODO':
      return state.map(t =>
        todo(t, action)
      );
    default:
      return state;
  }
};
```

If I later want to change it to be something other than an array, I can change the `getVisibleTodos` implementation. But I won't have to touch my components, because they don't rely on the state shape anymore.