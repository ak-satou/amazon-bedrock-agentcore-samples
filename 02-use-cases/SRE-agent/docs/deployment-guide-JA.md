# Amazon Bedrock AgentCore Runtime向けSREエージェントデプロイメントガイド

このガイドでは、ローカルテストから Amazon Bedrock AgentCore Runtime での本番デプロイメントまで、SRE エージェントの完全なデプロイメントプロセスについて説明します。

## 前提条件

- 適切な権限で設定されたAWS CLI
- Dockerのインストールと実行
- UVパッケージマネージャーのインストール
- Python 3.12+
- Amazon Bedrock AgentCore Runtimeへのアクセス
- `BedrockAgentCoreFullAccess`ポリシーと適切な信頼ポリシーを持つIAMロール（[認証設定](auth.md)を参照）

## 環境設定

SRE エージェントは設定に環境変数を使用します。これらは適切なディレクトリの `.env` ファイルから読み込まれます：

- **CLIテスト**: 環境変数は`sre_agent/.env`から読み込まれます
- **コンテナビルド**: 環境変数は`deployment/.env`から読み込まれます
- **Dockerプラットフォーム**: ローカルビルドは`Dockerfile.x86_64`（linux/amd64）を使用し、AgentCoreデプロイメントは`Dockerfile`（linux/arm64）を使用します

### 必須環境変数

以下の変数を含む適切な`.env`ファイルを作成してください：

**sre_agent/.env（CLIテストおよびローカルコンテナ実行用）:**
```bash
GATEWAY_ACCESS_TOKEN=your_gateway_access_token
LLM_PROVIDER=bedrock
DEBUG=false
# Anthropicプロバイダーを使用する場合は、以下も追加:
# ANTHROPIC_API_KEY=sk-ant-your-key-here
```

**deployment/.env（コンテナビルドおよびデプロイメント用）:**
```bash
GATEWAY_ACCESS_TOKEN=your_gateway_access_token
ANTHROPIC_API_KEY=sk-ant-your-key-here
# これらはビルド/デプロイ時に環境変数でオーバーライドできます
```

**注意**: `--env-file`を使用する場合、すべての必要な変数が.envファイルに含まれている必要があります。`-e`は.envファイルから特定の変数をオーバーライドする場合にのみ使用してください。

## デプロイメントシーケンス

### フェーズ1: CLIによるローカルテスト

まず、コマンドラインインターフェースを使用してSREエージェントをローカルでテストし、正しく動作することを確認します。

#### 1.1 環境設定

環境ファイルを作成し設定します：
```bash
# CLI環境ファイルのセットアップ
cp sre_agent/.env.example sre_agent/.env
# あなたの設定でsre_agent/.envを編集
```

**注意**: 環境変数は実行時にオーバーライドできますが、.envファイルを持つことで一貫した設定が保証されます。

#### 1.2 Bedrock（デフォルト）でCLIテスト

```bash
# デフォルトのBedrockプロバイダーでテスト
uv run sre-agent --prompt "list the pods in my infrastructure"

# デバッグ出力を有効にしてテスト
uv run sre-agent --prompt "list the pods in my infrastructure" --debug

# 特定のプロバイダーでテスト
uv run sre-agent --prompt "list the pods in my infrastructure" --provider bedrock --debug
```

#### 1.3 AnthropicプロバイダーでCLIテスト

```bash
# .envファイルにANTHROPIC_API_KEYが設定されていることを確認してから:
uv run sre-agent --prompt "list the pods in my infrastructure" --provider anthropic --debug
```

**期待される出力**: エージェントがリクエストを処理し、適切な専門エージェントにルーティングし、インフラストラクチャ情報を返すのを確認できるはずです。

### フェーズ2: ローカルコンテナテスト

CLIテストが成功したら、エージェントをローカルでコンテナとしてビルドしテストします。

#### 2.1 ローカルコンテナのビルド

ビルドスクリプトはオプションのECRリポジトリ名を受け取り、ターゲットプラットフォームに基づいて異なるDockerfileを使用します：

