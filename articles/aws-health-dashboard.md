---
title: "AWS Health Dashboard（AWS Health Event）が有能だった件"
emoji: "❤️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [aws, claude]
published: false
publication_name: "uniformnext"
---
# はじめに
大学で卒論の中間発表が終わり、いよいよ大学も卒業かと悲しい気持ちと、社会人が楽しみな気持ちが混ざった複雑なたくみです。

最近、AWSを触っている中でAWS Health Dashboardがとっても有能だったので紹介します。
CLF、SAAの勉強してる時、「こんなん使わんわ」と思っていた過去の自分をフルマラソン並みに助走つけて殴りに行こうと思います。

# AWS Health Dashboard
> AWS Health Dashboard – Service Health を使用して、すべてのヘルスを表示できます AWS のサービス。このページには、 AWS リージョンにわたるサービスについて報告されたサービスイベントが表示されます。 AWS Health ダッシュボード – サービスのヘルスページにアクセスするには AWS アカウント 、サインインしたり、 を持っている必要はありません。

https://docs.aws.amazon.com/ja_jp/health/latest/ug/aws-health-dashboard-status.html

このサービスはAWS上で発生する重大なイベントを確認することができます。
例えば、リージョン、サービスの障害や破壊的なサービスの変更、廃止などを確認することができ、メール送信やEvent Bridgeのネイティブ統合によりAWS障害にいち早く気がつくことができます。
ちなみに無料です。（というかデフォルトで有効化されてそう。）

# 気づいた瞬間
弊社では社内でRAGアプリを導入しています。このRAGアプリはgenuをベースとしてAWS環境上で動いていました。
https://aws-samples.github.io/generative-ai-use-cases/ja/index.html

先日、チャットの返答が返ってこないというお困りごとが発生しました。
何も変えていないはずなのにとてもおかしい...
bedrockを叩いているlambdaはタイムアウトで落ちている...なぜだ...
ぼけーとマネジメントコンソールを見ていると、右上に通知バッチが付いており、その中の一つにbedrockとの文言を見つけました。
なんかこんなのも有ったなーと思いながら詳しく見てみると、現在RAGアプリで使用しているモデル（claude sonnet 3.7）がレガシーモデルになるとのイベントが上がっていました。

「これかーーー！！こいつめっちゃ有能だろ！！」

と、安心する気持ちと同時に有能さを認識しました。今までバカにしていてごめんなさい。

と、いうことがあり、AWS Health Dashboardの有能さに気づきました。今後の登壇での好きなサービスはこれにします。

# 最後に
AWSサービスで無駄なやつなんてないなと実感し、改めてAWSの便利さを知りました。
AWSに関する重大な変更が掲載されるため、このイベントをSlackやChatworkに送信したいくらいです。

皆さんもAWS Health Dashboardを愛でましょう。
