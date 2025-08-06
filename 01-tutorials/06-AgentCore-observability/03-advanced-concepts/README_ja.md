# 高度なコンセプト - オブザーバビリティ

このフォルダには、Amazon Bedrock AgentCoreの高度なオブザーバビリティコンセプトとテクニックが含まれています。

## 📁 内容

### 01-custom-span-creation/
エージェントワークフローの拡張トレーシングと監視のためのカスタムスパンの作成方法を学習します。


## 🚀 はじめに

1. `.env.example`を`.env`にコピーし、AWSクレデンシャルとCloudwatch変数を設定します。
2. リージョンでAWS CloudWatchのTransaction Searchを有効化します。
3. 依存関係をインストールします：`pip install -r requirements.txt`
4. Jupyterノートブックを開いてチュートリアルに従います


## 🧹 クリーンアップ

チュートリアル完了後、サンプル実行中に作成されたCloudWatchログ グループをクリーンアップします。