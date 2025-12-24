# 課題04: TODOリストCLI

[[課題03-CSVパーサー|← 前の課題]]  |  [[README|トップに戻る]]

**難易度:** ⭐️⭐️⭐️
**学習時間目安:** 5-6時間
**学習トピック:** Option, Result, ファイルI/O, JSON/永続化

---

## 🎯 課題の目的

`Option`と`Result`を使ったエラーハンドリングと、データの永続化を学ぶためのTODOリストCLIツールを作成します。

### 学ぶこと
- `Option<T>` と `Result<T, E>` の実践的な使用
- `?` 演算子によるエラー伝播
- ファイルへのデータ保存
- JSONシリアライゼーション（`serde`）

---

## 📋 仕様

TODOアイテムの追加、完了、削除、一覧表示ができるCLIツールを作成します。

### 基本コマンド

```bash
# TODOを追加
$ cargo run -- add "Rustの勉強をする"
Added: Rustの勉強をする

# すべてのTODOを表示
$ cargo run -- list
1. [ ] Rustの勉強をする
2. [ ] 買い物に行く
3. [x] メールを返信する

# TODOを完了にする
$ cargo run -- done 1
Completed: Rustの勉強をする

# TODOを削除
$ cargo run -- remove 2
Removed: 買い物に行く
```

---

## 💡 Option vs Result

### いつ Option を使う？

**Option**は「値があるかもしれない、ないかもしれない」ときに使います。エラーではなく、**正常な状態**です。

```rust
// IDからアイテムを検索（見つからないのは正常）
fn find_by_id(&self, id: usize) -> Option<&TodoItem> {
    self.items.iter().find(|item| item.id == id)
}

// 使用例
match todos.find_by_id(1) {
    Some(item) => println!("Found: {}", item.task),
    None => println!("Item not found"),  // エラーではない
}
```

**C言語との比較：**

```c
// C言語: NULLポインタで表現（型安全でない）
TodoItem* find_by_id(int id) {
    // ...
    return NULL;  // 見つからない場合
}

// NULLチェック忘れでクラッシュ！
TodoItem *item = find_by_id(1);
printf("%s", item->task);  // itemがNULLならクラッシュ
```

```rust
// Rust: Optionで型安全に
fn find_by_id(&self, id: usize) -> Option<&TodoItem> {
    // ...
}

// コンパイラがNoneチェックを強制
let item = find_by_id(1);
println!("{}", item.task);  // コンパイルエラー！
// Optionを unwrap するか match で処理する必要がある
```

---

### いつ Result を使う？

**Result**は「成功するかもしれない、エラーになるかもしれない」ときに使います。エラー情報も一緒に返します。

```rust
// ファイルの読み込み（エラーの可能性がある）
fn load_todos() -> Result<TodoList, String> {
    let contents = fs::read_to_string(DATA_FILE)
        .map_err(|e| format!("Failed to read: {}", e))?;
    // ...
}

// 使用例
match load_todos() {
    Ok(todos) => { /* 処理 */ }
    Err(e) => eprintln!("Error: {}", e),  // エラー情報を表示
}
```

**C言語との比較：**

```c
// C言語: エラーコードを返す（エラー詳細は別途取得）
int load_todos(TodoList **out) {
    FILE *fp = fopen("todos.json", "r");
    if (!fp) {
        return -1;  // エラーコード（詳細不明）
    }
    // エラー詳細は errno や strerror() で別途取得
    // ...
}
```

```rust
// Rust: 成功/失敗とエラー詳細を同時に返す
fn load_todos() -> Result<TodoList, String> {
    // エラー時は詳細なメッセージを返す
    let file = File::open("todos.json")
        .map_err(|e| format!("Failed to open: {}", e))?;
    // ...
}
```

**★ 使い分けのポイント：**
- `Option<T>`: 値がない可能性があるだけ（正常）→ `find`, `get`, `first`, `last`
- `Result<T, E>`: エラーが発生する可能性がある → `parse`, `open`, `read`, `write`

---

## 🏗️ 実装の流れ

### ステップ1: プロジェクトの作成

```bash
cargo new todo_cli
cd todo_cli
```

### ステップ2: 依存関係の追加

