# Amazon CloudWatchでのAgentCore Observability

このリポジトリは、Amazon CloudWatchとOpenTelemetryを使用してエージェント用のAgentCore Observabilityを実装する方法を示しています。Amazon Bedrock AgentCore Runtime ホストエージェントと人気のオープンソースエージェントフレームワークの両方の例を提供します。

## プロジェクト構造

```
06-AgentCore-observability/
├── 01-Agentcore-runtime-hosted/
│   ├── images/
│   ├── .env.example
│   ├── README.md
│   ├── requirements.txt
│   └── runtime_with_strands_and_bedrock_models.ipynb
├── 02-Agent-not-hosted-on-runtime/
│   ├── CrewAI/
│   │   ├── .env.example
│   │   ├── CrewAI_Observability.ipynb
│   │   └── requirements.txt
│   ├── Langgraph/
│   │   ├── .env.example
│   │   ├── Langgraph_Observability.ipynb
│   │   └── requirements.txt
│   ├── Strands/
│   │   ├── .env.example
│   │   ├── requirements.txt
│   │   └── Strands_Observability.ipynb
│   └── README.md
├── 03-advanced-concepts/
│   └── 01-custom-span-creation/
│       ├── .env.example
│       ├── Custom_Span_Creation.ipynb
│       └── requirements.txt
├── README.md
└── utils.py
```

## 概要

このリポジトリは、開発者がGenAIアプリケーション用のObservabilityを実装するのに役立つ例とツールを提供します。AgentCore Observabilityは、統一された運用ダッシュボードを通じて、開発者がエージェントのパフォーマンスを追跡、デバッグ、監視することを支援します。OpenTelemetry互換のテレメトリとエージェントワークフローの各ステップの詳細な可視化のサポートにより、Amazon CloudWatch GenAI Observabilityは開発者がエージェントの動作を簡単に把握し、大規模な品質基準を維持することを可能にします。

## 内容

### 1. Bedrock AgentCore Runtime ホスト型（01-Agentcore-runtime-hosted）

Amazon OpenTelemetry Python InstrumentationとAmazon CloudWatchを使用して、Amazon Bedrock AgentCore Runtimeでホストされているストランドエージェントの観測性を示す例。

### 2. オープンソースエージェントフレームワーク（02-open-source-agents-3p）

Amazon Bedrock AgentCore Runtimeでホストされていない人気のオープンソースエージェントフレームワークの観測性を示す例：

- **CrewAI**: タスクを達成するためにロールで連携して作業する自律的なAIエージェントを作成
- **LangGraph**: 複雑な推論システム用のステートフル、マルチアクターアプリケーションでLangChainを拡張
- **Strands Agents**: モデル駆動エージェント開発を使用した複雑なワークフローでLLMアプリケーションを構築

### 3. 高度な概念（03-advanced-concepts）

高度な観測性パターンと技術：

- **カスタムスパン作成**: エージェントワークフロー内の特定の操作の詳細な追跡と監視のためのカスタムスパンを作成する方法を学習

## 開始方法

1. 探索したいフレームワークのディレクトリに移動
2. 要件をインストール
3. AWS認証情報を設定
4. `.env.example`ファイルを`.env`にコピーして変数を更新
5. Jupyter ノートブックを開いて実行

## 前提条件

- 適切な権限を持つAWSアカウント
- Python 3.10+
- Jupyter ノートブック環境
- 認証情報で設定されたAWS CLI
- Transaction Searchを有効化

## クリーンアップ

不要な料金を避けるために、例を完了した後はAmazon CloudWatchで作成されたログ・グループと関連リソースを削除してください。

## ライセンス

このプロジェクトは、リポジトリで指定された条件の下でライセンスされています。