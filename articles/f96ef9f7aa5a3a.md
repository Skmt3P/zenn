---
title: "Webフロントエンドの面倒な開発環境構築は、なんちゃってモジュラーモノリスで簡略化しよう！"
emoji: "🏢"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Nuxt", "Vue", "Nuxt3", "設計", "Bun", "Bun Workspaces", "アーキテクチャ"]
published: true
---

# はじめに

本稿は、「Nuxt / UnJS Advent Calendar 2024」の初日の記事です。明日の記事は、 @offich さんによる「Nuxt Server Component で作るリンクカード」記事の予定です。

一方で、本稿は「VR法人HIKKY Advent Calendar 2024」の初日の記事でもあります。明日の記事は @naninunenoy さんによる「シングルトンアンチのためのUnityにおけるオブジェクト共有方法」になります。

なお、本稿の記載内容は、私個人の経験や知見に基づき、私個人の責任範囲にて執筆しているものです。所属企業のメンバーとして、また私的に所有しているエストニア法人格としての執筆ではございません。また、本稿の情報は、2024年12月1日時点の情報となります。現時点において、私自身も本アーキテクチャの完全理解には至っていない旨、何卒ご容赦いただけますと幸いです。


# Webフロントエンドの開発環境構築、面倒くさいなぁ...

これはWebフロントエンドに限った話ではないかもしれないのですが、 

**開発環境の構築って、面倒くさくないですか？**

Webフロントエンドなら、JSフレームワークを導入して、UIライブラリや便利なUtils等もimportして、LinterやFormatterも、Testing Toolsも整備する。プロダクト開発や個人開発を進める中でも、導入したライブラリは更新され続けるので、適宜アップデートしていかなきゃならない。Dependabotに怒られちゃいますし、脆弱性になるリスクが伴いますしね。

これが単一のプロダクトであれば、まあ我慢できるのですが、似たようなアーキテクチャの複数のプロダクト開発が並走していたり、MVP(Minimum Viable Product)をパパッと作りたい時なんかには、とにかく面倒。「いちいち環境構築なんてしてられるか！」「プロダクト開発に集中させてくれー！」と思う人は少なくないと思います。

しかし、だからといって、サービスドメインが違うプロダクト群をモノリシックアーキテクチャで全て管理してしまうと、アプリケーションが肥大化するし、密結合になって取り回しが効かなくなるため、これはこれで面倒くさくなってしまいます。

**どうにかして、最小限の構成だけはモノリシックに管理しつつ、各々のサービスはいい感じに疎結合な感じで、開発ができないものか** 、この問いに対するひとつの解として、今回の記事の表題にもある「なんちゃってモジュラーモノリス環境」を構築してみました。


# 今回の成果物

そんなわけで、実際に、Bun WorkSpacesとNuxt Layerを使って構成した、なんちゃってモジュラーモノリス環境の実物がこちらです。多分動きます。動かなかったらごめんなさい。StarとForkしてみてください！

https://github.com/PublicHIKKY/vket-boilerplace-nuxt

Bun, Nuxt3, TypeScript, Zod, Scss, ESLintを用いたアプリケーション構成となっています。


# なんちゃってモジュラーモノリス ≒ モジュラーモノリス

なんちゃってモジュラーモノリスの詳細に移る前に、モジュラーモノリスとは何かというと、モノリシックアーキテクチャの中でも「モジュール化」を徹底する設計のことを指します。単一のデプロイメントユニットである点ではモノリスと共通していますが、内部を独立性の高いモジュールに分割することで、変更や拡張の柔軟性を確保する設計ですね。モノリスとマイクロサービスの中間的な位置付けにある設計ともいえます。

一般的なモジュラーモノリスであれば、上述の通り、各モジュール間は依存せずに独立性を保ちつつも、全体としてはモノリスなアプリケーションとなるのですが、上記のリポジトリでは、次の通り、rootにはアプリケーションが存在しておらず、baseレイヤーにmainレイヤーが依存する構成となっています。

