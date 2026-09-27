# rust-learning

`NEXT_PLAN.md`(`../frontend_passkey-go_bff-backend-multi/NEXT_PLAN.md`)の Rust ライン(Phase 0〜4)を進めるリポジトリ

## 目次

- [Phase 一覧](#phase-一覧)
- [構成についての方針](#構成についての方針)
- [セットアップ](#セットアップ)
- [Phase 0:Rust の基礎の進め方](#phase-0rust-の基礎の進め方)
- [Phase 共通の進め方](#phase-共通の進め方)
- [トラブルシューティング](#トラブルシューティング)

## Phase 一覧

| ディレクトリ | Phase | 種類 | 完了条件 | 状態 |
|---|---|---|---|---|
| `phase_0_basics` | 0 Rust の基礎 | 入れ物のディレクトリ | rustlings を全問解く / 練習用プロジェクトの課題を終える / 借用エラーを自分で直せる | 未着手 |
| `phase_1_http_server` | 1 HTTP Server 自作 | バイナリ | 負荷をかけてもスレッドプールが詰まらない / Keep-Alive に対応 | 未着手 |
| `phase_2_mini_redis_concurrent_processing_async` | 2 Mini Redis + 並行処理/async | バイナリ | `redis-cli` から GET/SET/DEL/EXPIRE ができる | 未着手 |
| `phase_3_persistence_wal_snapshot_recovery` | 3 永続化(WAL/Snapshot/Recovery) | バイナリ | `kill -9` の後に再起動してもデータが戻る | 未着手 |
| `phase_4_db_storage_engine_lsm_btree` | 4 DB / Storage Engine(LSM / B+Tree) | ライブラリ | mini-lsm の各章のテストが通る | 未着手 |

## 構成についての方針

- Cargo workspace は使わず、Phase ごとに独立したプロジェクトにしている
  - `target/` がプロジェクトごとに分かれる、依存ライブラリを毎回ビルドする、長いパッケージ名を何度も打つ、などのデメリットも体験するため
  - Phase 3〜4 まで進んでデメリットを一通り体験したら、workspace に作り替えること自体を練習
- `phase_0_basics` は Cargo プロジェクトにせず、入れ物のディレクトリにする
  - 中に rustlings と、テーマごとの小さな練習用プロジェクトを置く
  - `Cargo.toml` があるディレクトリの配下では `rustlings init` がエラーで止まることがあるため
- Phase 2 → 3 → 4 は、前の Phase のコードをコピーして始める([Phase 共通の進め方](#phase-共

## セットアップ

初めての環境でも、上から順に実行すれば同じ状態を作れるように書いている
各手順の「確認」で期待どおりの結果にならなければ、先へ進まずに[トラブルシューティング](#トラブルシューティング)を見ること

### 0. 前提

- macOS(Apple Silicon)
- ターミナルのシェルは bash または zsh
- Rust は公式のインストーラー(rustup)で管理する
  Homebrew の `rust` や `rustup` は使わない(同じ名前のコマンドを持つので、両方あるとどちら  なる)

作業するディレクトリを変数にしておく(以降の手順はすべてこの変数を使う)

```sh
export REPO="$HOME/git/rust-learning"
```

新しくターミナルを開いたときは、この `export` をもう一度実行すること

### 1. Xcode Command Line Tools と Git

Rust のリンク(実行ファイルを作る最後の工程)に、Apple のコンパイラとリンカが必要

```sh
xcode-select -p || xcode-select --install   # 入っていなければインストールの画面が出る
git --version
```

確認: `xcode-select -p` がパス(例: `/Library/Developer/CommandLineTools`)を表示し、`git --version` がバージョンを表示すること

### 2. Rust の環境

#### 2-1. Homebrew の rust / rustup が入っていれば外す

```sh
brew list --versions rust rustup 2>/dev/null   # 何か表示されたら、以下で外す
brew uninstall rust
brew uninstall rustup
```

- シェルの設定ファイル(`~/.bash_profile`、`~/.zshrc`)に `export PATH=/opt/homebrew/opt/rustup/bin:$PATH` のような行があれば削除する
- `/opt/homebrew/etc/rustup/settings.toml` が残っていたら、消す前に中身を確認する
  `default_toolchain` の行があれば、消した後に 2-3 の `rustup default stable` を必ず実行する

#### 2-2. 公式のインストーラーで rustup を入れる(`~/.cargo/bin/rustup` が無い場合)

```sh
ls -l ~/.cargo/bin/rustup 2>/dev/null || \
  curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source "$HOME/.cargo/env"
```

インストーラーは、シェルの設定ファイルに `. "$HOME/.cargo/env"` を追加する(これで `~/.cargo/bin` が PATH に入る)

#### 2-3. 既定のツールチェーンを stable にする

```sh
rustup default stable
```

すでに設定済みでも、実行して問題ない(何度実行しても同じ結果になる)

#### 2-4. 確認する

```sh
which -a cargo rustc rustup          # ~/.cargo/bin/ の1組だけが表示されること
ls -l ~/.cargo/bin/rustup            # 普通のファイル(数 MB〜十数 MB)で、別の場所へのリンクではないこと
rustup show                          # 「stable-aarch64-apple-darwin (active, default)」と
rustc --version
cargo --version
```

#### 2-5. よく使うツールを追加する

```sh
rustup component add rustfmt clippy rust-analyzer
rustup component list --installed    # rustfmt、clippy、rust-analyzer が含まれること
```

#### 更新のしかた

| 更新するもの | コマンド |
|---|---|
| rustup 自体 | `rustup self update` |
| Rust(コンパイラ) | `rustup update` |

### 3. リポジトリを作る

```sh
mkdir -p "$REPO"
cd "$REPO"
git init                             # cargo new より先に行う(各プロジェクトに .git が入れ
```

`.gitignore` を作る

```sh
cat > "$REPO/.gitignore" <<'GITIGNORE'
# Cargo のビルド成果物(各プロジェクトの直下にできる)
**/target/

# macOS
.DS_Store

# エディタ
.vscode/
.idea/
*.swp

# Phase 3 以降で WAL や Snapshot を書き出す場所(作るときに名前を合わせる)
**/data/
*.wal
*.snapshot
GITIGNORE
```

確認: `ls -a "$REPO"` で `.git` と `.gitignore` があること

### 4. Phase ごとのディレクトリを作る

```sh
cd "$REPO"
mkdir phase_0_basics                 # 入れ物のディレクトリ(Cargo プロジェクトにはしない)
cargo new phase_1_http_server
cargo new phase_2_mini_redis_concurrent_processing_async
cargo new phase_3_persistence_wal_snapshot_recovery
cargo new phase_4_db_storage_engine_lsm_btree --lib   # ライブラリとして作る
```

確認:

```sh
ls -d "$REPO"/phase_*                                # 5つのディレクトリがあること
find "$REPO" -maxdepth 2 -name .git                  # $REPO/.git の1つだけであること
grep '^name' "$REPO"/phase_*/Cargo.toml              # name がディレクトリ名と一致していること
ls "$REPO/phase_4_db_storage_engine_lsm_btree/src"   # lib.rs であること
```

ディレクトリ名を後から変えるときは、`Cargo.toml` の `name` も合わせて変え、`cargo clean` してからビルドし直すこと

### 5. ビルドを確認する

```sh
(cd "$REPO/phase_1_http_server" && cargo run)
(cd "$REPO/phase_2_mini_redis_concurrent_processing_async" && cargo run)
(cd "$REPO/phase_3_persistence_wal_snapshot_recovery" && cargo run)
(cd "$REPO/phase_4_db_storage_engine_lsm_btree" && cargo test)   # ライブラリなのでテストで確認する
du -sh "$REPO"/phase_*/target                                     # target/ はプロジェクト
```

確認: Phase 1〜3 で `Hello, world!` が表示され、Phase 4 で `test result: ok` が表示されること

### 6. 最初のコミット

```sh
# cd "$REPO" # 使用してない
git add .
git status                           # target/ が含まれていないこと
git commit -m "Initialize rust-learning with phase 1-4 projects"
```

`phase_0_basics` は空なので、この時点では Git に表示されない(Git は空のディレクトリを記録し

### 7. Phase 0 の準備

#### 7-1. rustlings を入れて初期化する

```sh
cargo install rustlings              # ~/.cargo/bin/rustlings に入る(数分かかる)
rustlings --version

cd phase_0_basics
rustlings init                       # $REPO/phase_0_basics/rustlings/ ができる
ls rustlings  # exercises/、Cargo.toml などがあること
```

#### 7-2. テーマごとの練習用プロジェクトを作る

そのテーマに入ったときに1つずつ作ってもよい

```sh
cargo new ownership_borrowing        # The Book 4章(所有権・借用)
cargo new structs_enums_match        # 5〜6章(構造体・enum・match)
cargo new error_handling             # 9章(Result / Option)
cargo new traits_generics_lifetime   # 10章(トレイト・ジェネリクス・ライフタイム)
cargo new smart_pointers             # 15章(Box / Rc / RefCell)
cargo new concurrency                # 16章(Arc / Mutex / RwLock / Send・Sync)
ll
```

#### 7-3. コミットする

```sh
git add phase_0_basics
git status                           # rustlings/ と練習用プロジェクトが追加され、target/ は含まれないこと
git commit -m "Add phase 0 basics: rustlings and exercise projects"
```

## Phase 0:Rust の基礎の進め方

### 教材

| 教材 | 使い方 |
|---|---|
| The Book(https://doc.rust-jp.rs/book-ja/) | 主教材。章を読んでから、対応する rustlings を解く |
| rustlings(`$REPO/phase_0_basics/rustlings`) | 手を動かす練習問題。コンパイルエラーを直し
| Comprehensive Rust(https://google.github.io/comprehensive-rust/ja/) | The Book で分かりにくかった箇所の別の説明として読む |

### 始め方

```sh
cd "$REPO/phase_0_basics/rustlings"
rustlings                            # 対話形式の画面が開く
```

- エディタで `exercises/` の問題ファイルを開いて直す。保存すると自動で答え合わせされる
- 画面で `h` を押すとヒントが出る。`n` で次の問題へ進む、`q` で終了する(操作は画面の表示に従う)
- 途中でやめても、次に `rustlings` を起動すると続きから始まる

### 学習の順序(The Book の章と rustlings の対応)

| 順 | The Book | rustlings の問題 | 練習用プロジェクト | NEXT_PLAN のキーワード |
|---|---|---|---|---|
| 1 | 1〜3章(はじめに・数当てゲーム・一般的な概念) | `00_intro`〜`04_primitive_types` | ― |
| 2 | 4章(所有権) | `06_move_semantics` | `ownership_borrowing` | ownership / borrowing |
| 3 | 5章(構造体)・6章(enum と match) | `07_structs`、`08_enums` | `structs_enums_match` |
| 4 | 7章(モジュール)・8章(コレクション) | `05_vecs`、`09_strings`、`10_modules`、`11_hashmaps` | ― | ― |
| 5 | 6章の Option・9章(エラー処理) | `12_options`、`13_error_handling` | `error_handling`
| 6 | 10章(ジェネリクス・トレイト・ライフタイム) | `14_generics`、`15_traits`、`16_lifetimes` | `traits_generics_lifetime` | trait / lifetime |
| 7 | 11章(テスト)・13章(イテレータとクロージャ) | `17_tests`、`18_iterators` | ― | ― |
| 8 | 15章(スマートポインタ) | `19_smart_pointers` | `smart_pointers` | Box / Rc / RefCell |
| 9 | 16章(並行性) | `20_threads` | `concurrency` | Arc / Mutex / RwLock / Send / Sync |
| 10 | 19章(マクロ)など | `21_macros`〜最後まで | ― | ― |

rustlings の問題の並び(番号)は The Book の章の順と完全には一致しないので、上の表を目安に行き来してよい

### 練習用プロジェクトの課題

rustlings を解くだけでなく、自分で一からコードを書いて確かめる

| プロジェクト | 課題の例 | 確かめたいこと |
|---|---|---|
| `ownership_borrowing` | `String` を関数に渡す・返す・参照で渡す、を書き比べる。わざと借用エラーを起こして、エラーメッセージを読んでから直す | move と借用の違い、`&` と `&mut` の同時に使えない規則 |
| `structs_enums_match` | 図形(円・四角)を enum で表し、`match` で面積を計算する | enum に  網羅性チェック |
| `error_handling` | 設定ファイル(テキスト)を読んで数値に変換する。ファイルが無い・形式が違う、をそれぞれ `Result` で返し、`?` で伝える | `Result` / `Option`、`?` 演算子、`unwrap` を使わない書き方 |
| `traits_generics_lifetime` | 「最も長い文字列を返す」関数をライフタイム付きで書く。自作の  | ライフタイム注釈が必要になる理由、トレイト境界 |
| `smart_pointers` | 単方向の連結リストを `Box` で作る。次に、双方向の連結リストを `Rc<RefCell<T>>` で作る | `Box` の再帰型、`Rc` の参照カウント、`RefCell` の実行時借用チェック(`borrow_mut` の二重呼び出しで
panic することも確かめる) |
| `concurrency` | 複数スレッドでカウンタを加算する(`Arc<Mutex<T>>`)。`RwLock` 版と、チャンネル(`mpsc`)版も作る | `Send` / `Sync` が必要になる場面、`Rc` をスレッドに渡すとコンパイルエラーになること |

各プロジェクトは次のコマンドで実行・確認する

```sh
cd "$REPO/phase_0_basics/<プロジェクト名>"
cargo run
cargo clippy                         # より良い書き方の提案を見る
cargo fmt                            # 書式を整える
```

### Phase 0 の完了条件

- [ ] rustlings を全問解いた(`rustlings` の画面で全問完了と表示される)
- [ ] 練習用プロジェクト6つの課題を終えた
- [ ] 借用エラー(`borrow of moved value`、`cannot borrow as mutable` など)を、エラーメッセ

完了したら、状態を「完了」に更新してタグを付ける

```sh
cd "$REPO"
git add -A
git commit -m "Complete phase 0"
git tag phase-0-done
```

## Phase 共通の進め方

- 詰まったときだけ参考実装を見る(写経ではなく、自分で仕様を決めて作る)
- こまめにコミットし、Phase が終わったらタグを付ける(例: `phase-1-done`)
- 各 Phase の教材は `NEXT_PLAN.md` を参照

### 前の Phase のコードを次へ持っていく手順

Phase 2 の完了後に Phase 3 を始めるときの例

```sh
cd "$REPO"
rsync -a --exclude target \
  phase_2_mini_redis_concurrent_processing_async/ \
  phase_3_persistence_wal_snapshot_recovery/
cd phase_3_persistence_wal_snapshot_recovery
sed -i '' 's/^name = "phase_2_mini_redis_concurrent_processing_async"/name = "phase_3_persistence_wal_snapshot_recovery"/' Cargo.toml
grep '^name' Cargo.toml              # name が phase_3_... になっていること
cargo run
```

- `Cargo.lock` もコピーされるので、依存のバージョンは Phase 2 と同じになる
- 以後、Phase 3 で直したバグは Phase 2 には反映されない(コピーする方式のデメリット)

## トラブルシューティング

| 症状 | 原因 | 対処 |
|---|---|---|
| `rustup-init: command not found` | Homebrew の `rustup` には `rustup-init` が含まれなくなった | セットアップ 2-2 の公式インストーラーを使う |
| `which -a cargo` に複数の場所が出る | Homebrew 版と公式版が両方入っている | セットアップ
| `rustup could not choose a version of cargo to run … no default is configured` | 既定のツールチェーンが設定されていない(Homebrew 版の設定ファイルを消した後などに起きる) | `rustup default stable` |
| `cd: …: No such file or directory` | 別のディレクトリにいる状態で相対パスを使った | `expoarning"` を実行し直し、`cd "$REPO/…"` の形で移動する |
| リンク時に `tapi error: malformed file … unknown architecture` | Command Line Tools が、新しい macOS SDK を読めない | Command Line Tools を更新する。または `ls /Library/Developer/CommandLineTools/SDKs/` で
SDK を確認し、`export SDKROOT=/Library/Developer/CommandLineTools/SDKs/MacOSX26.5.sdk` のよ |
| `rustlings init` がエラーで止まる | `Cargo.toml` があるディレクトリの配下で実行した | `$REPO/phase_0_basics` の中で実行する(`phase_0_basics` 自体に `Cargo.toml` を置かない) |
| ディレクトリ名を変えたらビルド結果の名前が古いまま | `Cargo.toml` の `name` が古いまま |  clean` し、ビルドし直す |
