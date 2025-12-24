# 課題03: CSVパーサー

[[課題02-文字列操作ツール|← 前の課題]]  |  [[README|トップに戻る]]  |  [[課題04-TODOリストCLI|次の課題 →]]

**難易度:** ⭐️⭐️
**学習時間目安:** 4-5時間
**学習トピック:** 構造体、列挙型、パターンマッチング、ベクタ

---

## 🎯 課題の目的

構造体と列挙型を使って、CSVファイルを解析し、データを操作するプログラムを作成します。

### 学ぶこと
- 構造体の定義と使用
- `impl` ブロックでのメソッド定義
- `enum` による型の表現
- ベクタの操作
- ファイルI/O

---

## 📋 仕様

CSVファイルを読み込み、データの表示、フィルタリング、集計を行うツールを作成します。

### サンプルデータ (users.csv)

```csv
id,name,age,city
1,Alice,25,Tokyo
2,Bob,30,Osaka
3,Carol,22,Tokyo
4,Dave,35,Nagoya
5,Eve,28,Tokyo
```

### 基本コマンド

```bash
# すべてのデータを表示
$ cargo run -- show users.csv
ID | Name  | Age | City
1  | Alice | 25  | Tokyo
2  | Bob   | 30  | Osaka
3  | Carol | 22  | Tokyo
4  | Dave  | 35  | Nagoya
5  | Eve   | 28  | Tokyo

# 特定の都市でフィルタ
$ cargo run -- filter users.csv --city Tokyo
ID | Name  | Age | City
1  | Alice | 25  | Tokyo
3  | Carol | 22  | Tokyo
5  | Eve   | 28  | Tokyo

# 年齢の平均を計算
$ cargo run -- average users.csv
Average age: 28.0
```

---

## 💡 C言語との比較

### C言語での実装

```c
typedef struct {
    int id;
    char name[50];    // 固定長配列
    int age;
    char city[50];    // 固定長配列
} User;

User* load_users(const char *filename, int *count) {
    FILE *fp = fopen(filename, "r");
    if (!fp) return NULL;

    User *users = malloc(100 * sizeof(User));  // 固定サイズ！
    *count = 0;

    char line[256];
    fgets(line, sizeof(line), fp);  // ヘッダースキップ

    while (fgets(line, sizeof(line), fp)) {
        sscanf(line, "%d,%49[^,],%d,%49[^\n]",
               &users[*count].id,
               users[*count].name,
               &users[*count].age,
               users[*count].city);
        (*count)++;
    }

    fclose(fp);
    return users;  // 呼び出し側で free() が必要！
}
```

**C言語の問題点：**
- 固定サイズの配列（バッファオーバーフローのリスク）
- 手動メモリ管理（free忘れでメモリリーク）
- エラーハンドリングが不十分（NULLチェック漏れ）
- カウント変数を別途渡す必要がある

### Rustでの実装

```rust
#[derive(Debug)]  // デバッグ出力を自動生成
struct User {
    id: u32,
    name: String,    // 動的サイズ
    age: u32,
    city: String,    // 動的サイズ
}

fn load_users(filename: &str) -> Result<Vec<User>, String> {
    // Vec<User> は動的サイズ（自動的に伸びる）
    // Result でエラーハンドリングが型で保証される
    // 自動メモリ管理（スコープを抜けると自動解放）
}
```

**Rustの利点：**
- 動的サイズ（Stringとvec!）
- 自動メモリ管理（所有権システム）
- 型システムによるエラーハンドリングの強制
- データと長さが一緒に管理される

---

## 🏗️ 実装の流れ

### ステップ1: プロジェクトの作成

```bash
cargo new csv_parser
cd csv_parser
```

### ステップ2: users.csvファイルを作成

プロジェクトのルートに`users.csv`を作成：

```csv
id,name,age,city
1,Alice,25,Tokyo
2,Bob,30,Osaka
3,Carol,22,Tokyo
4,Dave,35,Nagoya
5,Eve,28,Tokyo
```

### ステップ3: データ構造の定義

```rust
#[derive(Debug)]
struct User {
    id: u32,
    name: String,
    age: u32,
    city: String,
}

impl User {
    // TODO(human): このメソッドを実装する

    fn display(&self) {
        println!("{:<3} | {:<6} | {:<3} | {}",
                 self.id, self.name, self.age, self.city);
    }
}
```

**★ `impl` ブロックとは？**

C言語には構造体に関数を紐付ける機能はありません。Rustでは`impl`ブロックを使ってメソッドを定義できます。

