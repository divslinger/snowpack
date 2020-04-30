## Basic Usage

### snowpack install

An essential part of what Snowpack does is prepare your npm packages to run directly in the browser.NPM packages installed with npm or yarn can't do this by default, so Snowpack runs a special one-time task on a subset of your dependencies known as "web dependencies". Snowpack automatically installs your Web dependencies to run directly in the browser without a bundler:

```diff
{
  "dependencies": {
    "@babel/core": "^1.2.3",
-   /* previously managed by npm */
-   "react": "^16.13.0",
-   "react-dom": "^16.13.0"
  },
+ /* now managed by Snowpack */
+ "webDependencies": {
+   "react": "^16.13.0",
+   "react-dom": "^16.13.0"
+ }
}
```
``` bash
$ snowpack install
✔ snowpack installed: react, react-dom.
```

When you run `snowpack install`, Snowpack installs your `webDependencies` to the "web_modules/" directory in your current project directory. 

You can think of this as an alternative to npm for your frontend web dependencies. When Snowpack installs your npm dependencies, it does the work upfront to convert each package to a web-ready, single JS file that runs natively in your browser. 


### snowpack add/rm

Snowpack provides `add` & `rm` helper commands to help you manage your "webDependencies" config via the CLI (see above).

### snowpack dev

Snowpack ships with an bundle-free dev server built for the fastest possible development speed. By default, it acts a lot like a simple static file server: load any directory in the browser as if it were a hosted web app. 

By default, Snowpack will automatically rewrite your package imports to valid browser-native URLs. This lets you write your JavaScript imports by name without having to hardcode URLs into your source code.

``` js
// Input: Your source code
import * as React from "react"; 

// Output: Gets transformed to a URL that the browser can understand
import * as React from "/web_modules/react.js"; 
```

If you prefer a low-tooling dev environment, the default dev server behavior is enough to build even complex applications with HTML, CSS and JavaScript. 

Some popular frameworks, however, often rely on custom syntax that doesn't run natively in the browser. Svelte, Vue, and even React all use custom syntax (ex: JSX) that needs to be processed in some way (ex: passed through Babel) before the browser will understand it. 

For these cases, we provide a simple "scripts" interface to customize how each file type in your application is processed. If you're using one of the frameworks listed above, check out the "scripts" section below.

### snowpack build

When you're ready to deploy your application, run `snowpack build`. Snowpack's build workflow integrates directly with the dev server, so that you get a build guarenteed to match exactly what you saw during development. Snowpack will automatically run any custom "scripts" across your source code, transforming files and outputting them to their final location in a static, ready-to-deploy `build/` directory.

### snowpack build --bundle

The default output of the `snowpack build` command is an exact copy of your unbundled dev site. Deploying this would be fine for most sites, but you may want to optimize your site for production by bundling common files together. If that's you, Snowpack supports production bundling via a simple, zero-config `--bundle` flag. 

`snowpack build --bundle` runs your final build through [Parcel](https://parceljs.org/), a popular web application bundler. By bundling together your JavaScript and CSS files into larger shared chunks, you may see a production speed up as your users have fewer files to download. 
