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
今回Sonarがリリースされたことで、
生成AI検索エンジンのAPIを実行できるようになりました。

## 試してみる

早速

```python
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

```python
body["choices"][0]["message"]["content"]

'ガイアフォースについての情報は以下の通りです:\n\n- 馬名: ガイアフォース\n- レースの馬番: 1番\n- 騎手名: 長岡騎手\n- 調教師名: 杉山晴紀調教師\n\nポジティブな意見:\n- 昨年のフェブラリーステークスで2着と好成績を残している[1][8]\n- 東京コースでは4戦してすべて掲示板に乗る安定感がある[4]\n- 芝・ダートを問わずG1レースのワンターンマイルコースの巧者[8]\n- 調教の動きが良く、仕上がりに関して調教師が満足している[3]\n- 骨瘤の影響は全くなく、状態は前走と変わらない[3]\n\nネガティブな意見:\n- 最後に勝ったのが3歳のセントライト記念で、勝ち星から遠ざかっている[6]\n- 前走のチャンピオンズカップは4コーナーを回る器用さが求められるコースで、半年ぶりの故障明けだった[8]\n- 初速がなかなかつかない馬で、位置取りは前に行きづらい[3]'
```

```python
body["citations"]

['https://s.keibabook.co.jp/cyuou/odds/4/202501040811',
 'https://www.radionikkei.jp/keiba_article/news/c_215.html',
 'https://www.radionikkei.jp/keiba_article/news/s_600.html',
 'https://www.sportingnews.com/jp/horse-racing/news/jra-2025-february-stakes-prediction/590c5b90c1a17a14eca110b5',
 'https://spaia-keiba.com/news/detail/30812',
 'https://www.radionikkei.jp/keiba_article/news/gi_14.html',
 'https://news.netkeiba.com/?pid=news_view&no=289839',
 'https://blog.goo.ne.jp/tokuchan_mht/e/b6fd2f5538f86d0487805797be62ebcf?fm=rss',
 'https://www.keibalab.jp/db/race/202502230511/raceresult.html',
 'https://db.netkeiba.com/horse/2019104476/']
```

## LangChainとの連携

アプリケーションに組み込む際には
JSON形式で取得したいと考えました。

perplexityのAPIのドキュメントを読むと`response_format`を指定することで
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

実際に作成したPythonコードです

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

### output_parserを使用すると引用元が取得できない

Pydanticを使用してJSONレスポンスの形式を固定化する際は
次のように`with_structured_output`を使用して記述すべきです

```python
llm = ChatPerplexity(
    model="sonar-pro",
    temperature=0.2,
    pplx_api_key=PERPLEXITY_API_KEY,
    max_tokens=4096,
).with_structured_output(schema=HorseInfo)
```

しかしながら、引用元の取得が`response`のmodel_extra dictに格納されているため、output_parserを通していません
https://github.com/langchain-ai/langchain/issues/28108

また、今回は関係ありませんが、
ChatPerplexityクラスが`with_structured_output`に対応したのは
langchain-communityの0.3.18で最新版になります

https://github.com/langchain-ai/langchain/issues/29357

バージョン管理には気をつけましょう

## 参考

- [[LangChain] with_structured_output を使用して、Pydanticのクラスをレスポンスとして受け取る](https://zenn.dev/pharmax/articles/8ed156e9ec9a68)
- [LangChain `with_structured_output` メソッドによる構造化データ抽出](https://zenn.dev/ml_bear/articles/cb07549ec52175)
- [LangChainのOutput Parserを試す](https://zenn.dev/kun432/scraps/6954f9d07316cb)