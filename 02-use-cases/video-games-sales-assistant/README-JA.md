# Amazon Bedrock AgentCoreを用いた会話型データアナリストアシスタントソリューションのデプロイ

このソリューションは、ユーザーが自然言語インターフェースを通じてデータと相互作用できる生成AIアプリケーションリファレンスを提供します。このソリューションは、カスタムエージェントアプリケーションのデプロイ、実行、スケーリングを可能にする管理サービスである**[AWS Bedrock AgentCore](https://aws.amazon.com/bedrock/agentcore/)**と、**[Strands Agents SDK](https://strandsagents.com/)**を活用して、PostgreSQLデータベースに接続し、Webアプリケーションインターフェースを通じてデータ分析機能を提供するエージェントを構築します。

<div align="center">
<img src="./images/data-analyst-assistant-agentcore-strands-agents-sdk.gif" alt="Amazon Bedrock AgentCoreを用いた会話型データアナリストアシスタントソリューション">
</div>

🤖 データアナリストアシスタントは、企業が複雑なSQLクエリではなく自然言語会話を通じて構造化データと相互作用できるデータ分析アプローチを提供します。この種のアシスタントは、データ分析会話のための直感的な質問応答を提供し、ユーザーエクスペリエンスを向上させるデータ可視化を提供することで改善できます。

✨ このソリューションによりユーザーは以下が可能です：

- ビデオゲーム売上データについて自然言語で質問
- PostgreSQLデータベースへのSQLクエリに基づくAI生成応答の受信
- クエリ結果を表形式で表示
- 自動生成された可視化を通じたデータ探索
- AIアシスタントからの洞察と分析の取得

🚀 このリファレンスソリューションは以下のようなユースケースの探索に役立ちます：

- リアルタイムビジネスインテリジェンスによるアナリストの強化
- C級幹部への一般的なビジネス質問に対する迅速な回答提供
- データ収益化による新たな収益源の開放（消費者行動、オーディエンスセグメンテーション）
- パフォーマンス洞察によるインフラストラクチャの最適化

## ソリューション概要

以下のアーキテクチャ図は、Strands Agents SDKを使用して構築され、Amazon Bedrockによって駆動される生成AIデータアナリストアシスタントのリファレンスソリューションを示しています。このアシスタントにより、ユーザーはPostgreSQLデータベースに格納された構造化データに質問応答インターフェースを通じてアクセスできます。

![Video Games Sales Assistant](./images/gen-ai-assistant-diagram.png)

> [!IMPORTANT]
> このサンプルアプリケーションはデモ目的であり、本番対応ではありません。組織のセキュリティベストプラクティスでコードを検証することを確認してください。

### AgentCore RuntimeとMemoryインフラストラクチャ

**Amazon Bedrock AgentCore**は、組み込みのランタイムとメモリ機能を備えたカスタムエージェントアプリケーションのデプロイ、実行、スケーリングを可能にする完全管理サービスです。

- **[Amazon Bedrock AgentCore Runtime](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/agents-tools-runtime.html)**: エージェントインスタンスの呼び出しエンドポイント（`/invocations`）とヘルス監視（`/ping`）を備えた管理実行環境を提供
- **[Amazon Bedrock AgentCore Memory](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/memory.html)**: AIエージェントがイベントをキャプチャし、それらをメモリに変換し、必要に応じて関連コンテキストを取得することで、相互作用を通じて記憶、学習、進化する能力を提供する完全管理サービス

AgentCoreインフラストラクチャはすべての保存の複雑さを処理し、開発者が基盤インフラストラクチャを管理する必要なく効率的な検索を提供し、エージェント相互作用全体の継続性と追跡可能性を確保します。

### CDKインフラストラクチャデプロイメント

AWS CDKスタックは以下の管理サービスをデプロイし、設定します：

- **IAM AgentCore実行ロール**: Amazon Bedrock AgentCore実行に必要な権限を提供
- **VPCとプライベートサブネット**: データベースリソースのネットワーク分離とセキュリティ
- **Amazon Aurora Serverless PostgreSQL**: RDS Data API統合でビデオゲーム売上データを格納
- **Amazon DynamoDB**: 生のクエリ結果とエージェント相互作用を追跡
- **Parameter Store設定管理**: アプリケーション設定を安全に管理

### Strandsエージェントの機能

| 機能 | 説明 |
|----------|----------|
| ネイティブツール   | current_time - ユーザーのタイムゾーンに基づいて現在の日時情報を提供する組み込みStrandsツール。 |
| カスタムツール | get_tables_information - データベーステーブルの構造、列、関係を含むメタデータを取得して、エージェントがデータベーススキーマを理解できるようにするカスタムツール。<br>execute_sql_query - エージェントがユーザーの自然言語質問に基づいてPostgreSQLデータベースに対してSQLクエリを実行し、分析用に要求されたデータを取得できるようにするカスタムツール。 |
| モデルプロバイダー | Amazon Bedrock |

> [!NOTE]
> このソリューションは、Amazon Bedrock AgentCoreとAmazon DynamoDBに接続するためのAWS IAM認証情報の使用を参照しています。🚀 本番デプロイメントの場合、IAMユーザー認証情報を使用する代わりに、適切な認証と認可のためにAmazon Cognitoまたは別のアイデンティティプロバイダーの統合を検討してください。

> [!TIP]
> エージェントの指示とツール実装を適応させることで、優先データベースエンジンに接続するようにデータソースを変更することもできます。

> [!IMPORTANT] 
> **[Strands Agents SDK](https://strandsagents.com/latest/user-guide/safety-security/guardrails/)**が提供するシームレスな統合により、AIアプリケーション用の**[Amazon Bedrock Guardrails](https://aws.amazon.com/bedrock/guardrails/)**を実装してAIの安全性とコンプライアンスを強化してください。

**ユーザー相互作用ワークフロー**は以下のように動作します：

- Webアプリケーションがユーザーのビジネス質問をAgentCore Invokeに送信
- Strandsエージェント（Claude 3.7 Sonnetで駆動）が自然言語を処理し、データベースクエリの実行タイミングを決定
- エージェントの組み込みツールがAurora PostgreSQLデータベースに対してSQLクエリを実行し、質問への回答を作成
- AgentCore Memoryがセッション相互作用をキャプチャし、コンテキスト用の過去の会話を取得
- エージェントの応答がWebアプリケーションで受信された後、生のデータクエリ結果がDynamoDBテーブルから取得され、回答と対応するレコードの両方が表示
- チャート生成の場合、アプリケーションがモデル（Claude 3.5 Sonnetで駆動）を呼び出してエージェントの回答と生のデータクエリ結果を分析し、適切なチャート可視化をレンダリングするために必要なデータを生成

## デプロイメント手順

デプロイメントは2つの主要なステップで構成されます：

1. **バックエンドデプロイメント - [CDKによるデータソースと設定管理デプロイメント](./cdk-agentcore-strands-data-analyst-assistant/)**
1. **エージェントデプロイメント - [AgentCoreによるStrandsエージェントインフラストラクチャデプロイメント](./agentcore-strands-data-analyst-assistant/)**
2. **フロントエンド実装 - [すぐに使えるデータアナリストアシスタントアプリケーションとAgentCoreの統合](./amplify-video-games-sales-assistant-agentcore-strands/)**

> [!NOTE]
> *アプリケーションのデプロイにはOregon（us-west-2）またはN. Virginia（us-east-1）リージョンの使用を推奨します。*

> [!IMPORTANT] 
> 不要なコストを避けるため、テスト後は提供されるクリーンアップ手順に従ってリソースをクリーンアップすることを忘れないでください。

## アプリケーション機能

以下の画像は、自然言語回答、SQLクエリ生成にLLMが使用した推論プロセス、それらのクエリから取得されたデータベースレコード、結果のチャート可視化を含む会話体験分析を示しています。

![Video Games Sales Assistant](./images/preview.png)

- **ユーザーの質問に応答するエージェントとの会話インターフェース**

![Video Games Sales Assistant](./images/preview1.png)

- **表形式で表示される生のクエリ結果**

![Video Games Sales Assistant](./images/preview2.png)

- **エージェントの回答とデータクエリ結果から生成されたチャート可視化（[Apexcharts](https://apexcharts.com/)を使用して作成）**。

![Video Games Sales Assistant](./images/preview3.png)

- **データ分析会話から導出された要約と結論**

![Video Games Sales Assistant](./images/preview4.png)

## ありがとうございます

## ライセンス

このプロジェクトはApache-2.0ライセンスの下でライセンスされています。