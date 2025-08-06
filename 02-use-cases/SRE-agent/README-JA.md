# SRE エージェント - マルチエージェント サイトリライアビリティエンジニアリング アシスタント

## 概要

SRE エージェントは、インフラストラクチャの問題を調査するサイトリライアビリティエンジニア向けのマルチエージェントシステムです。Model Context Protocol（MCP）上に構築され、Amazon Nova と Anthropic Claude モデル（Claude は Amazon Bedrock または Anthropic から直接アクセス可能）によって駆動されるこのシステムは、問題の調査、ログの分析、パフォーマンスメトリクスの監視、運用手順の実行において協力する専門 AI エージェントを使用します。AgentCore Gateway は、MCP ツールとして利用可能なデータソースとシステムへのアクセスを提供します。この例は、本番環境用の Amazon Bedrock AgentCore Runtime を使用してエージェントをデプロイする方法も示しています。

### ユースケース詳細
| 情報               | 詳細                                                                                                                               |
|---------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| ユースケースタイプ | 対話型                                                                                                                             |
| エージェントタイプ | マルチエージェント                                                                                                                 |
| ユースケースコンポーネント | ツール（MCP ベース）、可観測性（ログ、メトリクス）、運用ランブック                                                               |
| ユースケース分野     | DevOps/SRE                                                                                                                         |
| 例の複雑度          | 上級                                                                                                                               |
| 使用 SDK            | Amazon Bedrock AgentCore SDK、LangGraph、MCP                                                                                       |

### ユースケースアーキテクチャ 

![Amazon Bedrock AgentCore を使った SRE サポートエージェント](docs/images/sre-agent-architecture.png)

### ユースケースの主要機能

- **マルチエージェント オーケストレーション**: 専門エージェントがリアルタイムストリーミングでインフラストラクチャ調査において協力
- **対話型インターフェース**: 単一クエリ調査とコンテキスト保持による対話型マルチターン会話
- **MCP ベース統合**: AgentCore Gateway が認証とヘルス監視による安全な API アクセスを提供
- **専門エージェント**: Kubernetes、ログ、メトリクス、運用手順用の 4 つのドメイン固有エージェント
- **ドキュメント作成とレポート**: 監査証跡付きの各調査に対する Markdown レポート生成

## 詳細ドキュメント

SRE エージェントシステムに関する包括的な情報については、以下の詳細ドキュメントを参照してください：

- **[システムコンポーネント](docs/system-components.md)** - 詳細なアーキテクチャとコンポーネント説明
- **[専門エージェント](docs/specialized-agents.md)** - 4 つの専門エージェントそれぞれの詳細な能力
- **[設定](docs/configuration.md)** - 環境変数、エージェント、ゲートウェイの完全な設定ガイド
- **[セキュリティ](docs/security.md)** - 本番デプロイメントのためのセキュリティベストプラクティスと考慮事項
- **[デモ環境](docs/demo-environment.md)** - デモシナリオ、データカスタマイゼーション、テストセットアップ
- **[利用例](docs/example-use-cases.md)** - 詳細なウォークスルーと対話型トラブルシューティング例
- **[検証](docs/verification.md)** - 根拠の真実の検証とレポート検証
- **[開発](docs/development.md)** - テスト、コード品質、貢献ガイドライン
- **[デプロイメントガイド](docs/deployment-guide.md)** - Amazon Bedrock AgentCore Runtime の完全なデプロイメントガイド

## 前提条件

