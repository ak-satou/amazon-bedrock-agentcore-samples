# Amazon Bedrock AgentCore Memory

## 概要

メモリは知能の重要な構成要素です。大規模言語モデル（LLM）は印象的な能力を持っていますが、会話を跨いだ永続的なメモリが不足しています。Amazon Bedrock AgentCore Memoryは、AIエージェントが時間を通じてコンテキストを維持し、重要な事実を記憶し、一貫性のあるパーソナライズされた体験を提供することを可能にする管理サービスを提供することで、この制限に対処します。

## 主要機能

AgentCore Memoryは以下を提供します：

- **コアインフラストラクチャ**: 組み込み暗号化と可観測性を持つサーバーレスセットアップ
- **イベントストレージ**: ブランチングサポート付きの生イベントストレージ（会話履歴/チェックポイント）
- **戦略管理**: 設定可能な抽出戦略（SEMANTIC、SUMMARY、USER_PREFERENCES、CUSTOM）
- **メモリレコード抽出**: 設定された戦略に基づく事実、設定、要約の自動抽出
- **セマンティック検索**: 自然言語クエリを使用した関連メモリのベクトルベース検索

## AgentCore Memoryの動作

![high_level_workflow](./images/high_level_memory.png)

AgentCore Memoryは2つのレベルで動作します：

### 短期メモリ

単一のインタラクションまたは密接に関連するセッション内での継続性を提供する即座の会話コンテキストとセッションベース情報。

### 長期メモリ

複数の会話を跨いで抽出・保存される永続的な情報。時間を通じてパーソナライズされた体験を可能にする事実、設定、要約を含みます。

## メモリアーキテクチャ

1. **会話ストレージ**: 完全な会話が即座のアクセスのために生の形で保存されます
2. **戦略処理**: 設定された戦略がバックグラウンドで自動的に会話を分析します
3. **情報抽出**: 戦略タイプに基づいて重要なデータが抽出されます（通常約1分かかります）
4. **組織化されたストレージ**: 抽出された情報は効率的な検索のために構造化された名前空間に保存されます
5. **セマンティック検索**: 自然言語クエリがベクトル類似性を使用して関連するメモリを検索できます

## メモリ戦略タイプ

AgentCore Memoryは4つの戦略タイプをサポートします：

- **セマンティックメモリ**: 類似性検索用のベクトル埋め込みを使用して事実情報を保存
- **要約メモリ**: コンテキスト保存のために会話要約を作成・維持
- **ユーザー設定メモリ**: ユーザー固有の設定と設定を追跡
- **カスタムメモリ**: 抽出と統合ロジックのカスタマイズを許可

## 開始方法

組織化されたチュートリアルを通じてメモリ機能を探索してください：

- **[短期メモリ](./01-short-term-memory/)**: セッションベースメモリと即座のコンテキスト管理について学習
- **[長期メモリ](./02-long-term-memory/)**: 永続的なメモリ戦略と会話を跨いだ継続性を理解

## サンプルノートブック概要

| メモリタイプ | フレームワーク           | ユースケース           | ノートブック                                                                                                                           |
| ----------- | ------------------- | ------------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| 短期  | Strands Agent       | パーソナルエージェント     | [personal-agent.ipynb](./01-short-term-memory/01-single-agent/with-strands-agent/personal-agent.ipynb)                             |
| 短期  | LangGraph           | フィットネスコーチ      | [personal-fitness-coach.ipynb](./01-short-term-memory/01-single-agent/with-langgraph-agent/personal-fitness-coach.ipynb)           |
| 短期  | Strands Agent       | 旅行計画    | [travel-planning-agent.ipynb](./01-short-term-memory/02-multi-agent/with-strands-agent/travel-planning-agent.ipynb)                |
| 長期   | Strands Hooks       | カスタマーサポート   | [customer-support.ipynb](./02-long-term-memory/01-single-agent/using-strands-agent-hooks/customer-support/customer-support.ipynb)  |
| 長期   | Strands Hooks       | 数学アシスタント     | [math-assistant.ipynb](./02-long-term-memory/01-single-agent/using-strands-agent-hooks/simple-math-assistant/math-assistant.ipynb) |
| 長期   | Strands Tool        | 料理アシスタント | [culinary-assistant.ipynb](./02-long-term-memory/01-single-agent/using-strands-agent-memory-tool/culinary-assistant.ipynb)         |
| 長期   | Strands Multi-Agent | 旅行予約     | [travel-booking-assistant.ipynb](./02-long-term-memory/02-multi-agent/with-strands-agent/travel-booking-assistant.ipynb)           |

## 前提条件

- Python 3.10以上
- Amazon Bedrockアクセス権を持つAWSアカウント
- Jupyter Notebook環境
- 必要なPythonパッケージ（個別のサンプルrequirements.txtファイルを参照）