# <a name="angular-2-performance-checklist"></a>Angular パフォーマンスチェックリスト

<img src="./assets/flash.png" width="1000">

- [English](./README.md)
- [中文版](./README.zh-CN.md)
- [Русский](./README.ru-RU.md)
- [Português](./README.pt-BR.md)
- [Español](./README.es-ES.md)

## <a name="introduction"></a>序論

このドキュメントは、Angularアプリケーションのパフォーマンスを向上させるのに役立つ実践方法の一覧を含みます。
「Angular パフォーマンスチェックリスト」では、実行時パフォーマンスとフレームワークによって実行される変更検出の最適化のための、サーバーサイドでのプリレンダリングならびにアプリケーションのバンドルについて説明しています。

この文章は大きく分けて２つの章に分かれています。

- ネットワークパフォーマンス - 主に私たちのアプリケーションのロード時間を改善するための一覧です。これは待機時間と帯域幅を削減するための方法が含まれています。
- 実行時のパフォーマンス - アプリケーション実行時のパフォーマンスを向上させる一覧です。主に変更の検出とレンダリング関連の最適化が含まれます。

いくつかの項目は両方に影響するため、多少被っている可能性がありますが、ユースケースの違いと理由については明確に言及します。

ほとんどの節では、特定の実践方法に関連したツール一覧を書いています、それにより開発フローを自動化し、効率を良くします。

ほとんどの内容はHTTP/1.1 と HTTP/2 の両方に有効です。
将来的に適用される可能性のあるプロトコルのバージョンは、節ごとに適宜指定することにより、例外的な実践方法についても別記します。

## <a name="table-of-content"></a>目次

