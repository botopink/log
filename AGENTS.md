# log

> Repository: `botopink/log` (`git@github.com:botopink/log.git`) · in the meta checkout: `repository/log/`

The `log` library (decisions 194 and 195): the five levels, the log
record, the ECS / GELF / logstash / plain renderers, the one error digest, and
a `Logger` whose sink is injected. One source compiled for erlang and
commonJS, so a record renders the same line and a fault the same digest on
both. The pure half of rakun-logging (`levels.bp`, `formats.bp`, `digest.bp`,
the `Logger`) moved here by `specs/1.0.11-beta/03-bundled-libs/106-log`;
rakun-logging's OTP cells stay rakun's, and rakun installs them as the sink.

Imports `std` and nothing else (`json`, `hash`, `io.clock`, `io.os`,
`io.process`). Ships `.bp` files only — target-native code is three inline
`#[@External.<Target>(…)]` templates in `sink.bp` (decision 117 rule 8).

**A library of its own** (decision 326). It was bundled with the compiler until
`03-bundled-libs/138` moved it here with its history; the compiler now embeds std alone.
A program that imports `from "log"` declares it in `dependencies` (decision 242) —
`{ "log": { "git": "https://github.com/botopink/log.git", "branch": "feat" } }`; inside the meta checkout that entry
resolves by name through the `repository/` root (`repository/log`), elsewhere
through the install store. Without the entry, `from "log"` is
`unresolved import source "log" — declare it in botopink.json "dependencies"`.
Both the
leaf form decision 195 writes, `import {errorDigest, Logger} from "log";`, and
the module-qualified form, `import {logging.Logger, digest.errorDigest} from
"log";`, resolve — measured from a scratch consumer on commonJS and erlang.

## Tree

```text
log/
├── botopink.json     "name": "log", "target": "erlang", "targets": ["erlang", "commonJS"], no dependencies
├── AGENTS.md         ← you are here
├── src/
│   ├── root.bp         pub mod levels; formats; digest; sink; logging
│   ├── levels.bp       Level { Trace, Debug, Info, Warn, Error }, levelName, levelRank, otpLevel,
│   │                   validLevels, rankOf, rankName
│   ├── formats.bp      LogRecord, Format { Ecs, Gelf, Logstash, Plain }, validFormats, parseFormat,
│   │                   formatName, isoTimestamp, gelfTimestamp, renderEcs, gelfLevel, gelfKey,
│   │                   renderGelf, logstashLevelValue, renderLogstash, renderPlain, renderRecord
│   ├── digest.bp       stripLineNumbers, topFramesOf, digestInput, errorDigest, clientErrorBody
│   ├── sink.bp         LogSink(enabled, write), defaultSink, setSink, currentSink   (the sink slot: templates)
│   └── logging.bp      Logger(name): trace/debug/info/warn/error (+ …With fields), log, isEnabled,
│                       lazily, logError
└── test/             digest · formats · levels · logging   (suites `digest:` `formats:` `levels:` `logging:`)
```

`botopink.json`'s `files` order is a dependency order: a module is listed before
the siblings that import it.

## The surface

```bp
pub type Level { Trace, Debug, Info, Warn, Error }
pub type LogRecord(millis: i64, level: Level, logger: string, message: string,
    fields: Array<#(string, string)>, trace: string, digest: string,
    pid: string, node: string, thread: string, app: string)
pub type Format { Ecs, Gelf, Logstash, Plain }
pub fn parseFormat(name: string) -> @Result<Format, string>
pub fn renderRecord(format: Format, r: LogRecord) -> string

pub fn errorDigest(module: string, errorClass: string, message: string, topFrames: string) -> string

pub type LogSink(enabled: fn(logger: string, level: Level) -> bool, write: fn(record: LogRecord) -> i32)
pub fn setSink(sink: LogSink) -> i32
pub fn defaultSink() -> LogSink

pub type Logger(name: string)
// Logger.logError(module, errorClass, message, topFrames, fields) -> string — the digest
```

- **No name std or a framework exports** (decision 163): the sink type is
  `LogSink` (rakun-stream has a `Sink`) and the dispatch `renderRecord`
  (rakun-actuator has a `render`). rakun-logging's own `Level`, `LogRecord`,
  `Logger`, `errorDigest`, … are the copies this package replaces.
- **The renderers** are rakun-logging's, field for field; the four lines its
  `format_test.bp` pins are pinned here unchanged (`test/formats_test.bp`), on
  both targets. The two timestamp cells it had are botopink over std
  `clock.toCivil` and the epoch reading's digits: UTC
  `YYYY-MM-DDTHH:MM:SS` + `Z`, or `.mmmZ` when there is a fraction; GELF's
  seconds with `.mmm` only when there is one. A reading before the epoch
  panics (`log: a record's millis must be at least 0, got <n>`) — the hosts do
  not round a negative reading alike. A schema is the `Format` enum, so
  `renderRecord` is total; a spelling is refused once, by `parseFormat`
  (`log.parseFormat: "<name>" is not a log format - accepted: ecs, gelf, logstash, plain`).
