---
title: Map type & Utility
date: 2022-03-10
tags: ["typescript"]
---

## 概要
前回の社内勉強会で`Map`型が話題に上がり、自分の復習のためにまとめる。

## マップ型とは
**オブジェクトのキーの値**と**プロパティの型を指定**することができる。 以下に例を示す。

```typescript
type Weekday = 'Mon' | 'Tue' | 'Wed' | 'Thu' | 'Fri'
type Day = Weekday | 'Sat' | 'Sun'

// 現在の曜日と次の曜日の組み合わせ
let nextDay: {[K in Weekday]: Day} = {
  Mon: 'Tue'  
}
```

上記は以下のようなエラーを示す。

```
型 '{ Mon: "Tue"; }'は、型'{ Mon: Day; Tue: Day; Wed: Day; Thu: Day; Fri: Day; }': Tue, Wed, Thu, Fri
のプロパティに見当たりません。
```

これは、`Weekday`のキーが欠けているために起こるエラーである。

したがって、下記のように全てのパターンを書き出してあげるとエラーは消える。

```typescript
let nextDay: {[K in Weekday]: Day} = {
  Mon: 'Tue',
  Tue: 'Wed',
  Wed: 'Thu',
  Thu: 'Fri',
  Fri: 'Sat'
}
```

また、**インデックスシグネチャ(`{ [index:string] : {message: string} }`)**と同様、**1つのオブジェクト**につき**最大1つのマップ型**を持つことができる。

```typescript
type MyMappedType = {
  [Key in UnionType]: ValueType
}
```

`Key`はキー、`UnionType`は合併型、`ValueType`は値の型を表す。

マップ型は、オブジェクトのキーと値に型を与えるだけでなく、**ルックマップ型と組み合わせると、どの値の型がどのキーの名前に対応するかを制限できる**。

```typescript
type Account = {
  id: number
  isEmployee: boolean
  notes: string[]
}

// すべてのフィールドを省略可能にする
type OptionalAccount = {
  [K in keyof Account]?: Account[K]
}

// すべてのフィールドをnull許容する
type NullableAccount = {
  [K in keyof Account]: Account[K] | null
}

// すべてのフィールドを読み取り専用にする
type ReadonlyAccount = {
  readonly [K in keyof Account]: Account[K]
}

// 全てのフィールドを再び書き込み可能になる（Accountと同じ）
type Account2 = {
  -readonly [K in keyof ReadonlyAccount]: Account[K]
}

// すべてのフィールドを再び必須にする（Accountと同じ）
type Account3 = {
  [K in keyof OptionalAccount]-?: Account[K]
}
```

### 上記コードの補足
`-`はマイナス演算子と呼び、マップ型でのみ利用可能な特別な型演算子である。

これを用いると、`?`や`readonly`**を取り消す**ことができ、フィールドを必須や書き込み可能に戻すことができる。

ちなみに、プラス演算子（`+`）も存在するが、暗黙的なものなので使う場面は無いと思われる。

つまり、`readonly`は`+readonly`と同等であり、`?`は`+?`と同等である。


---
## 組み込みのマップ型
マップ型を内部的に用いている組み込み型を一部紹介する。

### Record

オブジェクトのキーの値とプロパティの値の型を指定する。

キーにすることができるのは、**インデックスシグネチャ**が`string`, `number`だけであるのに対して、`string`, `number`, `symbol`のサブタイプである点が特徴。 

```typescript
type Record<K extends string | number | symbol, T> = { 
  [P in K]: T; 
}
```


### Partial

Object内の全てのフィールドを省略可能と指定する。

```typescript
type Partial<T> = { 
  [P in keyof T]?: T[P]; 
}
```

### Required

Object内の全てのフィールドを必須（省略不可）と指定する。

```typescript
type Required<T> = { 
  [P in keyof T]-?: T[P]; 
}
```

### Readonly

Object内のすべてのフィールドを読み取り専用と指定する。

```typescript
type Readonly<T> = { 
  readonly [P in keyof T]: T[P]; 
}
```

### Pick

指定されたKeysだけを持つ、Objectのサブタイプを返す。

```typescript
type Pick<T, K extends keyof T> = { 
  [P in K]: T[P]; 
}
```

## 参考文献
- [プログラミングTypeScript ――スケールするJavaScriptアプリケーション開発](https://www.oreilly.co.jp/books/9784873119045/)
- [公式ドキュメント](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)