```toml
# Cargo.toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

**serde とは？**

構造体を JSON に変換（シリアライゼーション）したり、JSON を構造体に変換（デシリアライゼーション）したりするライブラリです。

```rust
#[derive(Serialize, Deserialize)]  // この1行で JSON 変換機能が追加される
struct TodoItem {
    id: usize,
    task: String,
}

// 使用例
let item = TodoItem { id: 1, task: "Test".to_string() };
let json = serde_json::to_string(&item)?;  // JSON文字列に変換
// json = "{\"id\":1,\"task\":\"Test\"}"

let item2: TodoItem = serde_json::from_str(&json)?;  // JSON から復元
```

---

### ステップ3: データ構造の定義

```rust
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
struct TodoItem {
    id: usize,
    task: String,
    completed: bool,
}

#[derive(Debug, Serialize, Deserialize)]
struct TodoList {
    items: Vec<TodoItem>,
    next_id: usize,
}

impl TodoList {
    fn new() -> Self {
        TodoList {
            items: Vec::new(),
            next_id: 1,
        }
    }

    // TODO(human): これらのメソッドを実装する
}
```

---

## ● Learn by Doing

**Context:** TODOリストの基本構造は用意されています。JSONファイルにデータを保存し、プログラムを再起動しても TODO が残る仕組みを実装します。`Option`は「見つからない可能性」を、`Result`は「失敗する可能性」を表現します。これらを使い分けることで、型安全なエラーハンドリングを学びます。

**Your Task:** `src/main.rs`に以下のコードを作成し、TODO(human)の箇所を実装してください。

**Guidance:**
- `add`: `self.next_id`を使って新しいTODOを作成し、`items`に追加します。`next_id`をインクリメントし、最後に追加したアイテムの参照を返します
- `complete`: `items.iter_mut()`で可変イテレータを取得し、`find()`でIDが一致するアイテムを探します。見つかれば`completed = true`にして、`Some`で返します
- `remove`: `items.iter().position()`でインデックスを見つけ、`items.remove(index)`で削除します。見つからなければ`None`を返します

```rust
use serde::{Deserialize, Serialize};
use std::env;
use std::fs;
use std::path::Path;

const DATA_FILE: &str = "todos.json";

#[derive(Debug, Serialize, Deserialize)]
struct TodoItem {
    id: usize,
    task: String,
    completed: bool,
}

#[derive(Debug, Serialize, Deserialize)]
struct TodoList {
    items: Vec<TodoItem>,
    next_id: usize,
}

impl TodoList {
    fn new() -> Self {
        TodoList {
            items: Vec::new(),
            next_id: 1,
        }
    }

    // TODO(human): TODOを追加するメソッドを実装
    // ヒント:
    // 1. TodoItem を作成（id: self.next_id, task: 引数, completed: false）
    // 2. self.items.push(item) で追加
    // 3. self.next_id += 1 でIDをインクリメント
    // 4. self.items.last().unwrap() で最後の要素の参照を返す
    fn add(&mut self, task: String) -> &TodoItem {
        // ここに実装
        todo!()
    }

    // TODO(human): TODOを完了にするメソッドを実装
    // ヒント:
    // 1. self.items.iter_mut() で可変イテレータを取得
    // 2. .find(|item| item.id == id) で探す
    // 3. .map(|item| { ... }) で見つかった場合の処理
    // 4. item.completed = true に設定
    // 5. 参照を返す（&*item）
    fn complete(&mut self, id: usize) -> Option<&TodoItem> {
        // ここに実装
        todo!()
    }

    // TODO(human): TODOを削除するメソッドを実装
    // ヒント:
    // 1. self.items.iter().position(|item| item.id == id) でインデックスを探す
    // 2. .map(|index| self.items.remove(index)) で削除
    // 3. position が None なら None を返す
    fn remove(&mut self, id: usize) -> Option<TodoItem> {
        // ここに実装
        todo!()
    }

    fn list(&self) {
        if self.items.is_empty() {
            println!("No tasks!");
            return;
        }

        for item in &self.items {
            let status = if item.completed { "x" } else { " " };
            println!("{}. [{}] {}", item.id, status, item.task);
        }
    }
}

