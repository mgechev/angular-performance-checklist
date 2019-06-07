# Angular Performance Checklist

<img src="./assets/flash.png" width="1000">

## Вступление

В этой статье описаны полезные практики, которые помогут вам улучшить производительность ваших приложений на Angular. "Angular Performance Checklist" покрывает множество вопросов — от server-side pre-rendering и сборки приложений, до производительности в runtime и оптимизации change detection, который выполняется Angular.

Эта статья разделена на два основных блока:

- Network performance содержит в себе список практик, следуя которым, вы ускорите загрузку ваших приложений. Он также включает в себя способы оптимизации задержек и повышает эффективность в условиях медленого интернета.
- Runtime performance - содержит в себе практики, которые улучшат производительность ваших приложений в runtime. Они включают в себя оптимизации change detection и rendering.

Некоторые методы оптимизаций могут находиться сразу в двух котегориях, поэтому может быть небольшое пересечение. Однако, в этом случае будут перечислены различия в вариантах использования, а также их последствия.

Большинство инструментов связаны со специфичными проблемами. Эти инструменты помогут вам улучшить качество разработки за счет автоматизации процесса.

Обратите внимание, что большинство практик применимы к HTTP/1.1 и HTTP/2. В практиках, где делаются исключения, будут пометки о том, для какой версии протокола они предназначены.

## Содержание

