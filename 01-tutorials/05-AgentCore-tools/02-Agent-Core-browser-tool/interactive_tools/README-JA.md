# Amazon Bedrock Agentcore SDK ツールの例

このフォルダには、AgentCore SDK ツールの使用例が含まれています：

## ブラウザツール

* `browser_viewer.py` - 適切な表示サイズサポートを備えたAmazon Bedrock Agentcoreブラウザライブビューア。
* `run_live_viewer.py` - Bedrock Agentcoreブラウザライブビューアを実行するためのスタンドアロンスクリプト。

## コードインタープリタツール

* `dynamic_research_agent_langgraph.py` - 動的コード生成を備えたLangGraphベースの調査エージェント

## 前提条件

### Python依存関係
```bash
pip install -r requirements.txt
```

必要なパッケージ：fastapi、uvicorn、rich、boto3、bedrock-agentcore

### AWS認証情報（S3ストレージ用）
S3記録ストレージの場合、AWS認証情報が設定されていることを確認してください：
```bash
aws configure
```

## 例の実行

### ブラウザライブビューア
`02-Agent-Core-browser-tool`ディレクトリから：
```bash
python -m interactive_tools.run_live_viewer
```

### 動的調査エージェント
`02-Agent-Core-browser-tool`ディレクトリから：
```bash
python -m interactive_tools.dynamic_research_agent_langgraph
```

### Bedrockモデルアクセス
動的調査エージェントの例では、Amazon BedrockのClaudeモデルを使用します：
- AWSアカウントでAnthropic Claudeモデルへのアクセスが必要です
- デフォルトのモデルは`anthropic.claude-3-5-sonnet-20240620-v1:0`です
- `dynamic_research_agent_langgraph.py`のこの行を変更することでモデルを変更できます：
  ```python
  # DynamicResearchAgent.__init__()の38行目
  self.llm = ChatBedrockConverse(
      model="anthropic.claude-3-5-sonnet-20240620-v1:0", # <- これを希望のモデルに変更
      region_name=region
  )
  ```
- [Amazon Bedrockコンソール](https://console.aws.amazon.com/bedrock/home#/modelaccess)でモデルアクセスを申請してください

### セッションリプレイ
`02-Agent-Core-browser-tool/interactive_tools`ディレクトリから：
```bash
python -m live_view_sessionreplay.browser_interactive_session
```

## ブラウザライブビューア

Amazon DCV技術を使用したリアルタイムブラウザ表示機能。

### 機能

**表示サイズ制御**
- 1280×720（HD）
- 1600×900（HD+）- デフォルト
- 1920×1080（フルHD）
- 2560×1440（2K）

**セッション制御**
- コントロール取得：自動化を無効にして手動で操作
- コントロール解放：自動化にコントロールを戻す

### 設定
- カスタムポート：`BrowserViewerServer(browser_client, port=8080)`

## ブラウザセッション記録と再生

デバッグ、テスト、デモンストレーション目的でブラウザセッションを記録・再生。

### 重要な制限事項
このツールはビデオストリームではなく、rrwebを使用してDOMイベントを記録します：
- 実際のブラウザコンテンツ（DCVキャンバス）はブラックボックスとして表示される可能性があります
- ピクセル完璧なビデオ記録の場合は、画面録画ソフトウェアを使用してください

## トラブルシューティング

### DCV SDKが見つからない
DCV SDKファイルが`interactive_tools/static/dcvjs/`に配置されていることを確認してください

### ブラウザセッションが表示されない
- ブラウザコンソール（F12）でエラーを確認
- AWS認証情報が適切なアクセス許可を持っていることを確認

### 再生中に記録が見つからない
- 記録が保存されたときに表示された正確なパスを確認
- S3記録の場合は、完全なS3 URLを使用
- `aws s3 ls`または`ls`コマンドを使用してファイルが存在することを確認

### S3アクセスエラー
- AWS認証情報が設定されていることを確認
- S3操作のIAMアクセス許可を確認
- バケット名がグローバルに一意であることを確認

## パフォーマンスに関する考慮事項
- 記録はブラウザのパフォーマンスにオーバーヘッドを追加します
- ファイルサイズは通常1分あたり1-10MBです
- S3アップロードは記録停止後に実行されます
- 再生にはまず全ファイルのダウンロードが必要です

## アーキテクチャノート
- ライブビューアはFastAPIを使用して事前署名されたDCV URLを提供
- 記録はrrwebライブラリを介してDOMイベントをキャプチャ
- 再生は再生にrrweb-playerを使用
- すべてのコンポーネントは同じBrowserClientインスタンスを共有
- モジュラー設計により各コンポーネントの独立使用が可能