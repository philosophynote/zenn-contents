---
title: "【イベント参加レポート】Kaigi on Rails 2023"
emoji: "🛤️"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Rails","kaigionrails"]
published: false
---

2023/10/27, 28 に行われた Kaigi on Rails の感想です。

## 聴講したセッション

### [Simplicity on Rails - RDB, REST and Ruby](https://speakerdeck.com/moro/simplicity-on-rails-rdb-rest-and-ruby)

CRUDを主体としたRailsが得意とする設計について改めて学習することができました。
個人的にはForm ObjectやService ClassといったFat Modelへの対応策について
「Rubyとして複雑さに向き合えば良い」といったコメントが印象的でした。

多々多のリレーションを持つテーブルおよびモデルをイベントエンティティという観点で捉える
ことも勉強になりました。

### [seeds.rbを書かずに開発環境データを整備する](https://speakerdeck.com/gogutan/seeds-dot-rbwoshu-kazunikai-fa-huan-jing-detawozheng-bei-suru)

仕事で開発環境で共通で使用できるデータを用意したいという課題があり、
タイムリーな内容でした。
外部キー制約が原因でそのままではデータを投入できないという問題への対応や
実行のたびに発生する問題に対して1つずつ解決していく様子が参考になりました.

dumpファイルでのバックアップでも対応できるというコメントを終了後にXで見かけて、
その通りだと思いました。

### [32個のPRでリリースした依存度の高いコアなモデルの安全な弄り方](https://speakerdeck.com/shoheimitani/32ge-noprderirisusitayi-cun-du-nogao-ikoanamoderunoan-quan-nanong-rifang)

まずオンラインDDLをこれまで理解していなかったのでその点で勉強になりました。
(当たり前のようにオンラインだと思っていました)
加えてリリースに向けて注意すべき点を整理し、対応策を考案した上で
実際にリリースしていく様子が勉強になりました。

### [ペアプロしようぜ 〜3人で登壇!? 楽しくて速いペアプロ/モブプロ開発〜](https://speakerdeck.com/masuyama13/pair-mob-programming-kaigi-on-rails-2023)

ユニークさという点ではトップクラスだったと思います。
終了1秒前にtestが通過した瞬間は事前に仕組んでいたのかと思いました。
実際の発表内容についても、ペアプロの実際の進め方をリアルタイムで見ることができ、
自分でペアプロを実施する際の参考になりました。
すぐにコーディングに取り掛かるのではなく、issueやタスクを整理してタイピストとナビの間で
共通認識を作る点が特に参考になりました。

### [コードカバレッジ計測ツールを導入したらテストを書くのが楽しくなった話](https://speakerdeck.com/duckfalcon/kodokabaretuziji-ce-turuwodao-ru-sitaratesutowoshu-kunogale-sikunatutahua)

コードカバレッジを測定することの重要性はしばしば目にしていましたが、
実際に可視化するとどのような影響があるのかを知ることができました。
テストを書く意味は理解されているので、
長い眼でカバレッジを意識していくことが重要だと感じました。

## 資料だけ確認したセッション

### [Rails アプリの 5,000 件の N+1 問題と戦っている話](https://speakerdeck.com/makicamel/bulletmarkrepairer-auto-corrector-for-n-plus-1-queries)

N+1問題の解決を自動化できないかといった内容でした。
includesを記述する箇所の整理が勉強になりました。
また、会社でもincludes VS eager_load or preloadの議論があり、
どこでも議論されていることを再認識できました。

### [Exceptional Rails](https://speakerdeck.com/willnet/exceptional-rails)

Railsを学習する際に次の記事で例外処理を学習しました。
[Railsアプリケーションにおけるエラー処理（例外処理）の考え方](https://qiita.com/jnchito/items/3ef95ea144ed15df3637)
今回の資料でも同じように正常系/準異常系/異常系を分けて考えられており、
学習した内容を再確認できました。
加えて、エラートレッキングサービスとの連携やRack層での例外処理について
勉強することができました。

### [Fat_Modelを解消するためのCQRSアーキテクチャ](https://speakerdeck.com/krpk1900/fat-modelwojie-xiao-surutamenocqrsakitekutiya)
Fat_ControllerやFat_Modelへの対応方法として勉強になりました。
責務を明確にしてテストを書きやすくする点は魅力的に思いました。
ただ、今回初めて見たアーキテクチャだったので
参考にしている書籍などがあれは教えてほしかったです。

### [管理機能アーキテクチャパターンの考察と実践 / Learn Architecture through Admin](https://speakerdeck.com/ohbarye/learn-architecture-through-admin)
13枚目の「管理機能、後追いで作られがち」のスライドの状況がそのまま該当していて面白かったです。
エンジニアがSQLを操作して保守対応している内容やユーザーが処理している内容を
そのまま実装すればいいだろうと思っていましたが、
今回の発表ではアーキテクチャの理論的な話から複数の解決策が実際には存在しており、
状況に応じてどの解決策がベストかを考える必要があることを学びました。
同時に現在実装されている管理機能はアンチパターンに該当することを再確認できました。


## 10/28

- [Fintechプロダクトの開発事情とアーキテクチャ解説 (スポンサーLT）](https://speakerdeck.com/yutadayo/kaigi-on-rails-2023-sponsor-lt)
- [Railsの型ファイル自動生成における課題と解決](https://speakerdeck.com/ksss/railsnoxing-huairuzi-dong-sheng-cheng-niokeruke-ti-tojie-jue)
- [事業の試行錯誤を支える、コードを捨てやすくしてシステムをシンプルに保つ設計と工夫](https://speakerdeck.com/zuckey_17/shi-ye-noshi-xing-cuo-wu-wozhi-eru-kodowoshe-teyasukusite-sisutemuwosinpurunibao-tushe-ji-togong-fu)
- [Fat Modelを解消するためのCQRSアーキテクチャ](https://speakerdeck.com/krpk1900/fat-modelwojie-xiao-surutamenocqrsakitekutiya)
- [返金処理を通して学ぶ、カード決済電文の沼バトル](https://speakerdeck.com/hirotea/numa-battle-of-card-transactions)
- [E2E testing on Rails 2023](https://speakerdeck.com/yusukeiwaki/kaigionrails2023pub)
- 
- テーブル分割で実現するdeviseの責務の分離と柔軟な削除機構
- Multiple Databasesを用いて2つのRailsプロジェクトを統合する
- [Hotwire的な設計を追求して「Web紙芝居」に行き着いた話](https://speakerdeck.com/nay3/hotwirede-nashe-ji-wozhui-qiu-site-webzhi-zhi-ju-nixing-kizhao-itahua)
- [管理機能アーキテクチャパターンの考察と実践](https://speakerdeck.com/ohbarye/learn-architecture-through-admin)
- [APMをちゃんと使おうとしたら、いつのまにか独自gemを作っていた話](https://slides.com/kokuyouwind/kaigi_on_rails_2023)
- [自分の道具を自作してつくる喜びを体感しよう、Railsで。 〜4年続いたPodcastを実例に〜](https://speakerdeck.com/mktakuya/kaigi-on-rails-2023)
- 
- 数十億のレコードを持つ5年目サービスの設計と障害解決
- Railsアプリにパスワードレス認証を導入した知見
- ActiveSupport::CurrentAttributes: すっごく便利なのに嫌われ者？！
- 
- Hotwireを使って管理画面を簡単にプチSPA化する
