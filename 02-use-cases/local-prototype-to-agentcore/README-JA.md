# プロトタイプから本番環境へ: AWS Bedrock AgentCoreを使用したエージェントアプリケーション

> [!CAUTION]
> このサンプルは実験的および教育目的のみです。概念や技術を実証するものですが、本番環境での直接使用を意図したものではありません。[プロンプトインジェクション](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-injection.html)から保護するために、Amazon Bedrock Guardrailsを設置することを確認してください。

**重要:**
- このアプリケーションは本番環境での直接使用を**意図していません**。
- ここで提示されているコードとアーキテクチャは例であり、特定のユースケースのすべてのセキュリティ、スケーラビリティ、またはコンプライアンス要件を満たしていない可能性があります。
- 本番環境で類似のシステムをデプロイする前に、以下を行うことが重要です：
  - 徹底的なテストとセキュリティ監査の実施
  - すべての関連する規制と業界標準への準拠の確保
  - 特定のパフォーマンスとスケーラビリティのニーズに最適化
  - 適切なエラーハンドリング、ログ記録、監視の実装
  - 本番デプロイメントのためのすべてのAWSベストプラクティスの遵守

<div align="center">
  <img src="https://img.shields.io/badge/Python-3.10+-blue.svg" alt="Python 3.10+"/>
  <img src="https://img.shields.io/badge/AWS-Bedrock_AgentCore-orange.svg" alt="AWS Bedrock AgentCore"/>
  <img src="https://img.shields.io/badge/Strands-Agents-green.svg" alt="Strands Agents"/>
  <img src="https://img.shields.io/badge/FastAPI-0.100.0+-purple.svg" alt="FastAPI"/>
</div>

<!-- アーキテクチャ図は本番環境への移行セクションでASCIIアートとして表示 -->

## 🚀 概要

このリポジトリは、エージェントアプリケーションのローカルプロトタイプをAWS Bedrock AgentCoreを使用した本番対応システムに変換する方法を示しています。顧客が見積もり取得、車両情報検索、および保険契約管理を行えるように支援する自動車保険エージェントの完全な実装を提供します。

このリポジトリには2つの並列実装が含まれています：

- **`local_prototype/`** - 開発に重点を置いた実装で、ローカルサーバーと直接接続を使用
- **`agentcore_app/`** - AWS Bedrock AgentCoreサービスを活用した本番対応実装

## ✨ クラウド移行の主要な利点

| ローカルプロトタイプ | AgentCoreでの本番環境 | 利点 |
|----------------|--------------------------|---------|
| 手動認証 | CognitoでのOAuth2 | 🔒 セキュリティの強化 |
| ローカルログ記録 | CloudWatch統合 | 📊 集中監視 |
| 手動スケーリング | 自動スケーリングランタイム | 📈 パフォーマンス向上 |
| ローカルデプロイメント | コンテナ化デプロイメント | 🚀 運用の簡素化 |
| アドホックテスト | CI/CD統合 | 🧪 信頼性の高い品質 |
| 直接APIアクセス | API Gateway + Lambda | 💰 コスト最適化 |

## 🏗️ アーキテクチャ

本番アーキテクチャは3つの主要コンポーネントで構成されています：

1. **クラウド保険API** - AWS Lambda関数としてデプロイされたFastAPIアプリケーション
2. **クラウドMCPサーバー** - 保険APIをMCPツールとして公開するAgentCore Gateway設定
3. **クラウドStrands保険エージェント** - AgentCore Gatewayを使用するStrandsベースのエージェント

### コンポーネント相互作用フロー:

1. ユーザーが保険エージェントにクエリを送信
2. エージェントがOAuth認証を使用してAgentCore Gateway MCPサーバーに接続
3. AgentCore GatewayがAPI操作をMCPツールとして公開
4. 保険APIがリクエストを処理してデータを返す
5. エージェントがLLMとツール結果を使用して応答を作成

## 🛠️ 使用技術

- **AWS Bedrock AgentCore**: 管理されたエージェントランタイム環境
- **AWS Lambda & API Gateway**: サーバーレスAPIホスティング
- **Amazon Cognito**: 認証と認可
- **Strands Agents**: MCP統合を備えたエージェントフレームワーク
- **FastAPI**: Python用モダンAPIフレームワーク
- **CloudWatch**: 監視と観測性

## 📚 リポジトリ構造

```
insurance_final/
├── local_prototype/                  # 開発実装
│   ├── insurance_api/                # ローカルFastAPIサーバー
│   ├── native_mcp_server/            # ローカルMCPサーバー
│   └── strands_insurance_agent/      # ローカルエージェント実装
│
└── agentcore_app/       # 本番実装
    ├── cloud_insurance_api/          # Lambdaデプロイ済みFastAPI
    ├── cloud_mcp_server/             # AgentCore Gateway設定
    └── cloud_strands_insurance_agent/  # クラウドエージェント実装
```

