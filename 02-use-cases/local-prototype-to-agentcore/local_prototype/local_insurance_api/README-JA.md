# 保険API

現実的なサンプル データを使用して自動車保険APIをシミュレートするFastAPIベースのアプリケーション。

## 概要

このAPIは以下のエンドポイントを提供します：
- 顧客情報の取得
- 車両データと安全性評価へのアクセス
- さまざまな要因に基づく保険料の計算
- リスク評価の処理
- 利用可能な保険商品の表示
- 保険ポリシーの管理とクエリ

## セットアップ

### 前提条件

- Python 3.10+
- pip

### インストール

1. 仮想環境を作成：
   ```bash
   python -m venv venv
   ```

2. 仮想環境をアクティベート：
   - macOS/Linuxの場合：
     ```bash
     source venv/bin/activate
     ```
   - Windowsの場合：
     ```bash
     venv\Scripts\activate
     ```

3. 依存関係をインストール：
   ```bash
   pip install -r requirements.txt
   ```

## 使用方法

### APIサーバーを起動

```bash
python -m uvicorn server:app --port 8001
```

APIは`http://localhost:8001`で利用可能になります。

### APIエンドポイント

| エンドポイント | メソッド | 説明 | サンプル リクエスト ボディ |
|----------|--------|-------------|---------------------|
| `/` | GET | API情報を含むルート エンドポイント | N/A |
| `/health` | GET | ヘルスチェック | N/A |
| `/customer_info` | POST | 顧客情報を取得 | `{"customer_id": "cust-001"}` |
| `/customer_credit` | POST | 顧客信用情報を取得 | `{"customer_id": "cust-001"}` |
| `/vehicle_info` | POST | 車両情報を取得 | `{"make": "Toyota", "model": "Camry", "year": 2022}` |
| `/risk_factors` | POST | リスク評価を取得 | `{"customer_id": "cust-001", "vehicle_info": {"make": "Toyota", "model": "Camry", "year": 2022}}` |
| `/insurance_products` | POST | フィルタリング、ソート、フォーマット オプション付きで利用可能な保険商品を取得 | `{}`（下記の[保険商品オプション](#保険商品オプション)を参照）|
| `/vehicle_safety` | POST | 車両安全情報を取得 | `{"make": "Toyota", "model": "Camry"}` |
| `/policies` | GET | すべてのポリシーを取得 | N/A |
| `/policies` | POST | さまざまな条件でポリシーをフィルタリング | `{"status": "active"}`（下記の[ポリシー フィルタリング オプション](#ポリシー-フィルタリング-オプション)を参照）|
| `/policies/{policy_id}` | GET | IDで特定のポリシーを取得 | N/A |
| `/customer/{customer_id}/policies` | GET | 特定の顧客のすべてのポリシーを取得 | N/A |
| `/test` | GET | サンプル データを含むテスト エンドポイント | N/A |

## サンプル curlコマンド

### ルート エンドポイント
```bash
curl http://localhost:8001/
```

### ヘルスチェック
```bash
curl http://localhost:8001/health
```

### 顧客情報を取得
```bash
curl -X POST http://localhost:8001/customer_info \
  -H "Content-Type: application/json" \
  -d '{"customer_id": "cust-001"}'
```

### 車両情報を取得
```bash
curl -X POST http://localhost:8001/vehicle_info \
  -H "Content-Type: application/json" \
  -d '{"make": "Toyota", "model": "Camry", "year": 2022}'
```

### リスク要因を取得
```bash
curl -X POST http://localhost:8001/risk_factors \
  -H "Content-Type: application/json" \
  -d '{"customer_id": "cust-001", "vehicle_info": {"make": "Toyota", "model": "Camry", "year": 2022}}'
```

### 保険商品を取得
```bash
curl -X POST http://localhost:8001/insurance_products \
  -H "Content-Type: application/json" \
  -d '{}'
```

### すべてのポリシーを取得
```bash
curl http://localhost:8001/policies
```

### 特定のポリシーを取得
```bash
curl http://localhost:8001/policies/policy-001
```

### 顧客のポリシーを取得
```bash
curl http://localhost:8001/customer/cust-001/policies
```

## ポリシー フィルタリング オプション

`/policies` POSTエンドポイントはさまざまなフィルタリング オプションをサポートします：

| パラメーター | タイプ | 説明 | 例 |
|-----------|------|-------------|--------|
| `policy_id` | string | 特定のポリシーIDでフィルタリング | `{"policy_id": "policy-001"}` |
| `customer_id` | string | 顧客IDでフィルタリング | `{"customer_id": "cust-002"}` |
| `status` | string | ステータスでフィルタリング（active、expiredなど） | `{"status": "active"}` |
| `include_vehicles` | boolean | レスポンスに車両詳細を含める（デフォルト：true） | `{"include_vehicles": false}` |

### ポリシー フィルタークエリの例

```bash
curl -X POST http://localhost:8001/policies \
  -H "Content-Type: application/json" \
  -d '{
    "status": "active",
    "customer_id": "cust-001"
  }'
```

## 保険商品オプション

`/insurance_products`エンドポイントは、さまざまなフィルタリング、ソート、フォーマット オプションをサポートします：

### フィルタリング オプション

| パラメーター | タイプ | 説明 | 例 |
|-----------|------|-------------|--------|
| `product_id` | string または array | 特定の商品IDでフィルタリング | `{"product_id": "premium-auto"}` または `{"product_id": ["basic-auto", "standard-auto"]}` |
| `price_range` | object | 価格範囲でフィルタリング | `{"price_range": {"min": 500, "max": 1200}}` |
| `coverage_includes` | array | 特定の補償を含む商品をフィルタリング | `{"coverage_includes": ["collision", "comprehensive"]}` |
| `discount_includes` | array | 特定の割引を提供する商品をフィルタリング | `{"discount_includes": ["loyalty", "good-student"]}` |

### ソート オプション

| パラメーター | タイプ | 説明 | 例 |
|-----------|------|-------------|--------|
| `sort_by` | string | "price"、"name"、または"rating"でソート | `{"sort_by": "price"}` |
| `sort_order` | string | "asc"または"desc"の順序でソート | `{"sort_order": "desc"}` |

### レスポンス フォーマット オプション

| パラメーター | タイプ | 説明 | 例 |
|-----------|------|-------------|--------|
| `include_details` | boolean | 完全な商品詳細を含める（デフォルト：true） | `{"include_details": false}` |
| `format` | string | レスポンス形式 - "full"または"summary"（デフォルト："full"） | `{"format": "summary"}` |

### 複雑なクエリの例

```bash
curl -X POST http://localhost:8001/insurance_products \
  -H "Content-Type: application/json" \
  -d '{
    "price_range": {"min": 500, "max": 1200},
    "coverage_includes": ["liability"],
    "sort_by": "price",
    "sort_order": "asc",
    "format": "full"
  }'
```

## データ

APIは`data/`ディレクトリにあるサンプル データを使用します：

- `customers.json`：個人情報と運転履歴を含む顧客プロファイル
- `vehicles.json`：車両仕様と評価
- `credit_reports.json`：顧客信用情報
- `products.json`：保険商品詳細
- `pricing_rules.json`：保険料計算のルール
- `policies.json`：サンプル保険ポリシー

## プロジェクト構造
```
insurance_api/
├── app.py                 # FastAPIアプリケーション初期化
├── server.py              # アプリケーション実行のエントリーポイント
├── data_loader.py         # JSONファイルからデータを読み込み管理するユーティリティ
├── data/                  # サンプル データ ファイルを含むディレクトリ
├── requirements.txt       # プロジェクト依存関係
├── routes/                # ドメイン別に整理されたエンドポイント ハンドラー
│   ├── __init__.py        # パッケージ初期化
│   ├── customer.py        # 顧客関連エンドポイント
│   ├── general.py         # ルート、ヘルス、テスト エンドポイント
│   ├── insurance.py       # 保険関連エンドポイント
│   ├── policy.py          # ポリシー関連エンドポイント
│   └── vehicle.py         # 車両関連エンドポイント
└── services/              # ドメイン別に整理されたビジネス ロジック
    ├── __init__.py        # パッケージ初期化
    ├── data_service.py    # データ アクセス機能
    ├── policy_service.py  # ポリシー管理機能
    ├── product_service.py # 保険商品ビジネス ロジック
    └── utils.py           # ユーティリティ機能
```

### 主要コンポーネント

- **app.py**：ミドルウェアとルーター登録を含む、FastAPIアプリケーションの作成と設定。
- **server.py**：アプリケーションを実行するシンプルなエントリーポイント。
- **routes/**：ドメイン（顧客、車両、保険、ポリシー）別に整理されたエンドポイント ハンドラーを含む。
- **services/**：ビジネス ロジックとデータ アクセス機能を含む。
- **data_loader.py**：JSONファイルからサンプル データを読み込み、アクセスを提供。