```c
// C言語: 構造体と関数は分離
void user_display(const User *user) {
    printf("%d | %s\n", user->id, user->name);
}
```

```rust
// Rust: 構造体とメソッドが紐付く
impl User {
    fn display(&self) {  // &self は this に相当
        println!("{} | {}", self.id, self.name);
    }
}

// 使用例
user.display();  // user_display(&user) より自然
```

---

### ステップ4: CSVファイルの読み込み

```rust
use std::fs::File;
use std::io::{BufRead, BufReader};

fn load_users(filename: &str) -> Result<Vec<User>, String> {
    let file = File::open(filename)
        .map_err(|e| format!("Failed to open file: {}", e))?;

    let reader = BufReader::new(file);
    let mut users = Vec::new();

    for (i, line) in reader.lines().enumerate() {
        if i == 0 {
            continue;  // ヘッダー行をスキップ
        }

        let line = line.map_err(|e| format!("Failed to read line: {}", e))?;
        let user = User::from_csv_line(&line)?;
        users.push(user);
    }

    Ok(users)
}
```

**★ `map_err` とは？**

エラー型を変換するメソッドです。`std::io::Error`を`String`に変換しています。

```rust
// map_err なし（エラー型が合わない）
let file = File::open(filename)?;  // エラー: io::Error を String に変換できない

// map_err あり（エラー型を変換）
let file = File::open(filename)
    .map_err(|e| format!("Failed: {}", e))?;  // OK: String に変換
```

---

## ● Learn by Doing

**Context:** CSVパーサーの基本構造はすでに用意されています。ファイルを読み込み、各行を解析してUser構造体に変換し、フィルタリングや集計を行う機能を実装します。構造体のメソッドとして定義することで、データと操作を一体化させるRustの設計パターンを学びます。

**Your Task:** `src/main.rs`に以下のコードを作成し、TODO(human)の箇所を実装してください。

**Guidance:**
- `from_csv_line`: CSV行を`,`で分割し、各フィールドをパースします。`split(',')`と`collect()`を使い、`parse()`で文字列を数値に変換します。エラー時は`Err`を返します
- `filter_by_city`: イテレータの`filter()`メソッドを使い、都市名が一致するユーザーだけを抽出します
- `calculate_average_age`: イテレータの`map()`で年齢を取り出し、`sum()`で合計を計算します。空の場合は0.0を返します

```rust
use std::env;
use std::fs::File;
use std::io::{BufRead, BufReader};

#[derive(Debug)]
struct User {
    id: u32,
    name: String,
    age: u32,
    city: String,
}

impl User {
    // TODO(human): CSV行からUserを生成するメソッドを実装
    // ヒント:
    // 1. line.split(',').collect() で Vec<&str> に分割
    // 2. fields.len() != 4 なら Err を返す
    // 3. fields[0].parse() で id をパース（parse::<u32>()）
    // 4. fields[1].to_string() で name を取得
    // 5. parse() が失敗したら .map_err(|_| "...") でエラーメッセージに変換
    fn from_csv_line(line: &str) -> Result<User, String> {
        // ここに実装
        todo!()
    }

    fn display(&self) {
        println!("{:<3} | {:<6} | {:<3} | {}",
                 self.id, self.name, self.age, self.city);
    }
}

fn load_users(filename: &str) -> Result<Vec<User>, String> {
    let file = File::open(filename)
        .map_err(|e| format!("Failed to open file: {}", e))?;

    let reader = BufReader::new(file);
    let mut users = Vec::new();

    for (i, line) in reader.lines().enumerate() {
        if i == 0 {
            continue;  // ヘッダー行をスキップ
        }

        let line = line.map_err(|e| format!("Failed to read line: {}", e))?;
        let user = User::from_csv_line(&line)?;
        users.push(user);
    }

    Ok(users)
}

// TODO(human): 特定の都市でフィルタリングする関数を実装
// ヒント:
// 1. users.iter() でイテレータを取得
// 2. .filter(|u| u.city == city) で都市が一致するものを抽出
// 3. .collect() で Vec<&User> に変換
// 4. ライフタイム 'a は「参照の生存期間」を表す（詳細は後で学ぶ）
fn filter_by_city<'a>(users: &'a [User], city: &str) -> Vec<&'a User> {
    // ここに実装
    todo!()
}

// TODO(human): 年齢の平均を計算する関数を実装
// ヒント:
// 1. users.is_empty() で空チェック（空なら0.0を返す）
// 2. users.iter().map(|u| u.age).sum() で合計を計算
// 3. sum as f64 / users.len() as f64 で平均を計算
fn calculate_average_age(users: &[User]) -> f64 {
    // ここに実装
    todo!()
}

fn main() {
    let args: Vec<String> = env::args().collect();

    if args.len() < 3 {
        eprintln!("Usage: csv_parser <command> <filename> [options]");
        eprintln!("Commands:");
        eprintln!("  show <filename>              - すべてのデータを表示");
        eprintln!("  filter <filename> <city>     - 都市でフィルタリング");
        eprintln!("  average <filename>           - 年齢の平均を計算");
        std::process::exit(1);
    }

    let command = &args[1];
    let filename = &args[2];

    let users = match load_users(filename) {
        Ok(u) => u,
        Err(e) => {
            eprintln!("Error: {}", e);
            std::process::exit(1);
        }
    };

    match command.as_str() {
        "show" => {
            println!("{:<3} | {:<6} | {:<3} | {}", "ID", "Name", "Age", "City");
            for user in &users {
                user.display();
            }
        }
        "filter" => {
            if args.len() < 4 {
                eprintln!("Usage: csv_parser filter <filename> <city>");
                std::process::exit(1);
            }
            let city = &args[3];
            let filtered = filter_by_city(&users, city);

            println!("{:<3} | {:<6} | {:<3} | {}", "ID", "Name", "Age", "City");
            for user in filtered {
                user.display();
            }
        }
        "average" => {
            let avg = calculate_average_age(&users);
            println!("Average age: {:.1}", avg);
        }
        _ => {
            eprintln!("Unknown command: {}", command);
            eprintln!("Available commands: show, filter, average");
            std::process::exit(1);
        }
    }
}
```

