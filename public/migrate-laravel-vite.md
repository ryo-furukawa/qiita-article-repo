---
title: migrate-laravel-vite
tags:
  - 'Laravel'
  - 'Vue'
  - 'Laravel Mix'
  - 'Vite'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
## LaravelのWebpack MixからViteへの移行した話
この記事では、Laravelのプロジェクトにおいて、Webpack MixからViteへの移行プロセスとその結果について詳しく解説します。Laravel 9以降、Viteが標準のビルドツールとして採用されており、この記事ではその移行理由と実際の移行手順、得られた効果について紹介します。
基本的には公式ドキュメント通りに進め、プロジェクト環境によるエラーがあれば解決する形で対応しました。

### 環境
| 機能         | 移行前     | 移行後     |
| ------------ | ---------- | ---------- |
| Laravel      | 9.23.0     | 9.23.0     |
| Vue          | 3.2.37     | 3.2.37     |
| ビルドツール  | Laravel Mix 6.0.0 | Vite 5.4.0 |


### 移行前の課題と結果
- **ビルド時間の長さ**
    - 移行前: `npm run dev` 、`npm run build` の実行時間が約3分。
    - 移行後: `npm run dev` の実行時間が1秒以下、`npm run build` の実行時間が10秒に短縮されました。

- **開発効率の低下**
    - 移行前: Laravel Mixでのビルドに時間がかかり、開発効率が低下していました。
    - 移行後: HMR（ホットモジュールリプレースメント）がより効率的に動作し、差分更新やキャッシュを考慮した迅速な更新が可能になりました。

- **標準ツールの推奨**
    - Laravel 9以降は、Laravel MixからViteへの移行が推奨されており、標準のビルドツールとしてViteの採用が必要です。

