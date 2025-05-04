---
title: "時代の進歩についていけないのでLLMを比較した"
emoji: "⚖️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["LLM","ChatGPT","Claude","Gemini"]
published: false
---

TL;DR

- **最高精度・複雑バグ修正**なら o3（71 % SWE‑bench） 
- **長文脈**は GPT‑4.1 系と Gemini 2.5 Pro（どちらも 1 M token）。
- **量産タスク**は GPT‑4.1 nano／mini、Gemini Flash、Claude Haiku が圧倒的な「安さ×速さ」。
- **ビジョンを含む実装補助**では GPT‑4o / 4o‑mini が依然トップの多機能さ。

## 1.LLMモデル一覧（2025年4月19日時点）



| モデル                                          | コンテキスト長 | 入力$/M    | 出力$/M               | SWE‑bench&nbsp;Verified*              | 備考                 | 
| ----------------------------------------------- | -------------- | ---------- | --------------------- | -------------------------------------- | -------------------- | 
| GPT‑4.1 | 1 M | 2.00 | 8.00 | 54.6 % | 高精度・長文脈
| GPT‑4.1&nbsp;mini                              | 1&nbsp;M       | 0.40       | 1.60       | [(公表無し) GPT‑4o 超え](https://openai.com/index/gpt-4-1/) | 4o より 83&nbsp;% 安 | 
| GPT‑4.1&nbsp;nano                              | 1&nbsp;M       | 0.10       | 0.40       | [(公表無し) 最速](https://openai.com/index/gpt-4-1/)             | ライト級最安         | 
| GPT‑4o                                         | 128&nbsp;k     | 3.00       | 9.00       | 33&nbsp;%&nbsp;(22‑11‑20)     | ビジョン強み         | 
| GPT‑4o‑mini                                   | 128&nbsp;k     | 1.10       | 4.40       | 45&nbsp;% (社内)              | 高速・安価           | 
| o3                                              | 200&nbsp;k     | 10.00      | 40.00                 | ≈71&nbsp;%                            | 精度トップ           | 
| o4‑mini                                        | 128&nbsp;k     | 2.00       | 4.40                  | ≈45&nbsp;%                            | 4o‑mini後継         | 
| GPT‑4.5 preview                | ~256 k&nbsp;(噂値)  | 75         | 150        | 38 %   | 7 月廃⽌予定  | 
| Claude&nbsp;3&nbsp;Opus                         | 200&nbsp;k     | 15.00      | 75.00                 | 49&nbsp;%                              | Anthropic最上位      | 
| Claude&nbsp;3.7&nbsp;Sonnet (Thinking&nbsp;ON)  | 200&nbsp;k     | 3.00       | 15.00                 | 62.3&nbsp;%                            | 思考時間可変         | 
| Claude&nbsp;3.7&nbsp;Sonnet (Thinking&nbsp;OFF) | 200&nbsp;k     | 3.00       | 15.00                 | 56&nbsp;% ±                            | 低レイテンシ         |
| Claude&nbsp;3.5&nbsp;Haiku                      | 200&nbsp;k     | 0.80       | 4.00                  | 40&nbsp;%                              | 速度最優先           | 
| Claude&nbsp;3&nbsp;Haiku                        | 200&nbsp;k     | 0.25       | 1.25                  | —                                     | 最安値帯             | 
| Gemini&nbsp;2.5&nbsp;Pro                        | 1&nbsp;M       | 1.25–2.50 | 10–15                | 63.8&nbsp;%                            | 長文脈 + 高コスパ    | 
| Gemini&nbsp;2.5&nbsp;Flash                      | 1&nbsp;M       | 0.15       | 0.60 / 3.50 (think)   | —                                     | 思考量調整可         | 
| Gemini&nbsp;2.0&nbsp;Flash                      | 1&nbsp;M       | 0.15       | 0.60                  | —                                     | 定番ライト級         | 




## 2. コーディング用途での実感

### 2.1 典型ユースケース別 ✕ 推奨モデル

| ユースケース | 推奨モデル | 理由 |
|--------------|-----------|------|
| **巨大 mono‑repo（>30 万行）を丸ごとレビュー** | **GPT‑4.1 / mini**<br>**Gemini 2.5 Pro** | 1 M token 入力可。4.1 mini はコスト 0.4 $/M で Pro より安価、Pro は 63.8 % SWE‑bench と精度が高い |
| **未知の複雑バグを 1 パスで修正** | **o3** | 71 % SWE‑benchで最高精度、思考深度を自動調整 |
| **リファクタ案ブレスト・設計相談** | **Claude 3.7 Sonnet (Thinking ON)** | 62 % SWE‑benchと低価格のバランス。思考モード切替でコストを動的制御 |
| **CI/CD で毎コミット静的検査・テスト生成** | **GPT‑4.1 nano**<br>**Gemini 2.5 Flash (think=0)** | nano は OpenAI 最速・最安 (0.10 $/M)。Flash は "thinking budget"0 にすると 0.15 $/M で高トラフィック対応 |
| **コード＋画像の組み合わせ（OCR、図付き設計書）** | **GPT‑4o / 4o‑mini** | GPT‑4o は視覚＋音声＋テキストの統合推論で、4o‑mini でも HumanEval 87 % と小型最強クラス |
| **高速チャット補完・コードスニペット生成** | **Claude 3 Haiku**<br>**Gemini 2.0 Flash** | 0.25 $/M と 0.60 $/M 出力で最安帯。Haiku は応答遅延が 100 ms 台、Flash は動的推論量でさらに高速 |
| **マイグレーション／生成 AI 実験用に幅広い API 機能を試したい** | **GPT‑4o‑mini** | Function Calling・Streaming・Vision が全て格安で利用可、HumanEval 87 % と実力も十分 |
| **一時的に「創造性」や「EQ」重視** | **GPT‑4.5 preview** | コーディング精度は 38 % 程度ながら表現力とマルチモーダルが強く、プレビュー期間内の PoC に◎（ただし 7 ⽉停⽌予定） |

### 2.2 モデル別 強みと弱み

| モデル群 | 強み | 弱み / 注意 |
|----------|------|-------------|
| **GPT‑4.1 family** | 1 M token 長文脈。mini/nano で圧倒的コスパ（0.4 / 0.1 $/M）。nano は MMLU 80 % と中堅精度 | nano/mini はツール呼び出し回数制限がやや厳しい |
| **o‑series (o3, o4‑mini)** | SWE‑bench最高クラスの推論精度。o4‑mini でも 128 k context と高速化 | 価格が高め (o3 入力 10 $/M) |
| **Claude 3.7 Sonnet / 3.5 Haiku** | Thinking ON で深い推論、OFF で高速。価格は 3 $/M 入力〜0.8 $/M まで段階的 | 出力トークン単価は GPT‑4.1 mini と近く、長文脈は 200 k 止まり |
| **Gemini 2.5 family** | 1 M token、Flash は thinking budget で速度/精度/コストを可変。Pro は 63.8 % SWE‑bench と高精度 | Vision が英語 UI 中心。Pro はレイテンシがやや長い |
| **GPT‑4o / 4o‑mini** | マルチモーダル実行 (画像・音声)。mini が HumanEval 87 % と小型最強クラス | 128 k context 上限。入力単価は mini 1.1 $/M と nano より高い |
| **GPT‑4.5 preview** | 話題性・創造的回答で高評価。Vision/Tool 系 API 完備 | 256 k context・高コスト・7 ⽉廃⽌予定で本番運用には不向き |

### 2.3 実用 Tips

1. **プロンプト分割＋RAG**  
   - 1 M token でも API 上限に近い入力は遅くなる。リポジトリを論理モジュール単位で分割し、RAG の要約を挟むと応答が安定。  
2. **キャッシュと Batch API**  
   - GPT‑4.1/Claude/Gemini いずれも Batch で 30–50 % 割引あり。重いテスト生成を夜間バッチに回すだけで大幅コスト減。  
3. **思考量コントロール**  
   - Claude Sonnet の `extended_thinking=true`、Gemini Flash の `thinking_budget` を動的に切り替え、失敗タスクだけ深く考えさせると ROI が高い。

---

## 3. LLM製品比較

### 最新LLM機能対応比較表（2025年5月時点）

| 機能                          | **OpenAI – ChatGPT** (GPT-3.5 / GPT-4 系列)                                                                                                                 | **Anthropic – Claude** (Claude 3 ファミリ)                                                                                                          | **Google – Gemini**                                                                                            |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| **Deep Research**(自動深層リサーチ) | **提供あり**：*ChatGPT Deep Research* を 2025-02 に導入。2025-04-24 時点の上限：**Pro 250 件 / 月・Plus 25 件 / 月**。複数クエリを自律生成し数十〜数百の情報源をクロール、引用付きレポートを返す。                    | **提供あり**：Claude の **Research** 機能。複数検索クエリを自動生成して統合回答。社内文書（Google Workspace 連携）も横断検索。                                                            | **提供あり**：Gemini 2.5 Pro にて「Thinking」機能を搭載。最大100万トークンの長文コンテキスト処理が可能で、複数ファイルを統合して解析。                             |
| **PDF サポート**(PDF 読み込み)      | **対応**：Advanced Data Analysis モードで PDF をアップロードして解析可能。1ファイルあたり最大512MBまで対応。1回のチャットで最大10ファイルまでアップロード可能。                                                      | **対応**：Claude 3.5 Sonnet および Claude 3.7 Sonnet では、100ページ以下のPDFに対してテキストと画像の両方を解析可能。1ファイルあたり最大30MB、1チャットあたり最大20ファイルまでアップロード可能。                    | **対応**：Gemini 2.5 Pro では、1ファイルあたり最大50MB、最大1,000ページのPDFを解析可能。1回のプロンプトで最大3,000ファイルまでアップロード可能。                    |
| **Artifacts (生成物分離 UI)**    | **未対応**：2025年5月時点でChatGPTには公式な「Artifacts」や生成物分離UI機能は未実装。生成物はチャット画面にインライン表示のみ。*（以前噂された "Canvas" 機能は公式発表なし・未提供）*。 | **対応**：2024年6月にClaude 3.5 Sonnetで「Artifacts」機能が導入。生成物（コード・図・文書等）をチャット履歴と独立したUIで管理・共有でき、URL公開やダウンロード、共同編集も可能。2025年5月には「Claude Code」など開発者向け拡張も追加。 | **対応**：2025年3月の大型アップデートで「Canvas」機能が追加。Artifacts同様、生成物を分離UIで管理・共有できる。Gemini 2.5 Proでは無料で利用可能、関数呼び出しや構造化出力、共同作業もサポート。 |
| **マルチモーダル対応**(画像・音声・動画処理)   | **対応**：GPT-4o にて画像・音声入出力に対応。音声合成やリアルタイム会話も可能。                                                                                                             | **対応**：Claude 3 Opus などで画像入力に対応。音声対応は限定的。                                                                                                       | **対応**：Gemini 2.5 Pro でマルチモーダル対応強化。Gemini Live によりリアルタイムカメラ入力・画面共有が可能。                                         |
| **モデル切替**(複数モデル選択)          | **対応**：GPT-3.5 / GPT-4 / GPT-4o を手動で切替可能。                                                                                                                 | **対応**：Claude 3 Haiku / Sonnet / Opus を選択可能。                                                                                                    | **対応**：Gemini 1.5 Flash / Gemini 1.5 Pro などを用途に応じて選択可能。                                                        |
| **拡張機能・プラグイン**              | **対応**：GPTs・Browse・Code Interpreter などのツールを統合。外部API連携も可能。                                                                                                 | **限定対応**：社内検索やカスタム指示などに対応するが、外部プラグイン対応は限定的。                                                                                                     | **一部対応**：Google Workspace・NotebookLM・Gemini Live を通じたアシスタント統合あり。                                               |
| **バッチ処理**(一括非同期推論)          | **対応**：OpenAI Batch API を利用可能。複数リクエストを非同期で最大24時間以内に処理。通常の API より最大 50% 割安。GPT-3.5/4/4o で利用可。                                                              | **対応**：Claude Batch API により最大10万件まで一括非同期処理が可能。料金も割安で、Claude 3.5/3.7 などが対応。                                                                      | **対応**：Google Vertex AI 上で Gemini バッチ推論が可能。BigQuery や Cloud Storage 入力を一括処理。API コストは最大 50% 割引。                 |
| **ブラウザ操作**(GUI ナビゲーション)     | **対応**：ChatGPT Pro（月額200ドル）ユーザー向けに「Operator」機能を提供。AI が仮想ブラウザを操作し、フォーム入力・クリック・ページ遷移を自動化。研究プレビュー段階で一部の国から提供開始。                                              | **対応**：Claude 3.5+ の Computer Use 機能で仮想ブラウザを操作可能。リンククリックやスクロール、フォーム入力などを自動化。                                                                    | **未対応**：Gemini はウェブ検索は可能だが、GUI 上のフォーム操作は非対応。スマホではアプリ起動や情報カード操作に一部対応。                                           |
| **MCP対応**(外部ツール連携)          | **対応予定**：OpenAIはMCPへの対応を進めており、Agents SDKやChatGPTデスクトップアプリでのサポートを予定しています。詳細は今後数ヶ月以内に発表される予定です。                                                             | **対応**：ClaudeはMCPを通じて、Jira、Confluence、Zapier、Cloudflare、Intercom、Asana、Square、Sentry、PayPal、Linear、Plaidなどの外部ツールと連携可能な「Integrations」機能を提供しています。 | **対応**：GeminiはMCPをサポートしており、Gemini 2.5 ProではMCPサーバーを通じて外部ツールとの連携が可能です。                                          |
| **その他の行**                   | **モバイルアプリ対応、音声認識、WEB検索、文章編集、データ分析、画像説明、画像生成、動画生成、コード支援、ブラウザ操作、MCP対応、Google/Microsoft連携、バッチ処理などにそれぞれ差異あり。Claude は MCP やバッチ処理に強く、ChatGPT は解析・プラグイン拡張に優れる。** | **MCP、Artifacts、バッチ処理、Google Workspace連携に強み。音声出力は未実装。**                                                                                         | **Gemini 2.5 Pro により Thinking 機能・マルチモーダル強化・NotebookLM 統合・Live 機能（画面共有等）などが追加。Google連携機能が最も豊富で、動画生成や画像生成にも対応。** |

---

### 変更履歴

* **2025-05-04**  Deep Research の月間上限を追記。ChatGPT の PDF 上限値を「要確認」に訂正。Gemini の全体記述を 2.5 Pro 基準で更新。バッチ処理・ブラウザ操作・MCP対応の行を追加。
* **2025-04-xx**  初版作成。
