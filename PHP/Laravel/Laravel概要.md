# Laravel 概要

- [公式](https://readouble.com/laravel/9.x/ja/installation.html)

## フルスタック

- Laravel は、フルスタックフレームワークとして機能させることができます
- 「フルスタック」フレームワークとは、Laravel を使用して、アプリケーションへのリクエストをルーティングし、**Blade テンプレート**や **Inertia** などのシングルページアプリケーションハイブリッド技術でフロントエンドをレンダすることを意味します。
- これは、Laravel フレームワークの最も一般的な使用方法であり、私たちの意見では、Laravel を使用する最も生産的な方法です。
- Laravel をフルスタックフレームワークとして使用している場合、**Vite** を使用してアプリケーションの CSS と JavaScript をコンパイルする方法を学ぶのも強く推奨します。

## API バックエンド

- Laravel は、JavaScript シングルページアプリケーションまたはモバイルアプリケーションへの API バックエンドとしても機能させることもあります
- たとえば、Next.js アプリケーションの API バックエンドとして Laravel を使用できます
  - こうした使い方では、Laravel でアプリケーションに認証とデータの保存/取得を提供すると同時に、キュー、メール、通知などの Laravel の強力なサービスを利用できます。
  - Laravel **Breeze** は、API スタックと Next.js フロントエンド実装を提供しているため、すぐに開始できます。

## フロントエンド

### Blade

### Livewire

- Laravel Livewire は、Vue や React といったモダンな JavaScript フレームワークで作られたフロントエンドのように、ダイナミックでモダン、そして生き生きとした Laravel で動作するフロントエンドを構築するためのフレームワークです
- Livewire を使用する場合、レンダし、アプリケーションのフロントエンドから呼び出したり操作したりできるメソッドやデータを公開する UI 部分を Livewire「コンポーネント」として作成します

### Inertia（Vue,React の使用）

- Inertia は、Laravel アプリケーションとモダンな Vue または React フロントエンドの間のギャップを埋めるもの
- Vue や React を使って本格的でモダンなフロントエンドを構築しながら、ルーティング、データハイドレート、認証に Laravel ルートとコントローラを活用できます
- すべて単一のコードリポジトリ内で行えます
- このアプローチではどちらのツールの能力も損なうことなく、Laravel と Vue／React の両能力を享受することができます。

### アセットの結合（Vite）

- Blade と Livewire、Vue／React と Inertia のどちらを使用してフロントエンドを開発するにしても、アプリケーションの CSS をプロダクション用アセットへバンドルする必要があるでしょう。もちろん、Vue や React でアプリケーションのフロントエンドを構築することを選択した場合は、コンポーネントをブラウザ用 JavaScript アセットへバンドルする必要があります
- Laravel は、デフォルトで Vite を利用してアセットをバンドルします。[Vite](https://readouble.com/laravel/9.x/ja/vite.html) は、ローカル開発において、ビルドが非常に速く、ほぼ瞬時のホットモジュール交換（HMR）を提供しています。スターターキットを含むすべての新しい Laravel アプリケーションでは、vite.config.js ファイルがあり、軽量な Laravel Vite プラグインがロードされ、Laravel アプリケーションで Vite を楽しく使用できるようにしています。
- Laravel と Vite を使い始める最も早い方法は、Laravel Breeze を使ってアプリケーションの開発を始めることです。これは、フロントエンドとバックエンドで認証に必要なスカフォールドを生成してくれます。アプリケーション開発をロケットスタートする、最もシンプルなスターターキットです。

## スターターキット

- https://readouble.com/laravel/8.x/ja/starter-kits.html
- 新しい Laravel アプリケーションの構築をすぐに取りかかれるようするため、認証とアプリケーションのスターターキットを提供しています。これらのキットはアプリケーションのユーザーを登録および認証するために必要なルート、コントローラ、ビューを自動的にスカフォールドします。

### Laravel Breeze

- Laravel Breeze へログイン、ユーザー登録、パスワードのリセット、メールの検証、パスワードの確認など、Laravel のすべての認証機能を最小限シンプルに実装しました。Laravel Breeze のデフォルトビュー層は、Tailwind CSS でスタイルを設定したシンプルな Blade テンプレートで構成しています。
- Inertia と Vue／React を使用してフロントエンドを構築したい場合は、Breeze または Jetstream のスターターキットを活用して、アプリケーション開発を迅速に開始できます。
- これらのスターターキットは、Inertia、Vue／React、Tailwind、Vite を使用してアプリケーションのバックエンドとフロントエンドでの認証フローに必要なスカフォールディングを生成するため、次に開発する皆さんの大きなアイデアをすぐに構築開始できます。

### Breeze と Inertia

Laravel Breeze では、Vue や React を使った Inertia.js のフロントエンド実装も提供しています。Inertia スタックを使用するには、breeze:install Artisan コマンドを実行する際に、希望するスタックとして vue または react を指定します。

```sh
php artisan breeze:install vue

// もしくは…

php artisan breeze:install react

npm install
npm run dev
php artisan migrate
```

### Breeze と Next.js／API

Laravel Breeze は、Next や Nuxt などのモダンな JavaScript アプリケーションで認証する API をスカフォールドすることもできます。使い始めるには、breeze:install Artisan コマンドを実行する時、api スタックを希望するスタックとして指定します。

```sh
php artisan breeze:install api

php artisan migrate
```

インストール時に、Breeze はアプリケーションの.env ファイルへ環境変数 FRONTEND_URL を追加します。この URL は、あなたの JavaScript アプリケーションの URL でなければなりません。ローカル開発時、通常は http://localhost:3000 となります。
