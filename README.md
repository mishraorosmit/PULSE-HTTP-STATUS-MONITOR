# PULSE

> A small Rust tool that keeps an eye on your local services.

When you're building something locally, it's pretty normal to have a bunch of things running at once:

- frontend on `localhost:3000`
- backend on `localhost:8000`
- another API on `localhost:5000`
- maybe an auth server
- maybe a couple of Docker containers

And then, eventually, something stops working.

You open the browser.  
You refresh.  
You check the terminal.  
You restart something.  
You wonder which service actually died.

**PULSE exists to make that part less annoying.**

It periodically checks your services, measures how they're responding, and tells you what's alive and what's not.

---

## What is PULSE?

PULSE is a lightweight **HTTP and localhost service monitor written in Rust**.

You give it a set of endpoints to watch, and it checks them at regular intervals.

For each service, PULSE can keep track of things like:

```text
Is it reachable?
Did it return a valid response?
What HTTP status did it return?
How long did it take?
Did the request time out?
```

The idea is deliberately simple.

PULSE isn't trying to compete with Grafana, Prometheus, Datadog, or a full monitoring stack.

It's for the much smaller problem:

> **"I have a few services running. Are they actually alive?"**

---

## Why I built it

I wanted a Rust project that wasn't just another CLI calculator, todo app, or guessing game.

Something involving real networking seemed like a much better way to learn the language.

PULSE started from a pretty practical problem: when working on projects locally, I often have several services running at once, and figuring out which one stopped responding can be surprisingly annoying.

So I decided to build a small tool around that problem while learning how Rust handles:

- asynchronous code
- HTTP requests
- concurrency
- timeouts
- error handling
- configuration
- CLI applications

That is really the whole idea behind the project.

---

## What it does

At its core, PULSE watches HTTP endpoints.

For example:

```text
Frontend       http://localhost:3000
Backend        http://localhost:8000
Auth API       http://localhost:9000/health
```

A monitoring cycle might look something like:

```text
Frontend    → UP      14 ms    200
Backend     → UP      42 ms    200
Auth API    → DOWN    timeout
```

Instead of manually checking each service, PULSE does the checking for you.

---

## The basic idea

The monitoring loop is intentionally straightforward:

```text
Configured Services
        │
        ▼
   Start Checks
        │
        ├───────────────┐
        ▼               ▼
    HTTP Request    Timeout/Error
        │               │
        └───────┬───────┘
                ▼
           Check Result
                │
                ▼
             Output
```

Each service gets checked independently, which is especially useful when you're watching multiple local services at the same time.

---

## Example

A future/live dashboard-style output could look like:

```text
PULSE

● frontend    UP       12 ms    200
● backend     UP       41 ms    200
● auth        UP       67 ms    200
● payments    DOWN     timeout

3 / 4 services healthy
```

The exact interface is still evolving. The important part is that the information stays easy to read.

---

## Features

### HTTP health checks

PULSE can check HTTP and HTTPS endpoints and determine whether they're responding.

### Localhost support

It is especially useful for development services running on your own machine:

```text
http://localhost:3000
http://localhost:8000
http://127.0.0.1:8080/health
```

### Response time

A service being "up" doesn't always mean it's behaving well.

PULSE records how long a request takes, making it possible to notice slow services as well as completely dead ones.

### Status codes

A successful request and a server returning `500` are obviously not the same thing.

PULSE keeps the HTTP status visible so those cases don't get mixed together.

### Failure detection

Timeouts, connection failures, unreachable ports, and other request errors can be surfaced instead of leaving you wondering why something isn't working.

### Multiple services

The whole point is being able to watch several services together rather than checking them one by one.

---

## Why Rust?

Honestly, a monitoring tool is a pretty nice excuse to learn Rust.

PULSE relies on exactly the kind of things I wanted to understand better:

**async programming**

Several services need to be checked without blocking each other.

**networking**

The project deals with real HTTP requests rather than simulated input.

**timeouts**

Network requests can't be allowed to hang forever.

**concurrency**

Multiple checks can happen around the same time.

**error handling**

Networks fail. Services crash. Ports disappear. Rust makes you deal with those cases explicitly.

The project is as much about learning Rust properly as it is about building the tool itself.

---

## Tech Stack

The stack is intentionally small.

| Part | Technology |
|---|---|
| Language | Rust |
| Async runtime | Tokio |
| HTTP client | Reqwest |
| CLI | Clap |
| Serialization | Serde |
| Logging | Tracing |
| Build system | Cargo |

This may change as the project grows. I'd rather keep the dependency list small than add libraries just because they exist.

---

## Getting started

### Requirements

You'll need Rust installed.