## 🧪 ローカルプロトタイプ

クラウドにデプロイする前に、マシンで完全なアプリケーションスタックをローカルで実行してテストし、その機能を理解し、開発イテレーションを高速化できます。

### アーキテクチャ

ローカルプロトタイプは、マシンで実行される3つの主要コンポーネントで構成されています：

1. **ローカル保険API** (ポート8001): 保険データとビジネスロジックを含むFastAPIバックエンド
2. **ローカルMCPサーバー** (ポート8000): APIエンドポイントをツールとして公開するModel Context Protocolサーバー
3. **ローカルStrands保険エージェント**: Claudeを使用してユーザークエリを処理するインタラクティブエージェント

<!-- ローカルアーキテクチャ図は上記のテキストで説明 -->

### ローカルプロトタイプの実行

ローカルプロトタイプを開始するには、以下の手順に従ってください：

1. **保険APIの開始**
   ```bash
   cd local_prototype/local_insurance_api
   python -m venv venv
   source venv/bin/activate  # Windows: venv\Scripts\activate
   pip install -r requirements.txt
   python -m uvicorn server:app --port 8001
   ```

2. **MCPサーバーの開始**
   ```bash
   cd local_prototype/local_mcp_server
   python -m venv venv
   source venv/bin/activate  # Windows: venv\Scripts\activate
   pip install -r requirements.txt
   python server.py --http
   ```

3. **Strandsエージェントの実行**
   ```bash
   cd local_prototype/local_strands_insurance_agent
   python -m venv venv
   source venv/bin/activate  # Windows: venv\Scripts\activate
   pip install -r requirements.txt
   # Bedrockアクセス用のAWS認証情報を設定
   # (https://strandsagents.com/latest/documentation/docs/user-guide/quickstart/#configuring-credentials)を使用して認証情報を設定
   # インタラクティブエージェントの開始
   python interactive_insurance_agent.py
   ```

### ローカルプロトタイプのテスト

ローカルプロトタイプと複数の方法で相互作用できます：

- **APIテスト**: `http://localhost:8001/docs`にアクセスしてAPIエンドポイントを直接テスト
- **MCPインスペクター**: `npx @modelcontextprotocol/inspector`を実行してMCPツールを検査・テスト
- **チャットインターフェース**: ターミナルベースのチャットインターフェースを通じてエージェントと相互作用

### サンプルクエリ

```
You: 顧客cust-001について何か情報はありますか？
Agent: John Smithについて情報があります。彼は35歳で、Springfield, IL の123 Main Stに住んでいます。
       彼のメールアドレスはjohn.smith@example.comで、電話番号は555-123-4567です。

You: 彼はどんな車を持っていますか？
Agent: John Smithは2020年のToyota Camryを所有しています。

You: 2023年のHonda Civicの見積もりをもらえますか？
Agent: John Smithのプロフィールに基づくと、2023年のHonda Civicは包括補償で年間約1,245ドルかかります。
```

詳細な手順と例については、[ローカルプロトタイプREADME](local_prototype/README.md)をご覧ください。

## 🚦 はじめに

### 前提条件

- Bedrockアクセスが有効なAWSアカウント
- 適切な権限で設定されたAWS CLI
- Python 3.10以降
- Docker DesktopまたはFinchがインストール済み
- AWSサービスの基本的な理解

## 本番環境への移行

このセクションでは、ローカルプロトタイプの各コンポーネントがBedrock AgentCoreを使用した本番対応AWSソリューションにどのように変換されるかを説明します。

```
┌───────────────────┐      ┌───────────────────┐      ┌───────────────────┐
│                   │      │                   │      │                   │
│  Local Strands    │◄────►│  Local MCP Server │◄────►│  Local FastAPI    │
│  Agent            │      │                   │      │                   │
│                   │      │                   │      │                   │
└─────────┬─────────┘      └─────────┬─────────┘      └─────────┬─────────┘
          │                          │                          │
          ▼                          ▼                          ▼
┌───────────────────┐      ┌───────────────────┐      ┌───────────────────┐
│                   │      │                   │      │                   │
│  AgentCore        │◄────►│  AgentCore        │◄────►│  AWS Lambda +     │
│  Runtime Agent    │      │  Gateway          │      │  API Gateway      │
│                   │      │                   │      │                   │
└───────────────────┘      └───────────────────┘      └───────────────────┘
```

### 1. FastAPIからMangumを使用したAWS Lambda

