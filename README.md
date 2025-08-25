# WebSocket リアルタイムゲームサーバー

RustとWebSocketを使用したリアルタイムオンラインゲーム用サーバーです。

## 機能

- WebSocketベースのリアルタイム通信
- 複数クライアントの同時接続サポート
- プレイヤーの移動・アクション処理
- チャット機能
- ゲーム状態の同期
- 高性能な非同期処理 (tokio)
- スレッドセーフなクライアント管理

## セットアップ

### 1. Rustのインストール
```bash
# Rustupを使用してRustをインストール
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.cargo/env

# バージョン確認
rustc --version
cargo --version
```

### 2. プロジェクトのビルドと実行
```bash
# 依存関係をダウンロードしてビルド
cargo build

# サーバーを実行
cargo run
```

サーバーは `127.0.0.1:8080` で起動します。

### 3. テストクライアント

`test_client.html` をブラウザで開いてサーバーをテストできます。
- 複数のブラウザタブで開くことで、複数プレイヤーをシミュレート可能
- キーボード操作: WASD/矢印キー（移動）、スペース（攻撃）、E（アイテム取得）

## アーキテクチャ

### モジュール構成

- `main.rs` - エントリーポイント、WebSocketサーバーの起動
- `server.rs` - ゲームサーバーのメイン処理
- `client.rs` - クライアント接続の管理
- `game.rs` - ゲーム状態とロジック
- `message.rs` - メッセージ定義とプレイヤー構造体

### 主要な依存関係

- `tokio` - 非同期ランタイム
- `tokio-tungstenite` - WebSocket実装
- `serde` + `serde_json` - JSONシリアライゼーション
- `dashmap` - 並行安全なHashMap
- `uuid` - ユニークID生成
- `tracing` - ログ出力

### メッセージフォーマット

```rust
pub enum GameMessage {
    PlayerJoin { player_id: Uuid, name: String },
    PlayerLeave { player_id: Uuid },
    PlayerMove { player_id: Uuid, x: f32, y: f32 },
    PlayerAction { player_id: Uuid, action: String, data: Value },
    GameState { players: Vec<Player>, timestamp: u64 },
    Chat { player_id: Uuid, message: String },
    Error { message: String },
}
```

## 使用方法

### サーバー起動
```bash
cargo run
```

### クライアント接続
```javascript
const ws = new WebSocket('ws://127.0.0.1:8080');

// プレイヤー移動
ws.send(JSON.stringify({
    PlayerMove: {
        player_id: "your-player-id",
        x: 100.0,
        y: 200.0
    }
}));

// チャット送信
ws.send(JSON.stringify({
    Chat: {
        player_id: "your-player-id",
        message: "Hello, World!"
    }
}));
```

## カスタマイズ

### ゲームロジックの追加
`src/game.rs`の`handle_player_action`関数でアクション処理を追加:

```rust
async fn handle_player_action(&self, client_id: Uuid, action: &str, data: &serde_json::Value) {
    match action {
        "attack" => {
            // 攻撃処理
        },
        "jump" => {
            // ジャンプ処理
        },
        _ => {}
    }
}
```

### フレームレートの調整
`src/game.rs`の`tick_rate`を変更:

```rust
tick_rate: Duration::from_millis(16), // 60 FPS
tick_rate: Duration::from_millis(33), // 30 FPS
```

## パフォーマンス

- 非同期処理によるスケーラブル設計
- DashMapによる並行安全なクライアント管理
- 効率的なメッセージブロードキャスト
- メモリ効率的な構造体設計

## ログ

tracing crateを使用したログ出力:
```bash
# 詳細ログ有効化
RUST_LOG=debug cargo run
```

このサーバーは実際のゲーム開発の基盤として使用でき、具体的なゲームロジックを追加することで様々な種類のリアルタイムゲームを構築できます。