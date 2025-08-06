# Amazon Bedrock AgentCore と Strands Agent を使用した Confluence MCP 情報検索システム実装計画

## プロジェクト概要

本プロジェクトは、Amazon Bedrock AgentCore と Strands Agent を活用して、Confluence の MCP（Model Context Protocol）を Gateway 経由で利用し、情報検索を行うシステムの実装を目的とします。

## アーキテクチャ設計

```
[ユーザー] → [Strands Agent] → [AgentCore Gateway] → [Confluence MCP Server] → [Confluence API]
```

### 主要コンポーネント

1. **Strands Agent**: ユーザーインターフェースとしてのAIエージェント
2. **AgentCore Gateway**: MCPプロトコルのゲートウェイ
3. **Confluence MCP Server**: Confluence APIとMCPプロトコルの橋渡し
4. **認証システム**: OAuth2/OIDC による安全な認証

## 実装計画

### Phase 1: 環境準備と基盤設定

#### 1.1 開発環境セットアップ
```bash
# Python 3.10+ 仮想環境作成
uv python install 3.10
uv venv --python 3.10
source .venv/bin/activate

# 依存関係インストール
uv add bedrock-agentcore
uv add strands-agents
uv add mcp-atlassian
uv add boto3
uv add opentelemetry-api
```

#### 1.2 AWS リソース構成
- **IAM Role**: Gateway実行用ロール作成
- **Cognito User Pool**: OAuth認証設定
- **Lambda Functions**: 必要に応じてカスタムMCPハンドラー

### Phase 2: MCP サーバーセットアップ

#### 2.1 Confluence MCP サーバー設定

**オプション A: Atlassian公式MCPサーバー利用**
```python
# 公式リモートMCPサーバー設定
MCP_SERVER_CONFIG = {
    "mcpServers": {
        "atlassian-remote": {
            "url": "https://api.atlassian.com/mcp",
            "transport": "streamable_http",
            "headers": {
                "Authorization": "Bearer {oauth_token}"
            }
        }
    }
}
```

**オプション B: オープンソースMCPサーバー利用（推奨）**

sooperset/mcp-atlassianを使用してConfluence MCPサーバーを構築します。

#### 2.1.1 事前準備

```bash
# MCPサーバー用Docker イメージを取得
docker pull ghcr.io/sooperset/mcp-atlassian:latest

# 設定ディレクトリ作成
mkdir -p config/mcp-server
```

#### 2.1.2 認証方式の選択

**APIトークン認証（推奨 - Atlassian Cloud）**
```bash
# Confluence API トークンの取得
# 1. Atlassian アカウント設定 → セキュリティ → API tokens
# 2. 「Create API token」でトークンを生成
export CONFLUENCE_API_TOKEN="your_generated_api_token"
```

**OAuth 2.0認証（高度な設定）**
```bash
# OAuth セットアップウィザード実行
docker run --rm -i \
  -p 8080:8080 \
  -v "${HOME}/.mcp-atlassian:/home/app/.mcp-atlassian" \
  ghcr.io/sooperset/mcp-atlassian:latest --oauth-setup -v
```

#### 2.1.3 環境設定ファイル

**config/mcp-server/confluence.env**
```bash
# Confluence接続設定
CONFLUENCE_URL=https://your-company.atlassian.net/wiki
CONFLUENCE_USERNAME=your.email@company.com
CONFLUENCE_API_TOKEN=your_confluence_api_token

# 利用可能ツールの制限（オプション）
ENABLED_TOOLS=confluence_search,confluence_get_page,confluence_create_page,confluence_update_page

# スペースフィルター（オプション - 特定スペースのみアクセス）
CONFLUENCE_SPACES_FILTER=DEV,TEAM,DOC,PROJECT

# 読み取り専用モード（オプション - 作成/更新を無効化）
READ_ONLY_MODE=false

# 詳細ログ出力（開発時のみ）
MCP_VERBOSE=true

# プロキシ設定（必要に応じて）
HTTP_PROXY=http://proxy.company.com:8080
HTTPS_PROXY=https://proxy.company.com:8080
```

