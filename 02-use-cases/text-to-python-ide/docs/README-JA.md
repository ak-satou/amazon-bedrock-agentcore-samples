# AgentCore Code Interpreter

**strands-agents**を使用したコード生成と**Amazon Bedrock AgentCore**を使用したコード実行を特徴とする**ハイブリッドアーキテクチャ**で構築されたPython コード実行環境、AWS Cloudscapeデザインシステムを使用したReactフロントエンド付き。

## アーキテクチャ

このアプリケーションは**正しいAgentCore統合**を持つ**ハイブリッドマルチエージェントアーキテクチャ**を使用します：

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────────────┐
│   React UI      │    │   FastAPI        │    │   ハイブリッドエージェント │
│   (Cloudscape)  │◄──►│   Backend        │◄──►│   システム               │
│                 │    │                  │    │                         │
└─────────────────┘    └──────────────────┘    │  ┌─────────────────┐    │
                                                │  │ Strands Agent   │    │
                                                │  │ Code Generator  │    │
                                                │  │ Claude 3.7      │    │
                                                │  └─────────────────┘    │
                                                │           │             │
                                                │           ▼             │
                                                │  ┌─────────────────┐    │
                                                │  │ Strands Agent   │    │
                                                │  │ + AgentCore     │    │
                                                │  │ CodeInterpreter │    │
                                                │  │ Tool            │    │
                                                │  └─────────────────┘    │
                                                └─────────────────────────┘
```

## モデル階層

アプリケーションは**推論プロファイル**を使用したインテリジェントモデルフォールバックシステムを使用します：

1. **プライマリ**: Claude Sonnet 3.7 (`us.anthropic.claude-3-7-sonnet-20250219-v1:0`)
2. **フォールバック**: Nova Premier (`us.amazon.nova-premier-v1:0`) 
3. **最後の手段**: Claude 3.5 Sonnet (`anthropic.claude-3-5-sonnet-20241022-v2:0`)

システムは自動的にモデルの可用性を検出し、利用可能な最適なオプションを選択します。

1. **コード生成エージェント**（Strands-Agentsフレームワーク）：
   - strands-agents経由で**Claude Sonnet 3.7**（プライマリ）または**Nova Premier**（フォールバック）を使用
   - 自然言語からクリーンで実行可能なPythonコードを生成
   - コード生成タスクに最適化
   - 説明なしの純粋なPythonコードを返す

2. **コード実行エージェント**（ハイブリッドStrands-Agents + AgentCore）：
   - **プライマリモード**: AgentCore CodeInterpreterツール付きStrands-Agentsエージェント
   - **モデル**: コード生成器と同じ（Claude Sonnet 3.7 → Nova Premier → Claude 3.5 Sonnet）
   - 実際のコード実行にAgentCoreの`code_session`を使用
   - AWS管理のサンドボックス環境でPythonコードを実行

### AgentCore統合パターン

アプリケーションは**公式AgentCoreサンプルパターン**に従います：

```python
from bedrock_agentcore.tools.code_interpreter_client import code_session
from strands import Agent, tool  # strands-agentsパッケージ
import json

@tool
def execute_python(code: str, description: str = "") -> str:
    """公式サンプルに従ってAgentCoreを使用してPythonコードを実行"""
    if description:
        code = f"# {description}\n{code}"
    
    with code_session(aws_region) as code_client:
        response = code_client.invoke("executeCode", {
            "code": code,
            "language": "python",
            "clearContext": False
        })
    
    for event in response["stream"]:
        return json.dumps(event["result"])

# AgentCoreツール付きStrands-Agentsエージェント
agent = Agent(
    model=bedrock_model,
    tools=[execute_python],
    system_prompt="ツールを使用してコードを実行"
)
```

## 前提条件

- Python 3.8以上
- Node.js 16以上
- Bedrockアクセス付きAWSアカウント
- AWS CLI設定またはAWS認証情報

## インストール

1. **プロジェクトディレクトリにクローンしてナビゲート**：
   ```bash
   cd /Users/nmurich/strands-agents/agent-core/code-interpreter
   ```

2. **セットアップスクリプトを実行**：
   ```bash
   ./setup.sh
   ```

3. **AWS認証情報を設定**：
   AWS認証情報で`.env`ファイルを編集：
   ```bash
   AWS_REGION=us-east-1
   AWS_ACCESS_KEY_ID=your_access_key_here
   AWS_SECRET_ACCESS_KEY=your_secret_key_here
   ```

## 使用法

### クイックスタート

単一コマンドでバックエンドとフロントエンドの両方を実行：
```bash
./start.sh
```

これにより以下が開始されます：
- `http://localhost:8000`のバックエンドAPIサーバー
- `http://localhost:3000`のReactフロントエンド

