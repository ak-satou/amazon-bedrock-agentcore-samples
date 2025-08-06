# インシデントプレイブック

このドキュメントには、さまざまなタイプの運用問題に対するインシデント対応プレイブックが含まれています。

## 高メモリ使用率インシデント対応

**プレイブックID:** `memory-pressure-playbook`  
**インシデントタイプ:** パフォーマンス  
**深刻度:** 高  
**推定解決時間:** 15-30分

### 説明
高メモリ使用率インシデントを処理する手順

### トリガー
- メモリ使用率 > 85%
- ログ内のOutOfMemoryError
- メモリ不足によるPod退避

### 対応ステップ
1. `kubectl get pods --field-selector=status.phase=Running`を使用して影響を受けるPodを特定
2. メモリ使用量を確認: `kubectl top pods -n production`
3. 最近のメモリメトリクスと傾向を確認
4. 水平スケーリングが可能であればデプロイメントをスケールアップ
5. デプロイメント設定でメモリ制限を増加
6. 必要に応じて影響を受けるPodを再起動
7. 回復を監視し、正常運用を検証

### エスカレーション
- **プライマリ:** on-call-engineer
- **セカンダリ:** platform-team
- **マネージャー:** engineering-manager

### 関連ランブック
- pod-crashloop-troubleshooting
- resource-optimization

---

## データベース接続失敗対応

**プレイブックID:** `database-connection-failure`  
**インシデントタイプ:** 可用性  
**深刻度:** クリティカル  
**推定解決時間:** 5-15分

### 説明
データベース接続性問題を処理する手順

### トリガー
- データベース接続タイムアウトエラー
- 接続プール枯渇
- CrashLoopBackOff状態のデータベースPod

### 対応ステップ
1. データベースPodの状態を確認: `kubectl get pods -l app=database`
2. データベースログを確認: `kubectl logs -f database-pod-name`
3. データベースサービスエンドポイントを検証
4. サービス間のネットワーク接続を確認
5. 設定が正しい場合はデータベースPodを再起動
6. 必要に応じて接続プールをスケール
7. アプリケーションがデータベースに接続できることを検証

### エスカレーション
- **プライマリ:** database-admin
- **セカンダリ:** infrastructure-team
- **マネージャー:** site-reliability-manager

### 関連ランブック
- database-recovery
- connection-pool-tuning

---

## 高エラー率対応

**プレイブックID:** `high-error-rate-response`  
**インシデントタイプ:** 可用性  
**深刻度:** 高  
**推定解決時間:** 10-20分

### 説明
エラー率増加を処理する手順

### トリガー
- エラー率 > 10%
- 5xxエラーの増加
- 複数サービスの障害

### 対応ステップ
1. すべてのサービスの現在のエラー率を確認
2. エラーを引き起こしているソースサービスを特定
3. エラーパターンのアプリケーションログを確認
4. 最近のデプロイメントまたは設定変更を確認
5. 最近のデプロイメントの場合はロールバックを検討
6. 負荷関連の場合は影響を受けるサービスをスケール
7. カスケード障害の場合はサーキットブレーカーを有効化

### エスカレーション
- **プライマリ:** on-call-engineer
- **セカンダリ:** service-owner
- **マネージャー:** engineering-manager

### 関連ランブック
- rollback-procedures
- circuit-breaker-configuration

---

## Pod起動失敗解決

**プレイブックID:** `pod-startup-failure`  
**インシデントタイプ:** デプロイメント  
**深刻度:** 中  
**推定解決時間:** 10-30分

### 説明
Pod起動問題を解決する手順

### トリガー
- Pending状態でスタックしたPod
- ImagePullBackOffエラー
- initコンテナの失敗

### 対応ステップ
1. Podイベントを確認: `kubectl describe pod <pod-name>`
2. イメージの可用性とpullシークレットを検証
3. リソースクォータと制限を確認
4. 該当する場合はinitコンテナログを確認
5. 設定マップとシークレットを検証
6. ノードリソースとスケジューリング制約を確認
7. 修正された設定でPodを再作成

### エスカレーション
- **プライマリ:** platform-team
- **セカンダリ:** infrastructure-team
- **マネージャー:** platform-manager

### 関連ランブック
- kubernetes-troubleshooting
- deployment-best-practices

---

## データベースPod CrashLoopBackOffインシデント

**プレイブックID:** `database-pod-crashloop-incident`  
**インシデントタイプ:** 可用性  
**深刻度:** クリティカル  
**推定解決時間:** 10-15分  
**特定Pod:** `database-pod-7b9c4d8f2a-x5m1q`

### 説明
データベースPodが継続的にクラッシュするクリティカルインシデント対応

### 根本原因
PostgreSQL初期化を妨げるConfigMap 'database-config'の欠落

### トリガー
- CrashLoopBackOff状態のデータベースPod
- ConfigMap 'database-config'が見つからないエラー
- PostgreSQL初期化失敗
- ボリュームマウント権限エラー

### 対応ステップ
1. **即座に:** Podの状態を確認: `kubectl get pod database-pod-7b9c4d8f2a-x5m1q -n production`
2. Podログを確認: `kubectl logs database-pod-7b9c4d8f2a-x5m1q --previous -n production`
3. ConfigMapの存在を検証: `kubectl get configmap database-config -n production`
4. ConfigMapが欠落している場合は作成:
   ```bash
   kubectl create configmap database-config \
     --from-literal=database.conf='shared_buffers=256MB\nmax_connections=100\nlog_destination=stderr' \
     -n production
   ```
5. ボリューム権限を確認: `kubectl exec -it database-pod-7b9c4d8f2a-x5m1q -- ls -la /var/lib/postgresql/`
6. Podを強制再起動: `kubectl delete pod database-pod-7b9c4d8f2a-x5m1q -n production`
7. Pod起動を監視: `kubectl logs database-pod-7b9c4d8f2a-x5m1q -f -n production`
8. 実行後にデータベース接続を検証

### 影響評価
- **影響を受けるサービス:** web-service, api-service
- **影響を受けるユーザー:** すべてのユーザー - 完全なデータベース停止
- **ビジネス影響:** クリティカル - データ操作が不可能

### エスカレーション
- **プライマリ:** database-oncall@company.com
- **セカンダリ:** platform-oncall@company.com
- **マネージャー:** incident-manager@company.com
- **エスカレーション時間:** 5分

### 関連ランブック
- database-crashloop-troubleshooting
- configmap-management

### インシデント後のアクション
- デプロイメントマニフェストにConfigMapを追加
- CI/CDでConfigMap検証を実装
- ConfigMap存在の監視を追加
- 設定要件を文書化