# インテグレーション

このディレクトリには、Amazon Bedrock AgentCore と様々なサードパーティフレームワーク、サービス、プラットフォームとの統合例が含まれています。これらの例は、既存のツールやワークフローに AgentCore を組み込む方法を実演します。

## 利用可能なインテグレーション

### エージェンティックフレームワーク

AgentCore は幅広いエージェンティックフレームワークをサポートしています：

#### [ADK (Agent Development Kit)](agentic-frameworks/adk/)
Amazon の Agent Development Kit との統合例。Google検索機能を持つエージェントの実装。

#### [AutoGen](agentic-frameworks/autogen/)
Microsoft AutoGen フレームワークとの統合。マルチエージェント会話システムの構築。

#### [CrewAI](agentic-frameworks/crewai/)
CrewAI フレームワークとの統合。協調的なエージェントチームの構築。

#### [LangChain](agentic-frameworks/langchain/)
LangChain フレームワークとの統合。チェーン型のエージェントワークフロー。

#### [LangGraph](agentic-frameworks/langgraph/)
LangGraph を使用したグラフベースのエージェントワークフロー。Web検索機能を含む実装例。

#### [LlamaIndex](agentic-frameworks/llamaindex/)
LlamaIndex との統合。文書検索と質問応答エージェント。

#### [OpenAI Agents](agentic-frameworks/openai-agents/)
OpenAI の新しいエージェント API との統合。ハンドオフ機能とマルチエージェント実装。

#### [Strands Agents](agentic-frameworks/strands-agents/)
Strands エージェントフレームワークとの統合。ファイルシステム操作とストリーミング機能。

### Amazon Bedrock Agent との統合

#### [Bedrock Agent Integration](bedrock-agent/)
既存の Amazon Bedrock Agent と AgentCore Gateway の統合方法を示します。従来のBedrockエージェントをAgentCoreエコシステムに接続する手法。

## 各インテグレーションの特徴

### フレームワーク非依存性
- 任意のPythonベースエージェントフレームワークをサポート
- 最小限のコード変更で既存のエージェントを統合
- 統一されたAPIインターフェース

### モデルの柔軟性
- Amazon Bedrock モデル
- OpenAI モデル
- その他のLLMプロバイダー
- カスタムモデル統合

### プロトコルサポート
- **HTTP プロトコル**: RESTful API エンドポイント
- **MCP プロトコル**: Model Context Protocol for tools

## 開始方法

1. **フレームワークを選択**: 使用したいエージェンティックフレームワークを選択
2. **例を確認**: 対応するディレクトリの README.md を読む
3. **依存関係のインストール**: requirements.txt を使用してセットアップ
4. **設定**: 必要な環境変数とAPIキーを設定
5. **実行**: 提供されたサンプルコードを実行してテスト

## 共通する前提条件

- Python 3.8 以降
- AWS アカウントとプロファイル設定
- Amazon Bedrock へのアクセス権限
- 必要に応じて外部サービスのAPIキー（OpenAI、Google等）

## アーキテクチャパターン

### 標準的な統合フロー
1. **エージェント定義**: 選択したフレームワークでエージェントを定義
2. **AgentCore ラッパー**: AgentCore SDK でエージェントをラップ
3. **デプロイメント**: AgentCore Runtime にデプロイ
4. **呼び出し**: Gateway 経由でエージェントを呼び出し

### プロトコル選択
- **シンプルな要求/応答**: HTTP プロトコルを使用
- **ツール統合**: MCP プロトコルを使用
- **ストリーミング**: Server-Sent Events (SSE) をサポート

## ベストプラクティス

### セキュリティ
- APIキーの適切な管理
- IAMロールベースのアクセス制御
- ネットワークセキュリティの設定

### パフォーマンス
- 適切なタイムアウト設定
- エラーハンドリングとリトライ戦略
- リソース使用量の監視

### 開発・テスト
- ローカル開発環境の設定
- 単体テストとインテグレーションテスト
- 段階的デプロイメント

## コントリビューション

新しいフレームワークとの統合例や改善提案は歓迎します。詳細は [CONTRIBUTING.md](../CONTRIBUTING.md) をご覧ください。

## サポート

統合に関する質問や問題については、GitHub Issues を使用してください。各統合例には詳細なドキュメントとトラブルシューティングセクションが含まれています。