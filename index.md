# Angular 2 Performance Checklist

<img src="./assets/flash.jpg" width="200">

## Introduction

This document contains a list of practices which will help us boost the performance of our Angular 2 applications. "Angular 2 Performance Checklist" covers different topics - from server-side pre-rendering and bundling of our applications, to runtime performance and optimization of the change detection performed by the framework.

The document is divided into two main sections:

- Network performance - lists practices that are going to improve mostly the load time of our application. They include methods for latency and bandwidth reduction.
- Runtime performance - practices which improve the runtime performance of our application. They include mostly change detection and rendering related optimizations.

Some practices impact both categories so there could be a slight intersection, however, the differences in the use cases and the implications will be explicitly mentioned.

Most subsections list tools, related to the specific practice, that can make us more efficient by automating our development flow.

Note that most practices are valid for both HTTP/1.1 and HTTP/2. Practices which make an exception will be mentioned by specifying to which version of the protocol they could be applied.

## Table of Content

- [Angular 2 Performance Checklist](#angular-2-performance-checklist)
  - [Introduction](#introduction)
  - [Table of Content](#table-of-content)
  - [Network performance](#network-performance)
  - [Bundling](#bundling)
  - [Minification and Dead code elimination](#minification-and-dead-code-elimination)
    - [Tree-shaking](#tree-shaking)
  - [Ahead-of-Time (AoT) Compilation](#ahead-of-time-aot-compilation)
  - [Compression](#compression)
  - [Pre-fetching Resources](#pre-fetching-resources)
  - [Lazy-Loading of Resources](#lazy-loading-of-resources)
  - [Caching](#caching)
    - [Use Service Workers](#using-service-workers)
- [Runtime Optimizations](#runtime-optimizations)
  - [Use `enableProdMode`](#use-enableprodmode)
  - [Ahead-of-Time Compilation](#ahead-of-time-compilation)
  - [Web Workers](#web-workers)
  - [Server-Side Rendering](#server-side-rendering)
  - [Change Detection](#change-detection)
    - [`ChangeDetectionStrategy.OnPush`](#changedetectionstrategyonpush)
    - [Detaching the Change Detector](#detaching-the-change-detector)
  - [Use pure pipes](#use-pure-pipes)
- [Conclusion](#conclusion)
- [Contributing](#contributing)

## Network performance

Some of the tools in this section are still in development and are subject to change. The Angular core team is working on automating the build process for our applications as much as possible so a lot of things will happen transparently.

## Bundling

Bundling is a standard practice aiming to reduce the number of requests that the browser needs to perform in order to deliver the application requested by the user. In essence, the bundler receives as an input a list of entry points and produces one or more bundles. This way, the browser can get the entire application by performing only a few requests, instead of requesting each individual resource separately.

As your application grows bundling everything into a single large bundle would again be counter productive. Explore Code Splitting techniques using Webpack.

**Additional http requests will not be a concern with HTTP/2 because of the [server push](https://http2.github.io/faq/#whats-the-benefit-of-server-push) feature.**

**Tooling**

Tools which allows us to bundle our applications efficiently are:

- [Webpack](https://webpack.github.io/) - provides efficient bundling by performing [tree-shaking](#tree-shaking).
- [Webpack Code Splitting](https://webpack.github.io/docs/code-splitting.html) - Techniques to split your code.
- [Webpack & http2](https://medium.com/webpack/webpack-http-2-7083ec3f3ce6#.46idrz8kb) - Need for splitting with http2.
- [Rollup](https://github.com/rollup/rollup) - provides bundling by performing efficient tree-shaking, taking advantage of the static nature of the ES2015 modules.
- [Google Closure Compiler](https://github.com/google/closure-compiler) - performs plenty of optimizations and provides bundling support. Originally written in Java, since recently it also has a [JavaScript version](https://www.npmjs.com/package/google-closure-compiler-js) which can be [found here](https://www.npmjs.com/package/google-closure-compiler-js).
- [SystemJS Builder](https://github.com/systemjs/builder) - provides a single-file build for SystemJS of mixed-dependency module trees.
- [Browserify](http://browserify.org/).

**Resources**

- ["Building an Angular 2 Application for Production"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["2.5X Smaller Angular 2 Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

## Minification and dead code elimination

These practices allow us to minimize the bandwidth consumption by reducing the payload of our application.

**Tooling**

- [Uglify](https://github.com/mishoo/UglifyJS) - performs minification such as mangling variables, removal of comments & whitespace, dead code elimination, etc. Written completely in JavaScript, has plugins for all popular task runners.
- [Google Closure Compiler](https://github.com/google/closure-compiler) - performs similar to uglify type of minification. In advanced mode it transforms the AST of our program aggressively in order to be able to perform even more sophisticated optimizations. It has also a [JavaScript version](https://www.npmjs.com/package/google-closure-compiler-js) which can be [found here](https://www.npmjs.com/package/google-closure-compiler-js). GCC also supports *most of the ES2015 modules syntax* so it can [perform tree-shaking](#tree-shaking).

**Resources**

- ["Building an Angular 2 Application for Production"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["2.5X Smaller Angular 2 Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### Tree-shaking

For the final version of our applications we usually we don't use the entire code which is provided by Angular and/or any third-party library, even the one that we've written. Thanks to the static nature of the ES2015 modules, we're able to get rid of the code which is not referenced in our apps.

**Example**

```javascript
// foo.js
export foo = () => 'foo';
export bar = () => 'bar';

// app.js
import { foo } from './foo';
console.log(foo());
```
Once we tree-shake and bundle `app.js` we'll get:

```javascript
let foo = () => 'foo';
console.log(foo());
```

This means that the unused export `bar` will not be included into the final bundle.

**Tooling**

- [Webpack](https://webpack.github.io/) - provides efficient bundling by performing [tree-shaking](#tree-shaking). Once the application has been bundled, it does not export the unused code so it can be safely considered as dead code and removed by Uglify.
- [Rollup](https://github.com/rollup/rollup) - provides bundling by performing an efficient tree-shaking, taking advantage of the static nature of the ES2015 modules.
- [Google Closure Compiler](https://github.com/google/closure-compiler) - performs plenty of optimizations and provides bundling support. Originally written in Java, since recently it has also a [JavaScript version](https://www.npmjs.com/package/google-closure-compiler-js) which can be [found here](https://www.npmjs.com/package/google-closure-compiler-js).

*Node:* GCC does not support `export *` yet, which is essential for building Angular applications because of the heavy usage of the "barrel" pattern.

**Resources**

- ["Building an Angular 2 Application for Production"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["2.5X Smaller Angular 2 Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

## Ahead-of-Time (AoT) Compilation

A challenge for the available in the wild tools (such as GCC, Rollup, etc.) are the HTML-like templates of the Angular components, which cannot be analyzed with their capabilities. This makes their tree-shaking support less efficient because they're not sure which directives are referenced within the templates. The AoT compiler transpiles the Angular 2 HTML-like templates to JavaScript or TypeScript with ES2015 module imports. This way we are able to efficiently tree-shake during bundling and remove all the unused directives defined by Angular, third-party libraries or by ourselves.

**Tooling**

- [@angular/compiler-cli](https://github.com/angular/angular/tree/master/modules/%40angular/compiler-cli) - a drop-in replacement for [tsc](https://www.npmjs.com/package/typescript) which statically analyzes our application and emits TypeScript/JavaScript for the component's templates.

**Resources**

- ["Ahead-of-Time Compilation in Angular 2"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)

## Compression

Compression of the responses' payload is a standard practice for bandwidth usage reduction. By specifying the value of the header `Accept-Encoding`, the browser hints the server which compression algorithms are available on the client's machine. On the other hand, the server sets value for the `Content-Encoding` header of the response in order to tell the browser which algorithm has been chosen for compressing the response.

**Tooling**

The tooling here is not Angular-specific and entirely depends on the web/application server that we're using. Typical compression algorithms are:

- deflate - a data compression algorithm and associated file format that uses a combination of the LZ77 algorithm and Huffman coding.
- [brotli](https://github.com/google/brotli) - a generic-purpose lossless compression algorithm that compresses data using a combination of a modern variant of the LZ77 algorithm, Huffman coding and 2nd order context modeling, with a compression ratio comparable to the best currently available general-purpose compression methods. It is similar in speed with deflate but offers more dense compression.

*Note:* Brotli is [not widely supported yet](http://caniuse.com/#search=brotli).

**Resources**

- ["Better than Gzip Compression with Brotli"](https://hacks.mozilla.org/2015/11/better-than-gzip-compression-with-brotli/)
- ["2.5X Smaller Angular 2 Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

## Pre-fetching Resources

Resource pre-fetching is a great way to improve the user experience. We can either pre-fetch assets (images, styles, modules intended to be [loaded lazily](#lazy-loading-of-resources), etc.) or data. There are different pre-fetching strategies but most of them depend on specifics of the application.

## Lazy-Loading of Resources

In case the target application has a huge code base with hundreds of dependencies, the practices listed above may not help us reduce the bundle to a reasonable size (reasonable might be 100K or 2M, it again, completely depends on the business goals).

In such cases a good solution might be to load some of the application's modules lazily. For instance, lets suppose we're building an e-commerce system. In this case we might want to load the admin panel independently from the user-facing UI. Once the administrator has to add a new product we'd want to provide the UI required for that. This could be either only the "Add product page" or the entire admin panel, depending on our use case/business requirements.

**Tooling**

- [Webpack](https://github.com/webpack/webpack) - allows asynchronous module loading.

## Caching

Caching is another common practice intending to speed-up our application by taking advantage of the heuristic that if one resource was recently been requested, it might be requested again in near future.

For caching data we usually use a custom caching mechanism. For caching static assets we can either use the standard browser caching or Service Workers with the [CacheStorage API](https://developer.mozilla.org/en-US/docs/Web/API/Cache).

### Use Service Workers

We can think of the Service Worker as an HTTP proxy which is located in the browser. All requests sent from the client are first intercepted by the Service Worker which can either handle them or pass them through the network.

**Tooling**

- [Angular Mobile Toolkit](https://github.com/angular/mobile-toolkit) - aims to automate the process of managing Service Workers. It also contains Service Worker for caching static assets, and one for [generating application shell](https://developers.google.com/web/updates/2015/11/app-shell?hl=en).
- [Offline Plugin for Webpack](https://github.com/NekR/offline-plugin) - Webpack plugin that adds support for Servie Worker with a fall-back to AppCache.

**Resources**

- ["The offline cookbook"](https://jakearchibald.com/2014/offline-cookbook/)

# Runtime Optimizations

This section includes practices which can be applied in order to provide smoother user experience with 60 frames per second (fps).

## Use `enableProdMode`

In development mode Angular performs some extra checks in order to verify that performing change detection does not result to any additional changes to any of the bindings. This way the frameworks assures that the unidirectional data flow has been followed.

In order to disable these changes for production to not forget to invoke `enableProdMode`:

```typescript
import { enableProdMode } from '@angular/core';

if (ENV === 'production') {
  enableProdMode();
}
```

## Ahead-of-Time Compilation

AoT can be helpful not only for achieving more efficient bundling by performing tree-shaking, but also for improving the runtime performance of our applications. The alternative of AoT is Just-in-Time compilation (JiT) which is performed runtime, therefore we can reduce the amount of computations required for rendering of our application by performing the compilation as part of our build process.

**Tooling**

- [@angular/compiler-cli](https://github.com/angular/angular/tree/master/modules/%40angular/compiler-cli) - a drop-in replacement for [tsc](https://www.npmjs.com/package/typescript) which statically analyzes our application and emits TypeScript/JavaScript for the component's templates.
- [angular2-seed](https://github.com/mgechev/angular2-seed) - a starter project which includes support for AoT compilation.
- [angular-cli](https://cli.angular.io) Using the `ng serve --prod` 

**Resources**

- ["Ahead-of-Time Compilation in Angular 2"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)

## Web Workers

Usual problem in the typical single-page application (SPA) is that our code is usually run in a single thread. This means that if we want to achieve smooth user experience with 60fps we have **at most 16ms** for execution between the individual frames are being rendered, otherwise they'll drop by half.

In complex application with huge component tree, where the change detection needs to perform millions of check each second it will not be hard to start dropping frames. Thanks to the platform agnosticism of Angular and it being decoupled from DOM architecture it's possible to run our entire application (including change detection) in a Web Worker and leave the main UI thread responsible only for rendering.

**Tooling**

- The module which allows us to run our application in a Web Worker is supported by the core team. Examples how it can be used, can be [found here](https://github.com/angular/angular/tree/master/modules/playground/src/web_workers).
- [Webpack Web Worker Loader](https://github.com/webpack/worker-loader) - A Web Worker Loader for webpack.

**Resources**

- ["Using Web Workers for more responsive apps"](https://www.youtube.com/watch?v=Kz_zKXiNGSE)

## Server-Side Rendering

A big issue of the traditional SPA is that they cannot be rendered until the entire JavaScript required for their initial rendering is available. This leads to two big problems:

- Not all search engines are running the JavaScript associated to the page so they are not able to index the content of dynamic apps properly.
- Poor user experience, since the user will see nothing more than a blank/loading screen until the JavaScript associated with the page is downloaded, parsed and executed.

Server-side rendering solves this issue by pre-rendering the requested page on the server and providing the markup of the rendered page during the initial page load.

**Tooling**

- [Angular Universal](https://github.com/angular/universal) - Universal (isomorphic) JavaScript support for Angular 2.
- [Preboot](https://github.com/angular/preboot) - Library to help manage the transition of state (i.e. events, focus, data) from a server-generated web view to a client-generated web view.
**Resources**

- ["Angular 2 Server Rendering"](https://www.youtube.com/watch?v=0wvZ7gakqV4)
- ["Angular 2 Universal Patterns"](https://www.youtube.com/watch?v=TCj_oC3m6_U)

## Change Detection

On each asynchronous event Angular performs change detection over the entire component tree. Although the code which detects for changes is optimized for [inline-caching](http://mrale.ph/blog/2012/06/03/explaining-js-vms-in-js-inline-caches.html), this still can be a heavy computation in complex applications. A way to improve the performance of the change detection is to not perform it for subtrees which are not supposed to be changed based on the recent actions.

### `ChangeDetectionStrategy.OnPush`

The `OnPush` change detection strategy allows us to disable the change detection mechanism for subtrees of the component tree. By setting the change detection strategy to any component to the value `ChangeDetectionStrategy.OnPush`, will make the change detection perform **only** when the component have received different inputs. Angular will consider inputs as different when it compares them with the previous inputs by reference, and the result of the reference check is `false`. In combination with [immutable data structures](https://facebook.github.io/immutable-js/) `OnPush` can bring great performance implications for such "pure" components.

**Resources**

- ["Change Detection in Angular 2"](https://vsavkin.com/change-detection-in-angular-2-4f216b855d4c)

### Detaching the Change Detector

Another way of implementing a custom change detection mechanism is by `detach`ing and `reattach`ing the change detector (CD) for given component. Once we `detach` the CD Angular will not perform check for the entire component subtree.

This practice is typically used when user actions or interactions with an external services trigger the change detection more often than required. In such cases we may want to consider detaching the change detector and reattaching it only when performing change detection is required.

## Use pure pipes

As argument the `@Pipe` decorator accepts an object literal with the following format:

```typescript
interface PipeMetadata {
  name: string;
  pure: boolean;
}
```

The pure flag indicates that the pipe is not dependent on any global state and does not produce side-effects. This means that the pipe will return the same output when invoked with the same input. This way Angular can cache the outputs for all the input parameters the pipe has been invoked with, and reuse them in order to not have to recompute them on each evaluation.

The default value of the `pure` property is `true`.

# Conclusion

The list of practices will dynamically evolve over time with new/updated practices. In case you notice something missing or you think that any of the practices can be improved do not hesitate to fire an issue and/or a PR. For more information please take a look at the "[Contributing](#contributing)" section below.

# Contributing

In case you notice something missing, incomplete or incorrect, a pull request will be greatly appreciated. For discussion of practices which are not included in the document please [open an issue](https://github.com/mgechev/angular2-performance-checklist).

