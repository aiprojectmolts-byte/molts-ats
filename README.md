# THE MOLTS 採用管理 ATS

単一 HTML ファイルで動く、採用プロセス（スカウト送付 → 面談 → オファー → 入社）の一元管理ツール。
データは Google Drive（ScoutData/ats/ 配下）に保存され、Drive 未接続時は localStorage にフォールバックします。

## 構成

- `index.html` — アプリ本体（CSS / JS すべて含む単一ファイル）
- `config.json` — ブランド名・OAuth clientId・プラットフォーム定義・ロール・スコープなどの設定

## ローカル起動

Google OAuth は `file://` では動かないため、HTTP サーバー経由で開く必要があります。

```sh
cd scout/ats
python3 -m http.server 8080
# → http://localhost:8080/ をブラウザで開く
```

初回ログイン時は Google の同意画面で以下のスコープを付与します:

- `openid email profile` — ユーザー識別
- `drive.file` / `drive.appdata` — ScoutData/ats/ 配下の読み書き
- `calendar.events` — 採用フローの面談をカレンダーに登録（任意）

Gmail 連携は ATS ログインとは独立した OAuth フローで別トークンを発行します（設定 → Gmail連携 → +Gmailアカウントを追加）。

## 機能概要

- 候補者一覧／詳細、検索・フィルタ（期日超過 / 要対応 / 自分の担当）
- 新規候補者追加ウィザード（4ソース対応）
- Claude API によるスカウト生成・プロフィール要約・スコープ判定・やり取りログ要約
- やり取りログ（ラベル付き、返信全文収納、編集可）
- 採用フロー（8フェーズ、複数面接官、Google Calendar 連携、Meet 自動生成）
- 媒体URLの複数登録・メイン切替・プロフィール更新モーダル
- Gmail ポーリングによる候補者への自動紐付け（YOUTRUST / Wantedly / LinkedIn / TimeRex）
- Slack 通知（面談確定・リマインド・返信あり）／日次・週次サマリー
- タスク管理（自動生成・自動完了・営業日期限）
- ボール状態（最新ログのラベルから判定）
- RBAC（管理者 / メンバー / 面接官 / 閲覧者）、承認待ちフロー
- 操作ログ（誰がいつ何をしたか）

## 開発運用ルール

### ブランチ運用

- **大きな機能追加の前にブランチを切る**
  例: `git checkout -b feature/gmail-multi-account`
- **動作確認できたら main にマージ**
  例: `git checkout main && git merge --no-ff feature/xxx`
- **最低でも機能追加のたびにコミットする**
  中断中の作業でも小さく記録して意図を残す

### コミットメッセージ

1行目に何をしたかを簡潔に。
補足があれば空行を挟んで本文を書く。

```
候補者詳細に操作ログを追加

- pushActivity(c, type, detail) で操作ごとに記録
- detail 画面右下にタイムライン表示
```

### 動作確認チェックリスト

- [ ] `python3 -m http.server 8080` で起動 → http://localhost:8080/ でログイン
- [ ] 候補者を1件追加 → Drive の ScoutData/ats/ に反映
- [ ] ブラウザ DevTools の Console にエラーが出ていない
- [ ] 複数アカウントで動作させる場合はシークレットウィンドウなどで別ユーザーとしてもログイン確認

### Drive 関連の注意

- `drive.file` スコープはアプリが作成・オープンしたファイルのみアクセス可能。
  既存の ScoutData フォルダを触ったことがない場合、アプリが同名で別フォルダを新規作成することがあります。
- トークンはセッションごとに取り直されるため、ブラウザを閉じると再ログインが必要です。
