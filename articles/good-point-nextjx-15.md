---
title: "Next.js v15の嬉しいところ ７選"
emoji: "👉️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [next, react, frontend, typescript, javascript]
published: true
publication_name: "uniformnext"
---

# はじめに

最近ようやく zenn-cli で記事を書き始めたたくみです．

ついに Next.js v15 がリリースされましたね！！
待ちに待ったリリース！わくわくして夜しか寝れませんでした．

進化した Next.js，何が嬉しいのかや注意することを僕的にまとめてみました！

# 1. Async Request APIs

## TL;DR

`cookies`や`headers`，`params`，`searchParams`などの関数が非同期関数になりました！

## kwsk

### 概要

TL;DR の通り，リクエスト関係の関数が非同期関数に変わっています！
変わった関数は以下とおりです．

- `cookies`
- `headers`
- `draftMode`
- `params` （layout.js，page.js，route.js，default.js，opengraph-image，twitter-image，icon，apple-icon のみ）
- `searchParams`（page.js のみ）

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

TurboPack Dev が安定してリリースされました！

## kwsk

### 概要

TurboPack Dev では，ローカル環境での実行のパフォーマンスが劇的に向上しています．
使用するには下記のように，`dev`に`--turbo`をつける必要があります．

```bash
next dev --turbo
npm run dev -- --turbo
```

実際，`vercel.com`では，ローカルサーバーの起動が約 77%高速化されたり，Fast Refresh のコード更新が約 96%高速化しているっぽいです．
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

- レイアウトと読み込み UI がプリフェッチされます．
- フォーム送信時に，共有レイアウトとクライントの状態が保持されます．
- js が読み込まれていない場合でも，フォームはフルページナビゲーションを介して機能します．

https://nextjs.org/docs/app/api-reference/components/form

### 何が嬉しいのか

今まで，フォームを使用するためには，HTML の`<form>`タグを使用する必要がありました．
僕の感覚上，material-ui や独自 UI コンポーネントを使用している場合，これだけ HTML 純正のタグだと少し気持ち悪さが有りました．
しかし，このアップデートで`<Form>`が追加され，この気持ち悪さもなくなりました！
また，従来の`<form>`タグに比べてパフォーマンスも向上しているので，これからは`<Form>`を使っていきたいですね！

### 注意すること

これも特には無いと思います．

# 4. Server Actions のセキュリティ強化

## TL;DR

Server Actions の意図しないアクセスを防ぐようになりました！

## kwsk

### 概要

Server Actions にて，以下の機能強化が導入されました．

- 使用されていないサーバーアクションは js バンドルに公開されません．
- クライアントが Server Actions を呼び出すために，推測不可能で非決定的な ID を作成するようになりました．また，この ID はビルド間で定期的に再計算されます．

### 何が嬉しいのか

今までの Server Actions だと，export した関数は一つのエンドポイントとしてアクセスできてしまいました．しかし，これはセキュリティ的にとても危険で，意図しないアクセスを受ける可能性がありました．

今回のアップデートで，この危険性が緩和され，セキュリティが向上しています！

### 注意すること

しかし，HTTP エンドポイントは継続して公開されているため，ランダムな ID がバレてしまうとアクセスされる危険性は０では有りません．
そのため，これまでと同様にセキュリティを考慮して，Server Actions を設計する必要があります．

# 5. Caching Semantics

## TL;DR

API Routes や fetch，ClientRouterCache のキャッシュ関連がデフォルトでオフになりました．

## kwsk

### 概要

今までデフォルトで有効だった，API Routes や fetch，ClientRouterCache のキャッシュ関連がデフォルトでオフになりました．
https://nextjs.org/docs/app/building-your-application/upgrading/version-15#fetch-requests

### 何が嬉しいのか

正直，これは嬉しいことというより注意するべきことなので，嬉しいことはあんまりなさそうです．
ただ，Next.js がキャッシュの改善に力を入れてくれているということなので，今後に期待ができますね！

### 注意すること

デフォルトでオフになったので，キャッシュを使いたい場合は，明示的に有効にする必要があります．
例えば，`fetch`のキャッシュをオンにする場合は，以下のように引数に含める必要があります．

```js
await fetch("...", { cache: "force-cache" });
```

# 6. React 19

## TL;DR

React 19 に対応しました！

## kwsk

### 概要

Next.js v15 では，React 19 に対応しました！
React 19 では，`useFormState`が`useActionsState`になったり，`ref`の操作が変わったりなどしています．
個人的には`useActionsState`が，`pending`を持っていたり，`form`タグに渡したりできるので，便利だと思いました！
https://nextjs.org/docs/app/building-your-application/upgrading/version-15#react-19
https://ja.react.dev/blog/2024/04/25/react-19

### 何が嬉しいのか

最新の React を使用することができます．
ただ，Next.js は下位互換性も持っているので，React18 でも問題なく動作するとのことです．

### 注意すること

現在，React 19 は RC となっています．
そのため，本番環境で使用するのは GA リリースのほうが良いかもしれません．

# 7. `next.config.ts`のサポート

## TL;DR

`next.config`が typescript で書けるようになりました！

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

これによって，型安全に`next.config`を書くことができ，プロジェクト全体が Typescript で統一することができます！

### 注意すること

特に無いです！
ただ，typescript になった分，実行の際に遅くなったりするんでしょうか…？

# その他

その他，ESLint9 のサポート，セルフホスティングの改善，静的ルートインジケーターなど，様々なアップデートが行われています．
詳しくは[Next.js の公式ブログ](https://nextjs.org/blog/next-15)を参照してください！

# まとめ

個人的にいいなと思った機能をまとめてみました．
その他に含めた内容でも，重大だったり感動的なアップデートがあるかもしれません！
今すぐアップデートは，まだ安定リリースしたばっかなので危ないかもしれませんが，ぜひとも使ってみたいですね！

それでは，よい Next.js ライフを！

# 参考

https://nextjs.org/blog/next-15
https://nextjs.org/docs/app/building-your-application/upgrading/version-15#temporary-synchronous-usage-1
https://ja.react.dev/blog/2024/04/25/react-19
