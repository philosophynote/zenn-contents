---
title: "find_eachとpreloadを併用したらN+1問題が解決しなかった"
emoji: "🧑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rails"]
published: false
---

## 目的

この記事では、Railsでよく話題になる「N+1問題」と、
大量レコードを扱う際に使用されるfind_eachやfind_in_batches、in_batchesなどのメソッドを組み合わせた時に起きる落とし穴について解説します。
具体的には「事前にpreloadしているのに、find_eachでループするとN+1問題が発生するのはなぜか？」を理解し、解決策を提示することを目的としています。

## 結論

- **find_each / find_in_batches / in_batches**は大量データをメモリ節約しながら処理できるが、関連テーブルのロードタイミングを誤るとN+1問題が再発する。
- バッチ処理内で必要な関連を一括ロードするか、バッチサイズごとに適切に関連をロードする仕組みを設計すれば、N+1問題を回避しながらメモリを節約できる。

## 基礎知識

### find_each, find_in_batches, in_batchesの違いについて

- find_each
  - 概要
    - デフォルトで1000件ずつのバッチに分割してレコードを取得し、1件ずつブロック内で処理します。
    - 処理単位が細かいので、1レコードごとの処理フローをそのまま書きやすいです。
  - 例

  ```ruby
    Model.find_each(batch_size: 1000) do |record|
      # recordは1件ずつブロックに渡される
      # 各レコードに対して何らかの処理を行う
    end
  ```

  - 特徴
    - 大量データでも一度に全件を読み込まないため、メモリ使用量を抑えられる。
    - 1件ずつ処理する分、ループの中で関連テーブルを参照するとN+1が起きやすい場面もあるため注意。

- find_in_batches
  - 概要
    - バッチサイズごとにレコードを取得し、「1バッチ（配列）」単位でブロックに渡します。
    - バッチ単位でまとめて処理したい場合に使いやすいです。
  - 例

  ```ruby
    Model.find_in_batches(batch_size: 1000) do |batch|
      # batch は 1000件程度の配列
      batch.each do |record|
        # バッチ内の各レコードを処理
      end
    end
  ```

  - 特徴
    - ブロックに渡されるbatchは配列（またはRelationを.to_aしたもの）なので、1バッチごとに一括処理したい場合は便利。
    - 1バッチが終わるたびに次のバッチを取りに行く形なので、メモリ使用量の上限を抑えられる。

- in_batches
  - 概要
    - バッチ操作に対してさらに柔軟に制御できるメソッド。
    - デフォルトではブロックにActiveRecord::Relationが渡され、必要に応じて.to_aで配列化したり、追加のwhereなどをチェーンしたりできます。
    - `load: true`オプションを指定すると、その場でレコードを配列としてロードし、遅延ロードを行わずに使えるようになります。
  - 例

  ```ruby
    # load: true の場合
    Model.in_batches(of: 1000, load: true).each do |batch|
      # batchはすでに配列化されているイメージ
      batch.each do |record|
        # バッチ内のレコードを処理
      end
    end

    # load: false (デフォルト) の場合
    Model.in_batches(of: 1000).each do |batch|
      # batch は ActiveRecord::Relation
      filtered = batch.where(status: :active)
      filtered.each do |record|
        # 条件を追加した上で処理
      end
    end
  ```

  - 特徴
    - find_in_batchesよりも低レベルで柔軟に扱える。
    - `load: true`を使うとバッチをすぐに配列化できるので、あとでN+1が起きるタイミングをより制御しやすい。

### 何故メモリを節約できるのか

- 大量データを一度に読み込むとメモリを大量に消費しますが、
  これらのバッチ処理系メソッドで「一定件数ずつ」取得するようにすれば、一度に扱うレコード量を抑えられます。
- 例: find_each(batch_size: 1000)なら、1000件ずつDBから取得するため、アプリケーションメモリを過度に圧迫しない。

### 何故N+1問題を解決できるのか

