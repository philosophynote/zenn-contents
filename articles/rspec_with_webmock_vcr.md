---
title: "WEBMOCKとVCRの使い方"
emoji: "📼"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Ruby","Rspec","vcr"]
published: false
---

## WEBMOCKとVCRの使い方

外部APIを呼び出す処理がある場合、
テストを実行する度に外部と通信するのは避けたいところです。
Rubyで開発する際にWebMockやVCRを使うことで、
テスト時に外部APIと通信せず、
事前に取得したレスポンスを再利用することができます。
本記事では、WebMockとVCRを説明し、実際にOpenAIにアクセスするRubyクラスのRSpecを作成する方法を紹介します。

### Webmock

https://github.com/bblimke/webmock

### VCR

https://github.com/vcr/vcr

## 「モック」と「スタブ」

内容に入る前に、まず「モック」と「スタブ」の違いについて説明します。

:::message
モック：テスト対象システムからその依存に向かって行われる外部に向かうコミュニケーション（出力）を模倣し、そして、検証するのに使われる。

スタブ：依存からテスト対象システムに向かって行われる内部に向かうコミュニケーション（入力）を模倣するのに使われる。
:::

## サンプルコード

RubyでOpenAIのAPIを叩くことができるGem [ruby-openai](https://github.com/alexrudall/ruby-openai)を使用したサンプルコードを作成します。
Rubyのバージョンは3.2.2を使用しています。

```ruby
class Openai
  OPENAI_MODEL = "gpt-3.5-turbo"
  RESPONSE_TEMPERATURE = 0.1
  MAX_TOKEN = 1024

  def self.call
    access_token = ENV.fetch("OPENAI_ACCESS_TOKEN", nil)
    new(access_token).call
  end

  def initialize(access_token)
    @client = OpenAI::Client.new(access_token:)
  end

  def call
    @client.chat(parameters: {
      model: OPENAI_MODEL,
      temperature: RESPONSE_TEMPERATURE,
      max_tokens: MAX_TOKEN,
      messages:[{ role: "user", content: "こんにちは！"}],
    })
  rescue => e
    "エラーです" if e.response[:status] == 500
  end
end
```

## WebMock

Webmockは、テスト対象のコードが外部のWebサービスへ向けて送られるHTTPリクエストを実際には送らずにスタブとして模倣します。

外部のWebサービスに実際にリクエストを送る代わりに、レスポンスをスタブ化することで、
テストの安定性と速度を向上させることができます。

しかし、すべてのリクエストに対するレスポンスをコード内に記述する必要があります。

### 導入例

Gemをinstallした後
`spec/rails_helper.rb`に以下の設定を追加します。

```ruby
require 'webmock/rspec'

```

system_specなどでlocalhostにアクセスする場合、
`spec/rails_helper.rb`に以下の設定を追加します。
ローカルホスト以外に特定のホストへのアクセスを許可する場合は、
個別に設定を追加することができます。

```ruby
WebMock.disable_net_connect!(allow_localhost: true)
WebMock.disable_net_connect!(allow: 'www.example.com')
```

stub_requestでリクエストをモックすることで、
テスト時に外部APIと通信しないようになります。
withでリクエストの条件を指定し、to_returnでレスポンスを指定します。

```ruby

RSpec.describe Openai, type: :services do
  describe '.call' do
    it "有効な内容が返ってくる" do
      stub_request(:post, "https://api.openai.com/v1/chat/completions").to_return(status: 200, body: "こんにちは！")        
      response = Openai.call
      expect(response).to eq "こんにちは！"
    end

    it "アクセスエラー" do
      stub_request(:post, "https://api.openai.com/v1/chat/completions").to_return(status: 500, body: "エラーです")        
      response = Openai.call
      expect(response).to eq "エラーです"
    end
  end
end

```

このように、WebMockを使うことで、外部APIと通信せずにテストを実行することができます。
しかしながら、テスト項目に対応する全てのレスポンスを記述する必要があるため、テスト内容が複雑になる可能性があります。

## VCR

VCRは、実際のHTTPリクエストを記録し、その結果をキャッシュするGemです。
後続のテストではキャッシュからレスポンスを読み込むので、実際にWebリクエストを発行する必要がありません。VCRを使えばリクエストとレスポンスを自動的に再現できます。。

OpenAIのAPIキーのような機密情報を含むリクエストをキャッシュする場合、VCRのconfigに設定を記載することで、
マスクしたい文字列にマスクをかけることができます。

ただし上手くマスクできない場合もあるので、次の記事を参考にしてください。

https://qiita.com/gotchane/items/c2c29c0063bd44246510

### 導入例

Gemをinstallした後
`spec_helper.rb`に以下の設定を追加します。

```ruby
require 'vcr'
require 'webmock/rspec'

VCR.configure do |c|
  c.cassette_library_dir = "spec/vcr" # キャッシュの保存先
  c.hook_into :webmock # VCRの内部で利用するモックライブラリ
  c.configure_rspec_metadata! # :vcrをつけたテストケースでにVCRカセットを使用する
  c.filter_sensitive_data("<OPENAI_ACCESS_TOKEN>") { ENV.fetch("OPENAI_ACCESS_TOKEN", nil) } #機密情報をマスクする 
end
```

基本的な設定は以上です。
細かい設定は公式ドキュメントを読むか使用目的に合わせて次のリンク先で確認してください。

### リクエスト・レスポンスを見やすく整形する

https://tech.actindi.net/2021/06/21/083000

### カセットの更新条件を変更する

https://qiita.com/kazuooooo/items/30971a40511ea48da4a2

### ローカルホストなどの通信をVCRの対象外にする

https://shibaaa647.hatenablog.com/entry/2020/11/02/092936
