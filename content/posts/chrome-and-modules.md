---
title: "Chrome and Modules"
date: 2017-09-20
---

## Chrome is ready for modules but the Modules aren't ready for Chrome ##

As of Chrome 61 (released September 2017) and Safari 10.1 (released March 2017) we've had browsers that were for all intents and purposes ES6 [complete](http://kangax.github.io/compat-table/es6/), including full support of [Modules](http://beta.caniuse.com/?feat=i#feat=es6-module). Ideally, when developing, you should be able to use all the features of ES6 and the convenience of modules **without** the need for a transpiler like Babel or other build tools / file watchers. If you're strictly developing your own self-contained solution with **no** other libraries, then yes this can work for you, right now.

You are going to run into problems quickly if you want to use other libraries. Let's take what should be a simple case of importing a library purely for the side effects. Take [d3](https://d3js.org/) for example. As of version 4 it is split up into modules and the "main" library of d3 rolls up the most popular/common of these modules and will export it to the global context. Like many other modern, properly built libraries, d3 uses Rollup to create a UMD script for the browser and even another script for using in Node. 

### Side Effect Import
`import "./node_modules/d3/build/d3.js";` 
### NOPE

The problem is the UMD that Rollup makes doesn't provide a context when there is none. 
```js
(function (global, factory) {
	...
}(this, (function (exports) { 'use strict';
```
`this` is undefined when importing a module in ES6. For the UMD to work, it needs to be `this || Window || self || global` to really work in all contexts.

---

How about just using a library that was meant to be imported as modules. We are using this for development and don't care that it is a few more files to load. We are still planning on using a build process in the end. Let's just take the **collection** module of d3 and attempt to import that. 

### Library Module Import
`import * as collect from "./node_modules/d3-collection/index.js"`

index.js contains
```js
export {default as nest} from "./src/nest";
export {default as set} from "./src/set";
export {default as map} from "./src/map";
export {default as keys} from "./src/keys";
export {default as values} from "./src/values";
export {default as entries} from "./src/entries";
```
### NOPE

Why? Because the import paths don't have `.js`. You get 404 errors instead of module imports.

---

What if we get really crazy and go **ALL the way down** to an individual module. Let's just use the **nest** module.

### Individual Module Import
`import nest from "./node_modules/d3-collection/src/nest.js"`
### YES

Well we _finally_ got what we wanted, but we had to go to the micro module level to get it. Could you imagine trying to import all the modules included in d3 this way?!?

---

How about using example starter projects or various scaffolding tools that create project skeletons for you? You will run into the same problem as when we attempted the Library Module Import. Almost none of these projects or generators will produce import statements that use the `.js` in the path. So instead of your own module imports, you get 404 errors.

## The Future
The easiest change to make would be to update the UMD template in Rollup and other build tools to include fallback defaults for the `this` context. That change would allow side effect libraries to be easily imported. Next, scaffolding tools and starter projects should include full paths to modules. Until these changes are standard practice, you will still need some kind of live transpiler or module loader/packager to develop in ES6, even if you only use ES6 complete browsers during development.

