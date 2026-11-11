# Projects の Draft item 一覧

GitHub Projects に登録する Draft item の一覧。並びは実装順のめやす。
Issue 化するときは Draft item を変換せず、テンプレートから Issue を新しく作り、Draft item を削除する。

## MVP（Status: `Todo`）

| タイトル | 要点 |
|---|---|
| [backend] 設定：データディレクトリ | `KONBU_DIR`、未設定なら `~/konbu`。`daily/` と `memo/` がなければ作成 |
| [backend] NoteKind と Note 型、ID の生成 | `NoteKind::Daily / Memo` と `Note` の定義、作成日時からタイムスタンプ ID を作る |
| [backend] Store trait の定義 | backend と view の境界。view が必要とする操作を trait にまとめる |
| [view] TUI の基盤と MockStore | ratatui＋crossterm、イベントループ、`q` で終了。固定データを返す `MockStore` |
| [backend] frontmatter の読み書き | `id` / `type` / `created` の自前パースと書き出し |
| [backend] daily：指定日の日記のパス | 指定した日の日記のパスを返し、なければ frontmatter と見出し付きで作成 |
| [backend] daily：日記の一覧 | `daily/` の日記を日付順に返す |
| [backend] memo：新規作成 | タイトルからファイル名を作り（`_` 置換、同名はエラー）、frontmatter を付けて作成 |
| [backend] memo：指定日のメモ一覧 | `created` が指定した日のメモを返す |
| [backend] memo：全メモ一覧 | 全メモを作成日の新しい順に返す（`created` 不明のものも含む） |
| [backend] 本文の読み込み | 1件の本文を frontmatter を除いて返す |
| [backend] CLI：konbu today と open_in_editor | 引数なしで TUI、`today` で今日の日記を `$EDITOR` で開く |
| [view] カレンダーのサイドバー | 日付の移動（`hjkl`）と、日記がある日の目印 |
| [view] 一覧とプレビュー | 「その日」「全メモ」の2タブ、日記を先頭に固定、右にプレビュー |
| [view] エディタの起動と復帰 | Enter で TUI を一時停止して `$EDITOR` を開き、終了後に復帰 |
| [view] MockStore から本実装への切り替え | Markdown の実装を使うようにする（backend の一覧と読み込みが終わってから） |
| [view] TUI からのメモ新規作成 | `n` でタイトルを入力してメモを作成 |
| [docs] v0.1.0 のリリース | README の使い方、スクリーンショット、タグ付け |

## Backlog（Status: `Backlog`）

| タイトル | 要点 |
|---|---|
| 全文検索 | 日記とメモの本文を検索する |
| 日記へのリンク追記 | メモ作成時に、その日の日記へリンクを追記するオプション |
| 設定ファイル（TOML） | 環境変数に加えて設定ファイルでデータディレクトリなどを指定する |
| 同期の補助コマンド | データディレクトリの git の commit や push を補助する |
| cargo install での配布 | crates.io への公開と、インストール手順の整備 |
| 日付の切り替え時刻 | 「朝4時までは前日」のような切り替え時刻を設定可能にする |
| 削除とリネーム | TUI からメモを削除・リネームする（確認ダイアログ付き） |
| ID によるリンク | frontmatter の `id` を使ってノート間をリンクする |
