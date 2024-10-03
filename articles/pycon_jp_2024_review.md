---
title: "【イベント参加レポート】PyCon JP 2024"
emoji: "⛎"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Python","LLM","カンファレンス","pyconjp"]
published: true
---

2024/9/27, 28 に行われた PyCon JP 2024 の感想です。
２日目のみの参加です。

## Pythonで日本語処理 入門〜フリガナプログラムを作ろう〜

https://x.com/takanory/status/1839835716190192009

### 感想

- 来月会社の勉強会で形態素解析について扱うので発表資料作成の参考になった
- [ruby](https://developer.mozilla.org/ja/docs/Web/HTML/Element/ruby)タグの存在を初めて知った
- これまでMecab + neologdを使っていたが、辞書を考慮するとSudachiの使用も検討してみたい

### 参考

- [sudachi.rs](https://github.com/WorksApplications/sudachi.rs)

## Pythonの数学機能を学ぼう！その仕組みも学ぼう！

https://x.com/curekoshimizu/status/1839848263433982228

### 感想

- 普段意識していなかったPythonの数値型やメモリの扱いについて興味を持てた
- JSでn+1==nとなるケースがあることを知らなかった
- fma関数の実装意図など質問が積極的にされており聴講者の興味のコントロールが上手だった

### 参考

- [math.fma](https://docs.python.org/ja/3.13/library/math.html#math.fma)

## Pythonを活用したLLMによる構造的データ生成の手法と実践

@[speakerdeck](5611881887e94ab19953b9eaf60daf7d)

### 感想

- LLMに回答を生成させるのではなく、コードを生成させてその結果を利用するというアプローチが面白かった
- dataclassやPydanticを活用したデータ構造の工夫が参考になった
- WASMの特性を活かしてセキュアな環境でPythonコードを実行するというアイデアが興味深かった
- フォローアップ記事も楽しみにしている

### 参考

- [PAL(プログラム支援言語モデル)](https://www.promptingguide.ai/jp/techniques/pal)
- [PAL: Program-aided Language Models](https://proceedings.mlr.press/v202/gao23f.html)
- [batteries included](https://docs.python.org/ja/3/tutorial/stdlib.html#batteries-included)
- [Wasmtime](https://wasmtime.dev/)

## SQLModel入門 〜クエリと型〜

https://x.com/mizzsugar0425/status/1839883727373013477

### 感想

- FastAPIでのDB連携を簡素化するために開発されたライブラリであるSQLModelについての発表
- PythonでDB連携を必要とした開発を普段していないので内容が頭に入りづらかったが、内容が非常に豊富で勉強になった
- 外部キー連携、特に`back_populates`の注意点が参考になった

### 参考

- [SQLModel](https://sqlmodel.tiangolo.com/)

## WEBアプリケーションにおけるAWS Lambdaを用いた大規模な非同期処理の実践

@[speakerdeck](f76a5d300ea7448ba4e16faaac89e9f4)

### 感想

- LambdaとSQSを組み合わせた非同期処理の実践についての発表
- 非同期処理は便利である一方、実運用する上で考慮するべき点が多いがライブラリを利用することで解決できることを学べた 

### 参考
- [amazon-sqs-python-extended-client-lib](https://github.com/awslabs/amazon-sqs-python-extended-client-lib)
- [Powertools for AWS Lambda (Python)](https://docs.powertools.aws.dev/lambda/python/latest/)
- [ReportBatchItemFailures](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/services-sqs-errorhandling.html)

## プロダクションでのPython非同期ユースケース - Trio/Trio-Utilを中心に

https://x.com/JunyaFff/status/1840217644403962259

### 感想

- Pythonで非同期処理を実装する方法や注意点を学べた。
- 構造化された並行性という概念を初めて知れた。
- スライドのこだわりが強かった。LOVOT可愛い。

### 参考

- [trio](https://github.com/python-trio/trio)
- [asyncio](https://docs.python.org/ja/3/library/asyncio.html)
- [構造化された並行性](https://trio.readthedocs.io/en/stable/reference-core.html#structured-concurrency)

## まとめ

２日目だけの参加でしたが
プログラミングを本格的に開始した際に最初に触れた言語である
Pythonのカンファレンスに参加できて楽しかったです。

企業のブースも多く出展されており、
企業ごとに工夫されていて勉強になりました。
