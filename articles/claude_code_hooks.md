---
title: "Claude Code の Hooks 機能を使いこなす"
emoji: "🪝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["claudecode", "AI", "hook", "自動化"]
published: false
---

## はじめに

Claude Code には Hooks という仕組みがあり、
Claude が特定のアクションを実行する前後にシェルコマンドを自動実行できる。

たとえば「ファイルを編集したら自動でフォーマッターをかける」「ツールを呼び出すたびにログを残す」といった処理を、
Claude の実行フローに割り込むかたちで組み込める。

本記事では Hooks の仕組み・設定方法・実際の活用例を紹介する。

## Hooks とは

Hooks は Claude Code の特定のイベントに応じて**ユーザー定義のシェルコマンドを実行する**仕組みだ。

Claude がツールを呼び出したり、応答を返したりする際に任意のコマンドを差し込めるため、
ワークフローの自動化・監査ログの記録・外部サービスへの通知など幅広い用途に使える。

実行されるコマンドはあくまで**ユーザー自身が定義したスクリプト**なので、
Claude がどんな指示を出してきても、Hooks は常にユーザーの意図した処理だけを行う。

## 対応しているイベント

Claude Code が提供する Hook イベントは次の5種類。

| イベント | タイミング |
|---|---|
| `PreToolUse` | ツール呼び出しの**前** |
| `PostToolUse` | ツール呼び出しの**後** |
| `Notification` | Claude が通知を送るとき |
| `Stop` | メインエージェントが応答を終了するとき |
| `SubagentStop` | サブエージェントが応答を終了するとき |

### PreToolUse / PostToolUse

最も使用頻度の高いイベント。
`matcher` でツール名を絞り込め、特定のツールにのみ Hook を適用できる。

`PreToolUse` で `exit 2` を返すとツール呼び出しをブロックでき、
標準エラーに書いたメッセージが Claude へのフィードバックとして渡される。

### Notification

Claude がデスクトップ通知を送ろうとする際に発火する。
macOS の `osascript` などと組み合わせてカスタム通知を実装するのに使える。

### Stop / SubagentStop

Claude がターンを終了するタイミングで発火する。
完了後の後処理やサマリーの記録に活用できる。

## 設定方法

Hooks は `settings.json` に記述する。

設定ファイルの場所は次の通り。

| スコープ | パス |
|---|---|
| ユーザー全体 | `~/.claude/settings.json` |
| プロジェクト | `.claude/settings.json` |

### 基本的な書き方

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolName",
        "hooks": [
          {
            "type": "command",
            "command": "実行するコマンド"
          }
        ]
      }
    ]
  }
}
```

`matcher` はツール名との部分一致で動作する。
空文字列 `""` またはキー自体を省略するとすべてのツールにマッチする。

`EventName` には対応しているイベント名をそのまま記述する（例: `PreToolUse`）。

### 設定例: ファイル編集後に自動フォーマット

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write \"$CLAUDE_TOOL_INPUT_FILE_PATH\""
          }
        ]
      }
    ]
  }
}
```

### 設定例: Bash 実行前に確認ログを残す

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"[$(date)] CMD: $CLAUDE_TOOL_INPUT_COMMAND\" >> ~/claude_audit.log"
          }
        ]
      }
    ]
  }
}
```

## Hook に渡される情報

Hook コマンドが実行されるとき、Claude はツール呼び出しの詳細を**標準入力（stdin）の JSON** として渡す。

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.json",
  "hook_event_name": "PreToolUse",
  "tool_name": "Edit",
  "tool_input": {
    "file_path": "src/main.ts",
    "old_string": "foo",
    "new_string": "bar"
  }
}
```

`tool_input` の構造はツールごとに異なる。
`Bash` なら `command`、`Edit` なら `file_path` / `old_string` / `new_string` が含まれる。

また、以下の環境変数も利用できる。

| 環境変数 | 内容 |
|---|---|
| `CLAUDE_TOOL_INPUT_*` | ツールの各入力フィールド（`file_path` → `CLAUDE_TOOL_INPUT_FILE_PATH`） |
| `CLAUDE_SESSION_ID` | セッション ID |
| `CLAUDE_TRANSCRIPT_PATH` | トランスクリプトファイルのパス |
| `CLAUDE_HOOK_EVENT_NAME` | 発火したイベント名 |
| `CLAUDE_TOOL_NAME` | 呼び出されたツール名 |

