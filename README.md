# fizz-memory-api

Fizz の **memory HTTP サービス**。memory 操作(note / recall / search / wipe /
users)を HTTP エンドポイントとして公開する。実体のイベントログは l2 と同じ
append-only JSONL(`FIZZ_MEMORY_PATH`、既定 `memory.jsonl`)に持つ。

ハンドラはリクエストごとにファイルを読み直す stateless 実装。ディスパッチ表
(`route`)とフィルタ/集計ロジックは `src/mod.almd`(transport 非依存)。

## エンドポイント

| メソッド + パス | 動作 | レスポンス |
|---|---|---|
| `POST /note` | 本文に `MemoryEvent` (JSON) → 追記(id 採番) | `{"ok":true,"id":N}` |
| `GET /recall?user_id=N&limit=M` | 該当ユーザの末尾 M 件(`limit<=0` で全件) | `{"user_id":N,"events":[...]}` |
| `GET /search?q=...` | content に `q` を含むイベント | `{"q":"...","events":[...]}` |
| `POST /wipe` | ログ全消去 | `{"ok":true}` |
| `GET /users` | 出現した user_id 一覧 | `{"users":[...]}` |

```sh
FIZZ_MEMORY_PORT=8088 ./fizz-memory-api &
curl -X POST -d '{"id":0,"user_id":7,"session_id":null,"role":"user","content":"猫が好き","ts":1,"tags":[]}' localhost:8088/note
curl 'localhost:8088/recall?user_id=7&limit=10'
curl 'localhost:8088/search?q=%E7%8C%AB'   # 猫
```

ポートは `FIZZ_MEMORY_PORT`(既定 8088)。`q` は URL デコードする(almide の
`http.query_params` がデコードしないため自前で復号 — [almide/almide#684](https://github.com/almide/almide/issues/684))。

## 開発

```sh
almide check src/main.almd
almide test
almide build src/main.almd -o build/fizz-memory-api
```

ツールチェーン: [almide](https://github.com/almide/almide) **v0.27.6+**
(`http.serve` が動く + `fizz_protocol` が通る最初の版)。

## 契約

[fizz-protocol](https://github.com/Aid-On/fizz-protocol) の `memory` モジュール
(`MemoryEvent` とその Codec)に依存。
