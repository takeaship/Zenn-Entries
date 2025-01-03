---
title: "社外秘のスライドを Amazon Bedrock の Claude で韓国語に翻訳させた話"
emoji: "🔀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws", "bedrock", "llm", "claude", "ai"]
published: true
---

:::message
これは [LAPRAS Advent Calendar 2024](https://qiita.com/advent-calendar/2024/lapras) 4 日目の投稿です。
:::

スタートアップ企業で経営企画を務めるオジマさんに、ある日突然重厚なタスクが降ってきました。
韓国籍のパートナー企業に自社の戦略をより深く知ってもらうため、**社外秘の会社紹介資料の韓国語版を作成する**ように言われたのです。
資料は 30 ページ以上あり、Google 翻訳で文章をちまちま翻訳していたら日が暮れそうです。
困ったオジマさんは、全社になんでも質問できるチャンネルで助けを求めました。

> @翻訳作業が日本一得意な方
> お疲れ様です！
> 日本語のスライド(社外秘)を全部韓国語に変換したいのですが、
> こんなツールを使えば楽だよ！とか
> こういう方法を使えば効率的だよ！とか
> アイディアがあればご教授いただきたいです。

このスタートアップ企業とは、私が所属する LAPRAS 社のことです。
本エントリでは、オジマさんの質問を見た私が限られた時間の中でどう AI による翻訳を行い、どのくらい使える結果が得られたかを記していきます。

# Summary

- Amazon Bedrock の Claude を呼び出すプログラムを書いて、社外秘な資料から文章を抜き出し翻訳する作業をある程度自動化できた
- PDF の各ページを画像に変換し、文章を抜き出すプロンプトと抜き出した文章を翻訳するプロンプトを別々に発行することで精度を高められた
- 出力された文章には一部不適切な訳やハルシネーションが含まれたので、韓国語ネイティブの社員に校正してもらうことで社外に出せる品質になった
- スライド上の文章を翻訳文に置き換える作業ではまだ人手が必要

# 機密情報を、本業タスクも抱えている中の片手間で翻訳せよ

今回の制約条件は次の 2 つです。

1. 社外秘の情報を扱うため、使用できるツールが限られる
1. 翻訳作業は本業タスクも抱えている片手間で行う

私は普段 C 向けのプロダクト開発をしており、毎週ユーザーに新機能を届けるために今週もたくさんの Issue を抱えています。
**人助けは大事ですが、使える時間はせいぜい 1~2 時間**です。利用実績のないツールを調べながら使っている余裕はありません。

今までに使ったことがあり、機密情報を与えても問題ないツール … **社内向けに R&D チームが用意してくれた Amazon Bedrock** が思い浮かびました。
以前 Bedrock の API を叩いたスクリプトが手元にあります。こいつを改造すれば、数時間でとりあえずの出力は作れそうだ。そう考え、助けになりますよと名乗り出ました。

# なぜ Amazon Bedrock を使うのか

LAPRAS では、ユーザーに提供する機能や社内ツール等を開発するときに使える LLM 基盤として、Amazon Bedrock を使っています。
Bedrock には機密情報を与えてもリスクが十分低いと判断するに足る次の特徴があります。

1. IAM ロールを使用したエンタープライズ基準の認証
2. 入力情報は学習に利用されず、第三者にも渡されない[^1]
3. **入力データは不正利用防止のため機械によりレビューされるが、人によるレビューは行われない**[^2]

特に 3 つめの特徴が、他の企業向け LLM 基盤と比較して優れています。

例えば ChatGPT API / Enterprise や Azure OpenAI Service は 1, 2 を満たしますが、3 に関しては限定された人によるレビューが行われることがあるようです[^3][^4]。
（リスクは相対的に上がりますが、十分低い範囲とは考えられるので、避けるべきとまでは思いません。）

**個人向け ChatGPT や Claude 等のチャット画面に機密情報を入力するのは、当然避けるべきです。**

# 作成した Python スクリプト

サンプルスクリプトはこちらです。
AWS への認証は私の環境では aws-vault で通しているので、コードには含んでいません。
各々の環境に会わせて設定してください。

https://gist.github.com/takeaship/7ea99f19f5e7b62dbe0fabb0399b4696

## 得られる結果

スライドのサンプルと、出力結果です。
スライド上のテキストは手作業で置き換える必要があるため、どの日本語にどの韓国語が対応しているか分かる必要があります。そのため抜き出した日本語に続けて韓国語を出力しています。

![翻訳するスライドのサンプル](/images/sample_slide_to_translate.png)

:::details 出力結果

```txt
### page 1 ###
## ja ##
経営メンバー紹介

代表取締役 CEO
染谷 健太郎
Kentaro Someya

CTO
興梠 敬典
Takanori Koroki

2009年に東京大学文学部を卒業し、株式会社リクルートジョブズに入社。3年ほど営業を担当し、MVPなど表彰複数回。その後、新規事業開発を担当し、複数のSaaSサービスなどの新規サービスの立ち上げ・推進に、コアメンバーとして関わる。担当サービスにて「日本HRチャレンジ大賞人材サービス優秀賞」や「グッドデザイン賞」を受賞。2018年1月よりLAPRAS株式会社のプロダクトマーケティングマネージャーとしてジョイン。2022年2月、代表取締役CEOに就任。

豊田高専を卒業後、ソフトウェアエンジニアとして多様な開発案件に従事。複数の新規事業立ち上げに関わり、ビジネス/ソフトウェア双方の設計と構築を経験。2015年より株式会社Nextremer高知AIラボの代表として、事業や組織の立ち上げを主導。地域コミュニティや行政とも協力関係を構築し、地方でも先端技術に触れられる場作りにも貢献。掲げるミッションと価値観に共感し、2019年8月にLAPRASに入社。2020年10月、執行役員CTOに就任。
## kr ##
경영진 소개

대표이사 CEO
소메야 켄타로
Kentaro Someya

CTO
코로키 타카노리
Takanori Koroki

2009년 도쿄대학교 문학부를 졸업하고 주식회사 리크루트 잡스에 입사. 약 3년간 영업을 담당하며 MVP 등 여러 차례 수상. 이후 신규 사업 개발을 담당하며 여러 SaaS 서비스 등 신규 서비스의 런칭 및 추진에 핵심 멤버로 참여. 담당 서비스로 '일본 HR 챌린지 대상 인재 서비스 우수상'과 '굿 디자인상'을 수상. 2018년 1월부터 LAPRAS 주식회사의 프로덕트 마케팅 매니저로 합류. 2022년 2월, 대표이사 CEO로 취임.

토요타 고등전문학교를 졸업한 후 소프트웨어 엔지니어로 다양한 개발 프로젝트에 종사. 여러 신규 사업 런칭에 참여하며 비즈니스/소프트웨어 양쪽의 설계와 구축을 경험. 2015년부터 주식회사 Nextremer 고치 AI 연구소의 대표로서 사업과 조직의 설립을 주도. 지역 커뮤니티 및 행정과도 협력 관계를 구축하여 지방에서도 첨단 기술을 접할 수 있는 환경 조성에 기여. 추구하는 미션과 가치관에 공감하여 2019년 8월 LAPRAS에 입사. 2020년 10월, 임원 CTO로 취임.
```

:::

Google 翻訳で逆翻訳した結果、概ね正確に翻訳できていることが分かります。
![Google翻訳で逆翻訳した結果](/images/google_reverse_translate_result.png)

## 使い方

1. スライドの各ページを画像に変換し、`./images`ディレクトリに保存する
2. 環境に boto3, botocore をインストールする
3. スクリプトを実行する
4. コーヒーでも淹れながら待つ (1 枚あたり 30 秒ほどかかります)

## スクリプトの解説

スライド画像を 1 枚ずつループし、

1. LLM で画像からテキストを抜き出す
2. LLM で 1 で抜き出したテキストを韓国語に翻訳する
3. 1 と 2 の結果をファイルに書き込む

を繰り返し実行します。

1. のプロンプト

```txt
添付画像のテキストを抜き出してください。

# 出力
- 抜き出したテキストだけを出力すること
- Confidential という文字列が含まれている場合は、その文字列を含む行を削除すること
```

2. のプロンプト

```txt
次のテキストを韓国語訳してください。

# 出力
- 翻訳した韓国語だけを出力すること

# テキスト
{text}
```

## うまく行かなかった方法

このスクリプトにたどり着くまでに試した方法をいくつか紹介します。
これらの方法では途中までしか処理できなかったり、精度が著しく低かったりして、実用に耐えませんでした。

### PDF ファイルを添付してテキストの抜き出しを指示する

**Claude は PDF の添付にも対応していますが、画像を添付した場合と比べてテキストの抜き出し精度が大きく劣りました。**
特にスライド枚数が多い場合は、コンテキストウィンドウが足りずスライドの途中までしか処理してくれません。
画像に変換してから処理させることをおすすめします。

### テキストの抜き出しと翻訳を一度のプロンプトで指示する

LLM は性質の異なるタスクを同時に実行するのが苦手なようです。
**1 回のプロンプトでテキストの抜き出しと翻訳を指示するよりも、別々のプロンプトで指示したほうが精度が大きく高まりました。**

### Confidential という文字列の削除を指示しない

社外秘資料なので、Confidential という文字列が含まれています。
これを削除するよう指示しないと、抜き出したテキストに Confidential という文字列が含まれて手直しが必要になったり、**ひどい場合は「Confidential な資料のため、処理できません」と返答してきます。**

# 出力結果の評価

出力された韓国語が適切がどうか私には判断ができないため、弊社の韓国語ネイティブの SWE [@lostfind](https://lapras.com/public/lostfind) に校正してもらいました。
その上でオジマさんがスライドの文字を張り替え、無事韓国のパートナー企業に資料が提供できました。
お二人からのフィードバックを掲載します。

## オジマさんから

> - 全ては翻訳しきれておりませんでしたが 7 割くらいは埋められました
> - ファイルの中まで直接変えられたら更に神だったと思いました

## lostfind さんから

> ゼロから翻訳をするのと比較したら、7~8 割はもうできていた感じでした
> 日本語 -> 韓国語の翻訳自体は結構自然な形だったので、僕は直訳されている用語の修正や統一をしました。
> たまに伝わりづらいものもありましたが、全体の数%ぐらいかなって感じです。

> 途中で気付いたのが、日本語の抽出でハルシネーションがあったということです。
> テキストだけを見ると全然おかしくなかったですが、よく見るとあれ？と思う記述があり、元資料と比較した結果気づきました。
> 結果的には小さい文字で多く書かれたメンバー紹介ページだけがおかしくて、それ以外は大丈夫でした。

まだまだ完全自動化までの道のりは遠そうですが、0.7 \* 0.7 で翻訳作業の半分くらいは貢献できたかなと思います。

# 後日談

## オジマさんの資金調達話

その後、**辣腕[オジマさん](https://note.com/ojimakoto/)の尽力により、韓国籍のパートナー企業 [Wanted Lab](https://wantedlab.com/) から出資を受けることができました。**

https://prtimes.jp/main/html/rd/p/000000092.000024729.html

[LAPRAS Advent Calendar 2024](https://qiita.com/advent-calendar/2024/lapras) 最終日に、オジマさんが資金調達の話をしてくれるそうです。乞うご期待ください。
![LAPRASの資金調達の話をしようか](/images/ojima_finance_story.png)

## このスクリプトを使う機会がまた訪れた話

後日、このスクリプトのマイナーチェンジ版を使って別の資料を英語に翻訳する機会に恵まれました。
プロンプトをちょっと変えるだけで多くの言語に対応できるのが LLM の魅力ですね。

[^1]: https://docs.aws.amazon.com/bedrock/latest/userguide/data-protection.html
[^2]: https://docs.aws.amazon.com/bedrock/latest/userguide/abuse-detection.html
[^3]: https://openai.com/enterprise-privacy/
[^4]: https://learn.microsoft.com/en-us/legal/cognitive-services/openai/data-privacy#preventing-abuse-and-harmful-content-generation
