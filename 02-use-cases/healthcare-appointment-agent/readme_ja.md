# ヘルスケア予約エージェント

## 概要
**Amazon Bedrock AgentCore Gateway**を使用し、**Model Context Protocol (MCP)**でツールを公開して構築された、予防接種関連のヘルスケア予約AIエージェントです。このAIエージェントは、現在の予防接種状況/スケジュールの照会、予約スロットの確認、予約の取得をサポートします。また、ログインしたユーザー（成人）とその子供を認識してパーソナライズされた体験を提供し、**AWS Healthlake**を**FHIR R4** (Fast Healthcare Interoperability Resources) データベースとして使用します。

### ユースケース詳細
| 情報               | 詳細                                                                                                                               |
|--------------------|------------------------------------------------------------------------------------------------------------------------------------|
| ユースケースタイプ | 会話型                                                                                                                            |
| エージェントタイプ | シングルエージェント                                                                                                              |
| ユースケース構成要素 | Amazon Bedrock AgentCore関連コンポーネント：Gateway および Identity                                                              |
|  					 | その他のコンポーネント：Amazon Cognito、AWS Healthlake、Amazon API Gateway、AWS Lambda                                          |
|  					 | MCPツール：MCPツールはOpenAPI仕様を使用してBedrock AgentCore Gatewayに公開                                                      |
| ユースケース業界   | ヘルスケア                                                                                                                        |
| サンプル複雑度     | 中級                                                                                                                              |
| 使用SDK            | Amazon Bedrock AgentCore SDK および boto3								                                                      |


### ユースケースアーキテクチャ
![Image1](static/healthcare_gateway_flow.png)

### ユースケースの主要機能

## 前提条件
**注意：これらの手順はus-east-1およびus-west-2リージョンで動作するように設計されています。**

### 必要なIAMポリシー
必要なIAMアクセス許可を確認してください。管理者ロールからこのサンプルを実行する場合は無視してください。

このサンプルで使用されるCloudformationスタックには、AWS Healthlake、Cognito、S3、IAMロール、API Gateway、Lambda関数関連のリソースが含まれています。

クイックスタートとして、AWSマネージドIAMポリシーとインラインポリシーの組み合わせを使用して、このコードサンプルのデプロイと設定の問題を回避できます。ただし、本番環境では権限の原則に従うことをお勧めします。

**AWSマネージドIAMポリシー：**
* AmazonAPIGatewayAdministrator
* AmazonCognitoPowerUser
* AmazonHealthLakeFullAccess
* AmazonS3FullAccess
* AWSCloudFormationFullAccess
* AWSKeyManagementServicePowerUser
* AWSLakeFormationDataAdmin
* AWSLambda_FullAccess
* AWSResourceAccessManagerFullAccess
* CloudWatchFullAccessV2
* AmazonBedrockFullAccess

**インラインポリシー：**
```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": [
				"s3:GetObject",
				"s3:PutObject"
			],
			"Resource": [
				"arn:aws:s3:::amzn-s3-demo-source-bucket/*",
				"arn:aws:s3:::amzn-s3-demo-logging-bucket/*"
			]
		},
		{
			"Effect": "Allow",
			"Action": [
				"ram:GetResourceShareInvitations",
				"ram:AcceptResourceShareInvitation",
				"glue:CreateDatabase",
				"glue:DeleteDatabase"
			],
			"Resource": "*"
		},
		{
			"Effect": "Allow",
			"Action": [
				"bedrock-agentcore:*",
				"agent-credential-provider:*"
			],
			"Resource": "*"
		}
	]
}
```
### その他
* Python 3.12
* GIT
* AWS CLI 2.x
* Amazon BedrockでClaude 3.5 Sonnetモデルが有効化されている。設定については、こちらの[ガイド](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access-modify.html)に従ってください。

## ユースケースセットアップ
GITリポジトリをクローンし、Healthcare-Appointment-Agentディレクトリに移動します。

```bash
git clone <repository-url>
cd ./02-use-cases/05-Healthcare-Appointment-Agent/
```

### インフラストラクチャーのセットアップ
S3バケットを作成します（**既存のバケットを使用する場合は無視してください**）

```bash
aws s3api create-bucket --bucket <globally unique bucket name here>
```

lambdaのzipパッケージをS3バケットにプッシュします
```bash
aws s3 cp "./cloudformation/fhir-openapi-searchpatient.zip" s3://<bucket name here>/lambda_code/fhir-openapi-searchpatient.zip
```

