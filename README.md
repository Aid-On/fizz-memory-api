# fizz-memory-api

Fizz の **memory ディスパッチ**。memory 操作(note / recall / search / wipe /
users)を 1 つのエンドポイント群として公開する。実体のイベントログは l2 と同じ
append-only JSONL(`FIZZ_MEMORY_PATH`、既定 `memory.jsonl`)に持つ。

## トランスポートについて

本来の interface は HTTP(`POST /note`, `GET /recall?user_id=N` 等。ディスパッチ
表は `src/mod.almd` の `route` に定義)。ただし現状 almide の `http.serve` が
ハンドラ closure の表現不一致でビルドできない
([almide/almide#434](https://github.com/almide/almide/issues/434))。

直るまでの間、同じ操作を **NDJSON(stdin/stdout)** で公開する。route 表と純粋
ロジック(`recent_of` / `search` / `distinct_users` / `next_id`)は transport
非依存なので、`http.serve` が直れば HTTP 化は `main` の差し替えだけで済む。

## I/O(NDJSON)

```jsonc
{"op":"note","event":{"id":0,"user_id":7,"session_id":null,"role":"user","content":"猫が好き","ts":1,"tags":[]}}
{"op":"recall","user_id":7,"limit":10}
{"op":"search","q":"猫"}
{"op":"users"}
{"op":"wipe"}
```

- `note` → 追記(id 採番)。`{"ok":true,"id":N}`
- `recall` → 該当ユーザの末尾 `limit` 件。`{"user_id":N,"events":[...]}`
- `search` → content に `q` を含むイベント。`{"q":"...","events":[...]}`
- `users` → 出現した user_id 一覧。`{"users":[...]}`
- `wipe` → ログ消去。`{"ok":true}`

## 開発

```sh
almide check src/main.almd
almide test
almide build src/main.almd -o build/fizz-memory-api
```

ツールチェーン: [almide](https://github.com/almide/almide) v0.26.6+。

## 契約

[fizz-protocol](https://github.com/Aid-On/fizz-protocol) の `memory`
(`MemoryEvent` とその Codec)に依存。