### 手動開始

**バックエンド**：
```bash
source venv/bin/activate
cd backend
python main.py
```

**フロントエンド**：
```bash
cd frontend
npm start
```

## アプリケーション機能

### 1. コード生成
- コードに何をさせたいかを説明する自然言語プロンプトを入力
- Strandsエージェント経由でClaude Sonnet 3.7を使用してPythonコードを作成するため「コード生成」をクリック
- **生成されたコードは自動的にコードエディターにロードされます**
- 編集または実行前に生成されたコードをプレビュー
- 即座に実行するか編集してから実行するかを選択

### 2. コードエディター
- Python構文ハイライト付きMonacoエディター
- **生成されたコードで自動的に入力**
- 既存のPythonファイルをアップロード
- 生成またはアップロードされたコードを編集
- コードが生成された時の視覚的インジケーター
- **入力処理付きインタラクティブ実行サポート**
- シングルクリックでコードを実行

### 3. インタラクティブコード実行
- インタラクティブコード（input()呼び出し）の**自動検出**
- 必要な入力を特定し値を提案する**コード解析**
- 実行前に入力を提供するための**インタラクティブ実行モーダル**
- インタラクティブコードをテストするための**事前提供入力シミュレーション**
- 複雑なインタラクティブシナリオのサポート（ループ、条件文）
- インタラクティブ実行詳細を表示する**視覚的インジケーター**

### 4. 実行結果
- 構文ハイライト付きフォーマット済み表示で実行出力を表示
- 詳細なエラーメッセージと適切なスタイル付きエラーハンドリング
- 組み込みコピー機能で出力をクリップボードにコピー
- 提供された入力を表示する**インタラクティブ実行詳細**
- コードを簡単に再実行
- 成功した実行とエラーの明確な区別

### 5. セッション履歴
- エージェント帰属付きで生成されたすべてのコードとプロンプトを追跡
- タイムスタンプ付き実行履歴を表示
- 入力詳細付き**インタラクティブ実行追跡**
- 以前のコードスニペットを再実行
- アプリケーション使用中のセッション永続化
- どのエージェント（Strands対AgentCore）が各操作を処理したかを確認

## APIエンドポイント

### コード生成
```http
POST /api/generate-code
Content-Type: application/json

{
  "prompt": "フィボナッチ数を計算する関数を作成",
  "session_id": "optional-session-id"
}
```

### コード実行
```http
POST /api/execute-code
Content-Type: application/json

{
  "code": "print('Hello, World!')",
  "session_id": "optional-session-id",
  "interactive": false,
  "inputs": ["input1", "input2"]
}
```

### インタラクティブコード解析
```http
POST /api/analyze-code
Content-Type: application/json

{
  "code": "name = input('Your name: ')",
  "session_id": "optional-session-id"
}
```

### ファイルアップロード
```http
POST /api/upload-file
Content-Type: application/json

{
  "filename": "script.py",
  "content": "# Pythonコードコンテンツ",
  "session_id": "optional-session-id"
}
```

### セッション履歴
```http
GET /api/session/{session_id}/history
```

## 設定

### 環境変数

| 変数 | 説明 | デフォルト |
|----------|-------------|---------|
| `AWS_PROFILE` | AWS CLIプロファイル名（推奨） | `default` |
| `AWS_REGION` | Bedrock用AWSリージョン | `us-east-1` |
| `AWS_ACCESS_KEY_ID` | AWSアクセスキー（フォールバック） | プロファイルがない場合必須 |
| `AWS_SECRET_ACCESS_KEY` | AWSシークレットキー（フォールバック） | プロファイルがない場合必須 |
| `BACKEND_HOST` | バックエンドサーバーホスト | `0.0.0.0` |
| `BACKEND_PORT` | バックエンドサーバーポート | `8000` |
| `REACT_APP_API_URL` | フロントエンドAPI URL | `http://localhost:8000` |

### AWS認証優先度

アプリケーションは以下の認証優先度を使用します：

1. **AWSプロファイル**（推奨）：`.env`ファイルの`AWS_PROFILE`を使用
2. **アクセスキー**（フォールバック）：`AWS_ACCESS_KEY_ID`と`AWS_SECRET_ACCESS_KEY`を使用

