---
title: "Railsのコールバックメソッドを整理する"
emoji: "🔁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rails", "Ruby", "ActiveRecord"]
published: false
---

Railsのコールバックは、モデルのライフサイクルの特定のタイミングで処理を挟み込める仕組みです。
便利な反面、乱用するとデバッグが困難になるため、適切な理解と使いどころの見極めが重要です。

## コールバックとは

ActiveRecordモデルがデータベースへ保存・更新・削除される前後など、特定のタイミングで自動的に呼び出されるメソッドです。

```ruby
class User < ApplicationRecord
  before_save :normalize_email

  private

  def normalize_email
    self.email = email.downcase.strip
  end
end
```

`user.save` を呼ぶだけで `normalize_email` が自動的に実行されます。

## コールバックの種類

### 作成・更新・保存系

| コールバック | 実行タイミング |
|---|---|
| `before_validation` | バリデーション実行前 |
| `after_validation` | バリデーション実行後 |
| `before_save` | 保存前（create・update 両方） |
| `around_save` | 保存処理を囲む |
| `after_save` | 保存後（create・update 両方） |
| `before_create` | INSERTのSQLを実行する前 |
| `around_create` | INSERT処理を囲む |
| `after_create` | INSERTのSQLを実行した後 |
| `before_update` | UPDATEのSQLを実行する前 |
| `around_update` | UPDATE処理を囲む |
| `after_update` | UPDATEのSQLを実行した後 |

### 削除系

| コールバック | 実行タイミング |
|---|---|
| `before_destroy` | DELETEのSQLを実行する前 |
| `around_destroy` | DELETE処理を囲む |
| `after_destroy` | DELETEのSQLを実行した後 |

### 初期化・検索系

| コールバック | 実行タイミング |
|---|---|
| `after_initialize` | `new` や `find` でオブジェクトが生成された後 |
| `after_find` | `find` や `all` でレコードが取得された後 |

## 実行順序

`save` を呼んだ場合の実行順序は以下のとおりです。

```
before_validation
  → after_validation
  → before_save
    → before_create (初回保存の場合) または before_update (更新の場合)
    → (SQLが実行される)
    → after_create または after_update
  → after_save
→ after_commit (トランザクションコミット後)
```

### around コールバック

`around_*` はブロック形式で、`yield` の前後に処理を書きます。

```ruby
class Order < ApplicationRecord
  around_save :log_save_duration

  private

  def log_save_duration
    start = Time.current
    yield  # ← この yield で実際の保存処理が実行される
    Rails.logger.info "Save took #{Time.current - start}s"
  end
end
```

## コールバックをスキップする方法

### `update_columns` / `update_column`

コールバックとバリデーションをスキップしてSQLを直接実行します。

```ruby
user.update_columns(email: "new@example.com")
```

### `skip_callback`

特定のコールバックを一時的に無効化できます。テスト時などに使用します。

```ruby
User.skip_callback(:save, :before, :normalize_email)
user.save
User.set_callback(:save, :before, :normalize_email)
```

## `after_commit` と `after_save` の違い

`after_save` はトランザクション内で実行されますが、`after_commit` はトランザクションがコミットされた後に実行されます。

メール送信やWebhookなど、外部への副作用を伴う処理には `after_commit` を使うのが安全です。

```ruby
class User < ApplicationRecord
  # after_save だとロールバックされても実行されてしまう
  after_commit :send_welcome_email, on: :create

  private

  def send_welcome_email
    UserMailer.welcome(self).deliver_later
  end
end
```

`on:` オプションで `create` / `update` / `destroy` に絞ることができます。

## コールバックを止める方法

`before_*` コールバック内で `throw :abort` を呼ぶと、以降の処理とSQLの実行が止まります。

```ruby
class Payment < ApplicationRecord
  before_create :check_balance

  private

  def check_balance
    throw :abort if account.balance < amount
  end
end
```

`throw :abort` をしないと、`false` を返しても処理は止まりません（Rails 5以降の仕様）。

## よくある使いどころ

### データの正規化

```ruby
before_save :normalize_phone_number

def normalize_phone_number
  self.phone = phone.gsub(/[^0-9]/, "") if phone.present?
end
```

### 関連レコードの後処理

```ruby
after_destroy :cleanup_files

def cleanup_files
  attachments.each(&:purge)
end
```

### 監査ログの記録

```ruby
after_update :log_changes

def log_changes
  AuditLog.create!(record: self, changes: saved_changes)
end
```

## コールバックを使いすぎないために

コールバックは便利ですが、以下の点に気をつけることが重要です。

- **テストが難しくなる**: モデルを保存するだけで副作用が走るため、テストのセットアップが複雑になりがちです。
- **実行順序が把握しにくくなる**: コールバックが増えると処理の流れを追うのが困難になります。
- **意図しないタイミングで動く**: `find` 後に走る `after_find` などは気づきにくいバグの原因になります。

メール送信・外部APIコールなどの副作用が大きい処理は、コールバックではなくサービスオブジェクトや明示的なメソッド呼び出しで行うことを検討してみてください。

## まとめ

| 目的 | 推奨コールバック |
|---|---|
| 保存前にデータ整形 | `before_save` / `before_validation` |
| 作成後に関連処理 | `after_create` |
| 外部への副作用（メール等） | `after_commit` |
| 削除後のリソース解放 | `after_destroy` |

コールバックはRailsの強力な機能ですが、使いすぎるとモデルが「何をするかわからない塊」になりがちです。
処理の意図が明確になるよう、コールバックの責務を絞り込んで使うことをおすすめします。
