# Amazon Bedrock AgentCore ブラウザツール

## 概要

Amazon Bedrock AgentCore ブラウザツールは、AIエージェントが人間と同じようにウェブサイトと安全で完全に管理されたやり取りを行う方法を提供します。エージェントがウェブページをナビゲートし、フォームに入力し、複雑なタスクを完了することを、開発者がカスタム自動化スクリプトを記述・維持する必要なく可能にします。

## 仕組み

ブラウザツールサンドボックスは、AIエージェントがウェブブラウザと安全に相互作用できるようにする安全な実行環境です。ユーザーがリクエストを行うと、大規模言語モデル（LLM）が適切なツールを選択し、コマンドを翻訳します。これらのコマンドは、ヘッドレスブラウザとホストされたライブラリサーバー（PlaywrightなどのツールJ使用）を含む制御されたサンドボックス環境内で実行されます。サンドボックスは、ウェブインタラクションを制限された空間内に封じ込めることで分離とセキュリティを提供し、不正なシステムアクセスを防ぎます。エージェントはスクリーンショットを通じてフィードバックを受け、システムセキュリティを維持しながら自動化されたタスクを実行できます。この設定により、AIエージェントの安全なウェブ自動化が可能になります。

![architecture local](../02-Agent-Core-browser-tool/images/browser-tool.png)

## 主要機能

### 安全で管理されたウェブインタラクション

AIエージェントに、人間と同じようにウェブサイトと相互作用する安全で完全に管理された方法を提供し、カスタム自動化スクリプトを必要とすることなく、ナビゲーション、フォーム入力、複雑なタスク完了を可能にします。

### エンタープライズセキュリティ機能

ユーザーセッションとブラウザセッション間の1:1マッピングによるVM レベルの分離を提供し、エンタープライズグレードのセキュリティを提供します。各ブラウザセッションは、エンタープライズセキュリティニーズを満たすために分離されたサンドボックス環境で実行されます。

### モデル非依存統合

interact()、parse()、discover()などのツールを通じてブラウザアクションの自然言語抽象化を提供しながら、さまざまなAIモデルとフレームワークをサポートし、特にエンタープライズ環境に適しています。ツールはあらゆるライブラリからブラウザコマンドを実行でき、PlaywrightやPuppeteerなど様々な自動化フレームワークをサポートします。

### 統合

Amazon Bedrock AgentCore ブラウザツールは、統合SDKを通じて他のAmazon Bedrock AgentCore機能と統合されます：

- Amazon Bedrock AgentCore Runtime
- Amazon Bedrock AgentCore Identity
- Amazon Bedrock AgentCore Memory
- Amazon Bedrock AgentCore Observability

この統合は開発プロセスを簡素化し、ブラウザベースのタスクを実行する強力な機能を備えたAIエージェントの構築、展開、管理のための包括的なプラットフォームを提供することを目的としています。

### ユースケース

Amazon Bedrock AgentCore ブラウザツールは、以下を含む幅広いアプリケーションに適しています：

- ウェブナビゲーションとインタラクション
- フォーム入力を含むワークフロー自動化

## チュートリアル概要

これらのチュートリアルでは、以下の機能をカバーします：

- [Bedrock AgentCore ブラウザツールとNovaActの開始](01-browser-with-NovaAct/01_getting_started-agentcore-browser-tool-with-nova-act.ipynb)
- [Amazon Bedrock AgentCore ブラウザツール ライブビューとNova Act](01-browser-with-NovaAct/02_agentcore-browser-tool-live-view-with-nova-act.ipynb)
- [Bedrock AgentCore ブラウザツールとBrowser useの開始](02-browser-with-browserUse/getting_started-agentcore-browser-tool-with-browser-use.ipynb)
- [Amazon Bedrock AgentCore ブラウザツール ライブビューとBrowser Use](02-browser-with-browserUse/agentcore-browser-tool-live-view-with-browser-use.ipynb)