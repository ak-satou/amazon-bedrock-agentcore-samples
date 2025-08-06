# テキストからPython IDE

**Strands-Agents**フレームワークと**AWS Bedrock AgentCore**を組み合わせたインテリジェントなPythonコード開発のための強力なAI駆動コード生成・実行プラットフォーム。

## 🎯 概要

テキストからPython IDEは、ユーザーが以下を可能にするフルスタックアプリケーションです：

- 高度なAIモデルを使用して自然言語の説明からPythonコードを**生成**
- AWS管理のサンドボックス環境でコードを安全に**実行** 
- 生成されたコードでのデータ解析と処理のための**CSVファイルのアップロード**
- モダンなWebインターフェースを通じて**結果と相互作用**
- 永続的な会話履歴を持つ**セッション管理**

### 主要機能

- 🤖 **AI駆動コード生成** Claude Sonnet 3.7とNova Premierを使用
- ⚡ **リアルタイムコード実行** AWS Bedrock AgentCore経由
- 📊 **CSVファイルアップロード＆統合** データ解析ワークフロー用
- ⏱️ **実行タイマー** 長時間実行操作の視覚的フィードバック付き
- 🔄 **インテリジェントモデルフォールバック** 最大信頼性のため
- 🌐 **モダンWebインターフェース** ReactとAWS Cloudscapeで構築
- 📋 **セッション管理** 実行履歴付き
- 🔒 **セキュア実行** 分離されたAWS環境
- 📁 **ファイルアップロードサポート** 既存のPythonファイルとCSVデータ用
- 🚀 **パフォーマンス最適化** キャッシングと接続プーリング付き
- 🧪 **包括的テスト** 自動エンドツーエンド検証付き
- 🎨 **チャートレンダリング** matplotlib/seabornでのデータ可視化用

## 🏗️ アーキテクチャ

![Architecture Diagram](./img/agentcore_aws_architecture_simple.png)

### コンポーネント詳細

#### **フロントエンド (React + AWS Cloudscape)**
- **コードジェネレータータブ**: 自然言語からPythonコードへの変換
- **コードエディタータブ**: シンタックスハイライト付きMonacoベースエディタ
- **実行結果タブ**: エラーハンドリング付きフォーマット済み出力表示
- **セッション履歴タブ**: 実行と会話履歴の表示

#### **バックエンド (FastAPI + Strands-Agents)**
- **コードジェネレーターエージェント**: インテリジェントなコード生成のためのClaude Sonnet 3.7使用
- **コード実行エージェント**: 安全なコード実行のためのAgentCoreとの統合
- **セッション管理**: RESTful APIとWebSocketサポート
- **モデルフォールバック**: AIモデル間の自動フェイルオーバー

#### **AIモデル (AWS Bedrock)**
- **プライマリ**: Claude Sonnet 3.7 (Inference Profile) - `us.anthropic.claude-3-7-sonnet-20250219-v1:0`
- **フォールバック**: Nova Premier (Inference Profile) - `us.amazon.nova-premier-v1:0`
- **セーフティネット**: Claude 3.5 Sonnet - `anthropic.claude-3-5-sonnet-20241022-v2:0`

#### **実行環境 (AgentCore)**
- **サンドボックス化されたPython環境**: AWSでの分離実行
- **リアルタイム結果**: ストリーミング出力とエラーハンドリング
- **セッション永続化**: 実行間での状態維持

## 🚀 クイックスタート

### 前提条件

- **Python 3.8+** pipあり
- **Node.js 16+** npmあり
- **AWSアカウント** Bedrockアクセスあり
- **AWS CLI** 設定済みまたは認証情報利用可能

### 1. セットアップと開始

アプリケーションには自動セットアップが含まれています - 開始スクリプトを実行するだけです：

```bash
# アプリケーションの開始（自動セットアップ含む）
./start.sh

# スクリプトは自動的に以下を行います：
# - 必要に応じてPython仮想環境を作成
# - すべての依存関係をインストール
# - .env設定ファイルを作成
# - バックエンドとフロントエンドの両サーバーを開始
```

