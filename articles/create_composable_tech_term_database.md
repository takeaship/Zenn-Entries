---
title: "Webの最新データで更新可能な技術用語のデータベースを構築する実験"
emoji: "🌐"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "web", "api"]
published: false
---

これは [LAPRAS Advent Calendar 2023](https://qiita.com/advent-calendar/2023/lapras) の 5 日目のエントリーです。
[LAPRAS 株式会社](https://lapras.com/) で開発者をしている [takeaship](https://lapras.com/public/takeaship) と申します。

# TL;DR
- 技術タグを検索や選択できる機能を作るとき、裏側のタグデータを構築・維持するのは大変
- 既存のサービスで使えそうなものはなかったため、自前で構築できるか検証した
- ほどほどの手間で、そこそこ使えて自動更新できる技術用語データが構築できた

# 技術タグのデータベースをどうやって構築するか問題

これは幣サービス・LAPRAS の求人検索画面で、技術タグを選択して求人を検索できます。
![](/images/2023-12-06-00-24-44.png)

Zenn でも、トピックを選んでエントリーを検索できます。
![](/images/2023-12-06-00-38-47.png)

ここで少し考えてほしいのですが、もしあなたがこのような技術タグでの検索を実装するなら、裏側にあるタグデータはどうやって構築するでしょうか？
少し考えてみただけでも、手動で構築するのは現実的ではないと想像がつきます。

世の中にはすでに万を超える技術が存在します。それらには流行り廃りがあり、さらに年々新しい技術が生まれています。
ここ 1 年では Gen-AI 関連のリリースが大量にありましたし、Mojo や Bun なども話題になりました。
**一度データベースを構築するだけでも大変なのに、その後も継続的に最新情報を取り入れてアップデートし続けなければなりません。**

LAPRAS では Web のクロールデータからタグを自動で抽出していますが、お恥ずかしながらノイズも多いですし、そのような大がかりな仕組みを構築・維持するのは大変です。
Zenn はユーザーが自由に付けたタグのうち特に人気のある一部をトピック化していると想像しますが、それ以外は名寄せがされておらず、使い勝手はあまりよくありません。

もっと簡単に、個人開発レベルの労力で、**最近の動向を取り込んで自動更新し続ける技術用語データベースが構築できないか**検証してみました。

# 構築方法を検討する

## 1: 既存の技術タグデータベースを活用する

すぐに使い始められる既存サービスがあるなら、多少のお金を払ってでも、それを活用して車輪の再発名を避けるのが賢明です。

様々なソースを当たりましたが、**残念ながら有料・無料を問わず、データセット・ライブラリ・API としてそのままアプリケーションに取り込めるものはありませんでした。**

Rapid API で見つけた[Technology Stack API](https://rapidapi.com/findmassleadsapi/api/technology-stack3/)が最も近いものでしたが、現在はまともに動作しておらず、更新もされていないようです。
使えそうな既存サービスを見つけたら、ぜひ教えてください。

## 2: 誰でも利用可能な Web のデータを組み合わせる

様々な公開データを探した結果、次の 2 つを組み合わせて自前のデータベースを構築するのがベストだと考えました。

### 2.1. Stack Exchange API

[StackExchange](https://stackexchange.com/) は、StackOverflow や Ask Ubuntu などの Q&A サイトを運営する企業です。
[Stack Exchange API](https://api.stackexchange.com/) を使って、これらサイトのデータを取得できます。

利用規約には Stack Exchange Network が出典であることを明示したうえで、API から取得したデータを利用できると記されています[^1]。

今回は次の 2 つのリソースを利用し、StackOverflow に蓄積されたデータを取得します。

#### [`/tags`](https://api.stackexchange.com/docs/tags)

投稿に付けられたタグの一覧を取得できます。
人気順(利用されている頻度が高い順)でソートして取得できるため、特に重要な用語を優先して取得できます。

:::details StackOverflow のタグを人気順で取得した結果

```json
{
  "items": [
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 2520782,
      "name": "javascript"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 2178140,
      "name": "python"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 1912593,
      "name": "java"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 1609307,
      "name": "c#"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 1462897,
      "name": "php"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 1413983,
      "name": "android"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 1184323,
      "name": "html"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 1034732,
      "name": "jquery"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 803247,
      "name": "c++"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 801560,
      "name": "css"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 685397,
      "name": "ios"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 668306,
      "name": "sql"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 662048,
      "name": "mysql"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 501725,
      "name": "r"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 471917,
      "name": "reactjs"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 469698,
      "name": "node.js"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 415926,
      "name": "arrays"
    },
    {
      "has_synonyms": false,
      "is_moderator_only": false,
      "is_required": false,
      "count": 401933,
      "name": "c"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 374253,
      "name": "asp.net"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 359258,
      "name": "json"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 342267,
      "name": "python-3.x"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 337700,
      "name": "ruby-on-rails"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 336008,
      "name": ".net"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 333430,
      "name": "sql-server"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 331845,
      "name": "swift"
    },
    {
      "has_synonyms": false,
      "is_moderator_only": false,
      "is_required": false,
      "count": 310710,
      "name": "django"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 302048,
      "name": "angular"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 292387,
      "name": "objective-c"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 284923,
      "name": "pandas"
    },
    {
      "has_synonyms": true,
      "is_moderator_only": false,
      "is_required": false,
      "count": 284419,
      "name": "excel"
    }
  ],
  "has_more": true,
  "quota_max": 10000,
  "quota_remaining": 9986
}
...
```

:::

`javascript` `python` `java`など主要な技術が上位にきています。
一方で、`array` `python-3.x`などのノイズも含まれているので、これは取り除きたいです。

#### [`/tags/{tags}/synonyms`](https://api.stackexchange.com/docs/synonyms-by-tags)

タグを指定して、タグの同義語を取得できます。

:::details javascript の同義語を取得した結果

```json
{
  "items": [
    {
      "creation_date": 1279238876,
      "last_applied_date": 1703510973,
      "applied_count": 59194,
      "to_tag": "javascript",
      "from_tag": "js"
    },
    {
      "creation_date": 1539158585,
      "last_applied_date": 1703483163,
      "applied_count": 124,
      "to_tag": "javascript",
      "from_tag": "vanillajs"
    },
    {
      "creation_date": 1281538239,
      "last_applied_date": 1703401109,
      "applied_count": 645,
      "to_tag": "javascript",
      "from_tag": ".js"
    },
    {
      "creation_date": 1447191133,
      "last_applied_date": 1700441093,
      "applied_count": 97,
      "to_tag": "javascript",
      "from_tag": "vanilla-javascript"
    },
    {
      "creation_date": 1280762048,
      "last_applied_date": 1700171045,
      "applied_count": 648,
      "to_tag": "javascript",
      "from_tag": "ecmascript"
    },
    {
      "creation_date": 1431907386,
      "last_applied_date": 1698671444,
      "applied_count": 3,
      "to_tag": "javascript",
      "from_tag": "javascript-runtime"
    },
    {
      "creation_date": 1539158303,
      "last_applied_date": 1691872746,
      "applied_count": 46,
      "to_tag": "javascript",
      "from_tag": "vanilla-js"
    },
    {
      "creation_date": 1428794822,
      "last_applied_date": 1689095169,
      "applied_count": 119,
      "to_tag": "javascript",
      "from_tag": "javascript-library"
    },
    {
      "creation_date": 1395918773,
      "last_applied_date": 1687429660,
      "applied_count": 31,
      "to_tag": "javascript",
      "from_tag": "javascript-dom"
    },
    {
      "creation_date": 1447311382,
      "last_applied_date": 1643374818,
      "applied_count": 22,
      "to_tag": "javascript",
      "from_tag": "javascript-module"
    },
    {
      "creation_date": 1283885940,
      "last_applied_date": 1527596829,
      "applied_count": 50,
      "to_tag": "javascript",
      "from_tag": "javascript-execution"
    },
    {
      "creation_date": 1383827539,
      "last_applied_date": 1521102288,
      "applied_count": 4,
      "to_tag": "javascript",
      "from_tag": "javascript-alert"
    },
    {
      "creation_date": 1402396222,
      "last_applied_date": 1479835850,
      "applied_count": 20,
      "to_tag": "javascript",
      "from_tag": "javascript-disabled"
    },
    {
      "creation_date": 1333148281,
      "applied_count": 0,
      "to_tag": "javascript",
      "from_tag": "classic-javascript"
    }
  ],
  "has_more": false,
  "quota_max": 10000,
  "quota_remaining": 9966
}
```

:::

`js` `.js` などが取得できました。
これらの同義語を `javascript` に名寄せするのに使えそうです。

### 2.2. StackShare の`sitemap.xml`

[StackShare](https://stackshare.io/) とは、個人や企業が利用している技術スタックを登録して公開できるサービスです。

![stackshare.io/stacks](/images/2023-12-24-14-06-02.png)
_企業が公開している技術スタック。[stackshare.io/stacks](https://stackshare.io/stacks)より_

StackShare 自身が、ユーザーが登録するための技術のデータベースともなっています。
このデータベースは、StackShare の開発者やユーザーによってメンテされており、網羅性は極めて高いです。

![stackshareのreactのページ](/images/2023-12-24-14-21-16.png)
_React のページ。[stackshare.io/react](https://stackshare.io/react) より_

ただし、規約でクローリングは許可されているものの、スクレイピングは禁止されています[^2]。
実際、StackShare をスクレイピングしてデータベースをリバースエンジニアリングするような行為は、サイトに負荷をかける上、StackShare の権利を侵害するおそれがあるため避けるべきです。

そこで、今回は**StackShare の`sitemap.xml`を、その技術が存在するかどうかのチェックに利用します。**
:::message
sitemap.xml とは、Web サイトの構造をクローラーに伝えるためにサイトが公開するファイルのこと。
:::

StackShare の[sitemap.xml](https://stackshare.io/sitemap.xml)は分割されており、[sitemaps/tools.xml](https://stackshare.io/sitemaps/tools.xml) ~ [sitemaps/tools4.xml](https://stackshare.io/sitemaps/tools4.xml) までの 4 つのファイルに技術ページの一覧が記載されています。

:::details sitemaps/tools.xml の冒頭部分

```xml
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://stackshare.io/npm-gulp-mocha-phantomjs</loc>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://stackshare.io/carthage-hyperoslo-keychains</loc>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://stackshare.io/clojars-simple-xhr</loc>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://stackshare.io/elm-webbhuset-elm-review-forbid-specific-imports</loc>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://stackshare.io/elm-pd-andy-elm-limiter</loc>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://stackshare.io/clojars-minnow</loc>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://stackshare.io/elm-minibill-elm-ui-with-context</loc>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://stackshare.io/elm-jjant-unwrap</loc>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://stackshare.io/pypi-flake8-builtins</loc>
    <priority>1.0</priority>
  </url>
...
```

:::

全部で**77248 件**の技術ページがありました(2023/12 現在)。膨大な数ですね…。

# 方針: どうデータを組み合わせるか

今回は世に存在するあらゆる技術を網羅することより、ユーザーにとって使いやすことを優先したいです。
そこで、特に登場頻度の高い技術に絞って取得し、人気度順にソートしておくことを考えます。

StackOverflow のタグ情報のうち、特に人気のある上位 3000 件を取得し、それら全てについて同義語も取得します。
この中にはノイズも含まれるので、**ノイズの少ない StackShare の技術ページ URL と突き合わせ、双方に存在するものだけを抽出することでノイズを取り除きます**。
StackOverflow のタグと StackShare の URL では表記が異なることも考えられるので、同義語データも利用して取りこぼしを最小限にします。

## Python でのデータ取得・加工の実装

データの取得・加工を Python で実装したスクリプトがこちらです。
(GitHub 上で見たほうが見やすいです)
https://github.com/takeaship/auto-generated-tech-term-database/blob/main/fetch_tech_stack_data.ipynb

# 結果: 作成できたデータ

最終的に得られたデータを検索できる Web アプリケーションを用意しました。

https://search-tech-terms.streamlit.app/

検索した用語を選択すると、StackShare の技術ページへのリンクが表示されます。
ぜひ触ってみて、気になる技術がヒットするか検証してみてください。
検索したくなるような技術はかなり網羅されていると思います。

![](/images/2023-12-26-00-33-02.png)

## 生の CSV データ

生データはこちらです。
https://github.com/takeaship/auto-generated-tech-term-database/blob/main/data/stack_exchange_tags_with_stackshare.csv

取得した StackOverflow のタグはすべて含まれています。
`is_valid=True`のデータが、StackShare の技術ページに見つかったものです。

`arrays` `python-3.x` `xml` などのノイズを適切に除去できました。
一方で、`c++` `amazon web services` なども含めたかったですが、URL との表記が合わず取りこぼしてしまいました。
こちらはあいまい検索などを利用してすくい取ることを今後の課題としたいです。

# 無料 & ほどほどの手間で使える技術用語データが構築できた

Web 上に公開されているデータを組み合わせることで、ほどほどの手間でそこそこ使える技術用語データが構築できることがわかりました。
今回作成したスクリプトを再度実行すれば最新のデータを取得できるので、流行を取り入れて継続的にデータを更新していくことが可能です。
ご自身のアプリケーションに組み込んでみてください。
需要があればライブラリとして組み込めるよう改善していこうと思います。

# おまけ: 検討したが没にしたデータソース
いずれも優れたデータソースですが、今回の目的には合わなかったため没にしました。

## StackShare API

https://docs.stackshare.io/
StackShare の GraphQL API です。
Tools リソースで StackShare に登録されている技術を検索できます。

一見使えそうですが、入力したキーワードにヒットする技術を取得することしかできないため、一覧取得はできないし、人気度の情報もありません。
Rate Limit は無料枠で月 100 回までとかなり少ないです。
さらに、技術調査などでの利用を想定しているようで、取得した情報の再頒布は規約で禁止されています[^3]。

## Wappalyzer API

https://www.wappalyzer.com/docs/api/v2/lookup/

[Wappalyzer](https://www.wappalyzer.com/) は、閲覧している Web サイトの技術スタックを調査するためのブラウザ拡張です。

![](/images/2023-12-26-01-21-26.png)
_Wappalyzer でサイトの技術スタックを表示した例_

非常に便利な拡張機能で、技術の網羅率も非常に高そうですが、サイトを指定して技術スタックを取得するエンドポイントがあるのみで、技術の一覧取得はできませんでした。
価格も最低 $250~/月 と高額です。

## builtWith のデータセット

https://builtwith.com/datasets/entire-internet-datasets

[builtWith](https://builtwith.com/) は、Wapplyzer 同様に、閲覧している Web サイトの技術スタックを調査するためのブラウザ拡張です。

親切にも世のサイト \* 利用している技術のデータセットを公開してくれています。
しかしながら、技術として Postgres が含まれていないなど、Wappalyzer と異なりバックエンドの技術はカバーしていないようです。

## Dev Community API

https://dev.to/alfredosalzillo/the-state-of-devto-v0-api-1o2

[Dev Community](https://dev.to/) は、開発者向けのグローバルなコミュニティサイトです。
投稿に付けられたタグ一覧を人気順で取得できる API があります。
悪くないですが、Stack Exchange API と比べるとデータの網羅率は低く、同義語の取得もできません。

[^1]: https://stackexchange.com/legal/api-terms-of-use
[^2]: https://stackshare.io/terms
[^3]: https://stackshare.io/terms-api
