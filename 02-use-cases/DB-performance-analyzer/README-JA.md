# AI駆動DBパフォーマンスアナライザー

このプロジェクトは、Amazon Bedrock AgentCoreを使用してAI駆動のデータベースパフォーマンスアナライザーを構築する方法を実演します。自然言語会話を通じてデータベースパフォーマンスの分析、クエリの説明、推奨事項の提供、データベース運用の最適化支援ができるインテリジェントなエージェントを作成します。

## 概要

DBパフォーマンスアナライザーは、データベース管理者と開発者がPostgreSQLデータベースのパフォーマンス問題を特定・解決するのを支援するAI駆動アシスタントです。Amazon Bedrock AgentCoreと大規模言語モデルを活用することで、データベースメトリクスと統計に基づいた人間らしい分析と推奨事項を提供します。

## ユースケース

- **パフォーマンストラブルシューティング**: 遅いクエリ、接続問題、その他のパフォーマンスボトルネックの迅速な特定と診断
- **インデックス最適化**: インデックス使用状況の分析と、インデックスの作成、修正、削除の推奨事項
- **リソース使用率**: CPU、メモリ、I/O使用量の監視と最適化
- **メンテナンス計画**: autovacuumパフォーマンスの洞察とメンテナンスタスクの推奨事項
- **レプリケーション監視**: レプリケーション遅延の追跡と高可用性の確保
- **クエリ最適化**: 複雑なクエリの説明と改善提案

## アーキテクチャ

![DB performance analyzer architecture](./images/db-analyzer-architecture.png)

### VPC接続

Lambda関数はデータベースと同じVPCにデプロイされ、セキュアな通信を可能にします：

1. **自動VPC検出**: セットアップスクリプトがデータベースクラスターのVPC、サブネット、セキュリティグループを自動検出
2. **セキュリティグループ設定**: Lambda関数専用のセキュリティグループを作成し、データベースセキュリティグループを設定してアクセスを許可
3. **プライベートネットワーク通信**: すべてのデータベーストラフィックがVPC内に留まり、パブリックインターネットを通過しない
4. **セキュアな認証情報管理**: データベース認証情報はAWS Secrets Managerに保存され、Lambda関数によってセキュアにアクセス
5. **VPCエンドポイント**: 正しいDNS設定でSecrets ManagerやSSMなどのAWSサービス用に適切に設定されたVPCエンドポイント

## プロセスフロー

1. **ユーザークエリ**: ユーザーがAmazon Qを通じて自然言語でデータベースパフォーマンスについて質問
2. **クエリ処理**: Amazon Qがクエリを処理し、適切なAgentCore Gatewayにルーティング
3. **ツール選択**: AgentCore Gatewayがクエリに基づいて適切なツールを選択
4. **データ収集**: Lambda関数がデータベースに接続し、関連するメトリクスと統計を収集
5. **分析**: Lambda関数が収集したデータを分析し、洞察を生成
6. **応答生成**: 結果がフォーマットされ、自然言語の説明と推奨事項としてユーザーに返される

## プロジェクト構造

```
.
├── README.md               # このファイル
├── setup.sh                # メインセットアップスクリプト
├── setup_database.sh       # データベース設定スクリプト
├── cleanup.sh              # クリーンアップスクリプト
├── setup_observability.sh  # ゲートウェイとターゲットの観測性設定
├── cleanup_observability.sh # 観測性リソースのクリーンアップ
├── config/                 # 設定ファイル（セットアップ中に生成）
│   └── *.env               # 環境固有の設定ファイル（Gitにコミットされない）
└── scripts/                # サポートスクリプト
    ├── create_gateway.py   # AgentCore Gatewayの作成
    ├── create_iam_roles.sh # 必要なIAMロールの作成
    ├── create_lambda.sh    # Lambda関数の作成
    ├── create_target.py    # Gatewayターゲットの作成
    ├── lambda-target-analyze-db-performance.py # パフォーマンス分析ツール
    ├── lambda-target-analyze-db-slow-query.py  # 遅いクエリ分析ツール
    ├── get_token.py        # 認証トークンの取得/リフレッシュ
    └── test_vpc_connectivity.py # AWSサービス接続のテスト
```