---

## 🐛 デバッグのヒント

実装中にエラーが出たら、[[デバッグガイド]]を参照してください。

よくあるエラー：

1. **`error[E0308]: mismatched types`** - 型が合わない
   - `parse()`の返り値は`Result`なので、`?`演算子で処理する必要があります

2. **`error[E0382]: borrow of moved value`** - 所有権が移動している
   - `for user in users` ではなく `for user in &users` を使います

3. **`fields[0]` でパニック** - インデックスが範囲外
   - 先に`fields.len() != 4`でチェックします

---

## 📚 学習ポイント

### derive属性

```rust
#[derive(Debug)]  // {:?} でデバッグ出力可能に
struct User {
    // ...
}

// 使用例
let user = User { ... };
println!("{:?}", user);  // User { id: 1, name: "Alice", ... }
```

**よく使う derive 属性：**
- `Debug` - デバッグ出力（`{:?}`）
- `Clone` - 明示的なクローン（`.clone()`）
- `PartialEq` - 等価比較（`==`, `!=`）
- `Serialize, Deserialize` - JSON変換（serdeクレート）

### イテレータの活用

```rust
// filter, map, collect のチェーン
let tokyo_names: Vec<String> = users.iter()
    .filter(|u| u.city == "Tokyo")
    .map(|u| u.name.clone())
    .collect();

// sum で合計
let total_age: u32 = users.iter()
    .map(|u| u.age)
    .sum();
```

**C言語との比較：**

```c
// C言語: ループで手動実装
int total_age = 0;
for (int i = 0; i < count; i++) {
    total_age += users[i].age;
}
```

```rust
// Rust: イテレータで簡潔に
let total_age: u32 = users.iter().map(|u| u.age).sum();
```

---

## 🚀 発展課題

### レベル1: ソート機能

```bash
$ cargo run -- sort users.csv age
ID | Name  | Age | City
3  | Carol | 22  | Tokyo
1  | Alice | 25  | Tokyo
5  | Eve   | 28  | Tokyo
2  | Bob   | 30  | Osaka
4  | Dave  | 35  | Nagoya
```

ヒント:
```rust
users.sort_by_key(|u| u.age);
```

### レベル2: 年齢範囲でフィルタ

```bash
$ cargo run -- filter users.csv --age 25-30
```

### レベル3: 都市別の集計

```bash
$ cargo run -- summary users.csv
City    | Count | Avg Age
Tokyo   | 3     | 25.0
Osaka   | 1     | 30.0
Nagoya  | 1     | 35.0
```

ヒント: `HashMap<String, Vec<&User>>` を使って都市ごとにグループ化

### レベル4: CSVへの書き出し