#### 2.1.4 Docker Compose設定

**docker-compose.yml**
```yaml
version: '3.8'
services:
  confluence-mcp:
    image: ghcr.io/sooperset/mcp-atlassian:latest
    container_name: confluence-mcp-server
    ports:
      - "8080:8080"
    env_file:
      - ./config/mcp-server/confluence.env
    volumes:
      - ./data/mcp-atlassian:/home/app/.mcp-atlassian
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:8080/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    networks:
      - mcp-network

networks:
  mcp-network:
    driver: bridge
```

#### 2.1.5 利用可能なConfluence ツール

| ツール名 | 説明 | パラメーター |
|---------|------|------------|
| `confluence_search` | Confluenceページとコンテンツの検索 | query, space_key, limit |
| `confluence_get_page` | 特定ページの内容取得 | page_id または title + space_key |
| `confluence_create_page` | 新しいページの作成 | title, space_key, content, parent_id |
| `confluence_update_page` | 既存ページの更新 | page_id, title, content, version |
| `confluence_get_space` | スペース情報の取得 | space_key |
| `confluence_list_spaces` | 利用可能スペース一覧 | limit, start |

#### 2.1.6 MCPサーバー起動とテスト

```bash
# Docker Compose でサーバー起動
docker-compose up -d confluence-mcp

# ヘルスチェック
curl http://localhost:8080/health

# ログ確認
docker-compose logs -f confluence-mcp

# MCP ツール一覧確認（デバッグ用）
docker exec confluence-mcp-server cat /app/tools.json

# MCPサーバー停止
docker-compose down
```

#### 2.1.7 Claude Desktop との統合例

**.mcp.json**（Claude Desktop用設定）
```json
{
  "mcpServers": {
    "mcp-atlassian": {
      "command": "docker",
      "args": [
        "run", 
        "-i", 
        "--rm",
        "--env-file", 
        "./config/mcp-server/confluence.env",
        "ghcr.io/sooperset/mcp-atlassian:latest"
      ]
    }
  }
}
```

#### 2.1.8 トラブルシューティング

**よくある問題**：
1. **認証エラー**: API トークンの有効性確認
2. **接続エラー**: ネットワーク設定、プロキシ設定確認
3. **権限エラー**: Confluenceでの適切な権限設定確認

**デバッグコマンド**：
```bash
# 詳細ログで起動
docker run --rm -i \
  --env-file ./config/mcp-server/confluence.env \
  -e MCP_VERBOSE=true \
  ghcr.io/sooperset/mcp-atlassian:latest

# 接続テスト
curl -H "Authorization: Bearer ${CONFLUENCE_API_TOKEN}" \
  "${CONFLUENCE_URL}/rest/api/space"
```

### Phase 3: AgentCore Gateway 構築

#### 3.1 Gateway作成
```python
# gateway_setup.py
from bedrock_agentcore_starter_toolkit.operations.gateway.client import GatewayClient

def create_confluence_gateway():
    client = GatewayClient()
    
    # OAuth認証設定
    auth_config = client.create_oauth_authorizer_with_cognito(
        gateway_name="confluence-mcp-gateway"
    )
    
    # Gateway作成
    gateway = client.create_mcp_gateway(
        name="confluence-mcp-gateway",
        description="Confluence MCP Gateway for information retrieval",
        authorizer_config=auth_config["authorizer_config"]
    )
    
    return gateway
```

#### 3.2 Gateway Target設定（オープンソースMCPサーバー用）

