Part 3
# Webpack & Angular

In Part 1 & Part 2 we prepared setting up the project, all for this moment. Let's take advantage of using Webpack & Angular for creating modular code.

For some reason, Webpack & Angular reminds me of late 90s Acne commercials.

> It keeps your code: clean, clear & under control

What a terrible way to start a blog post, but I'll stick with it. No regrets.

Let's look at some great features of using Webpack's require while making a navbar directive.

### require('module').name

First of all, we'll need a module for handling our layout directives.

/app/core/layout.js

```js
export default angular.module('app.layout', [])
```

We can simply require the layout by its location.name, allowing us to change module names at any time.
Just make the loaded module a dependency.

/app/index.js

```js
module.exports = angular.module('app', [
  /* 3rd party */
  'lumx',
  /* modules */
  require('./core/layout').name
]);
```

### Modular directive names

Let's setup the navbar html.

/app/core/nav/nav.html

```html
<header class="header bgc-light-blue-600" ng-cloak>
<!-- Get the app info and put it in the navbar on the left -->
  <h1 class="main-logo">
    <a href="/" class="main-logo__link" lx-ripple="white">
      <span class="main-nav--title">{{::nav.app.title}} </span>
      <span class="main-nav--version">v{{::nav.app.version}}</span>
    </a>
  </h1>
<!-- Loop over the links and add them to the navbar on the right -->
  <nav class="main-nav main-nav--lap-and-up">
    <ul>
      <li ng-repeat="n in nav.app.links">
        <a href="{{::n.link}}" class="main-nav__link" lx-ripple="white">
          {{::n.text}}</a>
      </li>
    </ul>
  </nav>
</header>
```

Styles: 

/app/core/nav/nav.scss

```scss
.header {
  position: fixed;
  top: 0;
  right: 0;
  left: 0;
  z-index: 999;
  height: 60px;
  padding: 12px;
  color: white;
  background-color: #4fc1e9;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.3);
}
```

For the full `nav.scss` file, get it on [Github](). This is a short/ugly version.

Next add the controller (using ES6 classes) and directive:

/app/core/nav/nav.js

```js
class NavCtrl {
  constructor() {
    this.app = {
      title: 'Module Loaders',
      version: '0.3.0',
      links: [{
        text: 'Webpack',
        link: 'http://webpack.github.io'
      }, {
        text: 'Require.js',
        link: 'http://requirejs.org/'
      }, {
        text: 'Jspm',
        link: 'http://jspm.io/'
      }]
    };
  }
}
export default () => {
  require('./nav.scss');  // load styles for the component
  return {
    controller: NavCtrl,
    controllerAs: 'nav',
    templateUrl: 'app/core/nav/nav.html'
  };
};
```

Notice the directive was never named. You really only need to name it once, on its angular.module.

/app/core/layout.js

```js
export default angular.module('app.layout', [])
  .directive('lumxNavbar', require('./nav/nav');
```

This way name changes remain very flexible. 

Add the directive and you should be able to see our working navbar.

/app/index.html

```html
<body>
<lumx-navbar></lumx-navbar>
<!-- ... -->
```



### require(templates)

This `lumxNavbar` templateUrl doesn't allow us to move things around very much: `app/core/nav/nav.html`.

We can simplify this with the [`html-loader`](https://github.com/webpack/raw-loader). Install it as a dev dependency.

`npm install -D raw-loader`

Add another loader to our `app/webpack.config.js` file.

```js
{
    test: /\.html/,
    loader: 'raw'
}
```
Note: Restart the webpack-dev-server for any config changes to take effect.
And now we can require html files using relative paths, this makes it much easier to move folders around.

/app/core/nav/nav.js

```js
/* old templateUrl: 'app/core/nav/nav.html' */
template: require('./nav.html')
```

### require(json)

By now you're probably getting the hang of loaders. We want to load some json, get a [`json-loader`](https://github.com/webpack/json-loader).

`npm install -D json-loader`

Add it to the `webpack.config.js`.

/app/webpack.config.js

```js
{
    test: /\.json/,
    loader: 'json'
}
```

We can put all of our main data into an `index.json` file, so when we make navbar frequent navbar changes in the future, we won't have dive into `/app/core/nav/nav`.

/app/index.json

```json
{
  "title": "Module Loaders",
  "version": "0.3.0",
  "links": [{
    "text": "Webpack",
    "link": "http://webpack.github.io"
  }, {
    "text": "Require.js",
    "link": "http://requirejs.org/"
  }, {
    "text": "Jspm",
    "link": "http://jspm.io/"
  }]
}
```

Let's require this file in `nav.js`

/app/core/nav/nav.js

```js
class NavCtrl {
  constructor() {
    this.app = require('../../index.json');
  }
}
```

 
But what if we move the `nav` folder around? Wouldn't an absolute path to `app/index.json` be more useful?

Luckily, with webpack we can use both.

### require(absolute & || relative paths)

A relative path points to files relative to the current directory.
```
parent.js
├── file.js
│   ├── folder
│   │       └──child.js
```
- Parent `../parent.js`
- At the same level `./file.js`
- Nested `./folder/child.js`

But if you want to move the `folder` or `child.js` around, this path will break. Sometimes it's better to use an absolute path.

To do this with Webpack, we must first tell `webpack.config` our absolute root.

/app/webpack.config.js

```js
module.exports = {
  /* ... */
  resolve: {
    root: __dirname + '/app'
  }
};
```

Now we can point to our `index.json` file in a much cleaner way.
Note: Again, you'll have to restart the webpack-dev-server for any config changes to take effect. 

```js
class NavCtrl {
  constructor() {
    /* old this.app = require('../../index.json'); */
    this.app = require('index.json');
  }
}
```

### If () { require('module') }



