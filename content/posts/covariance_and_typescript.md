---
title: "Variance and TypeScript"
date: 2022-03-17
tags: ["typescript"]
---
## 概要
型について理解を深める上で、必要不可欠となるVariance（変性）をTypescriptを用いて学んだので、ここに書き留める。

---
## サブタイプとスーパータイプ
まずは変性について学ぶ上で、知っておくべき基本的な用語から触れていく。

### サブタイプ
A, Bという2つの型があり、BがAのサブタイプである場合、**Aが要求されているところ**はどこでも、**Bを安全に使うことができる**。

TypeScriptによる例を以下に示す。
- 配列はオブジェクトのサブタイプ
- タプルは配列のサブタイプ
- 全ての値は`any`のサブタイプ
- `never`は全ての値のサブタイプ
- `Animal`を拡張する`Bird`というクラスがある場合、`Bird`は`Animal`のサブタイプ

### スーパータイプ
A, Bという2つの型があり、BがAのスーパータイプである場合、**Bが要求されているところ**
はどこでも、**Aを安全に使うことができる。**

TypeScriptによる例を以下に示す。
- オブジェクトは配列のスーパータイプ
- 配列はタプルのスーパータイプ
- `any`は全ての値のスーパータイプ
- すべての値は`never`のスーパータイプ
- `Animal`は`Bird`のスーパータイプ

---
## 変性とは
型同士の関係性（型Aを指定したときに、そのサブタイプである型Bを当てはめられるかどうか）を判別するルールは、**プログラム言語間で大きな相違点となっている**。

このようなルールを一般的に指す言葉が**変性**であり、以下の**4種類**に分けられる。

### 不変性

Tそのものを必要とする。

### 共変性

`<:T`であるものを必要とする、つまり**Tそのもの**あるいは**Tのサブタイプ**を必要とする。

### 反変性

`>:T`であるものを必要とする、つまり**Tそのもの**あるいは**Tのスーパータイプ**を必要とする。

### 双変性

`<:T` または `>:T`のどちらかを必要とする、つまり**Tそのもの**あるいは**Tのサブタイプ**、もしくは**スーパータイプ**を必要とする。

---
## TypeScriptにおける変性

TypeScriptでは、オブジェクト、クラス、配列、関数の戻り値の型のような複雑な型はすべて、そのメンバーに対して**共変**である。

以下に共変であることの例を示す。

```typescript
class Animal {}

// Animalのサブクラス
class Bird extends Animal {
  chirp() {}
}

// Birdのサブクラス
class Crow extends Bird {
  caw() {}
}

const generate = (f: () => Bird) => {
  // ...
}

// Animalはgenerateで受け取る関数の戻り値の型のスーパータイプなのでNG
// Error: Argument of type '() => Animal' is not assignable to parameter of type '() => Bird'.
// Property 'chirp' is missing in type 'Animal' but required in type 'Bird'.
const toAnimal = (): Animal => new Animal
generate(toAnimal)

// Birdはgenerateで受け取る関数の戻り値の型と同一なのでOK
const toBird = (): Bird => new Bird
generate(toBird)

// Crowはgenerateで受け取る関数の戻り値の型のサブタイプなのでOK
const toCrow = (): Crow => new Crow
generate(toCrow)
```

ただし一つだけ例外が存在し、それは**関数のパラメータの型**。

`strictFunctionTypes`がオンの場合、関数のパラメータの型は共変ではなく**反変**、つまりサブタイプの場合はコンパイルエラーが起きる。

以下に例を示す。
```typescript
const clone = (f: (b: Bird) => Bird) => {
  // ...
}

// a: Animal はcloneの引数の関数パラメータ型 b: Bird のスーパータイプなのでOK
const animalToBird = (a: Animal): Bird => new Bird
clone(animalToBird)

// b: Bird はcloneの引数の関数パラメータ型 b: Bird と同一なのでOK
const birdToBird = (b: Bird): Bird => b
clone(birdToBird)

// c: Crow はcloneの引数の関数パラメータ型 b: Bird のサブタイプなのでNG
const crowToBird = (c: Crow): Bird => new Crow
// Error: Argument of type '(c: Crow) => Bird' is not assignable to parameter of type '(b: Bird) => Bird'.
// Types of parameters 'c' and 'b' are incompatible.
// Property 'caw' is missing in type 'Bird' but required in type 'Crow'.
clone(crowToBird)
```

`strictFunctionTypes`がオフの場合は**双変**、つまり上記のコードは全て通るようになる。

---
## まとめ
上記のことは覚えておいてもよいが、なかなか難しいと思われる（少なくとも私は無理）。

なのでワードだけ頭の片隅に入れておき、誤った型付けをしてIDEに怒られた際に「こんなのあったな〜」と思い出して調べることで原因特定の材料とすれば良い。

---
## 参考文献
- [プログラミングTypeScript ――スケールするJavaScriptアプリケーション開発](https://www.oreilly.co.jp/books/9784873119045/)
- [30歳からのプログラミング ――TypeScript における変性（variance）について](https://numb86-tech.hatenablog.com/entry/2020/07/04/095737)
- [TypeScript Deep Dive ――型の互換性](https://typescript-jp.gitbook.io/deep-dive/type-system/type-compatibility)