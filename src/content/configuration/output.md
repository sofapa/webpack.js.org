---
title: Output
sort: 5
contributors:
  - sokra
  - skipjack
  - tomasAlabes
  - mattce
  - irth
  - fvgs
  - dhurlburtusa
---

The top-level `output` key contains set of options instructing webpack on how and where it should output your bundles, assets and anything else you bundle or load with webpack.


## `output.auxiliaryComment`

`string` `object`

When used in tandem with [`output.library`](#output-library) and [`output.libraryTarget`](#output-librarytarget), this option allows users to insert comments within the export wrapper. To insert the same comment for each `libraryTarget` type, simply set `auxiliaryComment` to a string:

``` js
output: {
  library: "someLibName",
  libraryTarget: "umd",
  filename: "someLibName.js",
  auxiliaryComment: "Test Comment"
}
```

which will yield the following:

``` js
(function webpackUniversalModuleDefinition(root, factory) {
  // Test Comment
  if(typeof exports === 'object' && typeof module === 'object')
    module.exports = factory(require("lodash"));
  // Test Comment
  else if(typeof define === 'function' && define.amd)
    define(["lodash"], factory);
  // Test Comment
  else if(typeof exports === 'object')
    exports["someLibName"] = factory(require("lodash"));
  // Test Comment
  else
    root["someLibName"] = factory(root["_"]);
})(this, function(__WEBPACK_EXTERNAL_MODULE_1__) {
  // ...
});
```

For fine-grained control over each `libraryTarget` comment, pass an object:

``` js
auxiliaryComment: {
  root: "Root Comment",
  commonjs: "CommonJS Comment",
  commonjs2: "CommonJS2 Comment",
  amd: "AMD Comment"
}
```


## `output.chunkFilename`

`string`

This option determines the name of non-entry chunk files. See [`output.filename`](#output-filename) option for details on the possible values.

Note that these filenames need to be generated at runtime to send the requests for chunks. Because of this, placeholders like `[name]` and `[chunkhash]` need to add a mapping from chunk id to placeholder value to the output bundle with the webpack runtime. This increases the size and may invalidate the bundle when placeholder value for any chunk changes.

By default `[id].js` is used or a value inferred from [`output.filename`](#output-filename) (`[name]` is replaced with `[id]` or `[id].` is prepended).


## `output.chunkLoadTimeout`

`integer`

Number of milliseconds before chunk request expires, defaults to 120 000. This option is supported since webpack 2.6.0.


## `output.crossOriginLoading`

`boolean` `string`

Only used when [`target`](/configuration/target) is web, which uses JSONP for loading on-demand chunks, by adding script tags.

Enable [cross-origin](https://developer.mozilla.org/en/docs/Web/HTML/Element/script#attr-crossorigin) loading of chunks. The following values are accepted...

`crossOriginLoading: false` - Disable cross-origin loading (default)

`crossOriginLoading: "anonymous"` - Enable cross-origin loading **without credentials**

`crossOriginLoading: "use-credentials"` - Enable cross-origin loading **with credentials**


## `output.devtoolFallbackModuleFilenameTemplate`

`string | function(info)`

A fallback used when the template string or function above yields duplicates.

See [`output.devtoolModuleFilenameTemplate`](#output-devtoolmodulefilenametemplate).


## `output.devtoolLineToLine`

`boolean | object`

> Avoid using this option as it is __deprecated__ and will soon be removed.

Enables line to line mapping for all or some modules. This produces a simple source map where each line of the generated source is mapped to the same line of the original source. This is a performance optimization and should only be used if all input lines match generated lines.

Pass a boolean to enable or disable this feature for all modules (defaults to `false`). An object with `test`, `include`, `exclude` is also allowed. For example, to enable this feature for all javascript files within a certain directory:

``` js
devtoolLineToLine: { test: /\.js$/, include: 'src/utilities' }
```


## `output.devtoolModuleFilenameTemplate`

`string | function(info)`

This option is only used when [`devtool`](/configuration/devtool) uses an options which requires module names.

Customize the names used in each source map's `sources` array. This can be done by passing a template string or function. For example, when using `devtool: 'eval'`, this is the default:

``` js
devtoolModuleFilenameTemplate: "webpack:///[resource-path]?[loaders]"
```

The following substitutions are available in template strings (via webpack's internal [`ModuleFilenameHelpers`](https://github.com/webpack/webpack/blob/master/lib/ModuleFilenameHelpers.js)):

| Template                 | Description |
| ------------------------ | ----------- |
| [absolute-resource-path] | The absolute filename |
| [all-loaders]            | Automatic and explicit loaders and params up to the name of the first loader |
| [hash]                   | The hash of the module identifier |
| [id]                     | The module identifier |
| [loaders]                | Explicit loaders and params up to the name of the first loader |
| [resource]               | The path used to resolve the file and any query params used on the first loader |
| [resource-path]          | The path used to resolve the file without any query params |

When using a function, the same options are available camel-cased via the `info` parameter:

``` js
devtoolModuleFilenameTemplate: info => {
  return `webpack:///${info.resourcePath}?${info.loaders}`
}
```

If multiple modules would result in the same name, [`output.devtoolFallbackModuleFilenameTemplate`](#output-devtoolfallbackmodulefilenametemplate) is used instead for these modules.


## `output.filename`

`string`

This option determines the name of each output bundle. The bundle is written to the directory specified by the [`output.path`](#output-path) option.

For a single [`entry`](/configuration/entry-context#entry) point, this can be a static name.

``` js
filename: "bundle.js"
```

However, when creating multiple bundles via more than one entry point, code splitting, or various plugins, you should use one of the following substitutions to give each bundle a unique name...

Using entry name:

``` js
filename: "[name].bundle.js"
```

Using internal chunk id:

``` js
filename: "[id].bundle.js"
```

Using the unique hash generated for every build:

``` js
filename: "[name].[hash].bundle.js"
```

Using hashes based on each chunks' content:

``` js
filename: "[chunkhash].bundle.js"
```

Make sure to read the [Caching guide](/guides/caching) for details. There are more steps involved than just setting this option.

Note this option is called filename but you are still allowed to use something like `"js/[name]/bundle.js"` to create a folder structure.

Note this options does not affect output files for on-demand-loaded chunks. For these files the [`output.chunkFilename`](#output-chunkfilename) option is used. It also doesn't affect files created by loaders. For these files see loader options.

The following substitutions are available in template strings (via webpack's internal [`TemplatedPathPlugin`](https://github.com/webpack/webpack/blob/master/lib/TemplatedPathPlugin.js)):

| Template    | Description                                                                         |
| ----------- | ----------------------------------------------------------------------------------- |
| [hash]      | The hash of the module identifier                                                   |
| [chunkhash] | The hash of the chunk content                                                       |
| [name]      | The module name                                                                     |
| [id]        | The module identifier                                                               |
| [query]     | The module query, i.e., the string following `?` in the filename                    |

The lengths of `[hash]` and `[chunkhash]` can be specified using `[hash:16]` (defaults to 20). Alternatively, specify [`output.hashDigestLength`](#output-hashdigestlength) to configure the length globally.

T> When using the [`ExtractTextWebpackPlugin`](/plugins/extract-text-webpack-plugin), use `[contenthash]` to obtain a hash of the extracted file (neither `[hash]` nor `[chunkhash]` work).


## `output.hashDigest`

The encoding to use when generating the hash, defaults to `'hex'`. All encodings from Node.JS' [`hash.digest`](https://nodejs.org/api/crypto.html#crypto_hash_digest_encoding) are supported.


## `output.hashDigestLength`

The prefix length of the hash digest to use, defaults to `20`.


## `output.hashFunction`

The hashing algorithm to use, defaults to `'md5'`. All functions from Node.JS' [`crypto.createHash`](https://nodejs.org/api/crypto.html#crypto_crypto_createhash_algorithm) are supported.


## `output.hashSalt`

An optional salt to update the hash via Node.JS' [`hash.update`](https://nodejs.org/api/crypto.html#crypto_hash_update_data_input_encoding).


## `output.hotUpdateChunkFilename`

`string`

Customize the filenames of hot update chunks. See [`output.filename`](#output-filename) option for details on the possible values.

The only placeholders allowed here are `[id]` and `[hash]`, the default being:

``` js
hotUpdateChunkFilename: "[id].[hash].hot-update.js"
```

Here is no need to change it.


## `output.hotUpdateFunction`

`function`

Only used when [`target`](/configuration/target) is web, which uses JSONP for loading hot updates.

A JSONP function used to asynchronously load hot-update chunks.

For details see [`output.jsonpFunction`](#output-jsonpfunction).


## `output.hotUpdateMainFilename`

`string`

Customize the main hot update filename. See [`output.filename`](#output-filename) option for details on the possible values.

`[hash]` is the only available placeholder, the default being:

``` js
hotUpdateMainFilename: "[hash].hot-update.json"
```

Here is no need to change it.


## `output.jsonpFunction`

`string`

Only used when [`target`](/configuration/target) is web, which uses JSONP for loading on-demand chunks.

A JSONP function name used to asynchronously load chunks or join multiple initial chunks (CommonsChunkPlugin, AggressiveSplittingPlugin).

This needs to be changed if multiple webpack runtimes (from different compilation) are used on the same webpage.

If using the [`output.library`](#output-library) option, the library name is automatically appended.


## `output.library`

`string`

`string` or `object` (since webpack 3.1.0; for `libraryTarget: "umd"`)

Use `library`, and `libraryTarget` below, when writing a JavaScript library that should export values, which can be used by other code depending on it. Pass a string with the name of the library:

``` js
library: "MyLibrary"
```

The name is used depending on the value of the [`output.libraryTarget`](#output-librarytarget) options.

Note that `output.libraryTarget` defaults to `var`. This means if only `output.library` is used it is exported as variable declaration (when used as script tag it's available in the global scope after execution).

T> Read the [authoring libraries guide](/guides/author-libraries) guide for more information on `output.library` as well as `output.libraryTarget`.


## `output.libraryExport`

`string` or `string[]` (since webpack 3.0.0)

> Default: `_entry_return_`

Configure which module or modules will be exposed via the `libraryTarget`.

The default value `_entry_return_` is the namespace or default module returned by your entry file.

The examples below demonstrate the effect of this config when using `libraryTarget: "var"`, but any target may be used.

The following configurations are supported:

`libraryExport: "default"` - The **default export of your entry point** will be assigned to the library target:

```javascript
// if your entry has a default export of `MyDefaultModule`
var MyDefaultModule = _entry_return_.default;

// your users will use your library like:
MyDefaultModule.doSomething();
```

`libraryExport: "MyModule"` - The **specified module** will be assigned to the library target:

```javascript
// if your entry exports a module `MyModule`
var MyModule = _entry_return_.MyModule;

// your users will use your library like:
MyModule.doSomething();
```

`libraryExport: ["MyModule", "MySubModule"]` - The array is interpreted as a **path to a module** to be assigned to the library target:

```javascript
// if your entry exports `MyModule` which in turn exports `MySubModule`
var MySubModule = _entry_return_.MyModule.MySubModule;

MySubModule.doSomething();
```


## `output.libraryTarget`

`string`

> Default: `"var"`

Configure how the library will be exposed. Any one of the following options can be used.

The following options are supported:

`libraryTarget: "var"` - (default) When your library is loaded, the **return value of your entry point** will be assigned to a variable:

```javascript
var MyLibrary = _entry_return_;

// your users will use your library like:
MyLibrary.doSomething();
```

W> Not specifying a `output.library` will cancel this `"var"` configuration.


`libraryTarget: "this"` - When your library is loaded, the **return value of your entry point** will be assigned to this, the meaning of `this` is up to you:

```javascript
this["MyLibrary"] = _entry_return_;

// your users will use your library like:
this.MyLibrary.doSomething();
MyLibrary.doSomething(); //if this is window
```


`libraryTarget: "window"` - When your library is loaded, **the return value of your entry point** will be part `window` object.

 ```javascript
 window["MyLibrary"] = _entry_return_;

//your users will use your library like:
window.MyLibrary.doSomething();
 ```


`libraryTarget: "global"` - When your library is loaded, **the return value of your entry point** will be part `global` object.

 ```javascript
 global["MyLibrary"] = _entry_return_;

//your users will use your library like:
global.MyLibrary.doSomething();
 ```


`libraryTarget: "commonjs"` - When your library is loaded, **the return value of your entry point** will be part of the exports object. As the name implies, this is used in CommonJS environments:

```javascript
exports["MyLibrary"] = _entry_return_;

//your users will use your library like:
require("MyLibrary").doSomething();
```


`libraryTarget: "commonjs2"` - When your library is loaded, **the return value of your entry point** will be part of the exports object. As the name implies, this is used in CommonJS environments:

```javascript
module.exports = _entry_return_;

//your users will use your library like:
require("MyLibrary").doSomething();
```

T> Wondering the difference between CommonJS and CommonJS2? Check [this](https://github.com/webpack/webpack/issues/1114) out (they are pretty much the same).


`libraryTarget: "amd"` - In this case webpack will make your library an AMD module.

Note that your entry chunk must be defined with the `define` property, if not, webpack will create the AMD module, but without dependencies. The output will be something like this:

```javascript
define([], function() {
  // This module returns is what your entry chunk returns
});
```

If you download this script, you may get an error: `define is not defined`, it’s ok! If you are distributing your library with AMD, then your users need to use RequireJS to load it. Once you have RequireJS loaded, you can load your library.

So, with the following configuration...

```javascript
output: {
  library: "MyLibrary",
  libraryTarget: "amd"
}
```

users will be able to call your library like so:

```javascript
require(['MyLibrary'], function(MyLibrary) {
  // Do something with the library...
});
```

`libraryTarget: "umd"` - This is a way for your library to work with all the module definitions (and where aren't modules at all). It will work with CommonJS, AMD and as global variable. Take a look at the [UMD Repository](https://github.com/umdjs/umd) to learn more.

In this case, you need the `library` property to name your module:

```javascript
output: {
  library: "MyLibrary",
  libraryTarget: "umd"
}
```

And finally the output is:

```javascript
(function webpackUniversalModuleDefinition(root, factory) {
  if(typeof exports === 'object' && typeof module === 'object')
    module.exports = factory();
  else if(typeof define === 'function' && define.amd)
    define([], factory);
  else if(typeof exports === 'object')
    exports["MyLibrary"] = factory();
  else
    root["MyLibrary"] = factory();
})(this, function() {
  //what this module returns is what your entry chunk returns
});
```

Since webpack 3.1.0, you may specify an object for `library` for differing names per targets:

```javascript
output: {
  library: {
    root: "MyLibrary",
    amd: "my-library",
    commonjs: "my-common-library"
  },
  libraryTarget: "umd"
}
```

Module proof library.


`libraryTarget: "assign"` - Here webpack will blindly generate an implied global.

``` javascript
MyLibrary = _entry_return_;
```

Be aware that if `MyLibrary` isn't defined earlier your library will be set in global scope.

`libraryTarget: "jsonp"` - This will wrap the return value of your entry point into a jsonp wrapper.

``` javascript
MyLibrary(_entry_return_);
```

The dependencies for your library will be defined by the [`externals`](/configuration/externals/) config.


## `output.path`

`string`

The output directory as an **absolute** path.

```js
path: path.resolve(__dirname, 'dist/assets')
```

Note that `[hash]` in this parameter will be replaced with an hash of the compilation. See the [Caching guide](/guides/caching) for details.


## `output.pathinfo`

`boolean`

Tell webpack to include comments in bundles with information about the contained modules. This option defaults to `false` and **should not** be used in production, but it's very useful in development when reading the generated code.

``` js
pathinfo: true
```

Note it also adds some info about tree shaking to the generated bundle.


## `output.publicPath`

`string`

This is an important option when using on-demand-loading or loading external resources like images, files, etc. If an incorrect value is specified you'll receive 404 errors while loading these resources.

This option specifies the **public URL** of the output directory when referenced in a browser. A relative URL is resolved relative to the HTML page (or `<base>` tag). Server-relative URLs, protocol-relative URLs or absolute URLs are also possible and sometimes required, i. e. when hosting assets on a CDN.

The value of the option is prefixed to every URL created by the runtime or loaders. Because of this **the value of this option ends with `/`** in most cases.

The default value is an empty string `""`.

Simple rule: The URL of your [`output.path`](#output-path) from the view of the HTML page.

```js
path: path.resolve(__dirname, "public/assets"),
publicPath: "https://cdn.example.com/assets/"
```

For this configuration:

```js
publicPath: "/assets/",
chunkFilename: "[id].chunk.js"
```

A request to a chunk will look like `/assets/4.chunk.js`.

A loader outputting HTML might emit something like this:

```html
<link href="/assets/spinner.gif" />
```

or when loading an image in CSS:

```css
background-image: url(/assets/spinner.gif);
```

The webpack-dev-server also takes a hint from `publicPath`, using it to determine where to serve the output files from.

Note that `[hash]` in this parameter will be replaced with an hash of the compilation. See the [Caching guide](/guides/caching) for details.

Examples:

``` js
publicPath: "https://cdn.example.com/assets/", // CDN (always HTTPS)
publicPath: "//cdn.example.com/assets/", // CDN (same protocol)
publicPath: "/assets/", // server-relative
publicPath: "assets/", // relative to HTML page
publicPath: "../assets/", // relative to HTML page
publicPath: "", // relative to HTML page (same directory)
```

In cases where the `publicPath` of output files can't be known at compile time, it can be left blank and set dynamically at runtime in the entry file using the [free variable](http://stackoverflow.com/questions/12934929/what-are-free-variables) `__webpack_public_path__`.

```javascript
 __webpack_public_path__ = myRuntimePublicPath

// rest of your application entry
```

See [this discussion](https://github.com/webpack/webpack/issues/2776#issuecomment-233208623) for more information on `__webpack_public_path__`.


## `output.sourceMapFilename`

`string`

This option is only used when [`devtool`](/configuration/devtool) uses a SourceMap option which writes an output file.

Configure how source maps are named. By default `"[file].map"` is used.

The `[name]`, `[id]`, `[hash]` and `[chunkhash]` substitutions from [#output-filename](#output-filename) can be used. In addition to those, you can use substitutions listed below. The `[file]` placeholder is replaced with the filename of the original file. We recommend __only using the `[file]` placeholder__, as the other placeholders won't work when generating SourceMaps for non-chunk files.

| Template                   | Description                                                                         |
| -------------------------- | ----------------------------------------------------------------------------------- |
| [file]                     | The module filename                                                                 |
| [filebase]                 | The module [basename](https://nodejs.org/api/path.html#path_path_basename_path_ext) |


## `output.sourcePrefix`

`string`

Change the prefix for each line in the output bundles.

``` js
sourcePrefix: "\t"
```

Note by default an empty string is used. Using some kind of indentation makes bundles look more pretty, but will cause issues with multi-line strings.

There is no need to change it.


## `output.strictModuleExceptionHandling`

`boolean`

Tell webpack to remove a module from the module instance cache (`require.cache`) if it throws an exception when it is `require`d.

It defaults to `false` for performance reasons.

When set to `false`, the module is not removed from cache, which results in the exception getting thrown only on the first `require` call (making it incompatible with node.js).

For instance, consider `module.js`:

``` js
throw new Error("error");
```

With `strictModuleExceptionHandling` set to `false`, only the first `require` throws an exception:

``` js
// with strictModuleExceptionHandling = false
require("module") // <- throws
require("module") // <- doesn't throw
```

Instead, with `strictModuleExceptionHandling` set to `true`, all `require`s of this module throw an exception:

``` js
// with strictModuleExceptionHandling = true
require("module") // <- throws
require("module") // <- also throw
```


## `output.umdNamedDefine`

`boolean`

When using `libraryTarget: "umd"`, setting:

``` js
umdNamedDefine: true
```

will name the AMD module of the UMD build. Otherwise an anonymous `define` is used.