**`.env`設定例：**
```bash
# オプション1: AWSプロファイルを使用（推奨）
AWS_PROFILE=default
AWS_REGION=us-east-1

# オプション2: アクセスキーを使用（プロファイルがない場合のみ）
# AWS_ACCESS_KEY_ID=your_access_key_here
# AWS_SECRET_ACCESS_KEY=your_secret_key_here
```

### AgentCore設定

アプリケーションは2つの実行モードをサポートします：

- **AgentCoreモード**: Amazon Bedrock AgentCoreでのフルコード実行
- **Strandsシミュレーション**: AgentCoreが利用できない場合のコード解析とシミュレーション

AgentCoreには追加権限（`bedrock-agentcore:*`）が必要です。利用できない場合、アプリケーションは自動的にStrandsシミュレーションにフォールバックします。

## セキュリティ考慮事項

- コード実行はAgentCoreのサンドボックス環境で実行
- セッションデータはメモリに保存（再起動時に永続化されない）
- AWS認証情報は適切に保護する必要があります
- プロダクション使用では認証の実装を検討

## トラブルシューティング

### 一般的な問題

1. **「サーバーからの応答なし」エラー**：
   ```bash
   # バックエンドが実行中かチェック
   curl http://localhost:8000/health
   
   # 診断を実行
   python diagnose_backend.py
   
   # デバッグのためバックエンドを手動開始
   ./start_manual.sh
   ```

2. **バックエンドの開始に失敗**：
   - `.env`ファイルのAWS認証情報をチェック
   - AWSアカウントでBedrockアクセスが有効になっていることを確認
   - Python依存関係がインストールされていることを確認
   - 実行：`python test_strands.py`でstrands-agentsフレームワークをチェック

3. **Strands-Agentsフレームワークが見つからない**：
   ```bash
   # strands-agentsが利用可能かチェック
   python test_strands.py
   
   # インストールされていない場合、インストール
   pip install strands-agents
   
   # またはローカルに存在するかチェック
   ls -la ../strands*
   ```

4. **フロントエンドがバックエンドに接続できない**：
   - バックエンドがポート8000で実行中であることを確認
   - CORS設定をチェック
   - フロントエンド設定のAPI URLを確認

5. **コード実行が失敗**：
   - AgentCore初期化をチェック
   - Bedrockモデルアクセスを確認
   - 特定のエラーの実行ログをレビュー

### 診断ツール

1. **フル診断**：
   ```bash
   python diagnose_backend.py
   ```

2. **AWS認証テスト**：
   ```bash
   python test_aws_auth.py
   ```

3. **Strands-Agentsフレームワークテスト**：
   ```bash
   python test_strands.py
   ```

4. **AgentCore統合テスト**：
   ```bash
   python test_agentcore_integration.py
   ```

5. **フロントエンドコンポーネントテスト**：
   ```bash
   node test_frontend.js
   ```

6. **インタラクティブ実行テスト**：
   ```bash
   python test_interactive.py
   ```

### 手動起動

自動開始スクリプトが失敗した場合、手動起動を使用：

1. **バックエンドのみ**：
   ```bash
   ./start_manual.sh
   ```

2. **フロントエンドのみ**（別ターミナルで）：
   ```bash
   cd frontend
   npm start
   ```

### ログ

バックエンドログはコンソールに出力されます。デバッグ用：
```bash
# 詳細ログ付きでバックエンドを実行
cd backend
python main.py --log-level debug

# またはログファイルをチェック（改善されたstart.shを使用している場合）
tail -f backend.log
tail -f frontend.log
```

## 開発

### プロジェクト構造
```
├── backend/
│   └── main.py              # FastAPIバックエンドサーバー
├── frontend/
│   ├── src/
│   │   ├── components/      # Reactコンポーネント
│   │   ├── services/        # APIサービス
│   │   └── App.js          # メインReactアプリ
│   └── package.json
├── requirements.txt         # Python依存関係
├── setup.sh                # セットアップスクリプト
├── start.sh                # 開始スクリプト
└── README.md
```

### 新機能の追加

1. **バックエンド**: `backend/main.py`に新しいエンドポイントを追加
2. **フロントエンド**: `frontend/src/components/`に新しいコンポーネントを追加
3. **API**: 新しいエンドポイント用に`frontend/src/services/api.js`を更新

## 貢献

1. リポジトリをフォーク
2. 機能ブランチを作成
3. 変更を加える
4. 徹底的にテスト
5. プルリクエストを提出

## ライセンス

このプロジェクトはMITライセンスの下でライセンスされています。

## サポート

問題と質問については：
1. トラブルシューティングセクションをチェック
2. AWS Bedrock AgentCoreドキュメントをレビュー
3. リポジトリでイシューを開く