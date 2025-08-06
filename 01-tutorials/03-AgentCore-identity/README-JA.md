# AgentCore における AI エージェントのホスティング

## 概要

このチュートリアルでは、Amazon Bedrock AgentCore Python SDK を使用して、**Amazon Bedrock AgentCore Runtime** 上で AI エージェントをホストする方法を説明します。エージェントコードを標準化された HTTP サービスに変換し、Amazon Bedrock のインフラストラクチャとシームレスに統合する方法を学びます。

AgentCore Runtime は、**フレームワークとモデルに依存しない**プラットフォームであり、任意のエージェンティックフレームワーク（Strands Agents、LangGraph、CrewAI）および任意の LLM モデル（Amazon Bedrock、OpenAI など）で構築されたエージェントをホストできます。

Amazon Bedrock AgentCore Python SDK は、以下の機能を提供するラッパーとして機能します：

- **変換**: エージェントコードを AgentCore の標準化されたプロトコルに変換
- **処理**: HTTP および MCP サーバーインフラストラクチャを自動的に処理
- **集中**: エージェントのコア機能に集中できるようにサポート
- **サポート**: 2つのプロトコルタイプをサポート：
  - **HTTP プロトコル**: 従来のリクエスト/レスポンス REST API エンドポイント
  - **MCP プロトコル**: ツールとエージェントサーバー用の Model Context Protocol

### サービスアーキテクチャ

エージェントをホストする際、SDK は自動的に以下を実行します：
- エージェントをポート `8080` でホスト
- 2つの主要なエンドポイントを提供：
  - **`/entrypoint`**: プライマリエージェントインタラクション（JSON入力 → JSON/SSE出力）
  - **`/ping`**: 監視用のヘルスチェック

![Hosting agent](images/hosting_agent_python_sdk.png)

エージェントが AgentCore Runtime でのデプロイメント準備が整ったら、Amazon Bedrock AgentCore StarterKit を使用して AgentCore Runtime にデプロイできます。

StarterKit を使用すると、エージェントデプロイメントの設定、起動して Amazon ECR リポジトリとエージェントの設定および AgentCore Runtime エンドポイントの作成、検証のための作成済みエンドポイントの呼び出しが可能です。

![StarterKit](../images/runtime_overview.png)

デプロイメント後、AWS における AgentCore Runtime アーキテクチャは以下のようになります：

![RuntimeArchitecture](../images/runtime_architecture.png)

## チュートリアル例

このチュートリアルには、開始するための3つの実践的な例が含まれています：

| 例 | フレームワーク | モデル | 説明                                |
|---------|-----------|-------|--------------------------------------------|
| **[01-strands-with-bedrock-model](01-strands-with-bedrock-model)** | Strands Agents | Amazon Bedrock | AWS ネイティブモデルを使用した基本的なエージェントホスティング |
| **[02-langgraph-with-bedrock-model](02-langgraph-with-bedrock-model)** | LangGraph | Amazon Bedrock | LangGraph エージェントワークフロー                  |
| **[03-strands-with-openai-model](03-strands-with-openai-model)** | Strands Agents | OpenAI | 外部 LLM プロバイダーとの統合    |

## 主要な利点

- **フレームワーク非依存**: 任意の Python ベースのエージェントフレームワークで動作
- **モデル柔軟性**: Amazon Bedrock、OpenAI、その他の LLM プロバイダーをサポート
- **本番環境対応**: 組み込みのヘルスチェックと監視機能
- **簡単な統合**: 最小限のコード変更で実装可能
- **スケーラブル**: エンタープライズワークロード向けに設計

## 開始方法

上記のチュートリアル例の中から、お好みのフレームワークとモデルの組み合わせに基づいて選択してください。各例には以下が含まれています：
- ステップバイステップのセットアップ手順
- 完全なコードサンプル
- テストガイドライン
- ベストプラクティス

## 次のステップ

チュートリアルを完了した後、以下が可能です：
- これらのパターンを他のフレームワークやモデルに拡張
- 本番環境へのデプロイ
- 既存のアプリケーションとの統合
- エージェントインフラストラクチャのスケーリング