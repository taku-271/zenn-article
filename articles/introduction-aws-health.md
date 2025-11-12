---
title: "AWS Healthが有能だった件"
emoji: "❤️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [aws, claude]
published: true
publication_name: "uniformnext"
---
# はじめに
大学で卒論の中間発表が終わり、いよいよ卒業が近づいてきました。悲しさと社会人への期待が入り混じった複雑な心境のたくみです。

最近、AWS環境での開発中にAWS Healthの有用性を実感する機会がありました。
AWS認定試験（CLF、SAA）の勉強時には「こんなサービス使わないだろう」と思っていた過去の自分に、今すぐにでも教えてあげたい気持ちです。

# AWS Healthとは
AWS HealthではAWS上で発生する重大なイベントを確認することができます。
例えば、**リージョン、サービスの障害や破壊的なサービスの変更、廃止**などを確認することができ、メール送信やEvent Bridgeのネイティブ統合によりAWS障害にいち早く気がつくことができます。
ちなみに無料です。

### AWS Health Event
> AWS Health イベントは、ヘルスイベントとも呼ばれ、 AWS Healthが他の AWS サービスに代わって送信する通知です。これらのイベントを使用して、アカウントに影響する可能性のある今後の変更や予定された変更について知ることができます。
> https://docs.aws.amazon.com/ja_jp/health/latest/ug/aws-health-concepts-and-terms.html#aws-health-events

### AWS Health Dashboard
> AWS Health ダッシュボードを使用して、リージョン内のサービスの今後のメンテナンスの問題など、一般的な認識を提供するイベントについて確認することをお勧めします。 AWS Health ダッシュボードを使用して、アカウントの廃止されたリソースなど、ユーザーに直接影響する可能性のあるイベントについて確認することもできます。
> https://docs.aws.amazon.com/ja_jp/health/latest/ug/aws-health-concepts-and-terms.html#aws-health-dashboard-term

### AWS Health API
> APIを使用してAWS Health Dashboardに表示される情報にプログラムでアクセスできます。
> https://docs.aws.amazon.com/ja_jp/health/latest/ug/aws-health-concepts-and-terms.html#aws-health-api

# 気づいた瞬間
弊社では社内でRAGアプリを導入しています。このRAGアプリは[Generative AI Use Cases（GeuU）](https://aws-samples.github.io/generative-ai-use-cases/ja/index.html)をベースとしてAWS環境上で動いていました。

先日、チャットの返答が返ってこないというお困りごとが発生しました。

何も変えていないはずなのにおかしい...
bedrockを叩いているlambdaはタイムアウトで落ちている...
どうもbedrockへリクエストしている部分でレスポンスが返ってきていない...
なぜだ...

ぼけーとマネジメントコンソールを見ていると、右上に通知バッチ（AWS Health Event）が付いており、その中の一つにbedrockとの文言を見つけました。
なんかこんなのも有ったなーと思いながら詳しく見てみると、現在**RAGアプリで使用しているモデル（claude 3.7 Sonnet）がレガシーモデルになる**とのイベントが上がっていました。

**「これかーーー！こいつめっちゃ有能だろ！！」**

と、安心する気持ちと同時に有能さを認識しました。今までバカにしていてごめんなさい。今後の登壇での好きなサービスはこれにします。

# 最後に
AWSで無駄なサービスなんてないなと実感し、改めてAWSの便利さを実感しました。
AWSの障害、重大な変更が掲載されるため、このイベントをSlackやChatworkに送信したいくらいです。

皆さんもAWS Healthを愛でましょう。

# 参考
https://health.aws.amazon.com/health/status
https://docs.aws.amazon.com/ja_jp/health/latest/ug/aws-health-concepts-and-terms.html
