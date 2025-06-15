<h1 align="center">
    tower-sessions-ext
</h1>

<p align="center">
    🥠 Sessions as a `tower` and `axum` middleware.
</p>

<div align="center">
    <a href="https://crates.io/crates/tower-sessions-ext">
        <img src="https://img.shields.io/crates/v/tower-sessions-ext.svg" />
    </a>
    <a href="https://docs.rs/tower-sessions-ext">
        <img src="https://docs.rs/tower-sessions-ext/badge.svg" />
    </a>
    <a href="https://github.com/sagoez/tower-sessions-ext/actions/workflows/rust.yml">
        <img src="https://github.com/sagoez/tower-sessions-ext/actions/workflows/rust.yml/badge.svg" />
    </a>
    <a href="https://codecov.io/gh/sagoez/tower-sessions-ext" > 
        <img src="https://codecov.io/gh/sagoez/tower-sessions-ext/graph/badge.svg?token=74POF0TJDN"/> 
    </a>
</div>

## 🎨 Overview

This crate provides sessions, key-value pairs associated with a site
visitor, as a `tower` middleware.

It offers:

- **Pluggable Storage Backends:** Bring your own backend simply by
  implementing the `SessionStore` trait, fully decoupling sessions from their
  storage.
- **Minimal Overhead**: Sessions are only loaded from their backing stores
  when they're actually used and only in e.g. the handler they're used in.
  That means this middleware can be installed anywhere in your route
  graph with minimal overhead.
- **An `axum` Extractor for `Session`:** Applications built with `axum`
  can use `Session` as an extractor directly in their handlers. This makes
  using sessions as easy as including `Session` in your handler.
- **Simple Key-Value Interface:** Sessions offer a key-value interface that
  supports native Rust types. So long as these types are `Serialize` and can
  be converted to JSON, it's straightforward to insert, get, and remove any
  value.
- **Strongly-Typed Sessions:** Strong typing guarantees are easy to layer on
  top of this foundational key-value interface.

This crate's session implementation is inspired by the [Django sessions middleware](https://docs.djangoproject.com/en/4.2/topics/http/sessions) and it provides a transliteration of those semantics.

## 🔧 Maintenance

This crate is a maintained fork of the original [`tower-sessions`](https://github.com/maxcountryman/tower-sessions) project. We extend our sincere gratitude to [Max Countryman](https://github.com/maxcountryman) and all the original contributors for their foundational work and excellent design that made this project possible.

### Session stores

Session data persistence is managed by user-provided types that implement
`SessionStore`. What this means is that applications can and should
implement session stores to fit their specific needs.

That said, a number of session store implementations already exist and may be
useful starting points.

| Crate                                                                                                            | Persistent | Description                                                 |
| ---------------------------------------------------------------------------------------------------------------- | ---------- | ----------------------------------------------------------- |
| [`tower-sessions-ext-sqlx-store`](https://github.com/maxcountryman/tower-sessions-ext-stores/tree/main/sqlx-store)       | Yes        | SQLite, Postgres, and MySQL session stores                  |

Have a store to add? Please open a PR adding it.

## 📦 Install

To use the crate in your project, add the following to your `Cargo.toml` file:

```toml
[dependencies]
tower-sessions-ext = "0.14.0"
```

## 🤸 Usage

### `axum` Example

```rust
use std::net::SocketAddr;

use axum::{response::IntoResponse, routing::get, Router};
use serde::{Deserialize, Serialize};
use time::Duration;
use tower_sessions_ext::{Expiry, MemoryStore, Session, SessionManagerLayer};

const COUNTER_KEY: &str = "counter";

#[derive(Default, Deserialize, Serialize)]
struct Counter(usize);

async fn handler(session: Session) -> impl IntoResponse {
    let counter: Counter = session.get(COUNTER_KEY).await.unwrap().unwrap_or_default();
    session.insert(COUNTER_KEY, counter.0 + 1).await.unwrap();
    format!("Current count: {}", counter.0)
}

#[tokio::main]
async fn main() {
    let session_store = MemoryStore::default();
    let session_layer = SessionManagerLayer::new(session_store)
        .with_secure(false)
        .with_expiry(Expiry::OnInactivity(Duration::seconds(10)));

    let app = Router::new().route("/", get(handler)).layer(session_layer);

    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    let listener = tokio::net::TcpListener::bind(&addr).await.unwrap();
    axum::serve(listener, app.into_make_service())
        .await
        .unwrap();
}
```

You can find this [example][counter-example] as well as other example projects in the [example directory][examples].

> [!NOTE]
> See the [crate documentation][docs] for more usage information.

## 🦺 Safety

This crate uses `#![forbid(unsafe_code)]` to ensure everything is implemented in 100% safe Rust.

## 👯 Contributing

We appreciate all kinds of contributions, thank you!

[counter-example]: https://github.com/sagoez/tower-sessions-ext/tree/main/examples/counter.rs
[examples]: https://github.com/sagoez/tower-sessions-ext/tree/main/examples
[docs]: https://docs.rs/tower-sessions-ext
