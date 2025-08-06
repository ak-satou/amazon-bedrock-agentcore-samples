# Amazon Bedrock AgentCore を使用した AWS サポートエージェント

Amazon Bedrock AgentCore 上に構築された AWS サポート対話型 AI システムで、OAuth2 認証、MCP（モデル制御プロトコル）統合、包括的な AWS サービス操作を特徴としています。

## デモ

![AWS サポートエージェントのデモ](images/demo-agentcore.gif)

*Amazon Bedrock AgentCore を使用した AWS サポートエージェントのインタラクティブデモンストレーション*

## アーキテクチャ概要

![AWS サポートエージェントアーキテクチャ](images/architecture.jpg)

このシステムは、安全で分散されたアーキテクチャに従っています：

1. **チャットクライアント** は Okta OAuth2 経由でユーザー認証を行い、JWT トークンとともに質問を送信
2. **AgentCore ランタイム** はトークンを検証し、会話を処理し、セッションメモリを維持
3. **AgentCore ゲートウェイ** は MCP プロトコルを通じて安全なツールアクセスを提供
4. **AWS Lambda ターゲット** は適切な認証で AWS サービス操作を実行
5. **AgentCore アイデンティティ** はワークロード認証とトークン交換を管理

## 主要機能

- 🔐 **エンタープライズ認証**: JWT トークン検証を伴う Okta OAuth2
- 🤖 **デュアルエージェントアーキテクチャ**: FastAPI（DIY）と BedrockAgentCoreApp（SDK）の両方の実装
- 🧠 **会話メモリ**: AgentCore Memory による永続的なセッションストレージ
- 🔗 **MCP 統合**: 標準化されたツール通信プロトコル
- 🛠️ **20+ AWS ツール**: AWS サービス全体にわたる包括的な読み取り専用操作
- 📊 **本番環境対応**: 完全なデプロイメント自動化とインフラストラクチャ管理

## プロジェクト構造

```
AgentCore/
├── README.md                           # このドキュメント
├── requirements.txt                    # Python 依存関係
├── config/                             # 🔧 設定管理
│   ├── static-config.yaml              # 手動設定
│   └── dynamic-config.yaml             # ランタイム生成設定
│
├── shared/                             # 🔗 共有設定ユーティリティ
│   ├── config_manager.py               # 中央設定管理
│   └── config_validator.py             # 設定検証
│
├── chatbot-client/                     # 🤖 クライアントアプリケーション
│   ├── src/client.py                   # インタラクティブチャットクライアント
│   └── README.md                       # クライアント固有のドキュメント
│
├── agentcore-runtime/                  # 🚀 メインランタイム実装
│   ├── src/
│   │   ├── agents/                     # エージェント実装
│   │   │   ├── diy_agent.py            # FastAPI 実装
│   │   │   └── sdk_agent.py            # BedrockAgentCoreApp 実装
│   │   ├── agent_shared/               # 共有エージェントユーティリティ
│   │   │   ├── auth.py                 # JWT 検証
│   │   │   ├── config.py               # エージェント設定
│   │   │   ├── mcp.py                  # MCP クライアント
│   │   │   ├── memory.py               # 会話メモリ
│   │   │   └── responses.py            # レスポンス形式設定
│   │   └── utils/
│   │       └── memory_manager.py       # メモリ管理ユーティリティ
│   ├── deployment/                     # 🚀 デプロイメントスクリプト
│   │   ├── 01-prerequisites.sh         # IAM ロールと前提条件
│   │   ├── 02-create-memory.sh         # AgentCore Memory セットアップ
│   │   ├── 03-setup-oauth-provider.sh  # OAuth2 プロバイダー設定
│   │   ├── 04-deploy-mcp-tool-lambda.sh # MCP Lambda デプロイメント
│   │   ├── 05-create-gateway-targets.sh # ゲートウェイとターゲットセットアップ
│   │   ├── 06-deploy-diy.sh            # DIY エージェントデプロイメント
│   │   ├── 07-deploy-sdk.sh            # SDK エージェントデプロイメント
│   │   ├── 08-delete-runtimes.sh       # ランタイムクリーンアップ
│   │   ├── 09-delete-gateways-targets.sh # ゲートウェイクリーンアップ
│   │   ├── 10-delete-mcp-tool-deployment.sh # MCP クリーンアップ
│   │   ├── 11-delete-memory.sh         # メモリクリーンアップ
│   │   ├── Dockerfile.diy              # DIY エージェントコンテナ
│   │   ├── Dockerfile.sdk              # SDK エージェントコンテナ
│   │   ├── deploy-diy-runtime.py       # DIY デプロイメント自動化
│   │   └── deploy-sdk-runtime.py       # SDK デプロイメント自動化
│   ├── gateway-ops-scripts/            # 🌉 ゲートウェイ管理
│   │   └── [ゲートウェイ CRUD 操作]
│   ├── runtime-ops-scripts/            # ⚙️ ランタイム管理
│   │   └── [ランタイムとアイデンティティ管理]
│   └── tests/local/                    # 🧪 ローカルテストスクリプト
│
├── mcp-tool-lambda/                    # 🔧 AWS ツール Lambda
│   ├── lambda/mcp-tool-handler.py      # MCP ツール実装
│   ├── mcp-tool-template.yaml          # CloudFormation テンプレート
│   └── deploy-mcp-tool.sh              # Lambda デプロイメントスクリプト
│
├── okta-auth/                          # 🔐 認証セットアップ
│   ├── OKTA-OPENID-PKCE-SETUP.md      # Okta 設定ガイド
│   ├── iframe-oauth-flow.html          # OAuth フロー テスト
│   └── setup-local-nginx.sh           # ローカル開発セットアップ
│
└── docs/                               # 📚 ドキュメント
    └── images/
        └── agentcore-implementation.jpg # アーキテクチャ図
```

