---
title: "【備忘録】フォームオブジェクトについて"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

仕事で孫モデルまでを一回の処理で作成・更新する必要があり、
汚いコードでなんとか記述しました。

今後はなるべくスマートに記述したいと思い、Form Objectについてまとめることにしました。

## Form Objectとは何か？

form_withを使用することで目的が達成されるユースケースを実装するために用いるものです。

## 何故使用するのか？

Railsではresourcesメソッドによるリソースベースのルーティングが基本になっています。
URLで表されるリソースをデータベースのテーブルと一対一に対応させており、これらのCRUD操作を通してユーザーとやり取りすることを想定しています。

しかしながら、次のようなケースではこの考え方だけでは上手くいかないケースに該当します。

1. CRUD操作が複数のユースケースで行われる場合
 例えば、ユーザー登録を行う場合はフォームから登録する場合もあれば、CSVから登録される場合もあります。
 前者の場合は利用規約に同意するチェックボックスの値を確認する必要がありますが、後者の場合は確認する必要がないような場合だとこれに対応する形でモデルの肥大化やコールバックの複雑化といった問題が発生します。

2. 複数のモデルで登録・更新操作が必要になる場合
 親モデルに操作が行われた場合に子モデルの操作が必要になる場合を考えます。
 この場合`accepts_nested_attributes_for`メソッドが候補になります。
 しかしながらこのメソッドは非常に評判が悪く、使用を禁止している会社もあるとのことです。

 <https://zenn.dev/murakamiiii/articles/5ecefb7a58d1ef>

 <https://moneyforward.com/engineers_blog/2018/12/15/formobject/>
  
3. 対応するテーブルが存在しない場合
　　例えば、Cookieベースのセッションの作成・削除操作のようなユースケースでは対応するモデルが存在しません。他のモデルやコントローラーに処理を実装することもできますが、記述が肥大化してしまいます。
 他にもElasticSearchへのリクエストなどのケースが想定されます。

もう少し抽象的に知りたい場合は次の記事が参考になりました。

<https://applis.io/posts/rails-design-pattern-form-objects#form%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AE%E5%BF%85%E8%A6%81%E6%80%A7>

> ユーザの入力の整形や永続化をコントローラだけで行うと、コントローラが肥大化してしまいます。 この原因はコントローラがモデル層の知識をもちすぎるためにあります。 このときビューもフォームを表示するための知識をもつことになるため、コントローラと同じような問題が起こってしまいます。 このことは単一責任の原則に反し、**モデル層の変更がコントローラやビューに影響を及ぼす**ことになります。
逆にActiveRecordモデルにこういった責務をもたせると、今度はActiveRecordモデルがフォームの知識を持ちすぎてしまいます。 フォームという独立した責務があるのであれば、これをひとつのクラスにカプセル化する、というのがFormオブジェクトの役割です。

## 使用例

サービスが複数存在し、ユーザーはサービスの利用の有無を設定するルールを複数所有するケースを考えます。

テーブルとER図は次の通りです。

| Table | Column | Type |
| ---- | ---- | ---- |
| rules | id | interger |
|  | rule_name  | varchar |
| authorities | id | interger |
|  | authority  | interger |
| services | id | interger |
|  | service_name   | varchar |
| policy_service_relations | id | interger |
|  | policy_id   | interger |
|  | service_id   | interger |

