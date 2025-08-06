# プロジェクト構造

## 📁 整理されたファイル構造

```
strands-agents/agent-core/code-interpreter/
├── 📂 backend/                    # FastAPI バックエンド
│   ├── main.py                   # メインアプリケーションサーバー
│   └── requirements.txt          # Python依存関係
│
├── 📂 frontend/                   # React フロントエンド
│   ├── 📂 src/
│   │   ├── App.js               # メインReactアプリケーション
│   │   └── 📂 components/       # Reactコンポーネント
│   │       ├── CodeEditor.js    # Monacoコードエディター
│   │       ├── CodeDisplay.js   # コード出力表示
│   │       ├── ExecutionResults.js # 実行結果
│   │       └── SessionHistory.js   # セッション管理
│   ├── 📂 public/               # 静的アセット
│   ├── package.json             # Node.js依存関係
│   └── package-lock.json        # 依存関係ロックファイル
│
├── 📂 tests/                      # テストスイート
│   ├── run_all_tests.py         # 包括的テストランナー
│   ├── verify_setup.py          # セットアップ検証
│   ├── test_model_fallback.py   # モデルフォールバックテスト
│   ├── test_execution_fix.py    # 実行結果テスト
│   └── debug_code_generation.py # コード生成デバッグ
│
├── 📂 docs/                       # ドキュメント
│   ├── ARCHITECTURE.md          # システムアーキテクチャ
│   ├── SETUP.md                 # セットアップ手順
│   ├── OVERVIEW.md              # プロジェクト概要
│   └── [履歴ドキュメント]        # 以前のイテレーションドキュメント
│
├── 📂 venv/                       # Python仮想環境
│   ├── bin/                     # 実行ファイル
│   ├── lib/                     # Pythonパッケージ
│   └── ...
│
├── 🔧 設定ファイル
│   ├── .env                     # 環境変数
│   ├── .env.example             # 環境変数テンプレート
│   └── .gitignore               # Git無視ルール
│
├── 🚀 スクリプト
│   ├── setup.sh                 # 自動セットアップ
│   ├── start.sh                 # アプリケーション起動
│   └── cleanup.sh               # クリーンアップスクリプト
│
├── 📋 ドキュメント
│   ├── README.md                # メインドキュメント
│   └── PROJECT_STRUCTURE.md     # このファイル
│
└── 📊 ログ（生成される）
    ├── backend.log              # バックエンドログ
    ├── frontend.log             # フロントエンドログ
    └── *.pid                    # プロセスIDファイル
```

## 🎯 主要ディレクトリ

### `/backend/`
**目的**: Strands-Agents統合を備えたFastAPIサーバー
- **main.py**: REST APIとWebSocketハンドラーを持つコアアプリケーション
- **requirements.txt**: strands-agentsとbedrock-agentcoreを含むPython依存関係

### `/frontend/`
**目的**: AWS Cloudscape UIを使用したReactアプリケーション
- **src/App.js**: タブ付きインターフェースのメインアプリケーション
- **src/components/**: 再利用可能なReactコンポーネント
- **package.json**: Node.js依存関係とスクリプト

### `/tests/`
**目的**: 包括的なテストと検証
- **run_all_tests.py**: すべてのコンポーネントの完全テストスイート
- **verify_setup.py**: 迅速なセットアップ検証
- **test_*.py**: 特定コンポーネントのテスト
- **debug_*.py**: デバッグユーティリティ

### `/docs/`
**目的**: プロジェクトドキュメント
- **ARCHITECTURE.md**: システム設計とコンポーネント詳細
- **SETUP.md**: 詳細なセットアップ手順
- **OVERVIEW.md**: プロジェクト概要と使用例

## 🔧 設定

### 環境変数（`.env`）
```bash
# AWS設定
AWS_PROFILE=default
AWS_REGION=us-east-1

# アプリケーション設定
BACKEND_HOST=0.0.0.0
BACKEND_PORT=8000
REACT_APP_API_URL=http://localhost:8000
```

### スクリプト
- **setup.sh**: 自動環境セットアップ
- **start.sh**: バックエンドとフロントエンドの起動
- **cleanup.sh**: プロセスと一時ファイルのクリーンアップ

## 🧪 テスト構造

### テストカテゴリ
1. **環境テスト**: 依存関係、AWS設定、ファイル構造
2. **モデルテスト**: AIモデル初期化とフォールバック
3. **エージェントテスト**: Strands-Agents統合
4. **APIテスト**: RESTエンドポイントとWebSocketハンドラー
5. **統合テスト**: エンドツーエンド機能

### テスト実行
```bash
# 迅速な検証
python tests/verify_setup.py

# 包括的なテスト
python tests/run_all_tests.py

# 特定コンポーネントテスト
python tests/test_model_fallback.py
```

## 📊 ログとモニタリング

### ログファイル
- **backend.log**: FastAPIサーバーログ、エージェント応答、エラー
- **frontend.log**: React開発サーバーログ
- **.pid**: クリーンアップ用プロセスIDファイル

### ヘルスモニタリング
- **ヘルスエンドポイント**: `GET /health` - システムステータス
- **エージェントステータス**: `GET /api/agents/status` - エージェント情報
- **モデル情報**: 現在のモデルとフォールバックステータス

## 🚀 クイックコマンド

### セットアップと開始
```bash
./setup.sh          # 初期セットアップ
./start.sh           # アプリケーション開始
./cleanup.sh         # クリーンアップとリセット
```

### テスト
```bash
python tests/verify_setup.py      # セットアップ検証
python tests/run_all_tests.py     # すべてのテスト実行
```

### 開発
```bash
# バックエンドのみ
source venv/bin/activate
python backend/main.py

# フロントエンドのみ
cd frontend && npm start
```

## 📋 ファイル構成のメリット

### ✅ **保守性の向上**
- 明確な関心の分離
- 関連ファイルの論理的グループ化
- 簡単なナビゲーションと発見

### ✅ **より良いテスト**
- 集中化されたテストスイート
- 包括的なカバレッジ
- 簡単なテスト実行

### ✅ **強化されたドキュメント**
- 整理されたドキュメント構造
- 明確なセットアップ手順
- アーキテクチャ概要

### ✅ **合理化された開発**
- 自動化されたセットアップとクリーンアップ
- 一貫したプロジェクト構造
- 新しい開発者の簡単なオンボーディング

この整理された構造は、AgentCore Code Interpreterの開発、テスト、デプロイメントのための堅実な基盤を提供します。