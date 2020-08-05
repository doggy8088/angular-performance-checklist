<h1 id="angular-2-performance-checklist">Angular 效能檢查清單</h1>

<img src="./assets/flash.png" width="1000">

- [簡體中文](./README.zh-CN.md)
- [Русский](./README.ru-RU.md)
- [Português](./README.pt-BR.md)
- [Español](./README.es-ES.md)
- [日本語](./README.ja-JP.md)

<h2 id="introduction">簡介</h2>

這份清單涵蓋了許多提高 Angular 應用程式效能的最佳實踐。本清單涵蓋了不同的主題，其中包括伺服器端渲染，預先渲染和應用程式封裝，以及執行時期的效能和變更偵測。

這份清單分為兩個主要部分：

-網路效能 - 列出一些能夠改善整體應用程式的載入時間，包括了網路延遲和佔用頻寬減少的最佳實踐。
-執行效能 - 改善我們應用程式的執行時效能的最佳實踐。包括了與變更偵測和渲染相關的最佳化。

其中，部分實踐會同時涵蓋這兩個部分，因此可能會有些許交集。然而，在提到使用情境與實踐方式的時候，也會特別說明兩者之間的差異。
許多小節會列出一些最佳實踐的相關工具，用來協助我們更有效率將開發流程自動化，提高我們的開發效率。

注意：大多數的實踐都適用於 HTTP/1.1 與 HTTP/2 版本。如果有特定實踐只適用於特定 HTTP 版本，也會在文章裡特別說明。

<h2 id="table-of-content">目錄</h2>

