---
title: "Powershellスクリプトを「ファイル名を指定して実行」で即座に実行する"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Powershell", "Windows"]
published: false
---

# はじめに
Windowsで操作を自動化したいときはPowershellスクリプトを組むのが便利です。  
作成したスクリプトをいつでもどこでもすぐに呼び出すため、`Windows + R`キーで開く「ファイル名を指定して実行」から呼び出せるようにしました。

# したいこと
1. `Windows + R`キーで「ファイル名を指定して実行」ダイアログを開く
2. スクリプト名を入力してEnter、スクリプトが実行される

[![Image from Gyazo](https://i.gyazo.com/3e44ac35dea93d5f5353248809004e86.gif)](https://gyazo.com/3e44ac35dea93d5f5353248809004e86)

## スクリプト内容
メッセージダイアログを開き「Hello World!」と表示するだけのシンプルなスクリプトです。
```Powershell
Add-Type -AssemblyName System.Windows.Forms;
[System.Windows.Forms.MessageBox]::Show("Hello World!", "Powershellスクリプト");
```

# 手順
## 最終状態の説明
最終的に以下の設定ができていれば完了です。
1. Powershellの実行ポリシーでPowershellスクリプトの実行が許可されている
2. 実行したいPowershellスクリプトが用意されている
3. 2のスクリプトを実行するショートカットが用意されている
4. 3のショートカットが配置されているフォルダにPATHが通っている

## 1. Powershellの実行ポリシーを設定する
Powershellスクリプトはデフォルトで実行が制限されています。
> * [実行ポリシーについて - PowerShell | Microsoft Docs](https://docs.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.1)

Powershellを開き、`Get-ExecutionPolicy`コマンドを実行します。  
"Restricted" と表示された場合は変更が必要です。`Set-ExecutionPolicy RemoteSigned`コマンドを実行し、Powershellスクリプトの実行を許可します。  
すでに "Restricted" 以外に設定されている場合は変更不要です。

## 2. 実行したいPowershellスクリプトを用意する
Powershellスクリプトを用意し、任意の場所に配置します。  
今回は"HelloWorld.ps1"という名前でスクリプトを用意しました。
![](/images/2021-09-25-20-20-39.png)

## 3. スクリプトを実行するショートカットを用意する
2のフォルダにPATHを通すだけでは、「ファイル名を指定して実行」から実行できません。  
Powershellスクリプトはダブルクリックで安易に実行できないよう、デフォルトでは「メモ帳」で開くよう設定されているからです。  
「ファイル名を指定して実行」で "HelloWorld.ps1" を実行すると、メモ帳でファイルの中身が表示されるだけです。  

そこで、Powershellスクリプトを実行するショートカットを作成します。

1. Powershellスクリプトを選択し、右クリックで「ショートカットの作成」を選択する
2. 作成されたショートカットの名前を分かりやすく入力しやすいものに変更する(拡張子不要)。この名前が「ファイル名を指定して実行」で入力する名前になる
3. ショートカットを右クリックし、「プロパティ」を開く
4. 「リンク先」を編集する。既存のリンク先の値（Powershellスクリプトのフルパス）をダブルクオテーションで囲い、その前に半角スペース空けて 'powershell' を付加する。
    * ※ Powershell 6以降がインストールされている場合は 'pwsh' の利用を推奨
    * 入力例(pwshの場合):  
    `pwsh "C:\Repos\Scripts\PowerShell\HelloWorld.ps1"`
5. "OK" で保存

ショートカットの置き場所はPowershellスクリプトと異なる場所でもOKです。  
私の場合、最終的に以下の状態になりました。
![](/images/2021-09-25-20-43-20.png)

## 4. ショートカットへのPATHを通す
3で用意したショートカットの置き場にPATHを通します。  
PATHを通すのは基本中の基本なのでやり方がわからない方は各自検索してください。

## 設定完了！
ここまで設定すれば目的達成です。  
「`Windows + R`」 → 「3で用意したショートカット名を入力」 → 「Enter」 で、いつでもどこでもスクリプトの実行が可能になりました。 

私は複数のPCでスクリプトを使いまわしたいので、スクリプトとショートカットの両方をGoogle Driveに置き、それぞれのPCでショートカット置き場にPATHを通しています。  
Powershellで退屈なタスクをどんどん自動化していきましょう。  
