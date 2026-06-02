# @log/tracing-subscriber

A structured tracing subscriber for Zeta, ported from Rust's `tracing-subscriber` crate.

Provides subscriber implementations, filter directives (RUST_LOG-style), ANSI terminal
formatting, and log-crate compatibility — all without lifetimes or proc-macros.

## Features

- **FmtSubscriber** — drop-in subscriber with builder pattern
- **EnvFilter** — parse `RUST_LOG` directives (`info`, `crate=debug`, etc.)
- **Output formats** — Full, Compact, Pretty, and JSON
- **ANSI styling** — coloured log levels via `nu-ansi-term`
- **Layer system** — compose multiple subscribers with `Layer` trait + `Stack`
- **Registry** — track span lifecycles (enter/exit/clone/close)
- **Log compat** — bridge `log` crate records into tracing
- **Serde support** — serialize events & spans to JSON
- **Custom writers** — stdout, stderr, or custom `FmtWriter`

## Quick Start

```zeta
use @log/tracing_subscriber;

// Default subscriber (writes to stdout, INFO level and above)
FmtSubscriber::builder().init();

// With RUST_LOG-style filtering
let filter = EnvFilter::parse("my_crate=debug,other=info");
FmtSubscriber::builder()
    .with_env_filter(filter)
    .init();

// JSON output
FmtSubscriber::builder()
    .json()
    .init();

// Compact format (no target prefix)
FmtSubscriber::builder()
    .compact()
    .init();

// Pretty-printed with ANSI
FmtSubscriber::builder()
    .pretty()
    .init();
```

## FmtSubscriber Builder

| Method | Description |
|--------|-------------|
| `with_target(val)` | Show/hide the target module path |
| `with_level(val)` | Show/hide the log level |
| `with_line_number(val)` | Include source line numbers |
| `with_thread_ids(val)` | Show thread IDs |
| `with_thread_names(val)` | Show thread names |
| `json()` | Enable JSON output format |
| `compact()` | Compact format (no target prefix) |
| `pretty()` | Pretty-printed format with ANSI |
| `with_env_filter(filter)` | Apply an `EnvFilter` |
| `with_max_level(level)` | Set maximum log level |
| `with_writer(writer)` | Custom output writer |
| `init()` | Set as global default subscriber |

## EnvFilter

Supports standard `RUST_LOG` syntax:

```zeta
// Parse from environment (RUST_LOG)
let filter = EnvFilter::from_env();

// Parse explicit directives
let filter = EnvFilter::parse("warn,my_crate=debug,other=error");

// Build programmatically
let filter = EnvFilter::new()
    .with_directive("my_crate", "debug")
    .with_directive("other", "error")
    .with_default_level(LevelFilter::Warn);
```

**Directive precedence:** Module-specific directives override the default level.
If a target module matches multiple directives, the first match wins.

## Example

```zeta
use @log/tracing_subscriber;
use @log/tracing::{info, warn, error, debug};

FmtSubscriber::builder()
    .with_target(true)
    .with_level(true)
    .pretty()
    .init();

info!("Application started");
warn!("Low disk space: {}", "85%");
error!("Connection refused: {}", "192.168.1.1:8080");
```

Output (pretty mode with ANSI):
```
2025-01-01T00:00:00Z  INFO my_app: Application started
2025-01-01T00:00:00Z  WARN my_app: Low disk space: 85%
2025-01-01T00:00:00Z ERROR my_app: Connection refused: 192.168.1.1:8080
```

## Log Compatibility

Bridge `log` crate records into the tracing system:

```zeta
use @log/tracing_subscriber::log_compat::LogTracer;

// Install the log tracer
LogTracer::init();

// Now all `log` records are dispatched as tracing events
```

## Serde / JSON

Serialize spans and events to JSON:

```zeta
use @log/tracing_subscriber::serde_support;

let json = serde_support::serialize_event(event);
let span_json = serde_support::serialize_span(span_data);
```

## Layer System

Define custom layers to intercept span/event lifecycle:

```zeta
pub struct MyLayer;

impl Layer for MyLayer {
    fn on_new_span(&mut self, span: SpanData) {
        // Called when a new span is created
    }
    fn on_event(&mut self, event: Event) {
        // Called for each event
    }
    fn on_enter(&mut self, span_id: Id) {
        // Called when entering a span
    }
    fn on_exit(&mut self, span_id: Id) {
        // Called when exiting a span
    }
    fn on_close(&mut self, span_id: Id) {
        // Called when a span is closed
    }
    fn on_event_enabled(&mut self, metadata: Metadata) -> bool {
        return true;
    }
    fn enabled(&mut self, metadata: Metadata) -> bool {
        return true;
    }
}
```

## Bundled Sub-crates

This module bundles the following sub-crates as internal modules:

| Module | Description |
|--------|-------------|
| `tracing-subscriber` | Core subscriber types and builder |
| `matchers` | Pattern matching for filter directives |
| `nu-ansi-term` | ANSI terminal styling |
| `sharded-slab` | Concurrent slab storage for span data |
| `thread_local` | Thread-local storage wrapper |
| `tracing-log` | Log crate compatibility bridge |
| `tracing-serde` | Serde serialization support |

## License

Zeta Standard Library — MIT
