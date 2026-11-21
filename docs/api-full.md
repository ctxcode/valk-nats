
# Documentation

Namespaces: [main](#main)

---

# main

## Functions for 'main'

```js
// Parses a `nats://` or `tls://` URL into a `Config`.
+ fn config_from_url(text: String, timeout_ms: uint (5000)) Config !Error
// Connects to a NATS server described by `url` (default `nats://127.0.0.1:4222`).
+ fn connect(url: String ("nats://127.0.0.1:4222"), timeout_ms: uint (5000)) Client !Error
// Connects to a NATS server using `config`.
+ fn connect_with(config: Config) Client !Error
// Returns whether `queue` is a valid NATS queue group name (non-empty and free of whitespace).
+ fn is_valid_queue(queue: String) bool
// Returns whether `subject` is a valid NATS subscription subject (allows `*` and trailing `>` wildcards).
+ fn is_valid_sub_subject(subject: String) bool
// Returns whether `subject` is a valid NATS publish subject (no spaces, empty tokens, or wildcards).
+ fn is_valid_subject(subject: String) bool
// Returns whether a NATS subscription `pattern` (supporting `*` and `>`) matches `subject`.
+ fn match_subject(pattern: String, subject: String) bool
// Returns a unique reply subject such as `_INBOX.<random>`.
+ fn new_inbox(prefix: String ("_INBOX")) String
```

### config_from_url

Parses a `nats://` or `tls://` URL into a `Config`.

Supports credentials in the URL: `nats://user:pass@host:4222` sets `user` and `pass`,
while `nats://token@host:4222` sets `auth_token`. `tls://` enables TLS with default options.

### connect

Connects to a NATS server described by `url` (default `nats://127.0.0.1:4222`).

### connect_with

Connects to a NATS server using `config`.

### is_valid_queue

Returns whether `queue` is a valid NATS queue group name (non-empty and free of whitespace).

### is_valid_sub_subject

Returns whether `subject` is a valid NATS subscription subject (allows `*` and trailing `>` wildcards).

### is_valid_subject

Returns whether `subject` is a valid NATS publish subject (no spaces, empty tokens, or wildcards).

### match_subject

Returns whether a NATS subscription `pattern` (supporting `*` and `>`) matches `subject`.

### new_inbox

Returns a unique reply subject such as `_INBOX.<random>`.

## Classes for 'main'

```js
// A NATS client connection with background message dispatching.
+ class Client {
    // Whether the client has been closed or disconnected.
    ~ closed: bool
    // Settings this client was opened with.
    + config: Config
    // Whether `drain` is currently in progress.
    ~ draining: bool
    // Latest server metadata received from `INFO`.
    ~ info: ServerInfo
    // Most recent `-ERR` message received from the server, or `""`.
    ~ last_error: String
    // Whether this connection is running over TLS.
    ~ tls_active: bool

    // Immediately closes the connection and every active subscription.
    + fn close() void
    // Connects to a NATS server described by `url` (such as `nats://127.0.0.1:4222` or `tls://user:pass@host:4222`).
    + static fn connect(url: String ("nats://127.0.0.1:4222"), timeout_ms: uint (5000)) Client !Error
    // Connects to a NATS server using `config`, performs the `INFO` / TLS / `CONNECT` handshake, and starts the background reader coroutine.
    + static fn connect_with(config: Config) Client !Error
    // Gracefully drains all active subscriptions, flushes in-flight messages into their queues, and closes the connection.
    + fn drain(timeout_ms: uint (5000)) void !Error
    // Sends a `PING` to the server and waits for the matching `PONG` round-trip.
    + fn flush(timeout_ms: uint (5000)) void !Error
    // Publishes `payload` to `subject`, with optional `reply` subject and NATS/1.0 `headers`.
    + fn publish(subject: String, payload: String, reply: String (""), headers: ?Map[String] (null)) void !Error
    // Sends a request to `subject` and waits up to `timeout_ms` for a single reply message.
    + fn request(subject: String, payload: String, timeout_ms: uint (5000), headers: ?Map[String] (null)) Message !Error
    // Subscribes to `subject` (supporting `*` and `>` wildcards), optionally in `queue` group.
    + fn subscribe(subject: String, queue: String ("")) Subscription !Error
}
```

### Client

A NATS client connection with background message dispatching.

Supports pub/sub, queue groups, NATS/1.0 headers (`HPUB`/`HMSG`), request/reply over an
internal wildcard inbox, `PING`/`PONG` synchronization via `flush`, graceful `drain`,
and TLS upgrades.

#### closed

Whether the client has been closed or disconnected.

#### config

Settings this client was opened with.

#### draining

Whether `drain` is currently in progress.

#### info

Latest server metadata received from `INFO`.

#### last_error

Most recent `-ERR` message received from the server, or `""`.

#### tls_active

Whether this connection is running over TLS.

#### close

Immediately closes the connection and every active subscription.

#### connect

Connects to a NATS server described by `url` (such as `nats://127.0.0.1:4222` or
`tls://user:pass@host:4222`).

