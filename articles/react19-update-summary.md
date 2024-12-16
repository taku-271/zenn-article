---
title: "React19 アップデートまとめ"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [react, typescript, react19, update, javascript]
published: true
publication_name: "uniformnext"
---

# はじめに

こんにちはたくみです．

今朝，ふと X を見ていたら，React 公式から嬉しいポストがされていました．
なんと！！React19 が安定リリースされました！！！

この機会に React19 のアップデートの内容をまとめておこうと思います！

# Action

React には，「**アクション**」という概念があります．
アクションとは一言でいうと，「非同期トランジションを使用する関数」と呼ばれています．
昔の React では，ユーザーのアクションに対して何か行いたいとき，「送信状態」，「エラー」，「リクエストの順序」などを，すべて手動で行う必要がありました．アクションは，これらのデータの送信の管理を自動的にしてくれるものです．React18 で追加された`useTransition`は，アクションの一部で，送信状態を管理することができます．

## `useActionState`

フォーム内の Actions の一般的なケースを簡単かするためのフックです．
このフックでは，送信状態やエラー状態，action 関数を返します．
:::message
従来この関数は`useFormState`と呼ばれていましたが，名前の変更に伴って`useFormState`は非推奨になりました．
:::
https://react.dev/reference/react/useActionState

### form アクション

`<form>`，`<input>`，`<button>`タグに`action`props が追加されています．

:::details useActionState と form の組み合わせ
以上２つの追加されたフックやタグを使うことで，アクションを簡単に実装することができます．

```react
const [error, action, isPending] = useActionState(
  async (prevError, formData) => {
    const error = await submitData(formData);
    if (error) {
      return error;
    }

    return null
  },
  null,
);

return (
  <form action = {action}>
    <input type="text">
    <button type="submit" disable={isPending}>送信</button>
  </form>
)
```

:::

## `useOptimistic`

楽観的な更新を行う際に使用するフックです．
楽観的な更新とは，ユーザーが送信したデータがサーバーからのレスポンスを待たずに，即座に UI に反映されることです．
https://react.dev/reference/react/useOptimistic
:::details コード例

```react
const [text, setText] = useState('');
const [optimisticText, setOptimisticText] = useOptimistic(text);

const submitAction = async (formData) => {
  const newText = formData.get("text");
  setOptimisticText(newText);
  const updateText = await submitData(formData);
  setText(updateText);
};

return (
  <form action={submitAction}>
    <p>Your text is {optimisticText}</p>
    <input type="text" name="text" />
  </form>
)
```

:::

# `use`

React19 では，レンダリング中にリソースを読み込む`use`が導入されています．
`use`ではプロミスやコンテキストを読み込むことができます．
https://react.dev/reference/react/use
:::details プロミスの例
プロミスが解決するまでは React はサスペンド状態になります．

```react
const getData = async () => {
  const response = await fetch('https://api.example.com/data');
  return response.json();
}

const data = use(getData);
```

:::

:::details コンテキストの例
これにより，条件付きでコンテキストを読み取れるようになります．

```react
if (!children) {
  return null;
}

const test = use(TestContext);
```

:::

# React DOM static API

`prerender`，`prerenderToNodeStream`が追加されました！
これらの API を使用すれば，React ツリーを静的 HTML に事前レンダリングできます！
:::details コード例

```react
const {prelude} = await prerender(<App />, {
  bootstrapScripts: ["/main.js"],
})

return new Response(prelude, {
  headers: { "content-type": "text/html" },
})
```

:::
https://react.dev/reference/react-dom/static/prerender
https://react.dev/reference/react-dom/static/prerenderToNodeStream

# React Server Components

React19 では，試験的であった，React Server Components が導入されました．

## Server Components

おそらく，Next.js を使用している方は今まで使用してきたと思います．
Server Components は，バンドル前にコンポーネントを事前レンダリングできる機能です．
レンダリングは CI サーバー上でビルド時に実行することも，リクエストごとに実行することも可能です．
https://react.dev/reference/rsc/server-components

## Server Actions

