# 認証と認可の設定

このドキュメントでは、AgentCore Gatewayに必要なIAM権限とアイデンティティプロバイダー（IDP）設定の設定要件について説明します。

## IAM権限の設定

### AWS管理ポリシー

簡素化されたセットアップのため、AWSはBedrock AgentCore操作に必要なすべての権限を含む管理ポリシーを提供しています：

**ポリシー名**: `BedrockAgentCoreFullAccess`

この管理ポリシーは、AgentCore runtimeで使用されるIAMロールに接続する必要があります。以下が含まれます：
- すべてのbedrock-agentcore権限
- 必要なIAM:PassRole権限
- スキーマストレージ用のS3アクセス
- その他の必要なサービス権限

### 信頼ポリシーの要件

IAMロールには、Bedrock AgentCoreサービスがロールを引き受けることを許可する信頼ポリシーが含まれている必要があります：

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "bedrock-agentcore.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

### コアGateway権限（代替案）

管理ポリシーではなく詳細な権限を希望する場合は、Gateway TargetまたはGatewayでのCRUDL操作の呼び出し、InvokeTool API、およびListTool用のこのポリシーを使用してください：

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "bedrock-agentcore:*",
                "iam:PassRole"
            ],
            "Resource": "*"
        }
    ]
}
```

### S3スキーマアクセス

S3のAPIスキーマでターゲットを作成するために必要なポリシー（上記のポリシーと同じ呼び出し元アイデンティティに接続）：

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": "*"
        }
    ]
}
```

### Lambda Target権限

LambdaがGateway targetタイプの場合、実行ロールはlambdaを呼び出す権限を持つ必要があります：

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "lambda:InvokeFunction",
            "Resource": "arn:aws:lambda:us-west-2:<account-with-lambda>:function:TestLambda"
        }
    ]
}
```

### Smithy Target権限

Gateway TargetがSmithy Targetタイプの場合：
- 実行ロールには、呼び出したいツール/APIのAWS権限が含まれている必要があります
- 例：S3用のgateway targetを追加 → ロールに関連するS3権限を追加

AgentCoreサービスのベータアカウントがロールを引き受けることを信頼する必要があります：

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "bedrock-agentcore.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

### クロスアカウントLambdaアクセス

Lambdaが別のアカウントにある場合、lambda関数にリソースベースポリシー（RBP）を設定します：

```json
{
    "Version": "2012-10-17",
    "Id": "default",
    "Statement": [
        {
            "Sid": "cross-account-access",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::<gateway-account>:role/AgentCoreBetaLambdaExecuteRole"
            },
            "Action": "lambda:InvokeFunction",
            "Resource": "arn:aws:lambda:us-west-2:<account-with-lambda>:function:TestLambda"
        }
    ]
}
```

## アイデンティティプロバイダーの設定

### 重要：Cognito vs Auth0/Okta認証の違い

**AgentCore Gateway設定における重要な区別：**

| プロバイダー | 使用するJWTクレーム | Gateway設定 | トークンに含まれるもの |
|----------|---------------|---------------------|----------------|
| **Amazon Cognito** | `client_id` | `allowedClients: ["client-id"]` | ❌ `aud`クレームなし |
| **Auth0** | `aud` | `allowedAudience: ["audience"]` | ✅ `aud`クレームあり |
| **Okta** | `aud` | `allowedAudience: ["audience"]` | ✅ `aud`クレームあり |

**なぜこれが重要か：**
- Cognitoクライアント認証情報トークンには`aud`（オーディエンス）クレームが含まれていません
- `allowedAudience`を持つAgentCore GatewayはCognitoトークンを拒否します（401エラー）
- Cognitoの場合、アプリクライアントIDで`allowedClients`を使用する必要があります
- Auth0/Oktaの場合、API識別子で`allowedAudience`を使用する必要があります

### 1. Amazon Cognito設定

#### オプションA：自動設定（推奨）

完全なエンドツーエンドCognito設定には、自動化スクリプトを使用してください：

```bash
cd deployment
./setup_cognito.sh
```

このスクリプトは以下を行います：
- ユーザープールとドメインを作成
- スコープ付きリソースサーバーを作成
- 認証情報付きアプリクライアントを作成
- 必要なすべての変数で`.env`ファイルを生成
- gateway設定用の設定詳細を提供

#### オプションB：手動設定

Cognitoを手動で設定することを希望する場合は、以下の手順に従ってください：

#### ユーザープール作成

マシン間ユーザープールを作成：

```bash
# ユーザープール作成
aws cognito-idp create-user-pool \
    --region us-west-2 \
    --pool-name "test-agentcore-user-pool"

