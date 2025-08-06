# プロトタイプから本格運用へ：AWS Bedrock AgentCoreを使ったエージェント アプリケーション

<div align="center">
  <img src="https://img.shields.io/badge/Python-3.10+-blue.svg" alt="Python 3.10+"/>
  <img src="https://img.shields.io/badge/AWS-Bedrock_AgentCore-orange.svg" alt="AWS Bedrock AgentCore"/>
  <img src="https://img.shields.io/badge/Strands-Agents-green.svg" alt="Strands Agents"/>
  <img src="https://img.shields.io/badge/FastAPI-0.100.0+-purple.svg" alt="FastAPI"/>
</div>

このプロジェクトは、APIツールを持つローカルエージェントベースのMCPアプリケーションを、本格運用のメリットを得るためにAWSクラウドに移行する方法を示しています。実装にはAWS Bedrock AgentCoreを活用しており、認証、観測可能性、管理されたランタイム環境などの機能を通じて、エージェント アプリケーションの本格運用をサポートします。

## 概要

`production_using_agentcore`フォルダには、`local_prototype`フォルダにある保険アプリケーション コンポーネントのクラウドベース実装が含まれており、AWS Bedrock AgentCoreサービスを活用するように変更されています。

## アーキテクチャ
![Bedrock AgentCore保険アプリ アーキテクチャ](../agentcore-insurance-app-architecture.png)

ソリューションは3つの主要コンポーネントで構成されています：

1. **クラウド保険API** (`cloud_insurance_api/`)：API Gateway統合を持つAWS Lambda関数としてデプロイされたFastAPIアプリケーション
2. **クラウドMCPサーバー** (`cloud_mcp_server/`)：AWS Bedrock AgentCore Gatewayを通じて保険APIをMCPツールとして公開するゲートウェイ設定
3. **クラウドStrands保険エージェント** (`cloud_strands_insurance_agent/`)：AgentCore Gatewayに接続して保険APIツールにアクセスするStrandsベースのエージェント実装

## 前提条件

- 適切な権限を持つAWSアカウント
- 管理者アクセスで設定されたAWS CLI
- Python 3.10以上
- Docker DesktopまたはFinchがインストールされている（ローカルテストとデプロイメント用）
- AWSアカウントでBedrockモデルアクセスが有効になっている
- jqコマンドラインJSONプロセッサ

## セットアップ プロセス

セットアップには3つの主要ステップがあります：

### 1. クラウド保険APIのデプロイ

最初のステップは、AWS LambdaとAPI Gatewayを使用してサーバーレスアプリケーションとして保険APIをデプロイすることです：

```bash
cd cloud_insurance_api/deployment
chmod +x ./deploy.sh
./deploy.sh
```

これにより、AWS SAMを使用してFastAPIアプリケーションがデプロイされ、Lambda関数、API Gateway、および権限を含むすべての必要なリソースが作成されます。

### 2. AgentCore GatewayでのMCPサーバーのセットアップ

次に、保険APIをMCPツールとして公開するようにAWS Bedrock AgentCore Gatewayを設定します：

```bash
cd ../cloud_mcp_server

# OpenAPI統合でAgentCore Gatewayをセットアップ
python agentcore_gateway_setup_openapi.py
```

これにより、保険APIツールにアクセスするためのMCPエンドポイントを提供するOAuth認証を持つAgentCore Gatewayが作成されます。

### 3. Strands保険エージェントのデプロイ

最後に、MCPゲートウェイと相互作用するエージェントをデプロイします：

```bash
cd ../cloud_strands_insurance_agent

# IAM実行ロールのセットアップ
cd 1_pre_req_setup/iam_roles_setup
./setup_role.sh

# Cognito認証のセットアップ
cd ../cognito_auth
./setup_cognito.sh

# プロジェクトルートに戻る
cd ../../

# 例の環境ファイルをコピーして値を編集
cp .env_example .env
nano .env

# セットアップ出力またはAWSコンソールからロールARNを取得
ROLE_ARN=$(aws iam get-role --role-name BedrockAgentCoreExecutionRole --query 'Role.Arn' --output text)

# エージェントを設定
agentcore configure -e "agentcore_strands_insurance_agent.py" \
  --name insurance_agent_strands \
  -er $ROLE_ARN

# .envファイルから環境変数を読み込み
source .env

# クラウドにデプロイ
agentcore launch \
  -env MCP_SERVER_URL=$MCP_SERVER_URL \
  -env MCP_ACCESS_TOKEN=$MCP_ACCESS_TOKEN
```

## コンポーネント詳細

### クラウド保険API

以下のためのエンドポイントを提供するAWS Lambda関数としてデプロイされたFastAPIアプリケーション：
- 顧客情報の取得
- 車両情報の検索
- 保険見積もりの生成
- ポリシー管理

詳細については[クラウド保険API README](cloud_insurance_api/README.md)を参照してください。

### クラウドMCPサーバー

