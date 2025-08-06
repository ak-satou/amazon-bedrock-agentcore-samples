# クラウドMCPサーバー

このコンポーネントは、OpenAPI仕様を使用して保険API操作をMCPツールとして公開するAWS Bedrock AgentCore Gatewayをセットアップします。

## 概要

クラウドMCPサーバーは、LLMを搭載したエージェントと保険APIの間の橋渡しとして機能します。AWS Bedrock AgentCore Gatewayを活用して、エージェントが保険業務にアクセスするために使用できる安全で管理されたMCP（Model Control Protocol）エンドポイントを作成します。

## 機能

- **OpenAPI統合**：APIのOpenAPI仕様を自動的にMCPツールに変換
- **OAuth認証**：Amazon Cognitoによる安全なアクセスの設定
- **環境ベースの設定**：柔軟なデプロイメントのための環境変数の使用
- **ゲートウェイ管理**：AWS Bedrock AgentCore Gatewayの作成と設定

## 前提条件

- Bedrockアクセス権を持つAWSアカウント
- Python 3.10+
- 保険API用のOpenAPI仕様
- APIエンドポイントがデプロイされアクセス可能

## インストール

1. このリポジトリをクローン
2. 依存関係をインストール：

```bash
pip install -r requirements.txt
```

## 設定

`cloud_mcp_server`ディレクトリに以下の変数を含む`.env`ファイルを作成します：

```
# AWS設定
AWS_REGION=us-west-2
ENDPOINT_URL=https://bedrock-agentcore-control.us-west-2.amazonaws.com

# ゲートウェイ設定
GATEWAY_NAME=InsuranceAPIGatewayCreds3
GATEWAY_DESCRIPTION=Insurance API Gateway with OpenAPI Specification

# API設定
API_GATEWAY_URL=https://your-api-gateway-url.execute-api.us-west-2.amazonaws.com/dev
OPENAPI_FILE_PATH=../cloud_insurance_api/openapi.json

# API認証情報
API_KEY=your-api-key
CREDENTIAL_LOCATION=HEADER
CREDENTIAL_PARAMETER_NAME=X-Subscription-Token

# 出力設定
GATEWAY_INFO_FILE=./gateway_info.json
```

プレースホルダー値を実際の設定に置き換えてください。

## 使用方法

ゲートウェイ セットアップ スクリプトを実行します：

```bash
python agentcore_gateway_setup_openapi.py
```

これにより以下が実行されます：

1. 新しいAWS Bedrock AgentCore Gatewayを作成
2. Amazon Cognitoでの OAuth認証を設定
3. OpenAPI仕様をMCPツールとして登録
4. 将来の使用のためにゲートウェイ情報を保存

スクリプト実行後、以下を受け取ります：
- ゲートウェイIDとMCP URL
- 認証資格情報
- テスト用アクセス トークン

## ゲートウェイ情報

スクリプトは、すべてのゲートウェイ情報をJSONファイル（デフォルトでは`gateway_info.json`）に以下の構造で保存します：

```json
{
  "gateway": {
    "name": "InsuranceAPIGatewayCreds3",
    "id": "gateway-id",
    "mcp_url": "https://gateway-id.gateway.bedrock-agentcore.region.amazonaws.com/mcp",
    "region": "us-west-2",
    "description": "Insurance API Gateway with OpenAPI Specification"
  },
  "api": {
    "gateway_url": "https://your-api-gateway-url.execute-api.us-west-2.amazonaws.com/dev",
    "openapi_file_path": "/path/to/openapi.json",
    "target_id": "target-id"
  },
  "auth": {
    "access_token": "temporary-access-token",
    "client_id": "cognito-client-id",
    "client_secret": "cognito-client-secret",
    "token_endpoint": "https://auth-endpoint.amazonaws.com/oauth2/token",
    "scope": "gateway-name/invoke",
    "user_pool_id": "us-west-2_poolid",
    "discovery_url": "https://cognito-idp.region.amazonaws.com/user-pool-id/.well-known/openid-configuration"
  }
}
```

## エージェントの接続

エージェントは、MCP URLと認証トークンを使用してこのMCPサーバーに接続できます。例えば、strands-agentsライブラリを使用する場合：

```python
from strands import Agent
from strands.tools.mcp import MCPClient
from mcp.client.streamable_http import streamablehttp_client

# gateway_info.jsonからMCP URLとアクセス トークンを取得
MCP_SERVER_URL = "https://gateway-id.gateway.bedrock-agentcore.region.amazonaws.com/mcp"
access_token = "your-access-token"

# MCPクライアントを作成
mcp_client = MCPClient(lambda: streamablehttp_client(
    MCP_SERVER_URL, 
    headers={"Authorization": f"Bearer {access_token}"}
))

# エージェントでクライアントを使用
with mcp_client:
    tools = mcp_client.list_tools_sync()
    agent = Agent(
        model="your-model",
        tools=tools,
        system_prompt="Your system prompt"
    )
    response = agent(user_input)
```

## トラブルシューティング

- **認証エラー**：アクセス トークンが有効で期限切れでないことを確認してください
- **OpenAPIエラー**：スクリプトを実行する前にOpenAPI仕様を検証してください
- **ゲートウェイ作成の失敗**：AWS権限とBedrockサービス制限を確認してください
- **無効なエンドポイント**：API Gateway URLがアクセス可能であることを確認してください

## 次のステップ

MCPサーバーをセットアップした後：

1. MCP URLと認証を使用するようにエージェントを設定
2. 本番用にトークン更新をセットアップ
3. ゲートウェイ操作の監視とログ記録を追加