### 2. 環境の設定（オプション）

AWS認証情報をカスタマイズする必要がある場合は、`.env`ファイルを編集してください：

```bash
# AWS設定（1つの方法を選択）
AWS_PROFILE=your_profile_name          # 推奨

# アプリケーション設定
BACKEND_HOST=0.0.0.0
BACKEND_PORT=8000
REACT_APP_API_URL=http://localhost:8000
```

### 3. アプリケーションへのアクセス

`./start.sh`を実行後、以下にアクセス：
- **フロントエンド**: http://localhost:3000
- **バックエンドAPI**: http://localhost:8000

### 4. セットアップの検証（オプション）

```bash
# セットアップの検証（オプション - start.shが自動的に実行）
python tests/verify_setup.py

# 包括的テストの実行
python tests/run_all_tests.py
```

## 📋 使用方法

### コード生成
1. **コードジェネレーター**タブに移動
2. **オプション**: データ解析タスク用に「Upload CSV File」ボタンを使用してCSVファイルをアップロード
3. 自然言語の説明を入力（例：「フィボナッチ数を計算する関数を作成」または「アップロードされたCSVデータを分析して可視化を作成」）
4. **Generate Code**をクリック
5. **コードエディタ**タブで生成されたコードを確認

### CSVファイル統合
1. **コードジェネレーター**タブで**Upload CSV File**をクリック
2. CSVファイルを選択（.csv拡張子必須）
3. ファイルがコード生成で使用可能になります
4. プロンプトでデータ解析、ファイル、またはCSVについて言及すると、AIが自動的にアップロードされたデータを組み込みます
5. プロンプトでファイルについて言及しているがCSVをアップロードしていない場合、アップロードするよう促されます

### コード実行
1. **コードエディタ**タブでコードを確認または修正
2. 即座の実行には**Execute Code**をクリック
3. ユーザー入力が必要なコードには**Interactive Execute**をクリック
4. **実行結果**タブで結果を表示

### ファイルアップロード
1. **コードエディタ**タブでファイルアップロードコンポーネントを使用
2. `.py`または`.txt`ファイルを選択
3. ファイル内容がエディタに読み込まれます
4. 必要に応じて実行または修正

## 🛠️ 開発

### プロジェクト構造

```
├── backend/                 # FastAPIバックエンド
│   ├── main.py             # メインアプリケーション
│   └── requirements.txt    # Python依存関係
├── frontend/               # Reactフロントエンド
│   ├── src/
│   │   ├── App.js         # メインアプリケーション
│   │   └── components/    # Reactコンポーネント
│   └── package.json       # Node依存関係
├── tests/                  # テストスクリプト
│   ├── run_all_tests.py   # 包括的テストスイート
│   └── verify_setup.py    # セットアップ検証
├── docs/                   # ドキュメント
├── .env                    # 環境設定
├── setup.sh               # セットアップスクリプト
└── start.sh               # 開始スクリプト
```

### テストの実行

```bash
# セットアップの検証
python tests/verify_setup.py

# 包括的テストの実行
python tests/run_all_tests.py

# 自動エンドツーエンドテストの実行（ユーザー入力不要）
python tests/automated_e2e_test.py

# 特定コンポーネントのテスト
python -c "from tests.run_all_tests import TestRunner; runner = TestRunner(); runner.test_code_generation_api()"
```

### 開発モード

```bash
# バックエンドのみ
source venv/bin/activate
python backend/main.py

# フロントエンドのみ
cd frontend
npm start

# 自動リロード付きウォッチモード
# バックエンド: uvicorn --reloadを使用
# フロントエンド: npm start（ホットリロード含む）
```

## 🔧 設定

### 必要なAWS権限