以下を行うAWS Bedrock AgentCoreのゲートウェイ設定を提供します：
- 保険APIエンドポイントをMCPツールとして公開
- Amazon Cognitoを使用した認証の設定
- ツールの実行環境のセットアップ
- CloudWatchを通じた観測可能性の有効化

### クラウドStrands保険エージェント

以下を行うStrandsベースのエージェント実装：
- AgentCore Gateway MCPサーバーに接続
- 利用可能なツールを使用して保険関連のクエリを処理
- 会話インターフェースを通じたユーザー インタラクションの処理

詳細については[クラウドStrands保険エージェント README](cloud_strands_insurance_agent/README.md)を参照してください。

![Bedrock AgentCore保険アプリ会話](cloud_strands_insurance_agent/agentcore_strands_conversation.gif)

## AgentCoreの利点

- **認証**：CognitoによるOAuth2を通じたセキュアなアクセス
- **観測可能性**：CloudWatchを通じた監視とログ記録
- **スケーラビリティ**：自動スケールする管理されたランタイム環境
- **コンプライアンス**：組み込みセキュリティ制御を持つ管理サービス
- **コスト最適化**：従量課金制価格モデル

## 使用例

```bash
# cognito_authディレクトリに移動
cd cloud_strands_insurance_agent/1_pre_req_setup/cognito_auth

# 必要に応じてトークンを更新
./refresh_token.sh

# トークンをエクスポート
export BEARER_TOKEN=$(jq -r '.bearer_token' cognito_config.json)

# プロジェクトルートに戻る
cd ../../

# エージェントを呼び出し
agentcore invoke --bearer-token $BEARER_TOKEN '{"user_input": "自動車保険の見積もりを手伝ってもらえますか？"}'
```

## トラブルシューティング

- **424 Failed Dependency**：CloudWatchでエージェントログを確認してください
- **トークンの期限切れ**：`./1_pre_req_setup/cognito_auth/refresh_token.sh`を実行して`.env`ファイルを更新してください
- **権限拒否**：実行ロールがBedrockモデルアクセス権を持っていることを確認してください
- **ローカルテストの失敗**：Dockerが動作していることを確認してください
- **認証エラー**：`.env`ファイルのMCP_ACCESS_TOKENが有効で期限切れでないことを確認してください
- **IAMロールエラー**：IAMロールが`iam_roles_setup/README.md`で指定されたすべての必要な権限を持っていることを確認してください
- **Cognito認証の問題**：トラブルシューティングについては`cognito_auth/README.md`のドキュメントを確認してください

## クリーンアップ

agentcoreアプリの使用が終了したら、以下の手順でリソースをクリーンアップしてください：

1. **ゲートウェイとターゲットの削除**：
   ```bash
   # ゲートウェイIDを取得
   aws bedrock-agentcore-control list-gateways
   
   # ゲートウェイのターゲットを一覧表示
   aws bedrock-agentcore-control list-gateway-targets --gateway-identifier your-gateway-id
   
   # 最初にターゲットを削除（ゲートウェイ全体を削除しない場合）
   aws bedrock-agentcore-control delete-gateway-target --gateway-identifier your-gateway-id --target-id your-target-id
   
   # ゲートウェイを削除（これにより関連するターゲットもすべて削除されます）
   aws bedrock-agentcore-control delete-gateway --gateway-identifier your-gateway-id
   ```

2. **AgentCoreランタイム リソースの削除**：
   ```bash
   # エージェント ランタイムを一覧表示
   aws bedrock-agentcore-control list-agent-runtimes
   
   # エージェント ランタイム エンドポイントを一覧表示
   aws bedrock-agentcore-control list-agent-runtime-endpoints --agent-runtime-identifier your-agent-runtime-id
   
   # エージェント ランタイム エンドポイントを削除
   aws bedrock-agentcore-control delete-agent-runtime-endpoint --agent-runtime-identifier your-agent-runtime-id --agent-runtime-endpoint-identifier your-endpoint-id
   
   # エージェント ランタイムを削除
   aws bedrock-agentcore-control delete-agent-runtime --agent-runtime-identifier your-agent-runtime-id
   ```

3. **OAuth2認証情報プロバイダーの削除**：
   ```bash
   # OAuth2認証情報プロバイダーを一覧表示
   aws bedrock-agentcore-control list-oauth2-credential-providers
   
   # OAuth2認証情報プロバイダーを削除
   aws bedrock-agentcore-control delete-oauth2-credential-provider --credential-provider-identifier your-provider-id
   ```

4. **Cognitoリソース**：
   ```bash
   aws cognito-idp delete-user-pool-client --user-pool-id your-user-pool-id --client-id your-app-client-id
   aws cognito-idp delete-user-pool --user-pool-id your-user-pool-id
   ```

## 次のステップ

- より多くの保険商品とサービスの追加
- 会話記憶の実装
- エージェント インタラクション用のウェブUIの追加
- 保険サービス向上のための追加データソースとの統合