fn load_todos() -> Result<TodoList, String> {
    if !Path::new(DATA_FILE).exists() {
        return Ok(TodoList::new());
    }

    let contents = fs::read_to_string(DATA_FILE)
        .map_err(|e| format!("Failed to read file: {}", e))?;

    serde_json::from_str(&contents)
        .map_err(|e| format!("Failed to parse JSON: {}", e))
}

fn save_todos(todos: &TodoList) -> Result<(), String> {
    let json = serde_json::to_string_pretty(todos)
        .map_err(|e| format!("Failed to serialize: {}", e))?;

    fs::write(DATA_FILE, json)
        .map_err(|e| format!("Failed to write file: {}", e))
}

fn main() {
    let args: Vec<String> = env::args().collect();

    if args.len() < 2 {
        eprintln!("Usage: todo_cli <command> [args]");
        eprintln!("Commands:");
        eprintln!("  add <task>    - TODOを追加");
        eprintln!("  list          - すべてのTODOを表示");
        eprintln!("  done <id>     - TODOを完了にする");
        eprintln!("  remove <id>   - TODOを削除");
        std::process::exit(1);
    }

    let mut todos = match load_todos() {
        Ok(t) => t,
        Err(e) => {
            eprintln!("Error loading todos: {}", e);
            std::process::exit(1);
        }
    };

    let command = &args[1];

    match command.as_str() {
        "add" => {
            if args.len() < 3 {
                eprintln!("Usage: todo_cli add <task>");
                std::process::exit(1);
            }
            let task = args[2..].join(" ");
            let item = todos.add(task);
            println!("Added: {}", item.task);
        }
        "list" => {
            todos.list();
        }
        "done" => {
            if args.len() < 3 {
                eprintln!("Usage: todo_cli done <id>");
                std::process::exit(1);
            }
            let id: usize = args[2].parse().unwrap_or_else(|_| {
                eprintln!("Invalid ID");
                std::process::exit(1);
            });

            match todos.complete(id) {
                Some(item) => println!("Completed: {}", item.task),
                None => eprintln!("Task {} not found", id),
            }
        }
        "remove" => {
            if args.len() < 3 {
                eprintln!("Usage: todo_cli remove <id>");
                std::process::exit(1);
            }
            let id: usize = args[2].parse().unwrap_or_else(|_| {
                eprintln!("Invalid ID");
                std::process::exit(1);
            });

            match todos.remove(id) {
                Some(item) => println!("Removed: {}", item.task),
                None => eprintln!("Task {} not found", id),
            }
        }
        _ => {
            eprintln!("Unknown command: {}", command);
            std::process::exit(1);
        }
    }

    if let Err(e) = save_todos(&todos) {
        eprintln!("Error saving todos: {}", e);
        std::process::exit(1);
    }
}
```

---

## 🐛 デバッグのヒント

実装中にエラーが出たら、[[デバッグガイド]]を参照してください。

よくあるエラー：

1. **`error[E0502]: cannot borrow as mutable`** - 不変借用中に可変借用しようとしている
   - `iter_mut()`を使うときは、他の借用がないことを確認します

2. **`error[E0515]: cannot return reference to local variable`** - ローカル変数への参照を返そうとしている
   - `self.items.last()`は`self`のフィールドなので、参照を返せます

3. **`map() のクロージャの返り値が合わない`** - 型の不一致
   - `&*item`で可変参照を不変参照に変換します

---

## 📚 学習ポイント

### ? 演算子の活用

`?`演算子は`Result`のエラーを自動的に伝播させます。

```rust
// 冗長な書き方
fn load_todos() -> Result<TodoList, String> {
    let contents = match fs::read_to_string(DATA_FILE) {
        Ok(c) => c,
        Err(e) => return Err(format!("Failed: {}", e)),
    };

    match serde_json::from_str(&contents) {
        Ok(t) => Ok(t),
        Err(e) => Err(format!("Failed: {}", e)),
    }
}

