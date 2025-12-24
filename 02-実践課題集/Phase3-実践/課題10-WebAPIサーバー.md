# 課題10: WebAPIサーバー

[[課題09-CLIツール|← 前の課題]]  |  [[README|トップに戻る]]

**難易度:** ⭐️⭐️⭐️⭐️⭐️
**学習時間目安:** 15-20時間
**学習トピック:** axum/actix-web、非同期、データベース、REST API

---

## 🎯 課題の目的

Rustで実用的なWebバックエンドを構築し、非同期処理、データベース連携、REST APIの設計を学びます。

---

## 📋 プロジェクト: シンプルなTODO API

### エンドポイント

```
GET    /todos           - すべてのTODOを取得
GET    /todos/:id       - 特定のTODOを取得
POST   /todos           - 新しいTODOを作成
PUT    /todos/:id       - TODOを更新
DELETE /todos/:id       - TODOを削除
```

---

## 🏗️ 実装ガイド（axum使用）

### 依存関係

```toml
# Cargo.toml
[dependencies]
axum = "0.7"
tokio = { version = "1", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
sqlx = { version = "0.7", features = ["runtime-tokio-native-tls", "sqlite"] }
```

### 基本構造

```rust
use axum::{
    extract::{Path, State},
    http::StatusCode,
    routing::{get, post},
    Json, Router,
};
use serde::{Deserialize, Serialize};
use std::sync::Arc;
use tokio::sync::Mutex;

#[derive(Debug, Serialize, Deserialize, Clone)]
struct Todo {
    id: u64,
    title: String,
    completed: bool,
}

#[derive(Clone)]
struct AppState {
    todos: Arc<Mutex<Vec<Todo>>>,
}

#[tokio::main]
async fn main() {
    let state = AppState {
        todos: Arc::new(Mutex::new(Vec::new())),
    };

    let app = Router::new()
        .route("/todos", get(get_todos).post(create_todo))
        .route("/todos/:id", get(get_todo).put(update_todo).delete(delete_todo))
        .with_state(state);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    println!("Server running on http://localhost:3000");

    axum::serve(listener, app).await.unwrap();
}

async fn get_todos(State(state): State<AppState>) -> Json<Vec<Todo>> {
    let todos = state.todos.lock().await;
    Json(todos.clone())
}

async fn get_todo(
    State(state): State<AppState>,
    Path(id): Path<u64>,
) -> Result<Json<Todo>, StatusCode> {
    let todos = state.todos.lock().await;

    todos
        .iter()
        .find(|t| t.id == id)
        .cloned()
        .map(Json)
        .ok_or(StatusCode::NOT_FOUND)
}

#[derive(Deserialize)]
struct CreateTodo {
    title: String,
}

async fn create_todo(
    State(state): State<AppState>,
    Json(payload): Json<CreateTodo>,
) -> (StatusCode, Json<Todo>) {
    let mut todos = state.todos.lock().await;

    let id = todos.len() as u64 + 1;
    let todo = Todo {
        id,
        title: payload.title,
        completed: false,
    };

    todos.push(todo.clone());

    (StatusCode::CREATED, Json(todo))
}

async fn update_todo(
    State(state): State<AppState>,
    Path(id): Path<u64>,
    Json(payload): Json<Todo>,
) -> Result<Json<Todo>, StatusCode> {
    let mut todos = state.todos.lock().await;

    let todo = todos
        .iter_mut()
        .find(|t| t.id == id)
        .ok_or(StatusCode::NOT_FOUND)?;

    todo.title = payload.title;
    todo.completed = payload.completed;

    Ok(Json(todo.clone()))
}

async fn delete_todo(
    State(state): State<AppState>,
    Path(id): Path<u64>,
) -> StatusCode {
    let mut todos = state.todos.lock().await;

    if let Some(pos) = todos.iter().position(|t| t.id == id) {
        todos.remove(pos);
        StatusCode::NO_CONTENT
    } else {
        StatusCode::NOT_FOUND
    }
}
```

---

## 💡 テスト方法

### curlでのテスト

```bash
# TODOを作成
curl -X POST http://localhost:3000/todos \
  -H "Content-Type: application/json" \
  -d '{"title":"Rustを学ぶ"}'

# すべてのTODOを取得
curl http://localhost:3000/todos

# 特定のTODOを取得
curl http://localhost:3000/todos/1

# TODOを更新
curl -X PUT http://localhost:3000/todos/1 \
  -H "Content-Type: application/json" \
  -d '{"id":1,"title":"Rustをマスターする","completed":true}'

# TODOを削除
curl -X DELETE http://localhost:3000/todos/1
```

---

## 🚀 発展課題

### レベル1: SQLiteデータベース連携

```rust
use sqlx::SqlitePool;

#[derive(sqlx::FromRow)]
struct Todo {
    id: i64,
    title: String,
    completed: bool,
}

async fn get_todos(pool: &SqlitePool) -> Vec<Todo> {
    sqlx::query_as("SELECT * FROM todos")
        .fetch_all(pool)
        .await
        .unwrap()
}
```

### レベル2: 認証・認可

```toml
[dependencies]
jsonwebtoken = "9"
```

### レベル3: ページネーション

```rust
#[derive(Deserialize)]
struct Pagination {
    page: Option<u32>,
    per_page: Option<u32>,
}
```

### レベル4: Docker化

```dockerfile
FROM rust:1.75 as builder
WORKDIR /app
COPY . .
RUN cargo build --release

FROM debian:bookworm-slim
COPY --from=builder /app/target/release/todo-api /usr/local/bin/
CMD ["todo-api"]
```

---

## 🔗 関連資料

- [axum Documentation](https://docs.rs/axum/)
- [SQLx Documentation](https://docs.rs/sqlx/)
- [REST API Best Practices](https://restfulapi.net/)

---

## 🎓 完了チェックリスト

- [ ] REST APIの基本を理解した
- [ ] 非同期Webサーバーを構築できる
- [ ] データベース連携ができる
- [ ] エラーハンドリングができている
- [ ] 実際にAPIとして動作する

おめでとうございます！すべてのPhaseを完了しました！🎉

---

[[README|トップに戻る]]
