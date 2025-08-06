# Amazon AgentCore Bedrock Code Interpreterを使用した高度なデータ分析 - チュートリアル

## 概要

このチュートリアルでは、Pythonを使用したコード実行を通じて高度なデータ分析を実行するAIエージェントの作成方法を示します。Amazon Bedrock AgentCore Code Interpreterを使用して、LLMによって生成されたコードを実行します。

このチュートリアルでは、AgentCore Bedrock Code Interpreterを使用して以下の方法を示します：

1. サンドボックス環境の設定
2. ユーザークエリに基づいてコードを生成することで高度なデータ分析を実行するStrandsおよびLangChainベースのエージェントの設定
3. Code Interpreterを使用したサンドボックス環境でのコード実行
4. ユーザーへの結果の表示



### チュートリアル詳細

| 情報 | 詳細 |
|:--------------------|:---------------------------------------------------------------------------------|
| チュートリアルタイプ | 対話型 |
| エージェントタイプ | 単一 |
| エージェントフレームワーク | Langchain & Strands Agents |
| LLMモデル | Anthropic Claude Sonnet 3.5 & 3.7 |
| チュートリアルコンポーネント | AmazonBedrock AgentCore Code Interpreter |
| チュートリアル分野 | 横断的 |
| 例の複雑さ | 初級 |
| 使用SDK | Amazon BedrockAgentCore Python SDK および boto3 |


### チュートリアルアーキテクチャ

コード実行サンドボックスは、コードインタープリタ、シェル、ファイルシステムを含む分離された環境を作成することで、エージェントがユーザーのクエリを安全に処理できるようにします。大規模言語モデルがツール選択を支援した後、コードはこのセッション内で実行され、その後ユーザーまたはエージェントに結果が返されて統合されます。

<div style="text-align:left">
    <img src="images/code_interpreter.png" width="100%"/>
</div>