ローカルFastAPIアプリケーションは、Mangumアダプターを使ってサーバーレスデプロイメント用に適応されます：

- **Mangum統合**: AWS LambdaとASGIアプリケーション間のブリッジとして機能
- **Lambdaハンドラー**: API GatewayイベントをFastAPI処理用に変換するラッパー関数
- **デプロイメントパッケージ**: Lambda実行用に依存関係とバンドルされたアプリケーションコード
- **API Gateway**: Lambda関数のHTTPエンドポイントと認証を提供
- **AWS SAMテンプレート**: 必要なすべてのリソースを定義するInfrastructure as Code

```python
# 例: lambda_function.py - FastAPIがLambdaに接続する方法
from local_insurance_api.app import app  # FastAPIアプリをインポート
from mangum import Mangum

# API Gatewayが呼び出すLambdaハンドラーを作成
handler = Mangum(app)
```

### 2. MCPサーバーからAgentCore Gateway

ローカルMCPサーバーはAWS Bedrock AgentCore Gatewayに置き換えられます：

- **OpenAPI統合**: GatewayはOpenAPI仕様からAPI操作をインポート
- **ツールスキーマ定義**: APIエンドポイントがMCPツールスキーマに変換
- **OAuth認証**: Cognito統合がセキュアなアクセス制御を提供
- **管理されたインフラストラクチャ**: AWSがスケーリング、可用性、観測性を処理
- **Gateway設定**: ツール定義がエージェントコードとの互換性を維持

```python
# 例: OpenAPI仕様を使用したGateway設定
gateway = client.create_mcp_gateway(
    name="InsuranceAPIGateway",
    authorizer_config=cognito_response["authorizer_config"]
)

# OpenAPI仕様を使用してAPIをMCPツールとして登録
client.create_mcp_gateway_target(
    gateway=gateway,
    name="InsuranceAPITarget",
    target_type="openApiSchema",
    target_payload={"inlinePayload": json.dumps(openapi_spec)}
)
```

### 3. StrandsエージェントからAgentCore Runtime

ローカルStrandsエージェントがAWS Bedrock AgentCore Runtimeにデプロイされます：

- **BedrockAgentCoreApp**: StrandsエージェントをAWSにデプロイするためのラッパー
- **エントリーポイントデコレーション**: クラウド実行用の標準化されたインターフェース
- **コンテナデプロイメント**: 一貫した実行のためのDockerイメージとしてパッケージ化
- **管理されたスケーリング**: AWSがエージェント実行リソースを処理
- **認証情報管理**: ゲートウェイとモデルへのセキュアなアクセス

```python
# 例: AgentCore用に適応されたStrandsエージェント
from bedrock_agentcore.runtime import BedrockAgentCoreApp

app = BedrockAgentCoreApp()

@app.entrypoint
def invoke(payload):
    user_input = payload.get("user_input")
    
    # 認証を使用してGateway MCPに接続
    gateway_client = MCPClient(lambda: streamablehttp_client(
        gateway_url, 
        headers={"Authorization": f"Bearer {access_token}"}
    ))
    
    with gateway_client:
        # 標準的なStrandsエージェントコードがここで続行
        tools = gateway_client.list_tools_sync()
        agent = Agent(model="claude-3", tools=tools, system_prompt="...")
        return agent(user_input)
```

### 本番デプロイメント手順

完全な本番ソリューションをデプロイするには、以下の手順に従ってください：

1. **保険APIのデプロイ**
   ```bash
   cd agentcore_app/cloud_insurance_api/deployment
   ./deploy.sh
   ```
   これにより、AWS SAMを使用してFastAPIアプリケーションがAWS Lambdaにデプロイされ、API Gatewayエンドポイントが作成されます。

2. **AgentCore GatewayでMCPサーバーをセットアップ**
   ```bash
   cd ../cloud_mcp_server
   # 環境変数の設定
   cp .env.example .env
   # 設定で.envを編集
   
   # セットアップスクリプトの実行
   python agentcore_gateway_setup_openapi.py
   ```
   これにより、OAuth認証を使ってAPI操作をMCPツールとして公開するAgentCore Gatewayが作成されます。

3. **Strands保険エージェントのデプロイ**
   ```bash
   cd ../cloud_strands_insurance_agent
   
   # 環境変数の設定
   cp .env.example .env
   # MCP URLとトークンで.envを更新
   
   # 設定とデプロイ
   agentcore configure -e "agentcore_strands_insurance_agent.py" \
     --name insurance_agent_strands \
     -er <execution-role-arn>
   
   # クラウドにデプロイ
   agentcore launch
   ```
   これにより、エージェントがコンテナ化され、セキュアに呼び出し可能なAgentCore Runtimeにデプロイされます。

