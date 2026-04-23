# THE MOLTS 採用管理 ATS

## 起動方法

```sh
cd ~/Documents/scout/ats
python3 -m http.server 8080
```

→ http://localhost:8080 をブラウザで開く

## ファイル構成

- `index.html` … メインアプリ（単一ファイル）
- `config.json` … 設定ファイル
- `docs/` … 各種ドキュメント

## ブランチ運用ルール

- `main` … 動作確認済みの安定版
- `feature/xxx` … 機能追加時はブランチを切る

## 機能追加時の手順

```sh
git checkout -b feature/機能名
# 実装・動作確認
git add -A
git commit -m "機能名：説明"
git checkout main
git merge feature/機能名
```
