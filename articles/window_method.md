---
title: "【備忘録】ウィンドウ関数の活用例"
emoji: "🪟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["SQL","MySQL"]
published: false
---

仕事でBIツール [MetaBase](https://www.metabase.com/)を使用し、
非エンジニア向けにダッシュボードやエクセルのエクスポート機能を提供しています。

最近、以下のような要望が続いたため、ウィンドウ関数の活用例を自分用にまとめました。

- ある時点での最新のデータを1件表示してほしい
- 折れ線グラフの変動が激しいので移動平均線が欲しい

なお、本記事ではMySQL 8系でのウィンドウ関数の利用を前提としています。

## ウィンドウ関数とは

ウィンドウは、選択されたレコードの集合に対して、
順序付けやフレーム定義が行なわれたうえでのデータセット
を指します。

ウィンドウ関数は次の3つの要素で構成されます。

①：PARTITION BY句によるレコード集合のカット
②：ORDER BY句によるレコードの順序付け
③：フレーム句によるカレントレコードを中心としたサブセットの定義

### 例

```sql
SELECT
  sample_date AS  cur_date,
  MIN(sample_date)
  OVER(
    PARTITION BY server_id
    ORDER BY sample_date ASC 
    ROWS BETWEEN 1 PRECEDING AND 1 PRECEDING
  ) AS latest_date
FROM LoadSample;
```

(参考書籍 「2 必ずわかるウィンドウ関数 フレーム句を使って違う行を自分の行に持ってくる」章のSQL文にPARTITION BY句を追加)

`server_id`(サーバー)ごとの評価点数の過去1週間の移動平均を取得するクエリです。
`PARTITION BY`句で`server_id`ごとにデータを分割（①）し、
`ORDER BY`句で`sample_date`で昇順に並べ替え（②）、
`ROWS BETWEEN`句で範囲を指定（③）しています。

## 主なウィンドウ関数

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

## フレーム句の主なオプション

| オプション名       | 説明                                                              |
| ------------------ | ----------------------------------------------------------------- |
| ROWS               | 移動単位を行で設定する                                            |
| RANGE              | 移動単位を列の値で設定する。基準となる列はORDERBY句で指定された列 |
| nPRECEDING         | nだけ前へ（小さいほう）へ移動する。nは正の整数                    |
| nFOLLOWING         | nだけ後へ（大きいほう）へ移動する。nは正の整数                    |
| UNBOUNDEDPRECEDING | 無制限にさかのぼるほうへ移動する                                  |
| UNBOUNDEDFOLLOWING | 無制限に下るほうへ移動する                                        |
| CURRENTROW         | 現在行                                                            |

(参考書籍「2 必ずわかるウィンドウ関数 フレーム句を使って違う行を自分の行に持ってくる」章より引用)

## 具体的な利用ケース

飲食店の評価データを例に、ウィンドウ関数を利用したクエリを紹介します。
データ内容はChatGPTが作成した架空のデータです。

![ER図](/images/restaurant_er.png)

restaurant_mastersテーブル

- name: 飲食店名

![restaurant_masters](/images/restaurant_masters.png)

restaurant_ratingsテーブル

- restaurant_master_id: 飲食店ID(restaurant_mastersテーブルの外部キー)
- rating_date: 評価日
- rating_score: 評価点数

![restaurant_ratings](/images/restaurant_ratings.png)

restaurant_eventsテーブル

- restaurant_master_id: 飲食店ID(restaurant_mastersテーブルの外部キー)
- content:ニュースや口コミの内容
- event_date:発生日

![restaurant_rates](/images/restaurant_events.png)

### 1. ある出来事があった直後のデータを取得する

ニュースや口コミの内容が更新された直前と直後の評価データを取得するクエリです。

```sql
SELECT
    name,
    before_score,
    content,
    after_score
FROM
 (
    select
    rm.id as master_id,
    rm.name,
    rr.rating_score as before_score,
    re.content,
    SUM(rating_score) over (
      PARTITION BY rr.restaurant_master_id
      order by rating_date asc
      range between interval 1 day following and interval 1 day following
    ) as after_score,
    event_date,
    rr.rating_date
    from horecast_development.restaurant_ratings rr
    join restaurant_masters rm
    on rr.restaurant_master_id = rm.id
    join restaurant_events re
    on re.restaurant_master_id = rm.id
  ) tmp
where rating_date = event_date 
order by master_id;
```

`range between interval 1 day following and interval 1 day following`で、
出来事の直後のデータを取得しています。

![restaurant_event_study](/images/restaurant_event_study.png)

### 2. 移動平均を作成する

飲食店ごとの評価点数の過去1週間の移動平均を取得するクエリです。

```sql
SELECT
  restaurant_master_id,
  rating_date,
  rating_score,
  AVG(rating_score) OVER (
    PARTITION BY restaurant_master_id
    ORDER BY rating_date
    RANGE BETWEEN INTERVAL 6 DAY PRECEDING AND CURRENT ROW
  ) AS moving_avg
FROM
  restaurant_ratings;
```

![moving_avg](/images/moving_avg.png)

### 3. ランキングを取得する

飲食店ごとの評価点数のランキングを取得するクエリです。

```sql
SELECT
  name,
  rating_date,
  rating_score,
  RANK() OVER (
    PARTITION BY restaurant_master_id
    ORDER BY rating_score DESC
  ) AS each_restaurant_rating_rank
FROM
  restaurant_ratings re
join restaurant_masters rm
on re.restaurant_master_id = rm.id
order by restaurant_master_id,rating_date asc
```

![restaurant_rank](/images/restaurant_rank.png)

## 参考書籍・URL

- [達人に学ぶSQL徹底指南書 第2版 初級者で終わりたくないあなたへ](https://www.shoeisha.co.jp/book/detail/9784798157825)
- [MySQL :: MySQL 8.0 リファレンスマニュアル :: 12.21 ウィンドウ関数](https://dev.mysql.com/doc/refman/8.0/ja/window-functions.html)
