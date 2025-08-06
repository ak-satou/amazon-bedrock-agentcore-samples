# トラブルシューティングガイド

このドキュメントには、一般的な運用問題に対する詳細なトラブルシューティングガイドが含まれています。

## Pod CrashLoopBackOffトラブルシューティング

**ガイドID:** `pod-crashloop-troubleshooting`  
**カテゴリ:** Kubernetes

### トラブルシューティングステップ
1. Podの詳細を取得: `kubectl describe pod <pod-name>`
2. Podログを確認: `kubectl logs <pod-name> --previous`
3. 最近のイベントを確認: `kubectl get events --sort-by=.metadata.creationTimestamp`
4. リソース制限とリクエストを検証
5. livenessとreadinessプローブを確認
6. コンテナイメージと設定を確認
7. 環境変数とシークレットを検証

### 一般的な原因
- リソース不足（CPU/メモリ）
- 不正確なlivenessprobe設定
- 環境変数の欠落
- 無効なコンテナイメージまたはタグ
- デプロイメントの設定エラー

### 診断コマンド
- `kubectl get pod <pod-name> -o yaml`
- `kubectl logs <pod-name> --previous`
- `kubectl describe pod <pod-name>`
- `kubectl get events --field-selector involvedObject.name=<pod-name>`

---

## データベースPod CrashLoopBackOff解決

**ガイドID:** `database-crashloop-troubleshooting`  
**カテゴリ:** Kubernetes  
**特定Pod:** `database-pod-7b9c4d8f2a-x5m1q`

### トラブルシューティングステップ
1. Podログを確認: `kubectl logs database-pod-7b9c4d8f2a-x5m1q --previous`
2. ConfigMapが存在することを検証: `kubectl get configmap database-config -n production`
3. ボリュームマウントを確認: `kubectl describe pod database-pod-7b9c4d8f2a-x5m1q`
4. PVC状態を検証: `kubectl get pvc -n production`
5. データディレクトリの権限を確認
6. 必要に応じて欠落しているConfigMapを作成: `kubectl create configmap database-config --from-file=database.conf`
7. ボリューム権限を修正: `chmod 700 /var/lib/postgresql/data && chown postgres:postgres /var/lib/postgresql/data`

### 一般的な原因
- ConfigMap 'database-config'の欠落
- データディレクトリの不正なファイル権限
- ボリュームマウント失敗
- データベース設定ファイルの欠落
- PostgreSQL初期化失敗

### 診断コマンド
- `kubectl logs database-pod-7b9c4d8f2a-x5m1q -c postgres --previous`
- `kubectl describe pod database-pod-7b9c4d8f2a-x5m1q`
- `kubectl get configmap -n production | grep database`
- `kubectl get pvc -n production`
- `kubectl get events --field-selector involvedObject.name=database-pod-7b9c4d8f2a-x5m1q --sort-by='.lastTimestamp'`

### 解決

#### 即座の修正
```bash
kubectl create configmap database-config \
  --from-literal=database.conf='shared_buffers=256MB\nmax_connections=100' \
  -n production
```

#### 恒久的な修正
適切なConfigMapとボリューム権限を含むようにデプロイメントマニフェストを更新

**影響:** クリティカル - すべてのサービスに影響する完全なデータベース停止  
**推定解決時間:** 10-15分

---

## 高レスポンス時間調査

**ガイドID:** `high-response-time-troubleshooting`  
**カテゴリ:** パフォーマンス

### 調査ステップ
1. 現在のレスポンス時間メトリクスを確認
2. 影響を受けるエンドポイントとサービスを特定
3. CPUとメモリ使用率を確認
4. データベースクエリパフォーマンスを調査
5. ネットワーク遅延問題を確認
6. ボトルネックのアプリケーションログを確認
7. 外部サービス依存関係を検証

### ツール
- `kubectl top pods`
- Application Performance Monitoring（APM）
- データベースクエリ分析ツール
- ネットワーク監視ツール

### 一般的な原因
- データベースクエリ最適化が必要
- サービスリソース不足
- ネットワーク遅延またはパケット損失
- 外部サービス劣化
- キャッシュミスまたは無効化

---

## メモリリーク調査ガイド

**ガイドID:** `memory-leak-investigation`  
**カテゴリ:** パフォーマンス

### 調査ステップ
1. 時間経過とともにメモリ使用傾向を監視
2. メモリ使用量が増加しているサービスを特定
3. 可能であればヒープダンプをキャプチャ
4. 最近のコード変更を確認
5. 閉じられていないリソースを確認
6. オブジェクト割り当てパターンを分析
7. ステージング環境で修正をテスト

### 診断コマンド
- `kubectl top pods --containers`
- `kubectl exec <pod> -- jmap -heap <pid>`
- `kubectl exec <pod> -- jstat -gcutil <pid>`

### 予防措置
- 適切なリソースクリーンアップを実装
- 接続プーリングを使用
- 適切なJVMヒープ設定を設定
- メモリメトリクスを継続的に監視

---

## サービス発見トラブルシューティング

**ガイドID:** `service-discovery-issues`  
**カテゴリ:** ネットワーキング

### トラブルシューティングステップ
1. サービスエンドポイントを検証: `kubectl get endpoints`
2. サービスセレクターラベルを確認
3. PodからのDNS解決をテスト
4. ネットワークポリシーを検証
5. サービスポート設定を確認
6. Pod間の接続をテスト
7. ingress設定を確認

### 診断コマンド
- `kubectl get svc <service-name> -o yaml`
- `kubectl get endpoints <service-name>`
- `kubectl exec <pod> -- nslookup <service-name>`
- `kubectl exec <pod> -- curl <service-name>:<port>`

### 一般的な問題
- セレクターラベルの不一致
- 不正確なポート設定
- ネットワークポリシー制限
- DNS設定問題