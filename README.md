![plugin-store](banner.png)

Simple, persistent key-value store.

## Install

_This plugin requires a Rust version of at least **1.64**_

There are three general methods of installation that we can recommend.

1. Use crates.io and npm (easiest, and requires you to trust that our publishing pipeline worked)
2. Pull sources directly from Github using git tags / revision hashes (most secure)
3. Git submodule install this repo in your tauri project and then use file protocol to ingest the source (most secure, but inconvenient to use)

Install the Core plugin by adding the following to your `Cargo.toml` file:

`src-tauri/Cargo.toml`

```toml
[dependencies]
tauri-plugin-store = { git = "https://github.com/tauri-apps/plugins-workspace", branch = "dev" }
```

You can install the JavaScript Guest bindings using your preferred JavaScript package manager:

> Note: Since most JavaScript package managers are unable to install packages from git monorepos we provide read-only mirrors of each plugin. This makes installation option 2 more ergonomic to use.

```sh
pnpm add https://github.com/tauri-apps/tauri-plugin-store
# or
npm add https://github.com/tauri-apps/tauri-plugin-store
# or
yarn add https://github.com/tauri-apps/tauri-plugin-store
```

## Usage

First you need to register the core plugin with Tauri:

`src-tauri/src/main.rs`

```rust
fn main() {
    tauri::Builder::default()
        .plugin(tauri_plugin_store::Builder::default().build())
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

Afterwards all the plugin's APIs are available through the JavaScript guest bindings:

```javascript
import { Store } from "tauri-plugin-store-api";

const store = new Store(".settings.dat");

await store.set("some-key", { value: 5 });

const val = await store.get("some-key");
assert(val, { value: 5 });
```

## Usage from Rust

You can also access Stores from Rust, you can create new stores:

```rust
use tauri_plugin_store::store::StoreBuilder;
use serde_json::json;

fn main() {
    tauri::Builder::default()
        .plugin(tauri_plugin_store::Builder::default().build())
        .setup(|app| {
            let mut store = StoreBuilder::new(app.handle(), "path/to/store.bin".parse()?).build();

            store.insert("a".to_string(), json!("b")) // note that values must be serd_json::Value to be compatible with JS
        })
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

As you may have noticed, the Store crated above isn't accessible to the frontend. To interoperate with stores created by JS use the exported `with_store` method:

```rust
use tauri::Wry;
use tauri_plugin_store::with_store;

let stores = app.state::<StoreCollection<Wry>>();
let path = PathBuf::from("path/to/the/storefile");

with_store(app_handle, stores, path, |store| store.set("a".to_string(), json!("b")))
```

## Contributing

PRs accepted. Please make sure to read the Contributing Guide before making a pull request.

## License

Code: (c) 2015 - Present - The Tauri Programme within The Commons Conservancy.

MIT or MIT/Apache 2.0 where applicable.
