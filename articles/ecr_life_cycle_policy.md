---
title: "【忘備録】ECRのライフサイクルポリシーを設定する"
emoji: "💰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","ECR"]
published: false
--- 

## はじめに

私は競馬予想AIアプリケーションを個人開発しています。

https://www.horecast.net/

バックエンドのPythonの部分でAWS ECRを使用していますが
AWSの費用の多くがECRに起因していることに気づきました。

![aws_cost](/images/aws_cost.png)

ライフサイクルポリシーを設定して不要なコストを削減することにしました。

## ECRの課金の仕組み

ライフサイクルポリシーを設定する前に、ECRの課金の仕組みを把握していないので整理します。
ECRの課金は主に以下の2つの要素で構成されています：

1. **ストレージ使用量**：リポジトリに保存されているDockerイメージのサイズ合計（GB/月）
2. **データ転送量**：別リージョンのECRにアクセスする際などに発生
  
### 詳細（2025年７月現在）

**ストレージ料金**：**$0.10 USD/GB/月**

※プライベートリポジトリとパブリックリポジトリ共通

**プライベートリポジトリからのデータ転送料金(東京リージョンの場合)**：

- データ受信：**無料**
- データ送信：
  - 最初の9.999TB/月：**$0.114/GB**
  - 次の40TB/月：**$0.089/GB**
  - 次の100TB/月：**$0.086/GB**
  - 150TB超/月：**$0.0084/GB**

データ受信/送信はECRへの転送/ECRからの転送を意味します。
同一リージョン内のECRと他のサービス間で転送されるデータは無料です。

## ライフサイクルポリシーの設定方法

Github ActionsでECRにタグ付きでプッシュする設定をしました。
今回はタグがついていないものは2つだけ残して、
あとは削除するライフサイクルポリシーを設定します。

### 設定手順

1. ECRコンソールでリポジトリを選択
2. 左メニューの「Lifecycle Policy」をクリック
3. 「テストルールの編集」→「ルールを追加」と遷移
4. 「ライフサイクルテストルールを作成」で内容を入力

![create_test_rule](/images/create_test_rule.png)

5. 保存をクリックしてから「テストを実行」をクリックすると現時点での削除対象イメージが表示される

![match_rule](/images/match_rule.png)

6. 問題なければ「ライフサイクルポリシーとして適用」をクリックするとjsonが表示されるので確認して「保存」をクリック

![confirm_policy](/images/confirm_policy.png)

## 注意点

当たり前ですが、ライフサイクルポリシー設定時は
他のサービスでのECRのイメージの使用状況を確認しましょう。

緊急対応でECSでECRのイメージのタグ番号を直書きで設定した際に

**参考資料**：

- [AWS公式：Automate the cleanup of images by using lifecycle policies in Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/LifecyclePolicies.html)
- [AWS公式：Amazon ECR Pricing](https://aws.amazon.com/ecr/pricing/)