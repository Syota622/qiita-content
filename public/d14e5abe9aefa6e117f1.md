---
title: Ruby：「インスタンスメソッド」「クラスメソッド」「インスタンス変数」「クラス変数」について
tags:
  - Ruby
  - オブジェクト指向
  - クラス
  - プログラミング初心者
private: false
updated_at: '2023-05-03T11:01:44+09:00'
id: d14e5abe9aefa6e117f1
organization_url_name: null
slide: false
ignorePublish: false
---
# 実装
例として、Carクラスを考えます。Carクラスには、インスタンスごとに異なる状態（例：速度）があり、またクラス全体に関連する情報（例：生成された車の総数）があります。

```ruby
class Car
  # 初期化
  @@total_cars = 0

  # 初期化
  def initialize
    @speed = 0 # 速度
    @@total_cars += 1 # 車の総数
  end

  # クラスメソッド
  def self.total_cars
    @@total_cars
  end

  # インスタンスメソッド（ゲッター）
  def speed
    @speed
  end

  # インスタンスメソッド
  def accelerate(amount)
    @speed += amount
  end
end

# 各Carインスタンス
car1 = Car.new
car2 = Car.new

puts Car.total_cars # 2 (car1+car2=2：クラスメソッドを呼び出す)

car1.accelerate(20)
puts car1.speed # 20 (cat1インスタンスメソッドを呼び出す)

car2.accelerate(50)
puts car2.speed # 50 (car2インスタンスメソッドを呼び出す)

```

# 「インスタンスメソッド」「クラスメソッド」の違い

- インスタンスメソッド：speedとaccelerate(amount)は、インスタンスの状態（速度）に関連する操作を行うため、インスタンスメソッドとして定義されています。

- クラスメソッド：self.total_carsは、生成されたCarインスタンスの総数を返すクラスメソッドです。このメソッドはインスタンスに依存しないため、クラスメソッドとして定義されています。

# 「@(インスタンス変数)」「@@(クラス変数)」の違い

- @(インスタンス変数)：`@speed`はインスタンス変数であり、各Carインスタンスが固有の速度を保持。

- @@(クラス変数)：`@@total_cars`はクラス変数であり、生成されたCarインスタンスの総数を追跡。すべてのCarインスタンスは、同じ`@@total_cars`変数を参照します。

# まとめ
- インスタンスごとの状態や振る舞いに関連する情報や操作はインスタンスメソッドで定義し、インスタンスに依存しない情報や操作はクラスメソッドで定義します。
- インスタンス変数は、そのクラスのインスタンスに固有のデータを保持し、クラス変数は、そのクラスのすべてのインスタンスに共通のデータを保持します。