```python
# target_configuration.py
import json
from bedrock_agentcore_starter_toolkit.operations.gateway.client import GatewayClient

def create_confluence_mcp_target(gateway, mcp_server_url="http://localhost:8080"):
    """
    オープンソースMCPサーバー向けのGateway Target設定
    """
    client = GatewayClient()
    
    # sooperset/mcp-atlassian で利用可能なConfluence ツールスキーマ
    confluence_tools_schema = {
        "confluence_search": {
            "description": "Search Confluence pages and content across spaces",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {
                        "type": "string", 
                        "description": "Search query string"
                    },
                    "space_key": {
                        "type": "string", 
                        "description": "Specific space key to search in (optional)"
                    },
                    "limit": {
                        "type": "integer", 
                        "description": "Maximum number of results",
                        "default": 20,
                        "maximum": 100
                    },
                    "include_archived": {
                        "type": "boolean",
                        "description": "Include archived content",
                        "default": False
                    }
                },
                "required": ["query"]
            }
        },
        "confluence_get_page": {
            "description": "Get specific Confluence page content by ID or title",
            "parameters": {
                "type": "object",
                "properties": {
                    "page_id": {
                        "type": "string",
                        "description": "Confluence page ID"
                    },
                    "title": {
                        "type": "string", 
                        "description": "Page title (use with space_key)"
                    },
                    "space_key": {
                        "type": "string",
                        "description": "Space key (required when using title)"
                    },
                    "expand": {
                        "type": "string",
                        "description": "Comma-separated list of properties to expand",
                        "default": "body.storage,version"
                    }
                },
                "oneOf": [
                    {"required": ["page_id"]},
                    {"required": ["title", "space_key"]}
                ]
            }
        },
        "confluence_create_page": {
            "description": "Create a new Confluence page",
            "parameters": {
                "type": "object",
                "properties": {
                    "title": {
                        "type": "string",
                        "description": "Page title"
                    },
                    "space_key": {
                        "type": "string",
                        "description": "Target space key"
                    },
                    "content": {
                        "type": "string",
                        "description": "Page content in Confluence storage format"
                    },
                    "parent_id": {
                        "type": "string",
                        "description": "Parent page ID (optional)"
                    }
                },
                "required": ["title", "space_key", "content"]
            }
        },
        "confluence_update_page": {
            "description": "Update existing Confluence page",
            "parameters": {
                "type": "object",
                "properties": {
                    "page_id": {
                        "type": "string",
                        "description": "Page ID to update"
                    },
                    "title": {
                        "type": "string",
                        "description": "Updated page title"
                    },
                    "content": {
                        "type": "string",
                        "description": "Updated page content"
                    },
                    "version": {
                        "type": "integer",
                        "description": "Current page version number"
                    }
                },
                "required": ["page_id", "title", "content", "version"]
            }
        },
        "confluence_get_space": {
            "description": "Get information about a Confluence space",
            "parameters": {
                "type": "object",
                "properties": {
                    "space_key": {
                        "type": "string",
                        "description": "Space key"
                    },
                    "expand": {
                        "type": "string",
                        "description": "Properties to expand",
                        "default": "description,homepage"
                    }
                },
                "required": ["space_key"]
            }
        },
        "confluence_list_spaces": {
            "description": "List available Confluence spaces",
            "parameters": {
                "type": "object",
                "properties": {
                    "limit": {
                        "type": "integer",
                        "description": "Maximum number of spaces to return",
                        "default": 25,
                        "maximum": 100
                    },
                    "start": {
                        "type": "integer",
                        "description": "Starting index for pagination",
                        "default": 0
                    },
                    "type": {
                        "type": "string",
                        "description": "Filter by space type",
                        "enum": ["global", "personal"]
                    }
                }
            }
        }
    }
    
    # HTTP Target設定（MCPサーバーへの接続）
    target_config = {
        "mcp": {
            "http": {
                "url": mcp_server_url,
                "method": "POST",
                "headers": {
                    "Content-Type": "application/json",
                    "Accept": "application/json"
                },
                "toolSchema": {
                    "inlinePayload": json.dumps(confluence_tools_schema)
                },
                "timeout": 30
            }
        }
    }
    
    # Gateway Targetを作成
    target = client.create_mcp_gateway_target(
        gateway=gateway,
        name="confluence-mcp-tools",
        description="Confluence MCP tools via sooperset/mcp-atlassian",
        target_type="http",
        target_payload=target_config["mcp"]["http"],
        # オープンソースMCPサーバーは独自認証なし（ConfluenceのAPI認証はMCPサーバー内で処理）
        credentials={}
    )
    
    return target

def create_lambda_mcp_target(gateway, lambda_arn):
    """
    Lambda経由でMCPサーバーを利用する場合の設定
    """
    client = GatewayClient()
    
    # Lambda Target設定
    lambda_target_config = {
        "mcp": {
            "lambda": {
                "lambdaArn": lambda_arn,
                "toolSchema": {
                    "inlinePayload": json.dumps(confluence_tools_schema)
                }
            }
        }
    }
    
    # 認証情報設定
    credential_config = [{
        "credentialProviderType": "GATEWAY_IAM_ROLE"
    }]
    
    target = client.create_mcp_gateway_target(
        gateway=gateway,
        name="confluence-mcp-lambda",
        description="Confluence MCP tools via Lambda wrapper",
        target_type="lambda", 
        target_payload=lambda_target_config["mcp"]["lambda"],
        credential_provider_configurations=credential_config
    )
    
    return target
```