## Hook の終了コードによる制御

`PreToolUse` の Hook では終了コードでツールの実行を制御できる。

| 終了コード | 動作 |
|---|---|
| `0` | ツールをそのまま実行する |
| `2` | ツールの実行をブロックし、stderr の内容を Claude へ伝える |
| その他 | Hook の失敗として扱われる（ツールは実行される） |

`exit 2` を使ったブロックの例:

```bash
#!/bin/bash
# rm コマンドの実行を禁止するスクリプト
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // ""')

if echo "$COMMAND" | grep -qE '^\s*rm\s'; then
  echo "rm コマンドの実行はポリシーにより禁止されています" >&2
  exit 2
fi
```

この場合、Claude は `stderr` に書いた内容をフィードバックとして受け取り、
別の方法で対処しようとする。

## 実用的な活用例

### 1. コード整形の自動化

ファイルを編集するたびに自動でフォーマッターを実行する。

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "cd \"$CLAUDE_TOOL_INPUT_FILE_PATH\" && npx prettier --write . 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

### 2. macOS の通知

Claude が長い処理を終えたタイミングで macOS の通知を出す。

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude が応答を完了しました\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

### 3. 危険なコマンドのブロック

本番環境のデータを消去しかねないコマンドを事前にチェックする。

```bash
#!/bin/bash
# ~/.claude/hooks/check_destructive.sh
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // ""')

PATTERNS=("DROP TABLE" "DELETE FROM" "TRUNCATE" "rm -rf /")

for PATTERN in "${PATTERNS[@]}"; do
  if echo "$COMMAND" | grep -qi "$PATTERN"; then
    echo "危険なコマンドが検出されました: $PATTERN" >&2
    exit 2
  fi
done
```

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/hooks/check_destructive.sh"
          }
        ]
      }
    ]
  }
}
```

### 4. 操作ログの記録

すべてのツール呼び出しを JSON ログとして残す。

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "cat >> ~/claude_activity.jsonl"
          }
        ]
      }
    ]
  }
}
```

`cat >> ~/claude_activity.jsonl` と書くだけで、
stdin に流れてくる JSON をそのままファイルに追記できる。

## /hooks コマンドで設定する

`settings.json` を直接編集する以外に、Claude Code の `/hooks` コマンドから対話的に設定する方法もある。

Claude Code のチャット上で `/hooks` と入力すると設定フローが起動し、
イベントの選択からコマンドの登録まで対話的に行える。
設定内容は自動的に `settings.json` に反映される。

手動で JSON を書くよりもミスが少ないため、初めて Hooks を設定する場合はこちらから試すのがおすすめだ。

## 注意点

### 実行タイムアウト

Hook コマンドには実行タイムアウトがあり、デフォルトは60秒。
重い処理を Hook に含める場合は、バックグラウンド実行（`command &`）を検討する。

### セキュリティ

Hook は Claude ではなく**ユーザーの権限**で実行される。
Claude が Hook コマンドを変更することはできないが、
設定ファイル自体を編集するツールを Claude に許可している場合は
Hook 経由で意図しない処理が実行されるリスクがある点に注意が必要だ。

### 環境変数の展開

コマンド内で `$CLAUDE_TOOL_INPUT_FILE_PATH` のような変数を使う場合、
ダブルクォートで囲むことでスペースを含むパスに対応できる。

## まとめ

Claude Code の Hooks を使うと、Claude の実行フローに対してユーザー定義の処理を差し込める。

- **PreToolUse**: ツール実行前の検証・ブロックに使う
- **PostToolUse**: フォーマット・ログ記録・通知に使う
- **Stop**: 完了通知や後処理に使う

設定は `settings.json` に JSON で記述するか、`/hooks` コマンドから対話的に行う。
日常的な作業を自動化したり、危険な操作を防いだりと、応用の幅が広い機能なので積極的に活用してほしい。

## 参考

- [Claude Code Hooks（公式ドキュメント）](https://code.claude.com/docs/ja/hooks)
- [Claude Code の設定](https://code.claude.com/docs/ja/settings)
