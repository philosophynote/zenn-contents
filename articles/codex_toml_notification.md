---
title: "Codexで通知音が鳴らない原因はTOML の仕様のせい"
emoji: "🔔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Codex"]
published: false
---

## TL;DR

* Codex の `notify` は **トップレベル（root）のキー**として読む前提。([OpenAI Developers][1])
* ところが TOML は **`[table]` を書いた後に root に戻れない**（＝root は先頭〜最初の table header まで）。([TOML][2])
* その結果、`notify` を `[tui]` などの後ろに置くと **`tui.notify` 扱い**になり、Codex が拾わず「通知が鳴らない」。
* Codex の公式サンプルにも「**Root keys must appear before tables in TOML**」と注意書きがある。([OpenAI Developers][3])

---

## 想定読者

* Codex を入れたばかり（CLI / IDE Extension）
* 「タスク完了時に音を鳴らしたい」ので `notify` を触っている
* `notify.py` を単体実行すると動くのに、Codex 経由だと動かず困っている

---

## 前提知識（最小）

* Codex の設定は `~/.codex/config.toml` にある。([OpenAI Developers][4])
* `notify` は Codex がイベント発火時に外部プログラムを起動する仕組み（現状イベントは `agent-turn-complete` のみ）。([OpenAI Developers][1])

---

## 起きていた症状

* `notify.py` をターミナルから手動実行すると音が鳴る
* でも Codex を動かしても **音が鳴らない / ログが出ない**
* つまり「`notify` が呼ばれていないのでは？」という状態

---

## 最小再現：この config だと鳴らない

ポイントは **`notify` を `[table]` の後ろに書いてしまうこと**。

```toml
[tui]
# ...何か設定...

notify = ["/opt/homebrew/bin/python3", "/Users/naokitakahashi/.codex/notify.py"]
```

* これ、TOML 的には「`notify` は `[tui]` の配下」になります（`tui.notify`）。
* Codex はトップレベルの `notify` を見に行くので、結果として **notify 未設定扱い**になります。([OpenAI Developers][1])

---

## 原因：TOML の “root table” の仕様

TOML では、

* **root table は文書の先頭から、最初の table header（`[xxx]`）の直前まで**
* root table は **名前がなく、途中に移動（＝戻って再開）できない** ([TOML][2])

つまり、「`[tui]` の後に `notify = ...` を書く」のは、
あなたの感覚では “グローバル設定” でも、TOML のルールでは **`tui` テーブルの設定**になります。

Codex 側もこの罠を踏む人が多い前提で、公式サンプルに明確な注意書きを入れています。([OpenAI Developers][3])

---

## 解決：`notify` を “最初の `[table]` より前” に移動する

```toml
# ✅ root に置く（最初の [table] より前）
notify = ["/opt/homebrew/bin/python3", "/Users/naokitakahashi/.codex/notify.py"]

[tui]
# ...何か設定...
```

これで Codex がトップレベルの `notify` を認識できるようになります。([OpenAI Developers][1])

---

## ついでに：初心者向けおすすめ `notify.py`（音＋ログ）

「呼ばれてるか？」を確実に確認できる形にしておくと、次回の切り分けが楽です。

```python
import sys
import subprocess
from datetime import datetime

LOG = "/Users/naokitakahashi/.codex/notify.log"

def main() -> int:
    with open(LOG, "a") as f:
        f.write(f"{datetime.now().isoformat()} notify called argv={sys.argv}\n")

    subprocess.run(["/usr/bin/afplay", "/System/Library/Sounds/Ping.aiff"], check=False)
    return 0

if __name__ == "__main__":
    raise SystemExit(main())
```

---

## 検証チェックリスト（ハマりどころを潰す）

### 1) まず config の場所は合ってる？

* `~/.codex/config.toml` を編集しているか（CLI と IDE 拡張は同じ config を共有）([OpenAI Developers][4])

### 2) `notify` は **最初の `[table]` より前**にある？

* 公式サンプルの注意書きを思い出す：root keys は tables より前 ([OpenAI Developers][3])

### 3) 一時的に `-c` で上書きして動作確認できる？

* Codex は `-c key=value` の上書きをサポート（その回だけ有効）([OpenAI Developers][5])

例（※ 例は雰囲気、手元の環境に合わせて調整）：

```bash
codex -c 'notify=["/opt/homebrew/bin/python3","/Users/naokitakahashi/.codex/notify.py"]' "test"
```

---

## なぜ「エラーにならず静かに失敗」するのか

* TOML としては **完全に正しい**（`[tui]` の中に `notify` があるだけ）
* Codex は「トップレベルの `notify`」しか見ない
  → なので **エラーではなく「未設定扱い」**になりやすい

---

## まとめ

* Codex の `notify` が鳴らないとき、「Codex が壊れてる」より先に **TOML の root table 位置**を疑う
* **root key は tables より前**（公式サンプルもそう言ってる）([OpenAI Developers][3])
* 初心者ほど、設定は「トップレベル設定 → `[tui]` → `[mcp]`…」の順で並べるのが安全

---

## 参考（一次情報）

* Codex Sample Configuration（Root keys must appear before tables in TOML）([OpenAI Developers][3])
* Codex Advanced Configuration（notify / agent-turn-complete）([OpenAI Developers][1])
* Codex Basic Configuration（config の場所：`~/.codex/config.toml`）([OpenAI Developers][4])
* TOML spec（root table の定義：先頭〜最初の table header、移動不可）([TOML][6])
* Codex CLI reference（`-c` 上書き）([OpenAI Developers][5])

[1]: https://developers.openai.com/codex/config-advanced/?utm_source=chatgpt.com "Advanced Configuration"
[2]: https://toml.io/en/v1.0.0?utm_source=chatgpt.com "TOML: English v1.0.0"
[3]: https://developers.openai.com/codex/config-sample?utm_source=chatgpt.com "Sample Configuration"
[4]: https://developers.openai.com/codex/config-basic/?utm_source=chatgpt.com "Basic Configuration"
[5]: https://developers.openai.com/codex/cli/reference/?utm_source=chatgpt.com "Command line options"
[6]: https://toml.io/en/v1.1.0?utm_source=chatgpt.com "TOML: English v1.1.0"
