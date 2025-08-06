# Amazon Bedrock AgentCore Runtime

## 概要
Amazon Bedrock AgentCore Runtimeは、AIエージェントとツールをデプロイ・スケールするために設計された安全なサーバーレスランタイムです。
任意のフレームワーク、モデル、プロトコルをサポートし、開発者は最小限のコード変更でローカルプロトタイプを本番対応ソリューションに変換できます。

Amazon BedrockAgentCore Python SDKは、エージェント関数をAmazon Bedrockと互換性のあるHTTPサービスとしてデプロイするのに役立つ軽量ラッパーを提供します。HTTPサーバーの詳細をすべて処理するため、エージェントのコア機能に集中できます。

必要なのは、関数を`@app.entrypoint`デコレーターで装飾し、SDKの`configure`と`launch`機能を使用してエージェントをAgentCore Runtimeにデプロイすることだけです。その後、アプリケーションはSDKまたはboto3、AWS SDK for JavaScript、AWS SDK for JavaなどのAWSの開発者ツールを使用してこのエージェントを呼び出すことができます。

![Runtime概要](images/runtime_overview.png)

## 主要機能

### フレームワークとモデルの柔軟性

- 任意のフレームワーク（Strands Agents、LangChain、LangGraph、CrewAIなど）からエージェントとツールをデプロイ
- 任意のモデル（Amazon Bedrockかどうかを問わず）を使用

### 統合

Amazon Bedrock AgentCore Runtimeは、統合SDKを通じて他のAmazon Bedrock AgentCore機能と統合されます：

- Amazon Bedrock AgentCore Memory
- Amazon Bedrock AgentCore Gateway
- Amazon Bedrock AgentCore Observability
- Amazon Bedrock AgentCore Tools

この統合は、開発プロセスを簡素化し、AIエージェントの構築、デプロイ、管理のための包括的なプラットフォームを提供することを目的としています。

### ユースケース

このランタイムは、以下を含む幅広いアプリケーションに適しています：

- リアルタイム、インタラクティブAIエージェント
- 長時間実行される複雑なAIワークフロー
- マルチモーダルAI処理（テキスト、画像、音声、動画）

## チュートリアル概要

これらのチュートリアルでは、以下の機能をカバーします：

- [エージェントのホスティング](01-hosting-agent)
- [MCPサーバーのホスティング](02-hosting-MCP-server)
- [高度な概念](03-advanced-concepts)