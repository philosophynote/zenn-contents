---
title: "時代の進歩についていけないのでLLMを比較した"
emoji: "⚖️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["LLM","ChatGPT","Claude","Gemini"]
published: false
---

TL;DR

- **最高精度・複雑バグ修正**なら o3（71 % SWE‑bench） 
- **長文脈（⾼速ストリーミング）**は GPT‑4.1 系と Gemini 2.5 Pro（どちらも 1 M token）citeturn0search3turn0search6。  
- **量産タスク**は GPT‑4.1 nano／mini、Gemini Flash、Claude Haiku が圧倒的な「安さ×速さ」citeturn0search3turn0search6turn0search7。  
- **ビジョンを含む実装補助**では GPT‑4o / 4o‑mini が依然トップの多機能さciteturn0search5turn1search3。  
以下では「コーディング用途での実感」をユースケース別に整理します（GPT 系列に限定せず 16 モデル横断）。

## 商用提供中のLLMモデル一覧（2025年4月19日時点）



| モデル                                          | コンテキスト長 | 入力$/M    | 出力$/M               | SWE‑bench&nbsp;Verified*              | 備考                 | 
| ----------------------------------------------- | -------------- | ---------- | --------------------- | -------------------------------------- | -------------------- | 
| GPT‑4.1 | 1 M | 2.00 | 8.00 | 54.6 % | 高精度・長文脈
| GPT‑4.1&nbsp;mini                              | 1&nbsp;M       | 0.40       | 1.60       | [(公表無し) GPT‑4o 超え](https://openai.com/index/gpt-4-1/) | 4o より 83&nbsp;% 安 | 
| GPT‑4.1&nbsp;nano                              | 1&nbsp;M       | 0.10       | 0.40       | [(公表無し) “最速”](https://openai.com/index/gpt-4-1/)             | ライト級最安         | 
| GPT‑4o                                         | 128&nbsp;k     | 3.00       | 9.00       | 33&nbsp;%&nbsp;(22‑11‑20)     | ビジョン強み         | 
| GPT‑4o‑mini                                   | 128&nbsp;k     | 1.10       | 4.40       | 45&nbsp;% (社内)              | 高速・安価           | 
| o3                                              | 200&nbsp;k     | 10.00      | 40.00                 | ≈71&nbsp;%                            | 精度トップ           | 
| o4‑mini                                        | 128&nbsp;k     | 2.00       | 4.40                  | ≈45&nbsp;%                            | 4o‑mini後継         | 
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
| **巨大 mono‑repo（>30 万行）を丸ごとレビュー** | **GPT‑4.1 / mini**<br>**Gemini 2.5 Pro** | 1 M token 入力可。4.1 mini はコスト 0.4 $/M で Pro より安価、Pro は 63.8 % SWE‑bench と精度が高いciteturn0search3turn0search6 |
| **未知の複雑バグを 1 パスで修正** | **o3** | 71 % SWE‑benchで最高精度、思考深度を自動調整citeturn1search10 |
| **リファクタ案ブレスト・設計相談** | **Claude 3.7 Sonnet (Thinking ON)** | 62 % SWE‑benchと低価格のバランス。思考モード切替でコストを動的制御citeturn0search1 |
| **CI/CD で毎コミット静的検査・テスト生成** | **GPT‑4.1 nano**<br>**Gemini 2.5 Flash (think=0)** | nano は OpenAI 最速・最安 (0.10 $/M)citeturn0search3。Flash は “thinking budget”0 にすると 0.15 $/M で高トラフィック対応citeturn0search6 |
| **コード＋画像の組み合わせ（OCR、図付き設計書）** | **GPT‑4o / 4o‑mini** | GPT‑4o は視覚＋音声＋テキストの統合推論で、4o‑mini でも HumanEval 87 % と小型最強クラスciteturn0search5turn1search3 |
| **高速チャット補完・コードスニペット生成** | **Claude 3 Haiku**<br>**Gemini 2.0 Flash** | 0.25 $/M と 0.60 $/M 出力で最安帯。Haiku は応答遅延が 100 ms 台、Flash は動的推論量でさらに高速citeturn0search7turn0search6 |
| **マイグレーション／生成 AI 実験用に幅広い API 機能を試したい** | **GPT‑4o‑mini** | Function Calling・Streaming・Vision が全て格安で利用可、HumanEval 87 % と実力も十分citeturn1search3 |
| **一時的に「創造性」や「EQ」重視** | **GPT‑4.5 preview** | コーディング精度は 38 % 程度ながら表現力とマルチモーダルが強く、プレビュー期間内の PoC に◎（ただし 7 ⽉停⽌予定）citeturn0search4turn2search0 |

### 2.2 モデル別 強みと弱み

| モデル群 | 強み | 弱み / 注意 |
|----------|------|-------------|
| **GPT‑4.1 family** | 1 M token 長文脈。mini/nano で圧倒的コスパ（0.4 / 0.1 $/M）。nano は MMLU 80 % と中堅精度citeturn0news75turn0search3 | nano/mini はツール呼び出し回数制限がやや厳しい |
| **o‑series (o3, o4‑mini)** | SWE‑bench最高クラスの推論精度。o4‑mini でも 128 k context と高速化citeturn0search0 | 価格が高め (o3 入力 10 $/M) |
| **Claude 3.7 Sonnet / 3.5 Haiku** | Thinking ON で深い推論、OFF で高速。価格は 3 $/M 入力〜0.8 $/M まで段階的citeturn0search1turn0search7 | 出力トークン単価は GPT‑4.1 mini と近く、長文脈は 200 k 止まり |
| **Gemini 2.5 family** | 1 M token、Flash は thinking budget で速度/精度/コストを可変。Pro は 63.8 % SWE‑bench と高精度citeturn0search6 | Vision が英語 UI 中心。Pro はレイテンシがやや長い |
| **GPT‑4o / 4o‑mini** | マルチモーダル実行 (画像・音声)。mini が HumanEval 87 % と小型最強クラスciteturn1search3turn0search5 | 128 k context 上限。入力単価は mini 1.1 $/M と nano より高い |
| **GPT‑4.5 preview** | 話題性・創造的回答で高評価。Vision/Tool 系 API 完備citeturn0search4 | 256 k context・高コスト・7 ⽉廃⽌予定で本番運用には不向き |

### 2.3 実用 Tips

1. **プロンプト分割＋RAG**  
   - 1 M token でも API 上限に近い入力は遅くなる。リポジトリを論理モジュール単位で分割し、RAG の要約を挟むと応答が安定。  
2. **キャッシュと Batch API**  
   - GPT‑4.1/Claude/Gemini いずれも Batch で 30–50 % 割引あり。重いテスト生成を夜間バッチに回すだけで大幅コスト減citeturn0search7。  
3. **思考量コントロール**  
   - Claude Sonnet の `extended_thinking=true`、Gemini Flash の `thinking_budget` を動的に切り替え、失敗タスクだけ深く考えさせると ROI が高いciteturn0search1turn0search6。  

---

