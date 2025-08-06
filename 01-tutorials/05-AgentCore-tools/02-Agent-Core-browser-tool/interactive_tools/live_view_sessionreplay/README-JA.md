# Amazon Bedrock AgentCore SDK ツールの例

このフォルダには、Amazon Bedrock AgentCore SDKツールの使用例が含まれています：

## ブラウザツール

* `browser_viewer_replay.py` - 適切な表示サイズサポートを備えたAmazon Bedrock AgentCoreブラウザライブビューア。
* `browser_interactive_session.py` - ライブ表示、記録、再生機能を備えた完全なエンドツーエンドブラウザ体験。
* `session_replay_viewer.py` - 記録されたブラウザセッションを再生するためのビューア。
* `view_recordings.py` - S3から記録されたセッションを表示するスタンドアロンスクリプト。

## 前提条件

### Python依存関係
```bash
pip install -r requirements.txt
```

必要なパッケージ：fastapi、uvicorn、rich、boto3、bedrock-agentcore

### AWS認証情報
AWS認証情報が設定されていることを確認してください：
```bash
aws configure
```

## 例の実行

### 記録と再生を備えた完全なブラウザ体験
`02-Agent-Core-browser-tool/interactive_tools`ディレクトリから：
```bash
python -m live_view_sessionreplay.browser_interactive_session
```

### 記録の表示
`02-Agent-Core-browser-tool/interactive_tools`ディレクトリから：
```bash
python -m live_view_sessionreplay.view_recordings --bucket YOUR_BUCKET --prefix YOUR_PREFIX
```

## 記録と再生を備えた完全なブラウザ体験

ライブブラウザ表示、S3への自動記録、統合セッション再生を含む完全なエンドツーエンドワークフローを実行します。

### 機能
- S3への自動記録を備えたブラウザセッションの作成
- インタラクティブ制御（取得/解放）を備えたライブビュー
- 表示解像度のリアルタイム調整
- S3への自動セッション記録
- 記録を視聴するための統合セッション再生ビューア

### 仕組み
1. スクリプトは記録が有効なブラウザを作成します
2. ブラウザセッションが開始され、ローカルブラウザに表示されます
3. ブラウザの手動制御を取得するか、自動化を実行させることができます
4. すべてのアクションは自動的にS3に記録されます
5. セッションを終了した後（Ctrl+C）、記録を表示する再生ビューアが開きます

### 環境変数
- `AWS_REGION` - AWSリージョン（デフォルト：us-west-2）
- `AGENTCORE_ROLE_ARN` - ブラウザ実行用IAMロールARN（デフォルト：アカウントIDから自動生成）
- `RECORDING_BUCKET` - 記録用S3バケット（デフォルト：session-record-test-{ACCOUNT_ID}）
- `RECORDING_PREFIX` - 記録用S3プレフィックス（デフォルト：replay-data）

### 必要なIAMアクセス許可
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:CreateBucket",
                "s3:PutObject",
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::session-record-test-*",
                "arn:aws:s3:::session-record-test-*/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": "bedrock:*",
            "Resource": "*"
        }
    ]
}
```

## スタンドアロンセッション再生ビューア

新しいブラウザを作成せずに、S3から記録されたブラウザセッションを直接表示するための別のツールです。

### 機能
- S3に直接接続して記録を表示
- セッションIDを指定して過去の記録を表示
- セッションIDが提供されていない場合は、最新の記録を自動的に検索

### 使用方法

```bash
# バケット内の最新の記録を表示
python -m live_view_sessionreplay.view_recordings --bucket session-record-test-123456789012 --prefix replay-data

# 特定の記録を表示
python -m live_view_sessionreplay.view_recordings --bucket session-record-test-123456789012 --prefix replay-data --session 01JZVDG02M8MXZY2N7P3PKDQ74

# 特定のAWSプロファイルを使用
python -m live_view_sessionreplay.view_recordings --bucket session-record-test-123456789012 --prefix replay-data --profile my-profile
```

### 記録の検索

S3記録の一覧表示：
```bash
aws s3 ls s3://session-record-test-123456789012/replay-data/ --recursive
```

## トラブルシューティング

### DCV SDKが見つからない
DCV SDKファイルが`interactive_tools/static/dcvjs/`に配置されていることを確認してください

### ブラウザセッションが表示されない
- DCV SDKが適切にインストールされていることを確認
- ブラウザコンソール（F12）でエラーを確認
- AWS認証情報が適切なアクセス許可を持っていることを確認

### 記録が機能しない
- S3バケットが存在しアクセス可能であることを確認
- S3操作のIAMアクセス許可を確認
- 実行ロールが適切なアクセス許可を持っていることを確認

### セッション再生の問題
- S3に記録が存在することを確認（AWS CLIまたはコンソールを使用）
- コンソールログでエラーを確認
- S3バケットポリシーがオブジェクトの読み取りを許可していることを確認

### S3アクセスエラー
- AWS認証情報が設定されていることを確認
- S3操作のIAMアクセス許可を確認
- バケット名がグローバルに一意であることを確認

## アーキテクチャノート
- ライブビューアはFastAPIを使用して事前署名されたDCV URLを提供
- 記録はデータプレーンのブラウザサービスによって直接処理
- 再生は記録されたイベントの再生にrrweb-playerを使用
- すべてのコンポーネントは連携して動作することも独立して動作することも可能