```rust
fn save_users(users: &[User], filename: &str) -> Result<(), String> {
    use std::io::Write;

    let mut file = File::create(filename)
        .map_err(|e| format!("Failed to create file: {}", e))?;

    writeln!(file, "id,name,age,city")?;

    for user in users {
        writeln!(file, "{},{},{},{}",
                 user.id, user.name, user.age, user.city)?;
    }

    Ok(())
}
```

---

<details>
<summary>📝 完全な解答例を見る（実装後に確認）</summary>

```rust
use std::env;
use std::fs::File;
use std::io::{BufRead, BufReader};

#[derive(Debug)]
struct User {
    id: u32,
    name: String,
    age: u32,
    city: String,
}

impl User {
    fn from_csv_line(line: &str) -> Result<User, String> {
        let fields: Vec<&str> = line.split(',').collect();

        if fields.len() != 4 {
            return Err(String::from("Invalid CSV format: expected 4 fields"));
        }

        Ok(User {
            id: fields[0].parse().map_err(|_| "Invalid ID")?,
            name: fields[1].to_string(),
            age: fields[2].parse().map_err(|_| "Invalid age")?,
            city: fields[3].to_string(),
        })
    }

    fn display(&self) {
        println!("{:<3} | {:<6} | {:<3} | {}",
                 self.id, self.name, self.age, self.city);
    }
}

fn load_users(filename: &str) -> Result<Vec<User>, String> {
    let file = File::open(filename)
        .map_err(|e| format!("Failed to open file: {}", e))?;

    let reader = BufReader::new(file);
    let mut users = Vec::new();

    for (i, line) in reader.lines().enumerate() {
        if i == 0 {
            continue;  // ヘッダー行をスキップ
        }

        let line = line.map_err(|e| format!("Failed to read line: {}", e))?;
        let user = User::from_csv_line(&line)?;
        users.push(user);
    }

    Ok(users)
}

fn filter_by_city<'a>(users: &'a [User], city: &str) -> Vec<&'a User> {
    users.iter()
        .filter(|u| u.city == city)
        .collect()
}

fn calculate_average_age(users: &[User]) -> f64 {
    if users.is_empty() {
        return 0.0;
    }

    let sum: u32 = users.iter().map(|u| u.age).sum();
    sum as f64 / users.len() as f64
}

fn main() {
    let args: Vec<String> = env::args().collect();

    if args.len() < 3 {
        eprintln!("Usage: csv_parser <command> <filename> [options]");
        eprintln!("Commands:");
        eprintln!("  show <filename>              - すべてのデータを表示");
        eprintln!("  filter <filename> <city>     - 都市でフィルタリング");
        eprintln!("  average <filename>           - 年齢の平均を計算");
        std::process::exit(1);
    }

    let command = &args[1];
    let filename = &args[2];

    let users = match load_users(filename) {
        Ok(u) => u,
        Err(e) => {
            eprintln!("Error: {}", e);
            std::process::exit(1);
        }
    };

    match command.as_str() {
        "show" => {
            println!("{:<3} | {:<6} | {:<3} | {}", "ID", "Name", "Age", "City");
            for user in &users {
                user.display();
            }
        }
        "filter" => {
            if args.len() < 4 {
                eprintln!("Usage: csv_parser filter <filename> <city>");
                std::process::exit(1);
            }
            let city = &args[3];
            let filtered = filter_by_city(&users, city);

            println!("{:<3} | {:<6} | {:<3} | {}", "ID", "Name", "Age", "City");
            for user in filtered {
                user.display();
            }
        }
        "average" => {
            let avg = calculate_average_age(&users);
            println!("Average age: {:.1}", avg);
        }
        _ => {
            eprintln!("Unknown command: {}", command);
            eprintln!("Available commands: show, filter, average");
            std::process::exit(1);
        }
    }
}
```

</details>

---

## 🔗 関連資料

- [[デバッグガイド]] - エラーが出た時の解決方法
- [The Rust Book: 構造体](https://doc.rust-jp.rs/book-ja/ch05-00-structs.html)
- [The Rust Book: エラー処理](https://doc.rust-jp.rs/book-ja/ch09-00-error-handling.html)

---

## 🎓 完了チェックリスト

- [ ] `struct` の定義と使用ができる
- [ ] `impl` ブロックでメソッドを定義できる
- [ ] CSVファイルの読み込みができる
- [ ] イテレータを使ったデータ操作ができる
- [ ] `Result`型でエラーハンドリングができる
- [ ] 発展課題に1つ以上チャレンジした

---

[[課題04-TODOリストCLI|次の課題: TODOリストCLI →]]
