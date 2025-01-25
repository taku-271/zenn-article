---
title: "GraphQL Federation：APIモダナイズの最前線"
emoji: "📑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [contest2024, graphql, federation, microservice, backend]
published: true
publication_name: "uniformnext"
---

# はじめに

こんにちは，たくみです．

「今年最も大きなチャレンジ」ということで，RestAPI から GraphQL の移行や，認証基盤システムのインフラ構築，AppRouter の Container／Presentation 構成の移行など様々思い浮かびますが，一番困難だったのは「GraphQL Federation」の検証かなと思います．
ただ，GraphQL Federation を理解すると，今までの GraphQL には戻れない体になってしまい，マイクロサービスアーキテクチャの虜になってしまいました笑

この記事では実際に検証したときの手順と困難を紹介します．

誰かのお役に立てたら幸いです．

:::message
この記事では検証やデバッグの工程は省略しており，検証（導入）のまとめとなっています．
あと長いです．読み飛ばすなり，ほしい部分だけを読むなりしてください．
:::

# 環境

### 全体

- bun v1.1.42

### Gateway サーバー

- express v4.17.2
- Apollo Server v4.11.0
- Apollo Gateway v2.7.8

### 各 API サーバー

- express v4.17.2
- Apollo Server v4.11.0

# 概要

### 従来

現在の SaaS プロダクトは，サービスごとに分かれているマイクロサービスアーキテクチャとなっています．
しかし，API サーバー（バックエンド）もサービスごとに別れてしまっており，サービス間のデータのやり取りで，複数のエンドポイントにリクエストを送っている感じになっていました．
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

マイクロサービスアーキテクチャをしているなら，API サーバーをまとめ上げたほうが，エンドポイントが１つになり，フロントエンドから見るととてもわかり易くなりそうですよね．
また，API サーバーが別れていることから，複数のサービスからデータを取得してきて，それをフロントエンドで合体させて使用する形になっていました．
データの取得や変形は極力バックエンドに任せたいところなのですが，サーバーが別れているためどうしようにもありませんでした…

### BFF (Backend For Frontend)

マイクロサービスアーキテクチャを進めるために，フロントエンドからバックエンドにリクエストを送る際に中間層（BFF）を設ける技術も出てきました．
:::details BFF のアーキテクチャ図（簡略）

```mermaid
graph
    A[フロントエンド] --> |リクエスト| B[BFF]
    B --> |リクエスト| C[APIサーバー1]
    B --> |リクエスト| D[APIサーバー2]
    style A fill:#CDE6C7,stroke:#CDE6C7
    style B fill:#65BBE9,stroke:#65BBE9
    style C fill:#FAA755,stroke:#FAA755
    style D fill:#FAA755,stroke:#FAA755
```

:::
BFF を設けることで，API の集約ができます！また，認証やキャッシュなどの処理を置くことで，フロントエンドとバックエンドをまたがる処理をまとめる事ができます！
しかし，ルーティングの設定や設計が難しくなってしまいます．

### GraphQL Federation

ここで出てきたのが「GraphQL Federation」という技術です．
GraphQL Federation は，Gateway サーバーを用意しておき，フロントエンドからは Gateway サーバーのエンドポイントを叩くことで，Gateway サーバーがバックエンドにリクエストを伝搬してデータを返してくれると言ったものになります．
また，それだけでなく，クエリによって各サービスのデータを合体させて返すため，フロントエンド側での処理も少なくなり，BFF であったルーティングの設定などもよしなにやってくれます！
:::details GraphQL Federation のアーキテクチャ図（簡略）

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

# 構築の流れ（手順）

## 1. 各 API サーバーの準備

### 1-1．API サーバーの GraphQL 化

今までの API サーバーでは，GraphQL で作成されているものと REST API で作成されているものが混ざり合っていました．
このままでは，GraphQL Federation を導入することができないため，まずは API サーバーを GraphQL 化する必要があります．
ここの GraphQL 化は，使用している言語やフレームワークによって異なるので，それぞれの公式ドキュメントを参考にしてください．

### ex） Express.js

https://graphql.org/graphql-js/running-an-express-graphql-server/

### ex） NestJS

https://docs.nestjs.com/graphql/quick-start

### 1-2．API サーバーの Federation 対応

API サーバーを GraphQL 化したら，次に Federation に対応させるようコードを修正します．
これも各フレームワークによって異なりますが，Apollo Server を使用している場合は，以下のように修正します．
:::details コード例

#### パッケージのインストール

```bash
bun add http graphql-tag @apollo/subgraph @apollo/server cors express
bun add -D @types/cors
```