| 要件 | 説明 |
|-------------|-------------|
| Python 3.12+ と `uv` | Python ランタイムとパッケージマネージャー。[ユースケースセットアップ](#ユースケースセットアップ)を参照 |
| Amazon EC2 インスタンス | 推奨: `t3.xlarge` 以上 |
| 有効な SSL 証明書 | **⚠️ 重要:** Amazon Bedrock AgentCore Gateway は **HTTPS エンドポイントでのみ動作します**。例えば、Amazon EC2 を [no-ip.com](https://www.noip.com/) に登録し、[letsencrypt.org](https://letsencrypt.org/) から証明書を取得するか、他のドメイン登録と SSL 証明書プロバイダーを使用できます。[ユースケースセットアップ](#ユースケースセットアップ)セクションで `BACKEND_DOMAIN` としてドメイン名と証明書パスが必要です |
| EC2 インスタンスポート設定 | 必要なインバウンドポート（443、8011-8014）。[EC2 インスタンスポート設定](docs/ec2-port-configuration.md)を参照 |
| BedrockAgentCoreFullAccess ポリシーを持つ IAM ロール | AgentCore サービスに必要な権限と信頼ポリシー。[BedrockAgentCoreFullAccess ポリシーを持つ IAM ロール](docs/auth.md)を参照 |
| アイデンティティプロバイダー（IDP） | JWT 認証用の Amazon Cognito、Auth0、または Okta。自動 Cognito セットアップには `deployment/setup_cognito.sh` を使用。[認証セットアップ](docs/auth.md#identity-provider-configuration)を参照 |

> **注意:** すべての前提条件がユースケースセットアップに進む前に完了している必要があります。適切な SSL 証明書、IAM 権限、アイデンティティプロバイダー設定がないとセットアップは失敗します。

## ユースケースセットアップ

```bash
# リポジトリをクローン
git clone https://github.com/awslabs/amazon-bedrock-agentcore-samples
cd amazon-bedrock-agentcore-samples/02-use-cases/SRE-agent

# 仮想環境を作成してアクティベート
uv venv --python 3.12
source .venv/bin/activate  # Windows の場合: .venv\Scripts\activate

# SRE エージェントと依存関係をインストール
uv pip install -e .

# 環境変数を設定
cp .env.example sre_agent/.env
# sre_agent/.env を編集して Anthropic API キーを追加：
# ANTHROPIC_API_KEY=sk-ant-your-key-here

# OpenAPI テンプレートがバックエンドドメインで置き換えられ、.yaml として保存される
BACKEND_DOMAIN=api.mycompany.com ./backend/openapi_specs/generate_specs.sh

# サーバーバインディング用の EC2 インスタンスプライベート IP を取得
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600" -s)
PRIVATE_IP=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  -s http://169.254.169.254/latest/meta-data/local-ipv4)

# SSL でデモバックエンドサーバーを開始
cd backend
./scripts/start_demo_backend.sh \
  --host $PRIVATE_IP  \
  --ssl-keyfile /etc/ssl/private/privkey.pem \
  --ssl-certfile /etc/ssl/certs/fullchain.pem
cd ..

# AgentCore Gateway を作成して設定
cd gateway
./create_gateway.sh
./mcp_cmds.sh
cd ..

# エージェント設定でゲートウェイ URI を更新
GATEWAY_URI=$(cat gateway/.gateway_uri)
sed -i "s|uri: \".*\"|uri: \"$GATEWAY_URI\"|" sre_agent/config/agent_config.yaml

# ゲートウェイアクセストークンを .env ファイルにコピー
sed -i '/^GATEWAY_ACCESS_TOKEN=/d' sre_agent/.env
echo "GATEWAY_ACCESS_TOKEN=$(cat gateway/.access_token)" >> sre_agent/.env
```

## 実行手順

### 単一クエリモード
```bash
# 特定の Pod 問題を調査
sre-agent --prompt "Why are the payment-service pods crash looping?"

# パフォーマンス劣化を分析
sre-agent --prompt "Investigate high latency in the API gateway over the last hour"

# エラーパターンを検索
sre-agent --prompt "Find all database connection errors in the last 24 hours"
```

### インタラクティブモード
```bash
# インタラクティブ会話を開始
sre-agent --interactive

# インタラクティブモードで利用可能なコマンド：
# /help     - 利用可能なコマンドを表示
# /agents   - 利用可能な専門エージェントをリスト
# /history  - 会話履歴を表示
# /save     - 現在の会話を保存
# /clear    - 会話履歴をクリア
# /exit     - インタラクティブセッションを終了
```

#### 高度なオプション
```bash
# Amazon Bedrock を使用
sre-agent --provider bedrock --query "Check cluster health"

# 調査レポートをカスタムディレクトリに保存
sre-agent --output-dir ./investigations --query "Analyze memory usage trends"

# 特定のプロファイルで Amazon Bedrock を使用
AWS_PROFILE=production sre-agent --provider bedrock --interactive
```

## 開発から本番デプロイメントフロー

SRE エージェントは、ローカル開発から Amazon Bedrock AgentCore Runtime での本番環境まで、構造化されたデプロイメントプロセスに従います。詳細な手順については、**[デプロイメントガイド](docs/deployment-guide.md)** を参照してください。

```
ステップ 1: ローカル開発
┌─────────────────────────────────────────────────────────────────────┐
│  Python パッケージ開発 (sre_agent/)                                │
│  └─> CLI でローカルテスト: uv run sre-agent --prompt "..."         │
│      └─> エージェントが MCP プロトコル経由で AgentCore Gateway に接続 │
└─────────────────────────────────────────────────────────────────────┘
                                    ↓
ステップ 2: コンテナ化
┌─────────────────────────────────────────────────────────────────────┐
│  agent_runtime.py 追加 (FastAPI サーバーラッパー)                   │
│  └─> Dockerfile 作成 (AgentCore 用 ARM64)                          │
│      └─> deployment/build_and_deploy.sh スクリプトを使用            │
└─────────────────────────────────────────────────────────────────────┘
                                    ↓
ステップ 3: ローカルコンテナテスト
┌─────────────────────────────────────────────────────────────────────┐
│  ビルド: LOCAL_BUILD=true ./deployment/build_and_deploy.sh          │
│  └─> 実行: docker run -p 8080:8080 sre_agent:latest                │
│      └─> テスト: curl -X POST http://localhost:8080/invocations     │
│          └─> コンテナが同じ AgentCore Gateway に接続               │
└─────────────────────────────────────────────────────────────────────┘
                                    ↓
ステップ 4: 本番デプロイメント
┌─────────────────────────────────────────────────────────────────────┐
│  ビルド & プッシュ: ./deployment/build_and_deploy.sh                │
│  └─> コンテナを Amazon ECR にプッシュ                              │
│      └─> deployment/deploy_agent_runtime.py が AgentCore にデプロイ │
│          └─> テスト: uv run python deployment/invoke_agent_runtime.py │
│              └─> 本番エージェントが本番 Gateway を使用             │
└─────────────────────────────────────────────────────────────────────┘

重要なポイント：
• コアエージェントコード（sre_agent/）は変更なし
• deployment/ フォルダーにすべてのデプロイメント固有ユーティリティを含む
• 同じエージェントが環境設定経由でローカルと本番で動作
• AgentCore Gateway がすべての段階で MCP ツールアクセスを提供
```

## Amazon Bedrock AgentCore Runtime でのエージェントデプロイ

本番デプロイメントでは、SRE エージェントを Amazon Bedrock AgentCore Runtime に直接デプロイできます。これにより、エンタープライズグレードのセキュリティと監視を持つエージェント実行のスケーラブルで管理された環境が提供されます。

AgentCore Runtime デプロイメントは以下をサポートします：
- 自動スケーリングを伴う**コンテナベースデプロイメント**
- **複数の LLM プロバイダー**（Amazon Bedrock または Anthropic Claude）
- トラブルシューティングと開発用の**デバッグモード**
- 異なるデプロイメント段階用の**環境ベース設定**
- AWS IAM と環境変数による**安全な認証情報管理**

ローカルテスト、コンテナビルド、本番デプロイメントを含む完全なステップバイステップ手順については、**[デプロイメントガイド](docs/deployment-guide.md)** を参照してください。

## クリーンアップ手順

```bash
# すべてのデモサーバーを停止
cd backend
./scripts/stop_demo_backend.sh
cd ..

# 仮想環境を削除
deactivate
rm -rf .venv

# 生成されたファイルをクリーンアップ
rm -rf reports/
rm -rf gateway/.gateway_uri gateway/.access_token
rm -rf sre_agent/.env
```

## 免責事項
このリポジトリで提供される例は、実験および教育目的のみです。概念と技術を実証しますが、本番環境での直接使用を意図したものではありません。[プロンプトインジェクション](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-injection.html)から保護するために Amazon Bedrock Guardrails を配置していることを確認してください。

**重要な注意**: [`backend/data`](backend/data) のデータは合成的に生成されており、backend ディレクトリには実際の SRE エージェントバックエンドがどのように動作するかを示すスタブサーバーが含まれています。本番環境では、これらの実装は実際のシステムに接続し、ベクターデータベースを使用し、他のデータソースと統合する実際の実装に置き換える必要があります。