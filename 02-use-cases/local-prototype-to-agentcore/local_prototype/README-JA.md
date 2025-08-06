# 自動車保険プラットフォーム - ローカル プロトタイプ

このディレクトリには、3つの主要コンポーネントを持つ自動車保険プラットフォームの完全なローカル プロトタイプが含まれています：

1. **保険API**：保険関連のデータと機能を提供するFastAPIベースのバックエンド サービス
2. **ローカルMCPサーバー**：保険ツールをLLMに公開するModel Context Protocolサーバー
3. **Strands保険エージェント**：保険ツールと相互作用するためにClaude 3.7を使用するインタラクティブ エージェント

## システム概要

ローカル プロトタイプは、保険アプリケーション用の完全なエージェントベース アーキテクチャを実証します：

- **保険API**（ポート8001）：顧客、車両、ポリシー データを持つコア バックエンド サービス
- **MCPサーバー**（ポート8000）：保険APIエンドポイントをMCPツールとして公開するミドルウェア
- **Strandsエージェント**：Claude 3.7 Sonnetを使用して自然言語インターフェースを提供するフロントエンド

このアーキテクチャは、開発者が標準化されたプロトコルを通じて構造化されたデータ サービスと相互作用するLLMを搭載したアプリケーションを構築する方法を示しています。

## コンポーネント

### 1. 保険API

現実的なサンプル データを使用して自動車保険バックエンドをシミュレートするFastAPIアプリケーション。

**主な機能：**
- 顧客情報エンドポイント
- 車両データと安全性評価
- リスク評価計算
- 保険商品カタログと価格設定
- ポリシー管理（表示、フィルター、検索）

**サンプル データ：**
- 顧客プロファイル
- 車両仕様
- 信用レポート
- 保険商品
- 保険ポリシー

### 2. ローカルMCPサーバー

標準化されたツールを通じて保険APIへのアクセスを提供するModel Context Protocol（MCP）サーバー。

**主要ツール：**
- `get_customer_info`：顧客詳細を取得
- `get_vehicle_info`：車両仕様を取得
- `get_insurance_quote`：保険見積もりを生成
- `get_vehicle_safety`：安全性評価にアクセス
- `get_all_policies`：すべてのポリシーを表示
- `get_policy_by_id`：特定のポリシー詳細を取得
- `get_customer_policies`：顧客IDでポリシーを検索

### 3. Strands保険エージェント

MCPサーバーに接続するAnthropic's Strandsフレームワークで構築されたインタラクティブ エージェント。

**主な機能：**
- 保険ツールとの自然言語相互作用
- 会話履歴とコンテキスト持続性
- Claude 3.7 Sonnet用のAWS Bedrock統合
- 包括的なエラー処理とレスポンス フォーマット

### 4. Streamlitダッシュボード

保険プラットフォーム システム全体の可視化とテスト インターフェース。

**主な機能：**
- システム ステータス監視
- APIエンドポイント テスト
- MCPツール実行
- エージェント チャット シミュレーション
- システム アーキテクチャ可視化
- インタラクティブ データ探索

## セットアップ手順

### 前提条件

- Python 3.10+
- Claude 3.7 SonnetへのBedrockアクセス権を持つAWSアカウント
- Node.jsとnpm（MCP Inspector用）
- uvパッケージ マネージャー（推奨）

### 1. 保険APIセットアップ

```bash
# 保険APIディレクトリに移動
cd local_insurance_api

# 仮想環境を作成してアクティベート
python -m venv venv
source venv/bin/activate  # Windowsの場合：venv\Scripts\activate

# 依存関係をインストール
pip install -r requirements.txt

# APIサーバーを開始
python -m uvicorn server:app --port 8001
```

保険APIは`http://localhost:8001`で利用可能になります。

### 2. MCPサーバー セットアップ

```bash
# MCPサーバー ディレクトリに移動
cd local_mcp_server

# 仮想環境を作成してアクティベート
python -m venv venv
source venv/bin/activate  # Windowsの場合：venv\Scripts\activate

# 依存関係をインストール
pip install -r requirements.txt

# HTTPトランスポートでMCPサーバーを開始
python server.py --http
```

MCPサーバーは`http://localhost:8000/mcp`で利用可能になります。

### 3. Strandsエージェント セットアップ

```bash
# Strandsエージェント ディレクトリに移動
cd local_strands_insurance_agent

# 仮想環境を作成してアクティベート
python -m venv venv
source venv/bin/activate  # Windowsの場合：venv\Scripts\activate

# 依存関係をインストール
pip install -r requirements.txt

# AWS認証情報を設定
[リンク](https://strandsagents.com/latest/documentation/docs/user-guide/quickstart/#configuring-credentials)を使用して認証情報を設定

# インタラクティブ エージェントを開始
python interactive_insurance_agent.py
```

