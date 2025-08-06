# デバイス管理システム

## 概要

このユースケースは、Amazon Bedrock AgentCoreを使用した包括的なデバイス管理システムを実装しています。AWS Lambdaを通じてIoTデバイス、WiFiネットワーク、ユーザー、アクティビティを管理するための統一インターフェースを提供し、ユーザーが自然言語を通じてスマートホームデバイスと相互作用できるようにします。

| 情報 | 詳細 |
|-------------|---------|
| ユースケースタイプ | 会話型 |
| エージェントタイプ | 単一エージェント |
| ユースケースコンポーネント | ツール、Gateway |
| ユースケース分野 | IoT/スマートホーム |
| 例の複雑さ | 中級 |
| 使用SDK | Amazon Bedrock AgentCore SDK、boto3 |

## ユースケースアーキテクチャ

![Device Management Architecture](./images/device-management-architecture.png)

### プロセスフロー

1. **ユーザーとの相互作用**: ユーザーはAmazon QまたはMCPプロトコルをサポートする他のクライアントを通じてデバイス管理システムと相互作用します。

2. **リクエスト処理**:
   - リクエストがAmazon Bedrock AgentCoreに送信されます
   - AgentCoreがリクエストを適切なGatewayにルーティングします
   - GatewayがリクエストをGateway Target（Lambda関数）に転送します

3. **ツール実行**:
   - Lambda関数がリクエストを受信し、実行する適切なツールを特定します
   - 関数がDynamoDBテーブルと相互作用してCRUD操作を実行します
   - 結果がMCPプロトコルに従って処理・フォーマットされます

4. **応答フロー**:
   - Lambda関数が応答をGatewayに返します
   - GatewayがAgentCoreに応答を転送します
   - AgentCoreがクライアントに応答を配信します

5. **データフロー**: すべてのデバイスデータ、ユーザー情報、設定はDynamoDBテーブルに保存され、そこから取得されます。

## ユースケースの主要機能

デバイス管理システムは以下の機能を提供します：

- システム内のデバイス一覧表示
- デバイス設定の取得
- デバイスのWiFiネットワーク一覧表示
- WiFiネットワーク設定の更新（SSID、セキュリティタイプ）
- アカウント内のユーザー一覧表示
- 期間内のユーザーアクティビティクエリ

## 前提条件

- Python 3.10+
- 適切な権限を持つAWSアカウント
- boto3とAmazon Bedrock AgentCore SDK
- 設定されたアプリクライアントを持つCognito User Pool
- Cognitoドメイン（オプションですが、ホストUIには推奨）
- 適切な権限を持つBedrock Agent Core Gateway用IAMロール

IAMロールに以下のポリシーをアタッチしてください：
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "iam:PassRole",
                "bedrock-agentcore:*"
            ],
            "Resource": "*"
        }
    ]
}
```

## ユースケースセットアップ

### 1. 環境設定

プロジェクトルートに以下の変数を含む`.env`ファイルを作成してください：

```
# AWSとエンドポイント設定
AWS_REGION=<Your AWS Region>
ENDPOINT_URL=https://bedrock-agentcore-control.<REGION>.amazonaws.com

# Lambda設定
LAMBDA_ARN=arn:aws:lambda:<REGION>:your-account-id:function:DeviceManagementLambda

# ターゲット設定
GATEWAY_IDENTIFIER=your-gateway-identifier
TARGET_NAME=device-management-target
TARGET_DESCRIPTION=List, Update device management activities

