# Checklist de Performance do Angular

<img src="./assets/flash.png" width="1000">

[中文版](./README.zh-CN.md) [Русский](./README.ru-RU.md) [Português](./README.pt-BR.md) [Español](./readme-es-ES.md)

## Introdução

Este documento contém uma lista de boas práticas que vão nos ajudar a melhorar a performance das nossas aplicações. O "Checklist de Performance do Angular" cobre diferentes tópicos - de pré renderização no servidor e criação do pacote da nossa aplicação até performance em tempo de execução e otimização do sistema de detecção de mudança feita pelo framework.


Este documento é dividido em duas seções principais:

- Performance de rede - Lista de práticas que vão melhorar o tempo de carregamento da nossa aplicação. Ela inclui métodos para redução de latência e consumo de rede.

- Performance de Execução - Práticas que melhoram a performance de execução da nossa aplicação. Ela inclui principalmente otimizações de detecção de mudança e renderização

Algumas práticas impactam nas duas categorias então pode haver alguma interceção, mas de qualquer forma, as diferenças nos casos de uso e implicações serão explicitamente mencionadas.

A maioria das subseções com lista de ferramentas, relacionadas a prática específicas, podem nos fazer mais eficientes ao automatizar o nosso fluxo de desenvolvimento

Note que a maioria das práticas são válidas tanto para HTTP/1.1 quanto para HTTP2. Práticas que servem apenas para um protocolo específico será mencionado a versão que ele se aplica.


## Sumário

