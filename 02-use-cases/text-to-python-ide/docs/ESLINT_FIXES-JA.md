# ESLintエラー修正

## 一般的なESLintエラーと解決策

### 1. `'ComponentName' is not defined` (react/jsx-no-undef)

**エラー例:**
```
ERROR
[eslint] 
src/components/ExecutionResults.js
  Line 67:20:  'Badge' is not defined  react/jsx-no-undef
```

**解決策:**
インポート文に不足しているコンポーネントを追加:

```javascript
// 修正前（Badgeが不足）
import {
  Container,
  Header,
  SpaceBetween,
  Box,
  Button
} from '@cloudscape-design/components';

// 修正後（Badgeを追加）
import {
  Container,
  Header,
  SpaceBetween,
  Box,
  Button,
  Badge
} from '@cloudscape-design/components';
```

### 2. 未使用変数警告

**エラー例:**
```
WARNING
[eslint] 
src/components/CodeDisplay.js
  Line 2:35:  'SpaceBetween' is defined but never used  no-unused-vars
```

**解決策:**
未使用のインポートを削除:

```javascript
// 修正前（SpaceBetweenが未使用）
import { Box, Button, SpaceBetween } from '@cloudscape-design/components';

// 修正後（SpaceBetweenを削除）
import { Box, Button } from '@cloudscape-design/components';
```

### 3. コンソール文警告

**エラー例:**
```
WARNING
[eslint] 
src/App.js
  Line 45:7:  Unexpected console statement  no-console
```

**解決策:**
プロダクション用にコンソール文を削除するか、eslint-disableを使用:

```javascript
// オプション1: コンソール文を削除
// console.log('Debug info'); // これを削除

// オプション2: 特定行でeslintを無効化
// eslint-disable-next-line no-console
console.log('Debug info');

// オプション3: プロダクションで適切なログを使用
if (process.env.NODE_ENV === 'development') {
  console.log('Debug info');
}
```

## 一般的なCloudscapeコンポーネント

インポートが必要な最も一般的に使用されるCloudscapeコンポーネント:

```javascript
import {
  // レイアウト
  AppLayout,
  ContentLayout,
  Container,
  Header,
  Box,
  SpaceBetween,
  ColumnLayout,
  
  // フォームコントロール
  Button,
  Input,
  Textarea,
  FormField,
  FileUpload,
  Select,
  
  // フィードバック
  Alert,
  Spinner,
  StatusIndicator,
  Badge,
  
  // ナビゲーション
  Tabs,
  Modal,
  Link,
  
  // データ表示
  Table,
  CodeEditor
} from '@cloudscape-design/components';
```

## クイック修正コマンド

### ESLintチェックを実行
```bash
cd frontend
npm run lint
```

### ESLint問題を自動修正
```bash
cd frontend
npx eslint src/ --fix
```

### 特定ファイルをチェック
```bash
cd frontend
npx eslint src/components/ExecutionResults.js
```

## 予防のヒント

1. **IDE拡張機能を使用**: IDEにESLint拡張機能をインストール
2. **プリコミットフック**: コミット前にESLintが実行されるよう設定
3. **定期チェック**: 開発中に定期的に`npm run lint`を実行
4. **インポート整理**: インポートを整理し、未使用のものを削除

## フロントエンドテスト

フロントエンドテストスクリプトを実行して一般的な問題をチェック:

```bash
node test_frontend.js
```

これは以下をチェックします:
- 不足しているファイル
- 不足している依存関係
- インポート問題
- 非推奨コンポーネントの使用