AWSユーザー/ロールには以下の権限が必要です：

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "bedrock:ListFoundationModels"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "bedrock-agentcore:StartCodeInterpreterSession",
                "bedrock-agentcore:StopCodeInterpreterSession",
                "bedrock-agentcore:InvokeCodeInterpreter"
            ],
            "Resource": "*"
        }
    ]
}
```

または管理ポリシーを使用: `BedrockAgentCoreFullAccess`

### モデル設定

アプリケーションは自動的に最適な利用可能モデルを選択します：

1. **Claude Sonnet 3.7** (Inference Profile) - 主要選択
2. **Nova Premier** (Inference Profile) - 自動フォールバック
3. **Claude 3.5 Sonnet** - セーフティフォールバック

### 環境変数

| 変数 | 説明 | デフォルト |
|----------|-------------|---------|
| `AWS_PROFILE` | AWSプロファイル名 | - |
| `AWS_REGION` | AWSリージョン | `us-east-1` |
| `BACKEND_HOST` | バックエンドホスト | `0.0.0.0` |
| `BACKEND_PORT` | バックエンドポート | `8000` |
| `REACT_APP_API_URL` | フロントエンドAPI URL | `http://localhost:8000` |

#### タイムアウト設定

| 変数 | 説明 | デフォルト | 推奨最大 |
|----------|-------------|---------|-----------------|
| `AWS_READ_TIMEOUT` | AWS Bedrock読み取りタイムアウト（秒） | `600` | `600` |
| `AWS_CONNECT_TIMEOUT` | AWS接続タイムアウト（秒） | `120` | `300` |
| `AWS_MAX_RETRIES` | 最大再試行回数 | `5` | `10` |
| `AGENTCORE_SESSION_TIMEOUT` | AgentCoreセッションタイムアウト（秒） | `1800` | `1800` |
| `REACT_APP_EXECUTION_TIMEOUT_WARNING` | UI警告閾値（秒） | `300` | - |
| `REACT_APP_MAX_EXECUTION_TIME` | UI最大時間表示（秒） | `600` | - |

**注意**: これらのタイムアウト値は、データ解析、機械学習、可視化タスクを含む複雑なコード実行に最適化されています。

## 🧹 クリーンアップ

```bash
# アプリケーションの停止
# start.shを実行しているターミナルでCtrl+Cを押す

# または手動でプロセスを停止
lsof -ti:8000 | xargs kill -9  # バックエンド
lsof -ti:3000 | xargs kill -9  # フロントエンド

# 一時ファイルのクリーンアップ
rm -f backend.log frontend.log *.pid
```

## 🐛 トラブルシューティング

### よくある問題

**バックエンドが開始しない:**
- AWS認証情報を確認: `aws sts get-caller-identity`
- 依存関係を検証: `python tests/verify_setup.py`
- ログを確認: `tail -f backend.log`

**フロントエンドが開始しない:**
- 依存関係をインストール: `cd frontend && npm install`
- Nodeバージョンを確認: `node --version` (16+が必要)
- キャッシュをクリア: `npm start -- --reset-cache`

**コード生成が失敗する:**
- Bedrockアクセスを確認: AWS権限をチェック
- モデル可用性をテスト: `python tests/run_all_tests.py`
- リージョンを確認: モデルがリージョンで利用可能か確認

**コード実行が失敗する:**
- AgentCore権限を確認: `BedrockAgentCoreFullAccess`
- AgentCoreを直接テスト: `tests/`のテストスクリプトを参照
- セッション制限を確認: AgentCoreは同時セッション制限あり

### ヘルプの取得

1. **診断の実行**: `python tests/verify_setup.py`
2. **ログの確認**: `backend.log`と`frontend.log`
3. **コンポーネントのテスト**: `python tests/run_all_tests.py`
4. **AWSセットアップの検証**: `aws bedrock list-foundation-models`

## 📄 ライセンス

このプロジェクトはStrands-Agentsエコシステムの一部です。メインプロジェクトのライセンスを参照してください。

---

**AIでコーディングを始める準備はできましたか？`./start.sh`を実行してhttp://localhost:3000を訪問してください** 🚀