## クイックスタート

### 前提条件

- 適切な権限で設定された **AWS CLI**
- **Docker** と Docker Compose のインストール
- **Python 3.11+** のインストール
- **Okta 開発者アカウント** とアプリケーションの設定
- **yq** YAML 処理ツール（オプション、フォールバック利用可能）

### 1. 設定

特定の設定で設定ファイルを編集します：

```bash
# AWS と Okta の設定
vim config/static-config.yaml

# 更新する主要設定：
# - aws.account_id: あなたの AWS アカウント ID
# - aws.region: 優先する AWS リージョン（このプロジェクトは us-east-1 でテスト）
# - okta.domain: あなたの Okta ドメイン
# - okta.client_credentials.client_id: あなたの Okta クライアント ID
# - okta.client_credentials.client_secret: 環境変数で設定

# 重要: IAM ポリシーファイルをあなたの AWS アカウント ID で更新
# これらのファイルで YOUR_AWS_ACCOUNT_ID プレースホルダーを置換：
sed -i "s/YOUR_AWS_ACCOUNT_ID/$(aws sts get-caller-identity --query Account --output text)/g" \
  agentcore-runtime/deployment/bac-permissions-policy.json \
  agentcore-runtime/deployment/bac-trust-policy.json

# アカウント ID が正しく更新されたことを確認
grep -n "$(aws sts get-caller-identity --query Account --output text)" \
  agentcore-runtime/deployment/bac-permissions-policy.json \
  agentcore-runtime/deployment/bac-trust-policy.json
```

### 2. インフラストラクチャのデプロイ

デプロイメントスクリプトを順番に実行：

```bash
cd agentcore-runtime/deployment

# AWS 前提条件とロールをセットアップ
./01-prerequisites.sh

# 会話ストレージ用の AgentCore Memory を作成
./02-create-memory.sh

# Okta OAuth2 プロバイダーをセットアップ
./03-setup-oauth-provider.sh

# MCP ツール Lambda 関数をデプロイ
./04-deploy-mcp-tool-lambda.sh

# AgentCore ゲートウェイとターゲットを作成
./05-create-gateway-targets.sh

# エージェントをデプロイ（一つまたは両方を選択）
./06-deploy-diy.sh    # FastAPI 実装
./07-deploy-sdk.sh    # BedrockAgentCoreApp 実装
```

### 3. システムテスト

#### ローカルテストスクリプトの実行
```bash
# ローカルエージェント機能をテスト
cd agentcore-runtime/tests/local
./test-diy-simple.sh    # ローカルツールで DIY エージェントをテスト
./test-sdk-mcp.sh       # MCP ゲートウェイ統合で SDK エージェントをテスト
```

