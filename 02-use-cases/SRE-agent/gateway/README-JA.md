# ゲートウェイコンポーネント

このディレクトリには、SREエージェント用のMCP（Model Context Protocol）ゲートウェイ管理ツールが含まれています。

## 📁 ファイル

- `main.py` - AWS AgentCore Gatewayの作成と管理のためのAgentCoreゲートウェイ管理ツール
- `mcp_cmds.sh` - MCPゲートウェイ操作とセットアップ用のシェルスクリプト
- `generate_token.py` - ゲートウェイ認証用のJWTトークン生成
- `openapi_s3_target_cognito.sh` - S3とCognito統合を使用したOpenAPIターゲット追加スクリプト
- `config.yaml` - ゲートウェイ設定ファイル
- `config.yaml.example` - サンプル設定テンプレート
- `.env` - ゲートウェイセットアップ用の環境変数
- `.env.example` - サンプル環境変数テンプレート

## 🚀 ゲートウェイセットアップ

### ステップバイステップセットアップ

1. **ゲートウェイを設定**（設定をコピーして編集）:
   ```bash
   cd gateway
   cp config.yaml.example config.yaml
   cp .env.example .env
   # 特定の設定でconfig.yamlと.envを編集
   ```

2. **ゲートウェイを作成**:
   ```bash
   ./create_gateway.sh
   ```

3. **ゲートウェイをテスト**:
   ```bash
   ./mcp_cmds.sh
   
   # デバッグのためにログファイルに出力をキャプチャ:
   ./mcp_cmds.sh 2>&1 | tee mcp_cmds.log
   ```

このセットアッププロセスは以下を実行します：
- MCPゲートウェイインフラストラクチャを設定
- 適切な認証とトークン管理でゲートウェイを作成
- ゲートウェイ機能をテストし、セットアップを検証

## 🔧 コンポーネント

### ゲートウェイ管理（`main.py`）
メインゲートウェイ管理ツールは以下の機能を提供：
- AWS AgentCore Gatewayの作成と管理
- MCPプロトコル統合のサポート
- JWT認証の処理
- S3またはインラインスキーマからのOpenAPIターゲット追加

### MCPコマンド（`mcp_cmds.sh`）
以下を含むゲートウェイセットアッププロセスを調整するシェルスクリプト：
- ゲートウェイ作成
- 設定検証
- サービス登録
- ヘルスチェック

### トークン生成（`generate_token.py`）
ゲートウェイ認証用のJWTトークン生成ユーティリティ：
```bash
python generate_token.py --config config.yaml
```

### OpenAPI統合（`openapi_s3_target_cognito.sh`）
OpenAPI仕様をS3ストレージとCognito認証と統合するスクリプト。

## 🔍 使用方法

### クイックリファレンス
1. `config.yaml`で設定を構成
2. ゲートウェイを作成: `./create_gateway.sh`
3. ゲートウェイをテスト: `./mcp_cmds.sh`
4. デバッグの場合、出力をキャプチャ: `./mcp_cmds.sh 2>&1 | tee mcp_cmds.log`
5. ゲートウェイが実行中でアクセス可能であることを検証
6. クライアント認証のために必要に応じてトークンを生成

### 開発モード
開発とテストのために、コンポーネントを個別に実行することもできます：

```bash
# トークンを生成
python generate_token.py

# 特定の設定でゲートウェイを作成
python main.py --config config.yaml

# OpenAPIターゲットを追加
./openapi_s3_target_cognito.sh
```

## ⚠️ 重要な注意事項

- 常にgatewayディレクトリから`mcp_cmds.sh`を実行してください
- セットアップ前に`config.yaml`が適切に設定されていることを確認してください
- SREエージェント調査を開始する前にゲートウェイが実行されている必要があります
- 認証トークンを安全に保持し、定期的にローテーションしてください
- ログファイル（*.log）は自動的にgitによって無視されます - デバッグのために安全に作成できます

## 🔗 統合

ゲートウェイがセットアップされ実行されると、SREエージェントコアシステムがインフラストラクチャAPIとツールにアクセスするために接続するMCPエンドポイントを提供します。