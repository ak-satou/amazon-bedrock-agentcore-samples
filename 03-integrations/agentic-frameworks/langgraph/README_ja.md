# LangGraph Agent と Bedrock AgentCore の統合

| 項目                 | 詳細                                                                      |
|---------------------|---------------------------------------------------------------------------|
| エージェントタイプ      | 同期                                                                     |
| エージェントフレームワーク | LangGraph                                                                |
| LLMモデル           | Anthropic Claude 3 Haiku                                                |
| コンポーネント        | AgentCore Runtime                                                        |
| サンプルの複雑さ      | 簡単                                                                     |
| 使用SDK             | Amazon BedrockAgentCore Python SDK                                       |

この例では、LangGraph エージェントを AWS Bedrock AgentCore と統合し、ウェブ検索機能を持つエージェントをマネージドサービスとしてデプロイする方法を示します。

## 前提条件

- Python 3.10以上
- [uv](https://github.com/astral-sh/uv) - 高速なPythonパッケージインストーラーおよびリゾルバー
- Bedrockアクセス権限のあるAWSアカウント

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

`langgraph_agent_web_search.py` ファイルには、ウェブ検索機能を持つLangGraph エージェントがBedrock AgentCoreと統合されています：

```python
from typing import Annotated
from langchain.chat_models import init_chat_model
from typing_extensions import TypedDict
from langgraph.graph import StateGraph, START
from langgraph.graph.message import add_messages
from langgraph.prebuilt import ToolNode, tools_condition

# BedrockでLLMを初期化
llm = init_chat_model(
    "us.anthropic.claude-3-5-haiku-20241022-v1:0",
    model_provider="bedrock_converse",
)

# 検索ツールを定義
from langchain_community.tools import DuckDuckGoSearchRun
search = DuckDuckGoSearchRun()
tools = [search]
llm_with_tools = llm.bind_tools(tools)

# 状態を定義
class State(TypedDict):
    messages: Annotated[list, add_messages]

# グラフを構築
graph_builder = StateGraph(State)

def chatbot(state: State):
    return {"messages": [llm_with_tools.invoke(state["messages"])]}

graph_builder.add_node("chatbot", chatbot)
tool_node = ToolNode(tools=tools)
graph_builder.add_node("tools", tool_node)
graph_builder.add_conditional_edges("chatbot", tools_condition)
graph_builder.add_edge("tools", "chatbot")
graph_builder.add_edge(START, "chatbot")
graph = graph_builder.compile()

# Bedrock AgentCore との統合
from bedrock_agentcore.runtime import BedrockAgentCoreApp
app = BedrockAgentCoreApp()

@app.entrypoint
def agent_invocation(payload, context):
    tmp_msg = {"messages": [{"role": "user", "content": payload.get("prompt", "入力にプロンプトが見つかりません")}]}
    tmp_output = graph.invoke(tmp_msg)
    return {"result": tmp_output['messages'][-1].content}

app.run()
```

### 4. Bedrock AgentCore Toolkitでの設定と起動

```bash
# デプロイメント用にエージェントを設定
bedrock-agentcore-toolkit configure

# エージェントをデプロイ
bedrock-agentcore-toolkit deploy --app-file langgraph_agent_web_search.py
```

設定時に以下の項目が求められます：
- AWSリージョンの選択
- デプロイメント名の選択
- その他のデプロイメント設定

### 5. エージェントのテスト

デプロイ後、以下を使用してエージェントをテストできます：

```bash
bedrock-agentcore-toolkit invoke --prompt "量子コンピューティングの最新の開発状況は何ですか？"
```

エージェントは以下の処理を行います：
1. クエリを処理
2. DuckDuckGoを使用して関連情報を検索
3. 検索結果に基づいて包括的な応答を提供

### 6. クリーンアップ

デプロイしたエージェントを削除するには：

```bash
bedrock-agentcore-toolkit delete
```

## 動作原理

このエージェントは、LangGraphを使用してエージェントの推論用の有向グラフを作成します：

1. ユーザークエリがチャットボットノードに送信される
2. チャットボットがクエリに基づいてツールを使用するかどうかを決定
3. ツールが必要な場合、クエリがツールノードに送信される
4. ツールノードが検索を実行し、結果を返す
5. 結果がチャットボットに送り返され、最終的な応答が生成される

Bedrock AgentCoreフレームワークは、AWSでのエージェントのデプロイメント、スケーリング、管理を処理します。

## 追加リソース

- [LangGraph ドキュメンテーション](https://github.com/langchain-ai/langgraph)
- [LangChain ドキュメンテーション](https://python.langchain.com/docs/get_started/introduction)
- [Bedrock AgentCore ドキュメンテーション](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-core.html)