- **ローカルビルド**（LOCAL_BUILD=true）: linux/amd64プラットフォーム用に`Dockerfile.x86_64`を使用
- **AgentCoreビルド**（デフォルト）: linux/arm64プラットフォーム用に`Dockerfile`を使用（AgentCoreで必要）

```bash
# カスタム名でローカルテスト用コンテナをビルド
LOCAL_BUILD=true ./deployment/build_and_deploy.sh my_custom_sre_agent

# すべてのオプションのヘルプを表示
./deployment/build_and_deploy.sh --help
```

#### 2.2 Bedrockでローカルコンテナをテスト

デフォルトのBedrockプロバイダーでコンテナをローカル実行：
```bash
# sre_agentディレクトリの.envファイルを使用（推奨）
# sre_agent/.envにLLM_PROVIDER=bedrockが設定されていることを確認
docker run -p 8080:8080 --env-file sre_agent/.env my_custom_sre_agent:latest

# 代替: 明示的な環境変数（.envファイルを使用しない場合）
docker run -p 8080:8080 \
  -v ~/.aws:/root/.aws:ro \
  -e AWS_PROFILE=default \
  -e GATEWAY_ACCESS_TOKEN=your_token \
  -e LLM_PROVIDER=bedrock \
  my_custom_sre_agent:latest

# デバッグ有効（.envファイルのDEBUG設定をオーバーライド）
docker run -p 8080:8080 --env-file sre_agent/.env -e DEBUG=true my_custom_sre_agent:latest
```

**注意**: コンテナ名はビルド時に指定したECRリポジトリ名と一致します。

#### 2.3 Anthropicでローカルコンテナをテスト

```bash
# .envファイルを使用（sre_agent/.envにLLM_PROVIDER=anthropicが設定されていることを確認）
docker run -p 8080:8080 --env-file sre_agent/.env my_custom_sre_agent:latest

# デバッグ有効（.envファイルのDEBUG設定をオーバーライド）
docker run -p 8080:8080 \
  --env-file sre_agent/.env \
  -e DEBUG=true \
  my_custom_sre_agent:latest
```

**注意**: anthropicプロバイダーを使用する場合は、`sre_agent/.env`ファイルに`LLM_PROVIDER=anthropic`と`ANTHROPIC_API_KEY`の両方が設定されていることを確認してください。

#### 2.4 curlでコンテナをテスト

実行中のコンテナをテスト：
```bash
# 基本テスト
curl -X POST http://localhost:8080/invocations \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "prompt": "list the pods in my infrastructure"
    }
  }'

# ヘルスチェック
curl http://localhost:8080/ping
```

**期待される出力**: コンテナはエージェントの応答を含むJSONで応答するはずです。

### フェーズ3: Amazon Bedrock AgentCore Runtimeデプロイメント

ローカルコンテナテストが成功したら、AgentCoreにデプロイします。

#### 3.1 BedrockでAgentCoreにデプロイ

```bash
# カスタムリポジトリ名とデフォルト設定でデプロイ（deployment/.envから読み込み）
./deployment/build_and_deploy.sh my_custom_sre_agent

# デバッグ有効でデプロイ（環境変数オーバーライド）
DEBUG=true ./deployment/build_and_deploy.sh my_custom_sre_agent

# 特定のプロバイダーでデプロイ
LLM_PROVIDER=bedrock DEBUG=true ./deployment/build_and_deploy.sh my_custom_sre_agent
```

#### 3.2 AnthropicでAgentCoreにデプロイ

```bash
# Anthripoicプロバイダーでデプロイ（deployment/.envにANTHROPIC_API_KEYがあることを確認）
LLM_PROVIDER=anthropic ./deployment/build_and_deploy.sh my_custom_sre_agent

# Anthropicとデバッグ有効でデプロイ
DEBUG=true LLM_PROVIDER=anthropic ./deployment/build_and_deploy.sh my_custom_sre_agent

# 環境変数でAPIキーをオーバーライド
LLM_PROVIDER=anthropic ANTHROPIC_API_KEY=sk-ant-your-key ./deployment/build_and_deploy.sh my_custom_sre_agent
```

