
# Documentation

Namespaces: [main](#main)

---

# main

## Errors for 'main'

```js
// Thrown by every operation of this package.
+ error Error (connect, tls, auth, protocol, server, invalid_subject, timeout, closed) payload { message: String }
```

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
