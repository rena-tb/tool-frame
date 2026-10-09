# private_tool_frame（GAS WebApp の埋め込み用ページ）

仕様変更は都度、本CLAUDE.md（現在の仕様のみ）とNotion（ツール一覧DB、現状スナップショットのみ）に反映すること。変更の経緯・履歴はgit commitに残し、CLAUDE.md/Notionには書かない。

## 目的と想定利用者

- 目的: GAS WebAppのページ（iPhone中継・プランナーの編集ページ等）を画面いっぱいの枠（iframe）の中で開くだけのページ。`script.google.com` を直接開くと、Googleの「このアプリケーションは Google Apps Script のユーザーによって作成されたものです」のバナーが毎回出るため、このページ経由で開く（枠の中で開くとバナーは出ない）
- 想定利用者: 玲奈本人のみ

## 仕様

- `index.html` 1枚だけの静的ページ。GitHub Pages（rena-tb アカウントの公開リポジトリ `tool-frame`）で配信する
- 開くページはURLの `#` 以降に書く（`index.html#<GAS WebAppのURL>`）。`#` 以降はブラウザの外に送られないため、GASの鍵が配信元（GitHub）に渡ることはない。このリポジトリには鍵・URL・データを一切置かない（公開リポジトリのため）
- 開けるのは `https://script.google.com/macros/s/<ID>/exec` と `https://script.google.com/a/macros/<ドメイン>/s/<ID>/exec` だけ（任意のサイトを枠で開く踏み台にされないため）
- 枠の中で開けるよう、埋め込み先のGAS側は `doGet` で `setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL)` にしている（`~/private_mobile_bridge`・`~/work_mobile_bridge`・`~/private_schedule_planner`）
- タブの名前は、URLのツールID（`t=`）から決める（枠の中のページの題名は読めないため）
- このページ経由のURLは、各ツールの `python3 bridge.py --print-urls`（中継）・`python3 server.py --print-viewer-urls`（プランナー）で表示する

## 権限モデル・データ

- ページ自体は誰でも開けるが、中身は空の枠だけ。表示する内容と操作の可否は、埋め込み先のGASページの鍵で決まる
- 扱うデータ: なし（鍵はURLの `#` 以降にだけ現れ、このページは保存・送信しない）

## 技術スタック・デプロイとロールバック

- 静的HTML（JSは開くURLの検証と枠の作成だけ）。GitHub Pages（`main`ブランチのルート）
- デプロイ: `git push`（GitHub Pagesが自動で配信し直す）。ロールバック: 前のコミットに戻してpush
- 検証: ブラウザで `index.html#<GAS WebAppのURL>` を開き、バナーが出ずにツールが表示されること。許可していないURL（例: `#https://example.com/`）では「開けないURLです」と出ること
