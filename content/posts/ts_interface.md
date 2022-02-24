---
title: "Type alias vs Interface"
date: 2022-02-24
draft: true
tags: ["typescript"]
---
## 概要
社内で「TypeScriptの **インタフェース** と **型エイリアス** の違いが上手くわからない！」 という声が上がっていたので、まとめておく。

同じところを認識したうえで、違う箇所を見たほうが頭に残るかなということで、両方見ていこう。
まずは、同じところから。

---
## 同じところ
型エイリアスを使っている箇所はどこでも、インタフェースを使うことができる。

どちらの宣言も形状を定義しており、それらの形状は互いに割り当てが可能。

```typescript
// 型エイリアス
type Sushi = {
	calories: number
	salty: boolean
	tasty: boolean
}

// インタフェース
interface Sushi {
	calories: number
	salty: boolean
	tasty: boolean
}
```

更にどちらも**型を組み合わせる**こともできる。まずは型エイリアスから見てみる。

型エイリアスの場合は、`&`を用いて型を組み合わせる。

```typescript
type Food = {
	calories: number
	tasty: boolean
}

type Sushi = Food & {
	salty: boolean
}

type Cake = Food & {
	sweet: boolean
}
```

一方、ほとんど同じことをインタフェースでも実現可能。

こちらは組み合わせるときに`extends`キーワードを使う。

```typescript
interface Food {
	calories: number
	tasty: boolean
}

interface Sushi extends Food {
	salty: boolean
}

interface Cake extends Food {
	sweet: boolean
}
```

---
## 違うところ
異なる箇所は大きく分けて3つ存在する。
### 1. 型エイリアスのほう汎用的
型エイリアスのほうが、**右辺に任意の型を指定**でき、より汎用的である。

型エイリアスの右辺には、型の式（**型と「**`&`**」や「**`|`**」のような型演算子**）を指定することもできる。

一方、インタフェースの場合は、**右辺は形状**でなければならない。例えば、以下のような型エイリアスはインタフェースに書き換えられない。

```typescript
type A = number
type B = A | string
```

### 2. インタフェースの拡張時のチェック
インタフェースを拡張する場合、**拡張元のインタフェースが拡張先のインタフェースに割り当て可能か**TypeScriptが確認する点。

たとえば、次のようなチェックを行う。

```typescript
interface A {
  good(x: number): string
  bad(x: number): string
}

// Error: 
// Interface 'B' incorrectly extends interface 'A'.
// Types of property 'bad' are incompatible.
// Type '(x: string) => string' is not assignable to type '(x: number) => string'.
// Types of parameters 'x' and 'x' are incompatible.
// Type 'number' is not assignable to type 'string'.
interface B extends A {
  good(x: string | number): string
  bad(x: string): string
}
```

交差型を使う場合は、これとは異なる挙動をする。

最後の例のインタフェースを型エイリアスに変換して、`extends`を交差（`&`）に変えると、TypeScriptは**拡張元の拡張先の型を結合**し、`bad`の**オーバーロードされたシグネチャを作成**し、コンパイル時のエラーを回避する。

したがって、下記の場合はエラーになる。
```typescript
const b: B = {
  good: (x) => `${x}`,
	// Error: Type 'string | number' is not assignable to type 'string'.
  // Type 'number' is not assignable to type 'string'.
  bad: (x) => x
}
```
ちなみに関数ではなく、通常の型に対して行うとnever型になる。

以下の記事参照。

[サバイバルTypeScript interfaceとtypeの違い](https://typescriptbook.jp/reference/object-oriented/interface/interface-vs-type-alias#%E3%83%97%E3%83%AD%E3%83%91%E3%83%86%E3%82%A3%E3%81%AE%E3%82%AA%E3%83%BC%E3%83%90%E3%83%BC%E3%83%A9%E3%82%A4%E3%83%89)

### 3. 宣言のマージ
同じスコープ内に**同名のインタフェース**が存在した場合、それらは**自動的にマージ**される。

これを「**宣言のマージ**」と呼ぶ。

型エイリアスの場合は、コンパイルエラーになる。

宣言のマージはTypeScriptの特徴的な機能の1つなので、また別に記事を作成予定

## 参考文献
[プログラミングTypeScript ――スケールするJavaScriptアプリケーション開発](https://www.oreilly.co.jp/books/9784873119045/)
