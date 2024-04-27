---
title: "【備忘録】ウィンドウ関数を利用して時系列データを表現する"
emoji: "🪟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["SQL","MySQL"]
published: false
---

仕事で[MetaBase](https://www.metabase.com/)というBIツールを使用して
非エンジニア向けにダッシュボードやエクセルのエクスポート機能を提供しています。

「◯月時点の点数を表示して」
「折れ線グラフの上下が激しくて読みづらいから移動平均線を作成して」
「最新の1件を表示して」
といった要望が連続したため、
今回はウィンドウ関数を利用して時系列データを表現する方法を紹介します。
MySQL8系での利用を前提としています。

## ウィンドウ関数とは

ウィンドウ:FROM句から選択されたレコードの集合に対して、
ORDERBYによる順序付けやROWSBETWEENによるフレーム定義が行なわれたうえでのデータセット

①：PARTITIONBY句によるレコード集合のカット
②：ORDERBY句によるレコードの順序付け
③：フレーム句によるカレントレコードを中心としたサブセットの定義

1,2は従来のGROUPBY句やORDERBY句と同じですが、
フレーム句：データの特定の範囲

```sql
SELECT
  restaurant_master_id,
  rating_date,
  rating_score,
  AVG(rating_score) OVER (
    PARTITION BY restaurant_master_id
    ORDER BY rating_date
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
  ) AS rolling_avg_score
FROM
  restaurant_ratings;
```

`PARTITION BY`句で`restaurant_master_id`ごとにデータを分割（①）し、
`ORDER BY`句で`rating_date`で昇順に並べ替え（②）、
`ROWS BETWEEN`句で直近7日間のデータを取得（③）しています。

フレーム句で利用できる主なオプション(参考書籍より引用)

| オプション名       | 説明                                                              |
| ------------------ | ----------------------------------------------------------------- |
| ROWS               | 移動単位を行で設定する                                            |
| RANGE              | 移動単位を列の値で設定する。基準となる列はORDERBY句で指定された列 |
| nPRECEDING         | nだけ前へ（小さいほう）へ移動する。nは正の整数                    |
| nFOLLOWING         | nだけ後へ（大きいほう）へ移動する。nは正の整数                    |
| UNBOUNDEDPRECEDING | 無制限にさかのぼるほうへ移動する                                  |
| UNBOUNDEDFOLLOWING | 無制限に下るほうへ移動する                                        |
| CURRENTROW         | 現在行                                                            |

## 主なウィンドウ関数

主なウィンドウ専用関数は次の通りです。

| 関数名      | 説明                                               |
| ----------- | -------------------------------------------------- |
| ROW_NUMBER  | パーティション内の現在の行数                       |
| RANK        | パーティション内の現在の行のランク  (ギャップあり) |
| DENSE_RANK  | パーティション内の現在の行のランク  (ギャップなし) |
| FIRST_VALUE | ウィンドウフレームの最初の行からの引数の値         |
| LAST_VALUE  | ウィンドウフレームの最後の行からの引数の値         |
| NTH_VALUE   | ウィンドウフレームの N 番目の行からの引数の値      |
| LEAD        | パーティション内の現在の行の先頭行からの引数の値   |
| LAG         | パーティション内の現在行より遅れている行の引数の値 |

この他にも`SUM`や`AVG`などの集計関数をウィンドウ関数として利用することもできます。

## 相関サブクエリ・自己結合との比較

ウィンドウ関数は相関サブクエリや自己結合と比較して以下のメリットがあります。

## 具体的な利用ケース

### 1. 直近のデータを取得する

```sql
SELECT
  id,
  name,
  created_at,
  LAG(created_at) OVER (ORDER BY created_at) AS previous_created_at
FROM
  users;
```

### 2. ある時点におけるデータを取得する
  
```sql
SELECT
  id,
  name,
  created_at,
  LAG(created_at) OVER (ORDER BY created_at) AS previous_created_at
FROM
  users;
```

### 3. 移動平均を取得する

```sql
SELECT
  restaurant_master_id,
  rating_date,
  rating_score,
  AVG(rating_score) OVER (
    PARTITION BY restaurant_master_id
    ORDER BY rating_date
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
  ) AS rolling_avg_score
FROM
  restaurant_ratings;
```

### 4. ランキングを取得する

```sql
SELECT
  id,
  name,
  score,
  RANK() OVER (ORDER BY score DESC) AS rank
FROM
  users;
```

## 参考書籍

- [達人に学ぶSQL徹底指南書 第2版 初級者で終わりたくないあなたへ](https://www.shoeisha.co.jp/book/detail/9784798157825)