クライアントコンポーネントからサーバー上で実行される非同期関数を呼び出す事ができる Action です．
フォームの送信などのユーザーのアクションに対して，データの更新や削除などの非同期処理を行う場合に使用します．
:::details 補足
厳密に言うと，Server Actions は Action 機能を提供する Server Function の一種です．
:::
https://react.dev/reference/rsc/server-functions
:::message
`use server;`というものが追加されましたが，これは Server Actions を宣言するディレクティブであり，サーバーコンポーネントを示すものではありません．
:::

# Ref

## `ref`を props に渡す

`ref`を props に渡せるようになりました！
これにより今まで使用していた`forwardRef`を使用する必要がなくなり，`forwardRef`は非推奨になりました．
:::details コード例

```react
const MyComponent = ({ref}) => {
  return <input ref={ref} />
}

<MyComponent ref={ref}>
```

:::

## `ref`のクリーンアップ関数

`ref`コールバックからクリーンアップ関数を返すことがサポートされました！
コンポーネントがアンマウントされたとき，`ref`コールバックから返されたクリーンアップ関数を呼び出し，`ref`をクリーンアップすることができます．
従来，アンマウント時に`null`を引数に呼び出されていましたが，クリーンアップ関数の場合は`null`で呼び出さないようになっています．`null`で呼び出す動作は非推奨になる予定っぽいです．

## `useRef`の引数必須

`useRef`の引数が必須になりました．
これにより，`createContext`と同様に動作するようになります．

# ハイドレーションエラーのログ

ハイドレーションエラーの際のログが見やすくなっています．
新しいエラーログでは，ハイドレーションエラーであることが明確にされており，不一致分の差分も表示されます．

# `Context.Provider`

`<Context.Provider>`の代わりに`<Context>`が使用できるようになりました．
ゆくゆくは`<Context.Provider>`は非推奨になる予定です．
:::details コード例

```react
return (
  <TestContext value="test">
    {children}
  </TestContext>
)
```

:::

# メタデータ

`<title>`，`<link`，`<meta>`などのドキュメントメタデータ用のタグが追加されました．
各コンポーネント内でこれらのタグを呼び出すことで，ドキュメントのメタデータを上書きすることができます．
:::details コード例
上のコードのように書くと，下の HTML が生成されます．

```react
return (
  <article>
    <h1>テスト</h1>
    <title>テスト</title>
    <meta name="author" content="takumi" />
    <link rel="author" href="https://test.com" />
  </article>
)
```

```html
<!DOCTYPE html>
<html>
  <head>
    <title>テスト</title>
    <meta name="author" content="takumi" />
    <link rel="author" href="https://test.com" />
  </head>
  <body>
    <h1>テスト</h1>
  </body>
</html>
```

:::
https://react.dev/reference/react-dom/components/title
https://react.dev/reference/react-dom/components/meta
https://react.dev/reference/react-dom/components/link

# スタイルシート

スタイルシートの組み込みがサポートされました！
`<link>`タグにスタイルシートのパスを指定する，または`<style>`タグを使用することで組み込め，`precedence`で優先度を指定することができます．

:::details コード例
上のコードのように書くと，下の HTML が生成されます．

```react
<link rel="stylesheet" href="style1.css" precedence="default" />
<link rel="stylesheet" href="style2.css" precedence="high" />
```

```html
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="style2.css" />
    <link rel="stylesheet" href="style1.css" />
  </head>
  <body></body>
</html>
```

:::
https://ja.react.dev/reference/react-dom/components/link
https://ja.react.dev/reference/react-dom/components/style

# `Suspense`

React19 では，サスペンドすると兄弟ツリー全体のレンダリングを待たずに，最も近いサスペンス境界のフォールバックをすぐコミットします．
これによって，サスペンドフォールバックの表示が高速化され，ツリー内の遅延リクエストも引き続き処理されます．
https://react.dev/blog/2024/04/25/react-19-upgrade-guide#improvements-to-suspense

# その他

その他，エラーやスクリプトの改善が行われています．
https://react.dev/blog/2024/12/05/react-19
https://react.dev/blog/2024/04/25/react-19-upgrade-guide

# さいごに

ついに React19，リリースされましたね．
個人的には，Action 関係がフォーム送信の際にとっても役立つので嬉しかったです．
今後も React のアップデートには注目していきたいですね．

では，よい React ライフを！

:::message
ちなみに，いくつかの有名なライブラリは React19 に対応していないので，アップデートする際は注意が必要です．
:::
