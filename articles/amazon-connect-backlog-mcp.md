---
title: "Amazon ConnectにBacklogを操作させてみた"
emoji: "😺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

Amazon Connect アドベントカレンダー 2025、12日目の記事です！

# はじめに
もうすぐ大学卒業で社会人に対してワクワクしているたくみです。
以前ラスベガスで開催されたAWS Re:Inventにて、多くのAmazon Connectアップデートが行われていました。
Amazon Connectをあまり使用したことない私ですが、Amazon Connect AIエージェントがMCPクライアントに対応したとのことなので、学習も含め触ってみました。

本記事ではAmazon Connect AIエージェントにBacklog MCPを統合させ、ナレッジの取得やチケット記帳などを自動化できる仕組みを作っています。

# アップデート内容
AWS Re:Inventにて、Amazon ConnectのMCPサポート開始というアップデートが発表されました。
これにより、Amazon Connectによるチャットボットサービスやコールセンターで、お客様との会話内容から伝票を起票する、顧客データを作成するなどの様々な自動化が可能となります。
https://aws.amazon.com/jp/about-aws/whats-new/2025/11/amazon-connect-mcp-support/

:::message alert
12/12時点でAmazon Connect MCPクライアントはオレゴンリージョン（us-west-2）でのみ対応しています。
:::

# やってみよう