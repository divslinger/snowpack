## Configuration

Snowpack's behavior can be configured by CLI flags, a custom Snowpack config file, or both. [See the table below for the full list of supported options](#configuration-options).

### Config Files

Snowpack supports configuration files in multiple formats, sorted by priority order:

1. `--config [path]`: If provided.
1. `package.json`: A namespaced config object (`"snowpack": {...}`).
1. `snowpack.config.js`: (`module.exports = {...}`).
1. `snowpack.config.json`: (`{...}`).

### CLI Flags

``` bash
$ snowpack --help
```

CLI flags will be merged with (and take priority over) your config file values. Every config value outlined below can also be passed as a CLI flag. Additionally, Snowpack also supports the following flags:

- **`--help`** Show this help.
- **`--version`** Show the current version. 
- **`--reload`** Clear the local CDN cache. Useful when troubleshooting installer issues.


### All Config Options

```js
{
  "include": "src/",
  "additionalEntrypoints": [
    "htm",
    "preact",
    "preact/hooks", // A package within a package
    "unistore/full/preact.es.js", // An ESM file within a package (supports globs)
    "bulma/css/bulma.css" // A non-JS static asset (supports globs)
  ],
  "installOptions": { /* ... */ },
  "buildOptions": { /* ..... */ }
}
```

#### Top-Level Options

- **`extends`** | `string`
  - Inherit from a separate "base" config. Can be a relative file path, an npm package, or a file within an npm package. Your configuration will be merged on top of the extended base config.
- **`include`** | `string`
  - Your source directory, if one exists. Defaults to `"src/"` if a local `src/` directory exists in your project. 
  - **This is a special directory for Snowpack.** Snowpack will scan for package imports and run custom build "scripts" on these source files. All `src/*` files will be re-written to `/_dist_/*` in the final build (see  `buildOptions.dist` option below for more info).
- **`exclude`** | `string[]`
  - Exclude any files from the `--include` directory. Defaults to exclude common test file locations: `['**/__tests__/*', '**/*.@(spec|test).@(js|mjs)']`
  - Useful for excluding tests and other unnecessary files from the final build. Supports glob pattern matching. 
- **`additionalEntrypoints`** | `string[]`
  - Set any additional package entrypoints to install with Snowpack. 
  - Useful for imports that couldn't be detected by our scanner (ex: package CSS files).
- **`installOptions.*`** | `object`
  - Configure how npm packages are installed. See the section below for all options.
- **`buildOptions.*`** | `object`
  - Configure your dev server and build workflows. See the section below for all options.

#### Install Options

- **`dest`** | `string`
  - *Default:`"web_modules"`*
  - Configure the install directory.
- **`clean`** | `boolean`
  - *Default:`true`*
  - Delete the existing `dest` directory (any any outdated files) before installing.
- **`env`** | `{[ENV_NAME: string]: (string | true)}`
  - Sets a `process.env.` environment variable inside the installed dependencies. If set to true (ex: `{NODE_ENV: true}` or `--env NODE_ENV`) this will inherit from your current shell environment variable. Otherwise, set to a string (ex: `{NODE_ENV: 'production'}` or `--env NODE_ENV=production`) to set the exact value manually.
- **`babel`** | `boolean`
  - Transpile your installed dependencies to run on older browsers. 
