---
title: "WEBMOCKとVCRの使い方"
emoji: "📼"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Ruby","Rspec","vcr","webmock","テスト"]
published: true
---

外部と通信するコードについてテストがほとんど書かれておらず、
存在しているテストは実際のURLに毎回アクセスする記述となっていました。

次の記事に詳しく書かれているのですが、
外部と通信する箇所こそテストが必要です。

https://qiita.com/jnchito/items/640f17e124ab263a54dd

一方で実際のコードでスタブを返却させるために
環境変数で値を切り替えたり、
RSPEC内で`allow`を使用して毎回外部と通信しないようにするのは
コードの可読性が低下してしまいます。

この問題を解決するためにWebMockとVCRを使用したため、
復習を兼ねてまとめておきます。

## サンプルコード

RubyでOpenAIのAPIを叩くことができるGem [ruby-openai](https://github.com/alexrudall/ruby-openai)を使用したサンプルコードを作成しました。
サンプルなのでコードの記述は適当です。
なお、Rubyのバージョンは3.2.2を使用しています。
また、同じレポジトリ内でRailsのアプリケーションが動作しているものとします。

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
    @client = OpenAI::Client.new(access_token:, log_errors: true)
  end

  def call
    @client.chat(parameters: {
      model: OPENAI_MODEL,
      temperature: RESPONSE_TEMPERATURE,
      max_tokens: MAX_TOKEN,
      messages:[{ role: "user", content: "こんにちは！"}],
    })
  rescue
    "エラー"
  end
end
```

## WebMock

https://github.com/bblimke/webmock

Webmockは、外部HTTPリクエストを実際には送らずにスタブとして模倣するためのGemです。

レスポンスをスタブ化することで、
テストの安定性と速度を向上させることができます。

### 導入例

実際のテストコードは次のようになります。
`stub_request`でリクエストをモックすることで、
テスト時に外部APIと通信しないようになります。
`to_return`でレスポンスの内容を指定しています。

```ruby

RSpec.describe Openai, type: :services do
  describe '.call' do
    it "有効な内容が返ってくる" do     
      stub_request(:post, 
        "https://api.openai.com/v1/chat/completions"
      ).to_return(status: 200, body: {"id"=>"chatcmpl-hoge",
        "object"=>"chat.completion",
        "created"=>1715411192,
        "model"=>"gpt-3.5-turbo-0125",
        "choices"=>
          [{"index"=>0,
            "message"=>{"role"=>"assistant", "content"=>"こんにちは！元気ですか？何かお手伝いできることがありますか？"},
            "logprobs"=>nil,
            "finish_reason"=>"stop"}],
        "usage"=>{"prompt_tokens"=>9, "completion_tokens"=>24, "total_tokens"=>33},
        "system_fingerprint"=>nil}.to_json , headers: {})
      response = Openai.call
      expect(JSON.parse(response).dig("choices", 0, "message", "content")).to eq "こんにちは！元気ですか？何かお手伝いできることがありますか？"
    end

    it "アクセスエラー" do
      stub_request(:post, 
        "https://api.openai.com/v1/chat/completions"
      ).to_raise(StandardError)
      response = Openai.call
      expect(response).to eq "エラー"
    end
  end
end

```

`with`を使用してリクエストとして送信する内容を指定することもできます。
ここでの内容は送信するパラメータだけではなく、
ヘッダーなども指定することができます。

また、to_returnの内容が多い場合は
次のようにファイルに記述して読み込むこともできます。

```ruby
  File.open('/tmp/response_body.txt', 'w') { |f| f.puts 'abc' }

  stub_request(:any, "www.example.com").
    to_return(body: File.new('/tmp/response_body.txt'), status: 200)

  Net::HTTP.get('www.example.com', '/')    # ===> "abc\n"
```

https://github.com/bblimke/webmock?tab=readme-ov-file#response-with-body-specified-as-io-object

また、system_specなどでlocalhostにアクセスする必要がある場合は
`spec/rails_helper.rb`に次の設定を追加します。

```ruby
require 'webmock/rspec'

config.before(:suite) do
  WebMock.disable_net_connect!(
    allow_localhost: true
  )
end
```

ローカルホスト以外に特定のホストへのアクセスを許可する場合は、
個別に設定を追加することができます。

このように、WebMockを使うことで、外部APIと通信せずにテストを実行することができます。
しかしながら、テスト項目に対応する全てのレスポンスを記述する必要があるため、
テスト内容が複雑になる可能性があります。

## VCR

https://github.com/vcr/vcr

VCRは、テストケースのHTTPリクエストの結果をYAMLやJSON形式で保存するGemです。
１回記録するとそれ以降のテスト実行時には保存した結果からレスポンスを読み込むので、外部との通信を行わないまま、正確な状態でテストを実行することができます。

### 導入例

Gemをinstallした後
`spec_helper.rb`に以下の設定を追加します。

OPENAIのAPIキーのような機密情報をリクエストに使用する場合、
`filter_sensitive_data`を設定するようにしてください。
そのままだと保存したレスポンス結果に機密情報が含まれてしまいます。

```ruby
require 'vcr'
require 'webmock/rspec'

VCR.configure do |c|
  c.cassette_library_dir = "spec/vcr" # 記録したカセットの保存先
  c.hook_into :webmock # VCRの内部で利用するモックライブラリ
  c.configure_rspec_metadata! # :vcrをつけたテストケースでにVCRカセットを使用する
  c.filter_sensitive_data("<OPENAI_ACCESS_TOKEN>") { ENV.fetch("OPENAI_ACCESS_TOKEN", nil) } 
end
```

マスクされた機密情報

![token](/images/masked_token.png)

VCRを適用したいテストケースに`:vcr`をつけることで、
VCRがHTTPリクエストを記録します。

```ruby
  describe '.call', :vcr do
    it "有効な内容が返ってくる" do       
      response = Openai.call
      expect(response.dig("choices", 0, "message", "content")).to eq "こんにちは！元気ですか？何かお手伝いできることがありますか？"
    end

    it "アクセスエラー" do   
```

この状態で実行すると、`spec/vcr`ディレクトリにレスポンスが保存されます。

![vcr_filename](/images/vcr_filename.png)

1回実行すると再度実行する際には、保存されたレスポンスを読み込んでテストを実行するため
高速でテストが終了します。
記録した内容を更新する際には削除して再度記録すれば良いです。

## まとめ

今回はWebMockとVCRを使用した通信する箇所のテスト作成を行いました。
内容が正確であることを確認するためには、
VCRを使用してカセットとして内容を記録することが有効だと思いましたが、
発生し得るエラーを意図的に発生させることが難しく、
記録できないことがあるため、WebMockを併用することで対応することも視野に入れておくと良いと思いました。

外部APIと通信する際に使用するケースを想定して記述しましたが、
内部APIやスクレイピングなどのコードでも使用することができるので様々な場面で活用できます。

## 使用する上で参考になりそうな記事

WebMock

- [WebMockでリクエストの値そのものも見たいときはwithに内容を指定する](https://shinkufencer.hateblo.jp/entry/2018/12/10/000000)
- [WebMockでパラメータによって成功時のリクエストと失敗時のレスポンスのモックを分ける](https://shinkufencer.hateblo.jp/entry/2018/12/11/233000)

VCR

- [公式ドキュメント](https://benoittgt.github.io/vcr/#/)
- [VCR 設定 Tips](https://tech.actindi.net/2021/06/21/083000)
  - リクエスト・レスポンスを見やすく整形する
- [VCRを使って開発中にあほみたいにリクエストを飛ばさないようにする](https://qiita.com/kazuooooo/items/30971a40511ea48da4a2)
  - カセットの更新条件を変更する
- [RSpecでWebMockとVCRを使ったテストを書く](https://shibaaa647.hatenablog.com/entry/2020/11/02/092936)
  - ローカルホストなどの通信をVCRの対象外にする
- [VCR で外部 API へのリクエストをダンプするときに機密情報をマスクしたい](https://qiita.com/gotchane/items/c2c29c0063bd44246510)
  - 機密情報のマスク

その他
[RSpecの技術的負債をチームで解消した話](https://www.lifull.blog/entry/2022/04/07/100000)
