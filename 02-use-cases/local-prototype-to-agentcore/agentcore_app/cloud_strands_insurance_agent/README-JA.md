# AWS Bedrock AgentCoreを使用したクラウドStrands保険エージェント

このガイドでは、自動車保険見積もりと車両情報クエリを処理するためにAWS AgentCore Gateway MCPサービスに接続するStrandsベースの保険エージェントをデプロイして実行する方法を示します。

![Bedrock AgentCore保険アプリ会話](agentcore_strands_conversation.gif)

## 前提条件

- 適切な権限を持つAWSアカウント
- Docker DesktopまたはFinchがインストールされ実行中
- Python 3.10+
- AWS CLIがインストールされ設定済み
- jqコマンドライン JSONプロセッサ

## プロジェクト構造

```
cloud_strands_insurance_agent/
├── agentcore_strands_insurance_agent.py  # メイン エージェント コード
├── requirements.txt                      # 依存関係
├── 1_pre_req_setup/                      # セットアップ スクリプト
│   ├── cognito_auth/                     # 認証セットアップ
│   │   ├── setup_cognito.sh              # インタラクティブ セットアップ スクリプト
│   │   ├── refresh_token.sh              # トークン更新ユーティリティ
│   │   ├── cognito_config.json           # 設定ストレージ
│   │   └── README.md                     # セットアップ ドキュメント
│   └── iam_roles_setup/                  # IAMロール設定
│       ├── setup_role.sh                 # インタラクティブ IAMロール セットアップ
│       ├── policy_templates.py           # IAMポリシー定義
│       ├── config.py                     # 設定ユーティリティ
│       ├── collect_info.py               # インタラクティブ入力収集
│       └── README.md                     # セットアップ ドキュメント
└── .env_example                          # 環境変数テンプレート
```

## ステップ1：前提条件のセットアップ

必要なIAMロールとCognito認証をセットアップします：

### IAM実行ロール

```bash
cd 1_pre_req_setup/iam_roles_setup
./setup_role.sh
```

このインタラクティブ スクリプトは以下を実行します：
- AWS認証情報を確認
- 必要な情報（リージョン、リポジトリ名、エージェント名）を収集
- Bedrock AgentCore用の最小権限でIAMロールを作成
- 後で使用するためにロールARNを保存

### Cognito認証

```bash
cd ../cognito_auth
./setup_cognito.sh
```

このインタラクティブ スクリプトは以下を実行します：
- Cognitoユーザー プールとアプリ クライアントを作成
- 認証情報でテスト ユーザーをセットアップ
- 初期認証トークンを生成
- 簡単なアクセスのためにすべての設定を保存

## ステップ2：環境変数の設定

エージェントは設定に環境変数を使用します。例に基づいて`.env`ファイルを作成します：

```bash
# 例ファイルをコピーして値を編集
cp .env_example .env
nano .env
```

必要な環境変数：
```
# MCPサーバー設定
MCP_SERVER_URL="your-gateway-mcp-url"
MCP_ACCESS_TOKEN="your-access-token"

# モデル設定
MODEL_NAME="us.anthropic.claude-3-7-sonnet-20250219-v1:0"

# オプション：ゲートウェイ情報ファイル パス（トークン更新用）
GATEWAY_INFO_FILE="../cloud_mcp_server/gateway_info.json"
```

ゲートウェイ セットアップ中に生成されたgateway_info.jsonファイルからアクセス トークンとMCP URLを取得できます：

```bash
# gateway_info.jsonから値を抽出（利用可能な場合）
MCP_URL=$(jq -r '.gateway.mcp_url' ../cloud_mcp_server/gateway_info.json)
ACCESS_TOKEN=$(jq -r '.auth.access_token' ../cloud_mcp_server/gateway_info.json)

# 抽出した値で.envファイルを更新
sed -i "s|MCP_SERVER_URL=.*|MCP_SERVER_URL=\"$MCP_URL\"|g" .env
sed -i "s|MCP_ACCESS_TOKEN=.*|MCP_ACCESS_TOKEN=\"$ACCESS_TOKEN\"|g" .env
```

## ステップ3：エージェントの設定

実行ロール（ステップ1のARNを使用）でエージェントを設定します：

```bash
# セットアップ出力またはAWSコンソールからロールARNを取得
ROLE_ARN=$(aws iam get-role --role-name BedrockAgentCoreExecutionRole --query 'Role.Arn' --output text)

# エージェントを設定
agentcore configure -e "agentcore_strands_insurance_agent.py" \
  --name insurance_agent_strands \
  -er $ROLE_ARN
```

