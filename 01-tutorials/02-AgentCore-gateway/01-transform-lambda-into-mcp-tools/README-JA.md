# GatewayのためのLambda関数ツールの実装

## 概要
Bedrock AgentCore Gatewayは、既存のLambda関数をインフラストラクチャやホスティングの管理を必要とせずに、完全に管理されたMCPサーバーに変換する方法をお客様に提供します。お客様は既存のAWS Lambda関数を持参するか、ツールの前面に新しいLambda関数を追加できます。Gatewayはこれらすべてのツールにわたって統一されたModel Context Protocol（MCP）インターフェースを提供します。Gatewayは、受信リクエストとターゲットリソースへの発信接続の両方に対して安全なアクセス制御を確保するため、デュアル認証モデルを採用しています。フレームワークは2つの主要コンポーネントで構成されています：ゲートウェイターゲットへのアクセスを試みるユーザーを検証・認証するInbound Auth、および認証されたユーザーに代わってゲートウェイがバックエンドリソースに安全に接続できるようにするOutbound Authです。これらの認証メカニズムは連携して、ユーザーとターゲットリソース間の安全なブリッジを作成し、IAMクレデンシャルとOAuthベースの認証フローの両方をサポートします。

![動作の仕組み](images/lambda-iam-gateway.png)


### Lambdaコンテキストオブジェクトの理解
GatewayがLambda関数を呼び出すとき、context.client_contextオブジェクトを通じて特別なコンテキスト情報を渡します。このコンテキストには呼び出しに関する重要なメタデータが含まれており、関数がリクエストの処理方法を決定するために使用できます。
context.client_context.customオブジェクトでは以下のプロパティが利用可能です：
* bedrockagentcoreEndpointId: リクエストを受信したGatewayエンドポイントのID。
* bedrockagentcoreTargetId: 関数にリクエストをルーティングしたGatewayターゲットのID。
* bedrockagentcoreMessageVersion: リクエストで使用されるメッセージフォーマットのバージョン。
* bedrockagentcoreToolName: 呼び出されるツールの名前。これは、Lambda関数が複数のツールを実装している場合に特に重要です。
* bedrockagentcoreSessionId: 現在の呼び出しのセッションID。同じセッション内の複数のツール呼び出しを関連付けるために使用できます。

Lambda関数コードでこれらのプロパティにアクセスして、どのツールが呼び出されているかを判定し、関数の動作をそれに応じてカスタマイズできます。

![動作の仕組み](images/lambda-context-object.png)

### レスポンス形式とエラー処理

Lambda関数は、Gatewayが解釈してクライアントに返すことができるレスポンスを返す必要があります。レスポンスは以下の構造を持つJSONオブジェクトである必要があります。statusCodeフィールドは、操作の結果を示すHTTPステータスコードである必要があります：
* 200: 成功
* 400: 不正なリクエスト（クライアントエラー）
* 500: 内部サーバーエラー

bodyフィールドは、文字列またはより複雑なレスポンスを表すJSON文字列のいずれかです。構造化されたレスポンスを返したい場合は、JSON文字列にシリアライズする必要があります。

### エラー処理
適切なエラー処理は、クライアントに意味のあるフィードバックを提供するために重要です。Lambda関数は例外をキャッチし、適切なエラーレスポンスを返す必要があります。

### テスト

```__context__```フィールドは、Gatewayによって呼び出されるときに関数に渡される実際のイベントの一部ではないことに注意してください。これはテスト目的でのみ使用され、コンテキストオブジェクトをシミュレートします。
Lambdaコンソールでテストする際は、シミュレートされたコンテキストを処理するように関数を変更する必要があります。このアプローチにより、Gatewayターゲットとしてデプロイする前に、異なるツール名と入力パラメータでLambda関数をテストできます。

### クロスアカウントLambdaアクセス

Lambda関数がGatewayとは異なるAWSアカウントにある場合、GatewayがLambda関数を呼び出せるように、Lambda関数にリソースベースのポリシーを設定する必要があります。ポリシー例：

```
{
  "Version": "2012-10-17",
  "Id": "default",
  "Statement": [
    {
      "Sid": "cross-account-access",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/GatewayExecutionRole"
      },
      "Action": "lambda:InvokeFunction",
      "Resource": "arn:aws:lambda:us-west-2:987654321098:function:MyLambdaFunction"
    }
  ]
}
```
このポリシーでは：
- 123456789012はGatewayがデプロイされているアカウントID
- GatewayExecutionRoleはGatewayが使用するIAMロール
- 987654321098はLambda関数がデプロイされているアカウントID
- MyLambdaFunctionはLambda関数の名前

このポリシーを追加した後、Lambda関数が異なるアカウントにあっても、GatewayターゲットコンフィグレーションでLambda関数ARNを指定できます。

### チュートリアル詳細


| 項目                 | 詳細                                                   |
|:---------------------|:----------------------------------------------------------|
| チュートリアルタイプ        | インタラクティブ                                               |
| AgentCore構成要素 | AgentCore Gateway、AgentCore Identity                     |
| エージェント フレームワーク    | Strands Agents                                            |
| LLMモデル            | Anthropic Claude Sonnet 3.7、Amazon Nova Pro              |
| チュートリアル構成要素  | AgentCore Gatewayの作成とAgentCore Gatewayの呼び出し |
| チュートリアル分野    | 横断的                                            |
| 例の複雑さ   | 簡単                                                      |
| 使用SDK             | boto3                                                     |

## チュートリアル アーキテクチャ

### チュートリアル主要機能

* Lambda関数をMCPツールに公開
* AWS IAMとOAuthを使用したツール呼び出しのセキュア化

## チュートリアル概要

これらのチュートリアルでは、以下の機能について説明します：

- [AWS Lambda関数をMCPツールに変換](01-gateway-target-lambda.ipynb)