```
project-root/
├── layers/                     # レイヤー関連のフォルダ
│   ├── base/                   # 基本設定や共通の機能を定義
│   │   ├── app/                # Base LayerのApplication
│   │   ├── package.json        # Base LayerのPackages
│   │   ├── nuxt.config.ts      # Base LayerのNuxt Config
│   │   └── その他
│   ├── main/                   # メインアプリケーションの拡張
│   │   ├── app/                # Main LayerのApplication
│   │   ├── package.json        # Main LayerのPackages
│   │   ├── nuxt.config.ts      # Main LayerのNuxt Config
│   │   └── その他
├── package.json                # RootのPackages
└── その他
```

一方で、baseのアプリケーションも、mainのアプリケーションも、独自のプロセスで動かしながら開発したり、BuildやDeployをすることができます。また、mainフォルダをコピペして、別のレイヤーを複製すれば、baseの内容を受け継ぎつつ、mainに近似した別アプリケーションを構築することもできます。

本来のモジュラーモノリスであれば、管理機能、決済機能、商品管理機能、注文管理機能などのモジュール群を疎結合なアプリケーション化するところですが、執筆時時点では、mainとbaseしか用意できませんでした...将来的には、認証基盤を担うauthや、storybookを使用したコンポーネントライブラリなどは、単独モジュールとして構築したいと考えております。

このなんちゃってモジュラーモノリスは、全体のリポジトリがひとつに集約され、Packagesが共有されている点はモノリスに、モジュール単位で分割されている点はモジュラーモノリスに、個々のモジュール間はアプリケーションとして独立している点はマイクロサービスに近似しているとも言えます。一方で、baseモジュールを他モジュールに依存させることを是とする点は、モノリスともモジュラーモノリスともマイクロサービスとも言い難い設計手法です。

明確にこれと言いきれるアーキテクチャを思いつけなかったので、本稿ではこの設計をなんちゃってモジュラーモノリスと称します。以下、本設計の詳細の説明となります。


# Rootは極力プレーンにして、JSフレームワークの依存から解放される

この構成のRootディレクトリでは、一切のサービスドメインに依存させないだけでなく、Nuxt3をはじめとした、JSフレームワークにも依存しないようにしています。

```/package.json
  "devDependencies": {
    "@eslint/js": "^9.15.0",
    "@total-typescript/ts-reset": "^0.6.1",
    "@types/eslint__js": "^8.42.3",
    "@types/node": "^22.9.1",
    "@types/postcss-url": "^10.0.4",
    "cross-env": "^7.0.3",
    "eslint": "^9.15.0",
    "globals": "^15.12.0",
    "husky": "^9.1.6",
    "lint-staged": "^15.2.10",
    "postcss-html": "^1.7.0",
    "postcss-import": "^16.1.0",
    "postcss-url": "^10.1.3",
    "sass-embedded": "^1.81.0",
    "stylelint": "^16.10.0",
    "stylelint-config-standard-scss": "^13.1.0",
    "stylelint-order": "^6.0.4",
    "stylelint-rscss": "^0.4.0",
    "ts-node": "^10.9.2",
    "typescript": "^5.6.3",
    "typescript-eslint": "^8.14.0"
  }
  ```

こうすることによって、たとえばNuxtが提供終了されてしまった場合にも、Nextなどの他のJSフレームワークへの移行がしやすくなります。

https://github.com/PublicHIKKY/vket-boilerplace-nuxt

# Baseモジュールには、Nuxtプロジェクト共通の最小構成を放り込む

Baseモジュールには、すべてのNuxtプロジェクトで共通的に扱うライブラリ、コンポーネント、utils関数などを用意します。また、こうした共通機能のLinter, Formatter. ユニットテストも本モジュールに包含させます。

Nuxt, Zod, Vitestなどの共通的に使うライブラリや、i18n機能、トースト機能、日時操作機能、Fetch、Factoryなどは、共通的に用意しておくことで、プロジェクトごとに用意しなくて済むようになります。

また、Rootにあるlinterの設定を拡張し、Nuxtプロジェクトで扱えるようにもしておきましょう。

