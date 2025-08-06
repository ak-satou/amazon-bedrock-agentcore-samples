# Amazon Bedrock AgentCore Tools

## 概要
Amazon Bedrock AgentCore Toolsは、AIエージェントが複雑なタスクを安全かつ効率的に実行する能力を向上させるエンタープライズグレードの機能を提供します。このスイートには2つの主要ツールが含まれています：

- Amazon Bedrock AgentCore Code Interpreter
- Amazon Bedrock AgentCore Browser Tool

## Amazon Bedrock AgentCore Code Interpreter

### 主要機能

1. **安全なコード実行**: 分離されたサンドボックス環境でコードを実行し、内部データソースにアクセスしながらセキュリティを確保します。

2. **完全管理されたAWSネイティブソリューション**: Strands Agents、LangGraph、CrewAIなどのフレームワークとシームレスに統合されます。

3. **高度な設定サポート**: 入力と出力の両方での大容量ファイルサポートとインターネットアクセスを含みます。

4. **複数言語サポート**: JavaScript、TypeScript、Pythonを含むさまざまなプログラミング言語用の事前構築されたランタイムモード。

### 利点

- **エージェント精度の向上**: エージェントが複雑な計算とデータ処理を実行できるようになります。
- **エンタープライズグレードのセキュリティ**: 分離された環境で厳格なセキュリティ要件を満たします。
- **効率的なデータ処理**: Amazon S3のファイルを参照してギガバイト規模のデータを処理できます。

## Amazon Bedrock AgentCore Browser Tool

### 主要機能

1. **モデル非依存の柔軟性**: Anthropic のClaude、OpenAIのモデル、AmazonのNovaモデルを含む、さまざまなAIモデルからの多様なコマンド構文をサポートします。

2. **エンタープライズグレードのセキュリティ**: VM レベルの分離、VPC 接続、およびエンタープライズSSO システムとの統合を提供します。

3. **包括的な監査機能**: すべてのブラウザコマンドのCloudTrailログ記録とセッション記録機能を含みます。

### 利点

- **エンドツーエンド自動化**: AIエージェントが以前は手動介入が必要だった複雑なWebワークフローを自動化できるようになります。
- **強化されたセキュリティ**: 広範なセキュリティ機能と監査機能でエンタープライズ要件を満たします。
- **リアルタイム監視**: 即座の介入のためのLive Viewとデバッグ・監査のためのSession Replayを提供します。

## ユースケース

- 安全な環境での複雑なデータ分析と可視化
- フォーム入力、データ抽出、マルチステップ プロセスのための自動化されたWeb インタラクション
- 大規模データ処理と監視
- エンタープライズ設定でのAIエージェント用の安全なコード実行

## チュートリアル概要

1. [Amazon Bedrock AgentCore Code Interpreter](01-Agent-Core-code-interpreter)
2. [Amazon Bedrock AgentCore Browser Tool](02-Agent-Core-browser-tool)