### Phase 4: Strands Agent実装

#### 4.1 MCPクライアント作成
```python
# mcp_client.py
from strands.tools.mcp.mcp_client import MCPClient
from mcp.client.streamable_http import streamablehttp_client
import os

def create_confluence_mcp_client():
    gateway_url = os.getenv("CONFLUENCE_GATEWAY_URL")
    access_token = os.getenv("CONFLUENCE_ACCESS_TOKEN")
    
    def create_transport():
        headers = {"Authorization": f"Bearer {access_token}"}
        return streamablehttp_client(gateway_url, headers=headers)
    
    return MCPClient(create_transport)
```

#### 4.2 Strands Agent実装
```python
# confluence_agent.py
from strands import Agent
from strands.tools.mcp import MCPClient

CONFLUENCE_SYSTEM_PROMPT = """
あなたはConfluenceの情報検索と管理を支援するAIアシスタントです。

主な機能：
1. Confluenceページの検索
2. ページ内容の要約
3. 関連情報の抽出
4. 新しいページの作成提案

ユーザーの質問に対して、適切なConfluenceツールを使用して回答してください。
回答は日本語で、わかりやすく構造化して提供してください。
"""

class ConfluenceAgent:
    def __init__(self):
        self.mcp_client = create_confluence_mcp_client()
        self.model = "us.anthropic.claude-3-7-sonnet-20250219-v1:0"
    
    def search_confluence(self, query: str, space_key: str = None) -> str:
        with self.mcp_client:
            tools = self.mcp_client.list_tools_sync()
            
            agent = Agent(
                model=self.model,
                tools=tools,
                system_prompt=CONFLUENCE_SYSTEM_PROMPT
            )
            
            search_query = f"Confluenceで「{query}」について検索してください。"
            if space_key:
                search_query += f" 検索対象スペース: {space_key}"
            
            return agent(search_query)
    
    def get_page_summary(self, page_id: str) -> str:
        with self.mcp_client:
            tools = self.mcp_client.list_tools_sync()
            
            agent = Agent(
                model=self.model,
                tools=tools,
                system_prompt=CONFLUENCE_SYSTEM_PROMPT
            )
            
            return agent(f"ページID {page_id} の内容を取得して要約してください。")
```

### Phase 5: BedrockAgentCoreアプリケーション統合

#### 5.1 エントリーポイント実装
```python
# app.py
from bedrock_agentcore.runtime import BedrockAgentCoreApp
from confluence_agent import ConfluenceAgent
import json

app = BedrockAgentCoreApp()
confluence_agent = ConfluenceAgent()

@app.entrypoint
async def invoke(payload):
    """
    AgentCoreエントリーポイント
    """
    try:
        # ペイロードからユーザーメッセージを抽出
        user_message = payload.get("inputText", "")
        space_key = payload.get("spaceKey")
        
        # Confluenceエージェントで処理
        response = confluence_agent.search_confluence(
            query=user_message,
            space_key=space_key
        )
        
        return {
            "output": response,
            "metadata": {
                "source": "confluence-mcp",
                "timestamp": str(datetime.now())
            }
        }
        
    except Exception as e:
        return {
            "error": f"エラーが発生しました: {str(e)}",
            "output": "申し訳ありません。処理中にエラーが発生しました。"
        }
```

