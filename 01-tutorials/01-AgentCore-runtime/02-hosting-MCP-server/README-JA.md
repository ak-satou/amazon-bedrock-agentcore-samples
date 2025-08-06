# AgentCore RuntimeでのMCPサーバーホスティング

## 概要

このセッションでは、Amazon Bedrock AgentCore RuntimeでMCPツールをホストする方法について説明します。

Amazon Bedrock AgentCore Python SDKを使用して、エージェント関数をAmazon Bedrock AgentCoreと互換性のあるMCPサーバーとしてラップします。
MCPサーバーの詳細を処理するため、エージェントのコア機能に集中できます。

Amazon Bedrock AgentCore Python SDKは、エージェントまたはツールコードがAgentCore Runtimeで実行できるように準備します。

コードをAgentCoreの標準化されたHTTPプロトコルまたはMCPプロトコル契約に変換し、従来のリクエスト/レスポンスパターン（HTTPプロトコル）用の直接REST APIエンドポイント通信またはツールとエージェントサーバー用のModel Context Protocol（MCPプロトコル）を可能にします。

ツールをホストする際、Amazon Bedrock AgentCore Python SDKは[Stateless Streamable HTTP]転送プロトコルを`MCP-Session-Id`ヘッダーと共に実装し、[セッション分離]https://modelcontextprotocol.io/specification/2025-06-18/basic/transports#session-managementを行います。サーバーは、プラットフォーム生成のMcp-Session-Idヘッダーを拒否しないようにステートレス操作をサポートする必要があります。
MCPサーバーはポート`8000`でホストされ、1つの呼び出しパス：`mcp-POST`を提供します。このインタラクションエンドポイントはMCP RPCメッセージを受信し、ツールの機能を通じて処理します。応答コンテンツタイプとしてapplication/jsonとtext/event-streamの両方をサポートします。

AgentCoreプロトコルをMCPに設定すると、AgentCore Runtimeは、ほとんどの公式MCP server SDKでサポートされているデフォルトパスであるため、MCPサーバーコンテナがパス`0.0.0.0:8000/mcp`にあることを期待します。

AgentCore Runtimeでは、デフォルトでセッション分離を提供し、それなしのリクエストに対してMcp-Session-Idヘッダーを自動的に追加するため、MCPクライアントが同じBedrock AgentCore RuntimeセッションIDに接続を継続できるよう、ステートレスストリーム可能httpサーバーをホストする必要があります。

`InvokeAgentRuntime` APIのペイロードは完全にパススルーであるため、MCPなどのプロトコルのRPCメッセージを簡単にプロキシできます。

このチュートリアルでは以下を学習します：

* ツールを使用したMCPサーバーの作成方法
* サーバーをローカルでテストする方法
* サーバーをAWSにデプロイする方法
* デプロイされたサーバーを呼び出す方法

### チュートリアル詳細

| 情報         | 詳細                                                   |
|:--------------------|:----------------------------------------------------------|
| チュートリアルタイプ       | ツールホスティング                                             |
| ツールタイプ           | MCPサーバー                                                |
| チュートリアルコンポーネント | AgentCore Runtimeでのツールホスティング。MCPサーバーの作成 |
| チュートリアル分野   | クロス分野                                            |
| 例の複雑さ  | 簡単                                                      |
| 使用SDK            | Amazon BedrockAgentCore Python SDKとMCP Client         |

### チュートリアルアーキテクチャ
このチュートリアルでは、既存のMCPサーバーをAgentCore runtimeにデプロイする方法を説明します。

デモンストレーション目的で、3つのツール（`add_numbers`、`multiply_numbers`、`greet_users`）を持つ非常にシンプルなMCPサーバーを使用します

![MCPアーキテクチャ](images/hosting_mcp_server.png)

### チュートリアル主要機能

* MCPサーバーのホスティング