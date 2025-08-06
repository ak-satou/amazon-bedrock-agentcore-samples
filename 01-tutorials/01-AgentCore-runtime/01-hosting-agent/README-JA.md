# AgentCore RuntimeでのAIエージェントホスティング

## 概要

このチュートリアルでは、**Amazon Bedrock AgentCore Runtime**でAIエージェントをホストする方法をAmazon Bedrock AgentCore Python SDKを使用して説明します。エージェントコードをAmazon Bedrockのインフラストラクチャとシームレスに統合する標準化されたHTTPサービスに変換する方法を学習します。

AgentCore Runtimeは、任意のエージェンティックフレームワーク（Strands Agents、LangGraph、CrewAI）と任意のLLMモデル（Amazon Bedrock、OpenAIなど）で構築されたエージェントをホストできる**フレームワーク・モデル非依存**プラットフォームです。

Amazon Bedrock AgentCore Python SDKは以下を行うラッパーとして機能します：

- エージェントコードをAgentCoreの標準化されたプロトコルに**変換**
- HTTPおよびMCPサーバーインフラストラクチャを自動的に**処理**
- エージェントのコア機能に**集中**できるようにします
- 2つのプロトコルタイプを**サポート**：
  - **HTTPプロトコル**: 従来のリクエスト/レスポンスREST APIエンドポイント
  - **MCPプロトコル**: ツールとエージェントサーバー用のModel Context Protocol

### サービスアーキテクチャ

エージェントをホストする際、SDKは自動的に：

- エージェントをポート`8080`でホスト
- 2つの主要エンドポイントを提供：
  - **`/invocations`**: プライマリエージェントインタラクション（JSON入力 → JSON/SSE出力）
  - **`/ping`**: 監視用ヘルスチェック

![エージェントホスティング](images/hosting_agent_python_sdk.png)

エージェントがAgentCore Runtimeでのデプロイ準備が整ったら、Amazon Bedrock AgentCore StarterKitを使用してAgentCore Runtimeにデプロイできます。

StarterKitを使用すると、エージェントデプロイメントを設定し、それを起動してエージェントの設定とAgentCore Runtimeエンドポイントを含むAmazon ECRリポジトリを作成し、検証のために作成されたエンドポイントを呼び出すことができます。

![StarterKit](../images/runtime_overview.png)

デプロイ後、AWSでのAgentCore Runtimeアーキテクチャは以下のようになります：

![RuntimeArchitecture](../images/runtime_architecture.png)

## チュートリアル例

このチュートリアルには、開始に役立つ3つの実践的な例が含まれています：

| 例                                                                | フレームワーク | モデル          | 説明                                    |
| ---------------------------------------------------------------------- | -------------- | -------------- | ------------------------------------------ |
| **[01-strands-with-bedrock-model](01-strands-with-bedrock-model)**     | Strands Agents | Amazon Bedrock | AWSネイティブモデルを使用した基本エージェントホスティング |
| **[02-langgraph-with-bedrock-model](02-langgraph-with-bedrock-model)** | LangGraph      | Amazon Bedrock | LangGraphエージェントワークフロー                  |
| **[03-strands-with-openai-model](03-strands-with-openai-model)**       | Strands Agents | OpenAI         | 外部LLMプロバイダーとの統合    |

## 主要な利点

- **フレームワーク非依存**: あらゆるPythonベースのエージェントフレームワークで動作
- **モデル柔軟性**: Amazon Bedrock、OpenAI、その他のLLMプロバイダーのLLMをサポート
- **本番対応**: 組み込みヘルスチェックと監視
- **簡単統合**: 最小限のコード変更で済む
- **スケーラブル**: エンタープライズワークロード向け設計

## 開始方法

好みのフレームワークとモデルの組み合わせに基づいて上記のチュートリアル例の1つを選択してください。各例には以下が含まれています：

- ステップバイステップのセットアップ手順
- 完全なコードサンプル
- テストガイドライン
- ベストプラクティス

## 次のステップ

チュートリアル完了後、以下が可能です：

- これらのパターンを他のフレームワークやモデルに拡張
- 本番環境にデプロイ
- 既存のアプリケーションと統合
- エージェントインフラストラクチャをスケール