### Phase 6: 認証とセキュリティ

#### 6.1 OAuth2設定
```python
# auth_config.py
OAUTH_CONFIG = {
    "cognito": {
        "user_pool_id": os.getenv("COGNITO_USER_POOL_ID"),
        "client_id": os.getenv("COGNITO_CLIENT_ID"),
        "client_secret": os.getenv("COGNITO_CLIENT_SECRET")
    },
    "confluence": {
        "oauth_url": "https://auth.atlassian.com/oauth/token",
        "scope": "read:confluence-content.all read:confluence-space.all"
    }
}

def get_confluence_access_token():
    """Confluence OAuth2 トークン取得"""
    # 実装詳細は認証プロバイダーに依存
    pass
```

### Phase 7: テスト実装

#### 7.1 単体テスト
```python
# tests/test_confluence_agent.py
import pytest
from unittest.mock import Mock, patch
from confluence_agent import ConfluenceAgent

@pytest.fixture
def mock_mcp_client():
    return Mock()

def test_search_confluence(mock_mcp_client):
    agent = ConfluenceAgent()
    agent.mcp_client = mock_mcp_client
    
    # テストケース実装
    result = agent.search_confluence("テストクエリ")
    assert result is not None

def test_get_page_summary(mock_mcp_client):
    agent = ConfluenceAgent()
    agent.mcp_client = mock_mcp_client
    
    # テストケース実装
    result = agent.get_page_summary("123456")
    assert result is not None
```

#### 7.2 統合テスト
```bash
# integration_test.sh
#!/bin/bash

echo "統合テスト開始..."

# Gateway接続テスト
python -c "
from confluence_agent import ConfluenceAgent
agent = ConfluenceAgent()
result = agent.search_confluence('テスト')
print(f'テスト結果: {result}')
"

echo "統合テスト完了"
```

### Phase 8: デプロイメント設定

#### 8.1 AWS Lambda設定
```python
# lambda_handler.py
import json
from app import app

def lambda_handler(event, context):
    """
    AWS Lambda ハンドラー
    """
    try:
        response = app.invoke(event)
        
        return {
            'statusCode': 200,
            'body': json.dumps(response, ensure_ascii=False)
        }
    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps({
                'error': str(e)
            }, ensure_ascii=False)
        }
```

#### 8.2 CloudFormation/CDKテンプレート
```yaml
# infrastructure.yml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Confluence MCP AgentCore Infrastructure'

Parameters:
  ConfluenceApiToken:
    Type: String
    NoEcho: true

Resources:
  ConfluenceAgentLambda:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: confluence-mcp-agent
      Runtime: python3.10
      Handler: lambda_handler.lambda_handler
      Environment:
        Variables:
          CONFLUENCE_API_TOKEN: !Ref ConfluenceApiToken
          GATEWAY_URL: !Ref AgentCoreGateway
```

### Phase 9: 運用・監視設定

#### 9.1 ログ設定
```python
# logging_config.py
import logging
from opentelemetry import trace

# OpenTelemetry設定
tracer = trace.get_tracer(__name__)

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

def log_confluence_operation(operation, query, result):
    with tracer.start_as_current_span(f"confluence_{operation}"):
        logging.info(f"Confluence {operation}: query={query}, result_length={len(str(result))}")
```

#### 9.2 メトリクス収集
```python
# metrics.py
from opentelemetry import metrics

meter = metrics.get_meter(__name__)
confluence_requests = meter.create_counter(
    "confluence_requests_total",
    description="Total number of Confluence requests"
)
```

## 設定ファイル例

### config/static-config.yaml
```yaml
confluence:
  base_url: "https://your-company.atlassian.net"
  api_version: "latest"
  timeout: 30
  
mcp:
  gateway_url: "${GATEWAY_URL}"
  timeout: 60
  retry_attempts: 3

agent:
  model: "us.anthropic.claude-3-7-sonnet-20250219-v1:0"
  temperature: 0.1
  max_tokens: 4000
```

