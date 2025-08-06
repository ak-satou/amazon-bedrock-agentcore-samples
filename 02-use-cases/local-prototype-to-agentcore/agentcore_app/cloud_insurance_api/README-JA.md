# MangumでFastAPIをAWS Lambdaにデプロイ

このプロジェクトは、API Gateway統合でMangumアダプターを使用してFastAPIアプリケーションをAWS Lambdaにデプロイする方法を示しています。

## プロジェクト構造

```
cloud_insurance_api/
├── local_insurance_api/      # FastAPIアプリケーション ソースコード
│   ├── app.py                # FastAPIアプリケーション定義
│   ├── data_loader.py        # データロード ユーティリティ
│   ├── routes/               # APIルート モジュール
│   └── services/             # ビジネスロジック サービス
├── lambda_function.py        # Mangum統合を持つAWS Lambdaハンドラー
├── deployment/
│   ├── template.yaml         # インフラストラクチャ用AWS SAMテンプレート
│   └── deploy.sh             # AWS SAMを使用したデプロイメント スクリプト
├── openapi.json              # 生成されたOpenAPI仕様
└── README.md                 # プロジェクト ドキュメント
```

## 動作原理

### 1. FastAPIアプリケーション

`local_insurance_api`ディレクトリには、保険関連のエンドポイントを提供するFastAPIアプリケーションが含まれています。これは、ローカルで実行することも、AWS Lambdaにデプロイすることもできる標準的なFastAPIアプリケーションです。

### 2. Mangum統合

