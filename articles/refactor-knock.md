---
title: "【TypeScript】リファクタリング　本ノック"
emoji: "🎃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
publication_name: "uniformnext"
---

# はじめに

最近勉強したいことが多すぎてオーバーフローしているたくみです．

初心者の方は，まずアプリケーションが動くことを目標に開発を進めていくと思います．
ただ，アプリが動いてもコードが汚かったり，不必要な処理を行っていることがバグの温床になってしまうことがあります．適切なリファクタリングをすることで，開発の効率も上がると思います．

👱「でも，どうやってリファクタリングを学べばいいの？」

リーダブルコードなどの名著を読んで学習しても良いですが，どうせなら問題形式であるとやりやすいですよね．

ということで！
リファクタリング　本ノックを作ったので，楽しみながら学習してください！！

# 注意点

- ここでの「きれい」はあくまで僕の個人的な意見なので，「ここきもい」とかあったらコメント等で教えてください．

- 使用している言語は TypeScript です！

# 1. 変数名をわかりやすく

```ts:問題のコード.ts
const calc = (x: number, y: number): number => {
  return x * y / 2;
}
```

:::details 解答例
このコード，関数名，引数名に意味を持っていないため，とても分かりづらくなっています．

わかりやすい命名のコツは，

- 関数名，引数名は**この関数，引数が何をしているのかを名前からわかる**ようにする．
- 理想は関数の**中身を見なくても**，関数名，引数を見れば使用できることです．

今回の例では，三角形の面積を計算したいため，関数名を`calculateTriangleArea`にし，引数を`base(底辺)`，`height(高さ)`にすると良いと思います！

```ts:解答例.ts
const calculateTriangleArea = (base: number, height: number): number => {
  return base * height / 2;
}
```

:::

# 2. マジックナンバーを使用しない

```ts:問題のコード.ts
const calculateCircleArea = (radius: number) => {
  return r * r * 3.14;
}
```

:::details 解答例
このコード中の`3.14`，数学を知っている人からすれば「円周率なんだろうな」とわかりますが，円周率が`3.14`と知らない人からすればナンノコッチャわからないですよね．（まぁそんな人あんまりいないと思いますが，例なので許してください．）

この場合，マジックナンバー（意味もなくコード中に出てくる数字のこと）は定数として保持しておくととてもわかり易くなります！

```ts:解答例.ts
const PI = 3.14;

const calculateCircleArea = (radius: number) => {
  return r * r * PI;
}
```

:::
:::details Tips

### 定数宣言の違い

TypeScript には定数を宣言するために，`const value = hoge;`といったように`const`という変数宣言手法があります．
実は，型アサーションにも定数にキャストするための`as const`というものが存在します．

一体何が違うのかと．

実はこれ，配列，オブジェクトの場合に明確な違いが発生します．

下記のように`const`で宣言したものは，プロパティの値を書き換えることができます．
オブジェクトに対するポインタが定数であって中身は定数である保証がないイメージですかね？

```ts
const obj = {
  prop1: hoge,
  prop2: hoge,
};

/* OK! */
obj.prop1 = hogehoge;
```

これに`as const`をしようすると，プロパティの値も書き換え不可能な完全にイミュータブルなオブジェクトとなります．

```ts
const obj = {
  prop1: hoge,
  prop2: hoge,
} as const;

/* error! */
obj.prop1 = hogehoge;
```

:::

# 3. 不要な else を消す

```ts:問題のコード.ts
const isAdult = (age: number): boolean => {
  if (age >= 18) {
    return true;
  } else {
    return false;
  }
}
```

::::details 解答例
このコードでは，`else`文を使用する意味は正しいのですが，`else`文が意味のないものになっています．

18 歳以上の場合は，`if`の中で`return`されるのでそこで関数が終了してしまいます．
つまり，`else`を書いたとしても，書かないにしても，`if`文から抜けるとそれは 18 歳未満ということになるので，`false`を返せば良くなります．

ちなみに，この手法を「早期リターン」と呼びますね．

```ts:解答例.ts
const isAdult = (age: number): boolean => {
  if (age >= 18) {
    return true;
  }

  return false;
}
```

:::message
ただ，この書き方だと，18 歳以上でない場合`return false`を実行するということが明示的に書かれていないため読む人によっては分かりづらくはなります．ケースバイケースかもしれません．
:::

実は，この場合は`boolean`を返せばよいため，そもそも条件自体を返すようにすればもっときれいになりそうですね．

```ts:解答例.ts(こっちのほうが簡潔)
const isAdult = (age: number): boolean => age >= 18;
```

::::

# 4.
