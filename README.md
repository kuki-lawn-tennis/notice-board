# GitHub Pages / Google Apps Script の分離

## 配置

- `index.html` は GitHub Pages の公開ルートに配置する。
- `gas/Code.js` の内容をGASのスクリプトファイル（通常は `Code.gs`）、`gas/ApiBridge.html` をHTMLファイル `ApiBridge.html` として配置する。
- スプレッドシート、Drive、スクリプトプロパティなど、現在のGASプロジェクト設定は引き継ぐ。

GitHub Pages とGASの間は、GASのHTML Serviceを隠しiframeとして読み込み、`postMessage`でRPCを行う。
GAS側で `google.script.run` を呼び出すため、ブラウザーからGASへ直接 `fetch` する際のCORS制約を避けられる。ブリッジは許可したメソッドだけを実行し、親ページのオリジンも照合する。

## GASの設定とデプロイ

1. Apps Script の「プロジェクトの設定」→「スクリプト プロパティ」に `GITHUB_PAGES_ORIGINS` を追加します。値にはページのオリジンだけを指定する（例: `https://kuki-lawn-tennis.github.io`）。独自ドメインやローカル確認用URLがある場合はカンマ区切りで追加できる。パスや末尾の `/` は含めない。
2. `gas/Code.js` の内容をGASのスクリプトファイル（通常は `Code.gs`）、`gas/ApiBridge.html` をHTMLファイル `ApiBridge.html` に反映する。ウェブアプリとして新しいバージョンをデプロイし、実行ユーザーを自分、アクセスできるユーザーを公開サイトの利用者に設定する。管理操作は既存の管理者セッショントークンで保護される。
3. デプロイされた `/exec` URLをコピーする。

## コート当番フォルダの設定

1. Apps Script のスクリプトプロパティ `COURT_DUTY_FOLDER_ID` に、保存先DriveフォルダのID（またはフォルダURL）を設定する。IDはフォルダURLの `/folders/` の後ろの値である。
2. ウェブアプリの実行ユーザー（上記設定ならデプロイした自分のGoogleアカウント）に、そのフォルダへのアクセス権があることを確認する。共有ドライブの場合も同じアカウントがメンバーである必要がある。
3. Apps Script エディターで `verifyCourtDutyFolderAccess` を選んで実行し、初回のGoogle Drive権限確認を完了する。成功すれば実行ログに確認メッセージが表示される。
4. ウェブアプリを新しいバージョンにデプロイする。

## お知らせ・イベント・スケジュールのフォルダ

- Apps Script のスクリプトプロパティに、別々のDriveフォルダを指定する：`NOTICE_ATTACHMENTS_FOLDER_ID`（お知らせ）、`EVENT_ATTACHMENTS_FOLDER_ID`（イベント）、`SCHEDULE_ATTACHMENTS_FOLDER_ID`（スケジュール）。

## GitHub Pages の設定

1. リポジトリの **Settings → Secrets and variables → Actions → Variables** で、`GAS_WEB_APP_URL` というRepository variableを作成し、手順2の `/exec` URLを設定します。これは秘密情報ではないため、SecretではなくVariableを使う。
2. **Settings → Pages → Build and deployment → Source** を **GitHub Actions** に設定する。
