---
title: "WebMockとVCR"
emoji: "🪪"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Ruby","RSpec","WebMock","VCR"]
published: true
---

# WebmockとVCRを使用したRSPECの書き方

1. 概要
2. WebMockとVCRの違い
3. WebMockの利用
4. VCRの利用
5. まとめ

WebサービスをRubyで開発する際、APIリクエストのテストが必要になります。しかし、外部サービスに直接リクエストを送るテストは、遅くなる可能性があり、外部サービスの動作に依存してしまいます。そこで、WebMockやVCRといったモックツールを使ってテストの高速化や安定化を図ります。本記事では、WebMockとVCRの違いを説明し、実際にGitHubAPIにアクセスするRubyクラスのRSpecを作成する方法を紹介します。

## WebMockとVCRの違い

WebMockとVCRはどちらもHTTPリクエストをモックするためのツールですが、アプローチが異なります。

**WebMock**はコードでモックを定義します。どのようなリクエストに対してどのようなレスポンスを返すかを細かく指定できます。しかし、すべてのリクエストに対するレスポンスをコード内に記述する必要があります。

**VCR**は実際のHTTPリクエストを記録し、その結果をキャシュします。後続のテストではキャッシュからレスポンスを読み込むので、実際にWebリクエストを発行する必要がありません。VCRを使えばリクエストとレスポンスを自動的に再現できますが、テストごとに実際のリクエストを発行し、レスポンスをキャッシュする必要があります。

どちらのツールを使うかはケースバイケースですが、VCRの方が設定が簡単です。WebMockは細かいコントロールが可能ですが、モックの定義が大変になる可能性があります。

## WebMockの利用

WebMockの基本的な使い方を説明します。

```ruby
# lib/github_client.rb
require 'net/http'

class GithubClient
  def initialize(username)
    @username = username
  end

  def repositories
    uri = URI("https://api.github.com/users/#{@username}/repos")
    response = Net::HTTP.get_response(uri)
    JSON.parse(response.body)
  end
end
```

上記のクラスはGitHubのユーザーリポジトリ一覧を取得するためのものです。このクラスをテストするには、WebMockを使ってGitHubAPIへのリクエストをモックします。

```ruby
# spec/github_client_spec.rb
require 'webmock/rspec'
require_relative '../lib/github_client'

RSpec.describe GithubClient do
  let(:username) { 'octocat' }
  let(:client) { GithubClient.new(username) }

  before do
    stub_request(:get, "https://api.github.com/users/#{username}/repos")
      .to_return(status: 200, body: fixture('repositories.json'), headers: {})
  end

  it 'fetches user repositories' do
    repositories = client.repositories
    expect(repositories.length).to eq(2)
    expect(repositories[0]['name']).to eq('Hello-World')
    expect(repositories[1]['name']).to eq('Hello-Rails')
  end

  def fixture(file)
    File.read(File.join('spec', 'fixtures', file))
  end
end
```

`stub_request`メソッドでGitHubAPIへのリクエストをモックしています。`to_return`でレスポンスのステータスコード、本文、ヘッダーを指定しています。本文にはfixtureから読み込んだJSONを使っています。

このようにWebMockを使えば、実際にGitHubAPIにアクセスすることなく、リポジトリ一覧の取得をテストできます。

## VCRの利用

次にVCRの使い方を説明します。

```ruby
# spec/github_client_spec.rb
require 'vcr'
require_relative '../lib/github_client'

VCR.configure do |config|
  config.cassette_library_dir = 'spec/fixtures/vcr_cassettes'
  config.hook_into :webmock
end

RSpec.describe GithubClient do
  let(:username) { 'octocat' }
  let(:client) { GithubClient.new(username) }

  it 'fetches user repositories', :vcr do
    repositories = client.repositories
    expect(repositories.length).to eq(2)
    expect(repositories[0]['name']).to eq('Hello-World')
    expect(repositories[1]['name']).to eq('Hello-Rails')
  end
end
```

VCRを使うには、まずVCR.configureでキャッシュファイルの保存先を指定する必要があります。`hook_into :webmock`を呼び出すことで、WebMockとVCRを連携させています。

実際のテストでは、`:vcr`メタデータをつけたテストケースでVCRがHTTPリクエストを記録します。最初のテスト実行時にGitHubAPIにリクエストを発行し、レスポンスをキャッシュします。後続のテストではキャッシュからレスポンスを読み込むので、実際のHTTPリクエストは発行されません。

## まとめ

本記事では、WebMockとVCRの違いを説明し、実際にGitHubAPIにアクセスするRubyクラスのRSpecを作成する方法を紹介しました。

WebMockはコードでモックを定義するので柔軟性が高いものの、設定が大変になる可能性があります。一方、VCRは実際のリクエストを記録するだけで、後続のテストではその記録を再生するだけなので、設定が簡単です。

どちらを使うかは、ユースケースによって判断する必要がありますが、VCRの方が導入が簡単なので、まずはVCRを試してみるのが良いでしょう。
