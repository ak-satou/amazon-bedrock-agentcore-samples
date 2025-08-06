# Amazon Bedrock AgentCore Gateway

## 概要
Bedrock AgentCore Gatewayは、顧客が既存のAPIやLambda関数を、インフラの管理やホスティングを必要とすることなく、完全に管理されたMCPサーバーに変換する方法を提供します。顧客は既存のAPIにOpenAPI仕様やSmithyモデルを持参したり、ツールをフロントエンドとするLambda関数を追加したりできます。Gatewayは、これらすべてのツール全体で統一されたModel Context Protocol（MCP）インターフェースを提供します。Gatewayは、受信リクエストとターゲットリソースへの送信接続の両方に対して安全なアクセス制御を確保するための二重認証モデルを採用しています。このフレームワークは2つの主要コンポーネントで構成されています：ゲートウェイターゲットにアクセスしようとするユーザーを検証・認可するInbound Auth、および認証されたユーザーに代わってゲートウェイがバックエンドリソースに安全に接続することを可能にするOutbound Authです。これらの認証メカニズムが連携して、ユーザーとターゲットリソース間の安全なブリッジを作成し、IAM認証情報とOAuthベースの認証フローの両方をサポートします。GatewayはMCPのStreamable HTTP transport接続をサポートします。

![動作の仕組み](images/gateway-end-end-overview.png)

## 概念の定義

Amazon Bedrock AgentCore Gatewayを開始する前に、いくつかの重要な概念を定義しましょう：

* **Amazon Bedrock AgentCore Gateway**: 顧客が標準的なMCP操作（listToolsやinvokeToolなど）を実行するためにMCPクライアントで呼び出すことができるHTTPエンドポイント。顧客はboto3などのAWS SDKを使用してこのAmazonCore Gatewayを呼び出すこともできます。
* **Bedrock AgentCore Gateway Target**: 顧客がAmazonCore Gatewayにターゲットを接続するために使用するリソース。現在、AgentCore Gatewayのターゲットとして以下のタイプがサポートされています：
    * Lambda ARN
    * API仕様 → OpenAPI、Smithy
* **MCP Transport**: クライアント（LLMを使用するアプリケーション）とMCPサーバー間でメッセージが移動する方法を定義するメカニズム。現在、AgentCore Gatewayは転送として`Streamable HTTP接続`のみをサポートしています。

## 動作の仕組み

![動作の仕組み](images/gateway_how_does_it_work.png)

## インバウンドおよびアウトバウンド認証
Bedrock AgentCore Gatewayは、インバウンドおよびアウトバウンド認証を介して安全な接続を提供します。インバウンド認証では、AgentCore Gatewayが呼び出し時に渡されるOAuthトークンを分析して、ゲートウェイ内のツールへのアクセスを許可または拒否することを決定します。ツールが外部リソースにアクセスする必要がある場合、AgentCore GatewayはAPI Key、IAM、またはOAuth Tokenを介したアウトバウンド認証を使用して、外部リソースへのアクセスを許可または拒否できます。

インバウンド認証フロー中、エージェントまたはMCPクライアントは、OAuthアクセストークン（ユーザーのIdPから生成）を追加してAgentCore Gateway内のMCPツールを呼び出します。AgentCore Gatewayは、OAuthアクセストークンを検証してインバウンド認証を実行します。

AgentCore Gatewayで実行されているツールが外部リソースにアクセスする必要がある場合、OAuthは、Gatewayターゲット用のリソース認証情報プロバイダーを使用して、ダウンストリームリソースの認証情報を取得します。AgentCore Gatewayは、ダウンストリームAPIにアクセスするために認証情報を呼び出し元に渡します。

![安全なアクセス](images/gateway_secure_access.png)

### MCP認証とGateway

Amazon Bedrock AgentCore Gatewayは、受信MCP ツール呼び出しの認証について[MCP認証仕様](https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization)に準拠しています。

![安全なアクセス](images/oauth-flow-gateway.png)

### AgentCore GatewayとAgentCore Identityとの統合

