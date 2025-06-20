---
title: "【TypeScript】自社UIコンポーネントにMCPサーバーを作った話"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [contest2025ts]
published: false
publication_name: "uniformnext"
---

# はじめに

最近 Zenn の記事投稿をサボってたたくみです。

最近 Claude Code がやたら流行ってますね。この AI ブームに乗りたいと思い、**自社 UI コンポーネントである「Kurage UI」の MCP サーバーを作って AI 導入の一歩を踏み出しました。**

:::message
実装方法については後述していますが、初心者向けなので玄人の皆さんは読み飛ばすなりしてください
:::

# 背景

現在、弊社ユニフォームネクストでの新規 SaaS プロダクトでは Next.js（React）を使用しています。
以前は UI コンポーネントに Material UI を使用していましたが、**機能の豊富さやデザインの統一性から**、現在は自社 UI コンポーネントである**Kurage UI**を使用しています。
しかし、フロントエンドでは基本的に Kurage UI を使用するため、Vibe Coding などの **AI 主導**の開発はなかなか手が出せませんでした。
そこで、Kurage UI に MCP サーバーを作成し、AI が Kurage UI を活用しながら**自律的に開発を進められる**ようにしました。

# 使用技術

- Node.js（一部 bun）
- TypeScript
- Model Context Protocol TypeScript SDK

# MCP（Model Context Protocol）とは

最近 MCP について耳にする機会が増えたと思いますが、念のため簡単に説明しておきます。

AI に期待することとして、**作業の自動化**がまず挙げられます。例えば、Google カレンダーで空いている日を探し予定を追加することや、Notion のデータを要約し、その結果を Slack で共有するなどの作業を AI でできると便利じゃないですか？
このように、AI をエージェントとして活用し、あらゆる外部サービスを自動化させたいユースケースがあります。従来だと、外部サービスと連携するためには API サーバーやツールの実装を各サービスで行わなければいけません。しかもサービスによってこのサーバーの仕様が違うとあれば AI エージェントも困ります。
そこで、これらを規格化するプロトコルを作成することで、AI エージェントが使用するツールの規格を統一することができます。この役割を担うプロトコルが**MCP**です。

## MCP の構成

MCP では AI エージェントは**MCP クライアント**と **MCP サーバー**を利用して外部サービスと連携します。

### MCP クライアント

MCP サーバーへのアクセスを管理するレイヤーで、MCP サーバーへの接続やリクエストの送信、結果の処理を行って AI に渡します。AI はその情報（コンテキスト）をもとに回答を生成します。

### MCP サーバー

MCP サーバーは MCP クライアントからリクエストを受け取り、実際に外部サービスを操作をするレイヤーになります。
例えば、MCP クライアントから「Google カレンダーで空いている日を探してほしい」というリクエストがくると Google カレンダーの API を叩き、カレンダーを取得し、MCP クライアントへ返すというフローを行います。

このように各サービスの API の前に MCP サーバーを設置しておくことで、MCP クライアントはすべての外部サービスを**同じリクエスト方法**で利用できるようになります。

# MCP サーバー作成の方針

Kurage UI の MCP サーバーは以下のツールを持つように実装しました。

- すべてのコンポーネント、ユーティリティ関数を取得するツール（`get_available_components_and_utilities`）
  - AI エージェント がどのようなコンポーネント、ユーティリティ関数があるかどうかを把握するため。
- コンポーネントのソースコードを取得するツール（`get_component`）
  - AI エージェント が特定のコンポーネントの動作を理解するため。
- ユーリティティ関数のソースコードを取得するツール（`get_utility`）
  - AI エージェント が特定のユーティリティ関数の動作を理解するため。

これらのツールを利用して、AI エージェントは以下のフローで情報を取得します。

1. `get_available_components_and_utilities`を使用して使用可能なコンポーネント、ユーティリティ関数を取得。
2. 返された情報から、質問に最も近いコンポーネントやユーティリティ関数を判断し、`get_component`または`get_utility`を使用してソースコードを取得。
3. 取得したソースコードを AI エージェントが理解し、それをコンテキストとして回答を生成。

# 実装

方針が決まったので、実際に実装していきます。

## Kurage UI の構成

ソースコードを返すため、必要な部分の構成のみ提示します。

