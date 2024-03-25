---
title: "【イベント参加レポート】Object-Oriented Conference 2024"
emoji: "💊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["オブジェクト指向","ドメイン駆動設計","設計","ooc"]
published: false
---

2024/3/24に開催された[Object-Oriented Conference 2024](https://ooc.dev/2024/) の感想です。
普段Rubyを使用して開発しているにも関わらず、
オブジェクト指向に関する知識が不足していたため
良いきっかけになればと思い、参加しました。

## 聴講したセッション

### オブジェクト指向のリ・オリエンテーション　～歴史を振り返り、AI時代に向きなおる～

オブジェクト指向に関する知識が不足していたため、
歴史的な経緯を学びながら
オブジェクト指向の基本的な考え方を理解することができました。
SOLID原則から曖昧な状態であったため
OCPやリスコフの置換原則、インターフェースの重要性について理解することができました。

責務(responsibility)をオブジェクトに任せる
=responseするability(反応して仕事をする)を持たせる
という話は言われて初めて気がつきました。

また、『走る』という動詞は主語が人・車・プログラムと変わるが、
抽象的な観点で見ると同じ行動をしているという話が
オブジェクト指向の考え方を端的に表現していると感じました。

### 生成AIの不確実性と向き合うためのオブジェクト指向設計

生成AIの解決領域
A(自動化)・A(代行)・A(助言)・A(強化)に対して
増田さんの『ざっくりとでも、具体的すぎでもない』というコメントが印象的でした。

生成AIの活用を考える際、よく見かける情報は確かに
抽象的すぎるか、具体的すぎるかのどちらかであることが多いです。

今後はこの観点でも活用方法を検討していきたいと思いました。

### 現実世界の事象から学ぶSOLID原則

https://twitter.com/H0R15H0/status/1771737675017691382

前述した通り、SOLID原則から理解が曖昧であったため、
良い勉強になりました。
OCPの『拡張に対しては開いていて修正に対しては閉じていなければいけない』という意味がよくわかっていなかったのですが、
現実世界の例とサンプルコードを見ることで
どのような点が問題であるかを明確に理解することができました。

### クソコード動画『カプセル化 Mk-II』で考える、上手くカプセル化できない理由

https://twitter.com/MinoDriven/status/1771764585097535685

ミノ駆動さんのセッションに参加するのはこれで3回目でしたが、
今回も非常に面白かったです。

以前参加したセッションで『単一責任の原則とは単一目的の原則』であるという話が印象に残っていますが、
今回もシステム設計において、『目的』に着目することの重要性を改めて感じました。
モデリングのために紹介された次の書籍は設計以外でも
ビジネス層とのコミュニケーションでも役立っています。

[「具体⇔抽象」トレーニング 思考力が飛躍的にアップする29問](
https://www.amazon.co.jp/%E3%80%8C%E5%85%B7%E4%BD%93%E2%87%94%E6%8A%BD%E8%B1%A1%E3%80%8D%E3%83%88%E3%83%AC%E3%83%BC%E3%83%8B%E3%83%B3%E3%82%B0-%E6%80%9D%E8%80%83%E5%8A%9B%E3%81%8C%E9%A3%9B%E8%BA%8D%E7%9A%84%E3%81%AB%E3%82%A2%E3%83%83%E3%83%97%E3%81%99%E3%82%8B29%E5%95%8F-PHP%E3%83%93%E3%82%B8%E3%83%8D%E3%82%B9%E6%96%B0%E6%9B%B8-%E7%B4%B0%E8%B0%B7-%E5%8A%9F-ebook/dp/B0868GMSBG?ref_=ast_author_mpb)

今回取り上げられた次の書籍も読みたいと思います。

[目的ドリブンの思考法](
https://www.amazon.co.jp/%E6%88%A6%E7%95%A5%E3%82%B3%E3%83%B3%E3%82%B5%E3%83%AB%E3%82%BF%E3%83%B3%E3%83%88%E3%81%8C%E5%A4%A7%E4%BA%8B%E3%81%AB%E3%81%97%E3%81%A6%E3%81%84%E3%82%8B-%E7%9B%AE%E7%9A%84%E3%83%89%E3%83%AA%E3%83%96%E3%83%B3%E3%81%AE%E6%80%9D%E8%80%83%E6%B3%95-%E3%80%90DL%E7%89%B9%E5%85%B8-%E6%9C%AA%E5%8F%8E%E9%8C%B2%E5%8E%9F%E7%A8%BF-%E6%80%9D%E8%80%83%E3%81%AE%E5%9C%B0%E5%9B%B3%E3%80%91-%E6%9C%9B%E6%9C%88%E5%AE%89%E8%BF%AA-ebook/dp/B09ST7977S/ref=sr_1_1?crid=19U6FCTPGZKIJ&dib=eyJ2IjoiMSJ9.KZuYzN_Zzm7rZNrLmUDZlX9Wn73Zje3d89TPCUfDyFalFCXuQI4ZHGYj9fIYp3ePv4dUmszYlwcdB8_U178sf0UrEhdcNmRNmsvnjQIki4-1a5IgSnJgcNORT0dqQlqmJupdyoiXTDFjbIWy49LhMXDs6hVSfrrEzHHYKhlrfqlcQMH7rnZp_ZYzuIzJHW-j4cLpzUfjokTrhibA8tKxn59BnBtl9iYjF3V_hGP6Zdk.n9oXhKnN__eY69qgK8Ktfofnl144VEibXYI4dCvvT-o&dib_tag=se&keywords=%E7%9B%AE%E7%9A%84%E3%83%89%E3%83%AA%E3%83%96%E3%83%B3%E3%81%AE%E6%80%9D%E8%80%83%E6%B3%95&qid=1711288844&s=digital-text&sprefix=%E7%9B%AE%E7%9A%84%E3%83%89%E3%83%AA%E3%83%96%E3%83%B3%E3%81%AE%2Cdigital-text%2C295&sr=1-1)

### チーム開発でデプロイ頻度を上げるための設計とタスク分割

https://twitter.com/kotomin_m/status/1771808758051524760

インターフェースを設計・作成することにより
タスクを分割して、属人化を防ぎデプロイ頻度を上げることに成功したという内容でした。

リリース単位が大きくなることで生じるデメリットや属人化の問題については
自分も強い課題感を抱いており、
その解決策として勉強になりました。

最後の質疑応答で設計期間について伺ったのですが
約２ヶ月（実装との並行期間あり）とのことで
効率的なチーム開発を行う上では時間をかけた設計が必要であることを学びました。

### 設計の知識と技能で駆動するソフトウェア開発

https://twitter.com/masuda220/status/1771697817112793557

技術書を読んでいるだけでは設計ができるようにはならず、
実際に手を動かして体で覚えることや知識・経験を共有することが重要であることを実感しました。
ポート&アダプターがヘキサゴナルアーキテクチャのことを示していることを初めて知るくらい
設計に関する知識が不足していたため、
これから学んでいこうと思います。

セッション中に紹介されて興味を持った書籍

[ノンデザイナーズ・デザインブック](https://www.amazon.co.jp/%E3%83%8E%E3%83%B3%E3%83%87%E3%82%B6%E3%82%A4%E3%83%8A%E3%83%BC%E3%82%BA%E3%83%BB%E3%83%87%E3%82%B6%E3%82%A4%E3%83%B3%E3%83%96%E3%83%83%E3%82%AF-%E7%AC%AC4%E7%89%88-Robin-Williams/dp/4839955557)

技術的負債=分析モデルと設計モデルの差

https://t-wada.hatenablog.jp/entry/ward-explains-debt-metaphor

### ドメイン・ファーストで考える問題解決に役立つモデル設計

@[speakerdeck](domain-first-model-design)

戦術的プログラミングと戦略的プログラミングの話が印象に残りました。
サービスやプロダクトがどのように発展していくかは不明なため、
最初は前者の手法が採用されますが、
時間が経過した後で後者の手法で最善の設計を検討することが必要であることを学びました。
生成AIに開発全体を任せるだけではなく、設計の壁打ちの相手として使用するといった点も
現代的で勉強になりました。

### オブジェクト指向コードレビューの新しいアプローチ

https://twitter.com/akkiee76/status/1771564863443063115

すでに日常業務で生成AIによるコードレビューを使用していますが、
レビューの内容に統一感がないことを課題に感じており、タイムリーな内容でした。
公式ガイドブックの内容も参考にしながら使用しているプロンプトの見直しに着手したいと思います。

## 総評

https://twitter.com/philosophy_note/status/1771853746403786811

初めてオフラインの技術カンファレンスに参加できました！
オフラインの会場で登壇した人に直接質問したり、
スポンサーブースで様々の企業の方と話すことができて、
非常に楽しい一日でした！

各セッションの感想にも記載した通り、オブジェクト指向に関する幅広い知識を得ることができ、
学習意欲が大いに高まりました！
良いコード・設計ができるように精進します！