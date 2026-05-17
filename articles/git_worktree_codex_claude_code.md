---
title: "Codex AppとClaude Code Desktopのワークツリー機能について"
emoji: "🌲"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["git","worktree","codex","claudecode"]
published: false
---

## はじめに

Codexへの指示と人力での確認作業をアプリ内で完結できる点が気に入り、
Codex Appを公開後から日常的に使っています。
VS Codeを開かずに一日を終えることも増えました。

一方でCodex Appのワークツリー機能を何度か使用したものの、
雰囲気で使用している面があり、機能理解が曖昧なままでした。

また、Claude Code Desktopも最近利用し始めましたが、
こちらにもワークツリー機能があります。
Codex App と Claude Code Desktop の両方を触るなら、
それぞれのワークツリーが何を解決していて、どこが違うのかを整理しておきたいです。

この記事では、まず `git worktree`の基本を確認したうえで、
Codex App と Claude Code Desktop のワークツリー機能を比較します。

:::message
Codex App と Claude Code Desktop の説明は、2026年5月16日時点で確認した内容です。
UI や仕様は変わる可能性があるため、最新の挙動は公式ドキュメントも確認してください。
:::

## git worktree について

`git worktree` は、1つの Git リポジトリに複数の作業ツリーを紐づけて管理するためのコマンドです。

通常、1つのリポジトリで同時にチェックアウトできるブランチは1つです。
別のブランチで作業したい場合は、`git switch` などでブランチを切り替える必要があります。

一方で `git worktree` を使うと、
同じリポジトリを共有したまま、別ディレクトリに別ブランチの作業場所を作れます。

```bash
git worktree add -b hotfix ../my-repo-worktrees/hotfix
```

この場合、`../my-repo-worktrees/hotfix` に `hotfix` ブランチ用の作業ツリーが作られます。
元のディレクトリで進めていた変更を退避せずに、
別ディレクトリで修正作業を進められます。

### コマンド

#### listコマンド

紐づいている作業ツリーの一覧を表示します。

```bash
git worktree list
```

#### addコマンド

ワークツリーを新規作成します。

```bash
git worktree add <path> <branch>
```

`<path>`は新規ワークツリーが作成される場所、
`<branch>`は checkout するブランチです。

`-b`をつけることで新規ブランチを作成して
ワークツリーで作業することができます。

```bash
# 新しいブランチを作成して別ディレクトリにチェックアウトする
git worktree add -b feature ../my-repo-worktrees/feature

# リモートブランチを元に別ディレクトリへ作業ツリーを作る
git worktree add -b feature ../my-repo-worktrees/feature origin/feature
```

#### removeコマンド

ワークツリーを削除します。

```bash
git worktree remove <path>
```

先ほど緊急修正のためaddコマンドで作成した
ワークツリーを削除する場合は次のコマンドになります

```bash
git worktree remove ../my-repo-hotfix
```

#### pruneコマンド

手動でディレクトリを削除した場合など、
壊れたworktreeを掃除します。

```bash
git worktree prune
```

### 注意点

`git worktree` で作成される別ディレクトリには、
Git で管理されているファイルがチェックアウトされます。
そのため、`.gitignore` に記載されているファイルやディレクトリは反映対象外です。

たとえば、`node_modules/` のような依存関係の実体や、
`.env` のような環境変数ファイルは、worktree を追加しても自動では用意されません。

別の作業ツリーでアプリケーションを動かす場合は、
依存関係のインストールや環境変数ファイルの準備を作業ツリーごとに行う必要があります。

## Codex App の Worktrees について

Codex App では `git worktree` を使うことで、
同じプロジェクト内で複数の独立したタスクを実行しても、
互いに干渉しにくい作業場所を用意できます。

