# デモ環境

SREエージェントには、インフラストラクチャ操作をシミュレートするデモ環境が含まれています。これにより、本番システムに接続することなく、システムの機能を探索できます。

**重要な注意**: [`backend/data`](../backend/data)のデータは合成的に生成されており、backendディレクトリには実際のSREエージェントバックエンドがどのように動作するかを示すスタブサーバーが含まれています。本番環境では、これらの実装を実際のシステムに接続し、ベクターデータベースを使用し、他のデータソースと統合する実際の実装に置き換える必要があります。このデモは、バックエンドコンポーネントがプラグアンドプレイで交換可能に設計されているアーキテクチャの説明として機能します。

## デモバックエンドの開始

> **🔒 SSL要件:** AgentCore Gatewayを使用する場合、バックエンドサーバーはHTTPSで実行する必要があります。クイックスタートセクションのSSLコマンドを使用してください。

デモバックエンドは、異なるインフラストラクチャドメインに対してリアルな応答を提供する4つの専門APIサーバーで構成されています：

```bash
# すべてのデモサーバーをSSLで開始（推奨）
cd backend

# サーバーバインディング用のプライベートIPを取得
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600" -s)
PRIVATE_IP=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  -s http://169.254.169.254/latest/meta-data/local-ipv4)

# SSL証明書で開始
./scripts/start_demo_backend.sh \
  --host $PRIVATE_IP \
  --ssl-keyfile /etc/ssl/private/privkey.pem \
  --ssl-certfile /etc/ssl/certs/fullchain.pem

# 代替: SSLなしで開始（テスト専用 - AgentCore Gatewayと互換性なし）
# ./scripts/start_demo_backend.sh --host 0.0.0.0

# スクリプトは4つのAPIサーバーを開始します:
# - Kubernetes API（ポート8011）: 複数の名前空間を持つK8sクラスタをシミュレート
# - Logs API（ポート8012）: エラー注入を含む検索可能なアプリケーションログを提供
# - Metrics API（ポート8013）: 異常を含むリアルなパフォーマンスメトリクスを生成
# - Runbooks API（ポート8014）: 運用手順とトラブルシューティングガイドを提供
```

## デモシナリオ

デモ環境には、SREエージェントの機能を示すいくつかの事前設定されたシナリオが含まれています：

**データベースポッド障害シナリオ**: デモには、関連するエラーログとリソース枯渇メトリクスを伴う本番名前空間での障害データベースポッドが含まれています。このシナリオは、エージェントがどのように協力してメモリリークを根本原因として特定するかを示します。

**APIゲートウェイレイテンシシナリオ**: 対応するスロークエリログとCPUスパイクを伴うAPIゲートウェイでの高レイテンシをシミュレート。これは、システムが異なるデータソース間で問題を関連付ける能力を示します。

**カスケード障害シナリオ**: 障害した認証サービスが複数のサービス間でカスケード障害を引き起こす複雑なシナリオ。これは、エージェントが分散システムを通じて問題を追跡する能力を示します。

## デモデータのカスタマイズ

デモデータは`backend/data/`下のJSONファイルに保存されており、特定のユースケースに合わせてカスタマイズできます：

```bash
backend/data/
├── k8s_data/
│   ├── pods.json         # ポッドの定義とステータス
│   ├── deployments.json  # デプロイメント設定
│   └── events.json       # クラスタイベント
├── logs_data/
│   └── application_logs.json  # さまざまな重要度レベルのログエントリ
├── metrics_data/
│   └── performance_metrics.json  # 時系列メトリクスデータ
└── runbooks_data/
    └── runbooks.json     # 運用手順
```

## デモの停止

```bash
# すべてのデモサーバーを停止
cd backend
./scripts/stop_demo_backend.sh
```