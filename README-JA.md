# Amazon Bedrock AgentCore サンプル

Amazon Bedrock AgentCore サンプルリポジトリへようこそ！

> [!CAUTION]
> このリポジトリで提供されている例は、実験的・教育目的のみのものです。概念と技術を示すものですが、本番環境での直接使用を意図したものではありません。[プロンプトインジェクション](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-injection.html)から保護するために、Amazon Bedrock Guardrailsが適切に配置されていることを確認してください。

**Amazon Bedrock AgentCore**は、任意のエージェンティックフレームワークと任意のLLMモデルを使用して、エージェントを安全かつ大規模にデプロイ・運用するための完全な機能セットです。これにより、開発者はAIエージェントを迅速に本番環境に投入し、ビジネス価値実現までの時間を短縮することができます。

Amazon Bedrock AgentCoreは、エージェントをより効果的で高性能にするためのツールと機能、エージェントを安全にスケールするための専用インフラストラクチャ、信頼できるエージェントを運用するための制御機能を提供します。

Amazon Bedrock AgentCoreの機能は組み合わせ可能で、人気のオープンソースフレームワークや任意のモデルと連携するため、オープンソースの柔軟性とエンタープライズグレードのセキュリティ・信頼性のどちらかを選ぶ必要がありません。

このコレクションは、Amazon Bedrock AgentCore機能の理解、実装、およびアプリケーションへの統合を支援するための例とチュートリアルを提供しています。

## 📁 リポジトリ構造

### 📚 [`01-tutorials/`](./01-tutorials/)
**インタラクティブ学習・基礎**

このフォルダには、Amazon Bedrock AgentCore機能の基礎を実践的な例を通じて学習するノートブックベースのチュートリアルが含まれています。

構造はAgentCoreコンポーネント別に分けられています：
* **Runtime**: Amazon Bedrock AgentCore Runtimeは、フレームワーク、プロトコル、モデルの選択に関係なく、AIエージェントとツールの両方をデプロイ・スケールすることを可能にする安全なサーバーレスランタイム機能です—迅速なプロトタイピング、シームレスなスケーリング、市場投入時間の短縮を実現します
* **Gateway**: AIエージェントは実世界のタスクを実行するためにツールが必要です—データベース検索からメッセージ送信まで。Amazon Bedrock AgentCore GatewayはAPI、Lambda関数、既存サービスをMCP互換ツールに自動変換するため、開発者は統合を管理することなく、これらの重要な機能をエージェントに迅速に提供できます。
* **Memory**: Amazon Bedrock AgentCore Memoryは、完全に管理されたメモリインフラストラクチャとニーズに合わせてメモリをカスタマイズする機能により、開発者が豊富でパーソナライズされたエージェント体験を簡単に構築できるようにします。
* **Identity**: Amazon Bedrock AgentCore Identityは、SlackやZoomなどのサードパーティアプリケーションとAWSサービス全体でシームレスなエージェントアイデンティティ・アクセス管理を提供し、Okta、Entra、Amazon Cognitoなどの標準的なアイデンティティプロバイダーをサポートします。
* **Tools**: Amazon Bedrock AgentCoreは、エージェンティックAIアプリケーション開発を簡素化するための2つの組み込みツールを提供します：Amazon Bedrock AgentCore **Code Interpreter**ツールは、AIエージェントが安全にコードを記述・実行することを可能にし、精度を向上させ、複雑なエンドツーエンドタスクを解決する能力を拡張します。Amazon Bedrock AgentCore **Browser Tool**は、AIエージェントがWebサイトをナビゲートし、複数ステップのフォームを完成させ、低レイテンシの完全に管理された安全なサンドボックス環境内で人間のような精度で複雑なWebベースのタスクを実行することを可能にするエンタープライズグレードの機能です
* **Observability**: Amazon Bedrock AgentCore Observabilityは、統一された運用ダッシュボードを通じて、開発者がエージェントパフォーマンスを追跡、デバッグ、監視することを支援します。OpenTelemetry互換のテレメトリーとエージェントワークフローの各ステップの詳細な可視化のサポートにより、Amazon Bedrock AgentCore Observabilityは開発者がエージェントの動作を簡単に把握し、大規模な品質基準を維持することを可能にします。

**エンドツーエンド例**フォルダは、ユースケースで異なる機能を組み合わせる方法の簡単な例を提供します。

提供される例は、AIエージェントアプリケーションを構築する前に基本概念を理解したい初心者や学習者に最適です。

### 💡 [`02-use-cases/`](./02-use-cases/)
**エンドツーエンドアプリケーション**

Amazon Bedrock AgentCore機能を実際のビジネス問題解決に適用する方法を示す実用的なユースケース実装を探索します。

各ユースケースには、AgentCoreコンポーネントに焦点を当てた詳細な説明付きの完全な実装が含まれています。

### 🔌 [`03-integrations/`](./03-integrations/)
**フレームワーク・プロトコル統合**

Strands Agents、LangChain、CrewAIなどの人気のあるエージェンティックフレームワークとAmazon Bedrock AgentCore機能を統合する方法を学習します。

A2Aを使用したエージェント間通信と異なるマルチエージェント協調パターンを設定します。エージェンティックインターフェースを統合し、異なるエントリーポイントでAmazon Bedrock AgentCoreを使用する方法を学びます。

## 🚀 クイックスタート

**リポジトリをクローン**

   ```bash
   git clone https://github.com/awslabs/amazon-bedrock-agentcore-samples.git
   ```

エージェントをAgentCore Runtimeにデプロイするための前提条件をインストールする必要があります。環境を稼働させるために以下の手順に従ってください：

1. DockerまたはFinchをインストールします。[こちら](https://www.docker.com/get-started/)から始められます
1. DockerまたはFinchが実行されていることを確認してください
1. より良いパッケージ制御のために、アプリケーションを実行するための仮想環境を作成することを強く推奨します。`uv`ツールは、Python用の高速パッケージ・プロジェクトマネージャーです。ここでは環境管理に`uv`を使用することを推奨します。[こちら](https://docs.astral.sh/uv/getting-started/installation/)の手順で`uv`をインストールできます
1. `uv`をインストールしたら、以下のコマンドを使用して新しい環境を作成・アクティベートします：
```commandline
uv python install 3.10
uv venv --python 3.10
source .venv/bin/activate
uv init
```
次に、必要なパッケージを`uv`環境に追加します：
```commandline
uv add -r requirements.txt --active
```
`uv`環境からJupyterノートブックインスタンスを開始できます：
```commandline
uv run --with jupyter jupyter lab
```

## 📋 前提条件

- Python 3.10以上
- AWSアカウント
- DockerまたはFinchがインストール・実行されていること
- Jupyter Notebook（チュートリアル用）

## 🤝 コントリビューション

コントリビューションを歓迎します！詳細については[コントリビューションガイドライン](CONTRIBUTING.md)をご覧ください：

- 新しいサンプルの追加
- 既存の例の改善
- 問題の報告
- 機能拡張の提案

## 📄 ライセンス

このプロジェクトはApache License 2.0の下でライセンスされています。詳細は[LICENSE](LICENSE)ファイルをご覧ください。

## 🆘 サポート

- **課題**: [GitHub Issues](https://github.com/awslabs/amazon-bedrock-agentcore-samples/issues)経由でバグ報告や機能要求を行ってください
- **ドキュメント**: 特定のガイダンスについては各フォルダのREADMEをご確認ください

## 🔄 更新

このリポジトリは積極的にメンテナンスされ、新しい機能と例で更新されています。リポジトリをウォッチして最新の追加情報を把握してください。