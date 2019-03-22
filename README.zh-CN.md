
# Angular 性能检查清单

<img src="./assets/flash.png" width="1000">

## 简介
这份清单涵盖了许多提高 Angular 应用性能的实践。本清单 涵盖了不同的主题，其中包括服务端渲染，预渲染和构建应用，还包括运行时的性能和变更检测。


这份清单分为两个大的部分：

- 网络性能 -  列举一些能够改善你的应用程序加载时间的最佳实践，包括网络延迟和低带宽的情况。
- 运行时性能 - 提高我们应用运行时性能的最佳实践，包括变更检测机制和渲染相关的优化。

其中，部分实践会影响涵盖这两个类别，因此可能会有些许的交集，但是，下文将明确指出其中的二者交集的含义和差异。

不仅如此，我们还例举出最佳实践相关的工具链，这些工具协助我们更好完成的自动化的开发，提高我们的生产效率。

提示：大多数实践对HTTP/1.1和HTTP/2都有效。通过指定可以应用到哪一个协议版本，每个实践都会指出碰到异常该如何处理。


## 目录

- [Angular 性能检查清单](#angular-2-performance-checklist)
  - [简介](#introduction)
  - [目录](#table-of-content)
  - [网络性能](#network-performance)
    - [打包](#bundling)
    - [代码去重和优化](#minification-and-dead-code-elimination)
    - [删除空模板](#remove-template-whitespace)
    - [摇树优化](#tree-shaking)
    - [摇树优化的供应商](#tree-shakeable-providers)
    - [Ahead-of-Time (AoT) 编译](#ahead-of-time-aot-compilation)
    - [压缩](#compression)
    - [预加载资源](#pre-fetching-resources)
    - [懒加载资源](#lazy-loading-of-resources)
    - [禁止懒加载默认的 module](#dont-lazy-load-the-default-route)
    - [缓存](#caching)
    - [使用命令](#use-application-shell)
    - [使用 Service Workers](#use-service-workers)
  - [运行时优化](#runtime-optimizations)
    - [使用 `enableProdMode`](#use-enableprodmode)
    - [Ahead-of-Time 编译](#ahead-of-time-compilation)
    - [Web Workers](#web-workers)
    - [服务端渲染](#server-side-rendering)
    - [变更检测](#change-detection)
      - [`ChangeDetectionStrategy.OnPush`(译者注: `变更检测的 onPush 策略`)](#changedetectionstrategyonpush)
      - [Detaching the Change Detector(译者注: `ChangeDetectorRef.detach()`)](#detaching-the-change-detector)
      - [Run outside Angular（译者注: `ngZone.runOutsizeAngular()`）](#run-outside-angular)
    - [使用纯管道(pure pipe)](#use-pure-pipes)
    - [`*ngFor` 指令](#ngfor-directive)
      - [使用 `trackBy` 选项](#use-trackby-option)
      - [最小化 DOM 节点](#minimize-dom-elements)
    - [优化模板语法](#optimize-template-expressions)
- [结语](#conclusion)
- [贡献](#contributing)

## 网络性能
本节中的一些工具目前还在开发阶段，可能会发生更改。Angular核心团队正在尽可能多地自动化我们的应用程序的构建过程，因此许多优化和新特性都将接踵而至。

### 构建
打包是一种标准实践，旨在减少浏览器对于资源的多次请求，降低 http 请求数。本质上，bundler 接收一个入口点列表作为输入，并生成一个或多个bundler。这样，浏览器只需执行几个请求，而不是单独请求每个资源，就可以获得整个应用程序。

当你的应用程序增长到一定的量级，把所有资源打包到一个 bundle 可能会适得其反，这时就需要使用 Webpack 对项目进行依赖拆分。


**由于具有服务器推送功能，HTTP/2不会涉及HTTP请求。参照 [server push](https://http2.github.io/faq/#whats-the-benefit-of-server-push) feature.**

**工具**

帮助我们更高效的构建应用程序:

- [Webpack](https://webpack.js.org) - 提供高效的构建，打包及[摇树优化](#tree-shaking).
- [Webpack Code Splitting](https://webpack.js.org/guides/code-splitting/) - 将代码分割.
- [Webpack & http2](https://medium.com/webpack/webpack-http-2-7083ec3f3ce6#.46idrz8kb) - 与 http2 分离.
- [Rollup](https://github.com/rollup/rollup) -利用ES2015模块的静态特性，通过执行有效的摇树来进行构建
- [Google Closure Compiler](https://github.com/google/closure-compiler) - 提供大量优化和构建支持. 最初使用 JAVA 进行编写的, 最近也有一个 [JavaScript 版本](https://www.npmjs.com/package/google-closure-compiler-js) 参见 [这里](https://www.npmjs.com/package/google-closure-compiler-js).
- [SystemJS Builder](https://github.com/systemjs/builder) - 为最小依赖系统模块树提供单文件构建
- [Browserify](http://browserify.org/) （译者注：brwoser 通过捆绑依赖，让你在浏览器具备 `require('modules') ` 的能力）
- [ngx-build-modern](https://github.com/manfredsteyer/ngx-build-plus/tree/master/ngx-build-modern) - Angular-CLI 的插件， 把应用构建成两个版本:
  1. 支持 ES2015 的现代浏览器和通过 pollyfill 实现新特性的浏览器，构建出较小的包
  2. 使用不同polyfill和编译器目标的旧版本(默认)

**资源**

- ["制作一个 Angular 应用程序"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["使用 Google Closure Compiler 让 Angular 程序缩小 2.5 倍"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### 优化并删除无用代码
这些实践都会让我们的应用程序更轻巧并减少带宽的负担。

**工具**

- [Uglify](https://github.com/mishoo/UglifyJS)  代码压缩，如管理变量、删除注释和空白、消除死码等，完全用 javascript编写，所有流行的代码运行程序（IDE）都有插件。
- [Google Closure Compiler](https://github.com/google/closure-compiler) -执行类似于uglify类型的代码压缩。在高级模式下，它会积极地转换程序的 AST ，以便能够执行更复杂的优化. 同样具有 [JavaScript 版本](https://www.npmjs.com/package/google-closure-compiler-js) 参见 [这里](https://www.npmjs.com/package/google-closure-compiler-js). GCC还支持大多数 ES2015 模块语法，因此它可以[摇树优化]（#摇树优化）。

**资源**

- ["Building an Angular Application for Production"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### 移除无用模板

虽然我们看不到空白字符（与`\s` RegExp 匹配的字符），但它仍然由通过网络传输的字节表示。如果我们将模板中的空白减少到最小，我们将能够分别进一步减少 AOT 代码的包大小。

好在，“componentmetadata” 接口提供属性 “preserveWhitespaces”，默认值为“false”，因为删除空白可能会影响 DOM 布局。如果我们将属性设置为“false”，那么 Angular 将修剪不必要的空白，从而进一步减小构建体积的大小。

- [preserveWhitespaces in the Angular docs](https://angular.io/api/core/Component#preserveWhitespaces)

### 摇树优化


对于我们的应用程序的最终版本，我们通常不使用 Angular或任何第三方库提供的整个 Library，甚至是我们自己写的所有代码。由于ES2015模块的静态特性，我们能够剔除应用程序中未引用的代码。

**示例**

```javascript
// foo.js
export foo = () => 'foo';
export bar = () => 'bar';

// app.js
import { foo } from './foo';
console.log(foo());
```
一旦我们对app.js进行了摇树优化，那么最终打包的结果为：

```javascript
let foo = () => 'foo';
console.log(foo());
```
无用的方法 `bar` 最终没有被打包

**工具**

- [Webpack](https://webpack.js.org) - 通过执行 [摇树优化](#摇树优化)， 一旦应用构建完成,  bundle 中不会包含无用代码，因此通过 uglify 可以安全地删除无用代码 
- [Rollup](https://github.com/rollup/rollup) - 利用ES2015模块的静态特性，通过执行有效的摇树来进行构建
- [Google Closure Compiler](https://github.com/google/closure-compiler) - 提供大量优化和构建支持. 最初使用 JAVA 进行编写的, 最近也有一个 JavaScript 版本 参见 这里.

*Note:*  GCC还支持大多数 ES2015 模块语法，因此它可以[摇树优化]（#摇树优化）。

**资源**

- [“制作一个 Angular 应用程序”](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- [“使用 Google Closure Compiler 让 Angular 程序缩小 2.5 倍”](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)
- ["使用 Rx.js 操作符"](https://github.com/ReactiveX/rxjs/blob/master/doc/pipeable-operators.md)

### 摇树优化服务
自从 Agnular 发布 V6 版本以来，Angular 团队提供了一个可以将 services 摇树优化的新特性，这意味着，除非其他服务或组件正在使用该服务，否则该的服务将不会包含在最终捆绑包中，这可以通过从包中删除未使用的代码来帮助减少包的大小。

你可以在 Angular 应用中的 Services 里使用 `@injectable()` 的 `providedIn` 方法去定义服务应该在哪里初始化，从而更好的摇树优化。
然后，您应该将其从 “ngModule” 声明的 “providers” 属性以及其 import 语句中删除，如下所示:

Before:

```ts
// app.module.ts
import { NgModule } from '@angular/core'
import { AppRoutingModule } from './app-routing.module'
import { AppComponent } from './app.component'
import { environment } from '../environments/environment'
import { MyService } from './app.service'

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    ...
  ],
  providers: [MyService],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

```ts
// my-service.service.ts 
import { Injectable } from '@angular/core'

@Injectable()
export class MyService { }
```

After:

```ts
// app.module.ts
import { NgModule } from '@angular/core'
import { AppRoutingModule } from './app-routing.module'
import { AppComponent } from './app.component'
import { environment } from '../environments/environment'

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    ...
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

```ts
// my-service.service.ts
import { Injectable } from '@angular/core'

@Injectable({
  providedIn: 'root'
})
export class MyService { }
```

如果没有在任何组件/服务中注入 `myService`，那么它将不会出现在最终的打包文件中。

**资源**

- [Angular 服务供应商](https://angular.io/guide/providers)

### Ahead-of-Time (AoT) 编译
对于可用的构建工具（如gcc、rollup等）来说，用内置的功能去解析 Angular 组件的 HTML-like 模板是一项艰巨的挑战。这使得它们的摇树效率降低，因为它们不确定模板中引用了哪些指令。AOT编译器通过 `ES2015` 模块导入将 Angular HTML 类模板传输到 JavaScript 或 TypeScript。这样，我们就能够在绑定期间有效地进行摇树，并删除由 Angular、或第三方库或是我们自己定义的所有未使用的指令。

**资源**

- ["Angular Ahead-of-Time 编译"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)

### 压缩
压缩响应是减少网络负载的标准实践。通过指定头“Accept-Encoding”的值，浏览器提示服务器哪些压缩算法在客户端上可用。另一方面，服务器在响应时，会在 “Content-Encoding” 中告诉浏览器选择了哪个算法来压缩。

**工具**
这些工具不是 Angular 特有的，完全依赖于我们的应用/服务。典型的压缩算法有：


- deflate - 一种数据压缩算法和相关文件格式，使用LZ77算法和哈夫曼编码的组合
- [brotli](https://github.com/google/brotli) -一种通用无损压缩算法，它使用LZ77算法的现代变种、哈夫曼编码和二阶上下文建模相结合来压缩数据，压缩比与当前可用的最佳通用压缩方法相当。它在速度上与 deflate 相似，但提供更密集的压缩。



**资源**

- ["Better than Gzip Compression with Brotli"](https://hacks.mozilla.org/2015/11/better-than-gzip-compression-with-brotli/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### 预加载资源

资源预加载是提高用户体验的好方法。我们可以预先获取资源（图像、样式、模块等）或数据。有不同的预加载策略，但大多数都取决于应用程序的具体情况。

### 懒加载资源
假设目标应用程序有一个庞大的代码库，并且有数百个依赖，上面列出的实践可能无法帮助我们将 bundle 包体积减少到一个合理的大小（合理的可能是 100k 或 2M，同样，完全取决于业务指标）。


在这种情况下，一个好的解决方案可能是延迟加载应用程序的一些模块。举个例子，假设我们正在构建一个电子商务系统。在这种情况下，我们可能希望独立于面向用户的 UI 加载 admin 面板。一旦管理员必须添加一个新产品，我们将希望提供所需的用户界面。这可能只是“添加产品页面”或整个 admin 面板，具体取决于我们的业务需求。


**工具**

- [Webpack](https://github.com/webpack/webpack) - 异步加载模块
- [ngx-quicklink](https://github.com/mgechev/ngx-quicklink) - 自动延迟加载屏幕上所有链接所引用的 modules


### 最好不要懒加载默认模块

我们假设有如下的路由配置:

```ts
// Bad practice
const routes: Routes = [
  { path: '', redirectTo: '/dashboard', pathMatch: 'full' },
  { path: 'dashboard',  loadChildren: './dashboard.module#DashboardModule' },
  { path: 'heroes', loadChildren: './heroes.module#HeroesModule' }
];
```
第一次用户打开的 url 为： https://example.com/ ，将自动重定向到 `/dashboard` ，与此同时将触发 `dashedboard.module `的懒加载。实际上 Angular 在渲染启动组件所在模块时，将下载 `dasheboard.module` 的所有文件和依赖项，接着，这些文件需要 JavaScript 引擎去解析。
在初始化页面中触发外部 HTTP

请求并执行大量的计算是一种不好的实践方式，这将大大降低初始路由页的加载速度。考虑将默认模块声明为非惰性加载。


### 缓存
缓存是另一种加快程序运行速度的实践方式：请求了一个资源，很有可能不久之后又去请求一次。

我们通常使用自定义的缓存机制，对于静态资源我们直接使用浏览器缓存或者 Service Worker ：[CacheStorage API](https://developer.mozilla.org/en-US/docs/Web/API/Cache).


### 使用应用程序指令
要使应用程序的性能更快，请使用[Application Shell](https://developers.google.com/web/updates/2015/11/app-shell).

使用应用程序指令是我们向用户展示的最小用户界面，以指示他们应用程序将很快交付。为了动态生成应用程序指令，我们可以将 Angular Universal 与自定义指令一起使用，这些指令根据所使用的渲染平台有条件地显示元素（例子：除了`platform-server` 之外都隐藏掉）。


**工具**

- [Angular Service Worker](https://angular.io/guide/service-worker-intro) - 自动化管理 Service Workers ，以及静态资源的缓存 [generating application shell](https://developers.google.com/web/updates/2015/11/app-shell?hl=en).
- [Angular Universal](https://github.com/angular/angular/tree/master/packages/platform-server) - 为 Angular 提供同构支持

**资源**

- ["Instant Loading Web Apps with an Application Shell Architecture"](https://developers.google.com/web/updates/2015/11/app-shell)

### 使用 Service Workers
我们可以认为 Service Worker 是 客户端本地 对 HTTP 的代理。所有请求从客户端发出第一时间都被 Service Worker 拦截，也可以直接通过 HTTP 去处理这些请求。


**工具**

- [Angular Service Worker](https://angular.io/guide/service-worker-intro) - 自动化管理 Service Workers ，以及静态资源的缓存 [generating application shell](https://developers.google.com/web/updates/2015/11/app-shell?hl=en).
- [Webpack 离线插件](https://github.com/NekR/offline-plugin) - Webpack Service 提供的 Worker插件

**资源**

- ["离线指南"](https://jakearchibald.com/2014/offline-cookbook/)

## 运行时优化
这个部分涵盖了一些浏览器帧率优化的最佳实践（最佳用户体验 60FPS / S）


### 使用 `enableProdMode`
在生产模式下，为了验证执行变更检测不会导致对任何绑定进行任何其他更改，Angular 会执行一些额外检查。这样，框架就可以确保遵循单向数据流。

要在正式模式下禁用这些变化，记得调用 `enableProdMode`：


```typescript
import { enableProdMode } from '@angular/core';

if (ENV === 'production') {
  enableProdMode();
}
```

### Ahead-of-Time（AoT） 编译

AoT 不仅有助于在摇树阶段更有效率的打包，而且会提升我们应用的运行时性能。AoT 的另一种选择是运行时执行的实时编译（JIT），因此，我们可以通过将编译作为构建过程的一部分执行来减少呈现应用程序所需的计算量。

**工具**

- [angular2-seed](https://github.com/mgechev/angular2-seed) - 一个包含 Aot 编译的 starter 项目 [angular-cli](https://cli.angular.io) 使用 `ng serve --prod`

**资源**

- ["Angular 的 Aot"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)

### Web Workers
在典型的单页应用中，我们的代码通常跑在一个线程里，也就是说如果我们想要用户体验到 60FPS，我们最多只有 16ms 的时间在各个帧之间去执行我们的代码。

在具备巨大组件树的应用中，变更检测通常每秒钟要执行上百万次，因此很容易让页面的性能下降，帧率变低。还好 Angular 将 Dom 结构进行了剥离，我们可以在 Web Woker中运行整个应用程序（包括更改检测），只让主 UI 线程只负责渲染。


**工具**

Webpack 允许我们在 Web Worker 中运行应用程序的模块。示例如何使用[found here](https://github.com/angular/angular/tree/master/modules/playground/src/web_workers).
- [Webpack Web Worker Loader](https://github.com/webpack/worker-loader) - A Web Worker Loader for webpack.

**资源**

- ["为更多 APP 使用 Web Workers"](https://www.youtube.com/watch?v=Kz_zKXiNGSE)

### 服务端渲染
SPA 有一个大问题：在初始渲染页面所需的整个JavaScript 加载之前，无法渲染。这导致了两个大问题：

-并非所有的搜索引擎都在运行与页面相关联的 JavaScript，因此它们无法正确爬到动态应用程序的内容。

-糟糕的用户体验，因为在下载、分析和执行与页面相关的 JavaScript 之前，用户只会看到页面处于空白/加载状态。

**工具**

- [Angular Universal](https://github.com/angular/angular/tree/master/packages/platform-server) - 为 Angular 同构提供支持.
- [Preboot](https://github.com/angular/preboot) -用于帮助管理状态（即事件、焦点、数据）从服务器生成的 Web 视图到客户端生成的 Web 视图的转换的库。

**资源**

- ["Angular 服务端渲染"](https://www.youtube.com/watch?v=0wvZ7gakqV4)
- ["Angular 通用模式"](https://www.youtube.com/watch?v=TCj_oC3m6_U)

### 变更检测

在每个异步事件上，Angular 对整个组件树执行更改检测。虽然检测更改的代码针对[内联缓存](http://mrale.ph/blog/2012/06/03/explaining-js-vms-in-js-inline-caches.html)进行了优化，但在复杂的应用程序中，这仍然是一个大量复杂的计算。提高变更检测性能的一种方法是不对子树执行变更检测，子树不应根据最近的操作进行变更。



#### `ChangeDetectionStrategy.OnPush`

 `OnPush` 禁用组件树子树的更改检测机制. 将单个组件的变更策略设定为 `ChangeDetectionStrategy.OnPush`, 将使更改检测仅在组件收到 **不同输入** 时执行。当 Angular 将输入与以前的参考输入进行比较时，它会将输入视为不同的输入，并且参考检查的结果是 `false`。结合[不可变数据结构]（https://facebook.github.io/immutable-js/）`onpush`可以为此类 **纯** 组件带来巨大的性能提升。

**资源**

- ["Angular 变更检测机制"](https://vsavkin.com/change-detection-in-angular-2-4f216b855d4c)
- ["Angular 中你应该知道的所有关于变更检测的原理"](https://blog.angularindepth.com/everything-you-need-to-know-about-change-detection-in-angular-8006c51d206f)

#### 关闭变更检测
另外的一种方法，通过 `detach`ing 和 `reattach`ing 实现自定义变更检测机制，从而来控制组件的变更检测开关。一旦执行 `detach` ，Angular 变更检测时将不会对整个组件树进行检查。

当用户操作或与外部服务的交互比需要的更频繁地触发变更检测时，通常使用此实践。在这种情况下，我们可能需要考虑分离变更检测器，并仅在需要执行变更检测时通过重新 `reattach` 来打开变更检测的**开关**。




#### 运行于 Angular 之外
Angular 的变更检测机制是通过[zone.js](https://github.com/angular/zone.js) 实现的。
Zone.js 通过猴子补丁的方法代理了所有浏览器的异步 APIs，并在他们执行的时候以异步回调的形式触发变更检测。
在**极少情况**下，我们希望代码在 Angular 之外的上下文运行，所以不需要变更检测机制进行检查。这种情况下我们可以在组件中注入 `zone: NgZone` 服务，并且调用这个实例的 `runOutsizeAngular` 方法即可。

**例子**
在下面的代码片段中，您可以看到使用此实践的组件的示例。当调用 `_incrementPoints` 方法时，组件将开始每 10 ms 递增一次 `_points` 属性（默认情况下）。递增会造成动画的假象。因为在这种情况下，我们不希望触发整个组件树的更改检测机制，所以每 10 ms，我们可以在 Angular 区域的上下文之外运行 `incrementpoints`，并手动更新 DOM（请参见 `points` setter 访问器）。


```ts
@Component({
  template: '<span #label></span>'
})
class PointAnimationComponent {

  @Input() duration = 1000;
  @Input() stepDuration = 10;
  @ViewChild('label') label: ElementRef;

  @Input() set points(val: number) {
    this._points = val;
    if (this.label) {
      this.label.nativeElement.innerText = this._pipe.transform(this.points, '1.0-0');
    }
  }
  get points() {
    return this._points;
  }

  private _incrementInterval: any;
  private _points: number = 0;

  constructor(private _zone: NgZone, private _pipe: DecimalPipe) {}

  ngOnChanges(changes: any) {
    const change = changes.points;
    if (!change) {
      return;
    }
    if (typeof change.previousValue !== 'number') {
      this.points = change.currentValue;
    } else {
      this.points = change.previousValue;
      this._ngZone.runOutsideAngular(() => {
        this._incrementPoints(change.currentValue);
      });
    }
  }

  private _incrementPoints(newVal: number) {
    const diff = newVal - this.points;
    const step = this.stepDuration * (diff / this.duration);
    const initialPoints = this.points;
    this._incrementInterval = setInterval(() => {
      let nextPoints = Math.ceil(initialPoints + diff);
      if (this.points >= nextPoints) {
        this.points = initialPoints + diff;
        clearInterval(this._incrementInterval);
      } else {
        this.points += step;
      }
    }, this.stepDuration);
  }
}
```

**警告**: **只有当您确定要做什么时，才能非常小心地使用这个实践**，因为如果使用不当，它可能导致 DOM 的状态不一致而引发错误。还要注意，上面的代码不会在 Web Worker 中运行。为了使它与 Web Worker 兼容，需要使用 Angular 的 `renderer` 设置标签的值。


### 使用纯管道
作为参数，`@pipe` decorator接受具有以下格式参数：

```typescript
interface PipeMetadata {
  name: string;
  pure: boolean;
}
```
`pure` 标志表示管道不依赖于任何全局状态，不会产生副作用。这意味着当使用相同的输入调用时，管道将返回相同的输出。通过这种方式，Angular 可以缓存管道调用时使用的所有输入参数的输出，并重用它们，以便不必在每次计算时重新计算它们。

默认值 `pure` 为 `true`

### `*ngFor` 指令

 `*ngFor` 用于渲染集合（译者注：拥有迭代器的对象）

#### 使用 `trackBy` 选项
默认情况下，`*ngFor` 通过参考对象引用的唯一值来确认对象(译者注：对象的引用)。
也就是说，当开发人员在更新项的内容时中断对对象的引用时，Angular 将其视为删除旧对象和添加新对象。这会破坏列表中的旧 DOM 节点并在其位置添加新的 DOM 节点。

开发者可以提供一个关于 Angular 如何识别对象唯一性的提示：自定义跟踪函数作为 `*ngfor` 指令的 `trackby` 选项。跟踪函数接受两个参数：`index` 和 `item`。Angular 使用跟踪函数返回的值来跟踪每个迭代的对象。使用函数生成的值作为唯一键（key）。


**示例**

```typescript
@Component({
  selector: 'yt-feed',
  template: `
  <h1>Your video feed</h1>
  <yt-player *ngFor="let video of feed; trackBy: trackById" [video]="video"></yt-player>
`
})
export class YtFeedComponent {
  feed = [
    {
      id: 3849, // note "id" field, we refer to it in "trackById" function
      title: "Angular in 60 minutes",
      url: "http://youtube.com/ng2-in-60-min",
      likes: "29345"
    },
    // ...
  ];

  trackById(index, item) {
    return item.id;
  }
}
```

#### 精简 DOM 元素
在向 UI 添加元素时，操作 DOM 元素是很昂贵的操作。主要工作通常是将元素插入 DOM 并应用样式。如果 `*ngfor` 呈现很多元素，浏览器（尤其是旧的浏览器）可能会运行变得缓慢，因为需要更多的时间来完成所有元素的呈现。不仅仅是使用 Angular 会有这样的现象。

减少渲染时间，可以参考：

- 虚拟滚动  [CDK](https://material.angular.io/cdk/scrolling/overview) 或 [ngx-virtual-scroller](https://github.com/rintoj/ngx-virtual-scroller)
- 减少在模板中 `*ngfor` 遍历的 DOM 元素的数量。通常不需要/未使用的 DOM 元素是由一次又一次地扩展模板引起的。重新考虑它的结构可能会有帮助。
- 尽可能的使用 [`ng-container`](https://angular.io/guide/structural-directives#ngcontainer) 

**资源**

- ["NgFor 指令"](https://angular.io/docs/ts/latest/api/common/index/NgFor-directive.html) -  `*ngFor` 官方文档
- ["Angular — Improve performance with trackBy"](https://netbasal.com/angular-2-improve-performance-with-trackby-cc147b5104e5) - 显示方法的GIF演示
- [Component Dev Kit (CDK) Virtual Scrolling](https://material.angular.io/cdk/scrolling/overview) - API 描述
- [ngx-virtual-scroller](https://github.com/rintoj/ngx-virtual-scroller) -显示虚拟“无限”列表

### 优化模板表达式
Angular 在每次变更检测周期之后才去计算模板表达式。变更检测通常由一些异步动作去触发，例如 `promise`, `http返回结果`，`定时器 / 计时器事件` ，`键盘和鼠标事件`等。

表达式应该很快完成，否则用户体验可能会被拖走，尤其是在速度较慢的设备上。当计算代价高昂时，应该考虑缓存值。

**资源**
- [快速计算](https://angular.io/guide/template-syntax#quick-execution) - 模板计算-官方文档
- [提高性能-不仅仅是一个白日梦](https://youtu.be/I6ZvpdRM1eQ) - ng-conf 相关视频. 插值表达式中用管道代替函数

# 结语

实践列表将随着新的/更新的实践而动态演变。如果您发现遗漏或认为可以改进任何实践，请 PR/或 ISSUE。有关更多信息，请查看下面的 "[Contributing](#contributing)部分。

# 加入构建
如果您发现缺少，不完整或不正确的内容，我们将十分乐于看到您的 PR。对于文件中未包含的实践的讨论，请
[open an issue](https://github.com/mgechev/angular2-performance-checklist/issues).

# 协议

MIT
