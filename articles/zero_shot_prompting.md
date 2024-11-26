---
title: "ゼロショット"
emoji: "🪟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["SQL","MySQL"]
published: false
---

# はじめに

[ジーズアカデミー 技術記事書いてみた編 Advent Calendar 2024](https://qiita.com/advent-calendar/2024/gsacademy)の２日目を担当します[philosophy_note](https://twitter.com/philosophy_note)です。2021年に東京LAB11期を卒業して現在は卒業生の方が起業した会社でエンジニアとして働いています。

昨年は[kano_miori](https://x.com/kano_miori)さんにギーハラを受けて執筆しましたが、
[kano_miori](https://x.com/kano_miori)さんが執筆された次の素晴らしい記事に感銘を受けて、
今年も執筆することにしました。

https://note.com/kano_miori/n/n91989fdcceca


## 本題

エンジニア界隈では生成AIを日常的に使っている方も多いと思います。
私もChatGPTやGithub Copilotを使うとともにどのように活用できるかを日々考えています。

そんな折、Forkwellさん主催の次のイベントに参加しました。

https://forkwell.connpass.com/event/330408/

イベントの中で次の内容が印象に残りました。

運の良いことに[コード×AIーソフトウェア開発者のための生成AI実践入門](https://gihyo.jp/book/2024/978-4-297-14484-5)を入手できたため。詳しく読みました。

本の中ではいくつかZero-shot Promptingの活用例が紹介されていましたが、実際ではより多くの活用例があると感じたので

## 結論

|技術|スタイルガイド|URL|
|---|---|---|
|Ruby|Bozhidar BatsovのRubyスタイルガイド|https://rubystyle.guide/|
|Rails|Railsガイド|https://railsguides.jp/|
|RSPEC|Effective Testing with RSpec 3|https://a.co/d/b7wSB06|
|RSPEC|Better Specs|https://www.betterspecs.org/|
|RSPEC|Everyday Rails Testing with RSpec|https://leanpub.com/everydayrailsrspec-jp|
|JavaScript|Airbnb JavaScriptスタイルガイド|https://github.com/airbnb/javascript|
|JavaScript|Standard JS|https://standardjs.com/|
|TypeScript|TypeScript Deep Dive 日本語版|https://typescript-jp.gitbook.io/deep-dive/|
|TypeScript|Google TypeScript Style Guide|https://google.github.io/styleguide/tsguide.html|
|React|bulletproof-react|https://github.com/alan2207/bulletproof-react|
|Python|PEP8|https://peps.python.org/pep-0008/|
|Python|Effective Python |https://www.oreilly.co.jp/books/9784873119175/|
|SQL|Simon HolywellのSQL Style Guide|https://www.sqlstyle.guide/|
|SQL|SQLアンチパターン|https://www.oreilly.co.jp/books/9784873115894/|
|AWS|AWS Well-Architected Framework|https://aws.amazon.com/jp/architecture/well-architected/?wa-lens-whitepapers.sort-by=item.additionalFields.sortDate&wa-lens-whitepapers.sort-order=desc&wa-guidance-whitepapers.sort-by=item.additionalFields.sortDate&wa-guidance-whitepapers.sort-order=desc|
|AWS|AWS Prescriptive Guidance|https://aws.amazon.com/jp/prescriptive-guidance/?apg-all-cards.sort-by=item.additionalFields.sortDate&apg-all-cards.sort-order=desc&awsf.apg-new-filter=*all&awsf.apg-content-type-filter=*all&awsf.apg-code-filter=*all&awsf.apg-category-filter=*all&awsf.apg-rtype-filter=*all&awsf.apg-isv-filter=*all&awsf.apg-product-filter=*all&awsf.apg-env-filter=*all|
|GCP|Google Cloud Security Foundations Guide|https://cloud.google.com/architecture/security-foundations?hl=ja|
|Azure|Microsoft Cloud Adoption Framework for Azure|https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/|
|Docker|Building best practices|https://docs.docker.com/build/building/best-practices/|
|プロンプトエンジニアリング|
DAIR.AIのPrompt Engineering Guide|https://www.promptingguide.ai/|
|プロンプトエンジニアリング|OpenAI公式のPrompt engineering|https://platform.openai.com/docs/guides/prompt-engineering|
|プロンプトエンジニアリング|MicrosoftのPrompt engineering techniques|https://learn.microsoft.com/ja-jp/azure/ai-services/openai/concepts/prompt-engineering|
|プロンプトエンジニアリング|Google公式のプロンプト設計戦略|https://ai.google.dev/gemini-api/docs/prompting-strategies?hl=ja|
|リファクタリング|ThoughtWorks社のRefactoring Catalog|https://martinfowler.com/books/refactoring.html|
|テスト|Vladimir KhorikovのUnit Testing(邦訳:[単体テストの考え方/使い方](https://amzn.asia/d/fVt3XB3))|https://www.oreilly.com/library/view/unit-testing-principles/9781617296277/|
|テスト|Kent BeckのTest-Driven Development by Example|https://www.oreilly.com/library/view/test-driven-development/0321146530/|
|全般|公式ドキュメント|存在するのであれば文言だけでも入力すべき|
|全般|Googleスタイルガイド|いろいろあります|

