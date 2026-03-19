---
title: "Next.jsをLambdaで構築したくないですか？"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
publication_name: "uniformnext"
---
# はじめに
最近ちゃんとしないとなーと実感しているたくみです。

現在開発しているSaaSのアーキテクチャでNext.jsのホストにLambdaを使用してみて、思った以上に良かったので紹介します。

# Next.jsをLambdaで構築
## Lambda Web Adapter
さて、Lambdaといえばハンドラーを定義してそのコードを実行するものですが、どうやってサーバーをホストのでしょうか？
実はLambdaにはLambda Web Adapter（LWA）というライブラリが存在しており、これを使用することでホストするウェブアプリケーションをLambdaで実行することができます。
https://aws.amazon.com/jp/builders-flash/202301/lambda-web-adapter/

### 導入の流れ

## インフラコード
### アプリケーション

### CI/CD

## Next.jsの設定

## 問題点

#　評価

# 最後に