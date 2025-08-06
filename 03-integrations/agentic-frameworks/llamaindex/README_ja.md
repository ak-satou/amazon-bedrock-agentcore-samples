# LlamaIndex Agent と Bedrock AgentCore の統合

| 項目                 | 詳細                                                                      |
|---------------------|---------------------------------------------------------------------------|
| エージェントタイプ      | 同期                                                                     |
| エージェントフレームワーク | LlamaIndex                                                               |
| LLMモデル           | OpenAI GPT-4o-mini                                                       |
| コンポーネント        | AgentCore Runtime、Yahoo Finance Tools                                   |
| サンプルの複雑さ      | 簡単                                                                     |
| 使用SDK             | Amazon BedrockAgentCore Python SDK                                       |

この例では、LlamaIndex エージェントを AWS Bedrock AgentCore と統合し、ツール使用機能を持つ金融アシスタントをマネージドサービスとしてデプロイする方法を示します。

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

`llama_agent_hello_world.py` ファイルには、金融ツールと基本的な数学機能を持つLlamaIndex エージェントがBedrock AgentCoreと統合されています：

```python
import asyncio
from llama_index.core.agent.workflow import FunctionAgent
from llama_index.llms.openai import OpenAI
from llama_index.tools.yahoo_finance import YahooFinanceToolSpec

# カスタム関数ツールを定義
def multiply(a: float, b: float) -> float:
    """2つの数値を掛けて積を返す"""
    return a * b

def add(a: float, b: float) -> float:
    """2つの数値を足して和を返す"""
    return a + b

# その他の事前定義されたツールを追加
finance_tools = YahooFinanceToolSpec().to_tool_list()
finance_tools.extend([multiply, add])

# ツールを使ってエージェントワークフローを作成
agent = FunctionAgent(
    tools=finance_tools,
    llm=OpenAI(model="gpt-4o-mini"),
    system_prompt="あなたは役立つアシスタントです。",
)

from bedrock_agentcore.runtime import BedrockAgentCoreApp
app = BedrockAgentCoreApp()

@app.entrypoint
async def main(payload):
    # エージェントを実行
    response = await agent.run(payload.get("prompt","AMZNの現在の株価は何ですか？"))
    print(response)
    return response.response.content

# エージェントを実行
if __name__ == "__main__":
    app.run()
```

### 4. Bedrock AgentCore Toolkitでの設定と起動

```bash
# デプロイメント用にエージェントを設定
agentcore configure -e llama_agent_hello_world.py

# OpenAI APIキーを使用してエージェントをデプロイ
agentcore launch --env OPENAI_API_KEY=sk-...
```

### 5. エージェントのテスト

クラウドにデプロイする前に、エージェントをローカルでテストできます：

```bash
# OpenAI APIキーを使用してローカルで起動
agentcore launch -l --env OPENAI_API_KEY=sk-...

# クエリでエージェントを呼び出し
agentcore invoke -l '{"prompt":"今日のAMZN株価"}'
```

クラウドデプロイの場合は、`-l` フラグを削除してください：

```bash
# クラウドにデプロイ
agentcore launch --env OPENAI_API_KEY=sk-...

# デプロイされたエージェントを呼び出し
agentcore invoke '{"prompt":"今日のAMZN株価"}'
```

エージェントは以下の処理を行います：
1. 金融に関するクエリを処理
2. Yahoo Finance ツールを使用してリアルタイムの株価データを取得
3. 要求された金融情報で応答を提供
4. 必要に応じて数学ツールを使用して計算を実行

## 動作原理

このエージェントは、LlamaIndexのエージェントフレームワークを使用して、以下の機能を持つ金融アシスタントを作成します：

1. 株式と金融データに関する自然言語クエリの処理
2. Yahoo Finance ツールを通じたリアルタイム株式情報へのアクセス
3. 必要に応じた基本的な数学演算の実行
4. データに基づく包括的な応答の提供

エージェントはBedrock AgentCoreフレームワークでラップされており、以下を処理します：
- AWSへのデプロイメント
- スケーリングと管理
- リクエスト/レスポンスの処理
- 環境変数の管理

## 追加リソース

- [LlamaIndex ドキュメンテーション](https://docs.llamaindex.ai/en/stable/use_cases/agents/)
- [Bedrock AgentCore ドキュメンテーション](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-core.html)
- [OpenAI API ドキュメンテーション](https://platform.openai.com/docs/api-reference)
- [Yahoo Finance API ドキュメンテーション](https://pypi.org/project/yfinance/)