たとえば、定期的に指定した処理を自動実行する[Automations](https://developers.openai.com/codex/app/automations) では、
実行時に Worktree が作成されます。

Codex App のドキュメントでは、ユーザーが元々作った checkout を `Local checkout`、
Codex がそこから作成した Git worktree を `Worktree`、
Local と Worktree の間で thread を移動する流れを `Handoff` と呼んでいます。

作成した Worktree は、`CODEX_HOME/worktrees` 配下に保存されます。

### Worktree で実行する

Codex App で Worktree thread を開始する際は、
新規スレッド作成時に `新しいWorktree` を選択します。

このとき、後述する `Local environment` を事前に作成しておくと、
Worktree 作成時に `Local environment` で設定したコマンドが実行されます。

### 開発ワークフロー

Worktree を用いた開発ワークフローは、大きく次の2パターンに分けられます。

1. Worktree 上だけで作業する
2. スレッドを Local に Handoff する

#### Worktree 上だけで作業する

ワークツリー内で作業が完結できる場合は、
Codexの画面から Worktree をブランチ化できます。

![](/images/codex_right_side_bar.png)

![](/images/create_branch_here.png)

そこから通常の開発フローと同じように、プルリクエスト作成まで進められます。

#### スレッドを Local に Handoff する

ワークツリーだけでは確認が難しい場合は、
スレッドをフォアグラウンドに持っていく必要があります。

その場合は、Codexの画面からWorktree → ブランチに引き継ぐをクリックし、
移動先として `Local` を選択します。

![](/images/worktree_handoff.png)

これにより Codex は、Worktree と Local checkout の間でスレッドを
安全に移動するために必要な Git 手順を処理します。
また、スレッドを再び Worktree に Handoff することも可能です。

### Codex-managed worktree と permanent worktree

Codex App には、軽量で使い捨てを想定した `Codex-managed worktree` と、
長期利用する `permanent worktree` があります。

`Codex-managed worktree` は通常1 thread 専用です。
一方で `permanent worktree` はプロジェクトの三点メニューから作成でき、
自動削除されず、複数 thread を開始できます。

![](/images/codex_permanent.png)

### Local environment

`Local environment` では、ワークツリー用のセットアップ手順や、
プロジェクトでよく使う共通アクションを設定できます。

作成された設定はプロジェクト直下の `.codex` フォルダに保存され、
他のメンバーと共有できます。

![](/images/setup_local_environment.png)

#### セットアップスクリプト

前述した通り、ワークツリーを追加した際には
`.gitignore` に記述されたファイルは引き継がれません。
そのため、`.env` などが不足して作業に支障が生じるケースがあります。

`Setup scripts` はこの問題の解決に利用できます。
Codex App が新しいスレッド開始時に新しいワークツリーを作成すると、
自動的に実行されます。

これにより、`.env.sample` から `.env` の作成や、
依存関係のインストールの自動実行が可能になります。

また、プラットフォームごとにセットアップ手順が異なる場合は、
macOS、Windows、Linux ごとにセットアップスクリプトを定義できます。

#### クリーンアップスクリプト

ワークツリーのクリーンアップ時に自動実行するスクリプトを定義できます。

上に掲載した画像の通り、2026年5月10日現在、
Codex App の UI には存在しますが、
公式ドキュメントには記載がありませんでした。

#### アクション

アクションを使うと、開発サーバー起動やテスト実行のような
よく使うコマンドをヘッダーに表示して実行できます。

定義したアクションを実行すると、Codex アプリ上のターミナルで実行されます。
こちらもプラットフォームごとに設定できます。

## Claude Code Desktop のワークツリーについて

Claude Code Desktop でも Codex App と同じように、
ローカル環境、ワークツリー、クラウド環境での実行を選ぶことができます。

作成したワークツリーは、既定でプロジェクト直下の `.claude/worktrees/` に保存されます。
デフォルトでは、ワークツリーを作成するたびに
プロジェクト内にワークツリー側の作業ディレクトリが増えていきます。
`.claude/worktrees/`を`.gitignore`に追加して Git の追跡対象外にする運用が必要です。
また、設定画面の `Claude Code` → `ワークツリーの場所` から保存場所は変更可能です。

↓何も設定しない場合に増えていくワークツリー

![](/images/claude_worktree.png)

少なくとも確認した範囲では、Claude Code Desktop には Codex App の Handoff に相当する、
チェックアウトブランチとワークツリー間の移動を明示的に支援する機能は見当たりませんでした。

一方で、`.env` など Git 管理外だが worktree で必要なファイルについては、
プロジェクト直下に `.worktreeinclude` を置くことで、
ワークツリー作成時に新しい worktree へ自動コピーできます。

```text
.env
.env.local
```

作業が終わったセッションは、アーカイブすることで worktree を削除できます。
また、PR がマージまたはクローズされた後に、
セッションを自動アーカイブするように設定することもできます。

### Worktree に関する設定

以下の設定は、デスクトップ版だけでなく CLI でも利用できます。

#### baseRef

ワークツリー作成時の参照元を指定します。

`fresh` はデフォルト値で、`origin/<default-branch>` からブランチして、
リモートと一致するクリーンツリーを取得します。

`head` は現在のローカル HEAD からブランチします。
そのため、プッシュされていないコミットやフィーチャーブランチの状態が
worktree に存在します。

#### symlinkDirectories

メインリポジトリから各ワークツリーにシンボリックリンクするディレクトリを設定できます。

ファイルの複製は実行しないため、`node_modules` や `.cache` のような
大規模なディレクトリの重複を避け、
通常のブランチとワークツリーで共有できます。

#### sparsePaths

ワークツリーでチェックアウトするディレクトリを指定できます。
リストされたパスのみがディスクに書き込まれます。

大規模なリポジトリで、作業に必要なディレクトリだけを取り出したい場合に有効です。

## まとめ

`git worktree` は、同じリポジトリから複数の作業場所を作るための Git の機能です。
Codex App と Claude Code Desktop のワークツリー機能も、この考え方を土台にしています。

Codex App は、thread を Worktree と Local の間で動かす Handoff や、
作業環境を整える Local environment が特徴です。

Claude Code Desktop は、`.worktreeinclude` や worktree 設定によって、
作成されるワークツリーの中身を調整しやすい点が特徴です。

どちらを使う場合も、`.env` や依存関係のような Git 管理外ファイルは自動では揃わない前提で、
セットアップ方法を先に決めておくのが大事だと感じました。

## 参考

- [Git - git-worktree Documentation](https://git-scm.com/docs/git-worktree)
- [OpenAI Developers | Codex Worktrees](https://developers.openai.com/codex/app/worktrees)
- [OpenAI Developers | Local environments](https://developers.openai.com/codex/app/local-environments)
- [Claude Code Docs | Claude Code Desktop を使用する](https://code.claude.com/docs/ja/desktop)
- [Claude Code Docs | worktree を使用して並列セッションを実行する](https://code.claude.com/docs/ja/worktrees)
- [Claude Code Docs | Claude Code の設定](https://code.claude.com/docs/ja/settings#worktree-%E8%A8%AD%E5%AE%9A)
