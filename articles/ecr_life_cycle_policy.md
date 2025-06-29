---
title: "ECRのライフサイクルポリシーを設定して費用を節約する"
emoji: "💰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","ECR"]
published: false
---

## はじめに

AWS ECR（Amazon Elastic Container Registry）を使用していると、CI/CDパイプラインからプッシュされるDockerイメージが蓄積され、知らず知らずのうちにストレージコストが増加していることがあります。

特に昨今の円安の影響もあり、AWSの利用料金がかさんでいるプロジェクトも多いのではないでしょうか。ECRの料金はEC2やRDSと比べると高額ではありませんが、塵も積もれば山となります。

この記事では、**AWS公式ドキュメント（2025年6月29日時点）**の情報に基づき、ECRのライフサイクルポリシーを活用して**不要なイメージを自動削除し、コストを大幅に削減する方法**について詳しく解説します。

📖 **参考資料**：
- [AWS公式：Automate the cleanup of images by using lifecycle policies in Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/LifecyclePolicies.html)
- [AWS公式：Amazon ECR Pricing](https://aws.amazon.com/ecr/pricing/)

## ECRの課金の仕組み

ECRの課金は主に以下の2つの要素で構成されています：

1. **ストレージ使用量**：リポジトリに保存されているDockerイメージのサイズ合計（GB/月）
2. **データ転送量**：別リージョンのECRにアクセスする際などに発生

基本的には**ストレージ使用量**が課金の大部分を占めます。

### 料金例（2025年6月現在）

**グローバルリージョン**の場合：
- **ストレージ料金**：**$0.10 USD/GB/月**
- **パブリックリポジトリ**：月間50GBの無料枠あり
- **プライベートリポジトリ**：新規ユーザーは1年間500MB/月の無料枠

**データ転送料金**：
- データ転送IN：**無料**
- データ転送OUT：
  - 最初の9.999TB/月：**$0.09/GB**
  - 次の40TB/月：**$0.085/GB**
  - 次の100TB/月：**$0.07/GB**
  - 150TB超/月：**$0.05/GB**

**料金計算例**：
```
例：5GBのDockerイメージを5環境分ECRに保存している場合
5GB × 5環境 × 1ヶ月 × $0.10/GB/月 = $2.5/月
```

**重要**：同一リージョン内のAWSサービス間（EC2、Fargate、Lambda等）のデータ転送は**無料**です。

一見大きな金額ではありませんが、ECRには重要な特徴があります：

**新しいイメージがプッシュされても、過去の古いイメージは自動削除されず残り続ける**

つまり、定期的にDockerイメージをリビルドしてECRにプッシュする運用では、ビルドの度にストレージ使用量が増加し、課金額も増加していきます。

## ECRライフサイクルポリシーとは

ECRライフサイクルポリシーは、**不要になったイメージを自動的に削除する機能**です。

**重要**：ライフサイクルポリシーの条件に合致したイメージは、**24時間以内に削除**されます。

この機能を使用することで：
- 指定した条件に基づいて古いイメージを自動削除
- ストレージコストの削減
- 運用の自動化

が可能になります。

### 設定可能な削除条件

1. **プッシュ後の経過日数**（`sinceImagePushed`）：「30日以上前にプッシュされたイメージを削除」
2. **保持イメージ数**（`imageCountMoreThan`）：「最新10個のイメージのみ保持し、それ以外を削除」
3. **タグの状態**：
   - `tagged`：タグが付いているイメージのみ対象
   - `untagged`：タグが付いていないイメージのみ対象  
   - `any`：すべてのイメージを対象
4. **タグパターン**：
   - `tagPatternList`：ワイルドカード（`*`）を使用したパターンマッチング
   - `tagPrefixList`：特定のプレフィックスによる完全一致

⚠️ **注意**：`tagPatternList`と`tagPrefixList`は同時に指定できません。どちらか一方のみ使用可能です。

## ライフサイクルポリシーの設定方法

### AWSコンソールでの設定手順

1. ECRコンソールでリポジトリを選択
2. 左メニューの「Lifecycle Policy」をクリック
3. 「ルールを追加」をクリック
4. 条件を設定してルールを保存

### 設定例1：経過日数による削除

プッシュから30日経過したイメージを削除する設定：

![設定画面のイメージ]

- **イメージステータス**：すべて
- **一致条件**：イメージをプッシュしてから
- **日数**：30日後

### 設定例2：保持イメージ数による削除

最新10個のイメージのみ保持する設定：

- **イメージステータス**：すべて
- **一致条件**：イメージ数
- **保持数**：10個

### リハーサル機能の活用

**必ずリハーサル機能を使用してから本運用を開始してください。**

AWS公式が推奨するライフサイクルポリシーのワークフロー：

1. **テストルールの作成**：まず一つ以上のテストルールを作成
2. **プレビューの実行**：テストルールを保存し、プレビューを実行
3. **評価プロセス**：ライフサイクルポリシー評価器がすべてのルールを処理し、各ルールが影響するイメージをマーク
4. **優先度による適用**：ルールの優先度（低い数字ほど高優先度）に基づいて削除対象イメージを決定
5. **結果確認**：削除予定のイメージが意図通りか確認
6. **本運用適用**：テストルールを正式なライフサイクルポリシーとして適用

⚠️ **重要な評価ルール**：
- 画像は**正確に1つまたは0のルール**によって削除される
- 高優先度ルールにマークされた画像は、低優先度ルールでは削除されない
- マニフェストリストから参照されているイメージは、マニフェストリストが先に削除されるまで削除されない

### タグパターンマッチングの詳細

`tagPatternList`を使用する場合の重要な制限事項：

- **ワイルドカード制限**：1つの文字列につき最大4個の`*`まで
- **有効な例**：`["*test*1*2*3", "test*1*2*3*"]`
- **無効な例**：`["test*1*2*3*4*5*6"]`（ワイルドカードが5個以上）

パターンマッチング例：
- `prod*`：`prod`、`prod1`、`production-team1`にマッチ
- `*prod*`：`repo-production`、`prod-team`にマッチ

## 実際の設定例とベストプラクティス

### 環境別の設定例

複数のライフサイクルポリシーを優先順位をつけて適用可能です：

| 優先度 | ルール名 | 対象 | 設定内容 |
|--------|----------|------|----------|
| 5 | 本番用イメージ | タグ「prd-」 | 30イメージを保持 |
| 10 | ステージング用 | タグ「stg-」 | 20イメージを保持 |
| 20 | 開発用 | タグ「dev-」 | 10イメージを保持 |
| 90 | 全イメージ | タグ条件なし | 5イメージを保持 |

### シンプルな設定例

コミットハッシュのみをタグとして利用している場合：

| 優先度 | ルール名 | 設定内容 |
|--------|----------|----------|
| 10 | イメージ有効期限 | 30イメージを保持（タグ条件なし） |

## Infrastructure as Codeでの実装

### Terraformでの実装例

**例1：タグプレフィックスによる期間ベース削除**

```hcl
resource "aws_ecr_lifecycle_policy" "prefix_based" {
  repository = aws_ecr_repository.example.name

  policy = jsonencode({
    "rules": [
      {
        "action": {
          "type": "expire"
        },
        "selection": {
          "countType": "sinceImagePushed",
          "countUnit": "days",
          "countNumber": 14,
          "tagStatus": "tagged",
          "tagPrefixList": [
            "work"
          ]
        },
        "description": "workプレフィックスのイメージを14日後に自動削除",
        "rulePriority": 1
      }
    ]
  })
}
```

**例2：タグパターンによる数量ベース削除**

```hcl
resource "aws_ecr_lifecycle_policy" "pattern_based" {
  repository = aws_ecr_repository.example.name

  policy = jsonencode({
    "rules": [
      {
        "action": {
          "type": "expire"
        },
        "selection": {
          "countType": "imageCountMoreThan",
          "countNumber": 10,
          "tagStatus": "tagged",
          "tagPatternList": [
            "dev*",
            "*test*"
          ]
        },
        "description": "dev*および*test*パターンのイメージを10個まで保持",
        "rulePriority": 1
      }
    ]
  })
}
```

**例3：複数ルールの組み合わせ**

```hcl
resource "aws_ecr_lifecycle_policy" "multi_rules" {
  repository = aws_ecr_repository.example.name

  policy = jsonencode({
    "rules": [
      {
        "action": {
          "type": "expire"
        },
        "selection": {
          "countType": "imageCountMoreThan",
          "countNumber": 30,
          "tagStatus": "tagged",
          "tagPrefixList": ["prod"]
        },
        "description": "本番イメージは30個まで保持",
        "rulePriority": 1
      },
      {
        "action": {
          "type": "expire"
        },
        "selection": {
          "countType": "sinceImagePushed",
          "countUnit": "days",
          "countNumber": 7,
          "tagStatus": "tagged",
          "tagPrefixList": ["dev"]
        },
        "description": "開発イメージは7日で削除",
        "rulePriority": 2
      },
      {
        "action": {
          "type": "expire"
        },
        "selection": {
          "countType": "imageCountMoreThan",
          "countNumber": 1,
          "tagStatus": "untagged"
        },
        "description": "タグなしイメージは1個のみ保持",
        "rulePriority": 3
      }
    ]
  })
}
```

### CloudFormationでの実装例

```yaml
ECRLifecyclePolicyMultiRules:
  Type: AWS::ECR::LifecyclePolicy
  Properties:
    RepositoryName: !Ref ECRRepository
    PolicyText: !Sub |
      {
        "rules": [
          {
            "action": {
              "type": "expire"
            },
            "selection": {
              "countType": "imageCountMoreThan",
              "countNumber": 20,
              "tagStatus": "tagged",
              "tagPatternList": ["prod*"]
            },
            "description": "本番イメージを20個まで保持",
            "rulePriority": 1
          },
          {
            "action": {
              "type": "expire"
            },
            "selection": {
              "countType": "sinceImagePushed",
              "countUnit": "days",
              "countNumber": 30,
              "tagStatus": "tagged",
              "tagPatternList": ["staging*", "stg*"]
            },
            "description": "ステージングイメージを30日で削除",
            "rulePriority": 2
          },
          {
            "action": {
              "type": "expire"
            },
            "selection": {
              "countType": "imageCountMoreThan",
              "countNumber": 1,
              "tagStatus": "untagged"
            },
            "description": "タグなしイメージは1個のみ保持",
            "rulePriority": 3
          },
          {
            "action": {
              "type": "expire"
            },
            "selection": {
              "countType": "imageCountMoreThan",
              "countNumber": 5,
              "tagStatus": "any"
            },
            "description": "最終的な保護として5個を保持",
            "rulePriority": 99
          }
        ]
      }
```

⚠️ **CloudFormation実装時の注意点**：
- `tagStatus`が`any`のルールは最も低い優先度（最も高い数値）にする必要があります
- 複数の`tagPatternList`エントリは、すべてのパターンが一致する必要があります

## 注意点とバッドパターン

### ⚠️ 危険な設定例

#### 1. 保持イメージ数のみの制限

```json
{
  "description": "最新30個のイメージのみ保持"
}
```

**問題点**：
- テストフェーズで頻繁にイメージをプッシュすると、本番で使用中のイメージが削除される可能性
- 本番環境で障害が発生するリスク

#### 2. 経過日数のみの制限

```json
{
  "description": "30日経過したイメージを削除"  
}
```

**問題点**：
- しばらく更新されないが稼働し続ける必要があるサービスで、使用中のイメージが削除される可能性

### 🛡️ 安全な運用のための対策

1. **環境別リポジトリの分離**
   - 開発環境と本番環境でECRリポジトリを分ける
   - 本番用は保守的なポリシーを設定

2. **タグ命名ルールの整備**
   - 本番リリース用イメージには「release」「prod」等のプレフィックス
   - ライフサイクルポリシーから除外されるタグ設計

3. **段階的な導入**
   - まず開発環境で動作確認
   - リハーサル機能で必ず事前確認
   - 少しずつ本番環境に適用

## 実際の削減効果

### 削減事例1
- **削減前**：1300個のイメージが蓄積
- **削減後**：10個のイメージのみ保持
- **削減効果**：**75%のコスト削減**を実現

### 削減事例2
- **削減前**：月々増加し続ける課金
- **削減後**：週次でのクリーンアップ実行
- **削減効果**：**月額$500の削減**を達成

## より高度なポリシー要件への対応

ECRライフサイクルポリシーでは、複数条件をAND条件で組み合わせることができません。

例えば「プッシュから30日経過 AND 最新10個に含まれない」といった条件は標準機能では実現できません。

### Lambda関数による実装

このような場合は、以下のアプローチが有効です：

1. **Lambda関数**でカスタムロジックを実装
2. **Step Functions**でワークフローを管理
3. **EventBridge**で定期実行をスケジュール

```python
def filter_expired_images(images):
    RETAIN_IMAGE_COUNT = 10
    RETAIN_SINCE_IMAGE_PUSHED_DAYS = 30
    
    # pandasを使用してデータフレーム上で条件評価
    df = pd.DataFrame.from_dict(data=images, orient="columns")
    df = df.sort_values(by="imagePushedAt", ascending=False)
    
    # 最新N個の保持条件
    df["retainImageCount"] = df.index < RETAIN_IMAGE_COUNT
    
    # 経過日数の保持条件  
    df["retainSinceImagePushedDays"] = df.apply(
        lambda row: (datetime.now() - row["imagePushedAt"]).days <= RETAIN_SINCE_IMAGE_PUSHED_DAYS,
        axis=1
    )
    
    # いずれの保持条件にも該当しないものを削除対象
    df["isExpired"] = ~df["retainImageCount"] & ~df["retainSinceImagePushedDays"]
    
    return df[df["isExpired"]].to_dict(orient="records")
```

## まとめ

ECRライフサイクルポリシーは、以下のメリットを提供します：

✅ **コスト削減**：不要なイメージの自動削除によるストレージコスト削減  
✅ **運用自動化**：手動でのイメージ削除作業から解放  
✅ **リポジトリ整理**：ECRリポジトリを常にクリーンな状態に維持  

ただし、設定には注意が必要です：

⚠️ **慎重な設計**：使用中のイメージを誤って削除しないよう配慮  
⚠️ **段階的導入**：リハーサル機能を活用した安全な導入  
⚠️ **チーム合意**：タグ命名ルールや運用ポリシーの共有  

ECRライフサイクルポリシーを適切に設定することで、**75%ものコスト削減**も実現可能です。まずは開発環境から試してみて、徐々に本番環境に展開していくことをお勧めします。

円安が続く昨今、このような地道なコスト最適化の積み重ねが重要になってきています。ぜひ皆さんの環境でも導入を検討してみてください。

