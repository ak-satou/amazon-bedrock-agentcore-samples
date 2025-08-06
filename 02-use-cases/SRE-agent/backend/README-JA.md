# バックエンドデモインフラストラクチャ

このディレクトリには、SREエージェントのテストと開発のための完全なデモバックエンドインフラストラクチャが含まれています。

## 📁 構造

```
backend/
├── config_utils.py               # 設定ユーティリティ
├── data/                         # 組織化された模擬データ
│   ├── k8s_data/                # Kubernetesモックデータ
│   ├── logs_data/               # アプリケーションログ
│   ├── metrics_data/            # パフォーマンスメトリクス
│   └── runbooks_data/           # 運用手順書
├── openapi_specs/               # API仕様
│   ├── k8s_api.yaml            # Kubernetes API仕様
│   ├── logs_api.yaml           # ログAPI仕様
│   ├── metrics_api.yaml        # メトリクスAPI仕様
│   └── runbooks_api.yaml       # ランブックAPI仕様
├── servers/                     # モックAPI実装
│   ├── k8s_server.py           # Kubernetes APIサーバー
│   ├── logs_server.py          # ログAPIサーバー
│   ├── metrics_server.py       # メトリクスAPIサーバー
│   ├── runbooks_server.py      # ランブックAPIサーバー
│   ├── run_all_servers.py      # 全サーバー起動
│   └── stop_servers.py         # 全サーバー停止
└── scripts/                    # 運用スクリプト
    ├── start_demo_backend.sh   # 簡易起動
    └── stop_demo_backend.sh    # 簡易停止
```

## 🚀 クイックスタート

### 簡易起動（推奨）
```bash
# シンプルなPython HTTPサーバーでデモサーバーを起動
./scripts/start_demo_backend.sh
```

### 高度な起動（完全なFastAPIサーバー）
```bash
# FastAPIを使用した機能豊富なサーバーを起動
cd servers
python run_all_servers.py
```

## 🌐 APIエンドポイント

実行時、デモバックエンドは以下のエンドポイントを提供します：

- **Kubernetes API**: http://localhost:8001
- **ログAPI**: http://localhost:8002  
- **メトリクスAPI**: http://localhost:8003
- **ランブックAPI**: http://localhost:8004

## 📊 データ構成

### K8sデータ (`data/k8s_data/`)
- `deployments.json` - デプロイメントのステータスと設定
- `pods.json` - Podの状態とリソース使用量
- `events.json` - クラスターイベントと警告

### ログデータ (`data/logs_data/`)
- `application_logs.json` - アプリケーションログエントリ
- `error_logs.json` - エラー固有のログエントリ

### メトリクスデータ (`data/metrics_data/`)
- `performance_metrics.json` - レスポンス時間、スループット
- `resource_metrics.json` - CPU、メモリ、ディスク使用量

### ランブックデータ (`data/runbooks_data/`)
- `incident_playbooks.json` - インシデント対応手順
- `troubleshooting_guides.json` - ステップバイステップガイド

## 🔧 サーバー実装

### シンプルHTTPサーバー（デフォルト）
ファイルから直接JSONデータを提供する基本的なPython `http.server`実装。

### FastAPIサーバー（高度）
以下の機能を持つ完全機能のFastAPIサーバー：
- OpenAPIドキュメント
- リクエスト検証
- レスポンススキーマ
- ヘルスエンドポイント

## 📋 OpenAPI仕様

すべてのAPIのためのOpenAPI 3.0仕様の完全版：
- エンドポイント定義
- リクエスト/レスポンススキーマ
- 認証要件
- サンプルデータ

## 🛑 サービス停止

```bash
# シンプルな方法
./scripts/stop_demo_backend.sh

# 高度な方法  
cd servers
python stop_servers.py
```

## 🧪 テスト

個別APIのテスト：
```bash
# K8s APIテスト
curl http://localhost:8001/health

# 特定エンドポイントでのテスト
curl http://localhost:8001/api/v1/namespaces/production/pods
curl http://localhost:8002/api/v1/logs/search?query=error
```

## ⚙️ 設定

バックエンドは以下を含む現実的なデータシナリオを使用します：
- 失敗したデータベースPod
- メモリ不足の警告
- パフォーマンス劣化パターン
- 一般的なトラブルシューティング手順

これにより、SREエージェントシステムの包括的なテスト環境を提供します。