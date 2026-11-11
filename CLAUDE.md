# CLAUDE.md

konbu の開発パートナーとしての前提・ルール。

## プロジェクト概要

- Rust製の軽量な TUI メモ／日記アプリ。短めの文章（軽い日記、TODO程度）向け。長いドキュメント管理は対象外
- 自分用ミニアプリ兼ポートフォリオ。commit 履歴・Issue・PR の経過も成果物として残す
- 開発環境: WSL2 / Neovim（rust-analyzer、保存時に rustfmt、check は clippy）

## 役割分担（最重要）

- **backend**（データ構造、frontmatter、保存・読み込み、一覧、日付の扱い、CLI、`open_in_editor`、それらのテスト）: オーナーが書く（Rust の練習）
  - Claude は実装コードを書かない。ヒント、設計相談、参考ドキュメント、エラーの読み方の説明までにとどめる
- **view**（ratatui による TUI、キー操作、TUI の一時停止と復帰、`MockStore`）: Claude とオーナーの協働
- **chore / documentation**: オーナーの依頼があれば Claude が書いてよい
- backend と view の境界は `Store` trait。view から直接ファイルを操作しない
- Claude が書いたコード・ファイルの commit には `Co-Authored-By` を付け、オーナーの成果と区別する
- 読みやすさ優先。複雑な書き方を避け、要所にコメントを残す
- PR レビューは Claude が問題点を指摘し、修正はオーナー自身が行う。backend では修正コードを提示しない

## データ仕様

- データディレクトリ: `$KONBU_DIR`、未設定なら `~/konbu`
  - 日記: `daily/YYYY-MM-DD.md`（1日1ファイル。日付はファイル名が正。過去日も作成可）
  - メモ: `memo/<タイトル>.md`（タイトル = ファイル名。使えない文字は `_` に置換、同名はエラー）
- frontmatter（日記・メモ共通、自前パース、`---` で挟んだ `key: value`）
  - `id`: 作成日時のタイムスタンプ（例 `20261109235725`）。MVP では書き込むだけ。重複時は作成エラー
  - `type`: `daily` | `memo`。情報としてのみ保持し、種類の判定はディレクトリで行う
  - `created`: 作成日（例 `2026-11-09`）。「その日のメモ」の判定に使う。ファイルの作成日時は使わない
- frontmatter が無い／壊れたファイルも一覧に出す（`created` 不明扱い）
- TODO は本文中のチェックボックス（`- [ ]` / `- [x]`）
- 「今日」はローカル時刻 0 時で切り替え
- 用語は `daily` / `memo` / `NoteKind::Daily | Memo` に統一する
- 同期はデータディレクトリを別の非公開 git リポジトリにして運用。アプリ機能としては MVP 外

### 実データの扱い

- **日記・メモのデータを絶対にこのリポジトリへ commit しない**
- 手動の動作確認は `KONBU_DIR=./sandbox cargo run`（`/sandbox` は `.gitignore` 済み）
- テストは `tempfile` の一時ディレクトリを使う
- 公開リポジトリなので、Issue・PR にも実データを貼らない

## 構成

- lib + bin 構成。backend は lib 側、`main.rs` は起動処理のみ
- モジュールは必要になった Issue で `lib.rs` に追加する（空のモジュールを先に作らない）
- エラー: backend は自前 enum（または `thiserror`）、`main` と view は `anyhow`
- 日付 crate（chrono / jiff）は該当 Issue 内で比較して選び、結果を Issue コメントに残す
- rustfmt / clippy はデフォルト設定（設定ファイルは作らない）

## UI 仕様

- `konbu`: TUI 起動 / `konbu today`: 今日の日記を `$EDITOR` で開く
- 左: カレンダー + 一覧（「その日」「全メモ（作成日の新しい順）」の 2 タブ、その日の日記を先頭に固定）/ 右: プレビュー（読み取り専用）
- Enter で TUI を一時停止 → `$EDITOR` を全画面起動 → 終了後 TUI に復帰。アプリ内エディタは作らない
- キー操作は vim 風（`hjkl` / `Enter` / `n` / `Tab` / `q`）、矢印キーも可

## MVP の範囲

- 含む: 日記を開く／作る、一覧、開く、カレンダーのサイドバー、その日のメモ表示、全メモ一覧、メモの新規作成
- 含まない（Backlog）: 検索、日記へのリンク追記、TOML 設定、同期補助、`cargo install` 配布、日付切り替え時刻の設定、削除・リネーム、ID によるリンク

## 進め方

### Git / PR

- main へ直接 push しない。すべての変更は PR を通して反映する
- 1 Issue = 1 ブランチ = 1 PR（ブランチ名: `<type>/<Issue番号>-<内容>`、例 `feat/3-note-type`）
- commit メッセージ: `<type>: <なぜ変えたかを日本語で> (#Issue番号)`。type は `feat` / `fix` / `docs` / `style` / `refactor` / `test` / `chore`
- PR は Draft で作り、CI が緑になってから Ready for review にする。Issue の完了条件は PR に写さず Issue 側でチェックする
- merge は merge commit で行う（Squash merge は使わない。commit ごとの `Co-Authored-By` の区別を残すため）
- CI（`.github/workflows/ci.yml`）: `cargo fmt --all --check` / `cargo clippy --all-targets -- -D warnings` / `cargo test`。Claude がコードを書いたら完了報告の前に手元で同じコマンドを実行する
- 外部への操作（Issue 作成、push、PR 作成、リポジトリ設定の変更など）は事前にオーナーの確認を取る

### Issue / Projects

- 説明、Issue、PR の本文は日本語
- 進捗は GitHub Projects（user 所有、リポジトリにひも付け）。Status: `Backlog / Todo / In Progress / Done`
- マイルストーンは `v0.1.0 MVP` のみ。ラベル: `backend` / `view` / `chore` / `documentation` / `enhancement` / `bug`
- テンプレート: `.github/ISSUE_TEMPLATE/`（backend / view / chore / documentation / bug）、`.github/pull_request_template.md`
- MVP の残り項目と Backlog は Projects の Draft item（タイトルと1行の要点のみ）として置く
- Issue 化するときは Draft item を変換せず、テンプレートから Issue を新しく作り、Draft item を削除する（変換ではテンプレートもラベルも適用されないため）。Projects への追加は Auto-add workflow に任せ、マイルストーンは手動で付ける
- `gh` で Issue を作る場合は YAML テンプレートが適用されないので、テンプレートと同じ見出しで本文を書く
- backend の Issue には実装ではなく学習ポイントと参考資料を書く
- 参考資料は Rust book 日本語版（https://doc.rust-jp.rs/book-ja/ ）を主に、不足分は docs.rs / std / ratatui 公式で補う。リンクは実在を確認してから載せる

### その他

- 作業時間の長さを理由に区切りを提案しない