これにより作成されます：
- `.bedrock_agentcore.yaml` - 設定ファイル
- `Dockerfile` - コンテナ ビルド指示（まだ存在しない場合）
- `.dockerignore` - ビルドから除外するファイル

## ステップ4：ローカル テスト

クラウド デプロイメント前にエージェントをローカルでテストします：

```bash
# .envファイルから環境変数を読み込み
source .env

# 環境変数でローカル起動
agentcore launch -l \
  -env MCP_SERVER_URL=$MCP_SERVER_URL \
  -env MCP_ACCESS_TOKEN=$MCP_ACCESS_TOKEN
```

これにより以下が実行されます：
- Dockerイメージをビルド
- ポート8080でコンテナをローカル実行
- エージェント サーバーを開始

ローカル テスト：
```bash
curl -X POST http://localhost:8080/invocations \
  -H "Content-Type: application/json" \
  -d '{"user_input": "自動車保険の見積もりが必要です"}'
```

## ステップ5：クラウドへのデプロイ

エージェントをAWSにデプロイします：

```bash
# .envファイルから環境変数を読み込み
source .env

# AWS Bedrock AgentCoreにデプロイ
agentcore launch \
  -env MCP_SERVER_URL=$MCP_SERVER_URL \
  -env MCP_ACCESS_TOKEN=$MCP_ACCESS_TOKEN
```

これにより以下が実行されます：
- DockerイメージをビルドしてECRにプッシュ
- Bedrock AgentCoreランタイムを作成
- エージェントをクラウドにデプロイ
- 呼び出し用のエージェントARNを返す

## ステップ6：エージェントの呼び出し

ベアラー トークンを設定してデプロイしたエージェントを呼び出します：

```bash
# Cognitoベアラー トークンを取得
cd 1_pre_req_setup/cognito_auth

# 必要に応じてトークンを更新
./refresh_token.sh

# トークンをエクスポート
export BEARER_TOKEN=$(jq -r '.bearer_token' cognito_config.json)

# プロジェクト ルートに戻る
cd ../../

# エージェントを呼び出し
agentcore invoke --bearer-token $BEARER_TOKEN '{"user_input": "自動車保険の見積もりを手伝ってもらえますか？"}'
```

## エージェント コード構造

`agentcore_strands_insurance_agent.py`は以下のパターンに従います：

```python
from strands import Agent
from strands.tools.mcp import MCPClient
from mcp.client.streamable_http import streamablehttp_client
from bedrock_agentcore.runtime import BedrockAgentCoreApp

app = BedrockAgentCoreApp()

@app.entrypoint
def invoke(payload):
    user_input = payload.get("user_input", "自動車保険の見積もりが必要です")
    
    # 認証でGateway MCPに接続
    gateway_client = MCPClient(lambda: streamablehttp_client(
        gateway_url, 
        headers={"Authorization": f"Bearer {access_token}"}
    ))
    
    with gateway_client:
        tools = gateway_client.list_tools_sync()
        agent = Agent(
            model="us.anthropic.claude-3-7-sonnet-20250219-v1:0",
            tools=tools,
            system_prompt="あなたは保険エージェント アシスタントです..."
        )
        response = agent(user_input)
        return {"result": str(response)}
```

## 依存関係

`requirements.txt`の主要依存関係：
```
mcp>=0.1.0
strands-agents>=0.1.8
bedrock_agentcore
boto3
botocore
typing-extensions
python-dateutil
python-dotenv>=1.0.0
```

## トラブルシューティング

- **424 Failed Dependency**：CloudWatchでエージェント ログを確認してください
- **トークンの期限切れ**：`./1_pre_req_setup/cognito_auth/refresh_token.sh`を実行して`.env`ファイルを更新してください
- **権限拒否**：実行ロールがBedrockモデル アクセス権を持っていることを確認してください
- **ローカル テストの失敗**：Dockerが実行されていることを確認してください
- **認証エラー**：`.env`ファイルのMCP_ACCESS_TOKENが有効で期限切れでないことを確認してください
- **IAMロール エラー**：IAMロールが`iam_roles_setup/README.md`で指定されたすべての必要な権限を持っていることを確認してください
- **Cognito認証の問題**：トラブルシューティングについては`cognito_auth/README.md`のドキュメントを確認してください

## 監視と観測可能性

- CloudWatchでエージェント パフォーマンスを監視
- AWS X-Rayでトレースを表示
- 詳細なエラー情報についてはエージェント ログを確認

## 次のステップ

- トークン更新の自動化をセットアップ
- セッション管理を設定
- 追加の保険APIと統合
- エラー処理とログ記録を強化