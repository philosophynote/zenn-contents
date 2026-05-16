---
title: "CodexとClaude Codeのワークツリー機能について"
emoji: "🌲"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["git","git worktree","Codex","Claude Code"]
published: false
---

## はじめに

Codex Appのワークツリー機能を有効に使用できていないので、
git worktreeの概要理解から入り機能を理解する。
また、Claude Codeのデスクトップ版でもワークツリー機能があるため、
Codex Appとの差分を知ることで理解を深める。

## git worktreeについて

`git worktree`は、1つのGitリポジトリに複数の作業ツリーを紐づけて管理するためのコマンド。

通常、1つのリポジトリで同時にチェックアウトできるブランチは1つ。
別のブランチで作業したい場合は、`git switch` などでブランチを切り替える必要がある。
一方で `git worktree` を使うと、
同じリポジトリを共有したまま、別ディレクトリに別ブランチの作業場所を作れる。

`git worktree add <path>`で新しい作業ツリーを追加できる。
この作業ツリーは既存リポジトリに紐づくため、オブジェクトなどの共通情報は共有しつつ、
`HEAD` や `index` など作業ツリーごとに必要な情報は分けて持てる。

たとえば、現在の作業を残したまま緊急修正用のブランチを別ディレクトリに作る場合は、
次のようになる。

```bash
git worktree add -b hotfix ../my-repo
```

この場合、`../my-repo` に `hotfix` ブランチ用の作業ツリーが作られる。
元のディレクトリで進めていた変更を退避せずに、
別ディレクトリで修正作業を進められる。

使い終わった作業ツリーは、次のように削除する。

```bash
git worktree remove ../my-repo
```

### 関連コマンド

```bash
# 紐づいている作業ツリーの一覧を表示する
git worktree list

# 既存ブランチを別ディレクトリにチェックアウトする
git worktree add ../my-repo feature

# 新しいブランチを作成して別ディレクトリにチェックアウトする
git worktree add -b feature ../my-repo

# リモートブランチを元に別ディレクトリへ作業ツリーを作る
git worktree add -b feature ../my-repo origin/feature

# 作業ツリーを削除する
git worktree remove ../my-repo

# ディレクトリを手動削除した後に残った管理情報を掃除する
git worktree prune

# 作業ツリーのパスを変更する
git worktree move ../my-repo ../my-repo-renamed

# 外部ディスクなどにある作業ツリーが prune されないようにする
git worktree lock ../my-repo

# lock した作業ツリーを解除する
git worktree unlock ../my-repo
```

基本的には、追加は `git worktree add`、
確認は `git worktree list`、
削除は `git worktree remove` を使う。
`git worktree prune` は、作業ツリーのディレクトリを手動で消してしまった場合などに、
Git側に残った不要な管理情報を整理するためのコマンドです。
通常の削除では `rm -rf` ではなく `git worktree remove` を使う方が安全です。

### 注意点

`git worktree` で作成される別ディレクトリには、
Gitで管理されているファイルがチェックアウトされる。
そのため、`.gitignore` に記載されているファイルやディレクトリは反映対象外となる。

例えば、`node_modules/` のような依存関係の実体や、
`.env` のような環境変数ファイルは、worktreeを追加しても自動では用意されない。
別の作業ツリーでアプリケーションを動かす場合は、
依存関係のインストールや環境変数ファイルの準備を作業ツリーごとに行う必要がある。

## Codex AppのWorktreesについて

Codex Appでは、`git worktree`を使うことで、
同じプロジェクト内で複数の独立したタスクを実行しても、
互いに干渉しないようにすることを可能にしている。