![AgentCore IdentityとGateway](images/end-end-auth-gateway.png)

### ツールの検索
Amazon Bedrock AgentCore Gatewayには、エージェントと開発者が自然言語クエリを通じて最も関連性の高いツールを見つけることを支援し、**ツール選択のためにエージェントに渡されるコンテキストを削減する**強力な組み込みセマンティック検索機能も含まれています。この検索機能は、セマンティックマッチングにベクトル埋め込みを活用する事前構築されたツールとして実装されています。ユーザーは、CreateGateway APIを通じてオプトインすることで、Gateway作成時にこの機能を有効にできます。一度有効にすると、その後のCreateTarget操作は自動的にターゲットのツール用のベクトル埋め込みの生成をトリガーします。このプロセス中、埋め込みが生成されている間、CreateTargetレスポンスのSTATUSフィールドは「UPDATING」を示します

![tool_search](images/gateway_tool_search.png)

### チュートリアル詳細

| 情報          | 詳細                                                   |
|:---------------------|:----------------------------------------------------------|
| チュートリアルタイプ        | インタラクティブ                                               |
| AgentCoreコンポーネント | AgentCore Gateway、AgentCore Identity                     |
| エージェンティックフレームワーク    | Strands Agents                                            |
| LLMモデル            | Anthropic Claude Sonnet 3.7、Amazon Nova Pro              |
| チュートリアルコンポーネント  | AgentCore Gatewayの作成とAgentCore Gatewayの呼び出し |
| チュートリアル分野    | クロス分野                                            |
| 例の複雑さ   | 簡単                                                      |
| 使用SDK             | boto3                                                     |

## チュートリアルアーキテクチャ

### チュートリアル主要機能

#### 安全なツールアクセス

Amazon Bedrock AgentCore Gatewayは、受信MCP ツール呼び出しの認証について[MCP認証仕様](https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization)に準拠しています。
Amazon Bedrock AgentCore Gatewayは、Gatewayからの送信呼び出しの認証をサポートするために2つのオプションも提供します：
* API keyを使用する、または
* OAuthアクセストークンを使用する
Amazon Bedrock AgentCore IdentityのCredentials provider APIを使用して認証を設定し、それらをAgentCore Gateway Targetに接続できます。
各Target（AWS Lambda、Smithy、OpenAPI）は認証情報プロバイダーに接続できます。

#### 統合

Bedrock AgentCore Gatewayは以下と統合されます：
* Bedrock AgentCore Identity
* Bedrock AgentCore Runtime

### ユースケース

* MCPツールを呼び出すリアルタイムインタラクティブエージェント
* 異なるIdPを使用したインバウンド・アウトバウンド認証
* AWS Lambda関数、Open API、SmithyモデルのMCP化
* MCPツールの発見

### 利点

* Gatewayは、AIエージェント開発とデプロイを簡素化するいくつかの主要な利点を提供します：インフラ管理不要
* ホスティングの懸念がない完全管理サービス。Amazon Bedrock AgentCoreがすべてのインフラを自動的に処理します。
* 統一インターフェース：すべてのツールに対する単一のMCPプロトコルにより、エージェントコードで複数のAPIフォーマットと認証メカニズムを管理する複雑さが解消されます。
* 組み込み認証：OAuth および認証情報管理により、追加の開発労力なしでトークンのライフサイクル、リフレッシュ、安全な保存を処理します。
* 自動スケーリング：手動介入や容量計画なしで、さまざまなワークロードを処理するために需要に基づいて自動的にスケールします。
* エンタープライズセキュリティ：暗号化、アクセス制御、監査ログを含むエンタープライズグレードのセキュリティ機能により、安全なツールアクセスを確保します。

## チュートリアル概要

これらのチュートリアルでは、以下の機能をカバーします：

- [AWS Lambda関数をMCPツールに変換](01-transform-lambda-into-mcp-tools)
- [APIをMCPツールに変換](02-transform-apis-into-mcp-tools)
- [MCPツールの発見](03-discover-mcp-tools)