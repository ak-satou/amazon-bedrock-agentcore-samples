# 結果の検証

SREエージェントには、調査結果が幻覚情報ではなく実際のデータに基づいて正確であることを検証するためのツールが含まれています。

## グラウンドトゥルース検証

結果検証のために、包括的なグラウンドトゥルースデータセットを作成するデータダンプユーティリティを提供しています：

```bash
# 検証用の完全なデータダンプを生成
cd backend/scripts
./dump_data_contents.sh
```

このスクリプトは[`backend/data`](../backend/data)ディレクトリ内のすべてのファイル（`.json`、`.txt`、`.log`ファイルを含む）を処理し、[`backend/data/all_data_dump.txt`](../backend/data/all_data_dump.txt)に包括的なダンプを作成します。このファイルは、エージェントの応答が事実に基づいており、作り話ではないことを検証するためのグラウンドトゥルースとして機能します。

## レポート検証

[`reports`](../reports)フォルダには、いくつかの例のクエリに対する調査レポートが含まれています。LLM-as-a-judge検証システムを使用して、これらのレポートをグラウンドトゥルースデータと照合して検証できます：

```bash
# 特定のレポートをグラウンドトゥルースと照合して検証
python verify_report.py --report reports/example_report.md --ground-truth backend/data/all_data_dump.txt
```

## 検証ワークフローの例

```bash
# 1. 調査レポートを生成
sre-agent --prompt "payment-serviceポッドがクラッシュループしているのはなぜですか？"

# 2. グラウンドトゥルースデータダンプを作成
cd backend/scripts && ./dump_data_contents.sh && cd ../..

# 3. レポートが事実情報のみを含んでいることを検証
python verify_report.py --report reports/your_report_.md --ground-truth backend/data/all_data_dump.txt
```

>**⚠️ 重要な注意**: [`sre_agent/agent_nodes.py`](../sre_agent/agent_nodes.py)のシステムプロンプトとエージェントロジックは、本番使用前にさらなる改良が必要です。この実装はアーキテクチャアプローチを示し、本番対応のSREエージェント構築の基盤を提供しますが、プロンプト、エラーハンドリング、エージェント調整ロジックは実世界での信頼性のために追加の調整が必要です。