- [Checklist de Performance do Angular](#checklist-de-performance-do-angular)
  - [Introdução](#introdução)
  - [Sumário](#sumário)
  - [Performance de Rede](#performance-de-rede)
    - [Bundling](#bundling-empacotamento)
    - [Minificação e eliminação de código morto](#minificação-e-eliminação-de-código-morto)
    - [Remover espaços em branco do template](#remover-espaços-em-branco-do-template)
    - [Tree-shaking](#tree-shaking)
    - [Tree-shakeable providers](#tree-shakeable-providers)
    - [Compilação a Ahead-of-Time (AoT)](#compilação-a-ahead-of-time-aot)
    - [Compressão](#compressão)
    - [Pré Carregamento (Pre-fetching) de Recursos](#pré-carregamento-de-recursos)
    - [Carregamento Tardio de Recursos (Lazy Load)](#carregamento-tardio-de-recursos-lazy-load)
    - [Não use lazy-load com a rota padrão](#não-use-lazy-load-na-rota-padrão)
    - [Cache](#cache)
    - [Use uma casca da Aplicação](#use-uma-casca-da-aplicação)
    - [Use Service Workers](#use-service-workers)
  - [Otimizações de Execução](#otimizações-em-tempode-execução)
    - [Use o `enableProdMode`](#use-enableprodmode)
    - [Compilação Ahead-of-Time](#compilação-ahead-of-time-aot)
    - [Web Workers](#web-workers)
    - [Renderização no servidor (Server Side Rendering - SSR)](#renderização-no-servidor-server-side-rendering---ssr)
    - [Detecção de Mudança](#detecção-de-mudança)
      - [`ChangeDetectionStrategy.OnPush`](#changedetectionstrategyonpush)
      - [Removendo o Change Detector](##desacoplando-o-detector-de-mudança)
      - [Execute código fora do angular](#executar-fora-do-angular)
    - [Use pipes puros](#use-pipes-puros)
    - [Diretiva `*ngFor`](#diretiva-ngfor)
      - [Use a opção `trackBy`](#use-a-opção-trackby)
      - [Reduza a quantidade de elementos no DOM](#reduza-a-quantidade-de-elementos-no-dom)
    - [Otimize os template expressions ({{expression}})](#otimize-as-expressões-de-template-template-expressions)
- [Conclusão](#conclusão)
- [Contribuindo](#contribuindo)

## Performance de Rede

algumas dessas ferramentas desta seção ainda estão em desenvolvimento e estão sujeitas a mudança. A equipe do core do Angular está trabalhando em automatizar o máximo possível o processo de build para que muitas coisas fiquem mais transparentes.

### Bundling (Empacotamento)

Bundling ou empacotamento é uma prática padrão que visa reduzir o número de requisições que o browser precisa fazer para entregar a aplicação pedida pelo usuário. Na essência, o bundler recebe uma lista de entry points (Ex. Arquivos JS e CSS), junta esses arquivos e produz um ou mais bundles. Desta forma o browser pode baixar uma aplicação inteira fazendo apenas algumas requisições ao invés de baixar cada arquivo separado.

Conforme a sua aplicação cresce, juntar tudo em um único bundle gigante pode ser contra-produtivo. Explore técnicas de Code Splitting utilizando o webpack.


**Requisições http adicionais não são um problema no HTTP/2 por causa do recurso de [server push](https://http2.github.io/faq/#whats-the-benefit-of-server-push).**

**Ferramentas**

Ferramentas que nos permitem criar bundles para as nossas aplicações de forma eficiente:

- [Webpack](https://webpack.js.org) - provê um bundle eficiente por fazer uso do [tree-shaking](#tree-shaking).
- [Webpack Code Splitting](https://webpack.js.org/guides/code-splitting/) - Técnicas para dividir o seu código.
- [Webpack & http2](https://medium.com/webpack/webpack-http-2-7083ec3f3ce6#.46idrz8kb) - Necessário para dividir o código usando http2.
- [Rollup](https://github.com/rollup/rollup) - provê bundles fazendo uso eficiente do tree-shaking, tendo como vantagem a natuzera estática dos módulos ES2015.
- [Google Closure Compiler](https://github.com/google/closure-compiler) - faz várias otimizações e provê suporte a bundles. Originalmente escrito em Java, recentemente ganhou uma [Versão em javascript](https://www.npmjs.com/package/google-closure-compiler-js) que pode ser [encontrada aqui](https://www.npmjs.com/package/google-closure-compiler-js).
- [SystemJS Builder](https://github.com/systemjs/builder) - provê um bundle de um único arquivo para módulos mixos de injeção de dependencia para o SystemJS
- [Browserify](http://browserify.org/).
- [ngx-build-modern](https://github.com/manfredsteyer/ngx-build-plus/tree/master/ngx-build-modern) - Plugin para o Angular-CLI que cria bundles de duas formas:
  1. Para browsers modernos com módulos do ES2015 e polyfills específicos resultando em um bundle menor.
  2. Código legado adicional usando diferentes polyfills (Padrão)

**Recursos**

- ["Construindo uma aplicação em Angular para Produção (Em Inglês)"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["Aplicações em Angular 2.5x menor com Google Closure Compiler (Em Inglês)"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### Minificação e Eliminação de Código Morto

Essas práticas nos permitem minimizar o consumo de rede reduzindo o tamanho (Payload) da nossa aplicação.

**Ferramentas**

- [Uglify](https://github.com/mishoo/UglifyJS) - Faz a minificação como reduzir o tamanho do nome das variáveis, remover espaços e comentários, eliminação de código morto, etc. Completamente escrito em JavaScript, tem vários plugins para todos os task runners populares.
- [Google Closure Compiler](https://github.com/google/closure-compiler) - Faz a minificação similar ao uglify. No modo avançado transforma agressivamente a árvore sintática abstrata da nossa aplicação para que seja feita otimizações mais sofisticadas. Também possui uma [versão em JavaScript](https://www.npmjs.com/package/google-closure-compiler-js) que pode ser [encontrada aqui](https://www.npmjs.com/package/google-closure-compiler-js). GCC também suporte *a maioria dos módulos do ES2015* então também pode [realizar o tree-shaking](#tree-shaking).

**Recursos**

- ["Construindo uma aplicação em Angular para Produção (Em Inglês)"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["Aplicações em Angular 2.5x menor com Google Closure Compiler (Em Inglês)"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### Remover espaços em branco do template

Mesmo que a gente não veja os espaços em branco ( um caracter que bata com a regex `\s`) eles ainda são representados em bytes que são transferidos pela rede. Se reduzirmos os espaços em branco dos nossos templates também reduziremos o tamanho do código da nossa aplicação.

Ao menos não precisamos fazer isso manualmente. A interface `ComponentMetadata` provê a propriedade `preseveWhitespaces` que por padrão possui o valor `false`, porque os espaços em branco sempre podem influencia o layout do DOM. Caso mudamos esse valor para `true` o Angular vai remover os espaços em brancos desnecessários, levando a uma diminuição do tamanho final do bundle.
- [preserveWhitespaces na documentação do Angular (Em inglês)](https://angular.io/api/core/Component#preserveWhitespaces)


### Tree-shaking

Para a versão final das nossas aplicações, normalmente nós não usamos todo o código fornecido pelo Angular e/ou por bibliotecas de terceiros, até mesmo daquelas que nós escrevemos. Graças a natureza estática dos módulos ES2015, nós conseguimos nos livrar do código que não é utilizado nos nossos apps.

**Exemplo**

```javascript
// foo.js
export foo = () => 'foo';
export bar = () => 'bar';

// app.js
import { foo } from './foo';
console.log(foo());
```
Quando fazemos o tree-shake e compilamos o `app.js` nós temos

```javascript
let foo = () => 'foo';
console.log(foo());
```

Isso significa que não vamos incluir no nosso bundle final o export `bar` não utiizado

**Ferramentas**

- [Webpack](https://webpack.js.org) - provê um bundle eficiente fazendo uso do [tree-shaking](#tree-shaking). Uma vez que a aplicação foi empacotada (bundled), ele não exporta o código não utilizado então pode seguramente se considerado código morto e removido pelo Uglify.
- [Rollup](https://github.com/rollup/rollup) - provê um bundle eficiente fazendo uso do tree-shaking, tendo como vantagem a natureza estática dos módulos ES2015.
- [Google Closure Compiler](https://github.com/google/closure-compiler) - faz otimizações e tem suporte ao empacotamento (Bundling) Originalmente escrito em Java, recentemente ganhou uma [Versão em javascript](https://www.npmjs.com/package/google-closure-compiler-js) que pode ser [encontrada aqui](https://www.npmjs.com/package/google-closure-compiler-js).

*nota:* GCC ainda não suporte `export *`, que é essencial na construções de aplicações em Angular por causa do uso do padrão "barrel".

**Recursos**

- ["Construindo uma aplicação em Angular para Produção (Em Inglês)"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["Aplicações em Angular 2.5x menor com Google Closure Compiler (Em Inglês)"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)
- ["Usando operadores encadeados (Pipeable) no RxJS (Em Ingles)"](https://github.com/ReactiveX/rxjs/blob/master/doc/pipeable-operators.md)

### Tree-Shakeable Providers

Desde a versão 6 do Angular, o time do angular forneceu novas feature para permitir que as services possam fazer uso do tree-shake. Isso significa que as suas services não serão incluídas no seu bundle final a não ser que elas estejam sendo utilizadas por outras services ou componentes. Isso ajuda a reduzir o tamanho do bundle reduzindo a quantidade de código não usado.

Você pode fazer as suas services usarem o tree-shake definindo o atributo `provideIn` com o lugar em que a service deve ser inicializada quando usando o decorator `@Injectable()`. Dessa forma, você pode removê-los do atributo `providers` do seu `NgModule` da seguinte forma:


Antes:

```ts
// app.module.ts
import { NgModule } from '@angular/core'
import { AppRoutingModule } from './app-routing.module'
import { AppComponent } from './app.component'
import { environment } from '../environments/environment'
import { MinhaService } from './app.service'

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    ...
  ],
  providers: [MinhaService],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

```ts
// minha-service.service.ts
import { Injectable } from '@angular/core'

@Injectable()
export class MinhaService { }
```

Depois:

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
// minha-service.service.ts
import { Injectable } from '@angular/core'

@Injectable({
  providedIn: 'root'
})
export class MinhaService { }
```

Se `MinhaService` não for injetada em nenhum componente/service, ela não vai ser incluída no bundle final

**Recursos**

- [Angular Providers (Em Inglês)](https://angular.io/guide/providers)

### Compilação a Ahead-of-Time (AoT)

Um desafio para as ferramentas existentes por aí(Como o GCC, Rollup, etc) são os templates em tipo-HTML dos componentes, que não podem ser analizados com as suas capacidades. Isso faz o suporte ao tree-shaking menos eficientes porque eles não sabem quais diretivas estão sendo utilizadas dentro dos templates. O Compilar AoT transpila os templates do Angular para JavaScript ou TypeScript com os imports do ES2015. Dessa forma, nós conseguimos fazer um tree-shake eficiente durante o processo de bundling e removemos todas as diretivas não utilizadas que foram definidas pelo Angular, bibliotecas de terceiros ou por nós mesmos.


**Recursos**

- ["Compilação Ahead-of-Time no Angular (Em Inglês)"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)

### Compressão

Comprimir as respostas do servidor é uma prática para reduzir o consumo de banda. Ao especificar o valor do cabeçalho `Accept-Encoding`, o browser diz para o servidor quais são os algoritmos disponíveis na máquina do cliente. Do outro lado, o servidor seta o valor do cabeçalho `Content-Encoding` da resposta com a finalidade de dizer ao browser quais algoritmos foram escolhidos para comprimir as respostas.
Compression of the responses' payload is a standard practice for bandwidth usage reduction. By specifying the value of the header `Accept-Encoding`, the browser hints the server which compression algorithms are available on the client's machine. On the other hand, the server sets value for the `Content-Encoding` header of the response in order to tell the browser which algorithm has been chosen for compressing the response.

**Ferramentas**

The Ferramentas here is not Angular-specific and entirely depends on the web/application server that we're using. Typical compression algorithms are:

- deflate - a data compression algorithm and associated file format that uses a combination of the LZ77 algorithm and Huffman coding.
- [brotli](https://github.com/google/brotli) - a generic-purpose lossless compression algorithm that compresses data using a combination of a modern variant of the LZ77 algorithm, Huffman coding and 2nd order context modeling, with a compression ratio comparable to the best currently available general-purpose compression methods. It is similar in speed with deflate but offers more dense compression.

**Recursos**

- ["Better than Gzip Compression with Brotli"](https://hacks.mozilla.org/2015/11/better-than-gzip-compression-with-brotli/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### Pré-Carregamento de Recursos

O pré-carregamento de recursos é uma ótima forma de melhorar a experiência do usuário. Nós podemos inclusive pré-carregar (pre-fetch) assets (imagens, css, módulos que serão carregados com [lazy load](#lazy-loading-of-Recursos), etc) ou dados. Existem diferentes estratégias de pré-carregamento mas a maioria depende da especificidade da aplicação


### Carregamento Tardio de Recursos (Lazy Load)

Caso a aplicação tenha uma abse de código muito grande, com centenas de dependencias, a prática listada acima pode não nos ajudar a reduzir o tamanho do pacote da nossa aplicação para um tamanho aceitável (Aceitável pode ser 100K ou 2M. Isso, de novo, depende completamente do objetivo da aplicação)

Nesses casos, uma boa solução pode ser carregar alguns comentes da aplicação tardiamente. Por exemplo, vamos supor que estamos construindo um sistema de ecommerce. Nesse caso, nós gostariamos de carregar o painel de administração independentemente da interface que os usuários irão ver. Uma vez que o administrador cadastro um novo produto, a gente quer fornecer a interface para aquele produto. Pode ser apenas a página de adicionar um produto ou o painel inteiro, dependendo dos requisitos do negócio.

**Ferramentas**

- [Webpack](https://github.com/webpack/webpack) - allows asynchronous module loading.
- [ngx-quicklink](https://github.com/mgechev/ngx-quicklink) - router preloading strategy which automatically downloads the lazy-loaded modules associated with all the visible links on the screen

### Não use Lazy-Load na rota padrão

Vamos supor que nós temos a seguinte configuração de rota
Lets suppose we have the following routing configuration:

```ts
// Má Prática
const routes: Routes = [
  { path: '', redirectTo: '/dashboard', pathMatch: 'full' },
  { path: 'dashboard',  loadChildren: './dashboard.module#DashboardModule' },
  { path: 'heroes', loadChildren: './heroes.module#HeroesModule' }
];
```

A primeira vez que o usuário abrir a aplicação usando a url: https://exemplo.com ele vão ser redirecionado para o `/dashboard` que vai disparar a rota `dashboard` que está configurada para carregar com lazy lado. Para o Angular conseguir renderizar o componente do módulo, ele vai ter que baixar o arquivo `dashboard.module` e todas as suas dependências. Depois, os arquivos serão lidos pelo JavaScript VM e então processados.

Disparar uma requisição HTTP extra e fazer processamento desnecessário durante o carregamento da página inicial é uma má prática já que aumenta o tempo para renderizar a página inicial. Considere declarar a rota padrão sem usar o Lazy-Load

### Cache


Cache é outra forma bem comum para acelerar a nossa aplicação tendo como heurística que se um recurso foi recentemente requisitado, ele pode ser requisitado em um futuro próximo.
Para o cache, nós normalmente utilizamos macanismos customizados de Cache. Para arquivos estáticos nós podemos usar o cache padrão do navegador ou usar Service Workers com o [CacheStorage API](https://developer.mozilla.org/en-US/docs/Web/API/Cache).

### Use uma Casca da Aplicação

Para fazer a performance percebida da sua aplicação, use uma [Casca da Aplicação](https://developers.google.com/web/updates/2015/11/app-shell).

A casca da aplicação é uma interface mínima, que mostra aos usuários como a aplicação será entregue a eles. Para gerar uma casca da aplicação dinamicamente, você pode usar o Angular Universao com diretivas customizadas que condicionalmente exibe os elementos dependendo da plataforma onde está sendo renderizados (Exemplo: Esconda tudo exceto a Casca da Aplicação quando estiver usando `platform-server`).

**Ferramentas**

- [Angular Service Worker](https://angular.io/guide/service-worker-intro) - aims to automate the process of managing Service Workers. It also contains Service Worker for caching static assets, and one for [generating application shell](https://developers.google.com/web/updates/2015/11/app-shell?hl=en).
- [Angular Universal](https://github.com/angular/angular/tree/master/packages/platform-server) - Universal (isomorphic) JavaScript support for Angular.

**Recursos**

- [aInstant Loading Web Apps with an Application Shell Architecture"](https://developers.google.com/web/updates/2015/11/app-shell)

### Use Service Workers

Nós podemos pensar no Service Worker com um proxy HTTP que fica no Browser. Todas as requisiçõs feitas do cliente são interceptadas pelo Service Worker que pode processá-las ou passar adiante para a rede.

Você pode adicionar um Service Worker ao seu projeto ocm o seguinte comando
``` ng add @angular/pwa ```


**Ferramentas**

- [Angular Service Worker](https://angular.io/guide/service-worker-intro) - aims to automate the process of managing Service Workers. It also contains Service Worker for caching static assets, and one for [generating application shell](https://developers.google.com/web/updates/2015/11/app-shell?hl=en).
- [Offline Plugin for Webpack](https://github.com/NekR/offline-plugin) - Webpack plugin that adds support for Service Worker with a fall-back to AppCache.

**Recursos**

- ["The offline cookbook"](https://jakearchibald.com/2014/offline-cookbook/)
- ["Iniciando com Service Workers (Em Inglês)"](https://angular.io/guide/service-worker-getting-started)

## Otimizações em tempode execução

Essa seção inclui práticas que podem ser aplicadas para fornecer uma experiencia fluída para os nossos usuários com 60 frames por segundo (fps)

### Use `enableProdMode`

No modo de desenvolvimento, o Angular faz algumas checagens extras para verificar se o sistema de detecção de mudança não resultou em nenhuma diferença para nenhum dos bindings. Dessa forma, a framework garante que o fluxo unidirecional dos dados está sendo seguido.

Para desabilitar essas checagens adicionais em produção, não se esqueça de executar `enableProdMode`:

```typescript
import { enableProdMode } from '@angular/core';

if (ENV === 'production') {
  enableProdMode();
}
```

### Compilação Ahead-of-Time (AoT)

AoT pode ser útil não apenas por fazer bundles mais eficientes usando o tree-shake, mas também por fornecer melhor performance em tempo de execução na nossa aplicação. A Alternativa ao AoT é a Compilação em Tempo de Execução (Just-in-Time [JiT]) que é feita durante a execução do código, portante nós podemos reduzir a quantidade de processamento  necessária para a nossa aplicação fazendo com que a compilação seja parte do processo de build

**Ferramentas**

- [angular2-seed](https://github.com/mgechev/angular2-seed) - Projeto inicial com suporte a compilação AoT
- [angular-cli](https://cli.angular.io) Usando o `ng serve --prod`

**Recursos**

- ["Ahead-of-Time Compilation in Angular"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)

### Web Workers

Um problema típico das aplicações Single-Page (Single-Page Apllications [SPA]) é que nosso código por padrão roda em uma única thread. Isso significa que se nós quisermos fornecer uma experiência mais fluída aos nossos usuários com 60fps nós temos **no máximo 16ms** para executar cada frame que será renderizado, caso contrário essa valor cai pela metade.

Em aplicações complexas com um DOM muito grande, onde o sistema de detecção de mudança precisa realizar milhares de checagens a cada segundo não vai ser muito difícil começar a perder frames. Graças a plataforma agnóstica do Angular e ele ser desacoplado da arquitetura do DOM é possível rodar a nossa aplicação inteira, (inclusive a detecção de mudanças) em um Web Worker e deixar a thread principal responsável apenas pela renderização da interface.


**Ferramentas**

- O módulo que nossa aplicação rode em um Web Worker é suportado pela equipe do Angular. Exemplo sobre como usar, podem [ser encontrados aqui (Em Inglês)](https://github.com/angular/angular/tree/master/modules/playground/src/web_workers).
- [Webpack Web Worker Loader](https://github.com/webpack/worker-loader) -Um Loader Web Worker para o Webpack.

**Recursos**

- [“Usando Web Workers para um app mais responsivo" (Em Inglês)](https://www.youtube.com/watch?v=Kz_zKXiNGSE)

### Renderização no Servidor (Server-Side Rendering - SSR)

Um grande problema das SPA é que elas não podem ser renderizadas até que todo o Javascript necessário para a renderização inicial esteja disponível. Isso nos leva a 2 grandes problemas:

- Nem todos os motores de busca estão rodando o Javascript associado a página ent então elas não conseguem indexar adequadamente o conteúdo dinâmico da nossa aplicação
- Má experiência para o usuário, já que ele não vai ver nada do que uma tela em branco  ou carregando até que todo o JavaScript associado à página seja baixado, processado e executado.

A renderização no servidor resolve esses problemas pré-renderizando estas páginas no servidor e fornecendo o conteúdo da página durante a etapa inicial de carregamento.

**Ferramentas**

- [Angular Universal](https://github.com/angular/angular/tree/master/packages/platform-server) -  Suporte par aJavascript Universal  (isomórfico) do Angular
- [Preboot](https://github.com/angular/preboot) - Biblioteca que ajuda na transição dos estados (i.e. eventos, focos, dados)de uma view gerada no servidor para uma view gerada no cliente.

**Recursos**

- [“Rendererização no servidor com o Angular”](https://www.youtube.com/watch?v=0wvZ7gakqV4)
- [“Padrões para o Angular Universal“](https://www.youtube.com/watch?v=TCj_oC3m6_U)

### Detecção de Mudança

A cada evento assíncrono o Angular executa o sistema de detecção de mudança em toda a árvore de componentes. Mesmo que o código que detecte mudanças seja otimizado para [cache-inline](http://mrale.ph/blog/2012/06/03/explaining-js-vms-in-js-inline-caches.html), isso ainda pode ser um bastante pesado em aplicações maiores. Uma forma de melhorar a performance da detecção de mudanças é não executá-lo nas subárvore que supostamente não foram alteradas baseadas nas ações recents

#### `ChangeDetectionStrategy.OnPush`

A estratégia `OnPush` nos permite desabilitar o mecanismo de detecção de mudança para as subárvores de uma árvore de componentes.  Configurando essa estratágia para um componente, ela só vai disparar a detecção de mudança **apenas** quant o componente receber algum valor diferente. O Angular vai considerar uma entrada diferente quando comparar com o valor anterior por referencia, e se o resultado for `false`. Em conjunto com [estrutura de dados imutáveis] `OnPush` pode trazer grandes implicações de performance por usar componentes “puros” (Pure Components)


**Recursos**

- [“Deteção de mudança no angular (Em Inglês)"](https://vsavkin.com/change-detection-in-angular-2-4f216b855d4c)
- [“Tudo o que você precisa saber sobre a detecção de mudança no angular (Em Inglês)"](https://blog.angularindepth.com/everything-you-need-to-know-about-change-detection-in-angular-8006c51d206f)

#### Desacoplando o detector de mudança

Uma outra forma de implementar um mecanismo customizado de detecção de mudança é desacoplar ( detach) e reacoplar (reattach) o detector de mudança de um componente. Uma vez que descolamos o detector de mudança o Angular não vai mais verificar a árvore daquele componente.
Essa prática é utilizada normalmente quando o as ações do usuário ou interações com serviços externos disparam a detecção de mudanças mais vezes do que o necessário. Nesse caso, nós podemos considerar remover o detector de mudanças e adicioná-lo, apenas quando for necessário verificar alguma alteração na aplicação.


#### Executar fora do Angular

O Mecanismo de detecção de mudança do Angular é disparado graças ao [zone.js](https://github.com/angular/zone.js). Zone.js faz o monkey patch de todas as APIs assíncronas no browser e executa a detecção de mudança no final de qualquer callback assíncrono. Em **casos raros** nós podemos querer pegar um código e executar fora do contexto do Angular Zone, sem executar o mecanismo de detecção de mudança. Nesses casos nós podemos usar o método `runOutsideAngular` na instância do `NgZone`.

**Exemplo**

No snippet abaixo, você pode pode ver um exemplo de um componente que usa essa prática. Quando o método `#incrementarPontos` é chamado, o componente vai começar a incrementar a propriedade `#points` a cada 10ms ( por padrão). Ao incrementar teremos uma ilusão de uma animação. Como não queremos que o angular dispare o mecanismo de deteção de mudanças para toda a árvore do componente a cada 10ms, nós podemos rodar a função `#incrementarPontos` fora do contexto da Zona do Angular e atualizar o DOM manualmente (Veja o setter de `pontos`)


```ts
@Component({
  template: '<span #label></span>'
})
class PointAnimationComponent {

  @Input() duration = 1000;
  @Input() stepDuration = 10;
  @ViewChild('label') label!: ElementRef;

  @Input() set points(val: number) {
    this.#points = val;
    if (this.label) {
      this.label.nativeElement.innerText = this._pipe.transform(this.points, '1.0-0');
    }
  }
  get points() {
    return this.#points;
  }

   #incrementInterval: any;
   #points: number = 0;

  constructor(private _ngZone: NgZone, private _pipe: DecimalPipe) {}

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
        this.#incrementPoints(change.currentValue);
      });
    }
  }

  #incrementPoints(newVal: number) {
    const diff = newVal - this.points;
    const step = this.stepDuration * (diff / this.duration);
    const initialPoints = this.points;
    this.#incrementInterval = setInterval(() => {
      let nextPoints = Math.ceil(initialPoints + diff);
      if (this.points >= nextPoints) {
        this.points = initialPoints + diff;
        clearInterval(this.#incrementInterval);
      } else {
        this.points += step;
      }
    }, this.stepDuration);
  }
}
```

**Aviso**: Use essa prática **com muito cuidado e somente quando tiver certeza do que está fazendo** porque se não usado da forma correta pode levar a um estado inconsistent do DOM. Note também que o código acima não vai rodar em um WebWorker. Para fazer ele compatível com um WebWorker, você precisa alterar o valor da label usando o Renderer do Angular.

### Use pipes puros


O decorator `@Pipe`aceita um objeto literal como argumento usando o seguinte formato:

```typescript
interface PipeMetadata {
  name: string;
  pure: boolean;
}
```

A flag `pure` indica que o pipe não depende de nenhum estado global e não produz efeitos colaterais (Side-effects). Isso significa que o pipe vai retornar sempre o mesmo resultado quando chamado passando os mesmos valores. Dessa forma, o Angular pode fazer cache das saídas de todos os parâmetros de entradas passados para um pipe que já foi chamado, e reusar esse valor no lugar de recalcula-los em cada execução

O valor padrão da propriedade `pure`  é `true`.

### Diretiva `*ngFor`

A diretiva `*ngFor` é usada para renderizar uma coleção

#### Use a opção `trackBy`

Por padrão o `*ngFor` identifica a unicidade de um objeto por referência.
O que significa que quando a desenvolvedora quebra a referencia de um objeto ao atualizar o seu conteúdo, o Angular trata ele como a remoção de um antigo objeto e adição de um novo. O efeito disso é remover o nó do DOM da lista e adicionar um novo nó ao DOM em seu lugar.

A desenvolvedor pode fornecer uma dica para o angular identificar a unicidade de um objeto: Fornecer uma função de rastreamento customizada na opção `trackBy` para a diretiva `*ngFor`. Essa função recebe dois argumentos: `index` e `item`. O Angular usa o valor retornado da função de rastreamento para mapear a identidade dos itens. É muito comum utilizar a propriedade Id de um registro como a chave de unicidade.

**Exemplo**

```typescript
@Component({
  selector: 'yt-feed',
  template: `
  <h1>Seus vídeos</h1>
  <yt-player *ngFor="let video of feed; trackBy: trackById" [video]="video"></yt-player>
`
})
export class YtFeedComponent {
  feed = [
    {
      id: 3849, // Note o campo “id”. Nós vamos referenciá-la na função “trackById”
      title: "Angular em 60 minutos”,
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

#### Reduza a quantidade de elementos no DOM

Normalmente renderizar os elementos do DOM é a operação mais pesadas ao adicionar elementos à interface. A maior parte do trabalho é causada por inserir elementos no dom aplicando novos estilos. Se um `*ngFor` renderiza vários elementos, os navegadores (especialmente os antigos) podem ficar lentos e precisar de mais tempo para renderizar todos os elementos. Isso não é específico do Angular.
Rendering the DOM elements is usually the most expensive operation when adding elements to the UI.

Para reduzir o tempo de renderização, tente o seguinte


- Aplicar Virtual  Scrolling usando o  [CDK](https://material.angular.io/cdk/scrolling/overview) ou [ngx-virtual-scroller](https://github.com/rintoj/ngx-virtual-scroller)
- Reduzir a quantidade de elementos do DOM renderizados no `*ngFor` do seu template. Normalmente elementos não utilizados ou desnecessários aparecem ao extender um template várias vezes. Repensar a estrutura provavelmente vai fazer as coisas ficarem mais fáceis
- Use [`ng-container`](https://angular.io/guide/structural-directives#ngcontainer) onde possível

**Recursos**

- [“Diretiva NgFor”](https://angular.io/docs/ts/latest/api/common/index/NgFor-directive.html) - documentação oficial do `*ngFor`
- [“Angular - Melhorando a performance com o trackBy"](https://netbasal.com/angular-2-improve-performance-with-trackby-cc147b5104e5) - Exibe algumas gifs com esta prática
- [Component Dev Kit (CDK) Virtual Scrolling](https://material.angular.io/cdk/scrolling/overview) - Descrição da API
- [ngx-virtual-scroller](https://github.com/rintoj/ngx-virtual-scroller) - Exibe uma lista infinita virtualmente

### Otimize as expressões de template (Template Expressions)

Angular executa as expressões de template a cada ciclo de detecção de mudança. Os ciclos de detecção de mudança são disparados por várias atividades assíncronas, como promises que foram resolvidas, resultado de uma requisição http, eventos de timer, teclas que foram apertadas e movimento do mouse.

As expressões devem terminar rapidamente ou o usuário pode ter uma experiência ruim, especialmente em dispositivos mais lentos. Consider fazer cache de valores quando for necessário um processamento mais pesado.

**Recursos**
- [quick-execution](https://angular.io/guide/template-syntax#quick-execution) - Documentação oficial das expressões de template
- [Melhorando a Performance - Mais do que um sonho(Em inglês)](https://youtu.be/I6ZvpdRM1eQ) - Vídeo do ng-conf no youtube. Usando um pipe ao invés de uma função interpolada.

# Conclusão

Esta lista de práticas vai crescer dinamicamente ao longo do tempo com novas práticas ou atualizações. Caso você note alguma coisa faltando ou ache que estas práticas podem ser melhoradas não elite em criar uma issue ou pull request. Para mais informações veja a seção  “[Contribuindo](#contribuindo)” abaixo.

# Contribuindo

Caso você note algo faltando, incompleto ou incorreto um pull request vai ser imensamente apreciado. Para discutir as práticas que não estão inclusas neste documento por favor [abra uma issue](https://github.com/mgechev/angular2-performance-checklist/issues).

# Licença

MIT