- **`installTypes`** | `boolean`
  - Install TypeScript type declarations with your packages. Requires changes to your [tsconfig.json](#TypeScript) to pick up these types. 
- **`sourceMap`** | `boolean`  
  - Emit source maps for installed packages.
- **`rollup`**
  - Snowpack uses Rollup internally to install your packages. This `rollup` config option gives you deeper control over the internal rollup configuration that we use. 
  - **`rollup.plugins`** - Specify [Custom Rollup plugins](#custom-rollup-plugins) if you are dealing with non-standard files.
  - **`rollup.dedupe`** - If needed, deduplicate multiple versions/copies of a packages to a single one. This helps prevent issues with some packages when multiple versions are installed from your node_modules tree. See [rollup-plugin-node-resolve](https://github.com/rollup/plugins/tree/master/packages/node-resolve#usage) for more documentation.
  - **`rollup.namedExports`** - If needed, you can explicitly define named exports for any dependency. You should only use this if you're getting `"'X' is not exported by Y"` errors without it. See [rollup-plugin-commonjs](https://github.com/rollup/rollup-plugin-commonjs#usage) for more documentation.

#### Build Options

- **`port`** | `number` | Default: `3000`
  - The port number to run the dev server on.
- **`out`** | `string` | Default: `"build"`
  - The local directory that we output your final build to.
- **`dist`** | `string` | Default: `"/_dist_"`
  - The final URL path location for your "include" (`src/`) directory.
  - When Snowpack builds your application (or serves it during development) Snowpack will rewrite all `src/*` files to be hosted at the `/_dist_/*` URL path.
- **`fallback`** | `string` | Default: `"index.html"`
  - When using the Single-Page Application (SPA) pattern, this is the HTML "shell" file that gets served for every (non-resource) user route. Make sure that you configure your production servers to serve this as well.
- **`bundle`** | `string` | Default: `false`
  - Bundle the build output. Useful for production builds when performance is important.


## Build Scripts

By default, Snowpack will act a lot like a static file server for your current working directory. But, if you have a project `src/` directory, Snowpack can be extended to build, transpile, or otherwise transform any file in that directory before serving it to the browser. 

These transformations are called "Build Scripts".

### What are Build Scripts?

Build scripts allow you to customize how different source files in your application "src/" directory are processed before being served. This can be useful or even essential in some projects that rely on custom syntax. Svelte, vue, and even React all use custom syntax (ex: JSX) that needs to be processed in some way (ex: passed through Babel) before the browser will understand it. 

If you've ever used package.json scripts with npm or yarn, Snowpack's build scripts should feel familiar. Each script is a simple bash/CLI command. Based on the script ID (name) we pipe your source code into each command (via stdin) and then write it's output (via stdout) as the final file to serve to the browser.

### Examples

**"build:" is the basic building block for the Snowpack dev & build pipeline.** In this example, we pipe different file types (matched by the file extension(s0) in the script name) through different tools. 

```js
// snowpack.config.json
{
  "scripts": {
    // Run every .js & .jsx file through Babel CLI
    "build:js,jsx": "babel",
    // Run every .css file through PostCSS CLI
    "build:css": "postcss",
    // Run every .svg file through 'cat', essentially copying it without transforming
    "build:svg": "cat"
  }
}
```

`babel`, `postcss`, and `cat` in the example above all represent the literal CLI command of that name. `cat` tells Snowpack that ".svg" files don't need to be transformed, and can be outputted to build exactly as read from disk. 


### All Script Types

Snowpack supports several other script types in addition to the basic `"build:*"` script. These different types serve different goals so that you can fully customize and control your dev environment:

- `"build:*": "..."`
  - Pipe any matching file into this CLI command, and write it's output to disk.
  - ex: `"build:js,jsx": "babel`
- `"lint:*": "..."`
  - Pipe any matching file into this CLI command, and log any output.
  - ex: `"lint:js": "eslint"`
- `"lintall:*": "..."`
  - Run a single command once, log any output.
  - Useful for tools like TypeScript that lint multiple files / entire projects at once.
  - ex: `"lint:ts,tsx": "tsc"`
- `"copy:*": "copy DIR [--to URL]`
  - Copy a folder directly into the final build at the `--to` URL location.
  - If no `--to` argument is provided, the folder will be copied to the same location relative to the project directory.
  - ex: `"copy:public": "copy public --to /"`
  - ex: `"copy:web_modules": "copy web_modules"`
- `"plugin:*": "..."`
  - Connect a custom Snowpack plugins. See the section below for more info.

#### Script Modifiers

Additionally, we support script modifiers via  the `"::"` token. These are addons to a previous matching script that extend that script's behavior:

- `"lintall:*::watch"`
  - This adds a watch mode to a previous "lintall" script, so that you can turn any supported linter into a live-updating watch command during development. 
  
```js
// snowpack.config.json
{
  "scripts": {
    // Run TypeScript to lint your project.
    "lintall:ts,tsx": "tsc --noEmit",
    // Run TypeScript in --watch mode during development for live feedback.
    "lintall:ts,tsx::watch": "$1 --watch",
  }
}
```

Note that `$1` is used in these script modifiers to original script. This is useful so that you don't need to copy-paste the original script in two places.


### Script Plugins

For an even more powerful integration, Snowpack supports first-class plugins built specifically for Snowpack. Instead of running these plugins as CLI commands, each plugin is loaded as a JavaScript module that exports custom `build()` and `lint()` functions.

There are a few reasons you may want to use a plugin instead of a normal "build:" or "lint:" CLI command script:

**Speed:** Some CLIs may have a slower start-up time, which may become a problem as your site grows. Plugins can be faster across many files since they only need to be loaded & initialized once and not once for every file.

```js
"scripts": {
  // Babel plugin is 10x faster than using the Babel CLI directly
  "plugin:babel": "@snowpack/plugin-babel",
}
```

**Lack of CLI:** Some frameworks, like Svelte, don't maintain dedicated CLIs. Snowpack Plugins allow you to tap into a tool's JS interface directly without building a whole new CLI interface.

```js
"scripts": {
  // Lack of CLI: There is no Svelte CLI. Our plugin taps directly into the Svelte compiler 
  "plugin:svelte": "@snowpack/plugin-svelte",
}
```

**Greater Control:** A plugin is just a set of JavaScript functions, so it's easy to build your own local plugins using JavaScript if you prefer to write your own source code transformation.


```js
"scripts": {
  // Custom Behavior: Feel free to build your own!
  "plugin:vue": "./my-custom-vue-plugin.js",
}
```

### Default Transformations

If you've defined a 'src/' directory in your project, Snowpack will run some simple transformations on any JavaScript files within that directory by default:

- **Package Import URL Rewriting:** Any package imported by name will get re-written to point to the hosted package URL. `import 'react'` would become `import '/web_modules/react.js'`, etc. 
- **Legacy Browser Support (Build only!):** All JS files will be transpiled to run on the following set of browsers: `">0.75%, not ie 11, not UCAndroid >0, not OperaMini all"`.  

Additionally, the following transformations will run only if no other build script is detected for that file extension. 

- **TypeScript Support:** All TypeScript syntax will be stripped from `.ts` and `.tsx` files before being output to the browser as `.js`.
- **JSX Support:** All JSX syntax will be stripped from `.jsx` and `.tsx` files before being output to the browser as `.js`. Note that this only runs if we also detect "react" or "preact" as an imported package of the file itself.
