# カスタマーサポートエージェント

> [!CAUTION]
> このリポジトリで提供されている例は、実験的および教育目的のみです。概念や技術を実証するものですが、本番環境での直接使用を意図したものではありません。[プロンプトインジェクション](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-injection.html)から保護するために、Amazon Bedrock Guardrailsを設置することを確認してください。

これは、AWS Bedrock AgentCoreフレームワークを使用したカスタマーサポートエージェントの実装です。このシステムは、保証確認、顧客プロファイル管理、Googleカレンダー統合、およびAmazon Bedrock Knowledge Base検索機能を備えたAI駆動のカスタマーサポートインターフェースを提供します。

![architecture](./images/architecture.png)

## 目次

- [カスタマーサポートエージェント](#カスタマーサポートエージェント)
  - [目次](#目次)
  - [前提条件](#前提条件)
    - [AWSアカウントのセットアップ](#awsアカウントのセットアップ)
  - [デプロイ](#デプロイ)
  - [サンプルクエリ](#サンプルクエリ)
  - [スクリプト](#スクリプト)
    - [Amazon Bedrock AgentCore Gateway](#amazon-bedrock-agentcore-gateway)
      - [Amazon Bedrock AgentCore Gatewayの作成](#amazon-bedrock-agentcore-gatewayの作成)
      - [Amazon Bedrock AgentCore Gatewayの削除](#amazon-bedrock-agentcore-gatewayの削除)
    - [Amazon Bedrock AgentCore Memory](#amazon-bedrock-agentcore-memory)
      - [Amazon Bedrock AgentCore Memoryの作成](#amazon-bedrock-agentcore-memoryの作成)
      - [Amazon Bedrock AgentCore Memoryの削除](#amazon-bedrock-agentcore-memoryの削除)
    - [Cognito Credentials Provider](#cognito-credentials-provider)
      - [Cognito Credentials Providerの作成](#cognito-credentials-providerの作成)
      - [Cognito Credentials Providerの削除](#cognito-credentials-providerの削除)
    - [Google Credentials Provider](#google-credentials-provider)
      - [Credentials Providerの作成](#credentials-providerの作成)
      - [Credentials Providerの削除](#credentials-providerの削除)
    - [Agent Runtime](#agent-runtime)
      - [Agent Runtimeの削除](#agent-runtimeの削除)
  - [クリーンアップ](#クリーンアップ)
  - [🤝 貢献](#-貢献)
  - [📄 ライセンス](#-ライセンス)
  - [🆘 サポート](#-サポート)
  - [🔄 更新](#-更新)

## 前提条件

### AWSアカウントのセットアップ

1. **AWSアカウント**: 適切な権限を持つアクティブなAWSアカウントが必要です
   - [AWSアカウントを作成](https://aws.amazon.com/account/)
   - [AWSコンソールアクセス](https://aws.amazon.com/console/)

2. **AWS CLI**: 認証情報を使用してAWS CLIをインストールし、設定します
   - [AWS CLIのインストール](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
   - [AWS CLIの設定](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html)

   ```bash
   aws configure
   ```

3. **Bedrockモデルアクセス**: AWSリージョンでAmazon Bedrock Anthropic Claude 4.0モデルへのアクセスを有効にします
   - [Amazon Bedrockコンソール](https://console.aws.amazon.com/bedrock/)に移動
   - 「Model access」に移動して以下のアクセスをリクエストします：
     - Anthropic Claude 4.0 Sonnetモデル
     - Anthropic Claude 3.5 Haikuモデル
   - [Bedrock Model Accessガイド](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html)

4. **Python 3.10+**: アプリケーションの実行に必要です
   - [Pythonダウンロード](https://www.python.org/downloads/)

5. **カレンダーアクセス用のOAuth 2.0認証情報の作成**: Googleカレンダー統合のため
   - [Google OAuth Setup](./prerequisite/google_oauth_setup.md)をフォローしてください

## デプロイ

1. **インフラストラクチャの作成**

    ```bash
    python -m venv .venv
    source .venv/bin/activate
    pip install -r dev-requirements.txt

    chmod +x scripts/prereq.sh
    ./scripts/prereq.sh

    chmod +x scripts/list_ssm_parameters.sh
    ./scripts/list_ssm_parameters.sh
    ```

    > [!CAUTION]
    > すべてのリソース名に`customersupport`のプレフィックスを付けてください。

2. **Agentcore Gatewayの作成**

    ```bash
    python scripts/agentcore_gateway.py create --name customersupport-gw
    ```

3. **Agentcore Identityのセットアップ**

    - **Cognito Credential Providerのセットアップ**

    ```bash
    python scripts/cognito_credentials_provider.py create --name customersupport-gateways

    python test/test_gateway.py --prompt "Check warranty with serial number MNO33333333"
    ```

    - **Google Credential Providerのセットアップ**

    [Google Credentials](./prerequisite/google_oauth_setup.md)のセットアップ手順に従ってください。

    ```bash
    python scripts/google_credentials_provider.py create --name customersupport-google-calendar

    python test/test_google_tool.py
    ```

4. **Memoryの作成**

    ```bash
    python scripts/agentcore_memory.py create --name customersupport

    python test/test_memory.py load-conversation
    python test/test_memory.py load-prompt "My preference of gaming console is V5 Pro"
    python test/test_memory.py list-memory
    ```

5. **Agent Runtimeのセットアップ**

> [!CAUTION]
> エージェントの名前が`customersupport`で始まることを確認してください。
    
  ```bash
  agentcore configure --entrypoint main.py -er arn:aws:iam::<Account-Id>:role/<Role> --name customersupport<AgentName>
  ```

  `./scripts/list_ssm_parameters.sh`を使用して以下を入力してください：
  - `Role = ValueOf(/app/customersupport/agentcore/runtime_iam_role)`
  - `OAuth Discovery URL = ValueOf(/app/customersupport/agentcore/cognito_discovery_url)`
  - `OAuth client id = ValueOf(/app/customersupport/agentcore/web_client_id)`

  ![configure](./images/runtime_configure.png)

  > [!CAUTION]
  > agentcore launchを実行する前に、`.agentcore.yaml`を削除することを確認してください。

  ```bash

  rm .agentcore.yaml

  agentcore launch

  python test/test_agent.py customersupport<AgentName> -p "Hi"
  ```

  ![code](./images/code.png)

6. **Local Host Streamlit UI**

> [!CAUTION]
> Streamlitアプリはポート`8501`でのみ実行してください。

```bash
streamlit run app.py --server.port 8501 -- --agent=customersupport<AgentName>
```

## サンプルクエリ

1. Gaming Console Proデバイスを持っています。保証状態を確認したいのですが、保証シリアル番号はMNO33333333です。

2. 保証サポートガイドラインは何ですか？

3. 今日の予定は何ですか？

4. 保証更新のための通話をセットアップするイベントを作成できますか？

5. デバイスで過熱の問題があります。デバッグを手伝ってください。

## スクリプト

### Amazon Bedrock AgentCore Gateway

#### Amazon Bedrock AgentCore Gatewayの作成

```bash
python scripts/agentcore_gateway.py create --name my-gateway
python scripts/agentcore_gateway.py create --name my-gateway --api-spec-file custom/path.json
```

#### Amazon Bedrock AgentCore Gatewayの削除

```bash
# Gatewayの削除（gateway.configから自動読み込み）
python scripts/agentcore_gateway.py delete

# 確認をスキップして削除
python scripts/agentcore_gateway.py delete --confirm
```

### Amazon Bedrock AgentCore Memory

#### Amazon Bedrock AgentCore Memoryの作成

```bash
python scripts/agentcore_memory.py create --name MyMemory
python scripts/agentcore_memory.py create --name MyMemory --event-expiry-days 60
```

#### Amazon Bedrock AgentCore Memoryの削除

```bash
# Memoryの削除（SSMから自動読み込み）
python scripts/agentcore_memory.py delete

# 確認をスキップして削除
python scripts/agentcore_memory.py delete --confirm
```

### Cognito Credentials Provider

#### Cognito Credentials Providerの作成

```bash
python scripts/cognito_credentials_provider.py create --name customersupport-gateways
```

#### Cognito Credentials Providerの削除

```bash
# Providerの削除（SSMから名前を自動読み込み）
python scripts/cognito_credentials_provider.py delete

# 名前を指定してProviderの削除
python scripts/cognito_credentials_provider.py delete --name customersupport-gateways

# 確認をスキップして削除
python scripts/cognito_credentials_provider.py delete --confirm
```

### Google Credentials Provider

#### Credentials Providerの作成

```bash
python scripts/google_credentials_provider.py create --name customersupport-google-calendar
python scripts/google_credentials_provider.py create --name my-provider --credentials-file /path/to/credentials.json
```

#### Credentials Providerの削除

```bash
# Providerの削除（SSMから名前を自動読み込み）
python scripts/google_credentials_provider.py delete

# 名前を指定してProviderの削除
python scripts/google_credentials_provider.py delete --name customersupport-google-calendar

# 確認をスキップして削除
python scripts/google_credentials_provider.py delete --confirm
```

### Agent Runtime

#### Agent Runtimeの削除

```bash
# 名前を指定してAgent Runtimeの削除
python scripts/agentcore_agent_runtime.py customersupport

# 実際に削除せずに削除対象をプレビュー
python scripts/agentcore_agent_runtime.py --dry-run customersupport

# 名前を指定してAgent Runtimeの削除
python scripts/agentcore_agent_runtime.py <agent-name>
```

## クリーンアップ

```bash
chmod +x scripts/cleanup.sh
./scripts/cleanup.sh

python scripts/google_credentials_provider.py delete
python scripts/cognito_credentials_provider.py delete
python scripts/agentcore_memory.py delete
python scripts/agentcore_gateway.py delete
python scripts/agentcore_agent_runtime.py customersupport<AgentName>

rm .agentcore.yaml
rm .bedrock_agentcore.yaml
```

## 🤝 貢献

貢献を歓迎します！詳細については[貢献ガイドライン](../../CONTRIBUTING.md)をご覧ください：

- 新しいサンプルの追加
- 既存の例の改善
- 問題の報告
- 機能強化の提案

## 📄 ライセンス

このプロジェクトはMITライセンスの下でライセンスされています - 詳細は[LICENSE](../../LICENSE)ファイルをご覧ください。

## 🆘 サポート

- **問題**: [GitHub Issues](https://github.com/awslabs/amazon-bedrock-agentcore-samples/issues)でバグ報告や機能リクエストを行ってください
- **ドキュメント**: 特定のガイダンスについては、個別のフォルダのREADMEを確認してください

## 🔄 更新

このリポジトリは積極的にメンテナンスされ、新しい機能と例で更新されています。最新の追加情報を把握するためにリポジトリをウォッチしてください。