#### 直接 curl コマンドでテスト

**DIY エージェント（ポート 8080）：**
```bash
# DIY エージェントを開始
cd agentcore-runtime/tests/local && ./test-diy-simple.sh

# curl でテスト（SSE ストリームを返す）
curl -X POST http://localhost:8080/invocations \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "What time is it?",
    "session_id": "test-session-123",
    "actor_id": "user"
  }'

# テキストレスポンスのみを抽出
curl -s -X POST http://localhost:8080/invocations \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Hello!", "session_id": "demo", "actor_id": "user"}' \
  | grep '"type":"text_delta"' \
  | sed 's/.*"content": *"\([^"]*\)".*/\1/' \
  | tr -d '\n'

# ヘルスチェック
curl http://localhost:8080/ping
# 戻り値: {"status":"healthy","agent_type":"diy"}
```

**SDK エージェント（ポート 8081）：**
```bash
# SDK エージェントを開始
cd agentcore-runtime/tests/local && ./test-sdk-mcp.sh

# curl でテスト（異なる SSE 形式を返す）
curl -X POST http://localhost:8081/invocations \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "What time is it?",
    "session_id": "test-session-456",
    "actor_id": "user"
  }'

# ヘルスチェック
curl http://localhost:8081/ping
# 戻り値: {"status":"Healthy","time_of_last_update":1753954747}
```

#### インタラクティブチャットクライアントの使用
```bash
cd chatbot-client/src
python client.py
```

## コンポーネント詳細

### エージェント実装

#### DIY エージェント（FastAPI）
- **フレームワーク**: Uvicorn を使用した FastAPI
- **エンドポイント**: `/invoke` 
- **機能**: リクエスト/レスポンス処理を完全に制御するカスタム実装
- **コンテナ**: `agentcore-runtime/deployment/Dockerfile.diy`

#### SDK エージェント（BedrockAgentCoreApp）
- **フレームワーク**: BedrockAgentCoreApp SDK
- **機能**: 組み込み最適化によるネイティブ AgentCore 統合
- **コンテナ**: `agentcore-runtime/deployment/Dockerfile.sdk`

### 認証フロー

1. **ユーザー認証**: ユーザーは Okta OAuth2 PKCE フローで認証
2. **トークン検証**: AgentCore ランタイムは Okta のディスカバリーエンドポイントを使用して JWT トークンを検証
3. **ワークロードアイデンティティ**: ランタイムがユーザートークンをワークロードアクセストークンに交換
4. **サービス認証**: ワークロードトークンが AgentCore ゲートウェイとツールで認証

### メモリ管理

- **ストレージ**: AgentCore Memory サービスが永続的な会話ストレージを提供
- **セッション管理**: 各会話が相互作用間でセッションコンテキストを維持
- **保持**: 会話データの設定可能な保持期間
- **プライバシー**: ユーザーセッションごとのメモリ分離

### ツール統合

システムは MCP Lambda を通じて 20+ の AWS サービスツールを提供：

- **EC2**: インスタンス管理と監視
- **S3**: バケット操作とポリシー分析
- **Lambda**: 関数管理と監視
- **CloudFormation**: スタック操作とリソース追跡
- **IAM**: ユーザー、ロール、ポリシー管理
- **RDS**: データベースインスタンス監視
- **CloudWatch**: メトリクス、アラーム、ログ分析
- **Cost Explorer**: コスト分析と最適化
- **その他多数...**

## 設定管理

### 静的設定（`config/static-config.yaml`）
手動設定を含む：
- AWS アカウントとリージョン設定
- Okta OAuth2 設定
- エージェントモデル設定
- ツールスキーマと定義

### 動的設定（`config/dynamic-config.yaml`）
デプロイメント中に自動生成：
- ランタイム ARN とエンドポイント
- ゲートウェイ URL と識別子
- OAuth プロバイダー設定
- メモリサービス詳細

### 設定マネージャー
`shared/config_manager.py` が提供：
- 統合設定アクセス
- 環境固有設定
- 検証とエラー処理
- 下位互換性

## 開発

