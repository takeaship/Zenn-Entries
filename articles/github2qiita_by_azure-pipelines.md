---
title: "GitHubで執筆したエントリーをAzure PipelinesでQiitaに投稿する"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Azure Pipelines", "Qiita", "Markdown"]
published: false
qiita_id: ""
---

Zennと同様に、GitHubにpushしたエントリーをQiitaに投稿できる仕組みがほしいので作ってみました。  
Qiita上でどう表示されるかを、執筆途中でPreviewすることはできません。  
意図したものと異なる表示で投稿される場合があります。ご了承下さい。  

# Azure Pipelinesとは
Microsoftのプロジェクト管理ツールAzure DevOpsに含まれるCI/CDパイプライン構築ツールです。  
CI/CD 用のMicrosoft ホステッド ジョブを、1,800分/月まで無料で使えます。  

# GitHubとAzure Pipelinesを連携する