- **The digest** (decision 194): the first 16 hex of std `hash.strongHash`
  over `module|errorClass|message|topFrames`, the frames normalised (the first
  three whitespace-separated, `:<digits>` stripped). Known answers, on both
  targets: `errorDigest("app.billing.invoices", "gateway_declined", "charge
  declined", "InvoiceService.charge/2:88 InvoiceRepo.charge/2:41
  rakun.core.router.dispatch/1:12 …")` is `90e4cc2a07abe1fb` (rakun-logging's
  pinned value) and `errorDigest("", "", "", "")` is `be5be69f55e91af2` —
  each the SHA-256 of the documented input (`printf '%s' '<input>' |
  sha256sum`). No library computes a digest any other way.
- **The sink** (decision 195): `enabled(logger, level)` is asked before a
  record is built, `write(record)` gets the whole record and renders it
  itself. With none set, `defaultSink()`: `info` and above, the `plain` line,
  through OTP `logger` at `otpLevel(level)` on erlang (OTP's own primary
  level still applies — its default `notice` drops `info`) and `console` on
  node (`console.error` / `console.warn` / `console.log`).
- **`Logger.logError`** writes ONE error record — the message, then the fields
  `error.module`, `error.type`, `error.stack_trace` and the caller's, under
  `error.digest` — and answers the digest, also when the sink takes no
  errors. It is the render's way to the logger: there is no
  `RenderHooks.onError`.
- **The record's facts** a `Logger` fills: `millis` from `clock.nowMillis`,
  `pid` the OS process id (`io.process.pid`, imported as `host` — a
  module-level `process` shadows Node's global), `node` the host name,
  `thread` `main`; `app` and `trace` are `""`. A sink that knows an
  application name or a correlation id writes its own.

## Host cells

| State | Erlang | Node |
|---|---|---|
| the sink slot (`putSink` / `sinkOr`) | `persistent_term` entry `{log, sink}` — set once at boot, read by every process | `sink` on `globalThis.__bp_log` (`{ sink: null }`, created by whichever template runs first) |
| the default write (`hostWrite`) | `logger:log(<otp level>, "~ts", [Line])` | `console.error` / `console.warn` / `console.log` |

Erlang template variables are spelled `Lg<Name>__`, every template a `fun`
applied in place.

## Targets

`targets` is `["erlang", "commonJS"]` and both are tested. Measured from a
consumer: `botopink run --target beam` runs it too (beam compiles the Erlang
templates; the digest and a `logError` through a set sink answer as on
erlang), untested by `botopink test`, which runs no beam. `--target wasm` is
refused at the first std cell without a wasm binding the package reaches
(`` `quote` has no `#[@External.<Target>(…)]` for the wasm backend ``, from
`std/json.bp`), never compiled silently.

## Testing

```sh
../botopink-lang/zig-out/bin/botopink test --target erlang
../botopink-lang/zig-out/bin/botopink test --target commonJS
../botopink-lang/zig-out/bin/botopink format --check src test
```

Tests import the package's modules by their path inside the braces
(`import {formats.renderEcs};`, decision 206). The sink is global host
state and outlives a test: every test that sets one puts `defaultSink()` back
before it asserts; `test/logging_test.bp` captures through three test-local
cells. An epoch reading is built with `clock.parseIso8601` (an `i64` has no
literal). Every expected text is a literal; an expected line holding a JSON
escape is a quoted string, not a `\\` line, because a `\\` line reads `\n` as
the escape.

## Local gate

`scripts/git-hooks/pre-commit` is the tracked pre-commit gate, self-contained:
it sources `scripts/git-hooks/lib/runner-standalone.sh` from this repository and
reaches nothing outside it, so a standalone clone, a checkout inside the botopink
meta workspace and a worktree run the same gate. Install it once per clone:

```sh
git config core.hooksPath scripts/git-hooks
```

The repository is one plain package, so the gate's test stage runs
`botopink test --target <t>` at the root on each target `botopink.json` declares
(`erlang`, `commonJS`). Never commit with `--no-verify`; fix the red instead.
`scripts/git-hooks/pre-commit` and `scripts/git-hooks/lib/runner-standalone.sh`
are one text across every library repository: the meta repository's
`hook-integrity` workflow compares the bytes (its check 4), so a change to either
lands in all of them together. CI: `.github/workflows/test.yml` runs the same
package on linux and macos, on each declared target, with the compiler built from
`botopink/botopink-lang` `feat`.
