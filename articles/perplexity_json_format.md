---
title: "Sonar APIの結果出力をJSON形式で取得する"
emoji: "🔍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["perplexity","LangChain","LLM","Sonar"]
published: false
---

## はじめに

Perplexityから新モデルとしてSonarとSonar Proがリリースされました。

https://www.perplexity.ai/ja/hub/blog/introducing-the-sonar-pro-api

Perplexityは以前からAPIを提供していましたが、
精度が良くないとの声が多く、触っていませんでした。
今回Sonarがリリースされたので
試してみることにしました。

## 試してみる

公式ドキュメントを元に試してみます。[^1]

https://docs.perplexity.ai/api-reference/chat-completions

[^1]: OpenAI公式のライブラリ経由でも使用できます。https://docs.perplexity.ai/guides/getting-started#make-your-api-call

```python

import requests
from dotenv import load_dotenv
import os
import json

load_dotenv()

PERPLEXITY_API_URL = "https://api.perplexity.ai/chat/completions"
PERPLEXITY_API_KEY = os.environ["PERPLEXITY_API_KEY"]

system_prompt = """
  You operate as a “Horse Racing Information Assistant”. 
  Your main task is to collect user-supported information on the web about your target races.
  Answer in Japanese.
"""

your_prompt = """
2025年2月23日に実施されるフェブラリーSについて

上のレースにはガイアフォースという馬が出走します。
ガイアフォースの次の情報を教えてください。
- 馬名
- レースの馬番
- 騎手名
- 調教師名
- ポジティブな意見 (リスト形式)
- ネガティブな意見 (リスト形式)
"""

payload = {
    "model": "sonar-pro",
    "messages": [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": your_prompt},
    ],
    "max_tokens": 4096,
    "temperature": 0.2,
}
headers = {
    "Authorization": f"Bearer {PERPLEXITY_API_KEY}",
    "Content-Type": "application/json",
}


response = requests.request("POST", PERPLEXITY_API_URL, json=payload, headers=headers)
body = json.loads(response.text)
```

回答の文章は次のようになりました。

```python
body["choices"][0]["message"]["content"]

'ガイアフォースの情報は以下の通りです：\n\n- 馬名: ガイアフォース\n- レースの馬番: 15番[4]\n- 騎手名: 長岡 禎仁[5]\n- 調教師名: 杉山晴紀（栗東）[2][5]\n\nポジティブな意見:\n- 昨年のフェブラリーステークスで2着と好成績を残している[3]\n- 東京のマイルコースが合うとされている[9]\n- 追い切りでは躍動感のある動きを見せている[1]\n- 調教師は馬の状態に自信を持っている[1]\n- 外枠を引いたことで囲まれることがなく、有利と考えられている[3]\n\nネガティブな意見:\n- 初速が遅く、前に行きづらい傾向がある[9]\n- 6歳馬となり、年齢的な衰えの可能性がある\n- 昨年と異なる調整過程を取っているため、仕上がりに不安がある[10]\n- 外枠を引いたことで、スタート後のポジション取りが難しくなる可能性がある'
```

引用元も取得できます。

```python
body["citations"]

['https://www.radionikkei.jp/keiba_article/news/c_215.html',
 'https://umanity.jp/sp/racedata/db/horse_top.php?code=2019104476',
 'https://umatoku.hochi.co.jp/articles/20250222-OHT1T51240.html',
 'https://s.keibabook.co.jp/cyuou/odds/5/202501040811',
 'https://www.keibalab.jp/db/horse/2019104476/',
 'https://db.netkeiba.com/horse/ped/2019104476/',
 'https://www.sponichi.co.jp/gamble/news/2025/02/23/kiji/20250222s00004048383000c.html',
 'https://s.keibabook.co.jp/cyuou/seiseki/202501040811',
 'https://www.radionikkei.jp/keiba_article/news/s_600.html',
 'https://umanity.jp/racedata/race_newsdet.php?nid=11841697']
```