# Gateway作成設定
COGNITO_USERPOOL_ID=your-cognito-userpool-id
COGNITO_APP_CLIENT_ID=your-cognito-app-client-id
GATEWAY_NAME=Device-Management-Gateway
ROLE_ARN=arn:aws:iam::your-account-id:role/YourGatewayRole
GATEWAY_DESCRIPTION=Device Management Gateway
```

### 2. 依存関係のインストール

必要なPythonパッケージをインストールしてください：

```bash
pip install -r requirements.txt
```

### 3. Lambda関数のデプロイ

提供されたデプロイメントスクリプトを使用してLambda関数をデプロイできます：

```bash
chmod +x deploy.sh
./deploy.sh
```

デプロイメントスクリプトは以下のアクションを実行します：
- 依存関係と一緒にLambda関数コードをパッケージ
- 必要な権限を持つIAMロールを作成（存在しない場合）
- Lambda関数を作成または更新
- 適切なメモリとタイムアウト設定で関数を設定

### 4. Gateway作成（既存のゲートウェイを使用したい場合はオプション）

新しいゲートウェイを作成するには、以下のスクリプトを実行してください：

```bash
python create_gateway.py
```

### 5. Gateway Targetの作成

Lambda関数をデプロイした後、Lambda関数をターゲットとして公開するためのゲートウェイターゲットを作成してください：

```bash
python device-management-target.py
```

### 6. テストデータの生成（オプション）

テーブルに合成テストデータを設定するには：

```bash
python synthetic_data.py
```

## 実行手順

### 1. Lambda関数のテスト

提供されたテストスクリプトを使用してLambda関数をローカルでテストできます：

```bash
python test_lambda.py
```

### 2. Q CLIを使用したツールの呼び出し

#### Bearer Tokenの生成

認証に必要なBearerトークンを生成するには、以下のcurlコマンドを使用してCognito OAuth2エンドポイントからアクセストークンをリクエストしてください：

```bash
curl --http1.1 -X POST https://<cognito domain>.auth.us-west-2.amazoncognito.com/oauth2/token \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "grant_type=client_credentials&client_id=<client id>&client_secret=<client_secret>"
```

このコマンドは、MCP設定でBearerトークンとして使用するべきアクセストークンを含むJSON応答を返します。

#### MCPの設定

この設定でmcp.jsonファイルを更新してください：
```json
cd ~/.aws/amazonq
vi mcp.json
## このjsonを更新
{
  "mcpServers": {
    "<you_desired_mcp_server_name>": {
      "command": "npx",
      "timeout": 60000,
      "args": [
        "mcp-remote@latest",
        "https://<gateway id>.gateway.bedrock-agentcore.<REGION>.amazonaws.com/mcp",
        "--header",
        "Authorization: Bearer <Bearer token>"
      ]
    }
  }
}
```

`<Bearer token>`をCognito認証リクエストから取得したアクセストークンに置き換えてください。

### 3. サンプルプロンプト:

1. "すべての休眠デバイスをリストできますか？"
2. "デバイスID DG-10016のSSIDをXXXXXXXXXXに更新できますか？"
3. "利用可能なすべてのツールをリストできますか？"
4. "デバイスID DG-10016のデバイス設定を表示できますか？"
5. "デバイスID DB-10005のWifi設定を表示できますか？"

## クリーンアップ手順

このユースケースで作成されたリソースをクリーンアップするには：

1. Lambda関数の削除:
```bash
aws lambda delete-function --function-name DeviceManagementLambda
```

2. Gateway Targetの削除:
```bash
aws bedrock-agentcore delete-gateway-target --gateway-identifier <your-gateway-identifier> --target-name device-management-target
```

3. Gateway（新しく作成した場合）の削除:
```bash
aws bedrock-agentcore delete-gateway --gateway-identifier <your-gateway-identifier>
```

4. DynamoDBテーブル（このユースケース用に作成した場合）の削除:
```bash
aws dynamodb delete-table --table-name Devices
aws dynamodb delete-table --table-name DeviceSettings
aws dynamodb delete-table --table-name WifiNetworks
aws dynamodb delete-table --table-name Users
aws dynamodb delete-table --table-name UserActivities
```

## 免責事項

このリポジトリで提供されている例は、実験的および教育目的のみです。概念や技術を実証するものですが、本番環境での直接使用を意図したものではありません。プロンプトインジェクションから保護するために、Amazon Bedrock Guardrailsを設置することを確認してください。

## 追加情報

### ファイル

- `lambda_function.py`: すべてのMCPツールを実装するメインLambdaハンドラー
- `dynamodb_models.py`: DynamoDBテーブル定義と初期化
- `synthetic_data.py`: 合成テストデータを生成するスクリプト
- `requirements.txt`: Python依存関係
- `deploy.sh`: Lambda関数のデプロイメントスクリプト
- `create_gateway.py`: MCPサーバー用のゲートウェイ作成スクリプト
- `device-management-target.py`: Lambda関数用のゲートウェイターゲット作成スクリプト
- `.env`: 環境変数設定ファイル
- `test_lambda.py`: Lambda関数をローカルでテストするスクリプト

### IAM権限

Lambda関数には以下のIAM権限が必要です：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:Query",
        "dynamodb:Scan",
        "dynamodb:UpdateItem"
      ],
      "Resource": [
        "arn:aws:dynamodb:<Region>:*:table/Devices",
        "arn:aws:dynamodb:<Region>:*:table/DeviceSettings",
        "arn:aws:dynamodb:<Region>:*:table/WifiNetworks",
        "arn:aws:dynamodb:<Region>:*:table/Users",
        "arn:aws:dynamodb:<Region>:*:table/UserActivities",
        "arn:aws:dynamodb:<Region>:*:table/UserActivities/index/ActivityTypeIndex"
      ]
    }
  ]
}
```

### トラブルシューティング

- **環境変数の欠落**: すべての必要な変数が`.env`ファイルに設定されていることを確認
- **Lambdaデプロイメントの失敗**: AWS IAM権限とLambdaサービスクォータを確認
- **Gateway作成の失敗**: Cognito User Pool ID、App Client ID、IAMロールARNを確認
- **Gateway target作成の失敗**: ゲートウェイ識別子とLambda ARNが正しいことを確認
- **DynamoDBアクセス問題**: Lambda実行ロールが必要な権限を持っていることを確認