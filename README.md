# valk-nats

Pure-Valk [NATS](https://nats.io) messaging client for [Valk](https://valk-lang.dev), built on `valk.net`, `valk.sync`, `valk.json`, and `valk.crypto`. No external C or OS dependencies beyond Valk's standard library.

Requires Valk 0.9.1 or newer.

## Install

```
vman install github.com/zikue/valk-nats
```

## Features

- NATS wire protocol v1 (`INFO`, `CONNECT`, `PUB`, `SUB`, `UNSUB`, `MSG`, `PING`, `PONG`, `+OK`, `-ERR`)
- NATS/1.0 headers (`HPUB` / `HMSG`)
- Exact, `*`, and `>` wildcard subscriptions with coroutine-friendly channel-backed `Subscription`
- Queue group subscriptions
- Fast request/reply (`request` + `Message.respond`) over a multiplexed wildcard `_INBOX`
- Auto-unsubscribe via `unsubscribe(max_msgs)`
- `PING`/`PONG` keepalive & `flush(timeout_ms)` round-trip synchronization
- Graceful `drain()` and immediate `close()`
- User/password and token authentication (via URL or `Config`)
- TLS upgrades (`tls://`, server `tls_required`, custom CA bundle, and client certificates)
- Subject validation and wildcard matching helpers (`is_valid_subject`, `is_valid_sub_subject`, `match_subject`, `new_inbox`)

## Example

```rust
use nats

fn main() {
    let client = nats.Client.connect("nats://127.0.0.1:4222") ! panic(E.message)
    defer client.close()

    // Subscribe and publish with headers
    let sub = client.subscribe("orders.*") ! panic(E.message)
    client.flush() ! panic(E.message)

    client.publish("orders.created", "{\"id\":42}", "", .{ "X-Trace" => "t-1" }) ! panic(E.message)
    let msg = sub.next(2000) ! panic(E.message)
    println(msg.subject + ": " + msg.payload + " trace=" + (msg.header("X-Trace") ?? ""))

    // Request / reply
    let svc = client.subscribe("math.double", "workers") ! panic(E.message)
    client.flush() ! panic(E.message)
    let worker = co serve(svc)

    let reply = client.request("math.double", "21", 2000) ! panic(E.message)
    println("result: " + reply.payload) // 42

    svc.unsubscribe() ! panic(E.message)
    await worker
    client.drain() ! panic(E.message)
}

fn serve(sub: nats.Subscription) {
    let req = sub.next(2000) ! return
    let n = req.payload.to_int() !? 0
    req.respond("" + (n * 2)) _
}
```

## Configuration & TLS

Connect with a URL (`nats://`, `tls://`, optional `user:pass@` or `token@`), or pass a `Config` for full control:

```rust
let client = nats.Client.connect_with(nats.Config {
    host: "nats.example.com"
    port: 4222
    name: "order-service"
    user: "app"
    pass: "secret"
    echo: false
    tls: nats.TlsOptions {
        ca_file: "/etc/ssl/nats-ca.pem"
    }
}) ! panic(E.message)
```

## API Reference

See [docs/api.md](docs/api.md) (summary) and [docs/api-full.md](docs/api-full.md) (full reference).

## Development

- `make test` — run the self-contained test suite (uses an in-process NATS server)
- `make lint` — lint `./src` with `--deny-warnings`
- `make example` — run `example/main.valk` against a local NATS server
- `make docs` — regenerate `docs/api.md` and `docs/api-full.md`
