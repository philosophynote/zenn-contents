---
title: "Railsのトランザクション内でfind_byしても最新データが見えない理由"
emoji: "🔒"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rails", "MySQL", "PostgreSQL", "SQL"]
published: false
---

Railsでトランザクションを使っていると、トランザクション内で`find_by`を再実行しているのに、
他の処理で更新されたはずのデータが取得できないことがあります。

最初はActive Recordのキャッシュや`find_by`の挙動を疑いましたが、
原因はデータベースのトランザクション分離レベルとMVCCの見え方にありました。

この記事では、Railsのトランザクション内でデータを取得したときに、
なぜ「トランザクション開始時点のようなデータ」が見えることがあるのかを整理します。

## 結論

`find_by`だから古いデータを返しているわけではありません。

同一トランザクション内のSELECTが、データベースの分離レベルによって、
同じスナップショットを見続けることがあります。
その場合、トランザクションの外で別の処理がコミットしても、
トランザクション内の後続の`find_by`では更新後の値が見えません。

特にMySQLのInnoDBでは、デフォルトの分離レベルが`REPEATABLE READ`です。
この分離レベルでは、同一トランザクション内の一貫性読み取りは、
最初の読み取りで確立されたスナップショットを読み続けます。

## 例

次のような`users`テーブルを考えます。

```ruby
create_table :users do |t|
  t.string :email, null: false
  t.string :status, null: false
  t.timestamps
end
```

初期状態では、ユーザーの`status`が`pending`だとします。

```ruby
User.create!(email: "sample@example.com", status: "pending")
```

### トランザクションA

```ruby
ActiveRecord::Base.transaction do
  user = User.find_by!(email: "sample@example.com")
  puts user.status
  # => "pending"

  sleep 10

  user = User.find_by!(email: "sample@example.com")
  puts user.status
  # MySQLのREPEATABLE READでは "pending" が見えることがある
end
```

### トランザクションB

トランザクションAが`sleep`している間に、別の処理で同じユーザーを更新してコミットします。

```ruby
User.find_by!(email: "sample@example.com").update!(status: "active")
```

トランザクションBの更新はコミット済みです。
しかし、トランザクションAの2回目の`find_by!`では`active`ではなく、
最初に見えていた`pending`が返ることがあります。

## find_byの問題ではない

この挙動を見ると、`find_by`が結果をキャッシュしているように感じます。
しかし、ポイントは`find_by`そのものではありません。

`find_by`はSQLを発行します。
問題は、そのSQLが参照するデータの見え方をデータベースが決めていることです。

例えば次のように`reload`したとしても、
同じトランザクション内の同じスナップショットを読んでいる限り、
期待した最新データが見えないことがあります。

```ruby
ActiveRecord::Base.transaction do
  user = User.find_by!(email: "sample@example.com")

  # 別トランザクションで user.status が更新されても、
  # 分離レベルによっては同じスナップショットの値を読み続ける。
  user.reload
end
```

Active Recordのオブジェクトを取り直すかどうかではなく、
トランザクションがどの時点のスナップショットを読んでいるかが重要です。

## トランザクション分離レベルとは

トランザクション分離レベルは、並行して実行されるトランザクション同士が、
互いの変更をどの程度見えるようにするかを決める設定です。

代表的な分離レベルには次のようなものがあります。

| 分離レベル | 概要 |
| --- | --- |
| READ UNCOMMITTED | 他トランザクションの未コミットデータが見える可能性がある |
| READ COMMITTED | コミット済みデータだけが見える |
| REPEATABLE READ | 同一トランザクション内で同じ行を読み直したときに同じ結果を得やすい |
| SERIALIZABLE | 直列実行に近い厳密な分離を行う |

このうち、今回の話で重要なのは`READ COMMITTED`と`REPEATABLE READ`です。

## MySQLの場合

MySQLのInnoDBでは、デフォルトの分離レベルは`REPEATABLE READ`です。

`REPEATABLE READ`では、同一トランザクション内の一貫性読み取りが、
最初の読み取りで確立されたスナップショットを読み続けます。

そのため、次のような流れになります。

1. トランザクションAを開始する
2. トランザクションAで`User.find_by!`を実行し、`status: "pending"`を見る
3. トランザクションBで同じユーザーを`status: "active"`に更新してコミットする
4. トランザクションAで再度`User.find_by!`を実行する
5. トランザクションAでは引き続き`status: "pending"`が見えることがある

厳密には、MySQLのInnoDBでは「トランザクション開始時」ではなく、
最初の一貫性読み取りでスナップショットが確立されます。
ただし、アプリケーションから見ると、
トランザクションの中で最初に見た時点のデータを読み続けているように見えます。

## PostgreSQLの場合

