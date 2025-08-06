# 自動車保険API MCPサーバー

このディレクトリには、自動車保険APIへのアクセスを提供するModel Context Protocol（MCP）サーバーが含まれています。LLMやその他のアプリケーションが標準化されたプロトコルを通じて顧客情報、車両詳細、保険見積もりを取得できるようにします。

## 概要

MCPサーバーは、LLM（Claudeなど）と我々の自動車保険APIの間の橋渡しとして機能します。以下の用途に使用できるいくつかのツールを公開します：

1. 顧客情報の取得
2. 車両情報の取得
3. 保険見積もりの取得
4. 車両安全情報の取得
5. ポリシー情報の取得（すべてのポリシー、特定のポリシー、または顧客ポリシー）

## 前提条件

- Python 3.10以上
- Node.jsとnpm（MCP Inspector用）
- uvパッケージ マネージャー（推奨）

## セットアップ手順

### 1. 依存関係のインストール

```bash
# まだリポジトリをクローンしていない場合はクローン
cd local_prototype/native_mcp_server

# オプション1：セットアップ スクリプトを使用
chmod +x setup.sh
./setup.sh

# オプション2：手動セットアップ
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 2. 保険APIの開始

MCPサーバーは保険APIに接続するため、ポート8001で保険APIが動作している必要があります：

```bash
cd ../insurance_api
python -m uvicorn server:app --port 8001
```

### 3. MCPサーバーの開始

新しいターミナル ウィンドウで：

```bash
cd local_prototype/native_mcp_server
source venv/bin/activate

# uvで実行（推奨）
uv run server.py

# または通常のPythonで実行
python server.py
```

以下のような出力が表示されるはずです：
```
🚀 Starting LocalMCP v1.0.0 MCP server...
📂 Projects directory: /Users/username/local_mcp_projects
🔌 Insurance API URL: http://localhost:8001
✅ Server is running. Press CTRL+C to stop.
Starting with streamable-http transport...
INFO: Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```

### 4. MCP Inspectorとの接続

MCP Inspectorでサーバーをテストするには：

1. MCP Inspectorをインストールして開始：
```bash
npx @modelcontextprotocol/inspector
```

2. これにより、ブラウザでMCP Inspectorが自動的に開かれます

3. MCP サーバーURLを追加：
   - 「Add MCP URL」をクリック
   - 入力：`http://localhost:8000/mcp`
   - 「Connect」をクリック

4. 利用可能なツールが表示され、それらと相互作用できるようになります：
   - `get_customer_info`
   - `get_vehicle_info`
   - `get_insurance_quote`
   - `get_vehicle_safety`
   - `get_all_policies`
   - `get_policy_by_id`
   - `get_customer_policies`

## ツールの使用

### 例：顧客情報を取得

顧客に関する情報を取得するには：

1. MCP Inspectorで`get_customer_info`ツールを選択
2. パラメーターに有効な顧客ID（例：`cust-001`）を入力
3. 「Run」をクリック

### 例：ポリシー データの操作

MCPサーバーは、保険ポリシーを扱うための3つのツールを提供します：

#### すべてのポリシーを取得

利用可能なすべてのポリシーを取得するには：

1. MCP Inspectorで`get_all_policies`ツールを選択
2. パラメーターは不要
3. 「Run」をクリック

#### IDでポリシーを取得

特定のポリシーの詳細を取得するには：

1. MCP Inspectorで`get_policy_by_id`ツールを選択
2. パラメーターに有効なポリシーID（例：`policy-001`）を入力
3. 「Run」をクリック

#### 顧客のポリシーを取得

特定の顧客のすべてのポリシーを取得するには：

1. MCP Inspectorで`get_customer_policies`ツールを選択
2. パラメーターに有効な顧客ID（例：`cust-001`）を入力
3. 「Run」をクリック

![get_customer_infoツール呼び出しの例](local_mcp_call.png)

### 有効なテスト データ

テストには以下の値を使用してください：

- 顧客ID：`cust-001`、`cust-002`、`cust-003`
- 車両メーカー：`Toyota`、`Honda`、`Ford`
- 車両モデル：`Camry`、`Civic`、`F-150`
- 車両年式：2010年〜2023年の任意の年
- ポリシーID：`policy-001`、`policy-002`、`policy-003`

## トラブルシューティング

- **接続拒否エラー**：保険APIとMCPサーバーの両方が動作していることを確認してください
- **顧客が見つからないエラー**：有効な顧客IDを使用していることを確認してください
- **ポートがすでに使用中**：ポートを使用しているプロセスを終了するか、別のポートを指定してください

## アーキテクチャ

MCPサーバーは以下で構成されています：

1. `server.py` - MCPサーバーを初期化するメイン エントリーポイント
2. `tools/insurance_tools.py` - 保険APIと相互作用するツール
3. `config.py` - API URLとサーバー設定を含む設定

## 開発者向け

さらにツールを追加するには：

1. `tools/insurance_tools.py`に関数を追加
2. `register_insurance_tools`関数で登録
3. サーバーを再起動して利用可能にする

エラー処理を変更するには：

- ツール実装のtry/exceptブロックを編集してエラー メッセージをカスタマイズ