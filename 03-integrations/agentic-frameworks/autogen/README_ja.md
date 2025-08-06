# AutoGen Agent と Bedrock AgentCore の統合

| 項目                 | 詳細                                                                      |
|---------------------|---------------------------------------------------------------------------|
| エージェントタイプ      | 同期                                                                     |
| エージェントフレームワーク | AutoGen                                                                  |
| LLMモデル           | OpenAI GPT-4o                                                            |
| コンポーネント        | AgentCore Runtime                                                        |
| サンプルの複雑さ      | 簡単                                                                     |
| 使用SDK             | Amazon BedrockAgentCore Python SDK                                       |

この例では、AutoGen エージェントを AWS Bedrock AgentCore と統合し、ツール使用機能を持つ対話型エージェントをマネージドサービスとしてデプロイする方法を示します。

## 前提条件

- Python 3.10以上
- [uv](https://github.com/astral-sh/uv) - 高速なPythonパッケージインストーラーおよびリゾルバー
- Bedrockアクセス権限のあるAWSアカウント
- OpenAI APIキー（モデルクライアント用）

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

### 3. エージェントコードの理解

`autogen_agent_hello_world.py` ファイルには、天気情報ツール機能を持つAutoGen エージェントがBedrock AgentCoreと統合されています：

```python
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.ui import Console
from autogen_ext.models.openai import OpenAIChatCompletionClient
import asyncio
import logging

# ログ設定
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger("autogen_agent")

# モデルクライアントの初期化
model_client = OpenAIChatCompletionClient(
    model="gpt-4o",
)

# エージェントが使用できるシンプルな関数ツールを定義
async def get_weather(city: str) -> str:
    """指定された都市の天気を取得する。"""
    return f"{city}の天気は73度で晴れです。"

# モデルとツールを持つAssistantAgentを定義
agent = AssistantAgent(
    name="weather_agent",
    model_client=model_client,
    tools=[get_weather],
    system_message="あなたは役立つアシスタントです。",
    reflect_on_tool_use=True,
    model_client_stream=True,  # ストリーミングトークンを有効化
)

# Bedrock AgentCore との統合
from bedrock_agentcore.runtime import BedrockAgentCoreApp
app = BedrockAgentCoreApp()

@app.entrypoint
async def main(payload):
    # ユーザープロンプトを処理
    prompt = payload.get("prompt", "こんにちは！何かお手伝いできることはありますか？")
    
    # エージェントを実行
    result = await Console(agent.run_stream(task=prompt))
    
    # JSON シリアライゼーション用にレスポンスコンテンツを抽出
    if result and hasattr(result, 'messages') and result.messages:
        last_message = result.messages[-1]
        if hasattr(last_message, 'content'):
            return {"result": last_message.content}
    
    return {"result": "応答が生成されませんでした"}

app.run()
```

### 4. Bedrock AgentCore Toolkitでの設定と起動

```bash
# デプロイメント用にエージェントを設定
agentcore configure -e autogen_agent_hello_world.py

# OpenAI APIキーを使用してエージェントをデプロイ
agentcore launch --env OPENAI_API_KEY=...
```

### 5. エージェントのテスト

ローカルテスト用の起動：
```bash
agentcore launch -l --env OPENAI_API_KEY=...
```

エージェントの呼び出し：
```bash
agentcore invoke -l '{"prompt": "NYCの天気はどうですか？"}'
```

エージェントは以下の処理を行います：
1. クエリを処理
2. 適切な場合は天気ツールを使用
3. ツールの出力に基づいて応答を提供

> 注意：クラウドでの起動・呼び出しには `-l` フラグを削除してください

## 動作原理

このエージェントは、AutoGenのエージェントフレームワークを使用して、以下の機能を持つアシスタントを作成します：

1. 自然言語クエリの処理
2. クエリに基づいてツールを使用するタイミングの判断
3. ツールの実行と結果の応答への組み込み
4. リアルタイムでの応答ストリーミング

エージェントはBedrock AgentCoreフレームワークでラップされており、以下を処理します：
- AWSへのデプロイメント
- スケーリングと管理
- リクエスト/レスポンスの処理
- 環境変数の管理

## 追加リソース

- [AutoGen ドキュメンテーション](https://microsoft.github.io/autogen/)
- [Bedrock AgentCore ドキュメンテーション](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-core.html)
- [OpenAI API ドキュメンテーション](https://platform.openai.com/docs/api-reference)