---
title: "【イベント参加レポート】Kaigi on Rails 2024"
emoji: "🛤️"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Rails","kaigionrails","Ruby"]
published: true
---

2024/10/26, 27 に行われた[Kaigi on Rails](https://kaigionrails.org/2024/)の参加レポートになります。

![kaigi_on_rails_2024](/images/kaigi_on_rails_2024.jpeg)

# 10/26

## 基調講演：[Rails Way, or the highway](https://speakerdeck.com/palkan/kaigi-on-rails-2024-rails-way-or-the-highway)

@[speakerdeck](ce11ed892bda4717b96f12d064865a4a)

### 感想

- Rails WayはWebアプリケーションを構築する方法論に止まらず、**哲学**であり、**導きの星**である
- Railsを拡張するためにはRailsを習得し、Railsの抽象化に沿って拡張することが重要
- OSI参照モデルとTCP/IPモデルの対応図のようなLayed ArchitectureとRailsの抽象化の対応図が印象に残りました
- 抽象化は一朝一夕で身につくものではないと思うので、Rails Wayという哲学を頭に置きながらRailsを学び続けたい


### 参考
- [Layered Design for Ruby on Rails Applications](https://a.co/d/fkgONGl)

## [Railsの仕組みを理解してモデルを上手に育てる - モデルを見つける、モデルを分割する良いタイミング -](https://speakerdeck.com/igaiga/kaigionrails2024/)

@[speakerdeck](eea7a4e755784fa690362a48557c1802)

### 感想

- POROのユースケースやルール作りの話が勉強になりました
  - 「クラス名には返すオブジェクトの名前を名詞でつける」
- 複数のテーブルにまたがる複雑な処理の場合に安易にServiceクラスを作っていたが対応を考え直したいと思います
- Serviceクラスが何故望ましくないかを基調公演の内容も踏まえて理屈で理解することができました
- POROを使用してRails Wayに沿った設計を意識したり、新規にテーブルを追加するケースではイベント型モデルを探しだすようにしたい

### 参考

- [Railsの練習帳](https://zenn.dev/igaiga/books/rails-practice-note)
- [texta.fm](https://open.spotify.com/show/2BdZHve9cIU6c8OFyz7LeB)

## [Sidekiqで実現する長時間非同期処理の中断と再開](https://speakerdeck.com/hypermkt/pausing-and-resuming-long-running-asynchronous-jobs-with-sidekiq)

@[speakerdeck](519ed48333bb467f99800676350f6fc3)

### 感想

- 処理が長い非同期ジョブを実装したことがないため、中断や再会処理を意識したことはないが実装方法が勉強になりました
- Sidekiq Iterationを初めて知りました
- Sidekiqは複雑だが、機能が充実しているため、再度触る際にはドキュメントを改めて熟読したいと思います

## [リリース8年目のサービスの1800個のERBファイルをViewComponentに移行した方法とその結果](https://speakerdeck.com/katty0324/ririsu8nian-mu-nosabisuno1800ge-noerbhuairuwoviewcomponentniyi-xing-sitafang-fa-tosonojie-guo)

@[speakerdeck](d4b29cf4bcf94ee9887fced33e569cdf)

### 感想

- スクリプトを組んでViewComponentを自動生成する考え方が勉強になりました
- フィーチャーフラグやカナリアリリースを活用して段階的にリリースして安全性を確保している点も勉強になりました

## [ActionCableなら簡単? 生成 AIの応答をタイピングアニメーションで表示。実装、コスト削減、テスト、運用まで。](https://docs.google.com/presentation/d/1sPCFlWPKmnTcc11Nt99swIJ-M056JtdK2tqHYZ18QEs/edit?usp=sharing)

https://x.com/kaiba/status/1849729753865125982

### 感想

- 困ったらRailsガイドを読む
- VCRで記録したレスポンスを修正する考えはなかったのでユースケースとして覚えておきたい
- ローカルでLLmを実行できるLM Studioを初めて知った

### 参考

- [LM Studio](https://lmstudio.ai/)

## [デプロイを任されたので、教わった通りにデプロイしたら障害になった件 〜俺のやらかしを越えてゆけ〜](https://speakerdeck.com/techouse/depuroiworen-saretanode-jiao-watutatong-rinidepuroisitarazhang-hai-ninatutajian-an-noyarakasiwoyue-eteyuke)

@[speakerdeck](84105fdf2dfc4ba585e148378241a1e8)

### 感想

- 障害が発生した際の対応とその後の原因究明の話を明確にわかりやすく言語化できている点が素晴らしかったです
- アプリケーションの実装に意識が向きすぎていてECSの設定や仕組みの理解が疎かになりがちなので注意したい

## [Capybara+生成AIでどこまで本当に自然言語のテストを書けるか？](https://speakerdeck.com/yusukeiwaki/capybara-plus-sheng-cheng-aidedokomadeben-dang-nizi-ran-yan-yu-notesutowoshu-keruka)

@[speakerdeck](23b11c66d8924b49a64f0748552a9f4f)

### 感想

- 仕様書やドキュメントを元にE2Eテストを自動化する考えはLLMが流行り始めたときから抱いていたので興味がありました
- LLM関連のmeetupでスクリーンショットを元にブラウザを自動操作させる話を聞いたことがあり、その延長線上で理解できました
- 挙動のランダム要素がまだあるため実際に使えるレベルにはないが、近い将来に実用化される可能性があると感じました

# 10/27

## [ActiveRecord SQLインジェクションクイズ (Rails 7.1.3.4)](https://speakerdeck.com/kozy4324/activerecord-sqlinziekusiyonkuizu-rails-7-dot-1-3-dot-4)

@[speakerdeck](923f3b90563742f9b2b6ec0f63b9c5d6)

### 感想

- ActiveRecordでDB操作を行う際にどのケースがSQLインジェクションになるか曖昧に覚えていたので知識を再確認できました
- Brakemanを初めて知ったので導入方法や使い方を調べてみたいと思います

### 参考

- [ActiveRecord SQLインジェクションクイズ (Rails 7.1.3.4)](https://zenn.dev/kozy4324/articles/c22d7bc7e39417)
- [Brakeman](https://brakemanscanner.org/)

## [OmniAuthから学ぶOAuth2.0](https://speakerdeck.com/ykpythemind/omniauthkaraxue-buoauth2-dot-0-kaigi-on-rails-2024)

@[speakerdeck](83e1754b5a184bafa4948032039c6638)

### 感想

- 外部認証を実装する際に使用するOmniAuthの仕組みについての発表
- コードを読みながら自前で実装するプロセスをを通して外部認証の裏側の仕組みを大まかに理解できました

## [約9000個の自動テストの時間を50分から10分に短縮、偽陽性率(Flakyテスト)を1%以下に抑えるまでの道のり](https://speakerdeck.com/hatsu38/yue-9000ge-nozi-dong-tesutono-shi-jian-wo50fen-10fen-niduan-suo-flakytesutowo1-percent-yi-xia-niyi-etahua)

@[speakerdeck](d008fd301eff4a54b91fecabc818e684)

### 感想

- 自動テストの実行時間の長さが開発者体験を低下させていたので短縮する方法を知りたかったので勉強になりました
  - 会場のアンケートでも自動テストの実行時間が20分以上という声が多かったです
- まずはtest-profを導入して`before_all`や`let_it_be`がどの程度使用できるかを検証してみたいです
- Allure Report及びVercelにhtmlをデプロイしてテスト結果をレポートする方法も勉強になりました

### 参考

- [test-prof](https://github.com/test-prof/test-prof)
- [Allure Report](https://allurereport.org/)

## [Sidekiq vs Solid Queue](https://speakerdeck.com/willnet/sidekiq-vs-solid-queue)

@[speakerdeck](e9b495790aa54b4d9f9629c7f6b87949)

### 感想

- Rails8.0からデフォルトになるSolid Queueの話
- SidekiqはActive Jobを経由しないで使用した方が良いという説の事情やSolid Queueが導入された経緯について歴史的な背景を知ることができ勉強になりました

## [The One Person Framework 実践編](https://speakerdeck.com/asonas/practical-the-one-person-framework)

@[speakerdeck](b1d8ade237174e04aceadd25a240627a)

### 感想

- The One Person Frameworkという概念を初めて知りました
- ブログ記事に記述はありましたが、「一人分の脳に収まる程度の情報量でアプリを記述できるように」という表現がわかりやすかったです
- 「居住可能性」という表現も興味深かった
  - 心地よく自信を持って変更を加えられる
- それを更にチームの枠組み拡張するアプローチが興味深かった
- The One Person FrameworkもRailsの哲学としてアプリケーション開発及びマネジメントに活かしていきたい

### 参考

- [The One Person Framework](https://world.hey.com/dhh/the-one-person-framework-711e6318)
- [2024年のRailsと自由について考える](https://speakerdeck.com/takahashim/enishitechcconf2024)
- [認知負荷の種類と対策と組織文化について](https://sugamasao.hatenablog.com/entry/2023/12/03/160411)
- [えにしテック、あるいは人間関係のエクササイズ / #enishitech-15th-anniv](https://speakerdeck.com/kakutani/number-enishitech-15th-anniv)

## [Data Migration on Rails](https://speakerdeck.com/ohbarye/data-migration-on-rails)

@[speakerdeck](dedb021b0f6641b5b0b7d1194cca4a31)

### 感想

- データマイグレーションの運用を見直す良い機会になりました
- メリット・デメリットを整理やサーベイの確認、各アプローチの評価項目を整理して検討する方法が横展開できるので参考になりました

### 参考
- [maintenance_tasks](https://github.com/Shopify/maintenance_tasks)

## omakaseしないためのrubocop.yml のつくりかた

https://x.com/expajp/status/1850071250003198135

### 感想

- Rubocopのルールを決める際にチーム内でコンセンサスを取るまでの取り組み方についての発表でした
- 定期的なMTGと合わせて実施することで取り組みやすくする工夫やテンプレートを作成して議論を進めやすくする方法が参考になりました

### 参考

- [rubocop-rails-omakase](https://github.com/rails/rubocop-rails-omakase)

## [Identifying User Identity](https://speakerdeck.com/moro/identifying-user-idenity)

@[speakerdeck](b9b2650a1f6945389cea7553b92989ee)

### 感想

- 登壇者の方の前の発表や前日の[Railsの仕組みを理解してモデルを上手に育てる - モデルを見つける、モデルを分割する良いタイミング -](https://speakerdeck.com/igaiga/kaigionrails2024/)と内容が繋がる箇所があり、Railsのモデル設計についての理解が深まりました
- Userのような普遍的なモデルは様々な箇所で参照されるとともに役割を多く持つため、Fatになりやすいですが今回の発表の考え方が勉強になりました
- 最後のスライドで`status`列を参照することなく状態を表現する方法が興味深かったです。DBで参照するたびにテーブル内の状態を確認するのではなく、関連テーブルとの結合で状態を取得する方法が大変勉強になりました

### 参考

- [Simplicity on Rails -- RDB, REST and Ruby](https://speakerdeck.com/moro/simplicity-on-rails-rdb-rest-and-ruby)

## 基調講演：[WHOLENESS, REPAIRING, AND TO HAVE FUN: 全体性、修復、そして楽しむこと](https://speakerdeck.com/snoozer05/wholeness-repairing-and-to-have-fun)

@[speakerdeck](32baeb88f27846e6a571594e44c5f7b3)

### 感想

- 設計する際に選択する権利を持つことが大切であり、Railsは権利をサポートしてくれるフレームワークである
- 修復が必要な箇所を見つけるためには、システムを複雑にせず認知負荷を減らすことが大切
- 「楽しさとはビジネス価値がある」フレーズを念頭に置いて設計と修復を続けていこうと思いました


### 参考

- [The Rails Doctrine.](https://rubyonrails.org/doctrine)

## 全体を通して

Rails Wayはただの開発手法ではなく、哲学であることを実感しました。
1日目の基調公演が理想的な基調公演だったことを懇親会などで話しましたが、
2日目の発表でもこの基調公演の内容に早速触れられている方が何名かおり、
最後の基調講演でも再度触れられていたので、
Rails WayがRailsの開発者にとって重要なものであることを改めて認識しました。
