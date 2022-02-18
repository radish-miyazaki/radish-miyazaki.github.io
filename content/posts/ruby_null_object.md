---
title: Ruby Null Object
date: 2022-02-18
tags: ["ruby"]
---

## Null Objectとは
Null Objectとは、「**何もしないメソッド**」を持つオブジェクトのこと。

クラスベースのオブジェクト指向言語（Java, C#など）だと時折見かける。

```ruby
class Huga
  def huga
    'huga'
  end
end

class NullHuga
  def huga; end
end

class Hoge
  def process_huga(huga)
    huga || NullHuga.new
  end
end	
```

Null Objectのメソッドを呼び出すと、エラーを出さずにメッセージをゴミ箱に捨ててくれる。

つまり、初期化されていない参照をNull Objectで置換すると、`NoMethodError`**を起こさなくて済む**。

これにより、いちいち既存の処理に`if`チェックを入れなくても済む。

## moduleを応用してみる
対象となるクラスとNull Objectクラス間で共通のメソッドを持つようにするため、Interfaceのようなモジュールを持たせてみる。
```ruby
# 専用のエラーを作っておく
class InterfaceError < StandardError; end

module HogeInterface
  def hoge
    raise InterfaceError
  end
end

class Hoge
  include HogeInterface
  def hoge
    'hoge'
  end
end

class NullHoge
  include HogeInterface
  def hoge; end
end
```
これで、Interfaceモジュールに先にメソッド定義するよう心がければ、抜け漏れが減るかも？

### 参考書籍
[メタプログラミングRuby 第2版](https://www.amazon.co.jp/%E3%83%A1%E3%82%BF%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0Ruby-%E7%AC%AC2%E7%89%88-Paolo-Perrotta/dp/4873117437)