// ? 演算子で簡潔に
fn load_todos() -> Result<TodoList, String> {
    let contents = fs::read_to_string(DATA_FILE)
        .map_err(|e| format!("Failed: {}", e))?;

    serde_json::from_str(&contents)
        .map_err(|e| format!("Failed: {}", e))
}
```

**C言語との比較：**

```c
// C言語: エラーチェックが冗長
int load_todos(TodoList **out) {
    FILE *fp = fopen("todos.json", "r");
    if (!fp) {
        return -1;  // エラー
    }

    char *buf = malloc(1024);
    if (!buf) {
        fclose(fp);
        return -1;  // エラー（fcloseを忘れないように）
    }

    if (fread(buf, 1, 1024, fp) < 0) {
        free(buf);
        fclose(fp);
        return -1;  // エラー（両方のクリーンアップ必要）
    }

    // ...
    free(buf);
    fclose(fp);
    return 0;
}
```

```rust
// Rust: ? 演算子で自動的にエラー伝播、メモリも自動解放
fn load_todos() -> Result<TodoList, String> {
    let contents = fs::read_to_string(DATA_FILE)?;  // エラーなら自動で return
    let todos = serde_json::from_str(&contents)?;   // エラーなら自動で return
    Ok(todos)  // 成功
    // メモリは自動的に解放される
}
```

---

### Option のメソッド

```rust
// unwrap_or: デフォルト値を指定
let item = todos.find_by_id(1).unwrap_or(&default_item);

// unwrap_or_else: 関数でデフォルト値を計算
let item = todos.find_by_id(1).unwrap_or_else(|| create_default());

// map: Option の中身を変換
let task_name = todos.find_by_id(1).map(|item| &item.task);
// Some(item) → Some(&item.task)
// None → None

// and_then: Option を返す処理をチェーン
let result = todos.find_by_id(1)
    .and_then(|item| item.subtasks.first());
```

---

## 🚀 発展課題

### レベル1: 期限の追加

```rust
use chrono::NaiveDate;  // Cargo.toml に chrono = "0.4" を追加

#[derive(Debug, Serialize, Deserialize)]
struct TodoItem {
    id: usize,
    task: String,
    completed: bool,
    due_date: Option<NaiveDate>,  // 期限（オプション）
}

// 使用例
$ cargo run -- add "レポート提出" --due 2025-01-31
```

### レベル2: 優先度

```rust
#[derive(Debug, Serialize, Deserialize)]
enum Priority {
    Low,
    Medium,
    High,
}

#[derive(Debug, Serialize, Deserialize)]
struct TodoItem {
    // ...
    priority: Priority,
}

// 使用例
$ cargo run -- add "緊急タスク" --priority high
```

### レベル3: フィルタリング

```bash
$ cargo run -- list --completed
$ cargo run -- list --pending
$ cargo run -- list --priority high
```

実装ヒント：
```rust
fn list_filtered(&self, show_completed: Option<bool>, priority: Option<Priority>) {
    let filtered: Vec<_> = self.items.iter()
        .filter(|item| {
            // show_completed の条件をチェック
            if let Some(comp) = show_completed {
                if item.completed != comp {
                    return false;
                }
            }
            // priority の条件をチェック
            if let Some(ref prio) = priority {
                if item.priority != *prio {
                    return false;
                }
            }
            true
        })
        .collect();

    for item in filtered {
        // 表示
    }
}
```

### レベル4: 編集機能

```bash
$ cargo run -- edit 1 "新しいタスク内容"
```

---

<details>
<summary>📝 完全な解答例を見る（実装後に確認）</summary>

```rust
use serde::{Deserialize, Serialize};
use std::env;
use std::fs;
use std::path::Path;

const DATA_FILE: &str = "todos.json";

#[derive(Debug, Serialize, Deserialize)]
struct TodoItem {
    id: usize,
    task: String,
    completed: bool,
}

#[derive(Debug, Serialize, Deserialize)]
struct TodoList {
    items: Vec<TodoItem>,
    next_id: usize,
}

impl TodoList {
    fn new() -> Self {
        TodoList {
            items: Vec::new(),
            next_id: 1,
        }
    }

    fn add(&mut self, task: String) -> &TodoItem {
        let item = TodoItem {
            id: self.next_id,
            task,
            completed: false,
        };
        self.items.push(item);
        self.next_id += 1;
        self.items.last().unwrap()
    }

