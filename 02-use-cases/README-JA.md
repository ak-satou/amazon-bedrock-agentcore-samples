# ユースケース例

このディレクトリには、Amazon Bedrock AgentCore を使用した実世界のユースケース実装例が含まれています。これらの例は、さまざまな業界やシナリオでエージェントソリューションを構築する方法を実演します。

## 利用可能なユースケース

### 1. [AWS Operations Agent](AWS-operations-agent/)
AWS リソースの運用、監視、管理を支援するエージェント。システム管理者や DevOps エンジニア向け。

### 2. [Database Performance Analyzer](DB-performance-analyzer/)
データベースのパフォーマンス分析と最適化を支援するエージェント。DBA や開発者向け。

### 3. [SRE Agent](SRE-agent/)
Site Reliability Engineering（SRE）のタスクとインシデント対応を支援するエージェント。

### 4. [Asynchronous Shopping Assistant](asynchronous-shopping-assistant/)
非同期ショッピング体験を提供するエージェント。eコマースアプリケーション向け。

### 5. [Customer Support Assistant](customer-support-assistant/)
カスタマーサポートと問い合わせ対応を自動化するエージェント。

### 6. [Device Management Agent](device-management-agent/)
IoT デバイスや企業デバイスの管理を支援するエージェント。

### 7. [Healthcare Appointment Agent](healthcare-appointment-agent/)
医療機関での予約管理と患者対応を支援するエージェント。

### 8. [Local Prototype to AgentCore](local-prototype-to-agentcore/)
ローカルプロトタイプから AgentCore への移行方法を示すガイド。

### 9. [Text to Python IDE](text-to-python-ide/)
自然言語からPythonコードを生成し、実行環境を提供するIDE風エージェント。

### 10. [Video Games Sales Assistant](video-games-sales-assistant/)
ゲーム販売データの分析と洞察を提供するエージェント。

## 各ユースケースの構成

各ユースケースディレクトリには以下が含まれています：

- **README.md**: 詳細な設定手順と使用方法
- **デプロイメントスクリプト**: AWS環境への自動デプロイメント
- **設定ファイル**: エージェントの動作設定
- **要件ファイル**: 必要な依存関係
- **テスト/検証スクリプト**: 機能検証用

## 開始方法

1. 興味のあるユースケースディレクトリを選択
2. そのディレクトリの README.md を確認
3. 前提条件をインストール
4. 設定手順に従ってセットアップ
5. エージェントをテストして動作確認

## 共通する前提条件

- AWS アカウントとプロファイル設定
- Python 3.8 以降
- Amazon Bedrock へのアクセス権限
- AgentCore Runtime の基本理解

## サポートとコントリビューション

これらのユースケースは学習と開発の出発点として提供されています。本番環境での使用前に、セキュリティ、スケーラビリティ、コンプライアンス要件を確認してください。

コントリビューションやフィードバックは歓迎します。詳細は [CONTRIBUTING.md](../CONTRIBUTING.md) をご覧ください。