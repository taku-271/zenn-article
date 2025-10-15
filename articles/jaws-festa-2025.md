---
title: "JAWS FESTA 2025 in 金沢へ行ってきました！"
emoji: "👏"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [jawsug, jaws, jaws festa 2025, aws, event]
published: false
publication_name: "uniformnext"
---
# はじめに
最近、情報処理安全確保支援士を受験し、イベントに参加し、大学の中間発表の準備をしている多忙なたくみです。

10/11（土）に開催されたJAWS FESTA 2025 in 金沢に参加してきましたので、学んだことや思ったことをアウトプットします！
みんなもコミュニティの大きいイベントに参加したくなるような記事を目指します。
https://jawsfesta2025.jaws-ug.jp/

:::message
この記事では能登地震を取り上げます。苦手であれば閲覧をご遠慮ください。
:::

# JAWS FESTAとは？
## JAWS-UGとは
JAWS-UGとは、AWS User Group – Japanの略で、主にAWSを題材としたコミュニティになります。各都道府県やカテゴリ（ストレージや学生など）の様々な支部が全国にあり、いつも多様なイベントを開いています。
基本的にはどなたでも参加できるコミュニティで、つよつよエンジニアも多く参加しているため、参加することで良いインプット、アウトプットになります。
https://jaws-ug.jp/about-us/

## JAWS FESTAとは
JAWS FESTAとは、毎年秋ごろに開催されている、地方で行われる大きなイベントです。
去年は広島で今年は金沢となりました。参加人数は300人超えの大規模イベントであり、多数のセッション、ゲームが開かれています。

# 知ったきっかけ 
私がこのイベントを知ったきっかけは、XでJAWS-UGのイベント参加したいとポストしたことでした。このポストでJAWS FESTAのことを教えてくださり、金沢なら近いし、絶対応募しよう！となりました！
実は、このイベント、当日で募集枠が埋まってしまうほどの人気を誇っており、応募できるか不安でした。しかし、学生枠が設けてあり、なんとか応募することができました。

# 当日
当日は近江町市場近くのビルでイベントが進みました。

## 開会式
まずは開会式として、金沢支部運営メンバーであり、JAWS FESTA実行委員長である、加藤さんからの諸注意や開会宣言が行われていました。
多目的ホールのような場所で行われていましたが、人が多すぎて席はほぼ埋まっており、後ろでの立ち見の方も多数いらっしゃいました。
![](/images/jaws-festa-2025/jaws-festa.jpeg)

## セッション
開会式後は様々な方によるセッションが行われていました。

### 能登半島地震で見えた災害対応の課題と組織変革の重要性
このセッションは石川県前副知事の方が、能登半島地震の際にどのような部分で困ったのか、どのようにITを利用したのかを説明してくださっていました。
特に、被災者の方々の情報をどのように名寄せしてデータベースに統合し、管理するのか、その際に自衛隊の方との協力をした話、やはり県職員の方々に理解してもらう時間がかかってしまう話など、普段聞かないDRのお話を聞け、とても興味深かったです。
https://speakerdeck.com/ditccsugii/neng-deng-ban-dao-di-zhen-dejian-etazai-hai-dui-ying-noke-ti-tozu-zhi-bian-ge-nozhong-yao-xing?slide=
https://jawsfesta2025.jaws-ug.jp/sessions/timetable/166/

### JAWS-UGの災害支援と石川県の災害を振り返る - エンジニアクロストーク
このセッションではJAWS-UGの過去の災害支援と、石川県のエンジニアが能登半島地震の際にどのような行動を取ったのかというお話をしてくださっていました。
JAWS-UGの過去や現在の状況などがわかり、とても面白かったです。年間のイベント数が530回程度で、すご！となりました笑。
また、DRはまだブルーオーシャンであり、地域貢献に直結する会社になるため、需要が上がるのではないかとのお話もあり、確かに！となりました。
https://speakerdeck.com/ditccsugii/neng-deng-ban-dao-zai-hai-xian-chang-enziniakurosutoku-jaws-festa-2025-in-jin-ze
https://jawsfesta2025.jaws-ug.jp/sessions/timetable/101/

