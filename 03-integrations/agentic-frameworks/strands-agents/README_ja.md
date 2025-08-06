# Strands Agent と Bedrock AgentCore の統合

| 項目                 | 詳細                                                                      |
|---------------------|---------------------------------------------------------------------------|
| エージェントタイプ      | 同期                                                                     |
| エージェントフレームワーク | Strands                                                                  |
| LLMモデル           | Anthropic Claude 3 Haiku                                                |
| コンポーネント        | AgentCore Runtime                                                        |
| サンプルの複雑さ      | 簡単                                                                     |
| 使用SDK             | Amazon BedrockAgentCore Python SDK                                       |

これらの例では、Strands エージェントを AWS Bedrock AgentCore と統合し、エージェントをマネージドサービスとしてデプロイする方法を示します。`agentcore` CLI を使用してこれらのエージェントを設定・起動できます。

## 前提条件

- Python 3.10以上
- [uv](https://github.com/astral-sh/uv) - 高速なPythonパッケージインストーラーおよびリゾルバー
- Bedrock AgentCore アクセス権限のあるAWSアカウント

## セットアップ手順

### 1. uvを使用してPython環境を作成

```bash
# まだuvをインストールしていない場合はインストール

# 仮想環境を作成してアクティベート
uv venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
```

### 2. 依存関係のインストール

```bash
uv pip install -r requirements.txt
```

### 3. APIキープロバイダーの設定（OpenAI + strands例用）

Strands エージェントでOpenAI モデルを使用する場合は、APIキープロバイダーを設定する必要があります：

```python
from bedrock_agentcore.services.identity import IdentityClient
from boto3.session import Session
import boto3

boto_session = Session()
region = boto_session.region_name

# APIキープロバイダーを設定
identity_client = IdentityClient(region=region)
api_key_provider = identity_client.create_api_key_credential_provider({
    "name": "openai-apikey-provider",
    "apiKey": "sk-..." # OpenAIから取得したAPIキーに置き換えてください
})
print(api_key_provider)
```

### 4. エージェントコードの理解

`strands_agent_file_system.py` ファイルには、ファイルシステム機能を持つシンプルなStrands エージェントがBedrock AgentCoreと統合されています：

```python
import os
os.environ["BYPASS_TOOL_CONSENT"]="true"

from strands import Agent
from strands_tools import file_read, file_write, editor

# ファイルシステムツールでStrands エージェントを初期化
agent = Agent(tools=[file_read, file_write, editor])

# Bedrock AgentCore との統合
from bedrock_agentcore.runtime import BedrockAgentCoreApp
app = BedrockAgentCoreApp()

@app.entrypoint
def agent_invocation(payload, context):
    """エージェント呼び出しのハンドラー"""
    user_message = payload.get("prompt", "入力にプロンプトが見つかりません。promptキーを含むjsonペイロードを作成するよう顧客に案内してください")
    result = agent(user_message)
    return {"result": result.message}

app.run()
```

### 5. Bedrock AgentCore Toolkitでの設定と起動

```bash
# デプロイメント用にエージェントを設定
agentcore configure -e strands_agent_file_system.py

# エージェントをデプロイ
agentcore launch
```

### 6. エージェントのテスト

デプロイ後、以下を使用してエージェントをテストできます：

```bash
agentcore invoke '{"prompt":"こんにちは"}'
```