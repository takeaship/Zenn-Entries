---
title: "Softbank Air ターミナルの DHCP を完全に黙らせて純粋な AP として使うハック"
emoji: "📶"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["無線LAN", "hack"]
published: true
---

フリマサイトにて 1000 円ほどで投げ売りされている Softbank Air ターミナル。
本来は据え置き型のモバイルルーターとして動作しますが、特殊な設定をすることで Wi-Fi 6 対応の高性能無線 AP として使えることが一部で知られています。
その設定方法については先人の投稿に譲ります。

> [SoftBank の Air ターミナルを Wi-Fi6 無線 AP として再利用する方法 | ゆきろぐ](https://yukimomiji.net/softbankair-wifi6-ap/)


スペックの抜粋です[^1]。

- 無線LAN規格: IEEE 802.11a/b/g/n/ac/ax
- 無線LAN 伝送速度: 最大1.3Gbps
- WiFiクライアント最大接続数: 64

[^1]: [Airターミナル4の仕様を教えてください。 | よくあるご質問（FAQ） | サポート | ソフトバンク](https://www.softbank.jp/support/faq/view/25391)

HUAWEI 製のフラッグシップ機をベースにしているだけのことはあり、 **1000 円で手に入る無線 AP としては破格**です。

:::message
この使い方をした場合、ルーター機能のない純粋な AP となるため別でルーターを用意する必要があります。
我が家では古いハイエンドルーターに OpenWrt を焼いて IPv4 over IPv6 に対応させたものを使っています。

OpenWrt について詳しくはこちら
[私がルータを買う時は OpenWRT 対応かどうかを見てる話](https://zenn.dev/tantan_tanuki/articles/574203fe2e627a)
:::

一方で、この方法にはひとつ問題があります。

Air ターミナルの管理画面では DHCP サーバーを無効にする設定がありません。
先人の手法では特殊な設定により疑似的に無効にしていますが、 これは IP アドレスの払い出しをできなくしているだけで DHCP サーバー自体は動いています。
これが上流にあるルーターの DHCP サーバーと競合して IP アドレスの取得に失敗し、無線 LAN に接続できないことがあります。

実際に運用していても「インターネットなし」になることがときどき発生し不便を感じました。

> タイミングに依っては、Air ターミナル 4 の DHCP サーバが既存の DHCP サーバのやり取り中に DHCP NAK を通知してしまうことがあると考えられるため、IP アドレスの割当てや払出しに時間を要したり、いつまで経っても IP アドレスが割当てられないこともあり得ると思います。[^2]

[^2]: [SoftBank の Air ターミナルを Wi-Fi6 無線 AP として再利用する方法#DHCP サーバ衝突の問題 | ゆきろぐ](https://yukimomiji.net/softbankair-wifi6-ap/#DHCP%E3%82%B5%E3%83%BC%E3%83%90%E8%A1%9D%E7%AA%81%E3%81%AE%E5%95%8F%E9%A1%8C)

Web上ではこの問題を解決したという報告は見つかりませんでした。
しかしこのたび **Air ターミナルの DHCP を完全に無効化する方法を発見した**ので紹介します。

## ハックの前提条件

- DevTools の使い方を理解していること
- Air ターミナル4 または Airターミナル4 NEXTであること
  - 型番が _b610h-70a_, _b610h-71a_,  _b610h-72a_ のいずれか

:::message
_b610h-70a_ では動作確認済みですが、この機種は Wi-Fi 5 までにしか対応していないので Wi-Fi 6 対応の _b610h-71a_ 以降をおすすめします。
:::

:::message
_b610h-72a_ では動作確認できていませんでしたが、この方法で無効化できたと情報提供いただきました。ありがとうございました。
https://zenn.dev/link/comments/13d9aeff52bfa1
:::

### Air ターミナル 5 の場合
API 仕様が異なりますが、同様の方法で無効化できるようです。
次のポストを参考に、手順を読み替えて試してみてください。
https://mstdn.maud.io/@imofly2/112682244073359972

https://x.com/houchisanteam/status/1814603888735236428

こちらもコメント欄で情報提供いただきました。ありがとうございました。


## ハックの概要
Air ターミナルの管理画面は次の API で DHCP 設定を保存しています。

`POST /api/dhcp/settings`
- body: 
```xml
<request>
   <dhcpstartipaddress>192.168.3.254</dhcpstartipaddress>
   <dhcplannetmask>255.255.255.0</dhcplannetmask>
   <dhcpipaddress>192.168.3.254</dhcpipaddress>
   <dhcpstatus>1</dhcpstatus>
   <dhcpendipaddress>192.168.3.254</dhcpendipaddress>
   <accessipaddress>172.16.255.254</accessipaddress>
   <dhcpleasetime>86400</dhcpleasetime>
</request>
```

`GET /api/dhcp/settings` で現在の設定を同じ形式で取得できます。 

`dhcpstatus` フィールドが DHCP の on / off フラグになっています。
このフィールドは管理画面の UI からは変更できず、常に 1 をセットして POST するようになっています。
これを DevTools で強制的に 0 に上書きして POST します。

## ハック手順

1. Air ターミナルの管理画面にログインし、「ネットワークの設定」 → 「IPアドレス/DHCPサーバの設定」を開く
   ![](/images/2024-05-02-20-59-43.png)
2. DevTools を開いて「ソース」タブの「検索」にて`api/dhcp/settings` で検索し、`saveAjaxData()`を呼び出している直前にブレークポイントを設置する
   ![](/images/2024-05-02-21-02-37.png)
3. 管理画面で「設定を保存する」をクリックする
4. 2 で設置したブレークポイントで処理が停止する。この間に「コンソール」で次のスクリプトを実行し、保存する設定を上書きする
   ```javascript
   dhcp_value.DhcpStatus = 0;
   ```
   ![](/images/2024-05-02-21-05-09.png)
5. ブレークポイントで止めていた処理を続行して保存を完了させる
   ![](/images/2024-05-02-21-07-47.png)
6. Air ターミナルを再起動する。`/api/dhcp/settings` にアクセスしたとき`dhcpstatus: 0` が返ってきていれば成功
   ![](/images/2024-05-02-21-11-40.png)

## ハック前後でどう変わったのか

### ハック前
スマートフォンを無線 LAN に繋げたとき LAN 内で行われていた DHCP 通信が次です。WireShark でキャプチャしました。
Air ターミナル (192.168.3.254) が DHCP NAK を返し、これを受信したクライアントは処理を最初からやり直しています。
こうして IP アドレスの取得に手間取っている様子が見てとれます。

![](/images/2024-05-02-21-17-21.png)


### ハック後
同じくスマートフォンを無線 LAN に繋げたときの DHCP 通信です。
Air ターミナルが応答しないことで一発で IP アドレスの取得ができています。

![](/images/2024-05-02-21-19-01.png)

この状態で、以前のように「インターネットなし」になることがなく快適に使用できています。

## おまけ: この方法を見つけるまでの経緯

Air ターミナルを AP として利用する中で、スマホに加え特にスマートホーム機器の接続がときどきできなくなることに悩まされていました。なにせ数が多いので、いずれかの接続が切れることはよく起こります。

Google 検索では解決方法が見つからず。ふとXを検索してみたところ次のようなポストを見つけました。

https://twitter.com/falms/status/1538749223184674816

この話の出典は見つかりませんでしたが、DevTools で Air ターミナルの管理画面を分析してみました。
すると DHCP 設定を保存する API `POST /api/dhcp/settings` で `DHCPStatus` というフィールドを常に1に設定して POST しているのを見つけました。
(Form の非表示項目ではなかったが、JS のコードで設定されていた。)

このフィールドが何を意味するのかは分かりませんでしたが、GitHub を検索してみたところ HUAWEI 製 LTE ルーターの 設定 API に同名のフィールドがあることを発見しました。

44 行目に `:param dhcp_status: Turn dhcp server on/off.` とあります。これで確信を得て、設定保存時に値を 0 に上書きしてみたところ DHCP サーバーが無効化されることを確認できました。


https://github.com/Salamek/huawei-lte-api/blob/aa92de42f3cd1065e642f56666c66debcff63340/huawei_lte_api/api/Dhcp.py#L27-L53