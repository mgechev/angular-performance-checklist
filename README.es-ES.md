# Angular Performance Checklist

<img src="./assets/flash.png" width="1000">

- [中文版](./README.zh-CN.md)
- [Русский](./README.ru-RU.md)
- [Português](./readme-pt-BR.md)
- [Español](./readme-es-ES.md)

## Introducción

Este documento contiene una lista de prácticas las cuales nos ayudarán a aumentar el rendimiento de nuestras aplicaciones Angular. "Angular performance Checklist" cubre diferentes temas - desde server-side pre-rendering y bundle de nuestras aplicaciones, hasta rendimiento en ejecución y optimización de la detección del cambio realizada por el framework.

El documento se divide en dos secciones principales:

- Rendimiento de la red - lista de prácticas que mejorarán principalmente el tiempo de carga de nuestra aplicación. Incluyen métodos de latencia y reducción de ancho de banda.
- Rendimiento en ejecución - Prácticas que mejoran el rendimiento en ejecución de nuestra aplicación. Incluyen principalmente optimizaciones en la detección del cambio y relacionadas con el renderizado.

Algunas prácticas impactan en ambas categorías por lo que podría haber un ligero punto de encuentro, sin embargo, la diferencia en los casos de uso y las implicaciones serán explícitamente mencionadas.

La mayoría de subsecciones enumeran herramientas, relacionadas a la práctica específica, que nos harán más eficiente la automatización de nuestro entorno de desarrollo.

Tenga en cuenta que la mayoría de prácticas son válidas para HTTP/1.1 y HTTP/2. Se mencionarán las prácticas que sean la excepción especificando a qué versión del protocolo podrían aplicarse.

## Índice