- [Angular パフォーマンスチェックリスト](#angular-2-performance-checklist)
  - [序論](#introduction)
  - [目次](#table-of-content)
  - [ネットワークパフォーマンス](#network-performance)
    - [バンドリング](#bundling)
    - [ミニファイと不要コードの削除](#minification-and-dead-code-elimination)
    - [テンプレート中の空白を削除](#remove-template-whitespace)
    - [ツリーシェイキング](#tree-shaking)
    - [ツリーシェイキング可能なプロバイダ](#tree-shakeable-providers)
    - [Ahead-of-Time (AoT) コンパイル](#ahead-of-time-aot-compilation)
    - [圧縮](#compression)
    - [リソースの事前読込み](#pre-fetching-resources)
    - [リソースの遅延読み込み](#lazy-loading-of-resources)
    - [デフォルトのルートを遅延読み込みにしない](#dont-lazy-load-the-default-route)
    - [キャッシング](#caching)
    - [アプリケーションシェルを使う](#use-application-shell)
    - [サービスワーカーを使う](#use-service-workers)
  - [実行時の最適化](#runtime-optimizations)
    - [`enableProdMode`を使う](#use-enableprodmode)
    - [Ahead-of-Time コンパイル](#ahead-of-time-compilation)
    - [Webワーカー](#web-workers)
    - [サーバサイドレンダリング](#server-side-rendering)
    - [変更検出](#change-detection)
      - [`ChangeDetectionStrategy.OnPush`](#changedetectionstrategyonpush)
      - [Change Detectorの切り離し](#detaching-the-change-detector)
      - [Run outside Angular](#run-outside-angular)
    - [純粋なpipeを使う](#use-pure-pipes)
    - [`*ngFor` ディレクティブ](#ngfor-directive)
      - [`trackBy`のオプションを使う](#use-trackby-option)
      - [DOM要素を小さくする](#minimize-dom-elements)
    - [テンプレート中の式を最適化する](#optimize-template-expressions)
- [終わりに](#conclusion)
- [貢献する](#contributing)

## <a name="network-performance"></a>ネットワークパフォーマンス

この章にあるいくつかのツールはまだ開発中なので、変更される可能性があります。
Angularコアチームは、可能な限りアプリケーションのビルドプロセス自動化に取り組んでいるため、多くのことが透過的に行われています。

### <a name="bundling"></a>バンドリング

バンドリングは、ユーザーにアプリケーションを配信するために、ブラウザ側が実行するリクエストの数を減らすことを目的とした、標準的な方法です。
本質的に、バンドラーは入力としてエントリーポイントの一覧を受け取り、1つ以上のバンドルを生成します。
そうすることで、ブラウザが個々のリソースを個別に要求するのではなく、少ないリクエストでアプリケーション全体を取得することを可能にします。

アプリケーションが大きくなるにつれ、すべてを1つバンドルに纏めていくのは逆効果になる可能性があります。
その場合、Webpackを使ったコード分割を行う必要が出てきます。

**サーバプッシュによりhttp/2はhttpリクエストを含みません。[server push](https://http2.github.io/faq/#whats-the-benefit-of-server-push) 機能を参照してください**

**ツール一覧**

アプリケーションを効率的にバンドルできるツールは次のようなものがあります:

- [Webpack](https://webpack.js.org) - [ツリーシェイキング](#tree-shaking)を行うことによって効率的なバンドリングを提供します。
- [Webpack Code Splitting](https://webpack.js.org/guides/code-splitting/) - コード分割のためのテクニック。
- [Webpack & http2](https://medium.com/webpack/webpack-http-2-7083ec3f3ce6#.46idrz8kb) - http2を使った分割する時に必要です。
- [Rollup](https://github.com/rollup/rollup) - ES2015モジュールの静的な性質を利用し、効率的なツリーシェイキングによるバンドルを提供します。
- [Google Closure Compiler](https://github.com/google/closure-compiler) - 十分な最適化と、バンドルのサポートを提供します。
元となるJavaで書かれたものがあり、[ここ](https://www.npmjs.com/package/google-closure-compiler)には最近[JavaScriptで書かれたバージョン](https://www.npmjs.com/package/google-closure-compiler)もあります。
- [SystemJS Builder](https://github.com/systemjs/builder) - 依存関係の混在したモジュールツリーとなる、SystemJSベースとなる単一ファイルのビルド構成を提供します。
- [Browserify](http://browserify.org/).
- [ngx-build-modern](https://github.com/manfredsteyer/ngx-build-plus/tree/master/ngx-build-modern) - Angular-CLI用のプラグインで、アプリケーションのバンドルを2種類にビルドできます:

  1. ES2015モジュールと、必要となるpolyfillだけを搭載した、最新のブラウザでのみ動作するバンドルが小さい構成。
  2. 多くのpolyfillと幅広いコンパイラターゲットを使用する、追加のレガシーバージョン構成（基本の設定）。

**参考文献**

- ["プロダクション用Angularアプリケーション構築"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["Google Closure Compilerを利用した2.5倍小さいアプリケーション"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### <a name="minification-and-dead-code-elimination"></a>ミニファイと不要コードの削除

この方法により、アプリケーションのペイロードを減らすことができ、通信量を最小限に抑えることができます。

**ツール一覧**

- [Uglify](https://github.com/mishoo/UglifyJS) - 変数の名前修飾(mangling)、コメントと空白の削除、不要なコードの削除などでファイルサイズの縮小を実行します。すべてJavaScriptで書かれており、ほとんどのタスクランナーのためのプラグインが用意されています。
- [Google Closure Compiler](https://github.com/google/closure-compiler) - uglifyによるファイルサイズの縮小と動作が似ています。高度な利用方法には洗練された最適化を実行できるように、プログラムのASTを積極的に変換します。[JavaScriptのバージョン](https://www.npmjs.com/package/google-closure-compiler)が存在し、[ここで](https://www.npmjs.com/package/google-closure-compiler)詳細を見ることができます。
GCCはほとんどのES2015モジュール構文をサポートしているため、[ツリーシェイキングで実行](#tree-shaking)ができます。

**参考文献**

- ["プロダクション用Angularアプリケーション構築"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["Google Closure Compilerを利用した2.5倍小さいアプリケーション"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### <a name="remove-template-whitespace"></a>テンプレート中の空白を削除

空白文字（正規表現の`\s`と言った文字）は見えませんが、ネットワーク上で転送されるバイト数として数えられます。空白をテンプレートからなるべく減らすことで、AoTコードのバンドルサイズをさらに減らすことが可能になります。

幸いなことに、これは手動で行う必要はありません。
`ComponentMetadata`インターフェースは初期値として`false`を持つプロパティ`preserveWhitespaces`を提供します。空白を取り除くことは常にDOM側のレイアウトに影響するかもしれないからです。
このプロパティを `true`に設定した場合、Angularは不要な空白を削除してバンドルサイズをさらに縮小します。

- [AngularドキュメントのpreserveWhitespaces](https://angular.io/api/core/Component#preserveWhitespaces)

### <a name="tree-shaking"></a>ツリーシェイキング

私たちのアプリケーションの最終版では、Angularやサードパーティのライブラリ、そして私たちが書いたものでさえ全てのコードを使うことは通常ありません。
ES2015モジュールの静的な性質のおかげで、アプリで参照のないコードを取り除くことができます。

**サンプルコード**

```javascript
// foo.js
export foo = () => 'foo';
export bar = () => 'bar';

// app.js
import { foo } from './foo';
console.log(foo());
```

`app.js`をツリーシェイキングでバンドルすると、以下のようになります。

```javascript
let foo = () => 'foo';
console.log(foo());
```

これは、未使用のエクスポート`bar`が最終的なバンドルに含まれないことを意味します。

**ツール一覧**

- [Webpack](https://webpack.js.org) - [ツリーシェイキング](#tree-shaking)を実行することで効率的なバンドリングを提供します。 アプリケーションがバンドルされる時に、未使用のコードはエクスポートされず不要なコードと見なされ、結果としてUglifyによって削除されます。
- [Rollup](https://github.com/rollup/rollup) - ES2015モジュールの静的な性質を利用して、効率的なツリーシェイキングを実行したバンドルを提供します。
- [Google Closure Compiler](https://github.com/google/closure-compiler) - 十分な最適化と、バンドルのサポートを提供します。
元となるJavaで書かれたものがあり、[ここ](https://www.npmjs.com/package/google-closure-compiler)には最近[JavaScriptで書かれたバージョン](https://www.npmjs.com/package/google-closure-compiler)もあります。

**注意:** GCCはまだ export * をサポートしていません。これは"バレル"パターンが多用されるAngularアプリケーションの構築には重要な要素です。

**参考文献**

- ["プロダクション用Angularアプリケーション構築"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["Google Closure Compilerを利用した2.5倍小さいアプリケーション"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)
- ["RxJSでパイプ可能演算子を使用する"](https://github.com/ReactiveX/rxjs/blob/master/doc/pipeable-operators.md)

### <a name="tree-shakeable-providers"></a>ツリーシェイキング可能なプロバイダ

Angularのバージョン 6 以降のリリースでは、Angularチームはサービスをツリーシェイキング可能にするための新機能を提供しました。
つまり、他のサービスまたはコンポーネントによって使用されていない限り、そのサービスは最終的なバンドルに含まれません。
これは、バンドルから未使用のコードを削除することによってバンドルサイズを減らすのに役立ちます。

`providedIn`属性を使ってサービスをツリーシェイキング可能にするには、`@Injectable()`デコレータを使う時にサービスをどこに初期化するかを定義します。
次に、 `NgModule`宣言の`provider`属性とimport文から以下のように削除します。

修正前:

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

修正後:

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

`MyService`がどのコンポーネントやサービスにも注入されていない場合、バンドルには含まれません。

**参考文献**

- [Angular Providers](https://angular.io/guide/providers)

### <a name="ahead-of-time-aot-compilation"></a>Ahead-of-Time (AoT) コンパイル

既存のツール（GCC、Rollupなど）に対する課題は、コンポーネント中にあるHTMLテンプレートです。これらは既存ツールの機能では分析できません。
そしてこれは、どのディレクティブがテンプレート内で参照されているのかわからないため、ツリーシェイキングのサポートを非効率にしてしまいます。
AoTコンパイラは、AngularのHTMLテンプレートをES2015モジュールのインポートを利用しJavaScriptまたはTypeScriptに変換します。
このようにして、バンドル中に効率的にツリーシェイキングを行い、Angularのサードパーティ製ライブラリや自分たちの未使用ディレクティブを削除することができます。

**参考文献**

- ["AngularのAhead-of-Timeコンパイル"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)

### <a name="compression"></a>圧縮

レスポンスのデータ転送量の圧縮は、帯域幅使用量を減らすための標準的な方法です。
ヘッダに `Accept-Encoding`の値を指定することで、ブラウザはどの圧縮アルゴリズムがクライアントのマシンで利用可能であるかをサーバに教えます。
一方、サーバーは、レスポンスを圧縮するためにどのアルゴリズムが選択されたかをブラウザに伝えるために、レスポンスの `Content-Encoding`ヘッダに値を設定します。

**ツール一覧**

このツールはAngular固有のものではなく、私たちが使用しているWeb/アプリケーションサーバーに完全に依存します。
代表的な圧縮アルゴリズムは次のとおりです:

- deflate - LZ77アルゴリズムとハフマン符号化の組み合わせを使用するデータ圧縮アルゴリズムおよび関連ファイル形式。
- [brotli](https://github.com/google/brotli) - 最新のLZ77アルゴリズムの変種、ハフマン符号化、および2次コンテキストモデリングの組み合わせを使用して、現在利用可能な最善の汎用圧縮方法に匹敵する圧縮率でデータを圧縮できる汎用目的の可逆圧縮アルゴリズム。 それはdeflateと同じくらいの速度だが、より高密度な圧縮を提供します。

**参考文献**

- ["Gzip圧縮より優れているBrotli"](https://hacks.mozilla.org/2015/11/better-than-gzip-compression-with-brotli/)
- ["Google Closure Compilerを利用した2.5倍小さいアプリケーション"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### <a name="pre-fetching-resources"></a>リソースの事前読込み

リソースの事前読込みは、ユーザーエクスペリエンスを向上させるための優れた方法です。
アセット（画像、スタイル、[遅延読み込み](#lazy-loading-of-resources)されるモジュールなど）やデータは事前読み込みすることができます。
さまざまな事前読みの戦略がありますが、それらのほとんどはアプリケーションの仕様に依存します。

### <a name="lazy-loading-of-resources"></a>リソースの遅延読み込み

ターゲットのアプリケーションが数百の依存関係を持つ巨大なコードベースを持っている場合、上記のプラクティスは私たちのバンドルを合理的なサイズにするのを助けることができないかもしれません。（妥当サイズは100K~2M辺りですが、これも完全にビジネス目標に依存します）

そのような場合の良い解決策はアプリケーションのモジュールのいくつかを遅延してロードすることです。
たとえば、電子商取引システムを構築しているとします。
この時にユーザー向けのUIとは別に管理パネルをロードできるようにておくことで、管理者が新しい製品を追加する必要なときだけに必要なUIを提供できます。
この提供する内容は要件次第で「製品ページの追加」か「管理パネル全体」のいずれかになるでしょう。

**ツール一覧**

- [Webpack](https://github.com/webpack/webpack) - 非同期モジュールの読み込みを可能にします。
- [ngx-quicklink](https://github.com/mgechev/ngx-quicklink) - 画面に表示されているリンクに関連付けられている遅延ロードモジュールを自動的にダウンロードするルーター事前ロード戦略

### <a name="dont-lazy-load-the-default-route"></a>デフォルトのルートを遅延読み込みにしない

次のようなルーティング設定があるとしましょう:

```ts
// 良くない例
const routes: Routes = [
  { path: '', redirectTo: '/dashboard', pathMatch: 'full' },
  { path: 'dashboard',  loadChildren: () => import('./dashboard.module').then(mod => mod.DashboardModule) },
  { path: 'heroes', loadChildren: () => import('./heroes.module').then(mod => mod.HeroesModule) }
];
```

ユーザーが初めて url:`https://example.com/` にアクセスしてアプリケーションを開くと、パス`dashboard`の遅延ルートをトリガーする`/dashboard`にリダイレクトされます。 Angularがモジュールのブートストラップコンポーネントをレンダリングするためには、ファイル `dashboard.module`とそのすべての依存関係をダウンロードした後に、ファイルをJavaScript VMで解析し評価する必要があります。

最初のページのロード中に余分なHTTPリクエストをトリガーして不必要な計算を実行すると、最初のページレンダリングが遅くなるため、あまり良くありません。
デフォルトのページルートを遅延しないように宣言することを検討してください。

### <a name="caching"></a>キャッシング

キャッシングとは、あるリソースが最近要求された場合、近い将来また要求される可能性があるという推測に基づいてアプリケーションを高速化する一般的な方法です。

データをキャッシュするためには、一般的に独自のキャッシュの仕組みを使います。
静的資産のキャッシュには、標準のブラウザキャッシュまたはService Workersと[CacheStorage API](https://developer.mozilla.org/en-US/docs/Web/API/Cache)を使用することができます。

### <a name="use-application-shell"></a>アプリケーションシェルを使う

アプリケーションのパフォーマンスをより速くするためには、[アプリケーションシェル](https://developers.google.com/web/updates/2015/11/app-shell)を使用してください。

アプリケーションシェルは、ユーザーにアプリケーションがもう間もなく動作しはじめる事を伝えるための最小のユーザーインターフェイスです。
アプリケーションシェルを動的に生成するため、使用するレンダリングプラットフォーム毎に要素の表示を切り替えるカスタムディレクティブを付けてAngular Universalを使用できます。（つまり、`platform-server`を使うときはApp Shell以外のものはすべて隠蔽してください）

**ツール一覧**

- [Angular Service Worker](https://angular.io/guide/service-worker-intro) - サービスワーカーの管理プロセスを自動化することを目指しており、静的な資産をキャッシュするためのサービスワーカーと、[アプリケーションシェルの生成](https://developers.google.com/web/updates/2015/11/app-shell?hl=ja)も行います。
- [Angular Universal](https://github.com/angular/angular/tree/master/packages/platform-server) - Angularに対する Universal (isomorphic) JavaScriptをサポートします。

**参考文献**

- ["アプリケーションシェルアーキテクチャを使用したWebアプリケーションの即時ロード"](https://developers.google.com/web/updates/2015/11/app-shell)

### <a name="use-service-workers"></a>サービスワーカーを使う

サービスワーカーは、ブラウザ内にあるHTTPプロキシのようなものです。クライアントから送信されたすべての要求は最初にサービスワーカーによって傍受され、サービスワーカーは受け取ったデータを処理、またはネットワークを介して引き渡します。

次のコマンドを実行するだけで、Angularプロジェクトにサービスワーカーを追加できます。
```ng add @angular/pwa```

**ツール一覧**

- [Angular Service Worker](https://angular.io/guide/service-worker-intro) - サービスワーカーの管理プロセスを自動化することを目指しており、静的な資産をキャッシュするためのサービスワーカーと、[アプリケーションシェルの生成](https://developers.google.com/web/updates/2015/11/app-shell?hl=ja)も行います。
- [Offline Plugin for Webpack](https://github.com/NekR/offline-plugin) - AppCacheにフォールバックしてサービスワーカーのサポートを追加するWebpack用のプラグイン。

**参考文献**

- ["オフラインクックブック"](https://jakearchibald.com/2014/offline-cookbook/)
- ["サービスワーカーを始めよう"](https://angular.io/guide/service-worker-getting-started)

## <a name="runtime-optimizations"></a>実行時の最適化

このセクションには、毎秒60フレーム（fps）のスムーズなユーザーエクスペリエンスを提供するために利用できる実践的な方法が含まれています。

### <a name="use-enableprodmode"></a> `enableProdMode`を使う

開発モードのAngularは、変更検出を実行した時のバインディングが追加の変更が加えられないことを確認するために、いくつかの追加チェックを実行します。 これにより、フレームワークは単方向データフローに従ったことを保証します。

本番では開発モードを無効化する必要があるため、忘れずに`enableProdMode`を実行するようにしてください。

```typescript
import { enableProdMode } from '@angular/core';

if (ENV === 'production') {
  enableProdMode();
}
```

### <a name="ahead-of-time-compilation"></a>Ahead-of-Time コンパイル

AoTは、ツリーシェイキングを実行してより効率的なバンドリングを実現するだけでなく、アプリケーション実行時のパフォーマンスを向上させます。
AoTではない時に実行されているのはJust-in-Timeコンパイル（JiT）です。
つまり、ビルドプロセスの一部としてコンパイルを実行することで、アプリケーションのレンダリングに必要な計算量を減らすことができます。

**ツール一覧**

- [angular2-seed](https://github.com/mgechev/angular2-seed) - AoTコンパイルのサポートを含むスタータープロジェクト。
- [angular-cli](https://cli.angular.io) `ng serve --prod`コマンドを使う

**参考文献**

- ["AngularのAhead-of-Timeコンパイル"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)

### <a name="web-workers"></a>Webワーカー

一般的なシングルページアプリケーション(SPA)の問題は、通常のコードがシングルスレッドで動作することにあります。
つまり、60fpsのスムーズなユーザーエクスペリエンスを実現したい場合は、個々のフレームの間隔である最大**16ms**の間に処理を完了しなければならず、出来ない場合ユーザーエクスペリエンスは悪くなってしまうでしょう。

巨大なコンポーネントツリーを使用した複雑なアプリケーションでは、変更検出で毎秒数百万のチェックをしているので、フレームを減らし始めるのは難しくありません。
DOMアーキテクチャーはAngularのプラットフォームにより切り離されているため、Webワーカーでアプリケーション全体（変更検出を含む）を実行し、レンダリングのみをメインのUIスレッドに任せることができます。

**ツール一覧**

- コアチームによってサポートされているウェブワーカーでアプリケーションを実行できるようにするモジュールです。 使い方はこちらを[参照](https://github.com/angular/angular/tree/master/modules/playground/src/web_workers)してください。
- [Webpack Web Worker Loader](https://github.com/webpack/worker-loader) -webpack用のウェブワーカーローダー

**参考文献**

- ["レスポンシブアプリのためにWebワーカーを使う"](https://www.youtube.com/watch?v=Kz_zKXiNGSE)

### <a name="server-side-rendering"></a>サーバーサイドレンダリング

従来からあるSPAの大きな問題として、最初のレンダリングに必要なJavaScriptを読み込み終わるまで画面の表示ができませんでした。 これは2つの大きな問題を引き起こします:

- すべての検索エンジンがそのページ内のJavaScriptを実行しているわけではないので、アプリケーションの動的コンテンツに適切にインデックスを付けることはできません。
- ページ内のJavaScriptをダウンロード、解析、実行されるまで、ユーザーには空白/ロード中の画面しか表示されず、ユーザーエクスペリエンスが低下します。

サーバーサイドレンダリングは、要求されたページをサーバー上で事前にレンダリングし、最初のページの読み込み時に描画が完了したマークアップを提供することによってこの問題を解決します。

**ツール一覧**

- [Angular Universal](https://github.com/angular/angular/tree/master/packages/platform-server) - Angularに対する Universal (isomorphic) JavaScriptをサポートします。
- [Preboot](https://github.com/angular/preboot) - サーバーで生成されたWebビューからクライアントで生成されたWebビューへの状態（イベント、フォーカス、データ）の移行を管理するのに役立つライブラリ。

**参考文献**

- ["Angularのサーバーレンダリング"](https://www.youtube.com/watch?v=0wvZ7gakqV4)
- ["Angularのユニバーサルパターン"](https://www.youtube.com/watch?v=TCj_oC3m6_U)

### <a name="change-detection"></a>変更検出

非同期イベントが発生する毎に、Angularはコンポーネントツリー全体の変更検出を行います。
変更検出するコードは[インラインキャッシュ](http://mrale.ph/blog/2012/06/03/explaining-js-vms-in-js-inline-caches.html)用に最適化されていますが、複雑なアプリケーションでは重い処理になることがあります。
変更検出のパフォーマンスを向上させる方法は、直近の動作で変更のされないサブツリーに対しては実行しないことです。

#### <a name="changedetectionstrategyonpush"></a>`ChangeDetectionStrategy.OnPush`

`OnPush`変更検出戦略により、コンポーネントツリーのサブツリーに対する変更検出メカニズムを無効にすることができます。 任意のコンポーネントに対する変更検出方法を`ChangeDetectionStrategy.OnPush`と設定することで、コンポーネントが異なる入力を受け取ったときにのみ変更検出を**実行します**。 Angularは参照によって以前の入力と比較した場合に入力が異なると見なし、参照チェックの結果は「false」になります。 不変のデータ構造と組み合わせると、`OnPush`はこのような「純粋なコンポーネント」に大きなパフォーマンス改善をもたらすことができます。

**Resources**

- ["Angularの変更検出"](https://vsavkin.com/change-detection-in-angular-2-4f216b855d4c)
- ["Angularの変更検出について知るべき一通りのこと"](https://blog.angularindepth.com/everything-you-need-to-know-about-change-detection-in-angular-8006c51d206f)

#### <a name="detaching-the-change-detector"></a>Change Detectorの切り離し

カスタム変化検出メカニズムを実装するもう一つの方法として、 特定のコンポーネントの変化検出の機能(CD)の`切り離し`と`再接続`を行うことができます。AngularのCDが一度`切り離される`と、コンポーネントサブツリー全体のチェックは実行されません。

この方法は一般的に、ユーザーの操作や外部サービスとのやり取りによって、必要以上の変更検出が行われる場合に使用します。
この場合、必要な変更検出を実行する必要がある場合にだけ変更検出機能を再接続することを検討してください。

#### <a name="run-outside-angular"></a>Run outside Angular

Angularの変化検出メカニズムは[zone.js](https://github.com/angular/zone.js)によって実行されます。
Zone.jsのモンキーパッチは、ブラウザ内のすべての非同期APIにパッチを適用し、非同期コールバックの実行終了時に変更の検出が引き起します。
**レアケースとして**、変更検出メカニズムを実行したくないために、Angular Zoneのコンテキスト外で特定のコードを実行したい場合があります。
そんな時は、`NgZone`インスタンスのメソッド`runOutsideAngular`を使うことができます。

**例**

以下の小さなコードサンプルで、この方法を使ったコンポーネントの具体例を見ることができます。
`#incrementPoints`メソッドが呼ばれると、コンポーネントは（基本的に）10ms毎に`#points`プロパティの増加を始めていきます。
値の増加はアニメーションのような錯覚をさせるでしょう。
この時に、10msごとにコンポーネントツリー全体の変更検出メカニズムを起動したくないので、Angular Zoneのコンテキスト外で `#incrementPoints`を実行してDOMを手動で更新することができます。（`points` setter を参照）

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

**警告**: 正確に実行されないとDOMの状態に矛盾が生じる可能性があります、**自分が何をしているか確実に理解できるときだけ**この方法を使用してください。
また、上記のコードはウェブワーカーでは実行できません。
ウェブワーカー互換にするためには、Angularのレンダラを使用して表示するラベルの値を設定する必要があります。

### <a name="use-pure-pipes"></a>純粋なpipeを使う

`@Pipe`デコレータは引数に以下のような形式のオブジェクトリテラルを受け取れます。

```typescript
interface PipeMetadata {
  name: string;
  pure: boolean;
}
```

pureフラグは、パイプがどのグローバルの状態にも依存してなく、副作用を引き起こさないことを示します。
つまり、同じ入力で呼び出された場合、どんな時もパイプは同じ出力を返します。
こうすることでAngularはパイプが呼び出されたすべての入力パラメータの出力をキャッシュすることができ、それぞれの評価時に再計算するせず再利用を可能にしています。

`pure`プロパティの初期値は`true`です。

### <a name="ngfor-directive"></a>`*ngFor` ディレクティブ

`*ngFor`ディレクティブはコレクション(データやオブジェクトなどをまとめて格納するデータ構造)を表示するために使われます。

#### <a name="use-trackby-option"></a>`trackBy` オプションを使う

基本的なふるまいとして`*ngFor`は参照によってオブジェクトの一意性を識別します。

つまり、開発者がアイテムのコンテンツ更新中にオブジェクトへの参照を破棄した場合、Angularは古いオブジェクトが削除されて別のオブジェクトが生成されたと判断します。
これは、一覧にある古いDOMノードを破棄し、その場所に新しいDOMノードを追加するといった影響として現れます。

開発者は、オブジェクトの一意性を識別するためにAngularへヒントを教えることができます: `*ngFor`ディレクティブの`trackBy`オプションへカスタムトラッキング関数を設定します。
トラッキング関数には2つの引数があります: `index`と`item`です。
Angularは、トラッキング関数から返された値を使用してアイテムの識別情報を追跡します。
固有のキーとして特定のレコードのIDを使用することはよくある例です。

**例**

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
      id: 3849, // "id"フィールドに注意してください、"trackById"関数でそれを参照します
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

#### <a name="minimize-dom-elements"></a>DOM要素を小さくする

DOMのレンダリングは、要素をUIに追加する時に最もコストのかかる操作です。
よくある操作は、DOMに要素を挿入しスタイルを適用することによって起こります。
`*ngFor`で大量の要素を描画する時、ブラウザ（特に古いもの）は遅くなり、すべての要素の表示に多くの時間が必要となってしまうかもしれません。
そしてこれはAngularに限った話ではありません。

描画時間を短くするために、次のことを試してください:

- [CDK](https://material.angular.io/cdk/scrolling/overview)か[ngx-virtual-scroller](https://github.com/rintoj/ngx-virtual-scroller)を使い仮想スクロールを適用します。
- テンプレートの `*ngFor`セクションで表示されるDOM要素の量を減らす。 不要または未使用のDOM要素はテンプレートを幾度となく拡張することによって発生します。その構造を再考すれば、もっとシンプルにすることができるでしょう。
- 可能であれば[`ng-container`](https://angular.io/guide/structural-directives#ngcontainer)使ってください。

**参考文献**

- ["NgForディレクティブ"](https://angular.io/docs/ts/latest/api/common/index/NgFor-directive.html) - `*ngFor`の公式ドキュメント
- ["Angular - trackByでパフォーマンスを向上させる"](https://netbasal.com/angular-2-improve-performance-with-trackby-cc147b5104e5) - 取り組みについてのgif動画が見れます
- [Component Dev Kit (CDK) Virtual Scrolling](https://material.angular.io/cdk/scrolling/overview) - API description
- [ngx-virtual-scroller](https://github.com/rintoj/ngx-virtual-scroller) - 仮想**無限**リストをを見ることが出来ます

### <a name="optimize-template-expressions"></a>テンプレート中の式を最適化する

Angularは全ての変更検知サイクルを実行した後、テンプレートの式を評価します。
変更検知サイクルは、promiseの解決、httpの結果、タイマーイベント、キー操作、マウスの動きなどの様々な非同期となる動作によって引き起こされます。

式はすぐに終了する必要があります。そうしないと、特に遅いデバイスではユーザーエクスペリエンスが悪化する可能性があります。
計算にコストがかかる場合は値をキャッシュすることを検討してください。

**参考文献**

- [quick-execution](https://angular.io/guide/template-syntax#quick-execution) - テンプレート式の公式文書
- [Increasing Performance - more than a pipe dream](https://youtu.be/I6ZvpdRM1eQ) - YouTubeのng-conf動画。補間式で関数の代わりにパイプを利用する

# <a name="conclusion"></a>終わりに

これらの実践方法の一覧は、新たな/更新され、時間が経つにつれてダイナミックに進化します。
何かが足りない事に気付いた場合や、慣習のいずれかを改善できると思えた場合は、躊躇わずissuesやPRをしてください。
詳細については、次の「[貢献する](#contributing)」セクションをご覧ください。

# <a name="contributing"></a>貢献する

何かのミス、中途半端なもの、または間違いに気付いた場合はプルリクエストをいただければ幸いです。
文書に含まれていない慣習についての議論は[issueを開いてください](https://github.com/mgechev/angular2-performance-checklist/issues)。

# License

MIT
