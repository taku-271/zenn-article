---
title: "GraphQL Federation導入まとめ：APIモダナイズの最前線を行く"
emoji: "📑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [contest2024]
published: false
publication_name: "uniformnext"
---

# はじめに
こんにちはたくみです．

「今年最も大きなチャレンジ」ということで，RestAPIからGraphQLの移行や，認証基盤システムのインフラ構築，AppRouterのContainer／Presentation構成の移行など様々思い浮かびますが，一番困難だったのは「Graphql Federation」の導入かなと思います．
ただ，Graphql Federationを導入すると，今までのGraphQLには戻れない体になってしまい，マイクロサービスアーキテクチャの虜になってしまいました笑

この記事では実際に導入したときの手順と困難を紹介します．

誰かのお役に立てたら幸いです．

:::message
この記事では検証やデバッグの工程は省略しており，導入や結果のまとめとなっています．
:::

# 環境
### 全体
- Node.js v18.18.0
- npm v9.8.1
### Gatewayサーバー
- express v4.17.2
- Apollo Server v4.11.0
- Apollo Gateway v2.7.8

# 概要
### 従来
現在のSaaSプロダクトは，サービスごとに別れているマイクロサービスアーキテクチャとなっています．
しかし，APIサーバー（バックエンド）もサービスごとに別れてしまっており，サービス間のデータのやり取りで，複数のエンドポイントにリクエストを送っている感じになっていました．
:::details 従来のアーキテクチャ図（簡略）
```mermaid
graph
    A[フロントエンド] --> |リクエスト| B[APIサーバー1]
    A --> |リクエスト| C[APIサーバー2]
    style A fill:#CDE6C7,stroke:#CDE6C7
    style B fill:#FAA755,stroke:#FAA755
    style C fill:#FAA755,stroke:#FAA755
```
:::

マイクロサービスアーキテクチャをしているなら，APIサーバーをまとめ上げたほうが，エンドポイントが１つになり，フロントエンドから見るととてもわかり易くなりそうですよね．
また，APIサーバーが別れていることから，複数のサービスからデータを取得してきて，それをフロントエンドで合体させて使用する形になっていました．
データの取得や変形は極力バックエンドに任せたいところなのですが，サーバーが別れているためどうしようにもありませんでした…

### GraphQL Federation
ここで出てきたのが「GraphQL Federation」という技術です．
GraphQL Federationは，Gatewayサーバーを用意しておき，フロントエンドからはGatewayサーバーのエンドポイントを叩くことで，Gatewayサーバーがバックエンドにリクエストを伝搬してデータを返してくれると言ったものになります．
また，それだけでなく，クエリによって各サービスのデータを合体させて返すため，フロントエンド側での処理も少なくなります！
:::details GraphQL Federationのアーキテクチャ図（簡略）
```mermaid
graph
    A[フロントエンド] --> |リクエスト| B[Gatewayサーバー]
    B --> |リクエスト| C[APIサーバー1]
    B --> |リクエスト| D[APIサーバー2]
    style A fill:#CDE6C7,stroke:#CDE6C7
    style B fill:#65BBE9,stroke:#65BBE9
    style C fill:#FAA755,stroke:#FAA755
    style D fill:#FAA755,stroke:#FAA755
```
:::

この技術を使用すれば，従来の困っていた点を解決させることができるため，導入の流れになりました．

https://www.apollographql.com/docs/graphos/schema-design/federated-schemas/federation

# 導入の流れ（手順）
## 1. APIサーバーのGraphQL化
今までのAPIサーバーでは，GraphQLで作成されているものとREST APIで作成されているものが混ざり合っていました．
このままでは，GraphQL Federationを導入することができないため，まずはAPIサーバーをGraphQL化する必要があります．
ここのGraphQL化は，使用している言語やフレームワークによって異なるので，それぞれの公式ドキュメントを参考にしてください．

## 2. GraphQL Federation Gatewayサーバーの作成
すべてのAPIサーバーをGraphQL化したら，早速Federation用のGatewayサーバーを作成します．
ここで，GatewayサーバーはApollo ServerとApollo Gatewayを使用しました．
https://www.apollographql.com/docs/apollo-server
https://www.apollographql.com/docs/federation/v1/gateway

### 2-1. 各モジュールのインストール
```bash
npm install @apollo/server @apollo/gateway express cors http
npm install -D nodemon ts-node
```

### index.tsの作成
`index.ts`にGatewayサーバーを実装していきます．
:::details コード例
```typescript:./src/index.ts
import http from "http";

import { ApolloGateway, IntrospectAndCompose } from "@apollo/gateway";
import { ApolloServer } from "@apollo/server";
import { expressMiddleware } from "@apollo/server/express4";
import { ApolloServerPluginDrainHttpServer } from "@apollo/server/plugin/drainHttpServer";
import cors from "cors";
import express from "express";

const app = express();
const httpServer = http.createServer(app);

const serviceList = [
    { name: "service1", url: "http://localhost:4001/graphql" },
    { name: "service2", url: "http://localhost:4002/graphql" },
];

const gateway = new ApolloGateway({
    supergraphSdl: IntrospectAndCompose({
        serviceList,
    }),
})

const server = new ApolloServer({
    gateway,
    plugins: [ApolloServerPluginDrainHttpServer({ httpServer })],
});

(async () => {
    await server.start();

    app.use(
        "/graphql",
        cors<cors.CorsRequest>();
        express.json(),
        expressMiddleware(server, {
            context: async ({ req }) => {
                return { headers: req.headers };
            },
        })
    );

    await new Promise<void>((resolve) => httpServer.listen({ port: 4000 }, resolve));
    console.log("Server ready at 4000 port");
})();
```
:::