    fn complete(&mut self, id: usize) -> Option<&TodoItem> {
        self.items.iter_mut()
            .find(|item| item.id == id)
            .map(|item| {
                item.completed = true;
                &*item
            })
    }

    fn remove(&mut self, id: usize) -> Option<TodoItem> {
        self.items.iter()
            .position(|item| item.id == id)
            .map(|index| self.items.remove(index))
    }

    fn list(&self) {
        if self.items.is_empty() {
            println!("No tasks!");
            return;
        }

        for item in &self.items {
            let status = if item.completed { "x" } else { " " };
            println!("{}. [{}] {}", item.id, status, item.task);
        }
    }
}

fn load_todos() -> Result<TodoList, String> {
    if !Path::new(DATA_FILE).exists() {
        return Ok(TodoList::new());
    }

    let contents = fs::read_to_string(DATA_FILE)
        .map_err(|e| format!("Failed to read file: {}", e))?;

    serde_json::from_str(&contents)
        .map_err(|e| format!("Failed to parse JSON: {}", e))
}

fn save_todos(todos: &TodoList) -> Result<(), String> {
    let json = serde_json::to_string_pretty(todos)
        .map_err(|e| format!("Failed to serialize: {}", e))?;

    fs::write(DATA_FILE, json)
        .map_err(|e| format!("Failed to write file: {}", e))
}

fn main() {
    let args: Vec<String> = env::args().collect();

    if args.len() < 2 {
        eprintln!("Usage: todo_cli <command> [args]");
        eprintln!("Commands:");
        eprintln!("  add <task>    - TODOを追加");
        eprintln!("  list          - すべてのTODOを表示");
        eprintln!("  done <id>     - TODOを完了にする");
        eprintln!("  remove <id>   - TODOを削除");
        std::process::exit(1);
    }

    let mut todos = match load_todos() {
        Ok(t) => t,
        Err(e) => {
            eprintln!("Error loading todos: {}", e);
            std::process::exit(1);
        }
    };

    let command = &args[1];

    match command.as_str() {
        "add" => {
            if args.len() < 3 {
                eprintln!("Usage: todo_cli add <task>");
                std::process::exit(1);
            }
            let task = args[2..].join(" ");
            let item = todos.add(task);
            println!("Added: {}", item.task);
        }
        "list" => {
            todos.list();
        }
        "done" => {
            if args.len() < 3 {
                eprintln!("Usage: todo_cli done <id>");
                std::process::exit(1);
            }
            let id: usize = args[2].parse().unwrap_or_else(|_| {
                eprintln!("Invalid ID");
                std::process::exit(1);
            });

            match todos.complete(id) {
                Some(item) => println!("Completed: {}", item.task),
                None => eprintln!("Task {} not found", id),
            }
        }
        "remove" => {
            if args.len() < 3 {
                eprintln!("Usage: todo_cli remove <id>");
                std::process::exit(1);
            }
            let id: usize = args[2].parse().unwrap_or_else(|_| {
                eprintln!("Invalid ID");
                std::process::exit(1);
            });

            match todos.remove(id) {
                Some(item) => println!("Removed: {}", item.task),
                None => eprintln!("Task {} not found", id),
            }
        }
        _ => {
            eprintln!("Unknown command: {}", command);
            std::process::exit(1);
        }
    }

    if let Err(e) = save_todos(&todos) {
        eprintln!("Error saving todos: {}", e);
        std::process::exit(1);
    }
}
```

</details>

---

## 🔗 関連資料

- [[デバッグガイド]] - エラーが出た時の解決方法
- [The Rust Book: エラー処理](https://doc.rust-jp.rs/book-ja/ch09-00-error-handling.html)
- [serde ドキュメント](https://serde.rs/)

---

## 🎓 完了チェックリスト

- [ ] `Option`型と`Result`型の違いを理解した
- [ ] `?` 演算子を使ったエラー伝播ができる
- [ ] JSONでのデータ永続化ができる
- [ ] すべての基本コマンドが動作する
- [ ] 発展課題に1つ以上チャレンジした

完了したら、Phase 2に進みましょう！

---

次は [[課題05-汎用データ構造|Phase 2: 課題05-汎用データ構造]] へ
