# GHLM (GitHub Life Manager)

GitHub の Issue をターミナル上で閲覧・作成・編集するための TUI ツールです。
Rust と [Ratatui](https://ratatui.rs/) で作られており、キーボードだけで操作できます。
本文の編集には、普段使っているエディタ (`$EDITOR`) をそのまま使えます。

## 特徴

- **起動してすぐ自分のタスクが見える**: 自分にアサインされたオープンな Issue を全リポジトリから横断表示
- **リポジトリ単位の表示**: アクセス可能なリポジトリ（Public / Private）から選んで、その Issue 一覧に切り替え
- **前回の続きから**: 最後に開いたリポジトリを記憶し、次回起動時に自動で開く
- **好きなエディタで編集**: Issue 本文の編集・作成時に `$EDITOR` を起動し、保存すると GitHub に反映
- **認証の設定が不要**: GitHub CLI (`gh`) のログイン情報をそのまま利用

## 必要なもの

| ツール | 用途 |
| --- | --- |
| [Rust](https://www.rust-lang.org/tools/install) (stable) | ソースからビルドする場合 |
| [GitHub CLI (`gh`)](https://cli.github.com/) | 認証トークンの取得 |

GHLM は起動時に `gh auth token` を実行してトークンを取得します。事前にログインしておいてください。

```bash
gh auth login
```

## インストール

### ソースからビルド

```bash
git clone https://github.com/ki-mutaku/ghlm.git
cd ghlm
cargo install --path .
```

### リリースバイナリ

`v*` 形式のタグが push されると、GitHub Actions が Linux / macOS / Windows 向けのバイナリをビルドし、
[Releases](https://github.com/ki-mutaku/ghlm/releases) ページに添付します。

## 使い方

```bash
ghlm
```

起動時の挙動は次のとおりです。

1. 前回開いたリポジトリがあれば、そのリポジトリのオープンな Issue を表示
2. なければ（または取得に失敗したら）、自分にアサインされたオープンな Issue を表示

### 画面構成

```
┌────────────────────────────────────────────────────┐
│ ヘッダー（リポジトリ名など）                       │
├───────────────┬────────────────────────────────────┤
│ Issue 一覧     │ 選択中の Issue の詳細              │
│ (30%)         │ (70%)                              │
│               │                                    │
├───────────────┴────────────────────────────────────┤
│ ステータスバー（操作ヘルプ）                       │
└────────────────────────────────────────────────────┘
```

### キー操作

#### Issue 一覧

| キー | 動作 |
| --- | --- |
| `j` / `↓` | 下に移動 |
| `k` / `↑` | 上に移動 |
| `e` | 選択中の Issue 本文を外部エディタで編集（保存すると GitHub に反映） |
| `n` | 新規 Issue 作成（先にリポジトリを選択しておく必要があります） |
| `r` | リポジトリ選択画面を開く |
| `q` | 終了 |

#### リポジトリ選択

| キー | 動作 |
| --- | --- |
| `j` / `↓`, `k` / `↑` | 移動 |
| `Enter` | 選択したリポジトリの Issue 一覧を開く |
| `Esc` | 戻る |

#### 新規 Issue 作成

1. タイトルを入力して `Enter`（タイトルは必須）
2. 下書き画面で `e` を押すと外部エディタで本文を書ける
3. `Enter` で GitHub に Issue を作成、`Esc` でキャンセル

エラーメッセージが表示されたときは、任意のキーで閉じられます。

### 外部エディタ

環境変数 `EDITOR` に設定されたエディタが使われます。未設定の場合は `vi` が起動します。

```bash
export EDITOR=nvim
```

エディタを保存して終了すると内容が反映されます。エディタが異常終了した場合（例: Vim の `:cq`）は変更を破棄します。

## 設定ファイル

最後に開いたリポジトリは JSON で保存されます。保存先は OS ごとに異なります。

| OS | パス |
| --- | --- |
| macOS | `~/Library/Application Support/com.ghlm.ghlm/config.json` |
| Linux | `~/.config/ghlm/config.json` |
| Windows | `%APPDATA%\ghlm\ghlm\config\config.json` |

設定ファイルが壊れている場合は警告を表示し、デフォルト設定で起動します。

## 開発

```bash
cargo build          # ビルド
cargo run            # 実行
cargo test           # テスト
cargo clippy         # 静的解析
cargo fmt            # フォーマット
```

### ディレクトリ構成

```
src/
├── main.rs    # エントリポイント・イベントループ・キー入力の処理
├── app.rs     # アプリケーションの状態 (App, Screen)
├── ui.rs      # Ratatui による描画
├── gh.rs      # GitHub API 通信・トークン取得・外部エディタ連携
├── config.rs  # 設定ファイルの読み書き
└── models.rs  # データモデル (Repository など)
```

### 関連ドキュメント

- [spec.md](spec.md) — 仕様書
- [TESTING.md](TESTING.md) — テスト方針とテストケース一覧
- [.github/README.md](.github/README.md) — CI（自動コードレビュー）の説明

## 今後の予定

- ラベル・マイルストーンによる Issue の絞り込み
- バックグラウンドでの定期同期
- ステータスラインへのポモドーロタイマー表示
- ローカルキャッシュによるオフライン閲覧