- Railsのpreload / includes / eager_loadで関連先を事前ロードしておけば、ループ内で関連モデルを参照しても追加クエリが発行されません。
  - preload: 主テーブル → 関連テーブルを別クエリでまとめて取得
  - includes: Railsが状況に応じてpreload or eager_loadを選択
  - eager_load: JOINを使って1回のSQLでまとめて取得
- こうした「一括ロード」により、1件ずつ関連にアクセスするたびにクエリが走る「N+1問題」を回避できます。

## 今回の原因

### 何故find_eachとpreloadの組み合わせではN+1問題を解決できなかったのか

- find_eachやfind_in_batchesなどのバッチ処理系メソッドは、バッチごとにクエリを実行します。
- 事前ロードに使うpreloadは、基本的に「対象テーブルの全レコード」を1回で取得し終わる想定。
- しかし、バッチを分割して取得していると、ループ内で関連モデルへ都度アクセスするときにバッチごとに関連ロードが起きたり、アクセスのタイミングがずれてN+1が再発したりします。
- さらに、大量データに対してpreloadを一気に走らせるのもメモリ上の負荷が大きいため、単純なpreloadを組み合わせるだけでは対処できないケースがあります。

## 今回の解決策

- BlogPostをバッチ処理で読み込みつつ、ユーザーのAlertKeywordやTagをうまくロードし、RecommendPostを作成するときにN+1を回避する
- ポイントは、**「必要なバッチが確定したタイミングで、必要な関連テーブルをロードする」**という流れを明確にすることです。

```ruby
class RecommendPostBatchService
  BATCH_SIZE = 1000

  def create_recommend_posts_for_blog_posts
    # BlogPostをバッチ単位で取得
    BlogPost.in_batches(of: BATCH_SIZE, load: true).each do |batch|
      # batchはこの時点で配列化されているので .to_a は不要
      blog_posts_in_batch = batch

      # 例: ユーザーのAlertKeywordやTagを一度に取得する場合
      # 事前にUserを全て preload するのは膨大になるかもしれないので注意が必要。
      # 条件付きでロードしたい場合は、ここで絞り込みを行う。
      users = User.preload(:alert_keywords, :follow_tags) 
      # 全Userを取得する設計が厳しい場合は、別途必要なユーザーのみ抽出する仕組みを考える

      blog_posts_in_batch.each do |blog_post|
        # ここでAlertKeywordやTagに部分一致するかを判定
        # (例) user毎にalert_keywordsを持っている想定
        users.each do |user|
          # user.alert_keywordsからkeyword抽出
          # user.follow_tagsからtag_name抽出
          # blog_post.title / blog_post.content に含まれるかどうかをチェック
          next unless match_keyword_or_tag?(blog_post, user)

          # マッチしたらRecommendPostを作成する
          RecommendPost.create!(
            blog_id: blog_post.id,
            user_id: user.id
            # 必要に応じてその他のカラムをセット
          )
        end
      end
    end
  end

  private

  def match_keyword_or_tag?(blog_post, user)
    # alert_keywordsやタグ名が blog_post.title or blog_post.content に含まれているかチェック
    keywords = user.alert_keywords.map(&:keyword)
    tags = user.follow_tags.map { |ft| ft.tag.name }
    (keywords + tags).any? do |kw|
      blog_post.title.include?(kw) || blog_post.content.include?(kw)
    end
  end
end
```

1.BlogPostをin_batchesで細かくロードし、1バッチずつ処理することでメモリを節約します（load: trueオプションで配列化しているため、ループ内で意図しないタイミングのクエリ発行が抑えられる）。
2.ユーザー情報については、全ユーザーに対してpreload(:alert_keywords, :follow_tags)をしている例です。ただし、ユーザー数も膨大な場合は別の工夫が必要です。例えば「特定の条件を満たすユーザーだけを先に取り出す」など、バッチ処理を更に細かく分割すると良いでしょう。
3.ループ内では、すでにロード済みのalert_keywordsやfollow_tagsを使うため、N+1問題が発生しなくなる。