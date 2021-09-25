---
title: "クリップボードの画像データをファイルとして保存するPowerShellスクリプト(Windows限定)"
emoji: "🖼️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Powershell", "Windows"]
published: false
---

# はじめに
Windows PCを使っている方にとって、`Win + SHIFT + S` でスクリーンショットを撮る操作はおなじみです。  
撮ったスクリーンショットはクリップボードに保持されます。  
そのままペーストできる場合はいいのですが、ときどきペーストを受け付けてくれず、画像ファイルのアップロードを求められる場合があります。  
そんなときにPaintなどを開いてペースト→保存するのは煩わしいので、Powershellスクリプトで即座に保存できるようにしてみました。  

# できたもの
スクショを撮ったあと、Win + R で開く「ファイル名を指定して実行」からスクリプトを実行するとデスクトップに画像ファイルが保存されます。

[![Image from Gyazo](https://i.gyazo.com/43247772cf81adabdc4d3af7704acddf.gif)](https://gyazo.com/43247772cf81adabdc4d3af7704acddf)


# Powershellスクリプトの中身

```SaveClipboardImage.ps1

```

