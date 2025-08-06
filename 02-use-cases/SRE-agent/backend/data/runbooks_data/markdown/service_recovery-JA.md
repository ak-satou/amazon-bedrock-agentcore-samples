# サービス復旧手順

このドキュメントには、さまざまなサービスおよび完全スタック復旧プロセスの復旧手順が含まれています。

## Webサービス復旧

**復旧ID:** `web-service-recovery`  
**サービス:** web-service

### 復旧ステップ

#### ステップ1: サービスヘルスチェック
- **コマンド:** `kubectl get pods -l app=web-app`
- **期待結果:** すべてのPodがRunning状態であること

#### ステップ2: 不健全なPodの再起動
- **コマンド:** `kubectl delete pod <unhealthy-pod-name>`
- **期待結果:** 新しいPodが起動してready状態になること

#### ステップ3: 必要に応じてデプロイメントをスケール
- **コマンド:** `kubectl scale deployment web-app-deployment --replicas=5`
- **期待結果:** 追加のPodが起動すること

#### ステップ4: ロードバランサーを検証
- **コマンド:** `kubectl get svc web-app-service`
- **期待結果:** 外部IPが割り当てられていること

#### ステップ5: サービスエンドポイントをテスト
- **コマンド:** `curl http://<external-ip>/health`
- **期待結果:** 200 OKを返すこと

### ロールバック手順

**トリガー:** 30分後に復旧が失敗した場合

#### ロールバックステップ
1. 前のデプロイメントリビジョンを取得: `kubectl rollout history deployment/web-app-deployment`
2. 前のバージョンにロールバック: `kubectl rollout undo deployment/web-app-deployment`
3. ロールバック状況を監視: `kubectl rollout status deployment/web-app-deployment`
4. ロールバック後のサービスヘルスを検証

---

## データベース復旧

**復旧ID:** `database-recovery`  
**サービス:** database

### 復旧ステップ

#### ステップ1: データベースPodの状態確認
- **コマンド:** `kubectl get pods -l app=database`
- **期待結果:** Podが実行中であること

#### ステップ2: 永続ボリュームの検証
- **コマンド:** `kubectl get pv,pvc -n production`
- **期待結果:** PVCがバインドされていること

#### ステップ3: データベースログの確認
- **コマンド:** `kubectl logs -f database-pod-name`
- **期待結果:** クリティカルエラーがないこと

#### ステップ4: データベース接続のテスト
- **コマンド:** `kubectl exec -it database-pod -- psql -U postgres -c 'SELECT 1'`
- **期待結果:** クエリが正常に返ること

#### ステップ5: 該当する場合はレプリケーションを検証
- **コマンド:** `kubectl exec -it database-pod -- psql -U postgres -c 'SELECT * FROM pg_stat_replication'`
- **期待結果:** レプリカが接続されていること

### データ復旧

**バックアップ場所:** s3://backup-bucket/database/

#### 復元手順
1. アプリケーションの書き込みを停止
2. 空のボリュームで新しいデータベースPodを作成
3. 最新のバックアップから復元: `pg_restore -d dbname backup.dump`
4. データ整合性を検証
5. アプリケーショントラフィックを再開

---

## 完全スタック復旧

**復旧ID:** `full-stack-recovery`  
**タイトル:** 完全スタック復旧

### サービス優先順序
1. database
2. cache-service
3. api-service
4. web-service
5. ingress-controller

### 復旧前チェック
- クラスターヘルスを検証: `kubectl get nodes`
- リソース可用性を確認: `kubectl top nodes`
- 最近のイベントを確認: `kubectl get events --sort-by=.metadata.creationTimestamp`

### 復旧フェーズ

#### フェーズ1: インフラストラクチャ
- ノードヘルスを検証
- ネットワーク接続を確認
- ストレージ可用性を確保

#### フェーズ2: データ層
- データベースサービスを復旧
- データ整合性を検証
- 必要に応じてキャッシュを復元

#### フェーズ3: アプリケーション層
- バックエンドサービスを開始
- サービス発見を検証
- フロントエンドサービスを開始

#### フェーズ4: 検証
- ヘルスチェックを実行
- スモークテストを実施
- メトリクスを監視