# 📚 Amazon Bedrock AgentCore チュートリアル

Amazon Bedrock AgentCoreサンプルリポジトリのチュートリアルセクションへようこそ！

このフォルダには、Amazon Bedrock AgentCoreの基本機能を実践的な例を通じて学習するためのインタラクティブなノートブックベースのチュートリアルが含まれています。

チュートリアルはAmazon Bedrock AgentCoreコンポーネント別に整理されています：

* **Runtime**: Amazon Bedrock AgentCore Runtimeは、フレームワーク、プロトコル、モデルの選択に関係なく、組織がAIエージェントとツールの両方をデプロイ・スケールすることを可能にする安全なサーバーレスランタイム機能です—迅速なプロトタイピング、シームレスなスケーリング、市場投入時間の短縮を実現します
* **Gateway**: AIエージェントは実世界のタスクを実行するためにツールが必要です—データベース検索からメッセージ送信まで。Amazon Bedrock AgentCore GatewayはAPI、Lambda関数、既存サービスをMCP互換ツールに自動変換するため、開発者は統合を管理することなく、これらの重要な機能をエージェントに迅速に提供できます。
* **Memory**: Amazon Bedrock AgentCore Memoryは、完全に管理されたメモリインフラストラクチャとニーズに合わせてメモリをカスタマイズする機能により、開発者が豊富でパーソナライズされたエージェント体験を簡単に構築できるようにします。
* **Identity**: Amazon Bedrock AgentCore Identityは、SlackやZoomなどのサードパーティアプリケーションとAWSサービス全体でシームレスなエージェントアイデンティティ・アクセス管理を提供し、Okta、Entra、Amazon Cognitoなどの標準的なアイデンティティプロバイダーをサポートします。
* **Tools**: Amazon Bedrock AgentCoreは、エージェンティックAIアプリケーション開発を簡素化するための2つの組み込みツールを提供します：Amazon Bedrock AgentCore **Code Interpreter**ツールは、AIエージェントが安全にコードを記述・実行することを可能にし、精度を向上させ、複雑なエンドツーエンドタスクを解決する能力を拡張します。Amazon Bedrock AgentCore **Browser Tool**は、AIエージェントがWebサイトをナビゲートし、複数ステップのフォームを完成させ、低レイテンシの完全に管理された安全なサンドボックス環境内で人間のような精度で複雑なWebベースのタスクを実行することを可能にするエンタープライズグレードの機能です
* **Observability**: Observabilityは、統一された運用ダッシュボードを通じて、開発者がエージェントパフォーマンスを追跡、デバッグ、監視することを支援します。OpenTelemetry互換のテレメトリーとエージェントワークフローの各ステップの詳細な可視化のサポートにより、Amazon Bedrock AgentCore Observabilityは開発者がエージェントの動作を簡単に把握し、大規模な品質基準を維持することを可能にします。

さらに、実用的なシナリオでこれらのコンポーネントを組み合わせる方法を示す**エンドツーエンド**の例も提供しています。

## Amazon Bedrock AgentCore

Amazon Bedrock AgentCoreサービスは、独立して使用することも、組み合わせて本番対応のエージェントを作成することもできます。これらは任意のエージェンティックフレームワーク（Strands Agents、LangChain、LangGraph、CrewAIなど）や任意のモデル（Amazon Bedrockで利用可能かどうかを問わず）で動作します。

![Amazon Bedrock AgentCore概要](images/agentcore_overview.png)

これらのチュートリアルでは、各サービスを個別にまた組み合わせて使用する方法を学習します。

## 🎯 これらのチュートリアルの対象者

これらのチュートリアルは以下の方に最適です：

 - Amazon Bedrock AgentCoreを始める方
 - 高度なアプリケーションを構築する前にコア概念を理解したい方
 - Amazon Bedrock AgentCoreを使用したAIエージェント開発の確実な基盤を得たい方