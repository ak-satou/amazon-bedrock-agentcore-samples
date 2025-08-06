# ADK Agent と Bedrock AgentCore の統合

| 項目                 | 詳細                                                                      |
|---------------------|---------------------------------------------------------------------------|
| エージェントタイプ      | 同期                                                                     |
| エージェントフレームワーク | Google ADK                                                               |
| LLMモデル           | Gemini 2.0 Flash                                                        |
| コンポーネント        | AgentCore Runtime                                                        |
| サンプルの複雑さ      | 簡単                                                                     |
| 使用SDK             | Amazon BedrockAgentCore Python SDK                                       |

この例では、Google ADK エージェントを AWS Bedrock AgentCore と統合し、検索機能を持つエージェントをマネージドサービスとしてデプロイする方法を示します。

## 前提条件

- Python 3.10以上
- [uv](https://github.com/astral-sh/uv) - 高速なPythonパッケージインストーラーおよびリゾルバー
- Bedrockアクセス権限のあるAWSアカウント
- Google AI APIキー（Geminiモデル用）

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

`adk_agent_google_search.py` ファイルには、Google検索機能を持つGoogle ADKエージェントがBedrock AgentCoreと統合されています：

```python
from google.adk.agents import Agent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.tools import google_search
from google.genai import types
import asyncio

# エージェント定義
root_agent = Agent(
    model="gemini-2.0-flash", 
    name="openai_agent",
    description="Google検索を使用して質問に答えるエージェント。",
    instruction="インターネット検索で質問にお答えします。何でもお聞きください！",
    # google_searchは、エージェントがGoogle検索を実行できる事前構築ツール
    tools=[google_search]
)

# セッションとランナー
async def setup_session_and_runner(user_id, session_id):
    session_service = InMemorySessionService()
    session = await session_service.create_session(app_name=APP_NAME, user_id=user_id, session_id=session_id)
    runner = Runner(agent=root_agent, app_name=APP_NAME, session_service=session_service)
    return session, runner

# エージェントとの対話
async def call_agent_async(query, user_id, session_id):
    content = types.Content(role='user', parts=[types.Part(text=query)])
    session, runner = await setup_session_and_runner(user_id, session_id)
    events = runner.run_async(user_id=user_id, session_id=session_id, new_message=content)

    async for event in events:
        if event.is_final_response():
            final_response = event.content.parts[0].text
            print("エージェントの応答: ", final_response)
    
    return final_response

# Bedrock AgentCore との統合
from bedrock_agentcore.runtime import BedrockAgentCoreApp
app = BedrockAgentCoreApp()

@app.entrypoint
def agent_invocation(payload, context):
    return asyncio.run(call_agent_async(
        payload.get("prompt", "Bedrock Agentcore Runtimeとは何ですか？"), 
        payload.get("user_id", "user1234"), 
        context.session_id
    ))

app.run()
```

### 4. Bedrock AgentCore Toolkitでの設定と起動

```bash
# デプロイメント用にエージェントを設定
agentcore configure -e adk_agent_google_search.py

# Gemini APIキーを使用してエージェントをデプロイ
agentcore launch --env GEMINI_API_KEY=your_api_key_here
```

### 5. エージェントのローカルテスト

ローカルテスト用の起動：
```bash
agentcore launch -l --env GEMINI_API_KEY=your_api_key_here
```

エージェントの呼び出し：
```bash
agentcore invoke -l '{"prompt": "Amazon Bedrock Agentcore Runtimeとは何ですか？"}'
```

エージェントは以下の処理を行います：
1. クエリを処理
2. Google検索を使用して関連情報を検索
3. 検索結果に基づいて包括的な回答を提供

> 注意：クラウドでの起動・呼び出しには `-l` フラグを削除してください

## 動作原理

このエージェントは、Google の ADK（Agent Development Kit）フレームワークを使用して、以下の機能を持つアシスタントを作成します：

1. 自然言語クエリの処理
2. 関連情報を見つけるためのGoogle検索の実行
3. 検索結果を一貫性のある回答に組み込む
4. 対話間でのセッション状態の維持

エージェントはBedrock AgentCoreフレームワークでラップされており、以下を処理します：
- AWSへのデプロイメント
- スケーリングと管理
- リクエスト/レスポンスの処理
- 環境変数の管理

## 追加リソース

- [Google ADK ドキュメンテーション](https://github.com/google/adk)
- [Gemini API ドキュメンテーション](https://ai.google.dev/docs)
- [Bedrock AgentCore ドキュメンテーション](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-core.html)