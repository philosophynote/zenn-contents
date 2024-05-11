---
title: "【備忘録】group concatメソッド"
emoji: "🏘️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["SQL","MySQL"]
published: false
---

仕事でBIツール [MetaBase](https://www.metabase.com/)を使用し、
非エンジニア向けにダッシュボードやエクセルのエクスポート機能を提供しています。

先日記事に紐づくタグを1行で表示してほしいという要望があり、
`group_concat`メソッドを使用したので使い方をまとめました。

## group_concatメソッドとは

mysqlの公式ドキュメントには次のように記載されています。

> この関数は、グループから連結された非 NULL 値を含む文字列の結果を返します。 非 NULL 値がない場合は、NULL を返します。 

https://dev.mysql.com/doc/refman/8.0/ja/aggregate-functions.html#function_group-concat

```sql
GROUP_CONCAT([DISTINCT] expr [,expr ...]
             [ORDER BY {unsigned_integer | col_name | expr}
                 [ASC | DESC] [,col_name ...]]
             [SEPARATOR str_val])
```

複数行のデータを1行で表したい場合に使用します。
対象のデータに対して`distinct`を指定することで重複を除外したり、
`order by`で並び替えを行うことができます。

```sql
SELECT student_name,
  GROUP_CONCAT(test_score)
FROM student
GROUP BY student_name;
```

### 例

ChatGPTに生成してもらった次のデータで動作を試します。

```sql

-- ブログ記事を格納するテーブル
CREATE TABLE blog_posts (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,
    post_date DATE NOT NULL,
    author VARCHAR(100)
);

-- タグを格納するテーブル
CREATE TABLE blog_post_tags (
    id INT AUTO_INCREMENT PRIMARY KEY,
    post_id INT,
    tag_name VARCHAR(100),
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (post_id) REFERENCES blog_posts(id)
);

```

![blog](/images/blog.png)
![tag](/images/tag.png)

ブログ記事のテーブルのIDでグループ化して、
タグをカンマ区切りで表示する例が次のクエリです。

```sql

SELECT 
    bp.title as タイトル,
    bp.content as 本文,
    GROUP_CONCAT(bpt.tag_name SEPARATOR ', ') AS タグ
FROM 
    blog_posts bp
INNER JOIN 
    blog_post_tags bpt ON bp.id = bpt.post_id
GROUP BY 
    bp.id

```

![group_concat](/images/group_concat.png)

記事に紐づくタグがタグカラムにカンマ区切りで表示されています。
NULLは表示されていません。
重複しているカラムがある場合は`distinct`を指定することで重複を除外できます。

```sql
GROUP_CONCAT(DISTINCT bpt.tag_name SEPARATOR ', ') 
```

また、タグを最新順に並べたい場合は`order by`を指定します。

```sql
GROUP_CONCAT(bpt.tag ORDER BY bpt.created_at SEPARATOR ', ') 
```

## 余談

ChatGPTに設定を記憶させる「Memory」機能が
４月末にPlusプランの正式機能としてリリースされました。

https://www.itmedia.co.jp/news/articles/2404/30/news080.html

記憶させると具体例を作成させる際に私の趣味嗜好を反映してくれるので、
少し楽しいです
（もっと革新的な使い方が間違いなくあると思いますが...😅）