### requirements.txt
```txt
# Amazon Bedrock AgentCore
bedrock-agentcore>=1.0.0
bedrock-agentcore-starter-toolkit>=1.0.0

# Agent Framework
strands-agents>=1.0.0

# MCP関連（オープンソースMCPサーバー利用時）
mcp-atlassian>=0.11.9

# AWS SDK
boto3>=1.26.0
botocore>=1.29.0

# 可観測性
opentelemetry-api>=1.15.0
opentelemetry-sdk>=1.15.0
opentelemetry-instrumentation-boto3sqs>=1.15.0

# HTTP クライアント（MCP通信用）
httpx>=0.24.0
requests>=2.31.0

# データ処理
pydantic>=2.0.0
pyyaml>=6.0

# 開発・テスト
pytest>=7.0.0
pytest-asyncio>=0.20.0
pytest-mock>=3.10.0

# Webサーバー（開発用）
uvicorn>=0.20.0
fastapi>=0.100.0

# ログ・設定
structlog>=22.0.0
python-dotenv>=1.0.0

# Docker compose用ヘルスチェック
docker>=6.0.0
```

### docker-compose.override.yml（開発用）
```yaml
# 開発環境用のオーバーライド設定
version: '3.8'
services:
  confluence-mcp:
    environment:
      - MCP_VERBOSE=true
      - READ_ONLY_MODE=false
    ports:
      - "8080:8080"  # MCP サーバー
      - "9090:9090"  # メトリクス（オプション）
    volumes:
      - ./logs:/app/logs
      - ./config/mcp-server:/app/config
```

## 実行手順

### セットアップ（オープンソースMCPサーバー利用）

#### Step 1: 環境準備
```bash
# 1. リポジトリクローンと環境準備
git clone <repository>
cd agentcore-with-confluence-mcp

# 2. Python仮想環境作成（uv推奨）
uv python install 3.10
uv venv --python 3.10
source .venv/bin/activate
```

#### Step 2: 依存関係インストール
```bash
# Python依存関係インストール
uv add -r requirements.txt

# Docker確認
docker --version
docker-compose --version

# MCPサーバーイメージ取得
docker pull ghcr.io/sooperset/mcp-atlassian:latest
```

#### Step 3: Confluence認証設定
```bash
# Confluence API トークン取得（Atlassian Cloud）
# https://id.atlassian.com/manage-profile/security/api-tokens

# 環境変数設定
export CONFLUENCE_URL="https://your-company.atlassian.net/wiki"
export CONFLUENCE_USERNAME="your.email@company.com"  
export CONFLUENCE_API_TOKEN="your_generated_api_token"
export AWS_REGION="us-east-1"

# 設定ファイル作成
mkdir -p config/mcp-server data/mcp-atlassian logs

# 環境設定ファイル作成
cat > config/mcp-server/confluence.env << EOF
CONFLUENCE_URL=${CONFLUENCE_URL}
CONFLUENCE_USERNAME=${CONFLUENCE_USERNAME}
CONFLUENCE_API_TOKEN=${CONFLUENCE_API_TOKEN}
ENABLED_TOOLS=confluence_search,confluence_get_page,confluence_create_page,confluence_update_page,confluence_get_space,confluence_list_spaces
MCP_VERBOSE=true
READ_ONLY_MODE=false
EOF
```

#### Step 4: MCPサーバー起動と確認
```bash
# MCPサーバー起動
docker-compose up -d confluence-mcp

# ヘルスチェック
curl http://localhost:8080/health
# Expected: {"status": "healthy"}

# ログ確認
docker-compose logs -f confluence-mcp

# 利用可能ツール確認
docker exec confluence-mcp-server curl -X POST http://localhost:8080/tools

# Confluence接続テスト
curl -H "Authorization: Bearer ${CONFLUENCE_API_TOKEN}" \
  "${CONFLUENCE_URL}/rest/api/space?limit=5"
```

#### Step 5: AgentCore Gateway構築
```bash
# Gateway設定スクリプト実行
python scripts/setup_gateway.py

# Gateway接続確認
python scripts/test_gateway_connection.py
```

#### Step 6: アプリケーション統合テスト
```bash
# 単体テスト実行
pytest tests/ -v

# 統合テスト実行
pytest tests/integration/ -v

# エンドツーエンドテスト
./scripts/test_e2e.sh
```