## LangChainとの連携

アプリケーションに組み込むことを考えると
JSON形式で取得したいです。

PerplexityのAPIのドキュメントを読むと`response_format`を指定することで
JSON形式で取得できると記載されていましたが、
Tier-3以上のメンバーしか利用できないとのことでした。

https://docs.perplexity.ai/guides/structured-outputs

Tier-3以上のメンバーになるためには
$500の課金が必要です。

https://docs.perplexity.ai/guides/usage-tiers

JSON形式の出力のためだけに課金するのは少し高いと感じたため、
LangChainを利用して、
Sonar APIの結果をJSON形式で取得することを試みます。

LangChainには`ChatPerplexity`クラスがあるのでこれを活用します。

https://python.langchain.com/docs/integrations/chat/perplexity/

https://python.langchain.com/api_reference/community/chat_models/langchain_community.chat_models.perplexity.ChatPerplexity.html

実際に動かしたコードは次のようになりました。

```python
from pydantic import BaseModel, Field
from langchain_community.chat_models import ChatPerplexity
from langchain.prompts import PromptTemplate
from langchain_core.output_parsers import PydanticOutputParser
from typing import List

class HorseInfo(BaseModel):
    horse_name: str = Field(..., description="馬名")
    horse_number: int = Field(..., description="レースの馬番")
    jockey: str = Field(..., description="騎手名")
    trainer: str = Field(..., description="調教師名")
    positive_opinions: List[str] = Field(..., description="ポジティブな意見")
    negative_opinions: List[str] = Field(..., description="ネガティブな意見")

prompt_template = """
レース日: {race_date}
レース名: {race_name}

上のレースに出走するガイアフォースの次の情報を教えてください。
- 馬名
- レースの馬番
- 騎手名
- 調教師名
- ポジティブな意見 (リスト形式)
- ネガティブな意見 (リスト形式)

JSON形式で返却してください。
keyとvalueの組み合わせは以下の通りです。

"horse_name": 馬名,
"horse_number": レースの馬番,
"jockey": "騎手名",
"trainer": "調教師名",
"positive_opinions": ["ポジティブな意見1", "ポジティブな意見2"],
"negative_opinions": ["ネガティブな意見1", "ネガティブな意見2"],

"""


pydantic_parser = PydanticOutputParser(pydantic_object=HorseInfo)

prompt = PromptTemplate(
    template=prompt_template,
    input_variables=["race_date", "race_name"],
    partial_variables={
        "format_instructions": pydantic_parser.get_format_instructions()
    },
)

llm = ChatPerplexity(
    model="sonar-pro",
    temperature=0.2,
    pplx_api_key=PERPLEXITY_API_KEY,
    max_tokens=4096,
)

chain = prompt | llm 

inputs = {
    "race_date": "2025-02-23",
    "race_name": "フェブラリーS",
}

response = chain.invoke(inputs)

# AIMessage(content='{\n  "horse_name": "ガイアフォース",\n  "horse_number": 14,\n  "jockey": "長岡禎仁",\n  "trainer": "杉山晴紀",\n  "positive_opinions": [\n    "追い切りで躍動感のある動きを見せた[1]",\n    "昨年の2着馬で、リベンジに挑む[5]",\n    "具合が良く、寒い時期の歩様の硬さもない[5]",\n    "調教師は展開さえ噛み合えば十分勝ち負けになると考えている[3]",\n    "東京のマイルコースが合う[3]"\n  ],\n  "negative_opinions": [\n    "初速がなかなかつかない馬で、位置取りは前に行けない可能性がある[3]",\n    "勝ち星からはかなり離れている[1]"\n  ]\n}', additional_kwargs={'citations': ['https://www.radionikkei.jp/keiba_article/news/c_215.html', 'https://www.keibalab.jp/db/race/202502230511/syutsuba.html', 'https://www.radionikkei.jp/keiba_article/news/s_600.html', 'https://s.keibabook.co.jp/cyuou/odds/4/202501040811', 'https://hochi.news/articles/20250222-OHT1T51240.html', 'https://race.netkeiba.com/special/index.html?id=0019', 'https://db.netkeiba.com/horse/2019104476/', 'https://ja.wikipedia.org/wiki/%E3%82%AC%E3%82%A4%E3%82%A2%E3%83%95%E3%82%A9%E3%83%BC%E3%82%B9', 'https://race.netkeiba.com/race/shutuba.html?race_id=202505010811', 'https://www.youtube.com/watch?v=JqIirPcSPyY']}, response_metadata={}, id='run-781d7321-b706-4c62-a816-c19073b5a8c8-0')
```