4. **デプロイメントのテスト**
   ```bash
   # 必要に応じてトークンをリフレッシュ
   cd 1_pre_req_setup/cognito_auth
   ./refresh_token.sh
   
   # エージェントの呼び出し
   agentcore invoke --bearer-token $BEARER_TOKEN \
     '{"user_input": "I need a quote for a 2023 Toyota Camry"}'
   ```

   ![Bedrock AgentCore Insurance App Conversation](agentcore_app/cloud_strands_insurance_agent/agentcore_strands_conversation.gif)

## 🔍 コンポーネント詳細
![Bedrock AgentCore Insurance App Architecture](agentcore-insurance-app-architecture.png)

### クラウド保険API

保険APIはFastAPIで構築され、AWS LambdaとAPI Gatewayを使用してサーバーレスアプリケーションとしてデプロイされます。保険関連情報を照会するためのエンドポイントを提供します。

[クラウド保険APIの詳細](agentcore_app/cloud_insurance_api/README.md)

### クラウドMCPサーバー

MCPサーバーコンポーネントは、LLM駆動エージェントが使用できるMCPツールとして保険API操作を公開するようにAWS Bedrock AgentCore Gatewayを設定します。

[クラウドMCPサーバーの詳細](agentcore_app/cloud_mcp_server/README.md)

### クラウドStrands保険エージェント

保険エージェントはStrands Agentsフレームワークを使用して構築され、AgentCore Gatewayに接続して自然言語対話を通じて保険支援を提供します。

[クラウドStrands保険エージェントの詳細](agentcore_app/cloud_strands_insurance_agent/README.md)

## 🛡️ セキュリティに関する考慮事項

- すべての機密設定が環境変数に保存
- 認証はCognitoでのOAuth 2.0を使用
- APIアクセスはAPIキーとベアラートークンで保護
- IAMロールは最小権限の原則に従う

## 📊 監視と観測性

本番実装には以下が含まれます：
- すべてのコンポーネント用のCloudWatchログ
- リクエストフロー用のX-Rayトレーシング
- エージェントパフォーマンス用のカスタムメトリクス
- エラー報告とアラート

## クリーンアップ

agentcoreアプリの使用が完了したら、以下の手順でリソースをクリーンアップしてください：

1. **GatewayとTargetsの削除**:
   ```bash
   # Gateway IDの取得
   aws bedrock-agentcore-control list-gateways
   
   # ゲートウェイのターゲットリスト
   aws bedrock-agentcore-control list-gateway-targets --gateway-identifier your-gateway-id
   
   # 最初にターゲットを削除（ゲートウェイ全体を削除しない場合）
   aws bedrock-agentcore-control delete-gateway-target --gateway-identifier your-gateway-id --target-id your-target-id
   
   # ゲートウェイの削除（これにより関連するすべてのターゲットも削除されます）
   aws bedrock-agentcore-control delete-gateway --gateway-identifier your-gateway-id
   ```

2. **AgentCore Runtimeリソースの削除**:
   ```bash
   # エージェントランタイムのリスト
   aws bedrock-agentcore-control list-agent-runtimes
   
   # エージェントランタイムエンドポイントのリスト
   aws bedrock-agentcore-control list-agent-runtime-endpoints --agent-runtime-identifier your-agent-runtime-id
   
   # エージェントランタイムエンドポイントの削除
   aws bedrock-agentcore-control delete-agent-runtime-endpoint --agent-runtime-identifier your-agent-runtime-id --agent-runtime-endpoint-identifier your-endpoint-id
   
   # エージェントランタイムの削除
   aws bedrock-agentcore-control delete-agent-runtime --agent-runtime-identifier your-agent-runtime-id
   ```

3. **OAuth2 Credential Providersの削除**:
   ```bash
   # OAuth2認証情報プロバイダーのリスト
   aws bedrock-agentcore-control list-oauth2-credential-providers
   
   # OAuth2認証情報プロバイダーの削除
   aws bedrock-agentcore-control delete-oauth2-credential-provider --credential-provider-identifier your-provider-id
   ```

4. **Cognitoリソース**:
   ```bash
   aws cognito-idp delete-user-pool-client --user-pool-id your-user-pool-id --client-id your-app-client-id
   aws cognito-idp delete-user-pool --user-pool-id your-user-pool-id
   ```

## 📝 ライセンス

このプロジェクトはApache License 2.0の下でライセンスされています - 詳細は[LICENSE](../../LICENSE)ファイルをご覧ください。

---

<p align="center">
  AWS Bedrock AgentCoreとStrands Agentsを使用して❤️で構築
</p>