以下の手順でCloudformationテンプレートをデプロイします。スタックは約10分かかります。スタックの進行状況は、この[ガイド](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/monitor-stack-progress.html)に従って監視できます。
```bash
aws cloudformation create-stack \
  --stack-name healthcare-cfn-stack \
  --template-body file://cloudformation/healthcare-cfn.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --region <us-east-1 or us-west-2> \
  --parameters ParameterKey=LambdaS3Bucket,ParameterValue="<bucket name here>" \
               ParameterKey=LambdaS3Key,ParameterValue="lambda_code/fhir-openapi-searchpatient.zip"
```

### Python依存関係のインストールと環境の初期化
この[ガイド](https://docs.astral.sh/uv/getting-started/installation/)に従ってUVをインストールしてください

仮想環境を作成してアクティベートします

```bash
uv venv --python 3.12
source ./.venv/bin/activate
```

依存関係をインストールします

```bash
uv pip install -r requirements.txt
```

以下のコマンドを実行して環境を初期化します。これにより、環境変数に使用される**.env**ファイルが作成されます。上記のCloudformationテンプレートで使用したのと同じリージョン名を使用してください。出力で返される**APIEndpoint**と**APIGWCognitoLambdaName**をメモしてください。

```bash
python init_env.py \
--cfn_name healthcare-cfn-stack \
--openapi_spec_file ./fhir-openapi-spec.yaml \
--region <us-east-1 or us-west-2>
```

名前付きクレデンシャルプロファイルを使用する必要がある場合は、以下で実現できます。

```bash
python init_env.py \
--cfn_name healthcare-cfn-stack \
--region <us-east-1 or us-west-2> \
--openapi_spec_file ./fhir-openapi-spec.yaml \
--profile <profile-name here>>
```

**.env**ファイルは以下のようになります。
![EnvImage1](static/env_screenshot1.png)


**API GatewayでCognito認証を有効化**
```bash
aws lambda invoke \
--function-name <input APIGWCognitoLambdaName as noted earlier> \
response.json \
--payload '{ "RequestType": "Create" }' \
--cli-binary-format raw-in-base64-out \
--region <us-east-1 or us-west-2>
```

### AWS Healthlakeにテストデータを作成
以下のPythonプログラムを実行して、**test_data**フォルダに存在するテストデータを取り込みます。完了まで約5分かかる場合があります。
```bash
python create_test_data.py
```

## 実行手順
### Bedrock AgentCore GatewayとGateway Targetを作成
OpenAPI仕様ファイル**fhir-openapi-spec.yaml**を開き、**<your API endpoint here>**を先ほどメモした**APIEndpoint**に置き換えます。

**fhir-openapi-spec.yaml**ファイルのOpenAPI仕様に基づいてBedrock AgentCore GatewayとGateway Targetをセットアップします。後の手順で必要になるため、出力からGateway Idをメモしてください。

```bash
python setup_fhir_mcp.py --op_type Create --gateway_name <gateway_name_here>
```

### Strands Agentの実行
以下の手順でStrands Agentを実行します。

```bash
python strands_agent.py --gateway_id <gateway_id_here>
```

### Langgraph Agentの実行
以下の手順でLanggraph Agentを実行します。

```bash
python langgraph_agent.py --gateway_id <gateway_id_here>
```

### エージェントとの対話のサンプルプロンプト：
* どのようにお助けできますか？
* まず予防接種スケジュールを確認しましょう
* スケジュールされた日付頃のMMRワクチンのスロットを見つけてください

![Image1](static/appointment_agent_demo.gif)


## クリーンアップ手順
API GatewayでCognitoAuth を無効化

```bash
aws lambda invoke \
--function-name <input APIGWCognitoLambdaName as noted earlier> \
response.json \
--payload '{ "RequestType": "Delete" }' \
--cli-binary-format raw-in-base64-out \
--region <us-east-1 or us-west-2>
```

gatewayとgateway targetを削除します。複数のgatewayを作成した場合は、すべてのgatewayに対してこの手順を繰り返します。

```bash
python setup_fhir_mcp.py --op_type Delete --gateway_id <gateway_id_here>
```

Cloudformationスタックを削除します。

```bash
aws cloudformation delete-stack \
  --stack-name healthcare-cfn-stack \
  --region <us-east-1 or us-west-2>
```