contentの内部はjson形式の文字列になっているので`json.loads`で変換します。

```python
json.loads(response.content)
```

```json
{'horse_name': 'ガイアフォース',
 'horse_number': 15,
 'jockey': '長岡禎仁',
 'trainer': '杉山晴紀',
 'positive_opinions': ['前年のフェブラリーステークスで好走しており、東京のマイルコースが合う[4]',
  '調教の動きが良く、躍動感のある動きを見せている[4]',
  'ストライドが大きくなっており、状態が良い[4]',
  '展開さえ噛み合えば十分勝ち負けになると調教師が評価している[4]'],
 'negative_opinions': ['初速がつかない馬で、位置取りが前に行きづらい[4]', 'ダート適性は完全には証明されていない[4]']}
```

`pydantic_parser`で定義したjson形式になっています。
引用元は次のように取得できます。

```python
response.additional_kwargs["citations"]

['https://www.radionikkei.jp/keiba_article/news/c_215.html',
 'https://www.keibalab.jp/db/race/202502230511/syutsuba.html',
 'https://www.radionikkei.jp/keiba_article/news/s_600.html',
 'https://s.keibabook.co.jp/cyuou/odds/4/202501040811',
 'https://hochi.news/articles/20250222-OHT1T51240.html',
 'https://race.netkeiba.com/special/index.html?id=0019',
 'https://db.netkeiba.com/horse/2019104476/',
 'https://ja.wikipedia.org/wiki/%E3%82%AC%E3%82%A4%E3%82%A2%E3%83%95%E3%82%A9%E3%83%BC%E3%82%B9',
 'https://race.netkeiba.com/race/shutuba.html?race_id=202505010811',
 'https://www.youtube.com/watch?v=JqIirPcSPyY']
```

## 注意点

### ・output_parserを使用すると引用元が取得できない

PydanticOutputParserをそのまま利用すると、レスポンスのmodel_extraに格納された引用元情報が取得できません。
引用元情報を確実に取得するには、with_structured_outputメソッドを利用する必要があります。
この場合は次のように設定してください。

```python
llm = ChatPerplexity(
    model="sonar-pro",
    temperature=0.2,
    pplx_api_key=PERPLEXITY_API_KEY,
    max_tokens=4096,
).with_structured_output(schema=HorseInfo)
```

しかしながら、この方法だと`response`の`model_extra dict`に格納されている引用元が取得できません。

https://github.com/langchain-ai/langchain/issues/28108

### ・バージョン管理

ChatPerplexityクラスが`with_structured_output`に対応したのは
langchain-communityの0.3.18で最新版になります。

https://github.com/langchain-ai/langchain/issues/29357

LangChainと組み合わせる際にはバージョンに気をつけましょう。

## 参考

- [[LangChain] with_structured_output を使用して、Pydanticのクラスをレスポンスとして受け取る](https://zenn.dev/pharmax/articles/8ed156e9ec9a68)
- [LangChain `with_structured_output` メソッドによる構造化データ抽出](https://zenn.dev/ml_bear/articles/cb07549ec52175)