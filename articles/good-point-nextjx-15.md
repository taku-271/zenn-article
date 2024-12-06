---
title: "Next.js v15の嬉しいところ ７選"
emoji: "👉️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [next, react, frontend, typescript, javascript]
published: true
publication_name: "uniformnext"
---
# はじめに
最近ようやくzenn-cliで記事を書き始めたたくみです．

ついにNext.js v15がリリースされましたね！！
待ちに待ったリリース！わくわくして夜しか寝れませんでした．

進化したNext.js，何が嬉しいのかや注意することを僕的にまとめてみました！

# 1. Async Request APIs
## TL;DR
`cookies`や`headers`，`params`，`searchParams`などの関数が非同期関数になりました！
## kwsk
### 概要
TL;DRの通り，リクエスト関係の関数が非同期関数に変わっています！
変わった関数は以下とおりです．
- `cookies`
- `headers`
- `draftMode`
- `params` （layout.js，page.js，route.js，default.js，opengraph-image，twitter-image，icon，apple-iconのみ）
- `searchParams`（page.jsのみ）

https://nextjs.org/docs/app/building-your-application/upgrading/version-15#async-request-apis-breaking-change

### 何が嬉しいのか
今までのサーバーサイドコンポーネントは，リクエストを受け取ってからレンダリングする流れでした．つまり，リクエストを受け取るまでレンダリングができず，非効率な感じになっていました．

そこで，上記の様な，リクエストに関わる関数を非同期関数にすることで，リクエストを受け取る前に，リクエストに関係のない部分はレンダリングができるようになりました！

これで更にサーバーコンポーネントが使いやすくなりましたね！

### 注意すること
上記の関数が非同期関数に変わったとのことなので，関数を使う際は`await`を使って非同期関数として使うようにしましょう！
従来通り同期的に使用する方法もあるので，詳しくは公式ドキュメントを確認してください．

# 2. TurboPack Dev
## TL;DR
TurboPack Devが安定してリリースされました！

## kwsk
### 概要
TurboPack Devでは，ローカル環境での実行のパフォーマンスが劇的に向上しています．
使用するには下記のように，`dev`に`--turbo`をつける必要があります．
```bash
next dev --turbo
npm run dev -- --turbo
```

実際，`vercel.com`では，ローカルサーバーの起動が約77%高速化されたり，Fast Refreshのコード更新が約96%高速化しているっぽいです．
https://nextjs.org/blog/turbopack-for-development-stable

### 何が嬉しいのか
何と言っても，ローカル環境での起動やリフレッシュが高速化されていることです！
自分で試してみたところ，確かに高速化されていることがわかりました．

### 注意すること
特には無いと思います．

# 3. `<Form>` Component
## TL;DR
`<Form>`コンポーネントが追加されました！

## kwsk
### 概要
なんだかんだ今までなかった`<Form>`コンポーネントが追加されました！
このコンポーネントには`<form>`タグの機能はもちろんの上，以下の要素が追加されました．
- レイアウトと読み込みUIがプリフェッチされます．
- フォーム送信時に，共有レイアウトとクライントの状態が保持されます．
- jsが読み込まれていない場合でも，フォームはフルページナビゲーションを介して機能します．

https://nextjs.org/docs/app/api-reference/components/form

### 何が嬉しいのか
今まで，フォームを使用するためには，HTMLの`<form>`タグを使用する必要がありました．
僕の感覚上，material-uiや独自UIコンポーネントを使用している場合，これだけHTML純正のタグだと少し気持ち悪さが有りました．
しかし，このアップデートで`<Form>`が追加され，この気持ち悪さもなくなりました！
また，従来の`<form>`タグに比べてパフォーマンスも向上しているので，これからは`<Form>`を使っていきたいですね！

### 注意すること
これも特には無いと思います．

# 4. Server Actionsのセキュリティ強化
## TL;DR
Server Actionsの意図しないアクセスを防ぐようになりました！

