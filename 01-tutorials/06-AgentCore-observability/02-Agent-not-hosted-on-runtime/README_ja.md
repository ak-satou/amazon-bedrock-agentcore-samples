# AgentCore Runtimeでホストされていないオープンソースエージェントのための AgentCore Observability

このリポジトリには、Amazon OpenTelemetry Python InstrumentationとAmazon CloudWatchを使用した、Amazon Bedrock AgentCore Runtime で**ホストされていない**人気のあるAIオープンソースフレームワークのAgentCore Observabilityを紹介する例が含まれています。Observabilityは、統合された運用ダッシュボードを通じて、開発者が本格的な環境でエージェントのパフォーマンスをトレース、デバッグ、監視するのに役立ちます。OpenTelemetry互換のテレメトリーサポートと、エージェントワークフローの各ステップの詳細な可視化により、Amazon CloudWatch GenAI Observabilityは、開発者がエージェントの動作に簡単に可視性を得て、大規模な品質基準を維持できるようにします。

以下のオープンソースエージェントフレームワークでエージェントを作成します：

- **CrewAI**
- **LangGraph**
- **Strands Agents**

## プロジェクト構造

```
02-open-source-agents-3p/
├── CrewAI/
│   ├── .env.example
│   ├── CrewAI_Observability.ipynb
│   └── requirements.txt
├── Langgraph/
│   ├── .env.example
│   ├── Langgraph_Observability.ipynb
│   └── requirements.txt
└── Strands/
    ├── .env.example
    ├── requirements.txt
    └── Strands_Observability.ipynb
```

## 使い始める

各フレームワークには独自のディレクトリがあり、以下が含まれています：
- フレームワークの機能を実証するJupyterノートブック
- 必要な依存関係をリストするrequirements.txtファイル
- 必要な環境変数を示す.env.exampleファイル

## 使用方法

1. 探索したいフレームワークのディレクトリに移動する
2. 要件をインストールする：`pip install -r requirements.txt`
3. AWS認証情報を設定する
3. .env.exampleファイルを.envにコピーし、変数を更新する
4. Jupyterノートブックを開いて実行する

## フレームワーク概要

### CrewAI
[CrewAI](https://www.crewai.com/)は、役割を持ってタスクを達成するために協力できる自律的なAIエージェントの作成を有効にします。

### LangGraph
[LangGraph](https://www.langchain.com/langgraph)は、ステートフルなマルチアクターアプリケーションでLangChainを拡張します。LLMを使用した複雑な推論システムの作成に特に有用です。

### Strands Agents
[Strands](https://strandsagents.com/latest/)は、モデル駆動のエージェント開発に焦点を当て、複雑なワークフローを持つLLMアプリケーションを構築するためのフレームワークを提供します。

## クリーンアップ

Amazon CloudWatchで作成されたログループと関連リソースを削除してください。