- [Angular Performance Checklist](#angular-2-performance-checklist)
  - [Introducción](#Introducción)
  - [Índice](#Índice)
  - [Rendimiento de red](#rendimiento-de-red)
    - [Bundling](#bundling)
    - [Minification and Dead code elimination](#minificación-y-eliminación-de-código-no-utilizado-Dead-code)
    - [Remove template whitespace](#eliminar-espacios-en-blanco-de-las-plantillas)
    - [Tree-shaking](#tree-shaking)
    - [Tree-shakeable providers](#tree-shakeable-providers)
    - [Ahead-of-Time (AoT) Compilation](#compilación-Ahead-of-Time-AoT)
    - [Compression](#compresión)
    - [Pre-fetching Resources](#precarga-de-recursos-Pre-fetching)
    - [Lazy-Loading of Resources](#carga-diferida-de-recursos-Lazy-load)
    - [Don't lazy-load default route](#no-cargar-de-forma-diferida-la-ruta-por-defecto)
    - [Caching](#caché)
    - [Use Application Shell](#shell-de-la-aplicación)
    - [Use Service Workers](#service-workers)
  - [Optimizaciones en ejecución](#optimizaciones-en-ejecución)
    - [Utilizar enableProdMode](#enableProdMode)
    - [Compilación Ahead-of-Time](#compilación-ahead-of-time)
    - [Web Workers](#web-workers)
    - [Server-Side Rendering](#server-side-rendering)
    - [Detección del cambio](#detección-del-cambio)
      - [ChangeDetectionStrategy.OnPush](#changedetectionstrategyonpush)
      - [Desacoplando el detector de cambios](#desacoplando-el-detector-de-cambios)
      - [Ejecución fuera de Angular](#ejecución-fuera-de-Angular)
    - [Pipes puros](#pipes-puros)
    - [Directiva *ngFor](#directiva-ngFor)
      - [Utilizar opción trackBy](#utilizar-opción-trackBy)
      - [Minimizar elementos del DOM](#minimizar-elementos-del-DOM)
    - [Optimizar expresiones en plantilla](#optimizar-expresiones-en-plantilla-Template-expressions)
- [Conclusión](#conclusión)
- [Contribuyendo](#contribuyendo)

## Rendimiento de red

Algunas de las herramientas en esta sección están aún en desarrollo y sujetas a cambios. El equipo de "Angular core" está trabajando en automatizar el proceso de compilación de nuestras aplicaciones tanto como sea posible para que muchas cosas ocurran de forma transparente.

### Bundling

Bundling o empaquetado es una práctica estándar que permite reducir el número de solicitudes que el navegador necesita para entregar la aplicación solicitada por el usuario. En esencia, el "bundler" recibe una lista de puntos de entrada y los junta en uno o más bundles. De esta manera, el navegador puede obtener la aplicación completa realizando solo unas pocas solicitudes, en vez de ir solicitando de forma separada cada fichero.

Así como crezca tu aplicación empaquetar todo en un único fichero puede ser contraproducente. Vea las técnicas de división de código usando Webpack.

**Las solicitudes http adicionales no serán una preocupación con HTTP/2 gracias a la característica [server push](https://http2.github.io/faq/#whats-the-benefit-of-server-push)**

**Herramientas**

Las herramientas que nos permiten empaquetar nuestras aplicaciones de forma eficiente son:

- [Webpack](https://webpack.js.org) - Ofrece un bundle eficiente mediante la realización de [tree-shaking](#tree-shaking).
- [Webpack Code Splitting](https://webpack.js.org/guides/code-splitting/) - Técnicas para dividir el código.
- [Webpack & http2](https://medium.com/webpack/webpack-http-2-7083ec3f3ce6#.46idrz8kb) - Necesario para dividir el código usando http2.
- [Rollup](https://github.com/rollup/rollup) - Ofrece un bundle eficiente haciendo uso de "tree-shaking", aprovechando la naturaleza estática de los módulos ES2015.
- [Google Closure Compiler](https://github.com/google/closure-compiler) - realiza un montón de optimizaciones y proporciona soporte para el bundle. Originalmente escrito en Java, desde hace poco también tiene una [version JavaScript](https://www.npmjs.com/package/google-closure-compiler) la cual puede encontrarse [aquí](https://www.npmjs.com/package/google-closure-compiler).
- [SystemJS Builder](https://github.com/systemjs/builder) - Proporciona la generación de un único archivo para módulos mixtos de inyección de dependencias de SystemJS.
- [Browserify](http://browserify.org/).
- [ngx-build-modern](https://github.com/manfredsteyer/ngx-build-plus/tree/master/ngx-build-modern) - plugin para Angular-CLI el cual genera paquetes de dos maneras:
  1. Para navegadores modernos con módulos ES2015 y polyfills específicos ofreciendo un paquete de menor tamaño.
  2. Añade ficheros polyfill para navegadores más antiguos (tal y como es por defecto).

**Recursos**

- ["Compilando una aplicación angular para producción"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["Aplicaciones 2.5 veces más pequeñas usando Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### Minificación y eliminación de código no utilizado (Dead code)

Estas prácticas nos permiten minimizar el consumo de ancho de banda reduciendo la carga (payload) de nuestra aplicación

**Herramientas**

- [Uglify](https://github.com/mishoo/UglifyJS) - Realiza la minificación como el nombre de las variables, elimina comentarios y espacios en blanco, elimina código no utilizado (dead code), etc. Escrito completamente en JavaScript, tiene varios plugins para todos los task runners más populares.
- [Google Closure Compiler](https://github.com/google/closure-compiler) - Realiza la minificación de forma similar a uglify. En modo avanzado, transforma el AST (Sintaxis abstracta del árbol) de nuestro programa de forma agresiva para poder realizar optimizaciones aún más sofisticadas. También tiene una [versión JavaScript](https://www.npmjs.com/package/google-closure-compiler) que puedes [encontrar aquí](https://www.npmjs.com/package/google-closure-compiler). GCC también soporta *la sintaxis de la mayoría de módulos ES2015* por lo que puede [implementar tree-shaking](#tree-shaking).

**Recursos**

- ["Compilando una aplicación angular para producción"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["Aplicaciones 2.5 veces más pequeñas usando Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### Eliminar espacios en blanco de las plantillas

Aunque no veamos el carácter de espacio en blanco (Un caracter que coincida con la expresión regular `\s`) todavía está representado por bytes que son transferidos a través de la red. Si reducimos los espacios en blanco de nuestras plantillas al mínimo podremos reducir el tamaño de nuestro código AoT aún más.

Afortunadamente, no tenemos que hacer esto manualmente. La Interface `ComponentMetadata` ofrece la propiedad `preserveWhitespaces` la cual por defecto tiene valor `false`, porque eliminando los espacios en blanco siempre puede modificar la estructura del DOM. En el caso que cambiemos la propiedad a `true` Angular eliminará los espacios en blancos innecesarios, disminuyendo el tamaño final del bundle.

- [preserveWhitespaces en la documentación de Angular](https://angular.io/api/core/Component#preserveWhitespaces)

### Tree-shaking

Para la versión final de nuestra aplicación normalmente no usaremos el código entero que ofrece Angular y/u otras librerías de terceros, o incluso el que hemos escrito nosotros. Gracias a la naturaleza estática de los módulos ES2015, tenemos la posibilidad de deshacernos del código que no usamos en nuestras aplicaciones.

**Ejemplo**

```javascript
// foo.js
export foo = () => 'foo';
export bar = () => 'bar';

// app.js
import { foo } from './foo';
console.log(foo());
```
Una vez aplicado "tree-shake" en el bundle `app.js` obtendremos:

```javascript
let foo = () => 'foo';
console.log(foo());
```

Esto significa que la exportación `bar` no utilizada no estará incluida en el bundle final.

**Herramientas**

- [Webpack](https://webpack.js.org) - Proporciona un bundling eficiente mediante el [tree-shaking](#tree-shaking). Una vez la aplicación ha sido empaquetada, ésta no exporta el código no utilizado con lo que podremos considerar de manera segura la eliminación de código no utilizado con Uglify.
- [Rollup](https://github.com/rollup/rollup) - Ofrece un bundle eficiente haciendo uso de "tree-shaking", aprovechando la naturaleza estática de los módulos ES2015.
- [Google Closure Compiler](https://github.com/google/closure-compiler) - realiza un montón de optimizaciones y proporciona soporte para el bundle. Originalmente escrito en Java, desde hace poco también tiene una [version JavaScript](https://www.npmjs.com/package/google-closure-compiler) la cual puede encontrarse [aquí](https://www.npmjs.com/package/google-closure-compiler).

*Nota:* GCC todavía no sporta `export *` , el cual es fundamental para la construcción de aplicaciones angular por el amplío uso del patrón "barrel".

**Recursos**

- ["Compilando una aplicación angular para producción"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["Aplicaciones 2.5 veces más pequeñas usando Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)
- ["Uso de operadores encadenados (pipeable) en RxJS"](https://github.com/ReactiveX/rxjs/blob/master/doc/pipeable-operators.md)

### Tree-Shakeable Providers

Desde el lanzamiento de la versión 6 de Angular, el equipo de angular ofrece una nueva característica la cual nos permite que los servicios sean "tree-shakeable", esto significa que los servicios no estarán incluidos en la versión final de nuestro bundle a no ser que estén siendo utilizados por otros servicios o componentes. Esto puede ayudar a reducir el tamaño del bundle eliminando el código no utilizado en el bundle.

Puedes hacer tus servicios "tree-shakeables" utilizando el atributo `provideIn` para definir donde el servicio debe ser inicializado utilizando el decorador `@Injectable()`. De esta forma debes eliminar los servicios del atributo `providers` de la declaración de tu `ngModule` de la siguiente forma:

Antes:

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

Después:

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

Si `MyService` no está incluido en ningún componente/servicio, entonces no estará incluido en el bundle.

**Recursos**

- [Angular Providers](https://angular.io/guide/providers)

### Compilación Ahead-of-Time (AoT)

El desafío para las herramientas existentes (como GCC, Rollup, etc.) son las plantillas tipo-HTML de los componentes de Angular, las cuales no pueden ser analizadas con sus capacidades. Esto hace que el soporte para "tree-shaking" sea menos eficiente porque éstas no están seguras de qué directivas son utilizadas dentro de sus plantillas. El compilador AoT transpila las plantillas de Angular para JavaScript o TypeScript con el sistema de importación de los módulos ES2015. De esta manera somos capaces de aplicar "tree-shake" eficientemente durante el proceso de compilación y eliminar todas las directivas no utilizadas definidas por Angular, librerías de terceros o por nosotros mismos.

**Recursos**

- ["Compilación Ahead-of-Time en Angular"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)

### Compresión

Comprimir las respuestas del servidor es una práctica estándar para reducir el uso de ancho de banda. Al especificar el valor de la cabecera `Accept-Encoding`, el navegador sugiere al servidor qué algoritmos de compresión están disponibles en la máquina del cliente. Por otro lado, el servidor establece el valor de la cabecera `Content-Encoding` de la respuesta para indicar al navegador qué algoritmo se ha elegido para comprimir la respuesta.

**Herramientas**

Las herramientas aquí no son específicas de Angular y dependen completamente del servidor web / de aplicaciones que estamos usando. Algoritmos típicos de compresión son:


- deflate - un algoritmo de compresión de datos y un formato de archivo asociado que utiliza una combinación del algoritmo LZ77 y la codificación de Huffman.
- [brotli](https://github.com/google/brotli) - un algoritmo de compresión sin pérdida de propósito genérico que comprime datos usando una combinación de una variante moderna del algoritmo LZ77, codificación de Huffman y modelado de contexto de segundo orden, con una relación de compresión comparable a la de los mejores métodos de compresión de propósito general actualmente disponibles. Es similar en velocidad a deflate pero ofrece una compresión más densa.

**Recursos**

- ["Mejor compresión que Gzip con Brotli"](https://hacks.mozilla.org/2015/11/better-than-gzip-compression-with-brotli/)
- ["Aplicaciones 2.5 veces más pequeñas usando Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### Precarga de recursos (Pre-fetching)

La precarga de recursos es una gran manera de mejorar la experiencia de usuario. Podemos precargar assets (imágenes, estilos, modulos que serán cargados de [forma diferida](#lazy-loading-of-resources) (Lazy Load), etc.) o datos. Hay diferentes estrategias de precarga pero la mayoría de ellas dependen de cada aplicación.

### Carga diferida de recursos (Lazy load)

En caso de que la aplicación tenga una cantidad de código enorme con cientos de dependencias, es posible que las prácticas enumeradas anteriormente no nos ayuden a reducir el bundle a un tamaño razonable (razonable debe ser entre 100K o 2M, de nuevo, depende completamente de los requisitos de negocio.)

En tales casos, una buena solución podría ser cargar algunos de los módulos de la aplicación de forma diferida (lazy). Por ejemplo, supongamos que estamos construyendo un sistema de comercio electrónico. En este caso, podríamos querer cargar el panel de administración independientemente de la interfaz de usuario orientada al usuario. Una vez que el administrador tenga que agregar un nuevo producto, querremos proporcionar la interfaz requerida para eso. Esto podría ser solo la "Página de añadir producto" o el panel de administración completo, dependiendo de nuestro caso de uso / requisitos de negocio.

**Herramientas**

- [Webpack](https://github.com/webpack/webpack) - permite la carga de módulos de forma asíncrona.
- [ngx-quicklink](https://github.com/mgechev/ngx-quicklink) - router preloading strategy which automatically downloads the lazy-loaded modules associated with all the visible links on the screen
- [ngx-quicklink](https://github.com/mgechev/ngx-quicklink) - Estrategía de precarga basada en el enrutado la cual descarga de forma diferida (lazy load) los módulos asociados a los links visibles en pantalla.

### No cargar de forma diferida la ruta por defecto

Supongamos que tenemos la siguiente configuración de enrutado:

```ts
// Bad practice
const routes: Routes = [
  { path: '', redirectTo: '/dashboard', pathMatch: 'full' },
  { path: 'dashboard',  loadChildren: './dashboard.module#DashboardModule' },
  { path: 'heroes', loadChildren: './heroes.module#HeroesModule' }
];
```

La primera vez que el usuario abre la aplicación usando la url: https://example.com/ será redirigido hacia `/dashboard` la cual disparará la carga diferida (lazy-route) con la ruta `dashboard`. Para que Angular renderice el componente del módulo, deberá descargar el fichero `dashboard.module` y todas sus dependencias. Después, el archivo será parseado por la JavaScript VM y será evaluado.

Disparar solicitudes HTTP extra y ejecutar cálculos innecesarios durante la carga inicial es una mala práctica y ralentizará el renderizado de la página inicial. Considere declarar la ruta de la página predeterminada como no diferida (non-lazy).

### Caché

El almacenamiento en caché es otra práctica común que pretende acelerar nuestra aplicación aprovechando la heurística de que si se solicitó un recurso recientemente, podría solicitarse nuevamente en un futuro próximo.

Para almacenar datos en caché normalmente usamos un mecanismo de almacenamiento en caché personalizado. Para el almacenamiento en caché de assets, podemos utilizar el almacenamiento en caché estándar del navegador o el uso de Service Workers con el [API CacheStorage](https://developer.mozilla.org/en-US/docs/Web/API/Cache).

### Shell de la Aplicación

Para hacer más rápido el rendimiento percibido de su aplicación, usar una [Application Shell](https://developers.google.com/web/updates/2015/11/app-shell).

El shell de la aplicación es la interfaz de usuario mínima que mostramos a los usuarios para indicarles que la aplicación se entregará pronto. Para la generación dinámica del shell de la aplicación puedes utilizar Angular Universal con directivas personalizadas que de forma condicional mostrarán elementos dependiendo de la plataforma de renderizado (por ejemplo, ocultar todo excepto el shell cuando usemos `patform-server`).

**Herramientas**

- [Angular Service Worker](https://angular.io/guide/service-worker-intro) - Tiene como objetivo automatizar el proceso de gestión de los Service Workers. También contiene Service Workers para el almacenamiento en caché de assets, y uno [para generar el shell de la aplicación](https://developers.google.com/web/updates/2015/11/app-shell?hl=es).
- [Angular Universal](https://github.com/angular/angular/tree/master/packages/platform-server) - Universal (isomorphic) JavaScript support for Angular.

**Recursos**

- ["Instant Loading Web Apps with an Application Shell Architecture"](https://developers.google.com/web/updates/2015/11/app-shell)

### Service Workers

Podemos pensar de los Service Worker como un proxy HTTP que está en el navegador. Todas las peticiones enviadas desde el cliente son interceptadas primero por el Service Worker el cual puede procesarlas o enviarlas a través de la red.

Puedes añadir un Service Worker a tu proyecto Angular ejecutando ``` ng add @angular/pwa ```

**Herramientas**

- [Angular Service Worker](https://angular.io/guide/service-worker-intro) - Tiene como objetivo automatizar el proceso de gestión de los Service Workers. También contiene Service Workers para el almacenamiento en caché de assets, y uno [para generar el shell de la aplicación](https://developers.google.com/web/updates/2015/11/app-shell?hl=es).
- [Offline Plugin for Webpack](https://github.com/NekR/offline-plugin) - Plugin para Webpack que añade soporte para el Service Worker con soporte alternativo para AppCache.

**Recursos**

- ["The offline cookbook"](https://jakearchibald.com/2014/offline-cookbook/)
- ["Empezando con service workers"](https://angular.io/guide/service-worker-getting-started)

## Optimizaciones en ejecución

Esta sección incluye prácticas que podrán ser aplicadas con el fin de proporcionar una experiencia de usuario más suave con 60fps (Frames por segundo).

### Utilizar `enableProdMode`

En el ambiente de desarrollo, Angular realiza algunas comprobaciones adicionales para verificar que la detección del cambio no produce ninguna diferencia para alguno de los bindings. De esta manera el framework garantiza que el flujo unidireccional de los datos ha sido seguido.

Para deshabilitar estos cambios para producción no olvide habilitar `enableProdMode`:

```typescript
import { enableProdMode } from '@angular/core';

if (ENV === 'production') {
  enableProdMode();
}
```

### Compilación Ahead-of-Time

Aot puede ser beneficioso no solo para asegurarnos bundles más eficientes aplicando "tree-shaking", sino también para mejorar el rendimiento en tiempo de ejecución de nuestras aplicaciones. La alternativa de Aot es la compilación Just-in-Time (JiT) que se realiza en tiempo de ejecución, por lo tanto, podemos reducir la cantidad de cálculos necesarios para la representación de nuestra aplicación al realizar la compilación como parte de nuestro proceso de construcción.

**Herramientas**

- [angular2-seed](https://github.com/mgechev/angular2-seed) - Proyecto inicial que incluye soporte para la compilación AoT.
- [angular-cli](https://cli.angular.io) Utilizando `ng serve --prod`

**Recursos**

- ["Compilación Ahead-of-Time en Angular"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)

### Web Workers

El problema habitual de las aplicaciones Single-Page-Applicattion (SPA) es que nuestro código generalmente se ejecuta en un solo hilo. Esto significa que si queremos lograr una experiencia de usuario fluida con 60 fps tenemos **a lo sumo 16ms** para la ejecución entre los frames individuales que se están procesando, de lo contrario, se reducirán a la mitad.

En aplicaciones complejas con un gran arbolado de componentes, donde la detección de cambios necesita realizar millones de verificaciones por segundo, no será difícil comenzar a eliminar frames. Gracias al agnosticismo de la plataforma de Angular y su desacoplamiento de la arquitectura DOM, es posible ejecutar toda nuestra aplicación (incluida la detección de cambios) en un Service Worker y dejar el hilo principal de la UI responsable solo de la renderización

**Herramientas**

- El módulo que nos permite ejecutar nuestra aplicación en un Web Worker es apoyado por el equipo do Angular Core. Ejemplos de cómo puede ser utilizado, pueden [encontrarse aquí](https://github.com/angular/angular/tree/master/modules/playground/src/web_workers).
- [Webpack Web Worker Loader](https://github.com/webpack/worker-loader) - A Web Worker Loader for webpack.

**Recursos**

- ["Utilizando Web Workers para aplicaciones más responsive"](https://www.youtube.com/watch?v=Kz_zKXiNGSE)

### Server-Side Rendering

Un gran problema para las tracidionales SPA es que no pueden ser renderizadas hasta que esté disponible todo el código JavaScript solicitado. Esto lleva a dos grandes problemas:

- No todos los motores de búsqueda ejecutan el JavaScript asociado a la página, por lo que no pueden indexar correctamente el contenido de las aplicaciones dinámicas.
- Mala experiencia del usuario, ya que el usuario no verá más que una pantalla en blanco / cargando hasta que se descargue, analice y ejecute el JavaScript asociado con la página.

Server-side rendering soluciona estos problemas pre-renderizando la página requerida en el servidor y proporcionando el marcado de la página durante la carga incial de la página.

**Herramientas**

- [Angular Universal](https://github.com/angular/angular/tree/master/packages/platform-server) - Universal (isomorphic) JavaScript support for Angular.
- [Preboot](https://github.com/angular/preboot) - Library to help manage the transition of state (i.e. events, focus, data) from a server-generated web view to a client-generated web view.

**Recursos**

- ["Angular Server Rendering"](https://www.youtube.com/watch?v=0wvZ7gakqV4)
- ["Angular Universal Patterns"](https://www.youtube.com/watch?v=TCj_oC3m6_U)

### Detección del cambio

En cada evento asíncrono, Angular realiza la detección de cambios en todo el árbol de componentes. Aunque el código que detecta cambios está optimizado por [inline-caching](http://mrale.ph/blog/2012/06/03/explaining-js-vms-in-js-inline-caches.html), Esto puede ser un cálculo pesado en aplicaciones complejas. Una forma de mejorar el rendimiento de la detección de cambios es no realizarla en subárboles que no deben modificarse en función de las acciones recientes.

#### `ChangeDetectionStrategy.OnPush`

La estrategia de detección de cambios `OnPush` nos permite deshabilitar el mecanismo de detección de cambios para subárboles del árbol de componentes. Al establecer la estrategia de detección de cambios en cualquier componente con el valor `ChangeDetectionStrategy.OnPush`, hará que la detección de cambios se realice **solo** cuando el componente haya recibido diferentes entradas. Angular considerará las entradas como diferentes cuando las compare con las entradas anteriores por referencia, y el resultado de la verificación de referencia es `false`. En combinación con [estructuras de datos inmutables](https://facebook.github.io/immutable-js/) `OnPush` puede traer grandes implicaciones de rendimiento para tales componentes "puros".

**Recursos**

- ["Detección del cambio en Angular"](https://vsavkin.com/change-detection-in-angular-2-4f216b855d4c)
- ["Todo lo que necesitas saber sobre la detección del cambio en Angular"](https://blog.angularindepth.com/everything-you-need-to-know-about-change-detection-in-angular-8006c51d206f)

#### Desacoplando el detector de cambios

Otra forma de implementar un mecanismo de detección de cambios personalizado es mediante la separación (detach) y la reinserción (reattach) del detector de cambios (CD) para un componente determinado. Una vez que 'separemos' (`detach`) el CD Angular no realizará la verificación de todo el subárbol de componentes.

Esta práctica se usa normalmente cuando las acciones del usuario o las interacciones con servicios externos activan la detección de cambios con más frecuencia de la necesaria. En tales casos, es posible que desee considerar separar el detector de cambios y volver a instalarlo solo cuando se requiere la detección de cambios.

#### Ejecución fuera de Angular

El mecanismo de detección de cambios de Angular se dispara gracias a [zone.js](https://github.com/angular/zone.js). Zone.js monkey parchea todas las API asíncronas en el navegador y activa la detección de cambios al final de la ejecución de cualquier devolución de llamada asíncrona. En **casos raros** podemos querer que un código dado se ejecute fuera del contexto Angular Zone y, por lo tanto, sin ejecutar el mecanismo de detección de cambios. En tales casos, podemos usar el método `runOutsideAngular` de la instancia de 'NgZone`.

**Example**

En el ejemplo de abajo, puede ver un componente que utiliza esta práctica. Cuando el método `_incrementPoints` es llamado el componente comenzará a incrementar la propiedad `_points` cada 10ms (por defecto). Al incrementar tendremos la ilusión de una animación. Ya que en este caso no queremos activar el mecanismo de detección de cambios para todo el árbol de componentes, cada 10ms, podemos ejecutar `_incrementPoints` fuera del contexto de Angular´s Zone y actualizar el DOM manualmente (ver método set de `points`).

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

**Warning**: Use esta práctica **con mucho cuidado solo cuando esté seguro de lo que está haciendo** porque, si no se usa correctamente, puede llevar a un estado inconsistente del DOM. También tenga en cuenta que el código anterior no se ejecutará en WebWorkers. Para que sea compatible con WebWorker, debe establecer el valor de la etiqueta `label` utilizando el renderizador de Angular.

### Pipes puros

El decorador `@Pipe` acepta como argumento un objeto utilizando el siguiente formato:

```typescript
interface PipeMetadata {
  name: string;
  pure: boolean;
}
```

El flag "pure" indica que el Pipe no depende de ningún estado global y no producirá efectos colaterales (side-effects). Esto significa que el Pipe devolverá el mismo resultado cuando sea llamada con la misma entrada. De esta manera, Angular puede almacenar en caché las salidas de todos los parámetros de entrada con los que se ha invocado el Pipe y reutilizarlos para no tener que volver a calcularlos en cada evaluación.

El valor por defecto de la propiedad `pure` es `true`.

### Directiva `*ngFor`

La directiva `*ngFor` es utilizada para renderizar una colección.

#### Utilizar opción `trackBy`

Por defecto `*ngFor` identifica objetos únicos por referencia.

Lo que significa que cuando el desarrollador rompe la referencia al objeto durante la actualización del contenido del elemento, Angular lo trata como la eliminación del objeto anterior y la agregación del nuevo objeto. Esto afecta a la destrucción del antiguo nodo DOM en la lista y la agregación de un nuevo nodo DOM en su lugar.

El desarrollador puede ofrecer una pista a Angular sobre cómo identificar de forma única el objeto: función de seguimiento personalizado  como la opción `trackBy` para la directiva` * ngFor`. La función trackBy recibe dos argumentos: `índice` (index) y `elemento` (item). Angular utiliza el valor devuelto por la función para llevar un seguimiento
 Tracking function takes two arguments: `index` and `item`. Angular usa el valor devuelto por la función de seguimiento para rastrear la identidad de los elementos. Es muy común usar la ID del registro particular como clave única.

**Example**

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

#### Minimizar elementos del DOM

La representación de los elementos DOM suele ser la operación más costosa al agregar elementos a la interfaz de usuario. El trabajo principal generalmente se debe al insertar el elemento en el DOM y aplicar los estilos. Si `*ngFor` renderiza muchos elementos, los navegadores (especialmente antiguos) pueden disminuir la velocidad y necesitar más tiempo para finalizar la representación de todos los elementos. Esto no es específico para Angular.

Para reducir el tiempo de renderizado, prueba lo siguiente:
- Aplicar scroll virtual a través de [CDK](https://material.angular.io/cdk/scrolling/overview) o [ngx-virtual-scroller](https://github.com/rintoj/ngx-virtual-scroller)
- Reducir la cantidad de elementos DOM representados en la sección `* ngFor` de su plantilla. Por lo general, los elementos DOM innecesarios / no utilizados surgen al ampliar la plantilla una y otra vez. Volviendo a pensar su estructura probablemente hará las cosas más fáciles.
- Utilizar [`ng-container`](https://angular.io/guide/structural-directives#ngcontainer) cuando sea posible.

**Recursos**

- ["directiva NgFor"](https://angular.io/docs/ts/latest/api/common/index/NgFor-directive.html) - official documentation for `*ngFor`
- ["Mejorar rendimiento de Angular utilizando trackBy"](https://netbasal.com/angular-2-improve-performance-with-trackby-cc147b5104e5) - muestra algunos GIFs con esta práctica.
- [Component Dev Kit (CDK) Virtual Scrolling](https://material.angular.io/cdk/scrolling/overview) - API description
- [ngx-virtual-scroller](https://github.com/rintoj/ngx-virtual-scroller) - Muestra una lista "virtual" infinita.

### Optimizar expresiones en plantilla (Template expressions)

Angular ejecuta expresiones de plantilla después de cada ciclo de detección de cambios. Los ciclos de detección de cambios se activan mediante muchas actividades asíncronas, como resoluciones de promesa (promise), resultados http, eventos de temporizador (timer), pulsaciones de teclas y movimientos del ratón.

Las expresiones deben terminar rápidamente o la experiencia de usuario puede empeorar, especialmente en dispositivos más lentos. Considere cachear los valores cuando su cálculo es costoso.

**Recursos**
- [quick-execution](https://angular.io/guide/template-syntax#quick-execution) - Documentación oficial para expresiones en plantilla (template expressions)
- [Increasing Performance - more than a pipe dream](https://youtu.be/I6ZvpdRM1eQ) - ng-conf video in youtube. Using pipe instead of function in interpolation expression
- [Increasing Performance - more than a pipe dream](https://youtu.be/I6ZvpdRM1eQ) - Vídeo en youtube de ng-conf. Usando Pipe en vez de funciones de interpolación.

# Conclusión

La lista de prácticas evolucionará dinámicamente a lo largo del tiempo con prácticas nuevas / actualizadas. En caso de que note que falta algo o si cree que se puede mejorar cualquiera de las prácticas, no dude en reportar un problema y/o un PR. Para más información, por favor, eche un vistazo a la sección inferior "[Contribuyendo](#contributing)".

# Contribuyendo

En caso de que note que falta algo, incompleto o incorrecto, una "pull request" será muy apreciada. For discussion of practices which are not included in the document please [open an issue](https://github.com/mgechev/angular2-performance-checklist/issues).
En caso de que note que falta algo, incompleto o incorrecto, una "pull request" será muy apreciada. Para comentar las prácticas que no están incluidas en el documento, por favor  [abra una issue](https://github.com/mgechev/angular2-performance-checklist/issues).

# License

MIT