- [Angular Performance Checklist](#angular-performance-checklist)
  - [Вступление](#%D0%B2%D1%81%D1%82%D1%83%D0%BF%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5)
  - [Содержание](#%D1%81%D0%BE%D0%B4%D0%B5%D1%80%D0%B6%D0%B0%D0%BD%D0%B8%D0%B5)
  - [Network performance](#network-performance)
    - [Bundling](#bundling)
    - [Minification and dead code elimination](#minification-and-dead-code-elimination)
    - [Remove template whitespace](#remove-template-whitespace)
    - [Tree-shaking](#tree-shaking)
    - [Tree-shakeable providers](#tree-shakeable-providers)
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
- [Итоги](#%D0%B8%D1%82%D0%BE%D0%B3%D0%B8)
- [Contributing](#contributing)
- [License](#license)

## Network performance

Некоторые из инструментов в этом разделе все еще находятся в разработке и в будущем могут быть изменены. Команда разработчиков Angular занимается тем, чтобы максимально автоматизировать процесс сборки для наших приложений и сделать большинство вещей проще в использовании.

### Bundling

Bundling - это стандартная практика, направленная на уменьшение количества запросов браузером, которые он должен выполнить для загрузки приложения. По сути, в качестве входных параметров, bundler получает список модулей. Таким образом, браузер может загрузить все приложение, выполнив всего несколько запросов, вместо того, чтобы по отдельности запрашивать каждый модуль.

Скорее всего, по мере разработки вашего приложения, объединение всех модулей в один станет не эффективным. Поэтому рассмотрите Code Splitting, который можно сделать с помощью Webpack.

**Дополнительные http запросы не будут происходить в HTTP/2 из-за его функции [server push](https://http2.github.io/faq/#whats-the-benefit-of-server-push).**

**Инструменты**

Инструменты, которые позволяют эффективно упаковывать в модуль ваше приложение:

- [Webpack](https://webpack.js.org) - обеспечивает эффективное объединение кода выполняя [tree-shaking](#tree-shaking).
- [Webpack Code Splitting](https://webpack.js.org/guides/code-splitting/) - технология для разделения вашего кода.
- [Webpack & http2](https://medium.com/webpack/webpack-http-2-7083ec3f3ce6#.46idrz8kb) - требуется для разделения кода в HTTP/2.
- [Rollup](https://github.com/rollup/rollup) - позволяет объединять код, применяя tree-shaking, и используя преимущество статичных импортов модулей ES2015.
- [Google Closure Compiler](https://github.com/google/closure-compiler) - выполняет множество оптимизаций и обеспечивает объединение кода. Изначально был написан на Java, но также имеет реализацию на JavaScript [JavaScript](https://www.npmjs.com/package/google-closure-compiler-js), которую можете [найти здесь](https://www.npmjs.com/package/google-closure-compiler-js).
- [SystemJS Builder](https://github.com/systemjs/builder) - обеспечивает сборку приложения в один файл с помощью SystemJS и имеет поддержку зависимостей с различными версиями.
- [Browserify](http://browserify.org/).
- [ngx-build-modern](https://github.com/manfredsteyer/ngx-build-plus/tree/master/ngx-build-modern) - плагин для Angular CLI, который может собирать приложение в двух версиях:
  1. Для современных браузеров с модулями ES2015 и основные полифиламы, что делает bundle меньше;
  2. Дополнительная легаси версия, использующая остальные полифилы и другой compiler target (по-умолчанию).

**Полезные материалы**

- ["Сборка Angular приложения для Production"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["Сборка Angular приложения в 2.5X меньше вместе с Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### Minification and dead code elimination

В случае медленного интернет соединения эти методы позволяют нам оптимизировать загрузку приложения за счет уменьшения его веса.

**Инструменты**

- [Uglify](https://github.com/mishoo/UglifyJS) - делает минификацию кода, a именно уменьшает размер переменных, удаляет комментарии и пробелы, а также мертвый код и т.д. Он написан полностью на JavaScript, и имеет плагины для всех популярных task runners.
- [Google Closure Compiler](https://github.com/google/closure-compiler) - работает аналогично uglify. В продвинутом режиме он принудительно преобразует AST вашего приложения, чтобы проводить еще более сложные оптимизации. Он так же имеет [JavaScript версию](https://www.npmjs.com/package/google-closure-compiler-js), которую можно [найти здесь](https://www.npmjs.com/package/google-closure-compiler-js). GCC имеет *почти полную поддержку модулей ES2015*, поэтому может [делать tree-shaking](#tree-shaking). 

**Полезные материалы**

- ["Сборка Angular приложения для Production"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["Сборка Angular приложения в 2.5X меньше вместе с Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### Remove template whitespace

Хотя мы и не видим символ пробела (соотвествующий регулярному выражению `\s`), он все еще представлен байтами, которые передаются по сети. Однако, если мы максимально уменьшим количество пустых значений в шаблонах, то мы сможем уменьшить размер итогового AoT-кода.

К счастью, нам не нужно делать это вручную. В интерфейсе `ComponentMetadata` есть свойство `preserveWhitespaces`. Так как удаление пробелов может повлиять на DOM, оно по умолчанию имеет значение `false`. В случае, если мы установим свойство в `true`, то Angular очистит код от ненужных пробелов, что приведет к дополнительному уменьшению размера модуля.

- [Об preserveWhitespaces в документации Angular](https://angular.io/api/core/Component#preserveWhitespaces)

### Tree-shaking

В собранной версии приложения обычно не нужен весь код, который есть в Angular, сторонних библиотеках, или даже тот, который мы сами написали. Поэтому благодаря тому, что при импорте модулей ES2015 явно указывается что именно импортируется, можно избавиться от кода, который не был использован в приложении.

**Пример**

```javascript
// foo.js
export foo = () => 'foo';
export bar = () => 'bar';

// app.js
import { foo } from './foo';
console.log(foo());
```
После tree-shaking и сборки `app.js` мы получим:

```javascript
let foo = () => 'foo';
console.log(foo());
```

Это значит, что не использованный экспорт `bar` не будет включен в bundle.

**Инструменты**
- [Webpack](https://webpack.js.org) - предоставляет эффективную сборку с использованием [tree-shaking](#tree-shaking). После сборки приложения не экспортируется код, который не был использован. Таким образом код может быть помечен как dead code и удален с помощью Uglify.
- [Rollup](https://github.com/rollup/rollup) - предоставляет сборку с использованием tree-shaking, за счет статических импортов модулей ES2015.
- [Google Closure Compiler](https://github.com/google/closure-compiler) - предлагает множество оптимизаций и предоставляет возможность сборки приложения. Изначально он был написан на Java, но с недавнего времени поддерживает и [версию для JavaScript](https://www.npmjs.com/package/google-closure-compiler-js).

*Обратите внимание:* GCC еще не поддерживает `export *`. Однако функция важна для сборки Angular приложений из-за широкого использования "barrel" файлов.

**Полезные материалы**

- ["Сборка Angular приложения для Production"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["Сборка Angular приложения в 2.5X меньше вместе с Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)
- ["Использование pipeable операторов в RxJS"](https://github.com/ReactiveX/rxjs/blob/master/doc/pipeable-operators.md)

### Tree-shakeable providers

Начиная с версии Angular 6, команда Angular представила новую фичу, которая позволяет делать tree-shakable сервисы. Это значит, что сервисы не будут включены в финальный бандл пока они не будут использованы другими сервисами или компонентами. Это помогает уменьшить размер итогового бандла за счет удаления неиспользуемого кода.

Используя аттрибут `providedIn` в декораторе `@Injectable()` можно определить место, где сервис должен быть инициализирован и сделать его tree-shakeable. После этого нужно удалить его из аттрибута `providers` в инициализации `NgModule`, а также из импортов в файле `NgModule`.

До:

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

После:

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

Если `MyService` не используется ни в одном компоненте/сервисе/директиве, то он не будет включен в итоговый bundle.

**Полезные материалы**

- [Angular Providers](https://angular.io/guide/providers)

### Ahead-of-Time (AoT) Compilation

Проблемой низкоуровневых инструментов (таких как GCC, Rollup и т.д.) является то, что они не анализируют HTML-подобные шаблоны Angular компонентов. Это делает менее эффективной поддержку tree-shaking, потому что они не знают, на какие директивы имеются ссылки в шаблонах. Компилятор AoT конвертирует HTML-подобные шаблоны в JavaScript или TypeScript с импортами ES2015 модулей. Таким образом, мы можем эффективно делать tree-shaking во время сборки и удалять все неиспользуемые директивы, которые могут быть определенны Angular'ом, сторонними библиотеками или нашим приложением.

**Полезные материалы**

- ["Ahead-of-Time Compilation в Angular"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)

### Compression

Сжатие ответов является стандартной практикой уменьшения используемого трафика для загрузки приложения. Указав заголовок `Accept-Encoding`, браузер говорит серверу, какие алгоритмы сжатия он поддерживает на клиентском компьютере. В свою очередь сервер в заголовке ответа устанавливает значение для `Content-Encoding`, чтобы сообщить браузеру, какой алгоритм сжатия был применен.

**Инструменты**

Инструменты, приведенные здесь, не являются специфичными для Angular, и полностью зависит от используемого веб сервера. И вот основные инструменты для сжатия:

- deflate - алгоритмы сжатия данных, связанных с конкретным форматом файла, который использует комбинацию алгоритма LZ77 и Код Хаффмана. 
- [brotli](https://github.com/google/brotli) - алгоритм сжатия общего назначения без потерь, который сжимает данные, используя комбинацию современного варианта алгоритма LZ77, Кода Хаффмана и моделирование контекста 2-го порядка, со степенью сжатия, сопостовимой с лучшими в настоящее время способами сжатия общего назначения. Это сравнимо по скорости с deflate, но имеет лучшее сжатие.

**Полезные материалы**

- ["Сжатие с использованием Brotli лучше, чем Gzip"](https://hacks.mozilla.org/2015/11/better-than-gzip-compression-with-brotli/)
- ["Сборка Angular приложения в 2.5X меньше вместе с Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### Pre-fetching Resources

Предзагрузка ресурсов это отличный способ улучшить user experience. Мы можем загружать заранее как ассеты (изображения, стили, модули предназначенные для [lazy load](#lazy-loading-of-resources) и т.д.), так и данные. Существуют различные стратегии предзагрузки, но в большинстве случаев их использование зависит от специфики вашего приложения.

### Lazy-Loading of Resources

Когда приложение обладает большой кодовой базой с сотней зависимостей, подходы, описанные выше, могут оказаться бесполезными с точки зрения снижения размеров бандла (до разумных показателей 100кб или 2мб, но это полностью зависит от бизнес целей).

В таком случае разумно подгружать модули частично, лениво. Например, допустим, разрабатываемое приложение - это площадка для электронной торговли. В таком случае мы бы хотели, чтобы панель администратора загружалась независимо от интерфейса пользователя. Если, например, администратор должен добавить новый продукт, мы бы хотели обеспечить загрузку только необходимого для этого модуля. Это могла бы быть просто страница с добавлением продукта или вся панель администратора, в зависимости от бизнес логики приложения.

**Инструменты**

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

Кэширование - это еще одна распространенная практика, направленная на ускорение работы нашего приложения за счет использования предположения о том, что если недавно был запрошен один ресурс, он может быть запрошен снова в ближайшем будущем.

Для кэширования данных мы обычно используем кастомные методы. Для кэширования статических ресурсов мы можем использовать стандартные механизмы в браузере или Service Workers с [CacheStorage API](https://developer.mozilla.org/en-US/docs/Web/API/Cache).

### Use Application Shell
Для того, чтобы быстрее отобразить пользователю часть страницы используйте [Application Shell](https://developers.google.com/web/updates/2015/11/app-shell).

Application Shell - это минимальный пользовательский интерфейс, который мы показываем пользователям, чтобы показать, что приложение будет доступно в ближайшее время. Для динамического создания оболочки приложения вы можете использовать Angular Universal с пользовательскими директивами, которые по условиям отображают элементы в зависимости от используемой платформы рендеринга (т.е. скрывают все, кроме оболочки приложения, при использовании `platform-server`).

**Инструменты**

- [Angular Service Worker](https://angular.io/guide/service-worker-intro) - стремится автоматизировать процесс настройки Service Workers. Он включает в себя Service Worker для кэширования статичных ресурсов и инструмент для [генерации application shell](https://developers.google.com/web/updates/2015/11/app-shell?hl=en).
- [Angular Universal](https://github.com/angular/angular/tree/master/packages/platform-server) - Universal (изоморфный) JavaScript для Angular.

**Полезные материалы**

- ["Мгновенная загрузка приложения с Application Shell Architecture"](https://developers.google.com/web/updates/2015/11/app-shell)

### Use Service Workers

Мы думаем о Service Worker, как о HTTP-прокси, который находится в браузере. Все запросы, которые отправляются клиентом, перехватываются Service Worker. Он может обработать их или передать дальше по сети.

**Инструменты**

- [Angular Service Worker](https://angular.io/guide/service-worker-intro) - направлен на автоматизацию процесса управления Service Worker. Он так же содержит Service Worker для кэширования статических ресурсов и [генерацию application shell](https://developers.google.com/web/updates/2015/11/app-shell?hl=en).
- [Offline Plugin для Webpack](https://github.com/NekR/offline-plugin) - Webpack плагин добавляющий поддержку Service Worker с fall-back для AppCache.

**Полезные материалы**

- ["The offline cookbook"](https://jakearchibald.com/2014/offline-cookbook/)

## Runtime Optimizations

В этом разделе приведены рекомендации, которые необходимы для обеспечания плавной работы UI со скоростью 60 кадров в секунду (fps).

### Use `enableProdMode`

В development режиме Angular вызывает дополнительные проверки изменений, чтобы убедиться, что change detection не приводит к каким-либо дополнительным изменениям. Таким образом, Angular гарантирует, что соблюден однонаправленный поток данных.

Чтобы отключить эти проверки для production, не забудьте вызвать `enableProdMode`:

```typescript
import { enableProdMode } from '@angular/core';

if (ENV === 'production') {
  enableProdMode();
}
```

### Ahead-of-Time Compilation

AoT может быть не только полезен для достижения более эффективной сборки приложения, путем применения tree-shaking, но также для повышения производительности наших приложений в runtime. Альтернативой AoT является компиляция Just-in-Time (JiT), который выполняется в runtime. Поэтому AoT позволяет уменьшить количество вычислений, необходимых для рендеринга нашего приложения, выполняя компиляцию во время сборки.

**Инструменты**

- [angular2-seed](https://github.com/mgechev/angular2-seed) - стартер с поддержкой AoT компиляции.
- [angular-cli](https://cli.angular.io) - использование `ng serve --prod`

**Полезные материалы**

- ["Ahead-of-Time Compilation в Angular"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)

### Web Workers

Проблема типичного одностраничного приложения (SPA) заключается в том, что код выполняется в одом потоке. Это означает, что если мы хотим добиться плавного UX с 60fps, то у нас есть **максимум 16мс** для выполнения вычислений между кадрами. В противном случае UI будет тормозить.

В сложном приложении с серьезным деревом компонентов, где change detection должно выполнять миллионы проверок ежесекундно, нетрудно потерять целые кадры. Благодаря абстрагированности платформы Angular, а именно тому, что она отделена от архитектуры DOM, можно запустить наше приложение (включая change detection) в Web Worker, оставив основной поток ответственным только за рендеринг UI.

**Инструменты**

- Модуль, который позволяет запускать приложение в Web Worker, поддерживается командой Angular. Примеры использования, можно [найти здесь](https://github.com/angular/angular/tree/master/modules/playground/src/web_workers).
- [Webpack Web Worker Loader](https://github.com/webpack/worker-loader) - загрузчик Web Worker для webpack.

**Полезные материалы**

- ["Использование Web Workers для большей отзывчивости приложений"](https://www.youtube.com/watch?v=Kz_zKXiNGSE)

### Server-Side Rendering

Большая проблема традиционных SPA заключается в том, что их содержимое не может быть отрисовано пока не загрузится весь JavaScript, потому что весь рендеринг происходит после. Отсюда мы имеем две большие проблемы:

- Не все поисковые сервисы запускают JavaScript, содержащийся в приложениях, поэтому они не могут получить содержимое динамических веб-страниц.
- Не самый лучший UX, так как пользователь не увидит ничего, кроме пустой/загрузочной страницы, пока весь JavaScript, содержащийся на странице, не загрузится, не распарсится и не выполнится.

Server-side rendering решает эту проблему пре-рендерингом запрашиваемой страницы на сервере и отправкой готового шаблона во время инициациализации приложения.

**Инструменты**

- [Angular Universal](https://github.com/angular/angular/tree/master/packages/platform-server) - Universal (изоморфная) JavaScript поддержка для Angular.
- [Preboot](https://github.com/angular/preboot) - Библиотека для управления переноса состояния страницы (т.е. events, focus, data), которые были сгенерированы на сервере, на страницу, отображаемую в браузере

**Полезные материалы**

- ["Angular Server Rendering"](https://www.youtube.com/watch?v=0wvZ7gakqV4)
- ["Angular Universal Patterns"](https://www.youtube.com/watch?v=TCj_oC3m6_U)

### Change Detection

При каждом асинхронном событии Angular вызывает change detection для всего дерева компонентов. Несмотря на то что код, который обнаруживает изменения, оптимизирован для [inline-caching](http://mrale.ph/blog/2012/06/03/explaining-js-vms-in-js-inline-caches.html), он все равно может затратным для больших и сложных приложений. Способ, который поможет улучшить производительность change detection, заключается в том, что change detection не должен выполняться для поддеревьев компонента, в которых не было изменений.

#### `ChangeDetectionStrategy.OnPush`

`ChangeDetectionStrategy.OnPush` позволяет нам отключить механизм change detection для дерева компонентов. Указав для change detection strategy в компоненте значение `ChangeDetectionStrategy.OnPush`, изменения будут срабатывать **только** тогда, когда компонент получил inputs, отличающиеся от предыдущих. Angular сравнивает предыдущие и текущие inputs по ссылке, и когда результат проверки равен `false`, то inputs помечаются как изменившиеся. В сочетании с [иммутабельными структурами данных](https://facebook.github.io/immutable-js/), `OnPush` улучшает производительность для "чистых" компонентов.

**Полезные материалы**

- ["Change Detection в Angular"](https://vsavkin.com/change-detection-in-angular-2-4f216b855d4c)
- ["Все что вам нужно знать о change detection в Angular"](https://blog.angularindepth.com/everything-you-need-to-know-about-change-detection-in-angular-8006c51d206f)

#### Detaching the Change Detector

Другой реализацией кастомного механизма отслеживания изменений является открепление и прикрепления отслеживания изменений (CD) для конкретного компонента. Как только мы открепляем CD, Angular не будет делать проверки для компонента и всей его низлежащей структуры.

Данная практика обычно используется, когда действия или взаимодействия пользователя со внешними сервисами запускают цикл отслеживания изменений чаще, чем это действительно необходимо. В таких ситуациях мы можем откреплять отслеживания измненений и прикреплять его обратно, когда нужно совершить проверку изменений.

#### Run outside Angular

В основе механизма отслеживания изменений в Angular лежит [zone.js](https://github.com/angular/zone.js). Zone.js патчит все асинхронные API в браузере и запускает отслеживание изменений в конце исполнения любой асинхронной функции. В **редких** случаях может быть необходимо исполнить код вне контекста Angular Zone и тогда механизм отслежвания изменений не будет вызван. В таких случаях мы можем использовать метод `runOutsideAngular` из `NgZone`.

**Пример**

В отрывке кода далее, вы можете увидеть пример компонента с использованием данной практики. Когда метод `_incrementPoints` вызван, компонент начнет инкрементировать свойство `_points` каждые 10 мс (по умолчанию). Инкрементация создаст иллюзию анимации. Т.к. в данной ситуации мы не хотим вызывать проверку изменений для всего древа компонентов каждые 10 секунд, мы можем вызвать `_incrementPoints` вне контекста Angular Zone и обновить DOM вручную (`points` сеттер метод).

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

**Обратите внимание**: Используйте эту практику **очень осторожно и только тогда, когда вы знаете, что делаете**, потому что при некорректном использовании это может привести к неустойчивому состоянию DOM. Также обратите внимание, что код выше не расчитан для запуска в WebWorkers. Если это необходимо, вы можете сделать его WebWorker совместимым, для этого нужно установить label's value используя Angular Renderer.

### Use pure pipes

Аргумент декоратора `@Pipe` принимает объекты в следующем формате:

```typescript
interface PipeMetadata {
  name: string;
  pure: boolean;
}
```

Свойство pure означает, что pipe не зависит от какого-либо глобального состояния и не производит сторонних эффектов. Т.е. возвращаемое значение всегда будет одинаковым для конкретного входного аргумента. Таким образом Angular может кэшировать выходы для всех входных аргументов, передаваемых в этот pipe, и переиспользовать их в дальнейшем для избежания повторных вычислений.

Значение по умолчанию свойства `pure` является `true`.


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

**Полезные материалы**

- ["NgFor directive"](https://angular.io/docs/ts/latest/api/common/index/NgFor-directive.html) - официальная документация для `*ngFor`
- ["Angular — улучшение производительности с помощью trackBy"](https://netbasal.com/angular-2-improve-performance-with-trackby-cc147b5104e5) - gif-демонстрация подходов
- [Component Dev Kit (CDK) Virtual Scrolling](https://material.angular.io/cdk/scrolling/overview) - описание API
- [ngx-virtual-scroller](https://github.com/rintoj/ngx-virtual-scroller) - отображает виртуальный, "бесконечный" список

### Optimize template expressions

Angular извлекает выражения в шаблонах после каждого срабатывания цикла change detection. Change detection срабатывает вследствие асинхронных вызовов, например, выполнение промисов, получение ответа http, нажатие клавиш и движение курсором мыши.

Такие выражения должны завершаться быстро, иначе пользователь может замечать "дергания", особенно на слабых девайсах. Поэтому,  если сталкиваетесь с затратными вычислениями, стоит подумать о кэшировании.

**Полезные материалы**
- [quick-execution](https://angular.io/guide/template-syntax#quick-execution) - официальная документация по выражениям в шаблонах
- [Increasing Performance - more than a pipe dream](https://youtu.be/I6ZvpdRM1eQ) - ng-conf видеозапись на youtube. Использование pipe вместо функции для интерполяции строки

# Итоги

Представленный список со временем будет постепенно развиваться добавлением и обновлением текущих практик. Если вы заметили, что чего-то не хватает, или считаете, что какие-то практики можно улучшить, то не стесняйтесь создавать issue и/или PR. Для более подробной информации об этом, пожалуйста, посмотрите раздел [Contributing](#contributing)", который находится ниже.

# Contributing

В случае если вы заметите недостающую, незавершенную или некорректную информацию, вы можете сделать pull request, это будет очень ценно для нас. Для обсуждения лучших практик, которые не включены в документацию, пожалуйста, [создайте issue на github](https://github.com/mgechev/angular2-performance-checklist/issues).

# License

MIT