### 設定ファイル

セットアップ処理により、`config/`ディレクトリに複数の設定ファイルが自動生成されます：

- **cognito_config.env**: Cognitoユーザープール、クライアント、トークン情報を含む
- **gateway_config.env**: Gateway ID、ARN、リージョンを含む
- **iam_config.env**: IAMロールARNとアカウント情報を含む
- **db_dev_config.env/db_prod_config.env**: データベース接続情報を含む
- **vpc_config.env**: VPC、サブネット、セキュリティグループIDを含む
- **target_config.env**: Gatewayターゲット設定を含む
- **pgstat_target_config.env**: pg_stat_statementsターゲットの設定を含む

これらのファイルは機密情報を含むため、`.gitignore`によりGitから除外されています。

## 前提条件

- 適切な権限で設定されたAWS CLI
- Python 3.9以降
- Boto3ライブラリのインストール
- jqコマンドラインツールのインストール
- Amazon Aurora PostgreSQLまたはRDS PostgreSQLデータベースへのアクセス
- AWS Secrets Managerでシークレットを作成する権限
- AWS Systems Manager Parameter Storeでパラメータを作成する権限
- VPCセキュリティグループを作成・修正する権限
- VPC設定でLambda関数を作成する権限
- VPCエンドポイントを作成する権限（Lambda関数がAWSサービスにアクセスする必要がある場合）

## セットアップ手順

1. リポジトリのクローン:
   ```bash
   git clone https://github.com/awslabs/amazon-bedrock-agentcore-samples.git
   cd amazon-bedrock-agentcore-samples/02-use-cases/02-DB-performance-analyzer
   ```