![](https://storage.googleapis.com/zenn-user-upload/c23fbc0f948a-20221029.png)

### Formオブジェクト

```ruby:app/forms/rule_management_form.rb
class RuleManagementForm
 include Active::Model
 include Active::Attributes
 
 attribute :rule_name, :string
 attribute :policy_name, :string
 
 validates :rule_name, presence: true, length: { in: 1..30 }
 validates :policy_name, presence: true, length: { maximum: 50 }
 
 delegate :persisted?, to: :rule
 @form = RuleManagementForm.new
 # @from.rule.persisted?が@from.persisted?で書けるようになる
 delegate :persisted?, to: :policy
 
 def initialize(attributes = nil, rule: Rule.new, policy: Policy.new)
  @rule = rule
  @policy = policy
  attributes ||= default_attributes
  super(attributes)
 end
 
 def save
  return if invalid?

  ActiveRecord::Base.transaction do
      survice_ids = 
      user.update!(user_name: user_name, rule_name: rule_name, survice_ids: survice_ids)
    end
  rescue ActiveRecord::RecordInvalid
    false
 end
 
 def to_model
  user
 end
 
 private
  attr_reader :policy
  
  def default_attributes
    {
      rule_name: rule.rule_name,
      policy_name: policy.policy_name,
      policy_level: policy.policy_level,
      survice_ids: policy.service_ids
    }
  end
  
  
```

コードについて説明します。
`include Active::Model`はActiveModelが提供するモジュール郡の一部をまとめたモジュールです。
複数のモジュールを組み合わせて、コントローラやビューのメソッドとの連携に必要なインターフェースを提供します。

<https://railsguides.jp/active_model_basics.html#model%E3%83%A2%E3%82%B8%E3%83%A5%E3%83%BC%E3%83%AB>

`include Active::Attributes`は属性を定義するためのattributesメソッドを利用できるようになるモジュールです。attributesメソッドでは、第1引数で属性名を、第2引数で型名を指定します。

<https://railsguides.jp/active_model_basics.html#attributemethods%E3%83%A2%E3%82%B8%E3%83%A5%E3%83%BC%E3%83%AB>

なお、今回のコードはRails5系で確認しております。
Rails7系では`ActiveModel::Attributes`についても一緒にincludeされるようになりました。

<https://techracho.bpsinc.jp/hachi8833/2022_01_28/114954>

`delegated`は委譲に関するメソッドです。
ruleモデルから

`initialize`はフォームオブジェクトの値を初期化しています。
`super`はActiveModel::Modelの`initialize`を呼び出しており、書き込みメソッド（`rule_name=`など）を用いて値を代入しています。 つまり、フォームオブジェクトで用いる値は書き込みメソッドを定義する必要があります。

また、更新にも対応する場合は`default_attributes`のように保存済みのレコードをもとに値を設定する必要があります。 更新に対応しない場合は`initialize`を定義する必要はありません。 この場合は`ActiveModel::Model`の`initialize`により自動で値の初期化を行ってくれます。

値に書き込みメソッドだけでなく読み取りメソッドも定義しているのは、ビューのフォームに必要なためです。 たとえば`form.text_field :rule_name`は`RuleManagementForm#rule_name`から値を取得します。 フォームの内容に応じてメソッドを定義する必要があります。

`persisted?`と`to_model`はビューの表示（`form_with`）に必要なメソッドです。 `persisted?`は作成・更新に応じてフォームのアクションをPOST・PATCHに切り替えてくれます。 また`to_model`はアクションのURLを適切な場所（ここではrule_pathやrule_path(id)）に切り替えてくれます。

ActiveRecordモデルと同じく、#validateを用い独自のバリデーションを設定できます。 errorsオブジェクトにエラーを追加すれば、#valid?での検証失敗時にユーザにフィードバックを表示することもできます。

### コントローラ

```ruby:app/controllers/rule_management_controller.rb
class RuleManagementController < ApplicationController
  def new
    @form = RuleManagementForm.new
  end

  def create
    @form = RuleManagementForm.new(rule_management_params)

    if @form.save
      redirect_to rule_managements_path
    else
      render :new
    end
  end

  def edit
    load_rule

    @form = RuleManagementForm.new(rule: @rule, )
  end

  def update
    load_rule

    @form = RuleManagementForm.new(rule_management_params, rule: @rule)

    if @form.save
      redirect_to @rule
    else
      render :edit
    end
  end

  private

  def rule_management_params
    params.require(:rule).permit(:rule_name, 
 read: {
 service_ids: []
 },
 write: {
 service_ids: []
 }
    )
  end

  def load_rule
    @rule = current_user.rule.find(params[:id])
  end
 
```



### ビュー

```ruby:app/views/rule_managements/new.html.erb
# persisted?があることで自動的にPOSTとPATCHに切り替える
<%= form_with model: rule, local: true do |form| %>
  <%= form.text_field :rule_name %>
  <%= rule.fields_for :read do |read| %>
  <%= read.collection_check_boxes :service_ids, Service.ids, :service_ids, :service_name do |service| %>
  <% end %>
  <%= rule.fields_for :write do |write| %>
  <%= write.collection_check_boxes :service_ids, Service.ids, :service_ids, :service_name do |service| %>
  <% end %>
  <%= form.submit %>
<% end %>
```

## 気になった点

## 参考文献・記事

[パーフェクト Ruby on Rails【増補改訂版】](https://www.amazon.co.jp/%E3%83%91%E3%83%BC%E3%83%95%E3%82%A7%E3%82%AF%E3%83%88-Ruby-Rails-%E3%80%90%E5%A2%97%E8%A3%9C%E6%94%B9%E8%A8%82%E7%89%88%E3%80%91-%E3%81%99%E3%81%8C%E3%82%8F%E3%82%89-%E3%81%BE%E3%81%95%E3%81%AE%E3%82%8A-ebook/dp/B08D3DW7LP/ref=sr_1_1?crid=1GVVRQ00LI7MO&keywords=%E3%83%91%E3%83%BC%E3%83%95%E3%82%A7%E3%82%AF%E3%83%88ruby+on+rails+%E5%A2%97%E8%A3%9C%E6%94%B9%E8%A8%82%E7%89%88&qid=1667008413&qu=eyJxc2MiOiIwLjkwIiwicXNhIjoiMS4wMCIsInFzcCI6IjEuMDAifQ%3D%3D&sprefix=%E3%83%91%E3%83%BC%E3%83%95%E3%82%A7%E3%82%AF%E3%83%88+Ruby%2Caps%2C212&sr=8-1)

[Railsのデザインパターン: Formオブジェクト](https://applis.io/posts/rails-design-pattern-form-objects)