#### connect_with

Connects to a NATS server using `config`, performs the `INFO` / TLS / `CONNECT`
handshake, and starts the background reader coroutine.

#### drain

Gracefully drains all active subscriptions, flushes in-flight messages into their queues,
and closes the connection.

#### flush

Sends a `PING` to the server and waits for the matching `PONG` round-trip.

Guarantees that all prior `PUB`, `HPUB`, `SUB`, and `UNSUB` commands have been processed
by the server. Throws `timeout` if no `PONG` arrives within `timeout_ms`.

#### publish

Publishes `payload` to `subject`, with optional `reply` subject and NATS/1.0 `headers`.

#### request

Sends a request to `subject` and waits up to `timeout_ms` for a single reply message.

Uses an internal wildcard `_INBOX.<random>.*` subscription multiplexed across requests.

#### subscribe

Subscribes to `subject` (supporting `*` and `>` wildcards), optionally in `queue` group.

```js
// Settings for opening a NATS client connection.
+ class Config {
    // Token for token authentication.
    + auth_token: String
    // Whether messages published by this client are echoed back to its own subscriptions.
    + echo: bool
    // Advertise NATS 2.2+ header (`HPUB`/`HMSG`) support in `CONNECT`.
    + headers: bool
    // Host name or IP address of the NATS server.
    + host: String
    // Optional client name sent in `CONNECT`.
    + name: String
    // Password for user/pass authentication.
    + pass: String
    // Ask the server to run strict protocol checks.
    + pedantic: bool
    // Port of the NATS server.
    + port: u32
    // Connect, TLS handshake, and write timeout in milliseconds.
    + timeout_ms: uint
    // TLS settings, or null for a plain connection (still upgraded if server sets `tls_required`).
    + tls: ?TlsOptions
    // Username for user/pass authentication.
    + user: String
    // Ask the server to acknowledge every command with `+OK`.
    + verbose: bool
}
```

### Config

Settings for opening a NATS client connection.

#### auth_token

Token for token authentication.

#### echo

Whether messages published by this client are echoed back to its own subscriptions.

#### headers

Advertise NATS 2.2+ header (`HPUB`/`HMSG`) support in `CONNECT`.

#### host

Host name or IP address of the NATS server.

#### name

Optional client name sent in `CONNECT`.

#### pass

Password for user/pass authentication.

#### pedantic

Ask the server to run strict protocol checks.

#### port

Port of the NATS server.

#### timeout_ms

Connect, TLS handshake, and write timeout in milliseconds.

#### tls

TLS settings, or null for a plain connection (still upgraded if server sets `tls_required`).

#### user

Username for user/pass authentication.

#### verbose

Ask the server to acknowledge every command with `+OK`.

```js
// Thrown by every operation of this package.
+ union Error {
    connect(String)
    tls(String)
    auth(String)
    protocol(String)
    server(String)
    invalid_subject(String)
    timeout(String)
    closed(String)

    // Returns the error description carried by any variant.
    + get message: String
    + fn to_string() String
}
```

### Error

Thrown by every operation of this package.

