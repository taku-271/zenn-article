---
title: "Amazon ConnectでMCP操作してみた"
emoji: "😺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [aws, amazon connect, mcp, ai]
published: false
publication_name: "uniformnext"
---
Amazon Connect アドベントカレンダー 2025、12日目の記事です！

クラスメソッドさんとギークフィードさん、ユニフォームネクスト、AWS Japanさんの有志が募ってチャレンジしている企画になります。

(アドベントカレンダーのカレンダー一覧はこちら↓)
https://qiita.com/advent-calendar/2025/amazon-connect

# はじめに
もうすぐ大学卒業で社会人に対してワクワクしているたくみです。
以前ラスベガスで開催されたAWS Re:Inventにて、多くのAmazon Connectアップデートが行われていました。
Amazon Connectをあまり使用したことない私ですが、Amazon Connect AIエージェントがMCPクライアントに対応したとのことなので、学習も含め触ってみました。

本記事ではAmazon Connect AIエージェントにAWS knowledge MCPを統合させ、AWSのナレッジ取得ができる仕組みを作っています。

# アップデート内容
AWS Re:Inventにて、Amazon ConnectのMCPサポート開始というアップデートが発表されました。
これにより、Amazon Connectによるチャットボットサービスやコールセンターで、お客様との会話内容から伝票を起票する、顧客データを作成するなどの様々な自動化が可能となります。
https://aws.amazon.com/jp/about-aws/whats-new/2025/11/amazon-connect-mcp-support/

:::message alert
2025/12/12時点でAmazon Connect MCPクライアントはオレゴンリージョン（us-west-2）でのみ対応しています。
:::

# やってみよう
事前に、Amazon Connect MCPクライアントが対応しているリージョンで、Amazon Connectインスタンスを作成しておきます。

## Bedrock AgentCore Gatewayの作成
Amazon Connectで使用するMCPサーバーはBedrock AgentCore Gatewayでホストする必要があります。
そのため、まずはBedrock AgentCore Gatewayを作成します。

`AWSマネジメントコンソール > Bedrock AgentCore > ゲートウェイ`から作成します。

適当な名前を設定し、`インバウンド認証設定`は検証なので`No authorization`にしておきます。
![](/images/amazon-connect-mcp/image.png)
:::message
本来はJWT認証かIAM認証をすることをおすすめします。
:::
ターゲットとしてAWS knowledge base MCPサーバーを設定します。`MCPエンドポイント`に好きなMCPサーバーのエンドポイントを設定することができます。今回はAWS knowledge MCPサーバーのエンドポイントを設定しています。
![](/images/amazon-connect-mcp/image2.png)

## Amazon Connectのサードパーティーのアプリケーションの作成
Amazon Connect サードパーティーのアプリケーションで先ほど作成したBedrock AgentCore Gatewayにアクセスします。
`AWSマネジメントコンソール > Amazon Connect > サードパーティーのアプリケーション`からサードパーティーのアプリケーションを作成します。

適当な名前を設定し、`Application type`を`MCP Server`に設定します。`Application details`で先ほど設定した`Bedrock AgentCore Gateway`を設定します。また、`インスタンスの関連付け`で用意したインスタンスを設定しておきます。
![](/images/amazon-connect-mcp/image3.png)

## Amazon Qの有効化
`AWSマネジメントコンソール > Amazon Connect > インスタンス > 用意したインスタンス > Amazon Q`で`Add Domain`のボタンからAmazon Qのドメインを作成します。
![](/images/amazon-connect-mcp/image4.png)

## サードパーティーのアプリケーションを使用するAmazon Connect AIエージェントを作成
ここからはAmazon Connectの画面から設定を行います。
`AWSマネジメントコンソール > Amazon Connect > インスタンス > 用意したインスタンス > 概要`の`Access Url`からAmazon Connectにアクセスし、ログインします。
![](/images/amazon-connect-mcp/image5.png)

`AIエージェントデザイナー > AIエージェント > AIエージェントを作成`からAIエージェントを作成します。
![](/images/amazon-connect-mcp/image6.png)

`AIエージェントタイプ`を`Orchestration`、`Copy from existing`を`AgentAssistantOrchestrator`でAIエージェントを作成します。
![](/images/amazon-connect-mcp/image7.png)

`ロケール`を`Japanese`にし、`Security Profile`を設定したいセキュリティプロファイルに設定してください。ここでは`Admin`で設定します。
![](/images/amazon-connect-mcp/image8.png)

`Add Tool`ボタンをクリックし、先ほど作成したサードパーティーのアプリケーションを設定します。その際、使用したいツールも一緒に設定します。
![](/images/amazon-connect-mcp/image9.png)

その後、保存し公開します。

## セキュリティプロファイルにMCPを呼び出す権限を付与する
`ユーザー > セキュリティプロファイル`から先ほど設定したセキュリティプロファイルを選択します。
![](/images/amazon-connect-mcp/image10.png)

`Tools`から先ほど追加したMCPツールにチェックを入れ、保存します。
![](/images/amazon-connect-mcp/image11.png)

## フローを作成
では、先ほど作成したAIエージェントを使用するフローを作成します。
`フロー > フローを作成`からフローを作成します。
![](/images/amazon-connect-mcp/image12.png)

フローを以下のように設定します。なお、`Contact Assistant`の設定として、`ドメインを選択`にAmazon Qを有効化した時のドメイン、`AIエージェント`に先ほど作成したAIエージェントを設定します。
![](/images/amazon-connect-mcp/image13.png)
![](/images/amazon-connect-mcp/image14.png)

作成したフローを公開し、電話番号の`コンタクトフローIVR`に設定します。
![](/images/amazon-connect-mcp/image15.png)

以上で設定完了です！

## 使ってみる
コンタクトフローを設定した電話番号に対して電話をかけ、受電するとAmazon QからMCPツールを使用できます！
Lambdaについて聞くと、AWS Knowledge MCPより情報を検索し、応答が返ってきています。
![](/images/amazon-connect-mcp/image16.png)

# まとめ
本記事ではAmazon Connectを使用してMCPツールを呼び出せることを紹介しました。
今回使用したMCPツールはAWS Knowledge MCPなのであまり使い所ありませんが、独自SaaSのMCPサーバーを作成しAmazon Connectと連携させる、backlogやNotionのMCPをリモートホストして連携させるなどのことが可能です。
ぜひ使用してみてください。

では、良いAmazon Connectライフを！

# 参考
https://docs.aws.amazon.com/connect/latest/adminguide/3p-apps-mcp-server.html
