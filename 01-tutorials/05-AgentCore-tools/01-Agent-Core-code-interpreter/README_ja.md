# Amazon Bedrock AgentCore コードインタープリター

## 概要
Amazon Bedrock AgentCore コードインタープリターは、AIエージェントが直接コードを記述・実行してエンドツーエンドのタスクを完了できる、安全でサーバーレスな環境です。複雑なデータ分析の実行、シミュレーションの実行、可視化の生成、プログラミングタスクの自動化を可能にします。

## 仕組み

コード実行サンドボックスは、コードインタープリター、シェル、ファイルシステムを備えた分離された環境を作成することで、エージェントがユーザークエリを安全に処理できるようにします。大規模言語モデルがツール選択を支援した後、コードはこのセッション内で実行され、ユーザーまたはエージェントに合成のために返されます。

![architecture local](../01-Agent-Core-code-interpreter/images/code-interpreter.png)

## 主要機能

### 環境でのセッション

実行間でセッションを永続化する能力

### VPCサポートとインターネットアクセス

VPC接続と外部インターネットアクセスを含むエンタープライズグレードの機能を提供

### 複数の事前構築環境ランタイム

Python、NodeJS、TypeScript（近日提供予定：カスタムライブラリを使用したカスタムランタイムコード実行エンジンのサポート）を含む複数の事前構築ランタイム

### 統合

Amazon Bedrock AgentCore コードインタープリターは、統合SDKを通じて他のAmazon Bedrock AgentCore機能と統合されます：

- Amazon Bedrock AgentCore Runtime
- Amazon Bedrock AgentCore Identity
- Amazon Bedrock AgentCore Memory
- Amazon Bedrock AgentCore Observability

この統合は開発プロセスを簡素化し、強力なコード実行機能を備えたAIエージェントの構築、展開、管理のための包括的なプラットフォームを提供することを目的としています。

### ユースケース

Amazon Bedrock AgentCore コードインタープリターは、以下を含む幅広いアプリケーションに適しています：

- コード実行とレビュー
- データ分析と可視化

## チュートリアル概要

これらのチュートリアルでは、以下の機能をカバーします：

- [Amazon Bedrock AgentCore コードインタープリターを使用したファイル操作](01-file-operations-using-code-interpreter)
- [Amazon Bedrock AgentCore コードインタープリターを使用したエージェントでのコード実行](02-code-execution-with-agent-using-code-interpreter)
- [Amazon Bedrock AgentCore コードインタープリターを使用したAIエージェントでの高度なデータ分析](03-advanced-data-analysis-with-agent-using-code-interpreter)
- [Amazon Bedrock AgentCore コードインタープリターを使用したコマンド実行](04-run-commands-using-code-interpreter)