- `connect(message)`: the TCP connection could not be opened, the URL was invalid, or reading/writing failed.
- `tls(message)`: the TLS handshake failed, or its settings could not be used.
- `auth(message)`: the server rejected the credentials or authorization failed.
- `protocol(message)`: the server sent unexpected wire bytes, or a message exceeded `max_payload`.
- `server(message)`: the server replied with a `-ERR` message.
- `invalid_subject(message)`: a subject, reply subject, or queue group was invalid.
- `timeout(message)`: `flush`, `next`, or `request` did not complete in time.
- `closed(message)`: the client or subscription is closed.

#### message

Returns the error description carried by any variant.

```js
// A message received from a NATS subscription or request reply.
+ class Message {
    // NATS/1.0 headers attached to the message (empty for plain `MSG` frames).
    + headers: Map[String]
    // Message payload bytes.
    + payload: String
    // Optional reply subject for request/reply; empty when absent.
    + reply: String
    // Subscription identifier that received this message.
    + sid: uint
    // Optional status code from the NATS/1.0 header line (for example `"503"`), or `""`.
    + status: String
    // Subject the message was published to.
    + subject: String

    // Returns the header value for `name`, or null when absent.
    + fn header(name: String) ?String
    // Publishes `payload` (and optional `headers`) back to `this.reply`.
    + fn respond(payload: String, headers: ?Map[String] (null)) void !Error
}
```

### Message

A message received from a NATS subscription or request reply.

#### headers

NATS/1.0 headers attached to the message (empty for plain `MSG` frames).

#### payload

Message payload bytes.

#### reply

Optional reply subject for request/reply; empty when absent.

#### sid

Subscription identifier that received this message.

#### status

Optional status code from the NATS/1.0 header line (for example `"503"`), or `""`.

#### subject

Subject the message was published to.

#### header

Returns the header value for `name`, or null when absent.

#### respond

Publishes `payload` (and optional `headers`) back to `this.reply`.

Throws `invalid_subject` when the message has no reply subject, or `closed` if the
client is closed.

```js
// Server metadata received in the NATS `INFO` message.
+ class ServerInfo {
    // Client ID assigned by the server.
    + client_id: uint
    // Additional cluster server URLs advertised by the server.
    + connect_urls: Array[String]
    // Whether the server supports NATS headers (`HPUB`/`HMSG`).
    + headers: bool
    // Host reported by the server.
    + host: String
    // Whether JetStream is enabled on the server.
    + jetstream: bool
    // Maximum payload size in bytes accepted by the server.
    + max_payload: uint
    // Random challenge nonce for NKey/JWT auth, when enabled.
    + nonce: String
    // Port reported by the server.
    + port: u32
    // Protocol version advertised by the server.
    + proto: int
    // Unique identifier of the server.
    + server_id: String
    // Human-readable server name.
    + server_name: String
    // Whether TLS is available on the server.
    + tls_available: bool
    // Whether the server requires a TLS upgrade before `CONNECT`.
    + tls_required: bool
    // Whether the server requests a client certificate during TLS.
    + tls_verify: bool
    // Server software version.
    + version: String
}
```

### ServerInfo

Server metadata received in the NATS `INFO` message.

#### client_id

Client ID assigned by the server.

#### connect_urls

Additional cluster server URLs advertised by the server.

#### headers

Whether the server supports NATS headers (`HPUB`/`HMSG`).

#### host

Host reported by the server.

#### jetstream

Whether JetStream is enabled on the server.

#### max_payload

Maximum payload size in bytes accepted by the server.

#### nonce

Random challenge nonce for NKey/JWT auth, when enabled.

#### port

Port reported by the server.

#### proto

Protocol version advertised by the server.

#### server_id

Unique identifier of the server.

#### server_name

Human-readable server name.

#### tls_available

Whether TLS is available on the server.

#### tls_required

Whether the server requires a TLS upgrade before `CONNECT`.

#### tls_verify

Whether the server requests a client certificate during TLS.

#### version

Server software version.