#### Step 7: アプリケーション起動
```bash
# 開発モードでの起動
python app.py --dev

# または本番モード
python app.py --prod

# Lambda関数としてのテスト
sam local start-api
```

### クイックスタートスクリプト
```bash
#!/bin/bash
# quick_start.sh

echo "🚀 Confluence MCP AgentCore セットアップ開始..."

# 環境変数確認
if [[ -z "$CONFLUENCE_URL" || -z "$CONFLUENCE_API_TOKEN" ]]; then
    echo "❌ 環境変数が設定されていません"
    echo "CONFLUENCE_URL と CONFLUENCE_API_TOKEN を設定してください"
    exit 1
fi

# ディレクトリ作成
mkdir -p config/mcp-server data/mcp-atlassian logs

# MCPサーバー設定
cat > config/mcp-server/confluence.env << EOF
CONFLUENCE_URL=${CONFLUENCE_URL}
CONFLUENCE_USERNAME=${CONFLUENCE_USERNAME}
CONFLUENCE_API_TOKEN=${CONFLUENCE_API_TOKEN}
ENABLED_TOOLS=confluence_search,confluence_get_page
MCP_VERBOSE=true
READ_ONLY_MODE=true
EOF

# MCPサーバー起動
echo "📦 MCPサーバーを起動しています..."
docker-compose up -d confluence-mcp

# ヘルスチェック
echo "🔍 ヘルスチェック実行中..."
sleep 10
HEALTH=$(curl -s http://localhost:8080/health)
if [[ $HEALTH == *"healthy"* ]]; then
    echo "✅ MCPサーバーが正常に起動しました"
else
    echo "❌ MCPサーバーの起動に失敗しました"
    docker-compose logs confluence-mcp
    exit 1
fi

# テスト実行
echo "🧪 基本テストを実行しています..."
python -c "
import requests
response = requests.get('http://localhost:8080/health')
print(f'Health Check: {response.json()}')
"

echo "🎉 セットアップ完了!"
echo "次のステップ:"
echo "1. Gateway設定: python scripts/setup_gateway.py"
echo "2. テスト実行: pytest tests/"
echo "3. アプリケーション起動: python app.py"
```

## トラブルシューティング

### よくある問題と対処法（オープンソースMCPサーバー利用時）

#### 1. MCPサーバー関連のエラー

**問題**: MCPサーバーが起動しない
```bash
# 解決方法
# Docker ログ確認
docker-compose logs confluence-mcp

# コンテナ状態確認
docker-compose ps

# ポート競合確認
netstat -tlnp | grep :8080
lsof -i :8080

# 設定ファイル確認
cat config/mcp-server/confluence.env
```

**問題**: Confluence API 認証エラー
```bash
# 解決方法
# API トークンの有効性確認
curl -u "${CONFLUENCE_USERNAME}:${CONFLUENCE_API_TOKEN}" \
  "${CONFLUENCE_URL}/rest/api/user/current"

# スペース一覧取得テスト
curl -H "Authorization: Bearer ${CONFLUENCE_API_TOKEN}" \
  "${CONFLUENCE_URL}/rest/api/space?limit=5"

# 権限確認（管理者権限が必要な場合がある）
```

#### 2. Gateway接続エラー

**問題**: Gateway作成に失敗
```bash
# 解決方法
# AWS 認証情報確認
aws sts get-caller-identity

# IAM ロール権限確認
aws iam get-role --role-name bedrock-agentcore-gateway-role

# リージョン設定確認
echo $AWS_REGION
aws configure list
```

**問題**: Gateway Target設定エラー
```python
# デバッグコード
import json
from bedrock_agentcore_starter_toolkit.operations.gateway.client import GatewayClient

client = GatewayClient()
try:
    # Target設定テスト
    response = client.create_mcp_gateway_target(...)
    print(f"Success: {response}")
except Exception as e:
    print(f"Error: {e}")
    print(f"Error details: {e.response if hasattr(e, 'response') else 'No details'}")
```