**ビルドスクリプトの使用法:**
```bash
# 利用可能なすべてのオプションを表示
./deployment/build_and_deploy.sh --help

# スクリプトは1つのオプション引数を受け取ります: ECRリポジトリ名
# デフォルトのリポジトリ名は'sre_agent'です
# 注意: リポジトリ名にはハイフン（-）ではなくアンダースコア（_）を使用してください
```

**期待される出力**: スクリプトはビルド、ECRへのプッシュ、AgentCore Runtimeへのデプロイを行います。

#### 3.3 AgentCoreデプロイメントのテスト

invokeスクリプトを使用してデプロイされたエージェントをテスト：
```bash
# デプロイされたエージェントをテスト
uv run python deployment/invoke_agent_runtime.py \
  --prompt "list the pods in my infrastructure"

# カスタムランタイムARNでテスト
uv run python deployment/invoke_agent_runtime.py \
  --prompt "list the pods in my infrastructure" \
  --runtime-arn "arn:aws:bedrock-agentcore:us-east-1:123456789012:runtime/your-runtime-id"
```

## 環境変数リファレンス

### コア設定

| 変数 | 説明 | デフォルト | 必須 |
|----------|-------------|---------|----------|
| `GATEWAY_ACCESS_TOKEN` | ゲートウェイ認証トークン | - | はい |
| `BACKEND_API_KEY` | 認証プロバイダー用バックエンドAPIキー | - | はい（ゲートウェイ設定） |
| `LLM_PROVIDER` | 言語モデルプロバイダー | `bedrock` | いいえ |
| `ANTHROPIC_API_KEY` | Anthropic APIキー | - | anthropicプロバイダー使用時のみ |
| `DEBUG` | デバッグログとトレースを有効化 | `false` | いいえ |

### AWS設定

| 変数 | 説明 | デフォルト | 必須 |
|----------|-------------|---------|----------|
| `AWS_REGION` | デプロイメント用のAWSリージョン | `us-east-1` | いいえ |
| `AWS_PROFILE` | 使用するAWSプロファイル | - | いいえ |
| `RUNTIME_NAME` | AgentCoreランタイム名 | ECRリポジトリ名 | いいえ |

### ビルドスクリプト設定

| 変数 | 説明 | デフォルト | 注意 |
|----------|-------------|---------|-------|
| `LOCAL_BUILD` | ローカルテスト専用ビルド | `false` | trueの場合Dockerfile.x86_64を使用 |
| `PLATFORM` | ターゲットプラットフォーム | `arm64` | AgentCoreはarm64が必要、ローカルはx86_64を使用 |
| `ECR_REPO_NAME` | ECRリポジトリ名 | `sre_agent` | コマンドライン引数として渡すことも可能 |

## デバッグモード使用法

### CLIデバッグモード
```bash
# --debugフラグでデバッグを有効化
uv run sre-agent --prompt "your query" --debug

# または環境変数で
DEBUG=true uv run sre-agent --prompt "your query"
```

### コンテナデバッグモード
```bash
# ローカルコンテナでデバッグ（.envファイルのDEBUG設定をオーバーライド）
docker run -p 8080:8080 --env-file sre_agent/.env -e DEBUG=true my_custom_sre_agent:latest

# AgentCoreデプロイメントでデバッグ
DEBUG=true ./deployment/build_and_deploy.sh my_custom_sre_agent
```

### デバッグ出力例

**デバッグモードなし:**
```
🤖 Multi-Agent System: Processing...
🧭 Supervisor: Routing to kubernetes_agent
🔧 Kubernetes Agent:
   💡 Full Response: Here are the pods in your infrastructure...
💬 Final Response: I found 5 pods running in your infrastructure...
```

**デバッグモードあり:**
```
🤖 Multi-Agent System: Processing...

MCP tools loaded: 12
  - kubernetes-list-pods: List all pods in the cluster...
  - kubernetes-get-pod: Get details of a specific pod...

🧭 Supervisor: Routing to kubernetes_agent
🔧 Kubernetes Agent:
   🔍 DEBUG: agent_messages = 3
   📋 Found 3 trace messages:
      1. AIMessage: I'll help you list the pods...
   📞 Calling tools:
      kubernetes-list-pods(
        namespace=None
      ) [id: call_123]
   🛠️  kubernetes-list-pods [id: call_123]:
      {"pods": [...]}
   💡 Full Response: Here are the pods in your infrastructure...
💬 Final Response: I found 5 pods running in your infrastructure...
```