## kwsk
### 概要
Server Actionsにて，以下の機能強化が導入されました．
- 使用されていないサーバーアクションはjsバンドルに公開されません．
- クライアントがServer Actionsを呼び出すために，推測不可能で非決定的なIDを作成するようになりました．また，このIDはビルド間で定期的に再計算されます．

### 何が嬉しいのか
今までのServer Actionsだと，exportした関数は一つのエンドポイントとしてアクセスできてしまいました．しかし，これはセキュリティ的にとても危険で，意図しないアクセスを受ける可能性がありました．

今回のアップデートで，この危険性が緩和され，セキュリティが向上しています！

### 注意すること
しかし，HTTPエンドポイントは継続して公開されているため，ランダムなIDがバレてしまうとアクセスされる危険性は０では有りません．
そのため，これまでと同様にセキュリティを考慮して，Server Actionsを設計する必要があります．

# 5. Caching Semantics
## TL;DR
API Routesやfetch，ClientRouterCacheのキャッシュ関連がデフォルトでオフになりました．

## kwsk
### 概要
今までデフォルトで有効だった，API Routesやfetch，ClientRouterCacheのキャッシュ関連がデフォルトでオフになりました．
https://nextjs.org/docs/app/building-your-application/upgrading/version-15#fetch-requests

### 何が嬉しいのか
正直，これは嬉しいことというより注意するべきことなので，嬉しいことはあんまりなさそうです．
ただ，Next.jsがキャッシュの改善に力を入れてくれているということなので，今後に期待ができますね！

### 注意すること
デフォルトでオフになったので，キャッシュを使いたい場合は，明示的に有効にする必要があります．
例えば，`fetch`のキャッシュをオンにする場合は，以下のように引数に含める必要があります．
```js
await fetch("...", { cache: "force-cache" });
```
# 6. React 19
## TL;DR
React 19に対応しました！

## kwsk
### 概要
Next.js v15では，React 19に対応しました！
React 19では，`useFormState`が`useActionsState`になったり，`ref`の操作が変わったりなどしています．
個人的には`useActionsState`が，`pending`を持っていたり，`form`タグに渡したりできるので，便利だと思いました！
https://nextjs.org/docs/app/building-your-application/upgrading/version-15#react-19
https://ja.react.dev/blog/2024/04/25/react-19

### 何が嬉しいのか
最新のReactを使用することができます．
ただ，Next.jsは下位互換性も持っているので，React18でも問題なく動作するとのことです．

### 注意すること
現在，React 19はRCとなっています．
そのため，本番環境で使用するのはGAリリースのほうが良いかもしれません．

# 7. `next.config.ts`のサポート
## TL;DR
`next.config`がtypescriptで書けるようになりました！

## kwsk
### 概要
今まで`next.config.js`で設定を書いていましたが，`next.config.ts`で書けるようになりました！
```ts:next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  /* config */
};

export default nextConfig;
```

### 何が嬉しいのか
これによって，型安全に`next.config`を書くことができ，プロジェクト全体がTypescriptで統一することができます！

### 注意すること
特に無いです！
ただ，typescriptになった分，実行の際に遅くなったりするんでしょうか…？

# その他
その他，ESLint9のサポート，セルフホスティングの改善，静的ルートインジケーターなど，様々なアップデートが行われています．
詳しくは[Next.jsの公式ブログ](https://nextjs.org/blog/next-15)を参照してください！

# まとめ
個人的にいいなと思った機能をまとめてみました．
その他に含めた内容でも，重大だったり感動的なアップデートがあるかもしれません！
今すぐアップデートは，まだ安定リリースしたばっかなので危ないかもしれませんが，ぜひとも使ってみたいですね！

それでは，よいNext.jsライフを！

# 参考
https://nextjs.org/blog/next-15
https://nextjs.org/docs/app/building-your-application/upgrading/version-15#temporary-synchronous-usage-1
https://ja.react.dev/blog/2024/04/25/react-19
