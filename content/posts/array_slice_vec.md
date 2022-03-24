---
title: "Array vs Slice vs Vector"
date: 2022-03-24
tags: ["rust"]
---
## 概要
以前Rustを学習していたが、業務で新しい言語を学ぶ必要が出てきたので、一時中断していた。

最近Rustに入門し直したので、その時に出てきた配列とスライスとベクタの違いを備忘録として書き留める。

---
## 配列 (`[T; n]`)
配列は**固定長**で、同一型を格納するリストである。

配列内の要素を入れ替えることはできるが、サイズは変更できない。
```rust
fn main() {
    let mut arr = [3, 5, 1, 2, 4];
    arr.sort();
   
    println!("{:?}", arr) // [1, 2, 3, 4, 5]
    
    // Error: method not found in `[{integer}; 5]`
    arr.push(6); 
   
    println!("{:?}", arr)
}
```

実装する場合は、`[1, 2, 3]` または `[T; n]`(Tは値、nは要素数)で記述する。
```rust
fn main() {
    let arr1 = [1, 2, 3];
    let arr2 = [0; 3];
   
    println!("{:?} {:?}", arr1, arr2) // [1, 2, 3] [0, 0, 0]
}
```

配列では、要素内の型が同じであっても、**要素数が異なれば別の型**と判断される。

つまり、`[0; 3]` と `[0; 4]` は別の型扱いされる。
```rust
fn type_of<T>(_: T) {
    println!("{}", std::any::type_name::<T>());
}

fn main() {
    let arr1 = [0; 3];
    let arr2 = [0; 4];
   
    type_of(arr1); // [i32; 3]
    type_of(arr2); // [i32; 4]
}
```

境界チェックが行われるので、境界外のインデックス値を指定すると、`panic`を起こす。
```rust
fn main() {
    let arr = [0; 3];
    
    // Error: index out of bounds: the length is 3 but the index is 100
    arr[100]; 
}
```

---
## ベクタ（`Vec<T>`）
`vec!`マクロや`Vec::new()`などを用いて生成できる。

配列 と異なり、ヒープメモリに確保される値で**拡大・縮小が可能**。
```rust
fn main() {
    let mut v = vec!(1, 2, 3);
    println!("{:?}", v); // [1, 2, 3]
    
    v.push(4);
    println!("{:?}", v); // [1, 2, 3, 4]
}
```

型は`alloc::vec::Vec<T>`(Tは任意の型)。
```rust
fn main() {
    let vec = vec![1, 2, 3];
    type_of(vec); // alloc::vec::Vec<i32>
}
```

境界チェックが行われず、境界外のインデックス値を指定してもエラーは起きない。
```rust
fn main() {
    let vec = vec!(1, 2, 3);
    println!("{}", vec[1000000]);
}
```

要素へのアクセスなど、基本的な操作は配列と同じ。

---
## スライス（`[T]`）
配列とは異なり、**サイズは動的**（コンパイル時に不明）だが、**拡大・縮小することは出来ない**。

配列やベクタなどの**存在する位置を指す**ために用いられ、両者の参照は型強制によってスライスの参照にすることができる。

```rust
fn main() {
    let mut s: &[i32];

    let arr = [1, 2, 3];
    s = &arr;
    println!("{:?}", s); // [1, 2, 3]
    type_of(s); // &[i32]

    let vec = vec![1, 2, 3];
    s = &vec;
    println!("{:?}", s); // [1, 2, 3]
    type_of(s); // &[i32]
}
```

この特性を用いると、**引数に（構成要素の型が同じ）配列とベクタのどちらでも受け取ることができる**関数を実装できる。
```rust
fn print_slice(s: &[i32]) {
    println!("{:?}", s)
}

fn main() {
    print_slice(&[1, 2, 3]);
    print_slice(&vec!(4, 5, 6));
}
```

サイズが動的で、そのままでは変数に格納することができないので、一般的に**参照**(`&[T]`)を用いて扱う。このため、スライスの参照も含めて「スライス」と呼ばれる。

参照を用いると、その値は**スライスの最初の要素を指すポインタ**と**スライスの長さ**という2つの`usize`要素で構成される。

このような構造を**ファットポインタ**と呼ぶ。

また、境界チェックが行われるので、境界外のインデックス値を指定すると、`panic`を起こす。


## まとめ
`Go`を以前に触っていたので、スライスの意味合いが大分違って苦戦した…

だいぶ省略してしまった & 自分の理解がまだまだなので、Twitter等で「ここ違うよ！」とか「これも足したほうが良いよ！」などアドバイスをいただけますと助かります🙇

## 参考資料
- [Rust By Example --配列とスライス](https://doc.rust-jp.rs/rust-by-example-ja/primitives/array.html)
- [The Rust Programming Language --スライス型](https://doc.rust-jp.rs/book-ja/ch04-03-slices.html)
- [The Rust Programming Language --ベクタで値のリストを保持する](https://doc.rust-jp.rs/book-ja/ch08-01-vectors.html)
- [実践Rustプログラミング入門](https://www.shuwasystem.co.jp/book/9784798061702.html)