- [Angular 效能檢查清單](#angular-2-performance-checklist)
  - [簡介](#introduction)
  - [目錄](#table-of-content)
  - [網路效能](#network-performance)
    - [封裝](#bundling)
    - [最小化與刪除無用程式碼](#minification-and-dead-code-elimination)
    - [刪除範本中多餘空白](#remove-template-whitespace)
    - [搖樹最佳化](#tree-shaking)
    - [搖樹最佳化的服務提供者](#tree-shakeable-providers)
    - [Ahead-of-Time (AoT) 編譯](#ahead-of-time-aot-compilation)
    - [壓縮](#compression)
    - [資源預先載入](#pre-fetching-resources)
    - [資源延遲載入](#lazy-loading-of-resources)
    - [不要延遲載入預設模組](#dont-lazy-load-the-default-route)
    - [快取](#caching)
    - [使用程式殼層](#use-application-shell)
    - [使用 Service Workers](#use-service-workers)
  - [執行效能最佳化](#runtime-optimizations)
    - [使用 `enableProdMode`](#use-enableprodmode)
    - [Ahead-of-Time 編譯](#ahead-of-time-compilation)
    - [Web Workers](#web-workers)
    - [伺服器端渲染](#server-side-rendering)
    - [變更偵測](#change-detection)
      - [`ChangeDetectionStrategy.OnPush`(譯者注: `變更偵測的 onPush 策略`)](#changedetectionstrategyonpush)
      - [關閉變更偵測](#detaching-the-change-detector)
      - [執行於變更偵測區域外](#run-outside-angular)
    - [使用純管道(pure pipe)](#use-pure-pipes)
    - [`*ngFor` 指令](#ngfor-directive)
      - [使用 `trackBy` 選項](#use-trackby-option)
      - [最小化 DOM 元素](#minimize-dom-elements)
    - [最佳化範本表示式](#optimize-template-expressions)
- [結論](#conclusion)
- [貢獻](#contributing)
- [協議](#license)

<h2 id="network-performance">網路效能</h2>

本節中的一些工具目前還在開發階段，可能會發生更改。Angular 核心團隊正在儘可能多地自動化我們的應用程式的建構過程，因此許多最佳化和新特性都將接踵而至。

<h3 id="bundling">封裝</h3>

封裝是一種標準實踐，旨在減少瀏覽器對於資源的多次請求，降低 http 請求數。本質上，bundler 接收一個入口點列表作為輸入，並產生一個或多個 bundler。這樣，瀏覽器只需執行幾個請求，而不是單獨請求每個資源，就可以獲得整個應用程式。

當你的應用程式增長到一定的量級，把所有資源封裝到一個 bundle 可能會適得其反，這時就需要使用 Webpack 對專案進行依賴拆分。

**由於具有伺服器推送功能，HTTP/2 不會涉及 HTTP 請求。參照[伺服器推送功能](https://http2.github.io/faq/#whats-the-benefit-of-server-push)**

**工具**

幫助我們更高效的建構應用程式:

- [Webpack](https://webpack.js.org) - 提供高效的建構，封裝及[搖樹最佳化](#tree-shaking).
- [Webpack Code Splitting](https://webpack.js.org/guides/code-splitting/) - 將程式碼分割.
- [Webpack & http2](https://medium.com/webpack/webpack-http-2-7083ec3f3ce6#.46idrz8kb) - 與 http2 分離.
- [Rollup](https://github.com/rollup/rollup) -利用 ES2015 模組的靜態特性，透過執行有效的搖樹最佳化來進行建構
- [Google Closure Compiler](https://github.com/google/closure-compiler) - 提供大量最佳化和建構支援. 最初使用 JAVA 進行編寫的, 最近也有一個 [JavaScript 版本](https://www.npmjs.com/package/google-closure-compiler-js)。
- [SystemJS Builder](https://github.com/systemjs/builder) - 為最小依賴系統模組樹提供單檔案建構
- [Browserify](http://browserify.org/) （譯者注：brwoser 透過封裝相依性，讓你在瀏覽器具備 `require('modules')` 的能力）
- [ngx-build-modern](https://github.com/manfredsteyer/ngx-build-plus/tree/master/ngx-build-modern) - Angular-CLI 的外掛， 把應用建構成兩個版本:
  1. 支援 ES2015 的現代瀏覽器和透過 pollyfill 實現新特性的瀏覽器，構建出較小的封裝
  2. 使用不同 polyfill 和編譯器目標的舊版本(預設)

**資源**

- ["製作一個 Angular 應用程式"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["使用 Google Closure Compiler 讓 Angular 程式縮小 2.5 倍"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)



<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h3 id="minification-and-dead-code-elimination">最小化與刪除無用程式碼</h3>

這些實踐都會讓我們的應用程式更輕巧並減少頻寬的負擔。

**工具**

- [Uglify](https://github.com/mishoo/UglifyJS) 程式碼壓縮，如管理變數、刪除註釋和空白、消除死碼等，完全用 javascript 編寫，所有流行的程式碼執行程式（IDE）都有外掛。
- [Google Closure Compiler](https://github.com/google/closure-compiler) - 提供大量最佳化和建構支援. 最初使用 JAVA 進行編寫的, 最近也有一個 [JavaScript 版本](https://www.npmjs.com/package/google-closure-compiler-js)。

*Note:* [GCC](https://github.com/google/closure-compiler) 尚未支援 `export *`，但這是建構Angular應用程式中重度依賴的語法(`barrel`).

**資源**

- ["Building an Angular Application for Production"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)



<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h3 id="remove-template-whitespace">刪除範本中多餘空白</h3>

雖然我們看不到空白字元（與 `\s` RegExp 匹配的字元），但它仍然由透過網路傳輸的位元組表示。如果我們將範本中的空白減少到最小，我們將能夠分別進一步減少 AOT 程式碼的包大小。

好在，"componentmetadata" 介面提供屬性 "preserveWhitespaces"，預設值為 "false"，因為刪除空白可能會影響 DOM 佈局。如果我們將屬性設定為 "false" ，那麼 Angular 將修剪不必要的空白，從而進一步減小建構體積的大小。

- [preserveWhitespaces in the Angular docs](https://angular.io/api/core/Component#preserveWhitespaces)



<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h3 id="tree-shaking">搖樹最佳化</h3>

對於我們的應用程式的最終版本，我們通常不使用 Angular 或任何第三方函式庫提供的整個 Library，甚至是我們自己寫的所有程式碼。由於 ES2015 模組的靜態特性，我們能夠剔除應用程式中未參考的程式碼。

**範例**

```javascript
// foo.js
export foo = () => 'foo';
export bar = () => 'bar';

// app.js
import { foo } from './foo';
console.log(foo());
```

一旦我們對 app.js 進行了搖樹最佳化，那麼最終封裝的結果為：

```javascript
let foo = () => 'foo';
console.log(foo());
```

無用的方法 `bar` 最終沒有被封裝

**工具**

- [Webpack](https://webpack.js.org) - 透過執行[搖樹最佳化](#tree-shaking)， 一旦應用建構完成, 封裝中不會包含無用程式碼，因此透過 uglify 可以安全地刪除無用程式碼
- [Rollup](https://github.com/rollup/rollup) - 利用 ES2015 模組的靜態特性，透過執行有效的搖樹最佳化來進行建構
- [Google Closure Compiler](https://github.com/google/closure-compiler) - 提供大量最佳化和建構支援. 最初使用 JAVA 進行編寫的, 最近也有一個 [JavaScript 版本](https://www.npmjs.com/package/google-closure-compiler-js)。



**資源**

- ["製作一個 Angular 應用程式”](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["使用 Google Closure Compiler 讓 Angular 程式縮小 2.5 倍”](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)
- ["使用 RxJS 運算子"](https://github.com/ReactiveX/rxjs/blob/master/doc/pipeable-operators.md)



<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h3 id="tree-shakeable-providers">搖樹最佳化的服務提供者</h3>

自從 Agnular 釋出 V6 版本以來，Angular 團隊提供了一個可以將 services Tree-shaking 優化的新特性，這意味著，除非其他服務或元件正在使用該服務，否則該的服務將不會包含在最終捆綁包中，這可以透過從包中刪除未使用的程式碼來幫助減少包的大小。


你可以在 Angular 應用中的 Services 裡使用 `@injectable()` 的 `providedIn` 方法去定義服務應該在哪裡初始化，從而更好的Tree-shaking 優化。
然後，您應該將其從 `ngModule` 宣告的 `providers` 屬性以及其 import 語句中刪除，如下所示:

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

如果沒有在任何元件/服務中注入 `myService`，那麼它將不會出現在最終的封裝檔案中。

**資源**

- [Angular 服務提供者](https://angular.io/guide/providers)



<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h3 id="ahead-of-time-aot-compilation">Ahead-of-Time (AoT) 編譯</h3>

對於可用的建構工具（如 [GCC](https://github.com/google/closure-compiler) 、 [Rollup](https://github.com/rollup/rollup) 等）來說，用內建的功能去解析 Angular 元件的 HTML-like 範本是一項艱鉅的挑戰。這使得它們的Tree-shaking 效率降低，因為它們不確定範本中參考了哪些指令。AOT 編譯器透過 `ES2015` 模組匯入將 Angular HTML 類別範本傳輸到 JavaScript 或 TypeScript。這樣，我們就能夠在繫結期間有效地進行Tree-shaking ，並刪除由 Angular、或第三方函式庫或是我們自己定義的所有未使用的指令。

**資源**

- ["Angular Ahead-of-Time 編譯"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)



<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h3 id="compression">壓縮</h3>


壓縮響應是減少網路負載的標準實踐。透過指定頭 `Accept-Encoding` 的值，瀏覽器提示伺服器哪些壓縮演算法在客戶端上可用。另一方面，伺服器在回應時，會在 `Content-Encoding` 中告訴瀏覽器選擇了哪個演算法來壓縮。

**工具**

這些工具不是 Angular 特有的，完全依賴於我們的應用/服務。典型的壓縮演算法有：

- deflate - 一種資料壓縮演算法和相關檔案格式，使用 LZ77 演算法和哈夫曼編碼的組合
- [brotli](https://github.com/google/brotli) -一種通用無失真壓縮演算法，它使用 LZ77 演算法的現代變種、哈夫曼編碼和二階上下文建模相結合來壓縮資料，壓縮比與當前可用的最佳通用壓縮方法相當。它在速度上與 deflate 相似，但提供更密集的壓縮。

**資源**

- ["Better than Gzip Compression with Brotli"](https://hacks.mozilla.org/2015/11/better-than-gzip-compression-with-brotli/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)



<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h3 id="pre-fetching-resources">資源預先載入</h3>

資源預先載入是提高使用者體驗的好方法。我們可以預先獲取資源（影象、樣式、[延遲載入模組](#lazy-loading-of-resources) 等）或資料。有不同的預先載入策略，但大多數都取決於應用程式的具體情況。



<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h3 id="lazy-loading-of-resources">資源延遲載入</h3>

假設目標應用程式有一個龐大的程式碼函式庫，並且有數百個依賴，上面列出的實踐可能無法幫助我們將 bundle 包體積減少到一個合理的大小（合理的可能是 100k 或 2M，同樣，完全取決於業務指標）。

在這種情況下，一個好的解決方案可能是延遲載入應用程式的一些模組。舉個例子，假設我們正在建構一個電子商務系統。在這種情況下，我們可能希望獨立於面向使用者的 UI 載入 admin 面板。一旦管理員必須新增一個新產品，我們將希望提供所需的使用者介面。這可能只是"新增產品頁面"或整個 admin 面板，具體取決於我們的業務需求。

**工具**

- [Webpack](https://github.com/webpack/webpack) - 非同步載入模組
- [ngx-quicklink](https://github.com/mgechev/ngx-quicklink) - 自動延遲載入螢幕上所有連結所參考的 modules



<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h3 id="dont-lazy-load-the-default-route">不要延遲載入預設模組</h3>

我們假設有如下的路由配置:

```ts
// Bad practice
const routes: Routes = [
  { path: '', redirectTo: '/dashboard', pathMatch: 'full' },
  { path: 'dashboard',  loadChildren: () => import('./dashboard.module').then(mod => mod.DashboardModule) },
  { path: 'heroes', loadChildren: () => import('./heroes.module').then(mod => mod.HeroesModule) }
];
```

第一次使用者開啟的 url 為： https://example.com/ ，將自動重新導向到 `/dashboard` ，與此同時將觸發 `dashedboard.module`的延遲載入。實際上 Angular 在渲染啟動元件所在模組時，將下載 `dasheboard.module` 的所有檔案和依賴項，接著，這些檔案需要 JavaScript 引擎去解析。
在初始化頁面中觸發外部 HTTP

請求並執行大量的計算是一種不好的實踐方式，這將大大降低初始路由頁的載入速度。考慮將預設模組宣告為非惰性載入。



<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h3 id="caching">快取</h3>

快取是另一種加快程式執行速度的實踐方式：請求了一個資源，很有可能不久之後又去請求一次。

我們通常使用自訂的快取機制，對於靜態資源我們直接使用瀏覽器快取或者 Service Worker ：[CacheStorage API](https://developer.mozilla.org/en-US/docs/Web/API/Cache).



<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h3 id="use-application-shell">使用程式殼層</h3>

要使應用程式的效能更快，請使用[Application Shell](https://developers.google.com/web/updates/2015/11/app-shell).

使用應用程式指令是我們向用戶展示的最小使用者介面，以指示他們應用程式將很快交付。為了動態產生應用程式指令，我們可以將 Angular Universal 與自訂指令一起使用，這些指令根據所使用的渲染平臺有條件地顯示元素（例子：除了`platform-server` 之外都隱藏掉）。

**工具**

- [Angular Service Worker](https://angular.io/guide/service-worker-intro) - 自動化管理 Service Workers ，以及靜態資源的快取 [generating application shell](https://developers.google.com/web/updates/2015/11/app-shell?hl=en).
- [Angular Universal](https://github.com/angular/angular/tree/master/packages/platform-server) - 為 Angular 提供同構支援

**資源**

- ["Instant Loading Web Apps with an Application Shell Architecture"](https://developers.google.com/web/updates/2015/11/app-shell)



<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h3 id="use-service-workers">使用 Service Workers</h3>

我們可以認為 Service Worker 是 客戶端本地 對 HTTP 的代理。所有請求從客戶端發出第一時間都被 Service Worker 攔截，也可以直接透過 HTTP 去處理這些請求。

你可以透過下列指令新增 Service Worker 到你的 Angular 專案內
``` ng add @angular/pwa ```

**工具**

- [Angular Service Worker](https://angular.io/guide/service-worker-intro) - 自動化管理 Service Workers ，以及靜態資源的快取 [generating application shell](https://developers.google.com/web/updates/2015/11/app-shell?hl=en).
- [Webpack 離線外掛](https://github.com/NekR/offline-plugin) - Webpack Service 提供的 Worker 外掛

**資源**

- ["離線指南"](https://jakearchibald.com/2014/offline-cookbook/)
- ["開始使用Service Workers"](https://angular.io/guide/service-worker-getting-started)


<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h2 id="runtime-optimizations">執行效能最佳化</h2>

這個部分涵蓋了一些瀏覽器幀率優化的最佳實踐（最佳使用者體驗 60FPS / S）

<h3 id="use-enableprodmode">使用 `enableProdMode`</h3>

在開發模式下，為了驗證執行變更偵測不會導致對任何繫結進行任何其他更改，Angular 會執行一些額外檢查。這樣，框架就可以確保遵循單向資料流。

要在正式模式下禁用這些變化，記得呼叫 `enableProdMode`：

```typescript
import { enableProdMode } from '@angular/core';

if (ENV === 'production') {
  enableProdMode();
}
```


<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h3 id="ahead-of-time-compilation">Ahead-of-Time 編譯</h3>

AoT 不僅有助於在Tree-shaking 階段更有效率的打包，而且會提升我們應用的執行時期效能。AoT 的另一種選擇是執行時執行的即時編譯（JIT），因此，我們可以透過將編譯作為建構過程的一部分執行來減少呈現應用程式所需的計算量。

**工具**

- [angular2-seed](https://github.com/mgechev/angular2-seed) - 一個包含 Aot 編譯的 starter 專案
- [angular-cli](https://cli.angular.io) 使用 `ng serve --prod`

**資源**

- ["Angular 的 Aot"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)



<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h3 id="web-workers">Web Workers</h3>

在典型的單頁應用中，我們的程式碼通常跑在一個執行緒裡，也就是說如果我們想要使用者體驗到 60FPS，我們最多隻有 16ms 的時間在各個畫面幀之間去執行我們的程式碼。

在具備巨大元件樹的應用中，變更偵測通常每秒鐘要執行上百萬次，因此很容易讓頁面的效能下降，畫面更新率變低。還好 Angular 將 Dom 結構進行了剝離，我們可以在 Web Woker 中執行整個應用程式（包括變更檢測），只讓主 UI 執行緒只負責渲染。

**工具**

- Webpack 允許我們在 Web Worker 中執行應用程式的模組。[範例](https://github.com/angular/angular/tree/master/modules/playground/src/web_workers).
- [Webpack Web Worker Loader](https://github.com/webpack/worker-loader) - A Web Worker Loader for webpack.

**資源**

- ["為更多 APP 使用 Web Workers"](https://www.youtube.com/watch?v=Kz_zKXiNGSE)



<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h3 id="server-side-rendering">伺服器端渲染</h3>


SPA 有一個大問題：在初始渲染頁面所需的整個 JavaScript 載入之前，無法渲染。這導致了兩個大問題：

- 並非所有的搜尋引擎都在執行與頁面相關聯的 JavaScript，因此它們無法正確爬到動態應用程式的內容。

- 糟糕的使用者體驗，因為在下載、分析和執行與頁面相關的 JavaScript 之前，使用者只會看到頁面處於空白/載入狀態。

伺服器端渲染透過在伺服器上預先渲染使用者請求的頁面，並在初始畫面載入的期間提供渲染頁面的標記來解決這個問題。

**工具**

- [Angular Universal](https://github.com/angular/angular/tree/master/packages/platform-server) - 為 Angular 同構提供支援.
- [Preboot](https://github.com/angular/preboot) - 用於幫助管理狀態（即事件、焦點、資料）從伺服器產生的 Web 檢視到客戶端產生的 Web 檢視的轉換的函式庫。
- [Scully](https://github.com/scullyio/scully) - Angular 專案的靜態頁面產生器，用來實現 JAMStack 的目標。

**資源**

- ["Angular 伺服器端渲染"](https://www.youtube.com/watch?v=0wvZ7gakqV4)
- ["Angular 通用模式"](https://www.youtube.com/watch?v=TCj_oC3m6_U)
- ["透過 Angular & Scully 產生靜態網頁"](https://www.youtube.com/watch?v=ugTx-14jRrI)



<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h3 id="change-detection">變更偵測</h3>


在每個非同步事件上，Angular 對整個元件樹執行變更檢測。雖然檢測變更的程式碼針對[內聯快取](http://mrale.ph/blog/2012/06/03/explaining-js-vms-in-js-inline-caches.html)進行了優化，但在複雜的應用程式中，這仍然是一個大量複雜的計算。提高變更偵測效能的一種方法是不對子樹執行變更偵測，子樹不應根據最近的操作進行變更。

<h4 id="changedetectionstrategyonpush">`ChangeDetectionStrategy.OnPush`(譯者注: `變更偵測的 onPush 策略`)</h4>

`OnPush` 禁用元件樹子樹的更改檢測機制. 將單個元件的變更策略設定為 `ChangeDetectionStrategy.OnPush`, 將使更改檢測 **僅** 在元件收到不同輸入時執行。當 Angular 將輸入與以前的參考輸入進行比較時，它會將輸入視為不同的輸入，並且參考檢查的結果是 `false`。結合[不可變資料結構]（https://facebook.github.io/immutable-js/）`onpush`可以為 `單純` 的元件帶來巨大的效能提升。

**資源**

- ["Angular 變更偵測機制"](https://vsavkin.com/change-detection-in-angular-2-4f216b855d4c)
- ["Angular 中你應該知道的所有關於變更偵測的原理"](https://blog.angularindepth.com/everything-you-need-to-know-about-change-detection-in-angular-8006c51d206f)

<h4 id="detaching-the-change-detector">關閉變更偵測</h4>

另外的一種方法，透過 `detach`ing 和 `reattach`ing 實現自訂變更偵測機制，從而來控制組件的變更偵測開關。一旦執行 `detach` ，Angular 變更偵測時將不會對整個元件樹進行檢查。

當用戶操作或與外部服務的互動比需要的更頻繁地觸發變更偵測時，通常使用此實踐。在這種情況下，我們可能需要考慮分離變更偵測器，並僅在需要執行變更偵測時透過重新 `reattach` 來開啟變更偵測的**開關**。

<h4 id="run-outside-angular">執行於變更偵測區域外</h4>

Angular 的變更偵測機制是透過[zone.js](https://github.com/angular/zone.js) 實現的。
Zone.js 透過猴子補丁的方法代理了所有瀏覽器的非同步 APIs，並在他們執行的時候以非同步回呼(Callback)的形式觸發變更偵測。
在**極少情況**下，我們希望程式碼在 Angular 之外的區域執行，所以不需要變更偵測機制進行檢查。這種情況下我們可以在元件中注入 `zone: NgZone` 服務，並且呼叫這個實例的 `runOutsizeAngular` 方法即可。

**例子**

在下面的程式碼片段中，您可以看到使用此實踐的元件的範例。當呼叫 `_incrementPoints` 方法時，元件將開始每 10 ms 遞增一次 `_points` 屬性（預設情況下）。遞增會造成動畫的假象。因為在這種情況下，我們不希望觸發整個元件樹的更改檢測機制，所以每 10 ms，我們可以在 Angular 區域的上下文之外執行 `incrementpoints`，並手動更新 DOM（請參見 `points` setter 訪問器）。

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

**警告**: **只有當您確定要做什麼時，才能非常小心地使用這個實踐**，因為如果使用不當，它可能導致 DOM 的狀態不一致而引發錯誤。還要注意，上面的程式碼不會在 Web Worker 中執行。為了使它與 Web Worker 相容，需要使用 Angular 的 `renderer` 設定標籤的值。



<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h3 id="use-pure-pipes">使用純管道(pure pipe)</h3>

作為引數，`@pipe` decorator 接受具有以下格式引數：

```typescript
interface PipeMetadata {
  name: string;
  pure: boolean;
}
```

`pure` 標誌表示管道不依賴於任何全域性狀態，不會產生副作用。這意味著當使用相同的輸入呼叫時，管道將返回相同的輸出。透過這種方式，Angular 可以快取管道呼叫時使用的所有輸入引數的輸出，並重用它們，以便不必在每次計算時重新計算它們。

預設值 `pure` 為 `true`


<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h3 id="ngfor-directive">`*ngFor` 指令</h3>

`*ngFor` 用於渲染集合（譯者注：擁有疊代器的物件）

<h4 id="use-trackby-option">使用 `trackBy` 選項</h4>

預設情況下，`*ngFor` 透過參考物件參考的唯一值來確認物件(譯者注：物件的參考)。
也就是說，當開發人員在更新項目的內容時中斷對物件的參考時，Angular 將其視為刪除舊物件和新增新物件。這會破壞列表中的舊 DOM 節點並在其位置新增新的 DOM 節點。

開發者可以提供一個關於 Angular 如何識別物件唯一性的提示：自訂追蹤函式作為 `*ngfor` 指令的 `trackby` 選項。追蹤函式接受兩個引數：`index` 和 `item`。Angular 使用追蹤函式返回的值來追蹤每個疊代的物件。使用函式產生的值作為唯一鍵（key）。


**例子**

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

<h4 id="minimize-dom-elements">最小化 DOM 元素</h4>

在向 UI 新增元素時，操作 DOM 元素是很昂貴的操作。主要工作通常是將元素插入 DOM 並應用樣式。如果 `*ngfor` 呈現很多元素，瀏覽器（尤其是舊的瀏覽器）可能會執行變得緩慢，因為需要更多的時間來完成所有元素的呈現。不僅僅是使用 Angular 會有這樣的現象。

減少渲染時間，可以參考：
- 虛擬滾動 [CDK](https://material.angular.io/cdk/scrolling/overview) 或 [ngx-virtual-scroller](https://github.com/rintoj/ngx-virtual-scroller)
- 減少在範本中 `*ngfor` 遍歷的 DOM 元素的數量。通常不需要/未使用的 DOM 元素是由一次又一次地擴充套件範本引起的。重新考慮它的結構可能會有幫助。
- 儘可能的使用 [`ng-container`](https://angular.io/guide/structural-directives#ngcontainer)

**資源**

- ["NgFor 指令"](https://angular.io/docs/ts/latest/api/common/index/NgFor-directive.html) - `*ngFor` 官方文件
- ["Angular — Improve performance with trackBy"](https://netbasal.com/angular-2-improve-performance-with-trackby-cc147b5104e5) - 顯示方法的 GIF 示範
- [Component Dev Kit (CDK) Virtual Scrolling](https://material.angular.io/cdk/scrolling/overview) - API 描述
- [ngx-virtual-scroller](https://github.com/rintoj/ngx-virtual-scroller) -顯示虛擬"無限"列表



<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h3 id="optimize-template-expressions">最佳化範本表示式</h3>

Angular 在每次變更偵測週期之後才去計算範本表示式。變更偵測通常由一些非同步動作去觸發，例如 `promise`, `http返回結果`，`定時器 / 計時器事件` ，`鍵盤和滑鼠事件`等。

表示式應該很快完成，否則使用者體驗可能會被拖走，尤其是在速度較慢的裝置上。當計算代價高昂時，應該考慮快取值。

**資源**

- [快速計算](https://angular.io/guide/template-syntax#quick-execution) - 範本計算-官方文件
- [提高效能-不僅僅是一個白日夢](https://youtu.be/I6ZvpdRM1eQ) - ng-conf 相關視訊. 插值表示式中用管道代替函式



<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>

<h1 id="conclusion">結論</h1>

實踐列表將隨著新的/更新的實踐而動態演變。如果您發現遺漏或認為可以改進任何實踐，請 PR 或 ISSUE。有關更多資訊，請檢視下面的"[貢獻](#contributing)"部分。

<h1 id="contributing">貢獻</h1>

如果您發現短少，不完整或不正確的內容，我們將十分樂於看到您的 PR 。對於檔案中未包含的實踐的討論，請
[開啟一個問題](https://github.com/mgechev/angular2-performance-checklist/issues).

<h1 id="license">協議</h1>

MIT


<p align="right">
  <a href="#table-of-content">回到目錄</a>
</p>