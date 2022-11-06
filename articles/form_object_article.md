---
title: "【備忘録】Formオブジェクトについて"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rails"]
published: true
---

仕事で孫モデルまでを一回の処理で作成・更新する必要があり、
汚いコードでなんとか記述しました。

今後はなるべく簡潔に記述したいと思い、Formオブジェクトについてまとめることにしました。

## Form Objectとは何か？

form_withを使用することで目的が達成されるユースケースを実装するために用いるものです。

## 何故使用するのか？

Railsではresourcesメソッドによるリソースベースのルーティングが基本になっています。
URLで表されるリソースをデータベースのテーブルと１対１に対応させており、これらのCRUD操作を通してユーザーとやり取りすることを想定しています。

しかしながら、次のようなケースではこの考え方だけでは上手くいかないケースに該当します。

1. CRUD操作が複数のユースケースで行われる場合
 例えば、ユーザー登録を行う場合はフォームから登録する場合もあれば、CSVから登録される場合もあります。
 前者の場合は利用規約に同意するチェックボックスの値を確認する必要がありますが、後者の場合は確認する必要がないような場合だとこれに対応する形でモデルの肥大化やコールバックの複雑化といった問題が発生します。

2. 複数のモデルで登録・更新操作が必要になる場合
 親モデルに操作が行われた場合に子モデルの操作が必要になる場合を考えます。
 この場合`accepts_nested_attributes_for`メソッドが候補になります。
 しかしながらこのメソッドは非常に評判が悪く、使用を禁止している場合もあるとのことです。

 <https://zenn.dev/murakamiiii/articles/5ecefb7a58d1ef>

 <https://moneyforward.com/engineers_blog/2018/12/15/formobject/>
  
3. 対応するテーブルが存在しない場合
 例えば、Cookieベースのセッションの作成・削除操作のようなユースケースでは対応するモデルが存在しません。他のモデルやコントローラーに処理を実装することもできますが、記述が肥大化してしまいます。他にもElasticSearchへのリクエストなどのケースが想定されます。

もう少し抽象的に表現すると次の記事のように表現されます。

<https://applis.io/posts/rails-design-pattern-form-objects#form%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AE%E5%BF%85%E8%A6%81%E6%80%A7>

> ユーザの入力の整形や永続化をコントローラだけで行うと、コントローラが肥大化してしまいます。 この原因はコントローラがモデル層の知識をもちすぎるためにあります。 このときビューもフォームを表示するための知識をもつことになるため、コントローラと同じような問題が起こってしまいます。 このことは単一責任の原則に反し、**モデル層の変更がコントローラやビューに影響を及ぼす**ことになります。
逆にActiveRecordモデルにこういった責務をもたせると、今度はActiveRecordモデルがフォームの知識を持ちすぎてしまいます。 フォームという独立した責務があるのであれば、これをひとつのクラスにカプセル化する、というのがFormオブジェクトの役割です。

## 使用例

孫要素までまとめて保存する例を考えます。
サービスが複数存在し、
ユーザーはサービスの権限（読み書き）を設定するルールを複数所有するケースを考えます。

テーブルとER図は次の通りです。

| Table | Column | Type |
| ---- | ---- | ---- |
| rules | id | interger |
|  | rule_name  | varchar |
| authorities | id | interger |
|  | rule_id  | interger |
|  | authority  | interger |
| authority_service_relations | id | interger |
|  | authority_id   | interger |
|  | service_id   | interger |
| services | id | interger |
|  | service_name   | varchar |

