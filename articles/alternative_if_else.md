---
title: "【備忘録】２者間の条件分岐だからといって安易にif文を使用しない"
emoji: "🧑‍🔬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Ruby","Rails","初心者","リファクタリング"]
published: false
---

## はじめに

２者間の条件分岐を記述する際にはif~elseを使用することが多いと思います。
しかしながら、if~elseを使用することで可読性や拡張性が低下するケースがあります。
本記事ではif~elseを使用することで発生する問題点を紹介します。
なお、今回記述するコードはRuby on Railsのケースで記述しています。

例えば、山田さんと中島さんと伊藤さんの３人の年齢を比較してその中で一番年齢が上の人を取得する場合はmax_byメソッドを使用して次のように書けます。

```ruby
  yamada = User.create!(name: "山田", age: 20)
  nakajima = User.create!(name: "中島", age: 40)
  ito = User.create!(name: "伊藤", age: 30)
  [yamada, nakajima, ito].max_by{|user| user.age} # nakajimaが返却される
```

max_byメソッドについては次のリンクを参照してください。

https://docs.ruby-lang.org/ja/latest/method/Enumerable/i/max_by.html

それでは山田さんと中島さんの２人の年齢を比較して、年齢が取得する人の名前を取得する場合はどうでしょうか？
上の誘導があれば次のように書けます。

```ruby
  max_age_user = [yamada, nakajima].max_by{|user| user.age} # nakajimaが返却される
  max_age_user.name # 中島が返却される
```

ところが、実際には次のように書かれるケースがあります。

```ruby
  if yamada.age > nakajima.age
    max_age_user = yamada
  else
    max_age_user = nakajima
  end
  max_age_user.name # 中島が返却される
```

おそらく2者間の条件分岐であれば、if~elseを使用することが多いと思います。
しかしながら個人的には可能な限りif~elseを使用したくないと思っています。

## if~elseを使用する問題点

### 理由①：可読性が低い

条件分岐を使用すると行数が増えるため、
コードの読み手に読解を強いる箇所が増えることになり、認知負荷が高くなります。

### 理由②：拡張性が悪い

条件分岐を使用すると、条件が増えるたびにコードを修正する必要があります。
if~elseを使用しているとelsifを追加する対応などをした結果、
コードが複雑になるケースがあります。

２つの理由について実際に遭遇したケースを見ていきます。

## 条件が追加された場合

別の開発者が条件追加に伴うコード修正をした場合に、
純粋に条件分岐のみを修正する可能性があります。
if~elseを使用した次のコードを考えます。

```ruby

if hoge.content.present? && fuga.content.present?
  "ホゲ・フガ"
elsif hoge.content.present? && !fuga.content.present?
  "ホゲ"
elsif !hoge.content.present? && fuga.content.present?
  "フガ"
end

```

hoge,fugaの2つのオブジェクトがある場合に、
それぞれのオブジェクトのcontentが存在する場合には該当する文字列を返却するという処理を行っています。
この時点ですでに可読性が低いですが、扱うオブジェクトが増えた場合にこのように書かれるケースがあります。

```ruby

if hoge.content.present? && fuga.content.present? && piyo.content.present?
  "ホゲ・フガ・ピヨ"
elsif hoge.content.present? && fuga.content.present?
  "ホゲ・フガ"
elsif hoge.content.present? && piyo.content.present?
  "ホゲ・ピヨ"
elsif fuga.content.present? && piyo.content.present?
  "フガ・ピヨ"
elsif hoge.content.present?
  "ホゲ"
elsif fuga.content.present?
  "フガ"
elsif piyo.content.present?
  "ピヨ"
end

```

このようにオブジェクトが増えると、条件分岐の数が増えていきます。
今回の場合は次のような対応をはじめから考えておくべきでした。

```ruby

tmp = []
tmp << "ホゲ" if hoge.content.present?
tmp << "フガ" if fuga.content.present?
tmp.join("・")

```

## ネストした場合

次のように条件分岐がネストするケースも考えられます。

```ruby
  hoge = Hoge.find_by(id: params[:hoge_id])
  fuga = Fuga.find_by(id: params[:fuga_id])
  if hoge.present? && fuga.present?
    if hoge.created_at > fuga.created_at
      name = hoge.name
      date = hoge.created_at.strftime("%Y年%m月%d日")
    else
      name = fuga.name
      date = fuga.created_at.strftime("%Y年%m月%d日")
    end
  elsif hoge.present?
    name = hoge.name
    date = hoge.created_at.strftime("%Y年%m月%d日")
  elsif fuga.present?
    name = fuga.name
    date = fuga.created_at.strftime("%Y年%m月%d日")
  end
```

このように書けば条件分岐をネストせずに書けます。

```ruby
  hoge = Hoge.find_by(id: params[:hoge_id])
  fuga = Fuga.find_by(id: params[:fuga_id])

  content = [hoge, fuga].compact.max_by(&:created_at)
  return unless content

  name = content.name
  date = content.created_at.strftime("%Y年%m月%d日")
```

## まとめ

2者間の条件を比較する場合はif~elseを使用することが多いと思います。
しかしながら可読性や拡張性を踏まえると配列などを活用して
早い段階で条件分岐を排除することが望ましいと考えています。
配列やオブジェクトにすると、条件がわかりにくくなるケースもありますので、
一概にこのような修正が良いとは言えませんが、
安易に使用するのではなく、
条件分岐を排除する方法を考えることをおすすめします。