### ローカルテスト

```bash
# 完全なデプロイメントなしでエージェントをローカルでテスト
cd agentcore-runtime/tests/local

# シンプルな会話で DIY エージェントをテスト
./test-diy-simple.sh

# MCP ツールで SDK エージェントをテスト
./test-sdk-mcp.sh

# MCP ゲートウェイ機能をテスト
./test-diy-ec2-mcp.sh
```

### コンテナ開発

両方のエージェントは標準化されたコンテナ構造に従います：

```
/app/
├── shared/                     # プロジェクト全体のユーティリティ
├── agent_shared/              # エージェント固有のヘルパー
├── config/                    # 設定ファイル
│   ├── static-config.yaml
│   └── dynamic-config.yaml
├── [agent].py                 # エージェント実装
└── requirements.txt
```

### 新しいツールの追加

1. **ツールスキーマを定義** `config/static-config.yaml` で
2. **ツールロジックを実装** `mcp-tool-lambda/lambda/mcp-tool-handler.py` で
3. **ゲートウェイターゲットを更新** gateway-ops-scripts を使用
4. **統合をテスト** ローカルテストスクリプトで

## 監視と運用

### ランタイム管理
```bash
cd agentcore-runtime/runtime-ops-scripts

# デプロイされたすべてのランタイムをリスト
python runtime_manager.py list

# ランタイム詳細を確認
python runtime_manager.py get <runtime_id>

# OAuth フローをテスト
python oauth_test.py test-config
```

### ゲートウェイ管理
```bash
cd agentcore-runtime/gateway-ops-scripts

# すべてのゲートウェイをリスト
python list-gateways.py

# ゲートウェイターゲットを確認
python list-targets.py

# ゲートウェイ設定を更新
python update-gateway.py --gateway-id <id> --name "New Name"
```

### ログ分析
- **CloudWatch ログ**: エージェントランタイムログ
- **リクエストトレース**: 完全なリクエスト/レスポンスログ
- **エラー監視**: 一元化されたエラー追跡
- **パフォーマンスメトリクス**: レスポンス時間とリソース使用量

## クリーンアップ

デプロイされたすべてのリソースを削除するには：

```bash
cd agentcore-runtime/deployment

# ランタイムを削除
./08-delete-runtimes.sh

# ゲートウェイとターゲットを削除  
./09-delete-gateways-targets.sh

# MCP Lambda を削除
./10-delete-mcp-tool-deployment.sh

# メモリを削除し設定をクリア
./11-delete-memory.sh

# 完全クリーンアップ（オプション）
./12-cleanup-everything.sh
```

## セキュリティベストプラクティス

- **トークン検証**: すべてのリクエストが Okta JWT に対して検証
- **最小権限**: IAM ロールが最小権限の原則に従う
- **暗号化**: すべてのデータが転送中および保存時に暗号化
- **ネットワークセキュリティ**: 制御されたアクセスを持つプライベートネットワーキング
- **監査ログ**: すべての操作の包括的な監査証跡

## トラブルシューティング

### 一般的な問題

1. **トークン検証の失敗**
   - `static-config.yaml` の Okta 設定を確認
   - JWT オーディエンスと発行者設定を確認
   - `oauth_test.py` でテスト

2. **メモリアクセスの問題**
   - AgentCore Memory がデプロイされて利用可能であることを確認
   - `dynamic-config.yaml` のメモリ設定を確認
   - ローカルスクリプトでメモリ操作をテスト

3. **ツール実行の失敗**
   - MCP Lambda デプロイメント状態を確認
   - ゲートウェイターゲット設定を確認
   - MCP クライアントで個別ツールをテスト

4. **コンテナ起動の問題**
   - Docker ビルドログを確認
   - requirements.txt の互換性を確認
   - コンテナヘルスエンドポイントを確認

### ヘルプの取得

1. CloudWatch で**デプロイメントログを確認**
2. runtime-ops-scripts で**診断スクリプトを実行**
3. config_manager 検証で**設定を確認**
4. ローカルテストスクリプトを使用して**コンポーネントを個別にテスト**

## ライセンス

このプロジェクトは教育および実験目的です。組織のポリシーと AWS サービス規約への準拠を確実にしてください。