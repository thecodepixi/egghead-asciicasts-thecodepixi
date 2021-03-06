To add **React Router** to the project, I'm running `npm install --save react-router`. I'm going to import the `Router` component from it, as well as the `Route` configuration component. Now I can replace my app with the `Router`, and it's important that it's inside `Provider` so that any components rendered by the `Router` still have access to the `store`.

**Root.js**
```javascript
import { Router, Route } from 'react-router';
import App from './App';

const Root = ({ store }) => (
  <Provider store={store}>
    <Router>

    </Router>
  </Provider>
);
```

Inside, I put a single `Route` element that tells **React Router** that I want to render my `App` component at the root path. If I run the app, I can see that the `Router` matched the path correctly and rendered the `App` component.


![output](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1553542110/transcript-images/javascript-redux-adding-react-router-to-the-project-output.jpg)


If you see weird symbols after a hash sign in the address bar, it means that you're using the version of **React Router** that doesn't yet default to the `browserHistory`, and defaults to hash history instead.

To fix it, you can import `browserHistory` from React Router and pass it as a `history` prop to Router. Unless you target very old browsers like IE9, you can always use `browserHistory` and have a clean URL in the address bar.

**Root.js**
```javascript
import { Router, Route browserHistory} from 'react-router';
import App from './App';

const Root = ({ store }) => (
  <Provider store={store}>
    <Router history={browserHistory}>
      <Route path='/' component={App} />
    </Router>
  </Provider>
);
```

Let's recap the changes we made to add React Router to the application. I ran `npm install --save react-router`, and I imported the `Router` component and the `Route` configuration component from React Router.

**Root.js**
```javascript
import { Router, Route browserHistory} from 'react-router';
```

Instead of rendering the app directly, I replaced it with a `Router` component that has a single `route` at the root path that renders the `App` component. In order to avoid hash sign and weird symbols after it, I imported `browserHistory`, and I passed it as the `history` prop to the `Router` component.

**Root.js**
```javascript
const Root = ({ store }) => (
  <Provider store={store}>
    <Router history={browserHistory}>
      <Route path='/' component={App} />
    </Router>
  </Provider>
);
```