定期的なタイミングで指定した処理を自動実行する
[Automations](https://developers.openai.com/codex/app/automations)で実行される
処理は実行時にWorktreeを作成して実行される。

Codex Appのドキュメントではユーザーが
元々作ったcheckoutを`Local checkout`、
Codexがそこから作成したGit worktreeを`Worktree`、
LocalとWorktreeの間でthreadを移動する流れを`Handoff`と呼んでいる。

作成したWorktreeはCODEX_HOME/worktrees 配下に保存される。

### Worktreeで実行する

Codex AppでWorktree threadを開始する際には
新規スレッドで実行する際に`新しいWorktree`を選択する。
このとき、後述する`ローカル環境`を事前作成しておくと
Worktree作成時に`ローカル環境`で設定したコマンドが実行される。

#### handoff

Worktreeを用いた開発ワークフローは次の2パターンが考えられる。

1.worktree上だけで作業する
2.スレッドをLocalにhandoffする

##### 1.Worktree上だけで作業する

ワークツリーで作業が完結できる場合は
スレッドヘッダーの`Create branch here`ボタンを使ってWorktreeを
ブランチ化できる。
そこから通常の開発フローと同じくプルリクエスト作成まで実行可能。
注意点として、ワークツリーでブランチを作成した場合、
そのブランチを他のワークツリーでチェックアウトすることはできない。

##### 2.スレッドをLocalへhandoffする

ワークツリーでの確認ができない場合は、
スレッドをフォアグラウンドに持っていく必要がある。
スレッドヘッダーの Hand offをクリックし、移動先として Local を選択する。

これによりCodexは、ワークツリーとlocal checkoutの間でスレッドを
安全に移動するために必要なGit手順を処理する。
また、スレッドを再びworktreeにhandoffすることも可能。

#### Codex-managed worktreeとpermanent worktree

軽量で使い捨てを想定した`Codex-managed worktree`と、長期利用する`permanent worktree`がある。
`Codex-managed worktree`は通常1thread専用で、`permanent worktree`は
プロジェクトの三点メニューから作成でき、自動削除されず複数threadを開始できる。

![](images/codex_permanent.png)

### Local environment

`Local environment`では、ワークツリー用のセットアップ手順や、
プロジェクトでよく使う共通アクションを設定できる。

作成された設定はプロジェクト直下の`.codex`フォルダに保存され、
他のメンバーと共有が可能。

![](images/setup_local_environment.png)

#### セットアップスクリプト

前述した通り、ワークツリーをaddした際には
`.gitignore`で記述されたファイルは引き継がれないため、
`.env`などが不足して作業に支障が生じるケースが想定される。

`Setup scripts`はこの問題の解決に利用する。
Codex Appが新しいスレッド開始時に新しいワークツリーを作成すると
自動的に実行される。

これにより、`.env.sample`から`.env`の作成や、
依存関係のインストールの自動実行が可能となる。

また、プラットフォームごとにセットアップ手順が異なる場合は
macOS、Windows、Linuxごとにセットアップスクリプトを定義することが可能。

#### クリーンアップスクリプト

ワークツリーのクリーンアップ時に自動実行するスクリプトを定義できる。
上に掲載した画像の通り、2026年5月10日現在、
Codex Appには存在するが、
公式ドキュメントには記載がない。

#### アクション

アクションを使うと、開発サーバー起動やテスト実行のような
よく使うコマンドをヘッダーに表示して実行できる。

定義したアクションを実行するとCodexアプリ上のターミナルで実行される。
こちらについてもプラットフォームごとに設定が可能。

## Claude Code Desktopのワークツリーについて

Claude Code DesktopでもCodex Appと同じように
ローカル環境/ワークツリー/クラウド環境での実行を選ぶことが可能。

作成したワークツリーは既定でプロジェクト直下の`.claude/worktrees/`に保存される。
デフォルトだとワークツリーを作成する度に
プロジェクトにワークツリー側の作業ディレクトリに関するファイルが増えていくが、
設定 → Claude Code → ワークツリーの場所から保存場所は変更可能。

↓何も設定しない場合に増えていくワークツリー
![](images/claude_worktree.png)

Codexと異なり、チェックアウトブランチとワークツリー間の連携を行うコマンドは存在しない。
しかしながら、`.env`などGit管理外だがworktreeで必要なファイルについては、
プロジェクト直下に`.worktreeinclude`を置くことで
ワークツリー作成時に自動的に新しいworktreeへコピーすることが可能。

作業が終わったセッションをアーカイブすることでworktreeを削除することが可能。
また、PRがマージ、クローズされた後にセッションを自動アーカイブするように設定もできる。

### Worktreeに関する設定

これはデスクトップ限定ではなくCLIも同様です。

#### baseRef

ワークツリー作成時の参照元。
`fresh`（デフォルト）は`origin/<default-branch>`からブランチして、
リモートと一致するクリーンツリーを取得します。
`head`は現在のローカルHEADからブランチするため、
プッシュされていないコミットとフィーチャーブランチの状態が`worktree`に存在する。

#### symlinkDirectories

メインリポジトリから各ワークツリーにシンボリックリンクするディレクトリを設定できる。
ファイルの複製は実行しないため、`node_modules`や`.cache`のような大規模なディレクトリの重複を避け、
通常のブランチとワークツリーで共有することができる。

#### sparsePaths

ワークツリーでチェックアウトするディレクトリを指定することができる。
リストされたパスのみがディスクに書き込まれます。
大規模なレポジトリで有効に働きます。

## 参考

- [Git - git-worktree Documentation](https://git-scm.com/docs/git-worktree)
- [Codex Worktrees](https://developers.openai.com/codex/app/worktrees)
- [Local environments](https://developers.openai.com/codex/app/local-environments)
- [Claude Code Desktop を使用する](https://code.claude.com/docs/ja/desktop)
- [Claude Code の設定](https://code.claude.com/docs/ja/settings#worktree-%E8%A8%AD%E5%AE%9A)
