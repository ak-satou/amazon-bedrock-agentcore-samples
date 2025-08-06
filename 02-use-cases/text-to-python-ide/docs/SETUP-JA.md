# セットアップガイド

## 前提条件

AgentCore Code Interpreterをセットアップする前に、以下があることを確認してください：

- **Python 3.8+** pip付き
- **Node.js 16+** npm付き
- **Bedrockアクセス付きAWSアカウント**
- **AWS CLI**設定済みまたは認証情報利用可能

## クイックセットアップ

### 1. 初期セットアップ

```bash
# プロジェクトディレクトリにナビゲート
cd /path/to/strands-agents/agent-core/code-interpreter

# 自動セットアップスクリプトを実行
./setup.sh
```

セットアップスクリプトが実行すること：
- Python仮想環境を作成
- バックエンド依存関係をインストール
- フロントエンド依存関係をインストール
- 設定ファイルを作成

### 2. AWS設定

以下の方法のいずれかを選択：

#### オプションA: AWSプロファイル（推奨）
```bash
# プロファイルでAWS CLIを設定
aws configure --profile your_profile_name

# .envファイルを更新
echo "AWS_PROFILE=your_profile_name" >> .env
echo "AWS_REGION=us-east-1" >> .env
```

#### オプションB: アクセスキー
```bash
# 認証情報で.envファイルを更新
echo "AWS_ACCESS_KEY_ID=your_access_key" >> .env
echo "AWS_SECRET_ACCESS_KEY=your_secret_key" >> .env
echo "AWS_REGION=us-east-1" >> .env
```

### 3. AWS権限

以下のポリシーをAWSユーザー/ロールにアタッチ：

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

または管理ポリシーを使用：`BedrockAgentCoreFullAccess`

### 4. 検証

```bash
# セットアップを検証
python tests/verify_setup.py

# 包括的テストを実行
python tests/run_all_tests.py
```

### 5. アプリケーション開始

```bash
# バックエンドとフロントエンドの両方を開始
./start.sh

# アプリケーションにアクセス
# フロントエンド: http://localhost:3000
# バックエンドAPI: http://localhost:8000
```

## 手動セットアップ

自動セットアップが失敗した場合、以下の手動手順に従います：

### バックエンドセットアップ

```bash
# 仮想環境を作成
python -m venv venv

# 仮想環境をアクティベート
source venv/bin/activate  # Linux/Mac
# または
venv\Scripts\activate     # Windows

# 依存関係をインストール
pip install -r backend/requirements.txt
```

### フロントエンドセットアップ

```bash
# フロントエンドディレクトリにナビゲート
cd frontend

# 依存関係をインストール
npm install

# プロジェクトルートに戻る
cd ..
```

### 設定

```bash
# テンプレートから.envファイルを作成
cp .env.example .env

# 設定で.envファイルを編集
nano .env
```

## 環境変数

| 変数 | 説明 | 必須 | デフォルト |
|----------|-------------|----------|---------|
| `AWS_PROFILE` | AWSプロファイル名 | はい* | - |
| `AWS_ACCESS_KEY_ID` | AWSアクセスキー | はい* | - |
| `AWS_SECRET_ACCESS_KEY` | AWSシークレットキー | はい* | - |
| `AWS_REGION` | AWSリージョン | いいえ | `us-east-1` |
| `BACKEND_HOST` | バックエンドホスト | いいえ | `0.0.0.0` |
| `BACKEND_PORT` | バックエンドポート | いいえ | `8000` |
| `REACT_APP_API_URL` | フロントエンドAPI URL | いいえ | `http://localhost:8000` |

*AWS_PROFILEまたはAWS_ACCESS_KEY_ID + AWS_SECRET_ACCESS_KEYのいずれかが必要

## トラブルシューティング

### 一般的な問題

#### 仮想環境の問題
```bash
# venv作成が失敗した場合
python3 -m venv venv

# Mac/Linuxでアクティベーションが失敗した場合
chmod +x venv/bin/activate
source venv/bin/activate
```

#### 依存関係インストールの問題
```bash
# 最初にpipを更新
pip install --upgrade pip

# 詳細出力でインストール
pip install -r backend/requirements.txt -v

# フロントエンドの問題
cd frontend
npm cache clean --force
npm install
```

#### AWS設定の問題
```bash
# AWS認証情報をテスト
aws sts get-caller-identity

# Bedrockアクセスをテスト
aws bedrock list-foundation-models --region us-east-1

# AgentCoreアクセスをテスト（BedrockAgentCoreFullAccess必要）
python -c "from bedrock_agentcore.tools.code_interpreter_client import code_session; print('AgentCore accessible')"
```

#### ポート競合
```bash
# ポート3000と8000のプロセスを終了
lsof -ti:3000 | xargs kill -9
lsof -ti:8000 | xargs kill -9
```

### ヘルプを得る

1. **ログをチェック**: `backend.log`と`frontend.log`を確認
2. **診断を実行**: `python tests/verify_setup.py`
3. **コンポーネントをテスト**: `python tests/run_all_tests.py`
4. **AWSを確認**: `aws bedrock list-foundation-models`

## 開発セットアップ

開発作業用：

### バックエンド開発
```bash
# 仮想環境をアクティベート
source venv/bin/activate

# 自動リロード付きバックエンドを開始
uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000
```

### フロントエンド開発
```bash
# ホットリロード付きフロントエンドを開始
cd frontend
npm start
```

### テスト
```bash
# すべてのテストを実行
python tests/run_all_tests.py

# 特定のテストを実行
python -c "from tests.run_all_tests import TestRunner; runner = TestRunner(); runner.test_code_generation_api()"
```

## 次のステップ

成功したセットアップ後：

1. **アプリケーションを開始**: `./start.sh`
2. **ブラウザを開く**: `http://localhost:3000`にナビゲート
3. **コードを生成**: 「フィボナッチ数を計算する関数を作成」を試す
4. **コードを実行**: 「コードを実行」をクリックしてAgentCoreサンドボックスで実行
5. **機能を探索**: ファイルアップロード、インタラクティブコード、セッション履歴を試す

アプリケーションは使用準備完了です！🚀