#### 3. Strands Agent実装エラー

**問題**: MCP ツールが認識されない
```python
# デバッグコード
from strands.tools.mcp.mcp_client import MCPClient

def debug_mcp_tools():
    with mcp_client:
        try:
            tools = mcp_client.list_tools_sync()
            print(f"Available tools: {[tool.name for tool in tools]}")
            return tools
        except Exception as e:
            print(f"Tool listing error: {e}")
            return []
```

**問題**: Gateway との通信エラー
```python
# 接続テスト用コード
import requests

def test_gateway_connection():
    gateway_url = os.getenv("CONFLUENCE_GATEWAY_URL")
    access_token = os.getenv("CONFLUENCE_ACCESS_TOKEN")
    
    headers = {"Authorization": f"Bearer {access_token}"}
    
    try:
        response = requests.get(f"{gateway_url}/health", headers=headers)
        print(f"Gateway health: {response.status_code} - {response.text}")
    except Exception as e:
        print(f"Gateway connection error: {e}")
```

#### 4. ネットワーク・セキュリティ関連

**問題**: プロキシ経由でのアクセスエラー
```bash
# config/mcp-server/confluence.env に追加
HTTP_PROXY=http://proxy.company.com:8080
HTTPS_PROXY=https://proxy.company.com:8080
NO_PROXY=localhost,127.0.0.1,.local

# Docker Compose でプロキシ設定
# docker-compose.yml に追加:
# environment:
#   - HTTP_PROXY=${HTTP_PROXY}
#   - HTTPS_PROXY=${HTTPS_PROXY}
```

**問題**: SSL証明書エラー（オンプレミス環境）
```bash
# Server/Data Center用設定
CONFLUENCE_SSL_VERIFY=false
CONFLUENCE_URL=https://confluence.internal.company.com
```

#### 5. パフォーマンス関連

**問題**: MCPサーバーの応答が遅い
```yaml
# docker-compose.yml でリソース制限を調整
services:
  confluence-mcp:
    deploy:
      resources:
        limits:
          memory: 1G
          cpus: '0.5'
        reservations:
          memory: 512M
          cpus: '0.25'
```

**問題**: Confluenceページの検索が遅い
```bash
# 環境変数で検索結果数を制限
CONFLUENCE_SEARCH_LIMIT=20
CONFLUENCE_SPACES_FILTER=MAIN,DEV  # 検索対象スペース限定
```

#### 6. 開発・デバッグ用コマンド

```bash
# 詳細ログでMCPサーバー実行
docker run --rm -i \
  --env-file config/mcp-server/confluence.env \
  -e MCP_VERBOSE=true \
  -p 8080:8080 \
  ghcr.io/sooperset/mcp-atlassian:latest

# MCPサーバー内部状態確認
docker exec -it confluence-mcp-server /bin/bash
cat /app/config.json
ls -la /home/app/.mcp-atlassian/

# Gateway ログ確認（AWS CloudWatch）
aws logs describe-log-groups --log-group-name-prefix="/aws/bedrock-agentcore"

# Lambda関数ログ確認
aws logs tail /aws/lambda/confluence-mcp-agent --follow --start-time 1h
```

### ログ確認コマンド
```bash
# Lambda関数ログ
aws logs tail /aws/lambda/confluence-mcp-agent --follow

# Gateway ログ
aws logs tail /aws/bedrock-agentcore/gateway/{gateway-id} --follow
```

## 今後の拡張計画

1. **マルチスペース対応**: 複数Confluenceスペースの横断検索
2. **キャッシュ機能**: 頻繁にアクセスするページの高速化
3. **日本語NLP強化**: 日本語固有の検索最適化
4. **ダッシュボード機能**: 利用統計とパフォーマンスモニタリング

## 参考リンク

- [Amazon Bedrock AgentCore Documentation](https://docs.aws.amazon.com/bedrock/)
- [Strands Agent Framework](https://github.com/strands-ai)
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [Atlassian MCP Server](https://github.com/sooperset/mcp-atlassian)
- [Confluence REST API](https://developer.atlassian.com/cloud/confluence/rest/v1/)
