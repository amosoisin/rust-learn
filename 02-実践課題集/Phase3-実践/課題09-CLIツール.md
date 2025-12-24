# 課題09: 高機能CLIツール

[[課題08-組込み向けプロジェクト|← 前の課題]]  |  [[README|トップに戻る]]  |  [[課題10-WebAPIサーバー|次の課題 →]]

**難易度:** ⭐️⭐️⭐️⭐️
**学習時間目安:** 10-15時間
**学習トピック:** clap、tokio、async/await、非同期I/O

---

## 🎯 課題の目的

実用的なCLIツールを作成し、非同期プログラミングとコマンドライン引数の解析を学びます。

---

## 📋 プロジェクト例

### プロジェクト1: ファイル検索ツール（`find`の代替）

```bash
$ cargo run -- search "*.rs" --content "async"
src/main.rs:15: async fn process() {
src/lib.rs:42: async fn handle_request() {
```

### プロジェクト2: HTTPクライアント（`curl`の代替）

```bash
$ cargo run -- get https://api.github.com/users/rust-lang
{
  "login": "rust-lang",
  "name": "Rust Programming Language",
  ...
}
```

---

## 🏗️ 実装ガイド

### clap での引数解析

```toml
# Cargo.toml
[dependencies]
clap = { version = "4.0", features = ["derive"] }
tokio = { version = "1", features = ["full"] }
reqwest = "0.11"
```

```rust
use clap::{Parser, Subcommand};

#[derive(Parser)]
#[command(name = "mytool")]
#[command(about = "A powerful CLI tool", long_about = None)]
struct Cli {
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]
enum Commands {
    /// Search for files
    Search {
        /// File pattern
        pattern: String,

        /// Search in file contents
        #[arg(short, long)]
        content: Option<String>,
    },
    /// HTTP GET request
    Get {
        /// URL to fetch
        url: String,

        /// Output file
        #[arg(short, long)]
        output: Option<String>,
    },
}

#[tokio::main]
async fn main() {
    let cli = Cli::parse();

    match cli.command {
        Commands::Search { pattern, content } => {
            search_files(&pattern, content.as_deref()).await;
        }
        Commands::Get { url, output } => {
            fetch_url(&url, output.as_deref()).await;
        }
    }
}

async fn search_files(pattern: &str, content: Option<&str>) {
    // TODO: 実装
}

async fn fetch_url(url: &str, output: Option<&str>) {
    let response = reqwest::get(url).await.unwrap();
    let body = response.text().await.unwrap();

    match output {
        Some(path) => {
            tokio::fs::write(path, body).await.unwrap();
            println!("Saved to {}", path);
        }
        None => {
            println!("{}", body);
        }
    }
}
```

---

## 💡 async/await の理解

### 同期 vs 非同期

```rust
// 同期（ブロッキング）
fn fetch_sync(url: &str) -> String {
    // ネットワーク応答を待つ間、スレッドがブロックされる
    reqwest::blocking::get(url).unwrap().text().unwrap()
}

// 非同期（ノンブロッキング）
async fn fetch_async(url: &str) -> String {
    // 待機中に他のタスクを実行できる
    reqwest::get(url).await.unwrap().text().await.unwrap()
}
```

### 複数の非同期処理を並行実行

```rust
use tokio::join;

async fn fetch_multiple(urls: &[&str]) {
    let tasks: Vec<_> = urls.iter()
        .map(|url| fetch_async(url))
        .collect();

    let results = join!(tasks);  // すべて並行実行
}
```

---

## 🚀 発展課題

### レベル1: プログレスバー

```toml
[dependencies]
indicatif = "0.17"
```

### レベル2: 設定ファイルのサポート

```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
toml = "0.8"
```

### レベル3: 自動補完スクリプトの生成

---

## 🔗 関連資料

- [clap Documentation](https://docs.rs/clap/)
- [Tokio Tutorial](https://tokio.rs/tokio/tutorial)
- [The Async Book](https://rust-lang.github.io/async-book/)

---

## 🎓 完了チェックリスト

- [ ] clapでのCLI引数解析ができる
- [ ] async/awaitを理解した
- [ ] 非同期HTTPリクエストができる
- [ ] 実用的なツールを完成させた

---

[[課題10-WebAPIサーバー|次の課題: WebAPIサーバー →]]