The easiest way is through [rustup](https://rustup.rs/).

Check that everything is working:

```bash
rustc --version
cargo --version
```

### Clone the repository

```bash
git clone https://github.com/<your-username>/pulse.git
cd pulse
```

### Build it

```bash
cargo build
```

For a release build:

```bash
cargo build --release
```

### Run it

```bash
cargo run
```

To see the available options:

```bash
cargo run -- --help
```

> The CLI and configuration format are still being developed, so the exact commands may change.

---

## Configuration

The intended configuration is simple: tell PULSE what services you want to watch and how often to check them.

For example:

```toml
[[services]]
name = "Frontend"
url = "http://localhost:3000"
interval = 5

[[services]]
name = "Backend"
url = "http://localhost:8000/health"
interval = 5

[[services]]
name = "Auth"
url = "http://localhost:9000/health"
interval = 10
```

Nothing complicated.

Just:

**name → URL → check interval**

The configuration format may change while the project is being built.

---

## Project structure

The codebase is intentionally kept fairly simple.

The planned structure looks roughly like:

```text
pulse/
├── src/
│   ├── main.rs
│   ├── cli.rs
│   ├── config.rs
│   ├── monitor.rs
│   ├── checker.rs
│   ├── service.rs
│   └── output.rs
│
├── tests/
├── Cargo.toml
├── Cargo.lock
├── LICENSE
└── README.md
```

Some of these pieces may move around as the project develops.

---

## Development

A few commands I use regularly while working on PULSE:

```bash
cargo check
cargo test
cargo fmt
cargo clippy
```

Or, when I just want to run it:

```bash
cargo run
```

The general workflow is pretty simple:

```text
write code
   ↓
cargo check
   ↓
cargo test
   ↓
cargo fmt
   ↓
cargo clippy
```

---

## Roadmap

PULSE is still a work in progress.

The plan is to build it in small pieces rather than trying to throw everything into the first version.

### Core monitoring

- [x] Basic Rust project
- [ ] HTTP endpoint checks
- [ ] Concurrent checks
- [ ] Request timeouts
- [ ] Status-code validation
- [ ] Response-time measurement
- [ ] Clear CLI output

### Configuration

- [ ] Configuration file
- [ ] Custom check intervals
- [ ] Per-service timeouts
- [ ] Expected status codes
- [ ] Response-time thresholds

### Monitoring history

- [ ] Store check results
- [ ] Uptime calculation
- [ ] Latency statistics
- [ ] Failure history

### Better interface

- [ ] Live terminal dashboard
- [ ] Cleaner status indicators
- [ ] JSON output
- [ ] Exportable results

### Eventually

- [ ] TCP checks
- [ ] Process monitoring
- [ ] Docker/container checks
- [ ] Notifications
- [ ] Remote monitoring

Some of these may happen. Some may not.

I'd rather keep the project useful than keep adding features just to make the roadmap longer.

---

## What PULSE is not

PULSE isn't meant to be a production observability platform.

It doesn't try to replace:

- Prometheus
- Grafana
- Datadog
- New Relic
- distributed tracing systems
- enterprise infrastructure monitoring

Those tools solve much bigger problems.

PULSE is intentionally smaller.

Think of it as a **developer-side pulse check for your services**.

---

## Learning goals

One of the main reasons this project exists is to learn Rust by building something real.

Some of the things I'm exploring through PULSE:

```text
Rust
 ├── ownership & borrowing
 ├── error handling
 ├── modules
 ├── traits
 └── project structure

Async Rust
 ├── Tokio
 ├── tasks
 ├── timers
 └── concurrent work

Networking
 ├── HTTP
 ├── timeouts
 ├── connection failures
 └── service health

CLI development
 ├── arguments
 ├── configuration
 └── terminal output
```

So yes, PULSE is useful.

But it's also a learning project.

And that's intentional.

---

## Contributing

PULSE is primarily a personal project while I'm learning and experimenting with Rust, but contributions and ideas are welcome.

For larger changes, opening an issue first is usually better than immediately sending a huge pull request.

Basic setup:

```bash
git clone https://github.com/<your-username>/pulse.git
cd pulse

cargo build
cargo test
```

Then create your branch:

```bash
git checkout -b feature/my-feature
```

---

## License

PULSE is released under the **MIT License**.

See [`LICENSE`](LICENSE) for the full license text.

---

## Status

**PULSE is currently under development.**

The project is still evolving, especially the CLI, configuration system, and monitoring interface.

The goal isn't to build a massive monitoring platform.

The goal is to build a genuinely useful little tool, learn a lot of Rust along the way, and hopefully end up with something I'd actually keep running on my own machine.

---

## Author

**Orosmit Mishra**

Built with Rust and a suspicious number of services running on `localhost`.
