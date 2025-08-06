# Okta OpenID Connect PKCEセットアップガイド

AWS Support Agent AgentCoreシステム用のOkta OAuth2認証を設定するための完全ガイドです。

## 概要

このガイドでは、AgentCoreシステム用のOkta OAuth2認証を設定し、ユーザー認証（PKCEフロー）とマシン間認証（クライアントクレデンシャルフロー）の両方をサポートします。

## 前提条件

- **Oktaデベロッパーアカウント** ([developer.okta.com](https://developer.okta.com)で無料)
- **AWSアカウント** (Bedrock AgentCoreアクセス付き)
- **nginx** (テスト用にローカルにインストール)
- **Python 3.11+** (システム実行用)

## Oktaアプリケーションセットアップ

### 1. OIDCアプリケーションの作成

1. **Okta Developer Consoleにログイン**
2. **移動先**: Applications → Applications → Create App Integration
3. **選択**: OIDC - OpenID Connect → Single-Page Application (SPA)
4. **アプリケーション設定**:
   ```
   App name: aws-support-agent-client
   Grant types: ✅ Authorization Code, ✅ Refresh Token
   Sign-in redirect URIs: 
     - http://localhost:8080/callback
     - http://localhost:8080/okta-auth/
   Sign-out redirect URIs: http://localhost:8080/
   Controlled access: Allow everyone in your organization to access
   ```
5. **保存**してアプリケーションを保存し、**Client ID**をメモする

### 2. APIスコープの設定

1. **移動先**: Security → API → Authorization Servers → default
2. **スコープの存在確認**:
   - `openid` - OpenID Connectに必要
   - `profile` - ユーザープロファイル情報  
   - `email` - ユーザーメールアドレス
3. **カスタムスコープを追加**:
   - **名前**: `api`
   - **説明**: API access for AgentCore
   - **Include in public metadata**: ✅

### 3. マシン間アプリケーションの作成

AgentCoreワークロード認証用：

1. **新しいアプリを作成**: OIDC - OpenID Connect → Web Application
2. **設定**:
   ```
   App name: aws-support-agent-m2m
   Grant types: ✅ Client Credentials
   Client authentication: ✅ Client secret
   ```
3. **保存**し、**Client ID**と**Client Secret**をメモする

## プロジェクト設定

### 1. 静的設定の更新

`config/static-config.yaml`を編集：

```yaml
# Okta OAuth2 Configuration
okta:
  domain: "your-domain.okta.com"  # Oktaドメインに置き換え
  
  # OAuth2認証サーバー設定
  authorization_server: "default"
  
  # クライアントクレデンシャルフロー（アプリ間）のクライアント設定
  client_credentials:
    client_id: "YOUR_M2M_CLIENT_ID"      # M2Mアプリから
    client_secret: "${OKTA_CLIENT_SECRET}"  # 環境変数で設定
    scope: "api"
  
  # ユーザー認証（PKCEフロー）のクライアント設定
  user_auth:
    client_id: "YOUR_SPA_CLIENT_ID"      # SPAアプリから
    audience: "YOUR_SPA_CLIENT_ID"       # client_idと同じ
    redirect_uri: "http://localhost:8080/callback"
    scope: "openid profile email"
  
  # JWTトークン設定
  jwt:
    audience: "YOUR_SPA_CLIENT_ID"       # SPAクライアントID
    issuer: "https://your-domain.okta.com/oauth2/default"
    discovery_url: "https://your-domain.okta.com/oauth2/default/.well-known/openid-configuration"
    cache_duration: 300
    refresh_threshold: 60

# AgentCore JWT Authorizer設定
agentcore:
  jwt_authorizer:
    discovery_url: "https://your-domain.okta.com/oauth2/default/.well-known/openid-configuration"
    allowed_audience: 
      - "YOUR_SPA_CLIENT_ID"
```

### 2. 環境変数の設定

```bash
# Oktaクライアントシークレットを設定
export OKTA_CLIENT_SECRET="your_m2m_client_secret"

# オプション：デフォルトと異なる場合はAWSプロファイルを設定
export AWS_PROFILE="your-aws-profile"
```

## ローカルテストセットアップ

### 1. nginxの設定（オプション）

ローカルPKCEテスト用に、nginxサーバーブロックをnginx設定に追加する必要があります：

```bash
# プロジェクトディレクトリに移動
cd /path/to/your/AgentCore/project

# ステップ1：nginx設定ファイルのパスを更新
# プレースホルダーパスを実際のプロジェクトパスに置き換え
sed -i '' "s|/path/to/your/AgentCore|$(pwd)|g" okta-auth/nginx/okta-local.conf

# パスが正しく更新されたか確認
cat okta-auth/nginx/okta-local.conf | grep "root\|alias"

# ステップ2：重要 - okta-local.confには完全なnginx.confではなく、サーバーブロックのみが含まれています
# このサーバーブロックを既存のnginx設定に統合する必要があります。

# 手動統合手順：
# 1. nginx.confの場所を探す：
#    - macOS (Homebrew): /usr/local/etc/nginx/nginx.conf
#    - Linux: /etc/nginx/nginx.conf
#    - Docker: /etc/nginx/nginx.conf

# 2. 現在のnginx.confのバックアップを作成
sudo cp /usr/local/etc/nginx/nginx.conf /usr/local/etc/nginx/nginx.conf.backup  # macOS
# sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.backup                    # Linux

# 3. メインのnginx.confファイルを編集用に開く：
sudo nano /usr/local/etc/nginx/nginx.conf  # macOS
# sudo nano /etc/nginx/nginx.conf          # Linux

# 4. nginx.confでhttp {}ブロックを見つける
# 5. okta-auth/nginx/okta-local.confからサーバーブロック全体をコピー
# 6. 既存のhttp {}ブロック内（閉じ括弧}の前）に貼り付ける
# 7. ファイルを保存して閉じる

# 注意：setup-local-nginx.shスクリプトは機能しません。
# okta-local.confを完全なnginx設定として使用しようとして失敗するためです。

# ステップ3：nginxをテストしてリロード
# nginx設定の構文エラーをテスト
sudo nginx -t

# テストに合格した場合、nginx設定をリロード
sudo nginx -s reload

# nginxが実行中でポート8080をリッスンしているか確認
curl http://localhost:8080/health
# 戻り値：healthy
```

### 2. OAuthフローのテスト

提供されたHTMLテストページを使用：

1. **設定を更新** `okta-auth/iframe-oauth-flow.html`内：
   ```javascript
   const config = {
     clientId: 'YOUR_SPA_CLIENT_ID',
     redirectUri: 'http://localhost:8080/okta-auth/',
     authorizationEndpoint: 'https://your-domain.okta.com/oauth2/default/v1/authorize',
     tokenEndpoint: 'https://your-domain.okta.com/oauth2/default/v1/token',
     scope: 'openid profile email',
   };
   ```

2. **ブラウザを開く**: http://localhost:8080/okta-auth/
3. **「Login with Okta」をクリック**して認証を完了
4. **アクセストークンをコピー**してテスト用に使用

## システムのデプロイとテスト

### 1. インフラストラクチャーのデプロイ

```bash
# AgentCoreシステムをデプロイ
cd agentcore-runtime/deployment
./01-prerequisites.sh
./02-create-memory.sh
./03-setup-oauth-provider.sh  # Okta設定を使用
./04-deploy-mcp-tool-lambda.sh
./05-create-gateway-targets.sh
./06-deploy-diy.sh
./07-deploy-sdk.sh
```

### 2. 認証のテスト

```bash
# OAuthプロバイダーセットアップをテスト
cd agentcore-runtime/runtime-ops-scripts
python oauth_test.py test-config

# 期待される出力：
# ✅ M2M token obtained successfully
# ✅ OAuth2 token obtained successfully
```

### 3. エンドツーエンドのテスト

```bash
# チャットクライアントを使用
cd chatbot-client/src
python client.py --interactive

# またはcurlで直接テスト（ステップ2のトークンを使用）
curl -X POST http://localhost:8080/invocations \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{
    "prompt": "List my S3 buckets",
    "session_id": "test-session",
    "actor_id": "user"
  }'
```

## 認証フロー

### ユーザー認証（PKCE）
1. **ユーザー** → Oktaログイン → JWTトークン
2. **チャットクライアント** → AgentCoreランタイム（JWTあり）
3. **ランタイムがJWTを検証** Okta discoveryエンドポイントに対して
4. **ランタイムがユーザーリクエストを処理**

### マシン間（M2M）
1. **ランタイム** → AgentCore Identity → ワークロードトークン
2. **ワークロードトークン** → OAuthプロバイダー → M2Mトークン  
3. **M2Mトークン** → AgentCore Gateway → ツールアクセス
4. **ツール** → AWSサービス → 結果

## トラブルシューティング

### よくある問題

**1. 無効なクライアントエラー**
```bash
# static-config.yamlのクライアントIDを確認
grep -A 10 "client_credentials:" config/static-config.yaml
grep -A 10 "user_auth:" config/static-config.yaml
```

**2. トークン検証失敗**
```bash
# Okta discovery endpoint を確認
curl https://your-domain.okta.com/oauth2/default/.well-known/openid-configuration

# トークン検証をテスト
python agentcore-runtime/runtime-ops-scripts/oauth_test.py test-config
```

**3. M2M認証失敗**
```bash
# 環境変数を確認
echo $OKTA_CLIENT_SECRET

# M2Mフローを特別にテスト
python agentcore-runtime/runtime-ops-scripts/oauth_test.py workload-token bac-diy
```

**4. Gateway接続問題**
```bash
# gatewayとtargetがデプロイされているか確認
cd agentcore-runtime/gateway-ops-scripts
python list-gateways.py
python list-targets.py
```

### デバッグコマンド

```bash
# デプロイされたOAuthプロバイダーを確認
cd agentcore-runtime/runtime-ops-scripts
python credentials_manager.py list

# ランタイムログを表示
cd agentcore-runtime/runtime-ops-scripts  
python runtime_manager.py list
python logs_manager.py get <runtime-id>

# 設定検証をテスト
cd /path/to/your/AgentCore/project
python -c "from shared.config_manager import AgentCoreConfigManager; AgentCoreConfigManager().validate()"
```

### ログ分析

**AgentCoreランタイムログ**で以下を確認：
- `✅ OAuth initialized with provider`
- `✅ M2M token obtained successfully`
- `✅ MCP client ready`

**よくあるエラーパターン**：
- `❌ OAuth provider not found` → デプロイメントステップ03を確認
- `❌ Invalid client credentials` → クライアントシークレット環境変数を確認
- `❌ Token validation failed` → static-config.yamlのJWT設定を確認

## セキュリティベストプラクティス

1. **シークレットをコミットしない** - クライアントシークレットには常に環境変数を使用
2. **本番環境ではHTTPS使用** - 本番デプロイメント用にリダイレクトURIを更新
3. **トークンを定期的にローテート** - Oktaで適切なトークン有効期間を設定
4. **トークンオーディエンスを検証** - JWTオーディエンス検証を厳密にする
5. **認証を監視** - Oktaシステムログを定期的に確認

## 本番デプロイメント

本番環境では以下を更新：

1. **リダイレクトURI**を本番ドメインに
2. **環境変数**をデプロイメントパイプラインで
3. **ネットワークセキュリティ**を適切にアクセス制限
4. **トークン有効期間**をセキュリティ要件に基づいて

---

追加のヘルプについては、以下を参照してください：
- [Okta Developer Documentation](https://developer.okta.com/docs/)
- [AgentCore Documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/agentcore.html)
- [プロジェクトREADME](../README.md) 完全なシステムセットアップ用