```ts:packages/user,book/src/index.ts
import http from "http";

import { gql } from "graphql-tag";
import { buildSubgraphSchema } from "@apollo/subgraph";
import { ApolloServer } from "@apollo/server";
import { expressMiddleware } from "@apollo/server/express4";
import { ApolloServerPluginDrainHttpServer } from "@apollo/server/plugin/drainHttpServer";
import cors from "cors";
import express from "express";

const app = express();
const httpServer = http.createServer(app);

const typeDefs = gql`
    type Query {
        hello: String!
    }
`;

const resolvers = {
    Query: {
        hello: () => "Hello, World!",
    },
};

const server = new ApolloServer({
    /* ここを `buildSubgraphSchema` に変える */
    schema: buildSubgraphSchema([
        {
            typeDefs,
            resolvers,
        },
    ]),
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

## 2. GraphQL Federation Gateway サーバーの作成

すべての API サーバーを GraphQL 化したら，早速 Federation 用の Gateway サーバーを作成します．
ここで，Gateway サーバーは Apollo Server と Apollo Gateway を使用しました．
https://www.apollographql.com/docs/apollo-server
https://www.apollographql.com/docs/federation/v1/gateway

### 2-1. 各モジュールのインストール

```bash
bun add @apollo/server @apollo/gateway express cors http
bun add -D @types/cors
```

### 2-2．index.ts の作成

`index.ts`に Gateway サーバーを実装していきます．
:::details コード例

```typescript:packages/gateway/src/index.ts
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
    { name: "user", url: "http://localhost:5001/graphql" },
    { name: "book", url: "http://localhost:5002/graphql" },
];