```bash
/
├─ src
│  ├─ components                # コンポーネントが属しているカテゴリ名
│  │  └─ Test
│  │     ├─ Test.stories.tsx    # Storybook用
│  │     ├─ Test.css.ts         # コンポーネントで使用しているCSS
│  │     ├─ Test.tsx            # コンポーネント本体のコード
│  │     └─ index.ts            # コンポーネントをexportするファイル
│  ├─ index.ts                  # カテゴリディレクトリ以下のexportされたコンポーネントをまとめてexportするファイル
│  ├─ utility                   # ユーティリティ関数が属しているカテゴリ名
│  │  ├─ function.ts            # ユーティリティ関数本体のコード
│  │  └─ index.ts               # ユーティリティ関数をexportするファイル
│  └─ index.ts                  # すべてのカテゴリからexportされたモジュールをまとめてexportするファイル
└─ package.json
```

各`index.ts`ファイルの内容は以下のようになっています。

```typescript:src/components/index.ts
export * from "./Test";
```

```typescript:src/components/Test/index.ts
export * from "./Test";
```

```typescript:src/utility/index.ts
export * from "./function";
```

```typescript:src/index.ts
export * from "./components";
export * from "./utility";
```

## 環境構築

### MCP サーバー用のディレクトリの用意

MCP サーバー用のディレクトリを用意し、npm パッケージとして使用できるようにしました。（依存が怖かったのでワークスペースにはしていません。）

```bash
mkdir mcp
cd $_
npm init
mkdir src
```

### 依存関係のインストール

MCP サーバーを作成する際の依存関係をインストールします。

```bash
npm install @modelcontextprotocol/sdk zod
npm install -D @types/node typescript
```

## 実装

準備ができたので実装をしていきます！

### 現在のディレクトリ構成

実装ではディレクトリ構成がとても大事になるので一度このタイミングでの構成を再度提示します。
:::details ディレクトリ構成

```bash
/
├─ src
│  ├─ components
│  │  └─ Test
│  │     ├─ Test.stories.tsx
│  │     ├─ Test.css.ts
│  │     ├─ Test.tsx
│  │     └─ index.ts
│  ├─ index.ts
│  ├─ utility
│  │  ├─ function.ts
│  │  └─ index.ts
│  └─ index.ts
├─ mcp
│  ├─ src
│  │  └─ index.ts
│  └─ package.json
└─ package.json
```

:::

### コーディング

`mcp/src/index.ts`に MCP サーバーを実装していきます。

#### MCP サーバーの初期化

```typescript:mcp/src/index.ts
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { z } from "zod";

import path from "path";
import fs from "fs/promises";

const server = new McpServer({
  name: "mcp-server",
  version: "1.0.0",
})
```

#### 利用可能なコンポーネントとユーティリティ関数を取得するツール

```typescript:mcp/src/index.ts
/* index.tsから名前を取得 */
const getNamesFromIndexContent = (indexContent: string) =>
  indexContent
    .split('\n')
    .filter((line) => line.trim().startsWith('export * from'))
    .map((line) => {
      const match = line.match(/['"]\.\/(.+)['"]/);
      return match ? match[1] : '';
    })
    .filter((name) => name.length > 0);

server.tool("get_available_components_and_utilities", "利用可能なコンポーネントとユーティリティ関数を取得するツール", {}, async () => {
  /* ルートのパス */
  const rootDir = path.resolve(__dirname, "../../");
  /* カテゴリ別のディレクトリパス */
  const categoryIndexPath = path.resolve(rootDir, "src", "index.ts");
  /* カテゴリ別のindex.tsの内容 */
  const categoryIndexContent = await fs.readFile(categoryIndexPath, "utf-8");

  /* index.tsからカテゴリの名前一覧を取得 */
  const categoryNames = getNamesFromIndexContent(categoryIndexContent);

  const componentAndUtilityNames = await Promise.all(
    categoryNames.map(async (categoryName) => {
      /* コンポーネントまたはユーティリティ関数のパス */
      const componentAndUtilityIndexPath = path.resolve(rootDir, "src", categoryName, "index.ts");

      /* コンポーネントまたはユーティリティ関数のindex.tsの内容 */
      const componentAndUtilityIndexContent = await fs.readFile(componentAndUtilityIndexPath, "utf-8");

      /* index.tsからコンポーネントまたはユーティリティ関数の名前を取得 */
      const componentAndUtilityNames = getNamesFromIndexContent(
        componentAndUtilityIndexContent
      );

      /* カテゴリも返す */
      return componentAndUtilityNames.map((name) => ({
        name,
        category: categoryName,
      }));
    })
  )

  return {
    content: [
      {
        type: "text",
        text: componentAndUtilityNames.flat().map((item) => `category:${item.category},name:${item.name}`)).join("\n");
      }
    ]
  }
})
```