## プロバイダー設定

### Amazon Bedrock（デフォルト）を使用
```bash
# CLI（sre_agent/.envから読み込み）
uv run sre-agent --provider bedrock --prompt "your query"

# コンテナ（sre_agent/.envからLLM_PROVIDER=bedrockを読み込み）
docker run -p 8080:8080 --env-file sre_agent/.env my_custom_sre_agent:latest

# デプロイメント（deployment/.envから読み込み、環境変数でオーバーライド可能）
LLM_PROVIDER=bedrock ./deployment/build_and_deploy.sh my_custom_sre_agent
```

### Anthropic Claudeを使用
```bash
# CLI（sre_agent/.envからLLM_PROVIDERとANTHROPIC_API_KEYを読み込み）
uv run sre-agent --provider anthropic --prompt "your query"

# コンテナ（sre_agent/.envからLLM_PROVIDER=anthropicとANTHROPIC_API_KEYを読み込み）
docker run -p 8080:8080 --env-file sre_agent/.env my_custom_sre_agent:latest

# デプロイメント（deployment/.envから読み込み、環境変数でオーバーライド可能）
LLM_PROVIDER=anthropic ./deployment/build_and_deploy.sh my_custom_sre_agent

# APIキーを環境変数でオーバーライド（deployment/.envにない場合）
LLM_PROVIDER=anthropic ANTHROPIC_API_KEY=sk-ant-xxx ./deployment/build_and_deploy.sh my_custom_sre_agent
```

## トラブルシューティング

### よくある問題

1. **ゲートウェイトークンの問題**
   ```bash
   # トークンが設定されていることを確認
   echo $GATEWAY_ACCESS_TOKEN
   # または.envファイルをチェック
   cat sre_agent/.env
   ```

2. **プロバイダー設定**
   ```bash
   # Anthropicの場合、APIキーが有効であることを確認
   echo $ANTHROPIC_API_KEY
   # シンプルな呼び出しでAPIキーをテスト
   ```

3. **デバッグ情報**
   ```bash
   # デバッグモードを有効にして詳細ログを確認
   DEBUG=true uv run sre-agent --prompt "test"
   ```

4. **コンテナの問題**
   ```bash
   # コンテナログを確認
   docker logs <container_id>
   # デバッグで実行
   docker run -e DEBUG=true ... my_custom_sre_agent:latest
   ```

### 確認手順

1. **CLI動作確認**: エージェントがローカルでクエリに応答する
2. **コンテナ動作確認**: コンテナがcurlリクエストに応答する
3. **AgentCore動作確認**: デプロイされたエージェントがinvokeスクリプト経由で応答する

## クイックスタート: コピー&ペーストコマンド列

`my_custom_sre_agent`を使用した完全なデプロイメントのために、これらのコマンドを順番にコピー&ペーストしてください：

### 1. ローカルコンテナをビルド
```bash
LOCAL_BUILD=true ./deployment/build_and_deploy.sh my_custom_sre_agent
```

### 2. ローカルコンテナをテスト（Bedrock）
```bash
docker run -p 8080:8080 --env-file sre_agent/.env my_custom_sre_agent:latest
```

### 3. curlでテスト
```bash
curl -X POST http://localhost:8080/invocations \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "prompt": "list the pods in my infrastructure"
    }
  }'
```

### 4. AgentCoreにデプロイ
```bash
./deployment/build_and_deploy.sh my_custom_sre_agent
```

### 5. AgentCoreデプロイメントをテスト
```bash
uv run python deployment/invoke_agent_runtime.py \
  --prompt "list the pods in my infrastructure"
```

## ベストプラクティス

1. **開発**: 常にローカルで最初にテストする
2. **環境ファイル**: 一貫した設定のために`.env`ファイルを使用する
3. **デバッグモード**: トラブルシューティング時にデバッグモードを有効にする
4. **プロバイダーテスト**: 両方を使用する場合はBedrockとAnthropicプロバイダーの両方をテストする
5. **段階的デプロイメント**: 本番環境の前にステージング環境にデプロイする