# プールIDを取得するためにユーザープールをリスト
aws cognito-idp list-user-pools \
    --region us-west-2 \
    --max-results 60
```

#### ディスカバリーURL形式

```
https://cognito-idp.us-west-2.amazonaws.com/<UserPoolId>/.well-known/openid-configuration
```

#### リソースサーバー作成

```bash
aws cognito-idp create-resource-server \
    --region us-west-2 \
    --user-pool-id <UserPoolId> \
    --identifier "test-agentcore-server" \
    --name "TestAgentCoreServer" \
    --scopes '[{"ScopeName":"read","ScopeDescription":"Read access"}, {"ScopeName":"write","ScopeDescription":"Write access"}]'
```

#### ユーザープールドメイン作成

**重要**：アクセストークンを取得する前にドメインを作成する必要があります。

```bash
# ユーザープールドメイン作成（トークンエンドポイントに必要）
aws cognito-idp create-user-pool-domain \
    --region us-west-2 \
    --user-pool-id <UserPoolId> \
    --domain "your-unique-domain-name"

# 注意：ドメイン名は全AWSアカウント間でグローバルにユニークである必要があります
# 例："sre-agent-demo-12345"または"mycompany-sre-agent"
```

#### クライアント作成

```bash
aws cognito-idp create-user-pool-client \
    --region us-west-2 \
    --user-pool-id <UserPoolId> \
    --client-name "test-agentcore-client" \
    --generate-secret \
    --allowed-o-auth-flows client_credentials \
    --allowed-o-auth-scopes "test-agentcore-server/read" "test-agentcore-server/write" \
    --allowed-o-auth-flows-user-pool-client \
    --supported-identity-providers "COGNITO"
```

#### アクセストークン取得

```bash
# URLではUserPoolIdではなくドメイン名を使用
curl --http1.1 -X POST https://<your-domain-name>.auth.us-west-2.amazoncognito.com/oauth2/token \
    -H "Content-Type: application/x-www-form-urlencoded" \
    -d "grant_type=client_credentials&client_id=<ClientId>&client_secret=<ClientSecret>"
```

**注意**：UserPoolIdではなく、前のステップで作成したドメイン名を使用してください。URL形式は以下の通りです：
`https://<domain-name>.auth.<region>.amazoncognito.com/oauth2/token`

#### サンプルCognitoトークンクレーム

```json
{
    "sub": "<>",
    "token_use": "access",
    "scope": "default-m2m-resource-server-<>/read",
    "auth_time": 1749679004,
    "iss": "https://cognito-idp.us-west-2.amazonaws.com/us-west-<>",
    "exp": 1749682604,
    "iat": 1749679004,
    "version": 2,
    "jti": "<>",
    "client_id": "<>"
}
```

#### Cognito認証設定

```json
{
    "authorizerConfiguration": {
        "customJWTAuthorizer": {
            "allowedClients": ["<ClientId>"],
            "discoveryUrl": "https://cognito-idp.us-west-2.amazonaws.com/<UserPoolId>/.well-known/openid-configuration"
        }
    }
}
```

### 2. Auth0設定

#### 設定手順

1. **APIを作成**（リソースサーバーと1:1マッピング）
2. **アプリケーションを作成**（リソースサーバーのクライアントとして機能）
3. **API識別子を設定** API > Settings（オーディエンスクレームに追加）
4. **スコープを設定** API > Scopesセクション

#### ディスカバリーURL形式

```
https://<your-domain>/.well-known/openid-configuration
```

#### アクセストークン取得

```bash
curl --request POST \
    --url https://dev-<your-domain>.us.auth0.com/oauth/token \
    --header 'content-type: application/json' \
    --data '{
        "client_id":"YOUR_CLIENT_ID",
        "client_secret":"YOUR_CLIENT_SECRET",
        "audience":"gateway123",
        "grant_type":"client_credentials",
        "scope": "invoke:gateway"
    }'
```

