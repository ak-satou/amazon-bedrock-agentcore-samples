# Amazon Bedrock AgentCore RuntimeでのAmazon BedrockモデルによるLangGraphエージェントホスティング

## 概要

このチュートリアルでは、Amazon Bedrock AgentCore Runtimeを使用して既存のエージェントをホストする方法を学習します。 

Amazon BedrockモデルによるLangGraphの例に焦点を当てます。Amazon BedrockモデルによるStrands Agentsについては[こちら](../01-strands-with-bedrock-model)、OpenAIモデルによるStrands Agentsについては[こちら](../03-strands-with-openai-model)をご確認ください。

### チュートリアル詳細

| 情報         | 詳細                                                                      |
|:--------------------|:-----------------------------------------------------------------------------|
| チュートリアルタイプ       | 対話型                                                               |
| エージェントタイプ          | 単一                                                                       |
| エージェンティックフレームワーク   | LangGraph                                                                    |
| LLMモデル           | Anthropic Claude Sonnet 3.7                                                    |
| チュートリアルコンポーネント | AgentCore RuntimeでのエージェントホスティングLangGraphとAmazon Bedrockモデルの使用 |
| チュートリアル分野   | クロス分野                                                               |
| 例の複雑さ  | 簡単                                                                         |
| 使用SDK            | Amazon BedrockAgentCore Python SDKとboto3                                 |

### チュートリアルアーキテクチャ

このチュートリアルでは、既存のエージェントをAgentCore runtimeにデプロイする方法を説明します。 

デモンストレーション目的で、Amazon Bedrockモデルを使用するLangGraphエージェントを使用します

この例では、`get_weather`と`get_time`の2つのツールを持つ非常にシンプルなエージェントを使用します。 

<div style="text-align:left">
    <img src="images/architecture_runtime.png" width="100%"/>
</div>

### チュートリアル主要機能

* Amazon Bedrock AgentCore Runtimeでのエージェントホスティング
* Amazon Bedrockモデルの使用
* LangGraphの使用