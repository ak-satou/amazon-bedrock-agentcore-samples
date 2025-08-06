# Amazon Bedrock AgentCore RuntimeでのStrands AgentとAmazon Bedrockモデルによるストリーミングレスポンス

## 概要

このチュートリアルでは、既存のエージェントでAmazon Bedrock AgentCore Runtimeを使用してストリーミングレスポンスを実装する方法を学習します。

リアルタイムストリーミング機能を実証するStrands AgentとAmazon Bedrockモデルの例に焦点を当てます。

### チュートリアル詳細

| 項目                | 詳細                                                                             |
|:-------------------|:---------------------------------------------------------------------------------|
| チュートリアルタイプ    | ストリーミング対応の会話型                                                          |
| エージェントタイプ     | 単一                                                                             |
| エージェント フレームワーク | Strands Agents                                                               |
| LLMモデル           | Anthropic Claude Sonnet 3.7                                                    |
| チュートリアル構成要素  | AgentCore Runtimeによるストリーミングレスポンス。Strands AgentとAmazon Bedrockモデルを使用 |
| チュートリアル分野     | 横断的                                                                           |
| 例の複雑さ          | 簡単                                                                             |
| 使用SDK            | Amazon BedrockAgentCore Python SDKとboto3                                       |

### チュートリアル アーキテクチャ

このチュートリアルでは、AgentCore runtimeにストリーミングエージェントをデプロイする方法について説明します。

デモンストレーションの目的で、ストリーミング機能を持つAmazon Bedrockモデルを使用するStrands Agentを使用します。

この例では、`get_weather`、`get_time`、`calculator`の3つのツールを持つシンプルなエージェントを使用しますが、リアルタイムストリーミングレスポンス機能で強化されています。

<div style="text-align:left">
    <img src="images/architecture_runtime.png" width="100%"/>
</div>

### チュートリアル主要機能

* Amazon Bedrock AgentCore Runtimeでのストリーミングレスポンスの実装
* Server-Sent Events（SSE）を使用したリアルタイム部分結果配信
* ストリーミング機能を持つAmazon Bedrockモデルの使用
* 非同期ストリーミングサポートを持つStrands Agentsの使用
* 段階的レスポンス表示による向上したユーザーエクスペリエンス