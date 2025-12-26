---
title: "2025年を振り返って"
emoji: "🤖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false
publication_name: "uniformnext"
---

# はじめに
こんにちはたくみです。
あと少し寝るとお正月になってしまいます。2025年はとても時間の流れが早かったような気がします。

さて、年末なので2025年を振り返って何をしたのか、何ができなかったのかを考えてみようかなと思います。
日記を読むような感じで読んでいただけると幸いです。

# したこと
早速やったことをまとめてみました。

## 認証基盤刷新リリース
弊社が運用しているSaaSの認証基盤を一つにまとめるリリースを行いました。マイクロサービスアーキテクチャへの第一歩って感じです。
私は主にインフラ部分を担当し、AWS、Auth0を触っていました。

### TerraformでのAuth0管理

### AppRunnerからECS on Fargateへの置き換え

### Next.js on Lambdaへの置き換え

### サービス間通信にLatticeを使用

### 関連記事
https://zenn.dev/uniformnext/articles/2b45ce9712707b
https://zenn.dev/uniformnext/articles/graphql-federation

## 社内AIサービスのリリース
AWSが提供しているGenuを使用して社内向けAI RAGサービスをリリースしました。
私は主にインフラ部分を担当し、bedrock、bedrock knowledge base、s3 vectors、auth0を触っていました。

### Bedrock Knowledge BaseのベクトルデータベースにS3 Vectorsを使用

### 認証にCognitoではなくAuth0 SPAを使用

### Predict Stream関数をApi Gatewayに配置

## イベント参加
AWS SummitやCTOAの研修、JAWS UGなどの様々なイベントに参加しました。参加だけでなく登壇もすることができ、良い経験がたくさんできました。

### CTOA研修（ISUCON）

### AWS Summit 2025

### ZennCafe

### Jaws UG 新潟支部

### Jaws Festa

### Education Jaws

### 関連記事
https://zenn.dev/uniformnext/articles/aws-summit-2025
https://zenn.dev/uniformnext/articles/zenncafe-speaking
https://zenn.dev/uniformnext/articles/jaws-festa-2025

## 資格試験の受験
AWS認定試験やIPAの試験を受験しました。

### AWS認定試験

### IPA試験

## 社内イベント
内定者なのですが様々な社内イベントに参加しました。

### システムインターンシップ講師

### 長期インターンシップ管理

# できなかったこと
## AWS認定試験全冠

## AI エージェントを触る

## 社内ワークフローアプリのリリース
