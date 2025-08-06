# 設定

## 環境変数

SREエージェントは、機密設定値に環境変数を使用します。`sre_agent/`ディレクトリに以下の必要な変数を含む`.env`ファイルを作成してください：

```bash
# 必須：Claudeモデルアクセス用のAPIキー
# Anthropic直接アクセス用：
ANTHROPIC_API_KEY=sk-ant-api-key-here

# Amazon Bedrockアクセス用：
AWS_DEFAULT_REGION=us-east-1
AWS_PROFILE=your-profile-name  # またはAWS_ACCESS_KEY_IDとAWS_SECRET_ACCESS_KEYを使用

# 必須：AgentCore Gateway認証
GATEWAY_ACCESS_TOKEN=your-gateway-token-here  # gateway設定で生成

# オプション：デバッグとログ記録
LOG_LEVEL=INFO  # オプション：DEBUG、INFO、WARNING、ERROR
DEBUG=false     # 詳細出力用のデバッグモードを有効化
```

**注意**：SREエージェントは、プロジェクトルートではなく`sre_agent/`ディレクトリの`.env`ファイルを探します。これにより、モジュラー設定管理が可能になります。

## エージェント設定

エージェントの動作は`sre_agent/config/agent_config.yaml`で設定されます。このファイルは、エージェントと利用可能なツール間のマッピング、およびLLMパラメータを定義します：

```yaml
# エージェントからツールへのマッピング
agents:
  kubernetes_agent:
    name: "Kubernetes Infrastructure Agent"
    description: "Kubernetes運用とトラブルシューティングを専門とする"
    tools:
      - get_pod_status
      - get_deployment_status
      - get_cluster_events
      - get_resource_usage
      - get_node_status

  logs_agent:
    name: "Application Logs Agent"
    description: "ログ分析とパターン検出の専門家"
    tools:
      - search_logs
      - get_error_logs
      - analyze_log_patterns
      - get_recent_logs
      - count_log_events

  metrics_agent:
    name: "Performance Metrics Agent"
    description: "パフォーマンスメトリクスとトレンドを分析"
    tools:
      - get_performance_metrics
      - get_error_rates
      - get_resource_metrics
      - get_availability_metrics
      - analyze_trends

  runbooks_agent:
    name: "Operational Runbooks Agent"
    description: "運用手順とガイドを提供"
    tools:
      - search_runbooks
      - get_incident_playbook
      - get_troubleshooting_guide
      - get_escalation_procedures
      - get_common_resolutions

# すべてのエージェントで利用可能なグローバルツール
global_tools:
  - x-amz-bedrock-agentcore-search  # AgentCore検索ツール
  
# Gateway設定
gateway:
  uri: "https://your-gateway-url.com"  # 設定時に更新
```

## Gateway環境変数

AgentCore Gatewayには、認証用の追加環境変数が必要です。`gateway/`ディレクトリに以下を含む`.env`ファイルを作成してください：

```bash
# 必須：認証情報プロバイダー認証用のバックエンドAPIキー
BACKEND_API_KEY=your-backend-api-key-here

# オプション：環境変数でconfig.yaml値を上書き
# ACCOUNT_ID=123456789012
# REGION=us-east-1
# ROLE_NAME=your-role-name
# GATEWAY_NAME=MyAgentCoreGateway
# CREDENTIAL_PROVIDER_NAME=sre-agent-api-key-credential-provider
```

**注意**：`BACKEND_API_KEY`は、`create_gateway.sh`スクリプトが認証情報プロバイダーサービスで認証するために使用されます。

## Gateway設定

AgentCore Gatewayは`gateway/config.yaml`で設定されます。この設定は設定スクリプトで管理されますが、カスタマイズできます：

```yaml
# AgentCore Gateway設定テンプレート
# このファイルをconfig.yamlにコピーして、環境固有の設定で更新してください

# AWS設定
account_id: "YOUR_ACCOUNT_ID"
region: "us-east-1"
role_name: "YOUR_ROLE_NAME"
endpoint_url: "https://bedrock-agentcore-control.us-east-1.amazonaws.com"
credential_provider_endpoint_url: "https://us-east-1.prod.agent-credential-provider.cognito.aws.dev"

# Cognito設定
user_pool_id: "YOUR_USER_POOL_ID"
client_id: "YOUR_CLIENT_ID"

# S3設定
s3_bucket: "your-agentcore-schemas-bucket"
s3_path_prefix: "devops-multiagent-demo"  # OpenAPIスキーマファイル用のパスプレフィックス

# プロバイダー設定
# このARNはcreate_gateway.shがcreate_credentials_provider.pyを実行するときに自動生成されます
provider_arn: "arn:aws:bedrock-agentcore:REGION:ACCOUNT_ID:token-vault/default/apikeycredentialprovider/YOUR_PROVIDER_NAME"

# Gateway設定
gateway_name: "MyAgentCoreGateway"
gateway_description: "API統合用のAgentCore Gateway"

# Target設定
target_description: "OpenAPIスキーマ用のS3ターゲット"
```