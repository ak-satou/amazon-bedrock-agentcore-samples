# Amazon Bedrock AgentCore RuntimeでのOpenAIモデルによるStrands Agentsホスティング

## 概要

このチュートリアルでは、Amazon Bedrock AgentCore Runtimeを使用して既存のエージェントをホストする方法を学習します。 

OpenAIモデルによるStrands Agentsの例に焦点を当てます。Amazon BedrockモデルによるStrands Agentsについては[こちら](../01-strands-with-bedrock-model)、Amazon BedrockモデルによるLangGraphについては[こちら](../02-langgraph-with-bedrock-model)をご確認ください。

### チュートリアル詳細

| 情報         | 詳細                                                                  |
|:--------------------|:-------------------------------------------------------------------------|
| チュートリアルタイプ       | 対話型                                                           |
| エージェントタイプ          | 単一                                                                   |
| エージェンティックフレームワーク   | Strands Agents                                                           |
| LLMモデル           | GPT 4.1 mini                                                             |
| チュートリアルコンポーネント | AgentCore RuntimeでのエージェントホスティングStrands AgentとOpenAIモデルの使用 |
| チュートリアル分野   | クロス分野                                                           |
| 例の複雑さ  | 簡単                                                                     |
| 使用SDK            | Amazon BedrockAgentCore Python SDKとboto3                             |

### チュートリアルアーキテクチャ

このチュートリアルでは、既存のエージェントをAgentCore runtimeにデプロイする方法を説明します。 

デモンストレーション目的で、Amazon Bedrockモデルを使用するStrands Agentを使用します

この例では、`get_weather`と`get_time`の2つのツールを持つ非常にシンプルなエージェントを使用します。 

<div style="text-align:left">
    <img src="images/architecture_runtime.png" width="100%"/>
</div>

### チュートリアル主要機能

* Amazon Bedrock AgentCore Runtimeでのエージェントホスティング
* OpenAIモデルの使用
* Strands Agentsの使用