[Mangum](https://github.com/jordaneremieff/mangum)は、FastAPIなどのASGIアプリケーションをAWS LambdaおよびAPI Gatewayで使用するためのアダプターです。API Gatewayリクエストを、FastAPIが処理できる形式に変換し、その後、FastAPIレスポンスをAPI Gatewayが理解できる形式に戻します。

統合は`lambda_function.py`で行われます：

```python
import os
import sys

# 現在のディレクトリをPythonパスに追加
current_dir = os.path.dirname(os.path.abspath(__file__))
sys.path.append(current_dir)

# FastAPIアプリをインポート
from local_insurance_api.app import app

# AWS Lambda統合用のMangumをインポート
from mangum import Mangum

# ハンドラーを作成
handler = Mangum(app)
```

`handler`関数は、Lambda関数のエントリーポイントです。API GatewayがLambda関数を呼び出すとき、Mangumがイベントとコンテキストを処理し、リクエストをFastAPIアプリに渡し、API Gatewayが理解できる形式でレスポンスを返します。

### 3. AWS SAMテンプレート

`deployment/template.yaml`ファイルは、AWSサーバーレスアプリケーションモデル（SAM）を使用して作成されるAWSリソースを定義します。これらには以下が含まれます：

- Lambda関数：FastAPIアプリケーション コードを実行
- API Gateway：Lambda関数を呼び出すHTTPエンドポイントを提供
- IAMロール：Lambda実行権限
- CloudWatch Log Group：Lambda実行ログ

### 4. デプロイメント プロセス

デプロイメントは`deployment/deploy.sh`スクリプトによって処理され、AWS SAM CLIを使用して以下を行います：

1. デプロイメント アーティファクト用のS3バケットを作成
2. アプリケーション コードと依存関係を含むデプロイメント パッケージを構築
3. パッケージをS3にアップロード
4. 定義されたリソースでCloudFormationスタックをデプロイ

## 前提条件

デプロイする前に、以下があることを確認してください：

- AWS CLIがインストールされ、適切な権限で設定されている
- AWS SAM CLIがインストールされている
- Python 3.10+がインストールされている
- Lambda、API Gateway、CloudFormation、S3、およびIAMへのアクセス権を持つAWSアカウント

## デプロイメント手順

1. **依存関係のインストール**

   ローカル環境に必要なすべての依存関係があることを確認してください：

   ```bash
   pip install -r local_insurance_api/requirements.txt
   pip install aws-sam-cli
   ```

2. **OpenAPI仕様のエクスポート（オプション）**

   APIには`openapi.json`に事前生成されたOpenAPI仕様があります。

3. **AWSへのデプロイ**

   デプロイメント スクリプトを実行します：

   ```bash
   # 開発環境へのデプロイ（デフォルト）
   ./deployment/deploy.sh

   # 特定の環境へのデプロイ（dev、test、prod）
   ./deployment/deploy.sh prod
   ```

4. **APIのテスト**

   デプロイ後、curlまたは任意のHTTPクライアントを使用してAPIをテストします：

   ```bash
   # 実際のAPI Gateway URLに置き換えてください
   ENDPOINT="https://i0zzy6t0x9.execute-api.us-west-2.amazonaws.com/dev"

   # ヘルス エンドポイントのテスト
   curl $ENDPOINT/health

   # ポリシー エンドポイントのテスト
   curl $ENDPOINT/policies

   # 顧客情報エンドポイントのテスト（POSTリクエスト）
   curl -X POST $ENDPOINT/customer_info \
     -H "Content-Type: application/json" \
     -d '{"customer_id": "cust-001"}'
   ```

## ローカルテスト

デプロイ前にローカルでテストするには：

```bash
# FastAPIアプリケーションを直接実行
cd local_insurance_api
uvicorn app:app --reload --port 8001

# AWS SAMを使用してローカルテスト
cd ..
sam local start-api
```

## トラブルシューティング

よくある問題と解決策：

### 1. 循環インポート

FastAPIアプリケーションは、モジュラー構造を使用する際に循環インポートを持つことがあります。Lambdaでは、これらがデプロイメントの失敗を引き起こす可能性があります。解決策には以下が含まれます：

- モジュールレベルではなく、関数内で関数ベースのインポートを使用
- 必要な時にのみモジュールをインポートするラッパー関数の作成
- 依存性注入パターンの使用

例えば、我々のAPIでは：
```python
# 循環インポートを避けるために関数を使用してルーターをインポート
def get_routers():
    from local_insurance_api.routes.general import router as general_router
    # さらにインポート...
    return general_router, ...
```

### 2. インポート パスの問題

ローカル環境とLambdaの間でコードを移動する際、インポート パスが壊れることがあります。Lambda環境では常に完全修飾インポートを使用してください：

```python
# Lambda環境ではこれを使用
from local_insurance_api.services import data_service

# これは使用しない（ローカルでは動作するがLambdaで壊れる可能性があります）
from services import data_service
```

### 3. API Gatewayエラー

- **403 Forbidden**：IAM権限を確認してください
- **500 Internal Server Error**：CloudWatchでLambda実行ログを確認してください
- **"Missing Authentication Token"**：通常、URLパスが正しくないことを意味します

### 4. コールドスタート遅延

Lambda関数は「コールドスタート」遅延を経験する可能性があります。最小化するには：

- `template.yaml`でLambdaメモリ割り当てを増やす
- 重要なエンドポイントにプロビジョニング済み同時実行を使用
- 依存関係のサイズを最適化

## APIドキュメント

APIは保険関連の操作用にいくつかのエンドポイントを提供します：
- `/openapi.json`：生のOpenAPI仕様
- `/health`：ヘルスチェック エンドポイント
- `/policies`：すべての保険ポリシーを取得
- `/customer_info`：顧客情報を取得
- `/vehicle_info`：車両情報を取得
- `/insurance_products`：利用可能な保険商品を取得

## その他のリソース

- [FastAPIドキュメント](https://fastapi.tiangolo.com/)
- [Mangumドキュメント](https://mangum.io/)
- [AWS Lambdaドキュメント](https://docs.aws.amazon.com/lambda/)
- [AWS SAMドキュメント](https://docs.aws.amazon.com/serverless-application-model/)

## クリーンアップ

保険APIアプリケーションの使用が終了したら、以下の手順ですべてのAWSリソースをクリーンアップしてください：

1. **CloudFormationスタックの削除**：
   ```bash
   # スタック名を取得（覚えていない場合）
   aws cloudformation list-stacks --query "StackSummaries[?contains(StackName,'insurance-api')].StackName" --output text
   
   # スタックを削除
   aws cloudformation delete-stack --stack-name insurance-api-stack-dev
   ```

2. **S3デプロイメント バケットの削除**（もう必要ない場合）：
   ```bash
   # S3バケットを一覧表示してデプロイメント バケットを見つける
   aws s3 ls | grep insurance-api
   
   # 最初にバケットからすべてのファイルを削除
   aws s3 rm s3://insurance-api-deployment-bucket-1234 --recursive
   
   # 空のバケットを削除
   aws s3api delete-bucket --bucket insurance-api-deployment-bucket-1234
   ```

3. **リソース削除の確認**：
   ```bash
   # Lambda関数がまだ存在するか確認
   aws lambda list-functions --query "Functions[?contains(FunctionName,'insurance-api')].FunctionName" --output text
   
   # API Gatewayがまだ存在するか確認
   aws apigateway get-rest-apis --query "items[?contains(name,'insurance-api')].id" --output text
   ```

4. **CloudWatchログのクリーンアップ**（オプション）：
   ```bash
   # ログ グループを見つける
   aws logs describe-log-groups --query "logGroups[?contains(logGroupName,'/aws/lambda/insurance-api')].logGroupName" --output text
   
   # ログ グループを削除
   aws logs delete-log-group --log-group-name /aws/lambda/insurance-api-function-dev
   ```

注意：`insurance-api-stack-dev`、`insurance-api-deployment-bucket-1234`、`/aws/lambda/insurance-api-function-dev`などのプレースホルダー値を実際のリソース名に置き換えてください。