## MCP Inspectorでのテスト

MCP Inspectorツールを使用してMCPサーバーを直接テストできます：

```bash
# MCP Inspectorをインストールして開始
npx @modelcontextprotocol/inspector
```

これによりブラウザでMCP Inspectorが開きます。`http://localhost:8000/mcp`に接続して、利用可能なツールを探索およびテストします。

## 使用例

### 保険APIエンドポイント

```bash
# すべてのポリシーを取得
curl http://localhost:8001/policies

# 特定のポリシーを取得
curl http://localhost:8001/policies/policy-001

# 顧客のポリシーを取得
curl http://localhost:8001/customer/cust-001/policies

# 保険商品を取得
curl -X POST http://localhost:8001/insurance_products \
  -H "Content-Type: application/json" \
  -d '{}'
```

### Strandsエージェント クエリ

```
あなた：顧客cust-001についてどのような情報を持っていますか？
エージェント：[John Smithについての詳細情報を返す]

あなた：彼はどのような車を持っていますか？
エージェント：[コンテキストを使用して車両情報を提供]

あなた：2023年のToyota RAV4の見積もりをもらえますか？
エージェント：[顧客プロファイルに基づいて見積もりを生成]
```

## 有効なテスト データ

テストには以下の値を使用してください：

- 顧客ID：`cust-001`、`cust-002`、`cust-003`
- 車両メーカー：`Toyota`、`Honda`、`Ford`
- 車両モデル：`Camry`、`Civic`、`F-150`
- 車両年式：2010年〜2023年の任意の年
- ポリシーID：`policy-001`、`policy-002`、`policy-003`

## ディレクトリ構造

```
local_prototype/
├── local_insurance_api/            # コア バックエンド サービス
│   ├── data/                 # サンプル データ ファイル
│   ├── routes/               # APIエンドポイント
│   ├── services/             # ビジネス ロジック
│   ├── app.py                # アプリケーション初期化
│   └── server.py             # エントリー ポイント
├── local_mcp_server/        # MCPサーバー実装
│   ├── tools/                # MCPツール定義
│   ├── config.py             # サーバー設定
│   └── server.py             # エントリー ポイント
└── local_strands_insurance_agent/  # インタラクティブ エージェント
    └── interactive_insurance_agent.py  # エージェント実装
```

## アーキテクチャ図

```
┌───────────────────┐      ┌───────────────────┐      ┌───────────────────┐
│                   │      │                   │      │                   │
│  Strandsエージェント│◄────►│  MCPサーバー      │◄────►│  保険API         │
│  (Claude 3.7)     │      │  (Model Context   │      │  (FastAPI)        │
│                   │      │   Protocol)       │      │                   │
└───────────────────┘      └───────────────────┘      └───────────────────┘
       ▲                                                      ▲
       │                                                      │
       │                                                      │
       ▼                                                      ▼
┌───────────────────┐                               ┌───────────────────┐
│                   │                               │                   │
│  ユーザー         │                               │  サンプル データ  │
│  (コンソール)     │                               │  (JSONファイル)   │
│                   │                               │                   │
└───────────────────┘                               └───────────────────┘
```

## トラブルシューティング

### よくある問題

1. **ポートがすでに使用中**
   - エラー：「Address already in use」
   - 解決策：ポートを使用しているプロセスを終了するか、別のポートを指定

2. **接続拒否**
   - エラー：サービスに接続する際の「Connection refused」
   - 解決策：3つのコンポーネントすべてが実行されていることを確認

3. **AWS認証の問題**
   - エラー：「Could not connect to the endpoint URL」
   - 解決策：AWS認証情報とリージョンを確認

## 次のステップ

本番デプロイメントについては、AgentCoreを使用してこのアーキテクチャをAWSにデプロイする方法を実証する`agentcore_app`ディレクトリを参照してください。

## 📝 ライセンス

このプロジェクトはApache License 2.0の下でライセンスされています - 詳細については[LICENSE](../../../LICENSE)ファイルを参照してください。

## 注意

このローカル プロトタイプは、開発者が以下を使用してエージェント アプリケーションを構築する方法を実証します：
- FastAPIバックエンド サービス
- Model Context Protocol（MCP）サーバー
- Anthropic's Strandsエージェント フレームワーク
- Claude 3.7 Sonnet LLM