PostgreSQLのデフォルトの分離レベルは`READ COMMITTED`です。

`READ COMMITTED`では、各SQL文の開始時点でコミット済みのデータを見ます。
そのため、同一トランザクション内であっても、
2回目の`find_by!`を実行する前に別トランザクションの更新がコミットされていれば、
更新後の値が見えることがあります。

一方で、PostgreSQLでも`REPEATABLE READ`を指定した場合は、
トランザクション内で同じスナップショットを読み続けます。

```ruby
ActiveRecord::Base.transaction(isolation: :repeatable_read) do
  user = User.find_by!(email: "sample@example.com")

  # このトランザクション中は、同じスナップショットを見続ける。
  user = User.find_by!(email: "sample@example.com")
end
```

つまり、Railsのコードだけを見て判断するのではなく、
使っているDBと分離レベルを確認する必要があります。

## 最新の値を見たい場合

トランザクション内で常に最新の値を見たい場合は、
まず本当に同じトランザクション内で読み直す必要があるのかを考えます。

トランザクションは、複数の処理を一貫した単位として扱うための仕組みです。
そのため、途中で外部のコミット済み変更を見たい処理と、
一貫したスナップショットを維持したい処理は相性が悪い場合があります。

### トランザクションを分ける

最新データを読む必要がある処理は、トランザクションの外に出せないか検討します。

```ruby
ActiveRecord::Base.transaction do
  user = User.find_by!(email: "sample@example.com")
  user.update!(status: "processing")
end

# トランザクションを抜けてから読み直す。
user = User.find_by!(email: "sample@example.com")
```

処理全体を1つのトランザクションに閉じ込める必要がないなら、
この形が一番分かりやすいです。

### 分離レベルを変える

Railsでは、DBが対応していれば`transaction`に`isolation`を指定できます。

```ruby
ActiveRecord::Base.transaction(isolation: :read_committed) do
  user = User.find_by!(email: "sample@example.com")

  # DBによっては、後続のSELECTでより新しいコミット済みデータが見える。
  user = User.find_by!(email: "sample@example.com")
end
```

ただし、分離レベルを下げると、読み取りの一貫性も変わります。
最新データが見えるようになる代わりに、
同じトランザクション内で同じ問い合わせをしても結果が変わる可能性があります。

「最新が見たいから`READ COMMITTED`にする」と単純に決めるのではなく、
その処理で守りたい整合性を確認してから選ぶ必要があります。

### ロックを取る

他のトランザクションに更新されたくない行を扱う場合は、
読み取り時にロックを取る方法もあります。

```ruby
ActiveRecord::Base.transaction do
  user = User.lock.find_by!(email: "sample@example.com")
  user.update!(status: "processing")
end
```

`lock`は典型的には`SELECT ... FOR UPDATE`を発行します。
対象行を更新する前提がある場合は、単に読み直すよりも意図が明確になります。

ただし、ロックは待ち時間やデッドロックの原因にもなります。
対象範囲を小さくし、ロックを保持する時間を短くすることが重要です。

## 注意点

「トランザクション内では必ずトランザクション開始時のデータを取得する」
と覚えるのは少し危険です。

実際の挙動は、少なくとも次の要素で変わります。

- MySQLかPostgreSQLか
- 分離レベルが何か
- 通常のSELECTか、ロックを伴うSELECTか
- 読み取り専用の処理か、更新を伴う処理か
- そのトランザクション内ですでに同じデータを読んでいるか

特にMySQLの`REPEATABLE READ`では、
トランザクション内で最初に見たデータを読み続けるように見える場面があります。
この挙動を知らないと、
`find_by`や`reload`を書き直しても原因にたどり着きにくくなります。

## まとめ

Railsのトランザクション内で`find_by`を実行しても最新データが見えない場合、
まず疑うべきなのはActive Recordのメソッドではなく、
データベースのトランザクション分離レベルです。

MySQLのInnoDBのように`REPEATABLE READ`がデフォルトの場合、
同じトランザクション内のSELECTは同じスナップショットを読み続けることがあります。
その結果、別トランザクションでコミット済みの更新があっても、
トランザクション内では古い値が見えることがあります。

最新データが必要なのか、一貫した読み取りが必要なのかを整理したうえで、
トランザクションの範囲、分離レベル、ロックの使い方を選ぶのが良さそうです。

## 参考

- [Active Record API: Transaction isolation](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/DatabaseStatements.html)
- [MySQL 8.0 Reference Manual: Transaction Isolation Levels](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)
- [MySQL 8.4 Reference Manual: Consistent Nonlocking Reads](https://dev.mysql.com/doc/refman/8.4/en/innodb-consistent-read.html)
- [PostgreSQL Documentation: Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