https://github.com/PublicHIKKY/vket-boilerplace-nuxt/tree/develop/layers/base


# Mainモジュールは、デプロイ対象のアプリケーション

なんちゃってモジュラーモノリスにおけるデプロイメントの対象となるアプリケーション。サービスドメインに依存するソースコードやアセットなどは、すべてここに包含されます。

https://github.com/PublicHIKKY/vket-boilerplace-nuxt/tree/develop/layers/main

また、このMainモジュールを複製して別のSubモジュールを作り、それをMainモジュールに取り込むことで、本当のモジュラーモノリスのようにアプリケーションを拡張させることもできます。

また、このMainモジュールを複製することで、デプロイメントが異なる別アプリケーションを作ることもできます。MVPをたくさんつくったり、みんな大好きクソアプリ(褒め言葉)をWebアプリとして量産する時に便利かと。

https://qiita.com/advent-calendar/2024/kuso-app


# Bun Workspacesを使って、ライブラリを使い回す

このなんちゃってモジュラーモノリスを実現するにあたって、要となる技術要素は大きく二つあります。そのひとつがBun Workspacesです。

https://bun.sh/docs/install/workspaces

Bun Workspacesは、npmのWorkspacesをサポートしており、モジュラーモノリスアーキテクチャにおける、ライブラリの再利用を可能にします。workspacesの指定にはglobパターンも指定可能で、dependenciesの重複も削減してくれます。

https://docs.npmjs.com/cli/v9/using-npm/workspaces?v=true#description

導入は非常に簡単で、bunを用いた上で、rootのpackage.jsonに以下の記述をするだけです。

```/package.json
  "workspaces": [
    "layers/*"
  ]
```

また、各モジュール間の依存を解決するためには、以下のように指定すればOKです。

```/layers/main/package.json
  "dependencies": {
    "vket-boilerplace-nuxt-base": "workspace:*"
  }
```

上記の例では、baseのライブラリをmainでも使えるようにしています。この状態で、`bun install`をすることでlayers配下の全ライブラリをいい感じにrootにinstallしてくれます。

これで、「どうにかして、最小限の構成だけはモノリシックに管理しつつ、各々のサービスはいい感じに疎結合な感じで、開発ができないものか」という命題における、共通パッケージ管理の面倒さを簡略化することできるようになります。


# Nuxt Layersを使って、Nuxtアプリケーションを使い回す

しかしながら、Bun Workspacesだけだと、Nuxtまわりの面倒さは解決できません。Bun Workspacesでは、Nuxtまわりのcomponents等の依存や、auto importを解決することができないからです。そこで活躍するのがNuxt Layersになります。

https://nuxt.com/docs/getting-started/layers

Nuxt Layersでは、Nuxtアプリケーションの継承を行うことができます。これによって、componentsをlayer間で使い回したり、Configを共通化できるようになります。

これまた設定は簡単で、継承先のnuxt.config.tsに、以下の記述をするだけです。

```/layers/main/nuxt.config.ts
export default defineNuxtConfig({
  extends: path.resolve(__dirname, '../base')
})
```

ただし、パスとaliasの管理には気をつける必要があります。ここで結構つまづいた記憶があります。地道に解決していくだけではあるんですけどね...ただ、これで、Nuxtまわりの面倒さもだいぶ簡略化できるはずです。


# 課題もある。さらなるブラッシュアップが必要...!!

個人的には結構いい設計を作れた気がしているのですが、まだ案件やプロダクトでの活用実績がないため、さらなるブラッシュアップが必要だと認識してます。冒頭にも述べた通り、私自身、本アーキテクチャの完全理解には至っていないため、トライアンドエラーを繰り返して、よりよいものにしていきたいですね。

特に、Linter周りの設定が問題なく効いてるかを心配してます。ここらは実際に活用しながら、適宜修正していきたいです。

皆様のLGTM、Star、Fork、コントリビュートが後押しになるため、是非に拝読並びにご活用いただけますと幸甚です。よろしくお願いします！

https://github.com/PublicHIKKY/vket-boilerplace-nuxt
