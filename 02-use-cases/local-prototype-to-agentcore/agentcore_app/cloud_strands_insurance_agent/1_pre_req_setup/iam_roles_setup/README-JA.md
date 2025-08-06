# Bedrock AgentCore用のIAMロール セットアップ

このディレクトリには、適切な権限を持つAWS Bedrock AgentCoreアプリケーション用の必要なIAMロールをセットアップするためのスクリプトが含まれています。

## クイック セットアップ

迅速で簡単なセットアップには、シェル スクリプトを使用してください：

```bash
./setup_role.sh
```

このスクリプトは以下を実行します：
1. AWS認証情報をチェック
2. 適切なデフォルト値で必要な情報を求める
3. すべての必要な権限を持つIAMロールを作成
4. 設定で使用するためのロールARNを表示

## 手動セットアップ

よりカスタマイズされたセットアップを希望する場合は、Pythonモジュールを使用できます：

1. `iam_config.ini`を作成/編集して設定を構成：
   ```bash
   python3 config.py
   ```

2. インタラクティブにセットアップを実行：
   ```bash
   python3 -c "from collect_info import run_interactive_setup; run_interactive_setup()"
   ```

## 必要な権限

IAMロールには以下の権限が含まれます：
- ECR（コンテナ レジストリ アクセス）
- CloudWatch Logs
- X-Rayトレーシング
- CloudWatchメトリクス
- Bedrock AgentCoreアクセス トークン
- Bedrockモデル呼び出し

これらの権限は、最小権限の原則でAWSのベスト プラクティスに従います。

## 前提条件

- 適切な権限で設定されたAWS CLIがインストールされている
- IAMロールとポリシーを作成する権限を持つAWSアカウント

## ファイル

- `setup_role.sh` - クイック セットアップ シェル スクリプト
- `config.py` - 設定管理
- `policy_templates.py` - IAMポリシー テンプレート
- `collect_info.py` - インタラクティブ設定収集
- `trust-policy.json` - 信頼関係ポリシー テンプレート

## トラブルシューティング

問題が発生した場合は、以下を確認してください：

- AWS認証情報が適切に設定されている（`aws configure`）
- IAMロールを作成するのに十分な権限を持っている
- AWS CLIがインストールされ、PATHに含まれている

## セキュリティ ノート

作成されたIAMロールはセキュリティのベスト プラクティスに従います：
- 条件付きの厳格な信頼ポリシー
- 権限の最小権限の原則
- 可能な場合のリソースベースの制限