![](https://storage.googleapis.com/zenn-user-upload/5bb765de8b26-20221106.png)

この例はRuby 2.7.5,Ruby on Rails 6.1.6で動作を確認しています。

### Formオブジェクト

```ruby:app/forms/rule_form.rb
class RuleForm
 include ActiveModel::Model
 include ActiveModel::Attributes
 
 attribute :rule_name, :string
 attribute :read
 attribute :write
 
 validates :rule_name, presence: true, length: { in: 1..30 }
 
 delegate :persisted?, to: :rule
 
 def initialize(attributes = nil, rule: Rule.new)
  @rule = rule
  attributes ||= default_attributes
  super(attributes)
 end
 
 def save
  return if invalid?

  ActiveRecord::Base.transaction do  

    rule.authorities.destroy_all
    if read[:services]&.any?
      read_authority = rule.authorities.build(authority: Authority.authorities[:read] )
      read[:services].compact_blank!
      read[:services].each do |read_service|
        read_authority.authority_service_relations.build(service_id: read_service)
      end
    end
    
    if write[:services]&.any?
      write_authority = rule.authorities.build(authority: Authority.authorities[:write])
      write[:services].compact_blank!
      write[:services].each do |write_service|
        write_authority.authority_service_relations.build(service_id: write_service)
      end
    end
    rule.update!(rule_name: rule_name)
  end

  rescue ActiveRecord::RecordInvalid
    false
 end
 
 def to_model
  rule
 end
 
 private

    attr_reader :rule
    
    def default_attributes
      {
        rule_name: rule.rule_name,
        read: { services: rule.authorities.read.joins(:services) },
        write: { services: rule.authorities.write.joins(:services) },
      }
    end
end  
  
```

コードの説明です。
`include Active::Model`はActiveModelが提供するモジュール郡の一部をまとめたモジュールです。
複数のモジュールを組み合わせて、コントローラやビューのメソッドとの連携に必要なインターフェースを提供します。

<https://railsguides.jp/active_model_basics.html#model%E3%83%A2%E3%82%B8%E3%83%A5%E3%83%BC%E3%83%AB>

`include Active::Attributes`は属性を定義するためのattributesメソッドを利用できるようになるモジュールです。attributesメソッドでは、第1引数で属性名を、第2引数で型名を指定します。

<https://railsguides.jp/active_model_basics.html#attributemethods%E3%83%A2%E3%82%B8%E3%83%A5%E3%83%BC%E3%83%AB>

なお、今回のコードはRails6系で確認しております。
Rails7系では`ActiveModel::Attributes`についても一緒にincludeされるようになりました。

<https://techracho.bpsinc.jp/hachi8833/2022_01_28/114954>

`delegated`は委譲に関するメソッドです。
ruleモデルから`persisted?`を委譲することでビューからform_withで送信する際に
自動的にフォームのアクションを`POST`と`PATCH`に切り替えてくれます。

`initialize`はFormオブジェクトの値を初期化しています。
こちらについては先ほども引用した次の記事が勉強になりました。

[Railsのデザインパターン: Formオブジェクト](https://applis.io/posts/rails-design-pattern-form-objects#formオブジェクト)

`to_model`はビューの表示（`form_with`）に必要なメソッドです。 アクションのURLを適切な場所（ここではrule_pathやrule_path(id)）に切り替えてくれます。


### コントローラ

```ruby:app/controllers/rules_controller.rb
class RulesController < ApplicationController
  def index
    @rules = Rule.all
  end

  def new
    @form = RuleForm.new
  end

  def create
    @form = RuleForm.new(rule_params)

    if @form.save
      redirect_to rules_path
    else
      render :new
    end
  end

  def edit
    load_rule

    @form = RuleForm.new(rule: @rule)
  end

  def update
    load_rule
    @form = RuleForm.new(rule_params, rule: @rule)

    if @form.save
      redirect_to rules_path
    else
      render :edit
    end
  end

  private

  def rule_params
    params.require(:rule).permit(:rule_name, 
      read: {
        services: []
        },
      write: {
        services: []
        }
      )
  end

  def load_rule
    @rule = Rule.find(params[:id])
  end
end

```

### ビュー

```ruby:app/views/rules/new.html.erb
<%= form_with model: @form, local: true do |form| %>
  <%= form.text_field :rule_name %>
  <%= form.fields_for :read do |read| %>
    <%= read.collection_check_boxes :services, Service.all, :id, :service_name %>
  <% end %>
  <%= form.fields_for :write do |write| %>
    <%= write.collection_check_boxes :services, Service.all, :id, :service_name %>
  <% end %>
  <%= form.submit %>
<% end %>
```

```ruby:app/views/rules/edit.html.erb
<%= form_with model: @form, local: true do |form| %>
  <%= form.text_field :rule_name %>
  <%= form.fields_for :read do |read| %>
    <%= read.collection_check_boxes :services, Service.all, :id, :service_name,  checked: @form.read[:services].pluck(:service_id) %>
  <% end %>
  <%= form.fields_for :write do |write| %>
    <%= write.collection_check_boxes :services, Service.all, :id, :service_name,  checked: @form.write[:services].pluck(:service_id) %>
  <% end %>
  <%= form.submit %>
<% end %>
```

### モデル

```ruby:app/models/rule.erb
class Rule < ApplicationRecord
  has_many :authorities

  validates :rule_name, presence: true, length: { in: 1..30 }
end
```

```ruby:app/models/authority.erb
class Authority < ApplicationRecord
  belongs_to :rule
  has_many :authority_service_relations
  has_many :services, through: :authority_service_relations
  enum authority: { read: 1, write: 2 }
end
```

```ruby:app/models/service.erb
class Service < ApplicationRecord
  has_many :authorities, through: :authority_service_relation
end
```

```ruby:app/models/authority_service_relation.erb
class AuthorityServiceRelation < ApplicationRecord
  belongs_to :authority
  belongs_to :service
end
```

## 感想

Formオブジェクトを作成するとフォームに関する記述が一箇所にまとまるので
非常に読みやすいです。
自分で作成した時は一番上のモデル（親モデル）に孫モデルの保存処理まで記述してしまい、
読みにくくなったので大変ありがたいです。


また、`POST`と`PATCH`の切り替えなどは普段はあまり意識しておりませんでしたが、
自力で実装する体験を通して理解を深めることができました。

一方でRailsに頼るあまり、純粋なRubyのクラスの扱い方に苦戦しました。
今後はそちらについても理解を深めていきたいと思います。
## 参考文献・記事

[パーフェクト Ruby on Rails【増補改訂版】](https://www.amazon.co.jp/%E3%83%91%E3%83%BC%E3%83%95%E3%82%A7%E3%82%AF%E3%83%88-Ruby-Rails-%E3%80%90%E5%A2%97%E8%A3%9C%E6%94%B9%E8%A8%82%E7%89%88%E3%80%91-%E3%81%99%E3%81%8C%E3%82%8F%E3%82%89-%E3%81%BE%E3%81%95%E3%81%AE%E3%82%8A-ebook/dp/B08D3DW7LP/ref=sr_1_1?crid=1GVVRQ00LI7MO&keywords=%E3%83%91%E3%83%BC%E3%83%95%E3%82%A7%E3%82%AF%E3%83%88ruby+on+rails+%E5%A2%97%E8%A3%9C%E6%94%B9%E8%A8%82%E7%89%88&qid=1667008413&qu=eyJxc2MiOiIwLjkwIiwicXNhIjoiMS4wMCIsInFzcCI6IjEuMDAifQ%3D%3D&sprefix=%E3%83%91%E3%83%BC%E3%83%95%E3%82%A7%E3%82%AF%E3%83%88+Ruby%2Caps%2C212&sr=8-1)

[Railsのデザインパターン: Formオブジェクト](https://applis.io/posts/rails-design-pattern-form-objects)