```js
// An active NATS subscription backed by a coroutine-friendly channel.
+ class Subscription {
    // True once the subscription is closed or drained.
    ~ closed: bool
    // Auto-unsubscribe limit set by `unsubscribe(max_msgs)`, or 0 when unlimited.
    ~ max_msgs: uint
    // Optional queue group name; empty for a normal subscription.
    + queue: String
    // Number of messages delivered to this subscription so far.
    ~ received: uint
    // Server-assigned numeric subscription ID.
    + sid: uint
    // The subject pattern this subscription listens on.
    + subject: String

    // Unsubscribes on the server, flushes in-flight messages into this subscription's queue, and closes it cleanly so remaining buffered messages can still be read with `next`.
    + fn drain(timeout_ms: uint (5000)) void !Error
    // Waits for and returns the next message on this subscription.
    + fn next(timeout_ms: uint (0)) Message !Error
    // Returns how many messages are currently buffered and ready for `next`.
    + fn pending() uint
    // Returns the next buffered message without waiting, or null when none is queued yet.
    + fn try_next() ?Message !Error
    // Unsubscribes from the subject.
    + fn unsubscribe(max_msgs: uint (0)) void !Error
}
```

### Subscription

An active NATS subscription backed by a coroutine-friendly channel.

Call `next` to receive incoming messages, `unsubscribe` to stop receiving (optionally after
`max_msgs` messages), or `drain` to process in-flight messages before closing.

#### closed

True once the subscription is closed or drained.

#### max_msgs

Auto-unsubscribe limit set by `unsubscribe(max_msgs)`, or 0 when unlimited.

#### queue

Optional queue group name; empty for a normal subscription.

#### received

Number of messages delivered to this subscription so far.

#### sid

Server-assigned numeric subscription ID.

#### subject

The subject pattern this subscription listens on.

#### drain

Unsubscribes on the server, flushes in-flight messages into this subscription's queue,
and closes it cleanly so remaining buffered messages can still be read with `next`.

#### next

Waits for and returns the next message on this subscription.

`timeout_ms` 0 waits forever. Throws `timeout` when `timeout_ms` passes without a message,
and `closed` once the subscription is unsubscribed/closed and all buffered messages have
been read.

#### pending

Returns how many messages are currently buffered and ready for `next`.

#### try_next

Returns the next buffered message without waiting, or null when none is queued yet.

Throws `closed` once the subscription is closed and empty.

#### unsubscribe

Unsubscribes from the subject.

When `max_msgs` is 0, unsubscribes immediately and discards buffered messages. When
`max_msgs` is greater than 0, tells the server to auto-unsubscribe after `max_msgs`
total messages have been received.

```js
// TLS settings for a NATS connection.
+ class TlsOptions {
    // A directory of certificate authorities to trust instead of the system bundle.
    + ca_dir: ?String
    // A PEM file with certificate authorities to trust instead of the system bundle.
    + ca_file: ?String
    // A PEM file with the client certificate, sent when the server requests one.
    + certificate_file: ?String
    // The hostname to verify the certificate against and send as SNI; empty uses the server host.
    + host: String
    // Password for an encrypted client private key.
    + key_password: String
    // The PEM private key for `certificate_file`; null reads it from `certificate_file`.
    + private_key_file: ?String
    // Whether the server certificate is verified.
    + verify: bool
}
```

### TlsOptions

TLS settings for a NATS connection.

The defaults verify the server certificate against the system CA bundle. For a self-signed
server certificate, pass `ca_file` or set `verify: false` (which disables certificate checks).
Set `certificate_file` and `private_key_file` for mutual TLS.

#### ca_dir

A directory of certificate authorities to trust instead of the system bundle.

#### ca_file

A PEM file with certificate authorities to trust instead of the system bundle.

#### certificate_file

A PEM file with the client certificate, sent when the server requests one.

#### host

The hostname to verify the certificate against and send as SNI; empty uses the server host.

#### key_password

Password for an encrypted client private key.

#### private_key_file

The PEM private key for `certificate_file`; null reads it from `certificate_file`.

#### verify

Whether the server certificate is verified.