#### 特定のコンポーネントを取得

`get_components`だけ掲示します（`get_utility`も似たような実装）。

```typescript:src/mcp/index.ts
server.tool(
  'get_component',
  'コンポーネントのコードを取得する',
  {
    /* MCPクライアントからパラメーターとして`componentName`と`category`を取得 */
    parameters: z.object({
      componentName: z.string(),
      categoryName: z.string(),
    }),
  },
  async ({ parameters }) => {
    const { componentName, categoryName } = parameters;

    /* ルートのパス */
    const rootDir = path.resolve(__dirname, '../../');

    /* コンポーネントのパス */
    const componentPath = path.join(rootDir, 'src', categoryName, componentName);

    try {
      /* コンポーネントパスが存在するか確認 */
      await fs.access(componentPath);

      /* コンポーネントパス内のすべてのファイルを取得 */
      const files = await fs.readdir(componentPath);

      const fileContents: Record<string, string> = {};

      /* コンポーネントパス内のすべてのファイルを展開し、fileContentsに格納 */
      for (const file of files) {
        const filePath = path.join(componentPath, file);
        try {
          const content = await fs.readFile(filePath, 'utf-8');

          fileContents[file] = content;
        } catch (error) {
          console.error(`Error reading file ${filePath}:`, error);
        }
      }

      return {
        content: [
          {
            type: "text",
            text: `${componentName}\nAvailable files:\n${files.join('\n')}\ncontents:\n${Object.entries(fileContents)
                      .map(([file, content]) => `=== ${file} ===\n${content}\n`)
                      .join('\n')}`,
          }
        ]
      }
    } catch (error) {
      return {
        content: [
          {
            type: "text",
            text: `Component ${componentName} not found: ${error instanceof Error ? error.message : 'unknown error'}`,
          }
        ]
      }
    }
  },
);
```

### 配布

DevContainer を使用しているため、ファイルで配布するのは困難だと感じ、**Docker イメージ**を作成し、MCP クライアントが Docker を実行することで MCP サーバーを実行するようにしました。
Docker build コマンドはプロジェクトルートから実行するように npm script を組みました。

```Dockerfile:mcp/Dockerfile
FROM oven/bun AS base

FROM base AS build

WORKDIR /app/mcp

COPY ./mcp/package.json .

RUN bun install

FROM base AS runner

WORKDIR /app

COPY ./mcp ./mcp
COPY ./src ./src

COPY --from=build /app/mcp/node_modules ./node_modules

WORKDIR /app/mcp

RUN bun run build

CMD ["bun", "run", "dist/index.js"]
```

```sh:mcp/scripts/build-mcp.sh
sudo docker build -t mcp . -f mcp/Dockerfile
```

```json:package.json
...
"scripts": {
  "mcp": "bash mcp/scripts/build-mcp.sh",
}
```

こうすることで、プロジェクトルートで`npm run mcp`を実行すると MCP サーバーの Docker イメージが作られ、MCP クライアントから docker コマンドを叩けば実行できる状態になります！

### 動作確認

構築した MCP サーバーを MCP クライアントから動作させてみます。

#### Github Copilot + VsCode

```json:.vscode/mcp.json
{
  "servers": {
    "kurage-ui-mcp": {
      "command": "sudo docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "kurage-ui-mcp",
      ],
    }
  }
}
```

#### Cursor

```json:.cursor/mcp.json
{
  "mcpServers": {
    "kurage-ui-mcp": {
      "command": "sudo docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "kurage-ui-mcp"
      ]
    }
  }
}
```

これらのファイルを設置して、チャットで Kurage UI について質問すると MCP サーバーを参照して回答してくれるようになりました！

# 最後に

自社 UI コンポーネント（Kurage UI）を用いた Vibe Coding をするために MCP サーバーを実装しました！
実装自体はそこまで難しくなく、今後別のサービスに対しても MCP サーバーを作るのも有効だと感じました。
今後の展望として、社内 AI チャットシステムに組み込むことで、開発の際に Kurage UI について質問することができ、**生産性の向上が期待できそうです！**

ただし、もっとセキュアにするためには認証も組み込めるといいのかなとも思いました。

せっかく MCP サーバー構築したので、この調子で会社に AI を導入できていけたらなと思います。