#### サンプルAuth0トークンクレーム

```json
{
    "iss": "https://dev-<>.us.auth0.com/",
    "sub": "<>",
    "aud": "gateway123",
    "iat": 1749741913,
    "exp": 1749828313,
    "scope": "invoke:gateway read:gateway",
    "jti": "<>",
    "client_id": "<>",
    "permissions": [
        "invoke:gateway",
        "read:gateway"
    ]
}
```

#### Auth0認証設定

```json
{
    "authorizerConfiguration": {
        "customJWTAuthorizer": {
            "allowedAudience": ["gateway123"],
            "discoveryUrl": "https://dev-<your-domain>.us.auth0.com/.well-known/openid-configuration"
        }
    }
}
```

### 3. Okta設定

#### 設定手順

1. **アプリケーション作成** クライアント認証情報許可タイプで
   - [Oktaドキュメント](https://developer.okta.com/docs/guides/implement-grant-type/clientcreds/main/)に従う
   - 必要に応じて無料トライアルにサインアップ

2. **アプリケーション設定**
   - Admin → Applications → シークレット付きクライアントを作成に移動
   - 「Require Demonstrating Proof of Possession (DPoP) header in token requests」を無効にする

3. **認証サーバー設定**
   - Admin → Security → APIに移動
   - デフォルト認証サーバーを使用
   - 追加スコープを追加（例：「InvokeGateway」）
   - オプションでアクセスポリシーとクレームを追加

4. **設定を取得**
   - デフォルト認証サーバーのメタデータURIを取得（ディスカバリーURI）
   - JWT認証設定用のClientID/Secretを取得

## トークン検証

デバッグ中にベアラートークンをデコードして検証するには、[jwt.io](https://jwt.io/)を使用してください。

## 環境変数

アイデンティティプロバイダーの設定後、`.env`ファイルでこれらの環境変数を設定してください：

```bash
# Cognito用
COGNITO_DOMAIN=https://your-domain-name.auth.us-west-2.amazoncognito.com
COGNITO_CLIENT_ID=your-client-id
COGNITO_CLIENT_SECRET=your-client-secret

# 'your-domain-name'は以下で作成したドメインです：
# aws cognito-idp create-user-pool-domain --domain "your-domain-name"

# Auth0用
COGNITO_DOMAIN=https://dev-yourdomain.us.auth0.com
COGNITO_CLIENT_ID=your-client-id
COGNITO_CLIENT_SECRET=your-client-secret
```

## トラブルシューティング

### よくある401「Invalid Bearer token」エラー

**問題：** GatewayがHTTP 401を「Invalid Bearer token」メッセージで返す。

**根本原因：** トークンクレームとgateway設定の不一致。

**解決手順：**

1. **JWTトークンをデコード** [jwt.io](https://jwt.io/)を使用してクレームを検査
2. **トークンクレームを確認：**
   - Cognitoトークン：`client_id`クレームを探す（`aud`クレームなし）
   - Auth0/Oktaトークン：`aud`クレームを探す
3. **gateway設定がトークンと一致することを確認：**
   ```bash
   # トークンにclient_idはあるがaudクレームがない場合（Cognito）
   python main.py "Gateway" --allowed-clients "your-client-id" ...
   
   # トークンにaudクレームがある場合（Auth0/Okta）
   python main.py "Gateway" --allowed-audience "your-audience" ...
   ```
4. **よくある修正：**
   - **Cognitoユーザー：** `--allowed-audience`ではなく`--allowed-clients`を使用
   - **Auth0ユーザー：** `--allowed-clients`ではなく`--allowed-audience`を使用
   - **クライアントIDを確認：** 完全に一致する必要があります（大文字小文字を区別）
   - **オーディエンスを確認：** API識別子と完全に一致する必要があります

### その他のよくある問題

- トークンエンドポイントが機能しない場合は、ブラウザでディスカバリーURLを確認して正しい`token_endpoint`を見つけてください
- トークン要求とgateway設定間でオーディエンス値が一致することを確認してください
- IDPでスコープが適切に設定されていることを確認してください
- ディスカバリーURLがアクセス可能で有効なOpenID設定を返すことを確認してください
- Cognito用：アプリクライアントで`client_credentials`許可タイプが有効になっていることを確認してください