## 公式ドキュメント
Laravel9 Vite
[9.x アセットの構築（Vite） Laravel](https://readouble.com/laravel/9.x/ja/vite.html)

Vite切り替え移行ガイド
[vite-plugin/UPGRADE.md at main · laravel/vite-plugin](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-laravel-mix-to-vite)

## 移行対応

### 1. **ViteとLaravelプラグインをインストールする**

viteインストール

`npm install --save-dev vite laravel-vite-plugin`

 Vite プラグインをインストール

`npm install --save-dev @vitejs/plugin-vue`

:::note info
packageインストール時のエラーと解決

私の環境では、Viteインストール時に@types/nodeの依存関係と競合してインストールできないエラーが発生。
<details>
<summary>エラーログと解決詳細</summary>

```
npm error code ERESOLVE
npm error ERESOLVE could not resolve
npm error
npm error While resolving: vite@5.4.0
npm error Found: @types/node@17.0.45
npm error node_modules/@types/node
npm error   @types/node@"*" from @types/body-parser@1.19.2
npm error   node_modules/@types/body-parser
npm error     @types/body-parser@"*" from @types/express@4.17.13
npm error     node_modules/@types/express
npm error       @types/express@"*" from @types/serve-index@1.9.1
npm error       node_modules/@types/serve-index
npm error         @types/serve-index@"^1.9.1" from webpack-dev-server@4.9.3
npm error         node_modules/webpack-dev-server
npm error       2 more (http-proxy-middleware, webpack-dev-server)
npm error   @types/node@"*" from @types/bonjour@3.5.10
npm error   node_modules/@types/bonjour
npm error     @types/bonjour@"^3.5.9" from webpack-dev-server@4.9.3
npm error     node_modules/webpack-dev-server
npm error       webpack-dev-server@"^4.7.3" from laravel-mix@6.0.49
npm error       node_modules/laravel-mix
npm error         dev laravel-mix@"^6.0.0-beta.17" from the root project
npm error   12 more (@types/clean-css, @types/connect, ...)
npm error
npm error Could not resolve dependency:
npm error peerOptional @types/node@"^18.0.0 || >=20.0.0" from vite@5.4.0
npm error node_modules/vite
npm error   vite@"*" from the root project
npm error   peer vite@"^5.0.0" from laravel-vite-plugin@1.0.5
npm error   node_modules/laravel-vite-plugin
npm error     laravel-vite-plugin@"*" from the root project
npm error
npm error Conflicting peer dependency: @types/node@22.3.0
npm error node_modules/@types/node
npm error   peerOptional @types/node@"^18.0.0 || >=20.0.0" from vite@5.4.0
npm error   node_modules/vite
npm error     vite@"*" from the root project
npm error     peer vite@"^5.0.0" from laravel-vite-plugin@1.0.5
npm error     node_modules/laravel-vite-plugin
npm error       laravel-vite-plugin@"*" from the root project
npm error
npm error Fix the upstream dependency conflict, or retry
npm error this command with --force or --legacy-peer-deps
npm error to accept an incorrect (and potentially broken) dependency resolution.
npm error
npm error
npm error For a full report see:
npm error /home/sail/.npm/_logs/2024-08-14T07_53_26_297Z-eresolve-report.txt
npm error A complete log of this run can be found in: /home/sail/.npm/_logs/2024-08-14T07_53_26_297Z-debug-0.log
```

解決策：`@types/node` のバージョンを更新する

`vite` が依存している `@types/node` のバージョンと競合しないように、プロジェクト内の `@types/node` のバージョンを `18.x` 以上に更新してから、再度インストールを行う。

`npm install @types/node@18 --save-dev`

を実行後に再度viteインストールをすると成功
</details>
:::

## 2. Vite 設定ファイルの作成

`touch vite.config.js`
transformAssetUrlsは、`npm run build`で画像パスを読み込む際にエラーが発生していたため追加しました。

```jsx
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import { resolve } from 'path';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
  plugins: [
    vue({
          // This is needed, or else Vite will try to find image paths (which it wont be able to find because this will be called on the web, not directly)
          // For example <img src="/images/logo.png"> will not work without the code below
          template: {
            transformAssetUrls: {
              includeAbsolute: false
            }
          }
        }),
    laravel({
      input: [
        // TypeScript files
        'resources/js/app.ts',
        'resources/admin/js/adminapp.ts',

        // CSS files
        'resources/admin/css/adminapp.css',
        'resources/css/app.css',
        'resources/css/reset.css',
      ],
      refresh: true,
    }),
  ],
  resolve: {
    alias: {
      '/@': resolve(__dirname, 'resources'),
    }
  },
  build: {
    rollupOptions: {
      output: {
        entryFileNames: 'js/[name].[hash].js',
        chunkFileNames: 'js/[name].[hash].js',
        assetFileNames: 'assets/[name].[hash].[ext]',
      },
    },
  },
});

```

## 3. package.jsonのscriptを変更

```json
    "scripts": {
        "dev": "vite",
        "build": "vite build",
        "lint:script": "eslint --ext .ts,.vue --ignore-path .gitignore .",
        "fix:eslint": "eslint --ext .ts,.vue --fix --ignore-path .gitignore ."
    }
```

## 4.  Bladeファイルの更新

Bladeファイル内で、Viteのエントリーポイントを参照するように変更します。Viteは、Laravel Mixのような `mix()` 関数ではなく、`@vite()` ディレクティブを使用します。

`@vite(['resources/js/app.ts', 'resources/css/app.css'])`


```php
// 移行前サンプル
{{-- admin.blade.php --}}
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">

<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>サイトタイトル</title>

    <!-- Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Nunito:wght@400;600;700&display=swap" rel="stylesheet">
    <link href="{{ mix('css/reset.css') }}" rel="stylesheet">
    <link href="{{ mix('admin/css/adminapp.css') }}" rel="stylesheet">
    @if(config('app.env') === 'local')
    <script src="{{ mix('admin/js/adminapp.bundle.js') }}" defer></script>
    @else
    <script src="{{ mix('admin/js/adminapp.production.js') }}" defer></script>
    @endif
</head>

<body>
    <div id="admin-app"></div>
</body>

</html>
```

```php
// 移行後サンプル
{{-- student.blade.php --}}
@vite(['resources/js/app.ts', 'resources/css/app.css'])

{{-- admin.blade.php --}}
@vite(['resources/admin/js/adminapp.ts', 'resources/css/reset.css', 'resources/admin/css/adminapp.css'])

```

## 5. ビルド実行　テスト

:::note info
`npm run dev`実行後に下記のエラーと解決

[ERROR] "laravel-vite-plugin" resolved to an ESM file. ESM file cannot be loaded by `require`. See https://vitejs.dev/guide/troubleshooting.html#this-package-is-esm-only for more details. [plugin externalize-deps]

指定されたドキュメント確認後、下記をpackage.jsonに追加
`"type": "module”`
:::

## 6. ポート解放

`${VITE_PORT:-5173}:${VITE_PORT:-5173}`を追加

```yaml
version: "3"
services:
    laravel.test:
        build:
            context: ./docker/8.0
            dockerfile: Dockerfile
            args:
                WWWGROUP: "${WWWGROUP}"
        image: sail-8.0/app
        extra_hosts:
            - "host.docker.internal:host-gateway"
        ports:
            - "${APP_PORT:-80}:80"
            - "${VITE_PORT:-5173}:${VITE_PORT:-5173}"
```

:::note info
**なぜ5173を追加すると動くのか？**

`5173`ポートを追加すると動作する理由は、Viteが開発サーバーとして動作する際に、デフォルトで使用するポートが `5173` であるためです。具体的な理由は以下の通りです。

<details>
<summary>詳細はこちら</summary>

### Viteの動作とポート `5173`

1. **Viteの開発サーバー**:
    - Viteはフロントエンド開発のための高速なビルドツールであり、開発環境では開発サーバーを立ち上げて、ファイルのホットリロードやリアルタイムでの更新を行います。この開発サーバーは通常、デフォルトでポート `5173` を使用します。
2. **Dockerコンテナ内でのViteサーバー**:
    - Dockerコンテナ内でViteサーバーを動作させる場合、コンテナ内のポート `5173` でViteサーバーが立ち上がりますが、これをホストマシン（ローカルPC）からアクセスするためには、そのポートを外部に公開する必要があります。
3. **`5173`ポートの追加による効果**:
    - `docker-compose.yml` の `ports` セクションで `5173` ポートを公開する設定を行うことで、コンテナ内のViteサーバー（ポート `5173`）に対するリクエストがホストマシンのポート `5173` を通じてアクセスできるようになります。これにより、ホストマシンのブラウザから `http://localhost:5173` にアクセスすることで、Viteが提供するリソース（JS/CSSなど）にアクセスできるようになります。

### 設定が必要な理由

- **ホットリロード機能**: Viteは開発時にファイルの変更を即座に反映させるホットリロード機能を提供します。この機能が有効に動作するためには、ブラウザとViteサーバーが直接通信できるようにする必要があります。ポート `5173` を公開することで、Viteのホットリロード機能が正しく機能します。

### Dockerコンテナ内の動作を理解する

- Dockerコンテナ内で動作しているサーバー（この場合はVite）は、コンテナ内で指定されたポートを使用します。そのポートが外部（ホストマシン）からアクセス可能でなければ、ホスト側からコンテナ内のViteサーバーにアクセスできません。そのため、`docker-compose.yml` でポートのマッピング（例: `5173:5173`）を行う必要があります。

この設定によって、ホストマシンのブラウザから `http://localhost:5173` でViteサーバーにアクセスできるようになり、フロントエンドの開発環境が正しく動作するようになります。
</details>
:::

## 7. tailwind.config.jsの修正

- **ファイル名を `.cjs` に変更**: `tailwind.config.js` を `tailwind.config.cjs` にリネームすることで、このファイルが `CommonJS` として扱われ、問題を回避できます。

`mv tailwind.config.js tailwind.config.cjs`

## 8. svgファイルの読み込みエラー

vite.config.jsに以下を追加することで解決

`publicDir: 'public', *// 公開ディレクトリを指定*`

参考：Vite公式ドキュメント

[静的アセットの取り扱い](https://ja.vitejs.dev/guide/assets.html)

## 9. CommonJSのモジュール内容をESモジュールに書き換え

`require` は CommonJS のモジュールシステムの一部ですが、Vite は ES モジュールを前提としているため、`import` を使用することが推奨されます。

```tsx
// Before
require("./bootstrap");
require("./fontawsome");
```

```tsx
// After
import './bootstrap';
import './fontawsome';
```

:::note info
**CommonJS、 ESModulesとは？？**

**CommonJS** と **ESModules** は、JavaScriptのモジュールシステムを提供する二つの異なる方式です。モジュールシステムは、コードをモジュール（ファイル）に分割して再利用しやすくするための仕組みを提供します。以下に、それぞれの特徴と違いを説明します。

<details>
<summary>詳細はこちら</summary>

### CommonJS

- **概要**: CommonJS は、Node.js で使用されるモジュールシステムです。サーバーサイドJavaScriptのモジュールの標準化を目指して設計されました。
- **主な特徴**:
    - **`require` 関数**: 他のモジュールをインポートするために `require` 関数を使用します。
    - **`module.exports` オブジェクト**: モジュールをエクスポートするために `module.exports` を使用します。
    - **同期的**: `require` は同期的にモジュールを読み込むため、スクリプトの実行がブロックされる場合があります。これはサーバーサイドの環境では問題ありませんが、ブラウザ環境では非効率になることがあります。
    - CommonJS サンプル
        
        **math.js**:
        
        ```jsx
        function add(a, b) {
          return a + b;
        }
        
        module.exports = {
          add,
        };
        ```
        
        **main.js**:
        
        ```jsx
        const math = require('./math.js');
        ```
        

### ESModules (ESM)

- **概要**: ESModules（ESM）は、JavaScriptの公式なモジュールシステムです。ECMAScript 2015 (ES6) で導入され、Node.js やブラウザなど、広くサポートされています。
- **主な特徴**:
    - **`import` キーワード**: 他のモジュールをインポートするために `import` を使用します。
    - **`export` キーワード**: モジュールをエクスポートするために `export` を使用します。
    - **非同期的**: ESModulesは非同期的にモジュールをロードするため、ブラウザ環境でも効率的に動作します。
    - **静的解析可能**: `import` と `export` はファイルの先頭で宣言されるため、JavaScriptエンジンがスクリプトを解析する際に、依存関係を事前に把握できます。
    - ESModules (ESM) サンプル
        
        **math.js**:
        
        ```jsx
        export function add(a, b) {
          return a + b;
        }
        ```
        
        **main.js**:
        
        ```jsx
        import { add } from './math.js';
        console.log(add(2, 3)); // 出力: 5
        ```
        
        ### 
        

### 違い

- **書き方**: CommonJSでは `require` と `module.exports` を使いますが、ESModulesでは `import` と `export` を使います。
- **実行環境**: CommonJS は主にNode.jsで使用され、ESModules はブラウザや最新のNode.jsで使用されます。
- **同期 vs 非同期**: CommonJSは同期的にモジュールをロードし、ESModulesは非同期的にモジュールをロードします。
- **静的 vs 動的**: ESModulesは静的（スクリプトの解析時に決定される）、CommonJSは動的（スクリプトの実行時に決定される）です。

### プロジェクトのモジュールシステムの判定方法

- **package.json** ファイルに `"type": "module"` がある場合、そのプロジェクトはESModulesとして扱われます。
- **`require`/`module.exports` が使われている場合** はCommonJS、**`import`/`export` が使われている場合** はESModulesです。

この二つのモジュールシステムは、JavaScriptの環境によって選択され、使い分けられることがありますが、Node.jsの最新版では両方がサポートされています。

</details>
:::

## Laravel Mixの削除
Laravel MixのパッケージやMixによってビルドされていたファイルは不要になるため削除しました。
その他Mixで使用していた環境変数などもViteに移行しました。
環境によって使用有無があるかと思いますので、環境に応じて必要な移行対応を実施ください。

## まとめ
LaravelのビルドツールをLaravel MixからViteに移行することで、開発効率の向上とビルド時間の大幅な短縮が実現しました。移行前のLaravel Mixでは、`npm run dev` や `npm run build` の実行に時間がかかり、開発のスピードに影響を及ぼしていましたが、Viteに移行後はこれらのビルド時間が大幅に短縮され、開発プロセスがスムーズになりました。

具体的には、`npm run dev` の実行時間が約3分から1秒以下に、`npm run build` の実行時間が約3分から10秒に短縮されました。また、HMR（ホットモジュールリプレースメント）の効率化により、差分更新やキャッシュを考慮した迅速な開発が可能となり、開発者の生産性が向上しました。

移行対応も数時間ほどでスムーズに移行できたため、Laravel Mixを使用されている方は、是非Viteへの移行を検討してみてください。