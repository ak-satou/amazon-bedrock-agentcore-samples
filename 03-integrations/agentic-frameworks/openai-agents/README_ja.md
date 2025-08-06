# OpenAI Agents と Bedrock AgentCore の統合

| 項目                 | 詳細                                                                      |
|---------------------|---------------------------------------------------------------------------|
| エージェントタイプ      | 同期（ハンドオフあり/なし）                                                 |
| エージェントフレームワーク | OpenAI Agents SDK                                                        |
| LLMモデル           | GPT-4o                                                                   |
| コンポーネント        | AgentCore Runtime                                                        |
| サンプルの複雑さ      | 中級                                                                     |
| 使用SDK             | Amazon BedrockAgentCore Python SDK、OpenAI Agents SDK                   |

この例では、OpenAI Agents を AWS Bedrock AgentCore と統合し、特殊化されたタスクのためのエージェントハンドオフを紹介します。

## 前提条件

- Python 3.10以上
- [uv](https://github.com/astral-sh/uv) - 高速なPythonパッケージインストーラーおよびリゾルバー
- Bedrockアクセス権限のあるAWSアカウント
- OpenAI APIキー

## セットアップ手順

### 1. uvを使用してPython環境を作成

```bash
# まだuvをインストールしていない場合はインストール
pip install uv

# 仮想環境を作成してアクティベート
uv venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
```

### 2. 依存関係のインストール

```bash
uv pip install -r requirements.txt
```

## 例1: Hello World エージェント

`openai_agents_hello_world.py` ファイルには、ウェブ検索機能とBedrock AgentCore統合を持つシンプルなOpenAI エージェントが含まれています：

```python
from agents import Agent, Runner, WebSearchTool
import logging
import sys

# ログ設定
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[logging.StreamHandler(sys.stdout)]
)
logger = logging.getLogger("openai_agents")

# ウェブ検索ツールでエージェントを初期化
agent = Agent(
    name="Assistant",
    tools=[WebSearchTool()],
)

async def main(query=None):
    if query is None:
        query = "私の好みと今日のSFの天気を考慮して、どのコーヒーショップに行くべきでしょうか？"
    
    logger.debug(f"クエリでエージェントを実行中: {query}")
    result = await Runner.run(agent, query)
    return result

# Bedrock AgentCore との統合
from bedrock_agentcore.runtime import BedrockAgentCoreApp
app = BedrockAgentCoreApp()

@app.entrypoint
async def agent_invocation(payload, context):
    query = payload.get("prompt", "今日はどのようにお手伝いできますか？")
    result = await main(query)
    return {"result": result.final_output}

if __name__ == "__main__":
    app.run()
```

## 例2: エージェント ハンドオフ

`openai_agents_handoff_example.py` ファイルでは、特殊化された役割を持つエージェントシステムを作成し、互いにタスクをハンドオフする方法を示します：

```python
# 異なるタスク用の特殊化されたエージェントを作成
travel_agent = Agent(
    name="旅行専門家",
    instructions=(
        "あなたはユーザーの旅行計画をサポートする旅行専門家です。"
        "ウェブ検索を使用して、目的地、フライト、宿泊施設、"
        "旅行要件に関する最新情報を見つけてください。"
    ),
    tools=[WebSearchTool()]
)

food_agent = Agent(
    name="グルメ専門家",
    instructions=(
        "あなたはユーザーが素晴らしい食事のオプションを見つけるのをサポートするグルメ専門家です。"
        "ウェブ検索を使用して、レストラン、地元料理、"
        "グルメツアー、食事制限への対応に関する情報を見つけてください。"
    ),
    tools=[WebSearchTool()]
)

# 特殊化されたエージェントにハンドオフできるメインのトリアージエージェントを作成
triage_agent = Agent(
    name="旅行アシスタント",
    instructions=(
        "あなたは役立つ旅行アシスタントです。"
        "ユーザーが旅行計画、目的地、フライト、宿泊施設について質問した場合は、"
        "旅行専門家にハンドオフしてください。"
        "ユーザーが食事、レストラン、または食事のオプションについて質問した場合は、"
        "グルメ専門家にハンドオフしてください。"
    ),
    handoffs=[travel_agent, food_agent]
)
```

### ハンドオフの仕組み

1. トリアージエージェントが初期のユーザークエリを受信
2. クエリの内容に基づいて、どの特殊化されたエージェントがリクエストを処理すべきかを決定
3. 適切な特殊化されたエージェント（旅行専門家またはグルメ専門家）が引き継ぎ
4. 特殊化されたエージェントがツール（ウェブ検索）を使用して情報を収集
5. 最終的な応答がユーザーに返される

このパターンにより以下が可能になります：
- 異なるドメインでの特殊化された知識と行動
- エージェント間での関心事の明確な分離
- ドメイン固有のクエリに対するより正確で関連性の高い応答

## Bedrock AgentCore での設定と起動

```bash
# デプロイメント用にエージェントを設定
agentcore configure

# OpenAI APIキーを使用してエージェントをデプロイ
agentcore deploy --app-file openai_agents_handoff_example.py -l --env OPENAI_API_KEY=your_api_key_here
```

## エージェントのテスト

```bash
agentcore invoke --prompt "来月日本に旅行する予定です。何を知っておくべきでしょうか？"
```

システムは以下の処理を行います：
1. トリアージエージェントを通じてクエリを処理
2. 旅行専門家エージェントにハンドオフ
3. ウェブ検索を使用して日本旅行に関する情報を収集
4. 包括的な応答を返す

## 追加リソース

- [OpenAI Agents ドキュメンテーション](https://platform.openai.com/docs/assistants/overview)
- [Bedrock AgentCore ドキュメンテーション](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-core.html)