const gateway = new ApolloGateway({
    supergraphSdl: new IntrospectAndCompose({
        subgraphs: serviceList,
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
        cors<cors.CorsRequest>(),
        express.json(),
        expressMiddleware(server, {
            context: async ({ req }) => {
                return { headers: req.headers };
            },
        })
    );

    await new Promise<void>((resolve) => httpServer.listen({ port: 5000 }, resolve));
    console.log("Server ready at 5000 port");
})();
```

:::

### 2-3. Gateway サーバーの起動

`index.ts`が完成したら実際に動かしてみましょう．
なお，Gateway サーバー起動の際は各 API サーバーも起動しておいてください．

```bash
bun --watch packages/gateway/src/index.ts
```

:::details 各 API サーバーも起動する理由
ここで下記のようなエラーが発生する場合があります．

```bash
Service definition for service service1 is missing a url
/workspace/node_modules/@apollo/gateway/src/supergraphManagers/IntrospectAndCompose/loadServicesFromRemoteEndpoint.ts:32
const promiseOfServiceList = serviceList.map(async ({ name, url, datasource }) => {

Error: Tried to load schema for 'Service1' but no 'url' was specified.
```

エラーが出ても焦らないでください．よく読めばわかります．
`service definition is missing a url`と出ているので，service1 に対する url が見つからないと言われていることがわかります．

実は Gateway サーバーは，**スーパーグラフ**（すべての GraphQL のスキーマを統合したスキーマ）を作成するために，起動時に各 API サーバーへリクエストを送ります．
しかし，現在，`https://localhost:5001/graphql`は起動されていないので，Gateway サーバーはリクエストを送ることができません．

このエラーが出た場合は，各 API サーバーを起動してから Gateway サーバーを起動してください．
:::

# 3．動作確認

実は，ここまでで GraphQL Federation の導入は完了しています．
Gateway サーバーのエンドポイントにリクエストを送ることで，各 API サーバーのデータを取得することができます．

```bash:リクエスト
curl --request POST \
  --header 'content-type: application/json' \
  --url 'http://localhost:5000/graphql' \
  --data '{"query":"query { hello }"}'
```

```bash:レスポンス
{
  "data": {
    "hello": "Hello, World!"
  }
}
```

# 4. サービス間のデータのやり取り

ここまでで GraphQL Federation の導入は完了しましたが，サービス間のデータのやり取りができるよう各 API サーバーのスキーマやリゾルバーを修正します．

## 4-1．具体例

具体例があったほうがわかりやすいので，次のようなアーキテクチャを考えてみます．

- service1：本の情報を提供する API サーバー
- service2：ユーザーの情報を提供する API サーバー

```graphql:service1（Book）のスキーマ
type Book {
    id: ID!
    title: String!
    author: User!
}

query {
    getBooks: [Book]!
}
```

```graphql:service2（User）のスキーマ
type User {
    id: ID!
    name: String!
}
```

```mermaid
graph
    A[フロントエンド] --> |getBooksクエリ| B[Gatewayサーバー]
    B --> |Bookの解決| C[service1（Book）]
    B --> |Userの解決| D[service2（User）]
    style A fill:#CDE6C7,stroke:#CDE6C7
    style B fill:#65BBE9,stroke:#65BBE9
    style C fill:#FAA755,stroke:#FAA755
    style D fill:#FAA755,stroke:#FAA755
```

## 4-2．スキーマの修正

まずは設計図であるスキーマの修正をしながら具体例を整理していきましょう．

### 4-2-1．service1（Book）のスキーマ

service1 のスキーマは以下の通りとなります．

```graphql:service1のスキーマ
type Book {
    id: ID!
    title: String!
}

query {
    getBooks: [Book]!
}
```

あれ？`author`がないと思った方もいると思います．
service1 のスキーマには`author`がないのは，`author`は service2 でしか解決できない（データを持っていない）フィールドなので，service1 のスキーマには含めずません．

### 4-2-2．service2（User）のスキーマ

service2 のスキーマは以下の通りとなります．

```graphql:service2のスキーマ
type User {
    id: ID!
    name: String!
}

extend type Book @key(fields: "id") {
    id: ID! @external
    author: User!
}
```

ここで，`author`が出てきました！通常の GraphQL では使われない`@key`や`@external`が出てきましたが，これは Federation のためのディレクティブです．

#### `@key`：この型が他の型と関連付けられるキーを指定します．

DB でいうところの主キーに値するフィールドになります．
ここで指定したフィールドをキーとして，外部から参照できるようになります．
ここでは，`Book`を User という別サービスで使用しているため，`Book`にのみ`@key`を指定しています．

#### `extend type`：外部の型を拡張するためのディレクティブです．

他のサービスで解決される型を拡張し，新たなフィールドを追加することができます．
ここでは，`Book`型を拡張して，`author`フィールドを追加しています．

#### `@external`：他のサービスから解決されるフィールドを指定します．

ここで指定したフィールドは，他のサービスから取得されるデータを使用することができます．
このフィールドは後々，リゾルバーで使用することになります．

## 4-3. リゾルバーの修正

実際のデータのやり取りであるリゾルバーを修正していきます．
ここではデータベースなどの構築はしていないので，適当なモック配列を使用しています．

### 4-3-1．service1（Book）のリゾルバー

service1（Book）のリゾルバーは以下の通りとなります．

```typescript:packages/book/src/index.ts
const books = [
    { id: "1", title: "Book 1" },
    { id: "2", title: "Book 2" },
    { id: "3", title: "Book 3" },
    { id: "4", title: "Book 4" },
    { id: "5", title: "Book 5" },
];


const resolvers = {
    Query: {
        getBooks: () => {
            return books;
        }
    },
};
```

あくまで`author`は User サービス内のスキーマで書かれているため，Book サービスでは解決することができないため，ここでは User に関係ないフィールドのみを解決しています．

### 4-3-2．service2（User）のリゾルバー

service2（User）のリゾルバーは以下の通りとなります．

```typescript:packages/user/src/index.ts
const users = [
    /* リレーションを配列として表現しています． */
    { id: "1", name: "User 1", bookIds: ["1", "3"] },
    { id: "2", name: "User 2", bookIds: ["2", "4", "5"] },
];

const resolvers = {
    Book: {
        author(book: { id: string }) {
            return users.find((user) => user.bookIds.includes(book.id));
        },
    },
};
```

ここでは，`Book`の`author`フィールドを解決するリゾルバーを記述しています．

なんだかややこしそうに見えますが，やっていることとしてはそこまで難しくありません．

- **フィールドが解決できるサービスにスキーマを書く．**
  今回でいうと，`author`は User サービスでデータを保持していて，User サービスでしか解決できないため，Book のフィールドですが User サービスのスキーマに記述しました．
- **スキーマに書いてあるものをリゾルバーで解決する．**
  `author`は User サービスに書いてあるため，User サービスで解決しました．他のフィールドはすべて Book サービスのスキーマに書いてあるので，Book サービスで解決しました．

# 5. Federation も含めた動作確認

すべてのサービスを起動した状態で，Gateway サーバーにリクエストを送ってみましょう．

```bash:リクエスト
curl --request POST \
  --header 'content-type: application/json' \
  --url 'http://localhost:5000/graphql' \
  --data '{"query":"query { getBooks { id, title, author { id, name } } }"}'
```

:::details レスポンス 長いので折りたたんでいます．

```bash:レスポンス
{
  "data":{
    "getBooks":[
      {
        "id":"1",
        "title":"Book 1",
        "author":{
          "id":"1",
          "name":"User 1"
        }
      },
      {
        "id":"2",
        "title":"Book 2",
        "author":{
          "id":"2",
          "name":"User 2"
        }
      },
      {
        "id":"3",
        "title":"Book 3",
        "author":{
          "id":"1",
          "name":"User 1"
        }
      },
      {
        "id":"4",
        "title":"Book 4",
        "author":{
          "id":"2",
          "name":"User 2"
        }
      },
      {
        "id":"5",
        "title":"Book 5",
        "author":{
          "id":"2",
          "name":"User 2"
        }
      }
    ]
  }
}
```

:::
レスポンスを見ると，ちゃんと`author`フィールドが解決できていることが確認できます！！
これで別々のサービスから一つのクエリを作ることができました！！

# 6. 補足

以上で GraphQL Federation の導入は完了です．
ここでは，その他に使用できる便利な機能を紹介します．

## 6-1. リクエストの伝搬

おそらく，稼働しているサービスでは認証などに`HTTP Header`を使用していると思います．
デフォルトだとこれは Gateway サーバーで消えてしまい，各サブグラフに伝搬しません．
伝搬させるには以下のコードを追加する必要があります．
:::details コード例

```typescript:packages/gateway/src/index.ts
const gateway = new ApolloGateway({
  buildService({ url }) {
    return new RemoteGraphQLDataSource({
      url,
      willSendRequest({ request, context }) {
        /* requestが伝搬するリクエスト */
        /* contextが届いたリクエスト */
        request.http?.headers.set('Authorization', context.headers?.authorization);
      },
    });
  },
});
```

:::

## 6-2. ポーリング

Gateway サーバーは，各サブグラフのスキーマを取得するために，起動時に各サブグラフにリクエストを送ります．
しかし，これは起動時の一度だけであり，その後はスキーマが変更されても反映されません．
Gateway サーバーは定期的にスキーマを取得するポーリングを行うことができます．
:::details コード例

```typescript:packages/gateway/src/index.ts
const gateway = new ApolloGateway({
  buildService({ url }) {
    return new RemoteGraphQLDataSource({
      url,
      willSendRequest({ request, context }) {
        request.http?.headers.set('Authorization', context.headers?.authorization);
      },
    });
  },
  supergraphSdl: new IntrospectAndCompose({
    subgraphs: serviceList,
    /* POLL_INTERVAL_IN_MSにポーリングの間隔を指定（ms） */
    pollIntervalInMs: Number(process.env.POLL_INTERVAL_IN_MS),
  }),
});
```

:::

## 6-3. 自己証明書を許可する

ローカル環境では，自己証明書を使用して HTTPS 通信を行っています．
デフォルトだと Gatway サーバーは自己証明書を許可しません．
許可するためには以下のようにする必要があります．
:::details コード例

### 必要なパッケージのインストール

```bash
bun add make-fetch-happen
```

### コード例

```typescript:packages/gateway/src/index.ts
import { defaults } from 'make-fetch-happen';

const gateway = new ApolloGateway({
  buildService({ url }) {
    return new RemoteGraphQLDataSource({
      url,
      /* ローカルのみSSL証明書のチェックをしないようにする */
      fetcher: defaults({
        strictSSL: process.env.ENV_NAME === 'local' ? false : true,
      }),
      willSendRequest({ request, context }) {
        request.http?.headers.set('Authorization', context.headers?.authorization);
      },
    });
  },
  supergraphSdl: new IntrospectAndCompose({
    subgraphs: serviceList,
    pollIntervalInMs: Number(process.env.POLL_INTERVAL_IN_MS),
  }),
});
```

:::

## 6-4. graphql codegen

graphql codegen を使用することで，GraphQL スキーマから Typescript 用の型定義を生成することができます．
これを使用してスーパーグラフスキーマ（統合後のスキーマ）を下に型生成をすることができます．
https://the-guild.dev/graphql/codegen

:::details コード例

```typescript:codegen.ts
import { CodegenConfig } from '@graphql-codegen/cli';

const config: CodegenConfig = {
  schema: [
    {
      /* スキーマファイルではなく，URLからスキーマを取得できる． */
      'http://localhost:5000/graphql': {
        headers: {
          Authorization: 'dummy-token',
        },
      },
    },
  ],
  generates: {
    './src/lib/graphql.ts': {
      plugins: [
        'typescript',
      ],
    },
  },
};

export default config;
```

:::

## 6-5. ALB ヘルスチェック

AWS を使用している場合，GraphQL の場合，ALB のヘルスチェックがうまくいかない場合があります．
ALB がヘルスチェックで GraphQL を叩けないからだそうです．
僕の場合は Express でヘルスチェック専用のエンドポイントを作成しました．
:::details コード例

```typescript:packages/gateway/src/index.ts
app.get("/health", (_req, res) => {
  res.status(200).end();
})
```

:::

# 7. さいごに

以上で GraphQL Federation の導入は完了です．
正直うまく言ったとき，文明の利器を感じましたね．
これによって，API サーバーを統一することができ，よりマイクロサービス化が進んだと思います！

ぜひみなさんも GraphQL Federation を導入してみてください．

よい GraphQL ライフを！！

# 8. ソースコード

https://github.com/taku-271/zenn-graphql-federation-demo