2. Python仮想環境の作成:
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   ```

3. データベースアクセスの設定:
   ```bash
   ./setup_database.sh --cluster-name your-aurora-cluster --environment prod
   ```

   このスクリプトは以下を実行します：
   - クラスターの既存シークレットを検索
   - 見つかった場合、使用するものを選択させる
   - 見つからない場合、ユーザー名とパスワードを入力要求
   - RDSからクラスターエンドポイントとポートを取得
   - 必要な形式でAWS Secrets Managerにシークレットを作成
   - SSM Parameter Storeにシークレット名を保存
   - 設定をファイルに保存
   
   既存のシークレットを直接指定することもできます：
   ```bash
   ./setup_database.sh --cluster-name your-aurora-cluster --environment prod --existing-secret your-secret-name
   ```

4. メインセットアップスクリプトの実行:
   ```bash
   ./setup.sh
   ```

   このスクリプトは以下を実行します：
   - 認証用のAmazon Cognitoリソースを設定
   - 必要なIAMロールを作成
   - DBパフォーマンス分析用のLambda関数を作成
   - 必要に応じてAWSサービス用のVPCエンドポイントを設定
   - Amazon Bedrock AgentCore Gatewayを作成
   - Lambda関数用のGatewayターゲットを作成
   - すべてが連携するように設定

5. Amazon Qでゲートウェイを使用するための設定:
   ```bash
   source venv/bin/activate
   python3 scripts/get_token.py
   deactivate
   ```

   これにより、`~/.aws/amazonq/mcp.json`ファイルがゲートウェイ設定で更新されます。

## DBパフォーマンスアナライザーの使用

セットアップ後、Amazon Qを通じてDBパフォーマンスアナライザーを使用できます：

1. コマンドプロンプトでAmazon Q CLI `q chat`を開く
2. "db-performance-analyzer"エージェントが読み込まれる
3. データベースパフォーマンスについて以下のような質問をする：
   - "本番データベースの遅いクエリを分析して"
   - "dev環境の接続管理問題をチェックして"
   - "データベースのインデックス使用状況を分析して"
   - "本番のautovacuum問題をチェックして"

## 利用可能な分析ツール

DBパフォーマンスアナライザーは複数のツールを提供します：

- **遅いクエリ分析**: 遅いクエリを特定・説明し、最適化の推奨事項を提供
- **接続管理**: 接続問題、アイドル接続、接続パターンを分析してリソース使用率を改善
- **インデックス分析**: インデックス使用状況を評価し、不足または未使用のインデックスを特定し、改善を提案
- **Autovacuum分析**: autovacuum設定をチェックし、デッドタプルを監視し、設定変更を推奨
- **I/O分析**: I/Oパターン、バッファ使用状況、チェックポイント活動を分析してボトルネックを特定
- **レプリケーション分析**: レプリケーション状態、遅延、健全性を監視して高可用性を確保
- **システム健全性**: キャッシュヒット率、デッドロック、長時間実行トランザクションを含む全体的なシステム健全性メトリクスを提供
- **クエリ説明**: クエリ実行計画を説明し、最適化提案を提供
- **DDL抽出**: データベースオブジェクトのData Definition Language（DDL）ステートメントを抽出
- **クエリ実行**: 安全にクエリを実行し、結果を返す

## 主要な利点

- **自然言語インターフェース**: 平易な英語でデータベースと相互作用
- **予防的推奨事項**: パフォーマンス改善のための実行可能な提案を取得
- **時間節約**: 手動で診断するのに数時間かかる問題を迅速に特定
- **教育的**: AI説明を通じてデータベース内部とベストプラクティスを学習
- **アクセシブル**: 複雑なSQLクエリや監視コマンドを覚える必要なし
- **包括的**: 1つのツールでデータベースパフォーマンスの複数の側面をカバー

## 観測性

DBパフォーマンスアナライザーには、AgentCore GatewayとLambdaターゲットの監視とトラブルシューティングに役立つ観測性機能が含まれています。

### クイックセットアップ

1. 観測性セットアップスクリプトの実行:
   ```bash
   ./setup_observability.sh
   ```

2. CloudWatchコンソールでCloudWatch Transaction Searchを有効化。

3. CloudWatch Logs、X-Ray Traces、Transaction Searchでデータを確認。

### クリーンアップ

```bash
./cleanup_observability.sh
```

詳細なセットアップ手順、ランタイム外エージェントの設定オプション、カスタムヘッダー、ベストプラクティスを含むAgentCore観測性機能の包括的なドキュメントについては、[AgentCore Observability](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/observability.html)を参照してください。

## トラブルシューティング

### VPC接続問題

Lambda関数がAWSサービスへの接続に問題がある場合：

1. **DNS設定の確認**:
   - VPCでDNS解決とDNSホスト名が有効になっていることを確認
   - VPCエンドポイントでプライベートDNSが有効になっていることを確認

2. **環境変数の確認**:
   - Lambda関数には`AWS_REGION`環境変数を正しく設定する必要があります
   - Secrets Managerなどのサービスの場合、コードでリージョンが指定されていることを確認

3. **接続性のテスト**:
   ```bash
   python3 scripts/test_vpc_connectivity.py
   ```

4. **セキュリティグループの確認**:
   - LambdaセキュリティグループがVPCエンドポイントセキュリティグループへのアウトバウンドトラフィックを許可していることを確認
   - VPCエンドポイントセキュリティグループがLambdaセキュリティグループからのインバウンドトラフィックを許可していることを確認

5. **ルートテーブルの確認**:
   - Lambda関数が実行されるサブネットに適切なルートを持つルートテーブルがあることを確認

6. **手動確認コマンド**:
   ```bash
   # DNS設定の確認
   aws ec2 describe-vpc-attribute --vpc-id YOUR_VPC_ID --attribute enableDnsSupport
   aws ec2 describe-vpc-attribute --vpc-id YOUR_VPC_ID --attribute enableDnsHostnames

   # VPCエンドポイントのリスト
   aws ec2 describe-vpc-endpoints --filters "Name=vpc-id,Values=YOUR_VPC_ID"

   # サブネットのルートテーブル確認
   aws ec2 describe-route-tables --filters "Name=association.subnet-id,Values=YOUR_SUBNET_ID"
   ```

### Lambda関数の問題

Lambda関数が環境変数の欠落に関連するエラーで失敗する場合：

1. **必要な環境変数の確認**:
   - Lambda設定ですべての必要な環境変数が設定されていることを確認
   - `REGION`環境変数はAWSサービスアクセスに特に重要

2. **Lambda設定の更新**:
   ```bash
   aws lambda update-function-configuration \
     --function-name DBPerformanceAnalyzer \
     --environment "Variables={REGION=us-west-2}" \
     --region us-west-2
   ```

3. **エラーの説明**:
   `Error: Failed to extract database object DDL: 'REGION'`や`cannot access local variable 'conn'`のようなエラーが表示される場合、環境変数の欠落が原因の可能性があります。

4. **代替解決策**:
   Lambda関数コードを更新して、環境変数の欠落を適切なデフォルトで処理することもできます：
   ```python
   region = os.getenv('REGION', os.getenv('AWS_REGION', 'us-west-2'))
   ```

### シークレット管理の問題

データベースシークレットで問題がある場合：

1. **シークレット形式の確認**:
   - シークレットには`username`、`password`、`host`、`port`、`dbname`フィールドが含まれている必要があります
   - setup_database.shスクリプトはシークレット名の特殊文字を処理するようになりました

2. **シークレットアクセスの確認**:
   - Lambda実行ロールがシークレットにアクセスする権限があることを確認
   - シークレットがLambda関数と同じリージョンにあることを確認

3. **シークレットアクセスのテスト**:
   ```bash
   ./scripts/list_secrets.sh --filter your-cluster-name
   ```

### 観測性トラブルシューティング

観測性データが表示されない場合：

1. **CloudWatch Transaction Searchの確認**: CloudWatchコンソールで有効になっていることを確認
2. **ログループの確認**: ゲートウェイとターゲット用のログループが存在することを確認
3. **トラフィックの生成**: トレースとログを生成するためにゲートウェイにいくつかのリクエストを送信

## クリーンアップ

このプロジェクトで作成されたすべてのリソースを削除するには：

```bash
./cleanup.sh
```

これにより以下が削除されます：
- Lambda関数
- Gatewayターゲット
- Gateway
- Cognitoリソース
- IAMロール
- VPCエンドポイント（作成された場合）
- 設定ファイル

注意：デフォルトでは、スクリプトはAWS Secrets ManagerのシークレットやSSM Parameter Storeのパラメータを削除しません。これらのリソースも削除するには：

```bash
./cleanup.sh --delete-secrets
```

## 認証のリフレッシュ

認証トークンが期限切れになった場合：

```bash
source venv/bin/activate
python3 scripts/get_token.py
deactivate
```

## サンプルクエリ

DBパフォーマンスアナライザーに質問できるサンプルクエリ：

- "本番データベースで最も遅い5個のクエリは何ですか？"
- "dev環境で接続管理の問題はありますか？"
- "データベースのインデックス使用状況を分析して改善を提案してください"
- "本番データベースでautovacuumは効果的に機能していますか？"
- "現在データベースで高I/Oの原因は何ですか？"
- "データベースでレプリケーション遅延はありますか？"
- "本番データベースの全体的な健全性チェックをしてください"
- "このクエリの実行計画を説明してください: SELECT * FROM users WHERE email LIKE '%example.com'"
- "データベースのusersテーブルのDDLを抽出してください"

## 貢献

貢献を歓迎します！お気軽にPull Requestを提出してください。