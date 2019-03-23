# Angular Performance Checklist

<img src="./assets/flash.png" width="1000">

[中文版](./README.zh-CN.md)
## Introduction

This document contains a list of practices which will help us boost the performance of our Angular applications. "Angular Performance Checklist" covers different topics - from server-side pre-rendering and bundling of our applications, to runtime performance and optimization of the change detection performed by the framework.

The document is divided into two main sections:

- Network performance - lists practices that are going to improve mostly the load time of our application. They include methods for latency and bandwidth reduction.
- Runtime performance - practices which improve the runtime performance of our application. They include mostly change detection and rendering related optimizations.

Some practices impact both categories so there could be a slight intersection, however, the differences in the use cases and the implications will be explicitly mentioned.

Most subsections list tools, related to the specific practice, that can make us more efficient by automating our development flow.

Note that most practices are valid for both HTTP/1.1 and HTTP/2. Practices which make an exception will be mentioned by specifying to which version of the protocol they could be applied.

## Table of Content

- [Angular Performance Checklist](#angular-performance-checklist)
  - [Introduction](#introduction)
  - [Table of Content](#table-of-content)
  - [Network performance](#network-performance)
    - [Bundling](#bundling)
    - [Minification and dead code elimination](#minification-and-dead-code-elimination)
    - [Remove template whitespace](#remove-template-whitespace)
    - [Tree-shaking](#tree-shaking)
    - [Tree-Shakeable Providers](#tree-shakeable-providers)
    - [Ahead-of-Time (AoT) Compilation](#ahead-of-time-aot-compilation)
    - [Compression](#compression)
    - [Pre-fetching Resources](#pre-fetching-resources)
    - [Lazy-Loading of Resources](#lazy-loading-of-resources)
    - [Don't Lazy-Load the Default Route](#dont-lazy-load-the-default-route)
    - [Caching](#caching)
    - [Use Application Shell](#use-application-shell)
    - [Use Service Workers](#use-service-workers)
  - [Runtime Optimizations](#runtime-optimizations)
    - [Use `enableProdMode`](#use-enableprodmode)
    - [Ahead-of-Time Compilation](#ahead-of-time-compilation)
    - [Web Workers](#web-workers)
    - [Server-Side Rendering](#server-side-rendering)
    - [Change Detection](#change-detection)
      - [`ChangeDetectionStrategy.OnPush`](#changedetectionstrategyonpush)
      - [Detaching the Change Detector](#detaching-the-change-detector)
      - [Run outside Angular](#run-outside-angular)
    - [Use pure pipes](#use-pure-pipes)
    - [`*ngFor` directive](#ngfor-directive)
      - [Use `trackBy` option](#use-trackby-option)
      - [Minimize DOM elements](#minimize-dom-elements)
    - [Optimize template expressions](#optimize-template-expressions)
- [Conclusion](#conclusion)
- [Contributing](#contributing)
- [License](#license)

## Network performance

Some of the tools in this section are still in development and are subject to change. The Angular core team is working on automating the build process for our applications as much as possible so a lot of things will happen transparently.

### Bundling

Bundling is a standard practice aiming to reduce the number of requests that the browser needs to perform in order to deliver the application requested by the user. In essence, the bundler receives as an input a list of entry points and produces one or more bundles. This way, the browser can get the entire application by performing only a few requests, instead of requesting each individual resource separately.

As your application grows bundling everything into a single large bundle would again be counter productive. Explore Code Splitting techniques using Webpack.

**Additional http requests will not be a concern with HTTP/2 because of the [server push](https://http2.github.io/faq/#whats-the-benefit-of-server-push) feature.**

**Tooling**

Tools which allows us to bundle our applications efficiently are:

- [Webpack](https://webpack.js.org) - provides efficient bundling by performing [tree-shaking](#tree-shaking).
- [Webpack Code Splitting](https://webpack.js.org/guides/code-splitting/) - Techniques to split your code.
- [Webpack & http2](https://medium.com/webpack/webpack-http-2-7083ec3f3ce6#.46idrz8kb) - Need for splitting with http2.
- [Rollup](https://github.com/rollup/rollup) - provides bundling by performing efficient tree-shaking, taking advantage of the static nature of the ES2015 modules.
- [Google Closure Compiler](https://github.com/google/closure-compiler) - performs plenty of optimizations and provides bundling support. Originally written in Java, since recently it also has a [JavaScript version](https://www.npmjs.com/package/google-closure-compiler-js) which can be [found here](https://www.npmjs.com/package/google-closure-compiler-js).
- [SystemJS Builder](https://github.com/systemjs/builder) - provides a single-file build for SystemJS of mixed-dependency module trees.
- [Browserify](http://browserify.org/).
- [ngx-build-modern](https://github.com/manfredsteyer/ngx-build-plus/tree/master/ngx-build-modern) - plugin for Angular-CLI which builds the application bundle in two variants:
  1. For modern browsers with ES2015 modules and specific polyfills resulting in a smaller bundle.
  2. Additional legacy version using different polyfills and compiler target (as it is by default).

**Resources**

- ["Building an Angular Application for Production"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### Minification and dead code elimination

These practices allow us to minimize the bandwidth consumption by reducing the payload of our application.

**Tooling**

- [Uglify](https://github.com/mishoo/UglifyJS) - performs minification such as mangling variables, removal of comments & whitespace, dead code elimination, etc. Written completely in JavaScript, has plugins for all popular task runners.
- [Google Closure Compiler](https://github.com/google/closure-compiler) - performs similar to uglify type of minification. In advanced mode it transforms the AST of our program aggressively in order to be able to perform even more sophisticated optimizations. It has also a [JavaScript version](https://www.npmjs.com/package/google-closure-compiler-js) which can be [found here](https://www.npmjs.com/package/google-closure-compiler-js). GCC also supports *most of the ES2015 modules syntax* so it can [perform tree-shaking](#tree-shaking).

**Resources**

- ["Building an Angular Application for Production"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### Remove template whitespace

Although we don't see the whitespace character (a character matching the `\s` regex) it is still represented by bytes which are transfered over the network. If we reduce the whitespace from our templates to minimum we will be respectively able to drop the bundle size of the AoT code even further.

Thankfully, we don't have to do this manually. The `ComponentMetadata` interface provides the property `preserveWhitespaces` which by default has value `false`, because removing the whitespace always may influence the DOM layout. In case we set the property to `false` Angular will trim the unnecessary whitespace which will lead to further reduction of the bundle size.

- [preserveWhitespaces in the Angular docs](https://angular.io/api/core/Component#preserveWhitespaces)

### Tree-shaking

For the final version of our applications we usually don't use the entire code which is provided by Angular and/or any third-party library, even the one that we've written. Thanks to the static nature of the ES2015 modules, we're able to get rid of the code which is not referenced in our apps.

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

- [Webpack](https://webpack.js.org) - provides efficient bundling by performing [tree-shaking](#tree-shaking). Once the application has been bundled, it does not export the unused code so it can be safely considered as dead code and removed by Uglify.
- [Rollup](https://github.com/rollup/rollup) - provides bundling by performing an efficient tree-shaking, taking advantage of the static nature of the ES2015 modules.
- [Google Closure Compiler](https://github.com/google/closure-compiler) - performs plenty of optimizations and provides bundling support. Originally written in Java, since recently it has also a [JavaScript version](https://www.npmjs.com/package/google-closure-compiler-js) which can be [found here](https://www.npmjs.com/package/google-closure-compiler-js).

*Note:* GCC does not support `export *` yet, which is essential for building Angular applications because of the heavy usage of the "barrel" pattern.

**Resources**

- ["Building an Angular Application for Production"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)
- ["Using pipeable operators in RxJS"](https://github.com/ReactiveX/rxjs/blob/master/doc/pipeable-operators.md)

### Tree-Shakeable Providers

Since the release of Angular version 6, The angular team provided a new feature to allow services to be tree-shakeable, meaning that your services will not be included in the final bundle unless they're being used by other services or components. This can help reduce the bundle size by removing unused code from the bundle.

You can make your services tree-shakeable by using the `providedIn` attribute to define where the service should be initialized when using the `@Injectable()` decorator. Then you should remove it from the `providers` attribute of your `NgModule` declaration as well as its import statement as follows.

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

If `MyService` is not injected in any component/service, then it will not be included in the bundle.

**Resources**

- [Angular Providers](https://angular.io/guide/providers)

### Ahead-of-Time (AoT) Compilation

A challenge for the available in the wild tools (such as GCC, Rollup, etc.) are the HTML-like templates of the Angular components, which cannot be analyzed with their capabilities. This makes their tree-shaking support less efficient because they're not sure which directives are referenced within the templates. The AoT compiler transpiles the Angular HTML-like templates to JavaScript or TypeScript with ES2015 module imports. This way we are able to efficiently tree-shake during bundling and remove all the unused directives defined by Angular, third-party libraries or by ourselves.

**Resources**

- ["Ahead-of-Time Compilation in Angular"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)

### Compression

Compression of the responses' payload is a standard practice for bandwidth usage reduction. By specifying the value of the header `Accept-Encoding`, the browser hints the server which compression algorithms are available on the client's machine. On the other hand, the server sets value for the `Content-Encoding` header of the response in order to tell the browser which algorithm has been chosen for compressing the response.

**Tooling**

The tooling here is not Angular-specific and entirely depends on the web/application server that we're using. Typical compression algorithms are:

- deflate - a data compression algorithm and associated file format that uses a combination of the LZ77 algorithm and Huffman coding.
- [brotli](https://github.com/google/brotli) - a generic-purpose lossless compression algorithm that compresses data using a combination of a modern variant of the LZ77 algorithm, Huffman coding and 2nd order context modeling, with a compression ratio comparable to the best currently available general-purpose compression methods. It is similar in speed with deflate but offers more dense compression.

**Resources**

- ["Better than Gzip Compression with Brotli"](https://hacks.mozilla.org/2015/11/better-than-gzip-compression-with-brotli/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### Pre-fetching Resources

Resource pre-fetching is a great way to improve the user experience. We can either pre-fetch assets (images, styles, modules intended to be [loaded lazily](#lazy-loading-of-resources), etc.) or data. There are different pre-fetching strategies but most of them depend on specifics of the application.

### Lazy-Loading of Resources

Когда приложение обладает большой кодовой базой с сотней зависимостей, подходы, описанные выше, могут оказаться бесполезными с точки зрения снижения размеров бандла (до разумных показателей 100кб или 2мб, но это полностью зависит от бизнес целей).

В таком случае разумно подгружать модули частично, лениво. Например, допустим, разрабатываемое приложение - это площадка для электронной торговли. В таком случае мы бы хотели, чтобы панель администратора загружалась независимо от интерфейса пользователя. Если, например, администратор должен добавить новый продукт, мы бы хотели обеспечить загрузку только необходимого для этого модуля. Это могла бы быть просто страница с добавлением продукта или вся панель администратора, в зависимости от бизнес логики приложения.

**Tooling**

- [Webpack](https://github.com/webpack/webpack) - обеспечивает асинхронную загрузку модулей
- [ngx-quicklink](https://github.com/mgechev/ngx-quicklink) - стратегия предварительной загрузки роутера, которая обеспечивает автоматическую ленивую загрузку модулей, связанных со всеми видимыми ссылками на экране

### Don't Lazy-Load the Default Route

Предположим, имеется следующая конфигурация роутинга:

```ts
// Плохой пример
const routes: Routes = [
  { path: '', redirectTo: '/dashboard', pathMatch: 'full' },
  { path: 'dashboard',  loadChildren: './dashboard.module#DashboardModule' },
  { path: 'heroes', loadChildren: './heroes.module#HeroesModule' }
];
```

В первый раз пользователь открывает приложения, используя адрес: https://example.com/. После этого он будет перенаправлен на `/dashboard`, после чего будет произведена ленивая загрузка `DashboardModule`. 

Для того, чтобы Angular отобразил начальный компонент модуля, необходимо загрузить файл `dashboard.module` и все его зависимости. После этого файл должен быть проанализирован виртуальной машиной JavaScript и оценен.

Запуск дополнительных HTTP-запросов и выполнение ненужных вычислений во время начальной загрузки страницы является плохой практикой, поскольку она замедляет стартовый рендеринг страницы. Поэтому рассмотрите возможность объявления страницы по умолчанию в обход ленивой загрузки модулей.

### Caching

Caching is another common practice intending to speed-up our application by taking advantage of the heuristic that if one resource was recently been requested, it might be requested again in near future.

For caching data we usually use a custom caching mechanism. For caching static assets we can either use the standard browser caching or Service Workers with the [CacheStorage API](https://developer.mozilla.org/en-US/docs/Web/API/Cache).

### Use Application Shell

To make the perceived performance of your application faster, use an [Application Shell](https://developers.google.com/web/updates/2015/11/app-shell).

The application shell is the minimum user interface that we show to the users in order to indicate them that the application will be delivered soon. For generating an application shell dynamically you can use Angular Universal with custom directives which conditionally show elements depending on the used rendering platform (i.e. hide everything except the App Shell when using `platform-server`).

**Tooling**

- [Angular Service Worker](https://angular.io/guide/service-worker-intro) - aims to automate the process of managing Service Workers. It also contains Service Worker for caching static assets, and one for [generating application shell](https://developers.google.com/web/updates/2015/11/app-shell?hl=en).
- [Angular Universal](https://github.com/angular/angular/tree/master/packages/platform-server) - Universal (isomorphic) JavaScript support for Angular.

**Resources**

- ["Instant Loading Web Apps with an Application Shell Architecture"](https://developers.google.com/web/updates/2015/11/app-shell)

### Use Service Workers

We can think of the Service Worker as an HTTP proxy which is located in the browser. All requests sent from the client are first intercepted by the Service Worker which can either handle them or pass them through the network.

**Tooling**

- [Angular Service Worker](https://angular.io/guide/service-worker-intro) - aims to automate the process of managing Service Workers. It also contains Service Worker for caching static assets, and one for [generating application shell](https://developers.google.com/web/updates/2015/11/app-shell?hl=en).
- [Offline Plugin for Webpack](https://github.com/NekR/offline-plugin) - Webpack plugin that adds support for Service Worker with a fall-back to AppCache.

**Resources**

- ["The offline cookbook"](https://jakearchibald.com/2014/offline-cookbook/)

## Runtime Optimizations

This section includes practices which can be applied in order to provide smoother user experience with 60 frames per second (fps).

### Use `enableProdMode`

In development mode Angular performs some extra checks in order to verify that performing change detection does not result to any additional changes to any of the bindings. This way the frameworks assures that the unidirectional data flow has been followed.

In order to disable these changes for production do not forget to invoke `enableProdMode`:

```typescript
import { enableProdMode } from '@angular/core';

if (ENV === 'production') {
  enableProdMode();
}
```

### Ahead-of-Time Compilation

AoT can be helpful not only for achieving more efficient bundling by performing tree-shaking, but also for improving the runtime performance of our applications. The alternative of AoT is Just-in-Time compilation (JiT) which is performed runtime, therefore we can reduce the amount of computations required for rendering of our application by performing the compilation as part of our build process.

**Tooling**

- [angular2-seed](https://github.com/mgechev/angular2-seed) - a starter project which includes support for AoT compilation.
- [angular-cli](https://cli.angular.io) Using the `ng serve --prod`

**Resources**

- ["Ahead-of-Time Compilation in Angular"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)

### Web Workers

Проблема типичного одностраничного приложения (SPA) заключается в том, что код выполняется в одом потоке. Это означает, что если мы хотим добиться плавного UX с 60fps, то у нас есть **максимум 16мс** для выполнения вычислений между кадрами. В противном случае UI будет тормозить.

В сложном приложении с серьезным деревом компонентов, где change detection должно выполнять миллионы проверок ежесекундно, нетрудно потерять целые кадры. Благодаря абстрагированности платформы Angular, а именно тому, что она отделена от архитектуры DOM, можно запустить наше приложение (включая change detection) в Web Worker, оставив основной поток ответственным только за рендеринг UI.

**Tooling**

- Модуль, который позволяет запускать приложение в Web Worker, поддерживается командой Angular. Примеры использования, можно [найти здесь](https://github.com/angular/angular/tree/master/modules/playground/src/web_workers).
- [Webpack Web Worker Loader](https://github.com/webpack/worker-loader) - загрузчик Web Worker для webpack.

**Resources**

- ["Использование Web Workers для большей отзывчивости приложений"](https://www.youtube.com/watch?v=Kz_zKXiNGSE)

### Server-Side Rendering

Большая проблема традиционных SPA заключается в том, что их содержимое не может быть отрисовано пока не загрузится весь JavaScript, потому что весь рендеринг происходит после. Отсюда мы имеем две большие проблемы:

- Не все поисковые сервисы запускают JavaScript, содержащийся в приложениях, поэтому они не могут получить содержимое динамических веб-страниц.
- Не самый лучший UX, так как пользователь не увидит ничего, кроме пустой/загрузочной страницы, пока весь JavaScript, содержащийся на странице, не загрузится, не распарсится и не выполнится.

Server-side rendering решает эту проблему пре-рендерингом запрашиваемой страницы на сервере и отправкой готового шаблона во время инициациализации приложения.

**Tooling**

- [Angular Universal](https://github.com/angular/angular/tree/master/packages/platform-server) - Universal (изоморфная) JavaScript поддержка для Angular.
- [Preboot](https://github.com/angular/preboot) - Библиотека для управления переноса состояния страницы (т.е. events, focus, data), которые были сгенерированы на сервере, на страницу, отображаемую в браузере

**Resources**

- ["Angular Server Rendering"](https://www.youtube.com/watch?v=0wvZ7gakqV4)
- ["Angular Universal Patterns"](https://www.youtube.com/watch?v=TCj_oC3m6_U)

### Change Detection

При каждом асинхронном событии Angular вызывает change detection для всего дерева компонентов. Несмотря на то что код, который обнаруживает изменения, оптимизирован для [inline-caching](http://mrale.ph/blog/2012/06/03/explaining-js-vms-in-js-inline-caches.html), он все равно может затратным для больших и сложных приложений. Способ, который поможет улучшить производительность change detection, заключается в том, что change detection не должен выполняться для поддеревьев компонента, в которых не было изменений.

#### `ChangeDetectionStrategy.OnPush`

`ChangeDetectionStrategy.OnPush` позволяет нам отключить механизм change detection для дерева компонентов. Указав для change detection strategy в компоненте значение `ChangeDetectionStrategy.OnPush`, изменения будут срабатывать **только** тогда, когда компонент получил inputs, отличающиеся от предыдущих. Angular сравнивает предыдущие и текущие inputs по ссылке, и когда результат проверки равен `false`, то inputs помечаются как изменившиеся. В сочетании с [иммутабельными структурами данных](https://facebook.github.io/immutable-js/), `OnPush` улучшает производительность для "чистых" компонентов.

**Resources**

- ["Change Detection in Angular"](https://vsavkin.com/change-detection-in-angular-2-4f216b855d4c)
- ["Everything you need to know about change detection in Angular"](https://blog.angularindepth.com/everything-you-need-to-know-about-change-detection-in-angular-8006c51d206f)

#### Detaching the Change Detector

Another way of implementing a custom change detection mechanism is by `detach`ing and `reattach`ing the change detector (CD) for given component. Once we `detach` the CD Angular will not perform check for the entire component subtree.

This practice is typically used when user actions or interactions with an external services trigger the change detection more often than required. In such cases we may want to consider detaching the change detector and reattaching it only when performing change detection is required.

#### Run outside Angular

The Angular's change detection mechanism is being triggered thanks to [zone.js](https://github.com/angular/zone.js). Zone.js monkey patches all asynchronous APIs in the browser and triggers the change detection in the end of the execution of any async callback. In **rare cases** we may want given code to be executed outside the context of the Angular Zone and thus, without running change detection mechanism. In such cases we can use the method `runOutsideAngular` of the `NgZone` instance.

**Example**

In the snippet below, you can see an example for a component which uses this practice. When the `_incrementPoints` method is called the component will start incrementing the `_points` property every 10ms (by default). The incrementation will make the illusion of an animation. Since in this case we don't want to trigger the change detection mechanism for the entire component tree, every 10ms, we can run `_incrementPoints` outside the context of the Angular's zone and update the DOM manually (see the `points` setter).

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

**Warning**: Use this practice **very carefully only when you're sure what you are doing** because if not used properly it can lead to an inconsistent state of the DOM. Also note that the code above is not going to run in WebWorkers. In order to make it WebWorker-compatible, you need to set the label's value by using the Angular's renderer.

### Use pure pipes

As argument the `@Pipe` decorator accepts an object literal with the following format:

```typescript
interface PipeMetadata {
  name: string;
  pure: boolean;
}
```

The pure flag indicates that the pipe is not dependent on any global state and does not produce side-effects. This means that the pipe will return the same output when invoked with the same input. This way Angular can cache the outputs for all the input parameters the pipe has been invoked with, and reuse them in order to not have to recompute them on each evaluation.

The default value of the `pure` property is `true`.

### `*ngFor` directive

Директива `*ngFor` используется для отрисовки коллекции.

#### Use `trackBy` option

По умолчанию `*ngFor` сравнивает объекты по ссылке.

Это значит, что когда разработчик меняет ссылку на объект во время изменения содержимого элемента, Angular распознает это как удаление старого объекта и создание нового. Это способствует уничтожению старого DOM элемента из списка и добавлению нового на его место.

Разработчик может указать, как Angular будет идентифицировать уникальность объекта: кастомная индексирующая функция в виде параметра `trackBy` для директивы `*ngFor`. Данная функция принимает два параметра: `index` и `item`. Angular использует значение, возвращаемое функцией, для идентификации элементов. Очень часто используют ID определенного элемента в качестве уникального ключа.

**Пример**

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
      id: 3849, // обратите внимание на поле "id", мы ссылаемся на него в "trackById" функции
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

#### Minimize DOM elements

Рендеринг DOM элементов обычно является самой дорогой операцией, например, при добавлении элементов в UI. Основные затраты вызваны вставкой элемента в DOM и применением стилей. Если `*ngFor` рендерит множество элементов, браузер (особенно старый) может тормозить, поэтому ему может потребоваться больше времени, чтобы отрендерить все элементы. Но это не относится к оптимизациям в Angular.

Чтобы снизить количество времени на рендеринг, попробуйте следующее:
- Виртуальная прокрутка посредством [CDK](https://material.angular.io/cdk/scrolling/overview) или [ngx-virtual-scroller](https://github.com/rintoj/ngx-virtual-scroller)
- Уменьшение количества DOM элементов, отображаемых с помощью `*ngFor` в шаблоне. Обычно ненужные/неиспользуемые DOM элементы возникают в результате расширения шаблона. Переосмысление структуры, скорее всего, сделает шаблон более простым.
- Используйте [`ng-container`](https://angular.io/guide/structural-directives#ngcontainer), где это возможно

**Resources**

- ["NgFor directive"](https://angular.io/docs/ts/latest/api/common/index/NgFor-directive.html) - официальная документация для `*ngFor`
- ["Angular — Improve performance with trackBy"](https://netbasal.com/angular-2-improve-performance-with-trackby-cc147b5104e5) - gif-демонстрация подходов
- [Component Dev Kit (CDK) Virtual Scrolling](https://material.angular.io/cdk/scrolling/overview) - описание API
- [ngx-virtual-scroller](https://github.com/rintoj/ngx-virtual-scroller) - отображает виртуальный, "бесконечный" список

### Optimize template expressions

Angular извлекает выражения в шаблонах после каждого срабатывания цикла change detection. Change detection срабатывает вследствие асинхронных вызовов, например, выполнение промисов, получение ответа http, нажатие клавиш и движение курсором мыши.

Такие выражения должны завершаться быстро, иначе пользователь может замечать "дергания", особенно на слабых девайсах. Поэтому,  если сталкиваетесь с затратными вычислениями, стоит подумать о кэшировании.

**Полезные материалы**
- [quick-execution](https://angular.io/guide/template-syntax#quick-execution) - официальная документация по выражениям в шаблонах
- [Increasing Performance - more than a pipe dream](https://youtu.be/I6ZvpdRM1eQ) - ng-conf видеозапись на youtube. Использование pipe вместо функции для интерполяции строки

# Conclusion

The list of practices will dynamically evolve over time with new/updated practices. In case you notice something missing or you think that any of the practices can be improved do not hesitate to fire an issue and/or a PR. For more information please take a look at the "[Contributing](#contributing)" section below.

# Contributing

В случае если вы заметите недостающую, незавершенную или некорректную информацию, вы можете сделать pull request, это будет очень ценно для нас. Для обсуждения лучших практик, которые не включены в документацию, пожалуйста, [создайте issue на github](https://github.com/mgechev/angular2-performance-checklist/issues).

# License

MIT
