# Strands保険エージェント

ローカルMCPサーバーに接続して保険情報と見積もりを提供するStrandsで構築されたインタラクティブ エージェント。

![Strands保険エージェント デモ](images/strands_local_agent_conversation.gif)

## 概要

このプロジェクトは、Strands AgentsをMCP（Model Context Protocol）サーバーと一緒に使用してインタラクティブ保険アシスタントを作成する方法を実証します。エージェントは、AWS BedrockをしてClaude 3.7 Sonnetを活用し、MCPサーバーを通じて公開されたローカル保険APIツールに接続します。

## 前提条件

- Python 3.10以上
- Claude 3.7 SonnetへのBedrockアクセス権を持つAWSアカウント
- ローカルMCPサーバーがhttp://localhost:8000/mcpで動作中
- 保険APIがhttp://localhost:8001で動作中

## プロジェクト構造

```
strands-insurance-agent/
├── interactive_insurance_agent.py  # メイン インタラクティブ エージェント
├── strands_insurance_agent.py      # 非インタラクティブ エージェント
├── requirements.txt                # プロジェクト依存関係
├── strands_local_agent.png         # 動作中のエージェントのスクリーンショット
└── README.md                       # このファイル
```

## セットアップ手順

### 1. リポジトリのクローン（まだの場合）

```bash
git clone <repository-url>
cd local_prototype/strands-insurance-agent
```

### 2. 仮想環境のセットアップ

```bash
python -m venv venv
source venv/bin/activate  # Windowsの場合：venv\Scripts\activate
```

### 3. 依存関係のインストール

```bash
pip install -r requirements.txt
```

### 4. AWS認証情報のセットアップ

このエージェントは、AWS BedrockてClaude 3.7 Sonnetにアクセスします。AWS認証情報が適切に設定されていることを確認してください：

1. まだAWS CLIをインストールしていない場合はインストール：
   ```bash
   pip install awscli
   ```

2. [リンク](https://strandsagents.com/latest/documentation/docs/user-guide/quickstart/#configuring-credentials)を使用して認証情報を設定


### 5. 必要なサービスの開始

エージェントを実行する前に、これらのサービスが動作していることを確認します：

1. **保険API**：
   ```bash
   cd ../insurance_api
   python -m uvicorn server:app --port 8001
   ```

2. **MCPサーバー**：
   ```bash
   cd ../native_mcp_server
   python server.py
   ```

## インタラクティブ エージェントの実行

すべてがセットアップされたら、インタラクティブ エージェントを実行します：

```bash
python interactive_insurance_agent.py
```

これにより、保険商品について質問し、個人的な見積もりを取得できるインタラクティブ チャット セッションが開始されます。エージェントは、セッション全体を通じてローカル チャット履歴を維持し、自然な会話の流れとフォローアップ質問を可能にします。

### チャット履歴

ローカル チャット履歴機能：
- セッション中にメモリ内のすべての会話ターンを保存
- 以前のやり取りについてのコンテキストをモデルに提供
- エージェントがより関連性が高く個人的なレスポンスを提供できるようにする
- 特定の顧客や車両について議論する際の連続性を維持する
- プログラム終了時にクリアされる（セッション間の永続ストレージなし）

## 機能

インタラクティブ保険エージェントには以下の機能が含まれます：

1. **インタラクティブ チャット インターフェース**：
   - 絵文字強化コンソール インターフェース
   - 自然な会話の流れ
   - コンテキスト保持付きコマンド履歴

2. **保険API統合**：
   - 顧客情報検索
   - 車両情報取得
   - 保険見積もり生成
   - 車両安全性評価

3. **高度な機能**：
   - 包括的なログ記録（コンソールとファイルベース）
   - エラー処理と回復
   - より良い可読性のためのレスポンス フォーマット
   - ツール使用追跡

4. **ローカル チャット履歴と会話コンテキスト**：
   - すべての会話ターンのメモリ内履歴を維持
   - シームレスなフォローアップのためにセッション全体を通じて持続
   - エージェントが以前の質問と回答を参照できるようにする
   - クエリ全体で顧客と車両の詳細を記憶
   - 会話の流れに基づくコンテキスト レスポンスを提供

## クエリ例

試すことができるクエリ例：

- 「顧客cust-001についてどのような情報を持っていますか？」
- 「2020年のToyota Camryについて教えてもらえますか？」
- 「顧客ID cust-001向けの2020年Toyota Camryに利用可能な保険オプションは何ですか？」
- 「Honda Civicの安全性評価は何ですか？」
- 「顧客cust-002向けの2022年Ford F-150の見積もりをもらえますか。」

### チャト履歴を使用した会話例

エージェントは以前のコンテキストを記憶するため、以下のような自然な会話ができます：

```
あなた：顧客cust-001についてどのような情報を持っていますか？
エージェント：[John Smithについての情報を返す]

あなた：彼はどのような車を持っていますか？
エージェント：[以前のコンテキストを使用してJohn Smithについて質問されていることを知る]

あなた：彼の2023年Toyota RAV4の見積もりをもらえますか？
エージェント：[顧客情報と新しい車両リクエストを組み合わせる]
```

このコンテキスト認識により、やり取りがよりくて効率的になります。

## アーキテクチャ

Strands保険エージェントは以下の間の橋渡しとして機能します：

1. **ユーザー**（コンソール インターフェース経由）
2. **Strandsエージェント**（Claude 3.7 Sonnetを使用）
3. **MCPサーバー**（保険ツールを提供）
4. **保険API**（実際のデータを提供）

質問をすると：
1. エージェントがそれをフォーマットしてStrandsを通じてClaude 3.7に送信
2. Claudeが質問に基づいて使用するツールを決定
3. エージェントがMCPサーバーを通じて必要なAPI呼び出しを実行
4. 結果が収集され、フォーマットされ、提示される

## ログ記録

エージェントはすべてのやり取りを以下にログ記録します：
- コンソール（デフォルトでDEBUGレベル）
- ファイル（`insurance_agent.log` - ファイル ログ記録が有効な場合）

ログには以下が含まれます：
- ユーザー入力とエージェント レスポンス
- ツール呼び出しとその引数
- レスポンス処理詳細
- エラー情報

## トラブルシューティング

### よくある問題

1. **AWS認証情報が不足または無効**
   - エラー：「Could not connect to the endpoint URL」
   - 解決策：AWS認証情報とリージョンを確認

2. **MCPサーバーが動作していない**
   - エラー：MCPサーバーに接続する際の「Connection refused」
   - 解決策：http://localhost:8000/mcpでMCPサーバーを開始

3. **保険APIが動作していない**
   - エラー：「Error connecting to auto insurance API」
   - 解決策：http://localhost:8001で保険APIが動作していることを確認

4. **レスポンス フォーマットト問題**
   - エラー：エージェントがクリーンなテキストの代わりに生のJSONやリスト形式を表示
   - 解決策：新しいStrandsバージョンに対してレスポンス パーサーの更新が必要な可能性

## 📝 ライセンス

このプロジェクトはApache License 2.0の下でライセンスされています - 詳細については[LICENSE](../../../../LICENSE)ファイルを参照してください。

## 注意

- MCPサーバー：LocalMCP FastMCPサーバーに基づく
- 保険API：FastAPIで構築
- エージェント フレームワーク：AnthropicのStrands Agents
- LLM：AWS Bedrock経由のClaude 3.7 Sonnet