### スタートアップで高速検証するためのAmplify Gen 2 〜利便性と、ハマるS3認可や一覧画面実装の解決テンプレート〜
このセッションではAWS Jr. Championsである[エイミ](https://x.com/amixedcolor)さんによる、Amplify Gen 2の概要や使い方、はまりどころをお話ししてくださっていました。
特に、スタートアップのプロダクトにとってプロトタイプの速度がとても大事になってきており、そのような場面でAmplifyが活躍するのではないかとの部分が確かに！となりました。
Amplify自体、あまり使用したことがないため、使ってみようかなと思います。
https://speakerdeck.com/amixedcolor/sutatoatupudegao-su-jian-zheng-surutamenoamplify-gen-2-li-bian-xing-to-hamarus3ren-ke-ya-lan-hua-mian-shi-zhuang-nojie-jue-tenpureto
https://jawsfesta2025.jaws-ug.jp/sessions/timetable/77/

### Amazon Verified Permissions実践入門 〜Cedar活用とAppSync導入事例
このセッションでは、Amplify + App Syncを使用している際の認可をマネージドにやろうといった内容で、認可を別でまとめることができるのが強いなと思いました。
個人的にはOpen FGAも気になっており、これと比べてみたいなとも思いました。
https://speakerdeck.com/fossamagna/practical-introduction-to-amazon-verified-permissions
https://jawsfesta2025.jaws-ug.jp/sessions/timetable/232/

### 地方だからできる！コミュニティ参加と登壇を続ける意義
このセッションでは、登壇者である[seike](https://x.com/seike460)さんが今までどのような経緯で地方でのイベントに参加し、どのようにして現在の状況になれたのかを説明されてました。
僕も地方の人なので、タイトルを見た瞬間これは見なければとなりました笑。
特に、コミュニティは熱を広げてくれる、10年後のあなたは何をしていますか？という部分がとても面白く、憧れました。イベント参加頑張ります。
https://docs.google.com/presentation/d/1s3CBgDUqPKVYzw_qcpmusvogKPhDxrCR9BF6iimDC5Q/edit?slide=id.g38bb8eba3b9_2_1#slide=id.g38bb8eba3b9_2_1
https://jawsfesta2025.jaws-ug.jp/sessions/timetable/88/

### なぜAWSを活かしきれないのか？技術と組織への処方箋
このセッションではAWSの試験本を執筆している方のセッションで、AWSを有効活用できる組織形態の話をされていました。
文化の構築、行動との関係、ルールを設ける方法などと、組織形成の話がとても参考になりました。
https://speakerdeck.com/nrinetcom/nazeawswohuo-kasikirenainoka-ji-shu-tozu-zhi-henochu-fang-jian
https://jawsfesta2025.jaws-ug.jp/sessions/timetable/200/

### セキュアな認可付きリモートMCPサーバーをAWSマネージドサービスでつくろう！
このセッションでは、リモートMCPサーバーに対する認可をAWSマネージドサービスで作るといった内容で、MCPサーバーの運用を考えている身として絶対見ないとなと思ったセッションでした。
特に好きなRFCの流れがとても面白く、認証認可がとても好きな方なんだなと感じました。
内容も、できないならばどうするのかなどの順序が示されており、MCP認可だけでなく思考方法としても見れるセッションだったと思います。
https://speakerdeck.com/kaminashi/lets-build-an-oauth-protected-remote-mcp-server-based-on-aws-managed-services
https://jawsfesta2025.jaws-ug.jp/sessions/timetable/93/

### コラボセッション 〜地元の若手とJr.Championsで活動の原点を語る〜 
実は一番楽しみにしていました！このセッションは北陸のエンジニアの方々とAWS Jr. Championsの方々とのクロスセッションで、なぜエンジニアになったのか、なぜAWS Jr. Championsをとったのか、イベントに参加した経緯などを話されていて、自分も見習わないとな！と思うことが多くありました！
質疑応答では僕がコメントした、コミュニティで話せないというコメントを拾ってくださって、とりあえず「こんにちは！」と話しかけてみることが大事だと再実感しました！積極的に話しかけに行きます！
https://jawsfesta2025.jaws-ug.jp/sessions/timetable/81/

## 設営
以上、セッションでしたが、各企業様の設営などもありました！
AWS Japan様の設営では、Builder Centerの説明と、BuilderCardsの配布も行っていました！会社のメンバーと是非やってみたいと思います！
クラスメソッド様ではDeveloper塩をプレゼントしてたり、くらにゃんキラキラシールやZenn Tシャツを配布していて、ZennCafeから欲しかったものが手に入りました！

喫茶店やワークショップ、AIによる試着などもありましたが、いろいろバタバタしていていけず...次回以降のイベントでは積極的に参加してみます！

## 閉会式
閉会式では、再度加藤さんが閉会の言葉とアンケートの記入を実施されていました。また、次のJAWS Daysの告知もされていて、絶対行きたい！となりました！
https://jawsdays2026.jaws-ug.jp/

## 懇親会
なんだかんだ一番楽しかった懇親会でした！
会場はなんと、ANAホリデイ・イン金沢スカイの高いところ！高級そうな場所でリッチな気持ちになりました笑。
![](/images/jaws-festa-2025/konsinkai.jpeg)
![](/images/jaws-festa-2025/konsinkai2.jpeg)

ここでは様々なエンジニアの方とお話することができ、様々な意見をいただきました！
福井の企業の方もおり、福井でもイベントしたいねーなどの話で盛り上がりました！
また、backlog world 2024の運営委員長の方ともお会いでき、弊社の[backlog award 2024](https://backlog.com/ja/blog/backlog-world-2024-gpa/)の発表についてお話しできました。
ご飯もとてもおいしかったです。
![](/images/jaws-festa-2025/all.png)

# 最後に
次の日にも能登に行くツアーや金沢を回るツアーなど開催されておりましたが、情報処理安全確保支援士を受けるため、やむなし福井に帰りました...
本当は二次会とか行きたかったので、次回以降は行きます！

以上、とても良いインプットになる場でした！今後はこのような場でもセッションしてみたいなとも思いました。
とてもモチベーションが上がったところで、今月末に開催されるEducation-JAWSのLTに登壇者として申し込みました笑。この調子でアウトプットも積極的にやります！
https://education-jaws.connpass.com/event/366944/

今後もJAWSだけでなく、いろいろなイベントやコミュニティに参加して、様々なインプット、アウトプットを行っていこうと思います！