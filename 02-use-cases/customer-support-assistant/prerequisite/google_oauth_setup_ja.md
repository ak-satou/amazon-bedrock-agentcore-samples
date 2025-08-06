# 🔐 エージェント用Google OAuth2クレデンシャルの設定
このガイドでは、Google Cloudプロジェクトを作成し、Google DriveなどのサービスとのintegrationのためにOAuth 2.0クレデンシャルを設定する手順を説明します。

## ✅ 1. Google Developer ConsoleでプロジェクトをUpdate
1. [Google Developer Console](https://console.developers.google.com/)にアクセスします。
2. 上部のナビゲーションバーで「Create Project」をクリックします。
3. プロジェクト名を入力します。
4. 組織を選択するか、該当しない場合は「No organization」のままにします。
5. 作成をクリックします。

新しいプロジェクトがプロジェクトリストに表示されます。

## 📦 2. Google Calendar APIを有効化

1. プロジェクトが選択された状態で、左側のメニューを開き、APIs & Services > Libraryに移動します。
2. 検索バーで「Google Calendar API」と入力します。
3. 結果からGoogle Calendar APIをクリックします。
4. 有効化をクリックします。

## 🛡️ 3. OAuth同意画面の設定

1. 左側のメニューで、APIs & Services > OAuth consent screenに移動します。

2. 「Get started」をクリックします。

3. 必須フィールドを入力します：アプリ名、ユーザーサポートメール
4. Next をクリックし、その後：ユーザータイプ（Internal または External）を選択します。External を選択した場合は、テスターのメールアドレスを追加します。開発者連絡先情報（あなたのメール）を提供します。
5. 規約に同意し、Finishをクリックします。
6. Createをクリックして同意画面を確定します。

## 🔧 4. OAuth 2.0クレデンシャルの作成

1. 左側のメニューからAPIs & Services > Credentialsに移動します。
2. Create Credentials > OAuth client IDをクリックします。
3. アプリケーションタイプとして「Web application」を選択します。
4. クレデンシャルの名前を入力します。
5. Authorized redirect URIsで、以下のリダイレクトURIを追加します：
   - `https://bedrock-agentcore.us-east-1.amazonaws.com/identities/oauth2/callback`
6. Createをクリックします。

## 🔑 5. Client IDとClient Secretの取得

作成後、ダイアログにClient IDとClient Secretが表示されます。JSONをダウンロードしてクレデンシャルをファイルに保存します。このファイルを`credentials.json`としてプロジェクトに保存します。

## 🔍 6. データアクセススコープの更新

1. APIs & Services > Credentialsに移動します。
2. 作成したOAuth 2.0 client IDをクリックします。
3. 左側のメニューで、Data accessを選択します。
4. 「Add or remove scopes」をクリックします。
5. Manually add scopesで、スコープ：`https://www.googleapis.com/auth/calendar`を入力します。
6. Updateをクリックし、Saveをクリックして設定を確認します。