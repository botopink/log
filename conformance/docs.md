# log — conformance

> Path: `repository/log/conformance/`
> Sibling (AGENTS): [`./AGENTS.md`](AGENTS.md)

This reference describes `log`: levels and per-name thresholds, the log record and its four renderers, the one error
digest, the `Logger` over an injected sink, the sinks and the capture of the runtime's own reports (decisions 194, 195,
349). It is pure botopink with host cells, compiled from one source for erlang and commonJS, so a line and a digest read
the same everywhere; it imports `std` and nothing else. A program names each item from the module that declares it,
`import {digest.errorDigest} from "log"; import {logging.Logger} from "log";` (decisions 206, 337, 763). Every example
is a conformance cell (decision 461); the library holds no feature cell yet, and the decision cells below show the
points the decisions fix, each as a `draft/main.bp` until front 166 promotes it to a cell (decision 649). A table is
fenced as `text`.

| Section | What |
|---|---|
| *levels* | the five levels, their ranks and spellings, per-name thresholds (`Threshold`, `Levels`) |
| *formats* | the `LogRecord`, the `Format` enum, the four renderers and the timestamps |
| *digest* | the one error digest and the client error body |
| *sink* | the `LogSink`, the sink in force, the console sink and `fanOut` |
| *logfile* | the file sink with rotation |
| *logging* | the `Logger` |
| *reports* | the capture of the runtime's own reports |
| Decisions | `modules/decisions-taken/`: The ledger's index: one `<slug>-<N>/` per code-bearing row of this repository's generated `decisions-taken.md` (front 166, decisions 646–651), its `docs.md` walking through one example program |

The package declares `pub mod` lines in `src/root.bp`, an `"entry"` and a `"files"` list in `botopink.json`, and its
sources write `Array<T>` in places; the language has retired these (every `.bp` under `src/` is a module, a library has
no entry, the manifest names no files, and the type is `T[]`; decisions 592, 733, 703, 641). The library's codemod is
pending, and this reference describes the API in the spelling in force. The comments of `sink.bp` name `@print`, which
is `terminal.print` now (decision 606).

The decision cells that show the library as a whole: [log_logger_injected_sink-195](modules/decisions-taken/log_logger_injected_sink-195/draft/main.bp) (a `Logger` over an injected
sink), [log_owns_sinks-349](modules/decisions-taken/log_owns_sinks-349/draft/main.bp) (what a framework does with `log` at boot), [libraries_run_at_build_on_wasm-594](modules/decisions-taken/libraries_run_at_build_on_wasm-594/draft/main.bp)
(a library runs at build on the wasm comptime runtime), [own_module_import_without_from-206](modules/decisions-taken/own_module_import_without_from-206/draft/main.bp) and
[shorthand_import_refused-337](modules/decisions-taken/shorthand_import_refused-337/draft/main.bp) (how a module of a package is named), and [optional_typescript_operators-330](modules/decisions-taken/optional_typescript_operators-330/draft/main.bp)
(`?T` is read with operators, as the optional answers of this library are).

## levels

The five levels and the thresholds a program sets over its loggers (decisions 195, 349).

`levels.Level` is `Trace`, `Debug`, `Info`, `Warn` or `Error`, ranked 0 to 4.

- `levels.levelName(level)` answers the lowercase name (`trace`, `debug`, `info`, `warn`, `error`);
  `levels.levelRank(level)` the rank; `levels.rankName(rank)` the uppercase name the renderers write, `OFF` for a rank
  that is none of the five.
- `levels.otpLevel(level)` answers the OTP `logger` severity a level is written at. The mapping is stated, not
  inferred, and `Trace` and `Debug` both become `debug`, so a sink that filters must filter on the rank before the host
  call, as `LogSink.enabled` does.
- `levels.rankOf(raw)` answers the rank a spelling names, `-1` for one that is not a level. `warning` is accepted as
  OTP's name for `warn`; `off` is 5 (a threshold, never the level of a record); case and surrounding whitespace do not
  matter. `levels.validLevels()` lists the accepted spellings for a diagnostic.

A `Threshold` is the lowest level a logger name (or a sink) takes, `From(level)`, or `Off`.
`levels.parseThreshold(name)` answers `Result<Threshold, string>`: the threshold a spelling names, or an `Error`
naming the spelling and the accepted ones. `levels.thresholdAdmits(threshold, level)` answers whether a record at
`level` passes.

`levels.Levels(root, names)` holds the thresholds of a program's loggers: `root`, and one `#(name, threshold)` per name.
A threshold set on a name applies to it and to every dotted name beneath it, the longest prefix set wins, and `root`
applies when none is.

- `levels.levelsProblem(levels)` answers the first reason the levels cannot be resolved, or `""`: an empty name, a name
  with an empty segment (`a..b`, `.a`, `a.`), a name set twice. Nothing guesses which entry wins.
- `levels.thresholdFor(levels, logger)` answers the threshold in force for a logger; a prefix set twice panics, naming
  it.
- `levels.levelEnabled(levels, logger, level)` answers whether a record at `level` from `logger` passes the levels in
  force.

Where the names and their thresholds come from, rakun's typed configuration and its run-time overrides, is the
framework's: it builds a `Levels` and installs a sink over it again when one changes
([log_owns_sinks-349](modules/decisions-taken/log_owns_sinks-349/draft/main.bp)).

## formats

The record and its four output schemas (decision 195).

`formats.LogRecord(millis, level, logger, message, fields, trace, digest, pid, node, thread, app)` is one record:
`millis` the epoch reading as `i64`, `fields` a list of `#(string, string)` pairs, and `trace`, `digest` and `app`
empty when there is none. `formats.Format` is `Ecs`, `Gelf`, `Logstash` or `Plain`; the schema is an enum, so
`formats.renderRecord(format, record)` is total.

- `formats.parseFormat(name)` answers `Result<Format, string>`: exactly `ecs`, `gelf`, `logstash` or `plain`; anything
  else is an `Error` naming it and the four values (`formats.validFormats()`). `formats.formatName(format)` is its
  inverse.
- `formats.renderEcs`, `renderGelf`, `renderLogstash` and `renderPlain` write one record in one schema.
  `formats.renderRecord` dispatches on the `Format`.

The field order within a schema is fixed, so two lines of one schema diff cleanly. An optional field with no value
(`service.name` with no application name, `trace.id` outside any correlation scope, `error.digest` on a record that is
not a fault) is left out rather than written empty, and never moves another field:

```text
ecs       @timestamp log.level message service.name process.thread.name log.logger trace.id error.digest + fields
gelf      version host short_message timestamp level _logger _trace_id _error_digest + _-fields
logstash  @timestamp @version message logger_name level level_value error_digest + fields
plain     <timestamp> <LEVEL> <pid> --- [<app>] [<thread>] <logger> : <message>[ | k=v,…]
```

JSON strings go through std's RFC 8259 quoting, so a message cannot break a line or a record. `plain` is for a human and
escapes only line breaks, so one record stays one line.

- `formats.gelfLevel(level)` answers GELF 1.1's syslog severity as text (error 3, warning 4, info 6, debug 7);
  `formats.gelfKey(key)` answers an additional field's key: `_`-prefixed, with every character outside `[A-Za-z0-9-]`
  folded to `_` (`invoice.id` is `_invoice_id`), so a collector does not read the dot as nesting.
- `formats.logstashLevelValue(level)` answers Logback's level integer as text (`TRACE` 5000 up to `ERROR` 40000).
- `formats.isoTimestamp(millis)` answers UTC `YYYY-MM-DDTHH:MM:SS` then `Z` on a whole second and `.mmmZ` otherwise;
  `formats.gelfTimestamp(millis)` the seconds, with `.mmm` only when there is a fraction. Both are botopink over std's
  calendar, so the line is the same on erlang and commonJS. A record before the epoch is refused with a panic naming the
  reading: the two hosts do not round a negative reading alike, and no clock answers one.

## digest

The error digest: one scheme, one implementation (decision 194). The digest is the only part of a server error that
crosses to the client: `Logger.logError` writes one error record holding the full message, the class, the module and
the frames, and answers the digest. No library computes a digest any other way ([error_digest_one_scheme-194](modules/decisions-taken/error_digest_one_scheme-194/draft/main.bp)).

The digest is the first 16 hex characters of std's `hash.strongHash` over `module|errorClass|message|topFrames`, where
`topFrames` is the first three whitespace-separated frames with their line numbers (`:<digits>`) stripped. The timestamp,
the request id, the node and any argument value are not in the input: the digest names a fault, not an occurrence, so a
thousand hits of one bug are one digest on every node and every target of the build, and it changes when the code does.
Argument values travel as record fields, which the digest never reads.

- `digest.errorDigest(module, errorClass, message, topFrames)` answers the 16 lowercase hex characters.
- `digest.digestInput(module, errorClass, message, topFrames)` answers the text that is hashed;
  `digest.topFramesOf(frames)` the first three frames joined by one space; `digest.stripLineNumbers(frame)` removes
  `:<digits>` wherever it appears (`invoice.bp:42` is `invoice.bp`) and keeps a `:` not followed by a digit.
- `digest.clientErrorBody(digest)` answers the whole payload a renderer may send to the client for a server error,
  `{"digest":"<digest>"}`.

The error boundary hands the runtime's class and frames to `Logger.logError` ([error_boundary_logs_class_frames-220](modules/decisions-taken/error_boundary_logs_class_frames-220/draft/main.bp)).

## sink

Where a record goes: the injected sink (decisions 195, 349). A framework chooses where a line is written and which
levels are on, and hands both to the package at boot with `sink.setSink`; rakun and `rakun-ssr` only choose, configure and
install a sink ([log_owns_sinks-349](modules/decisions-taken/log_owns_sinks-349/draft/main.bp)).

`sink.LogSink(enabled, write)` is any pair of functions. `enabled(logger, level)` is asked before a record is built, so
a message for a level that is off is never rendered; `write(record)` receives the whole record and renders it itself (a
sink may write one schema to the console and another to a file) and answers a count.

- `sink.defaultSink()` is in force until one is set: levels `info` and above, the `plain` line, written through the
  host's logger. On erlang that is OTP `logger` at `otpLevel(level)`, so OTP's own primary level still applies (its
  default `notice` drops `info`); on commonJS `console.error` for error, `console.warn` for warn and `console.log`
  below; on wasm the line on standard output, on both hosts.
- `sink.setSink(sink)` installs a sink for every later record, in every process, and answers 0.
  `sink.currentSink()` answers the sink in force, the last one set or `defaultSink()`. The sink is host state: botopink
  has no top-level mutable state, so erlang keeps it in `persistent_term` and commonJS on `globalThis`.
- `sink.consoleSink(format, levels)` answers `Result<LogSink, string>`: a sink writing each record it takes as one
  `format` line on standard output, a record passing when `levels` admits its logger and level. It is an `Error` when
  `levelsProblem(levels)` is not empty.
- `sink.fanOut(sinks)` answers one sink over several: a record is wanted when any sink takes it, and each sink that takes
  it writes it, in list order. Its write answers the sum of the writes.

On wasm no binding holds a value across calls, so the sink slot is refused at its declaration and never answered
wrongly (the source marks it `// LANGUAGE GAP`); `consoleSink` works there.

## logfile

A file sink with rotation (decision 349). `logfile.LogFile(path, maxBytes, maxFiles)` names the file, the size that
rotates it and the archives kept; `logfile.fileSink(file, format, levels)` answers `Result<LogSink, string>`: a sink
appending each record it takes to `file.path` as one `format` line, creating the directories on the way, and rotating
the file. It is an `Error` when `logFileProblem(file)` or `levelsProblem(levels)` is not empty.

- `logfile.logFileProblem(file)` answers the first reason the file cannot be written, or `""`: an empty path, a
  `maxBytes` below 1, a negative `maxFiles`.
- Rotation is `logger_std_h`'s, written once so every target rotates alike. After a write leaves the file at `maxBytes`
  or more (`logfile.appendRotating(file, line)`), `logfile.rotate(file)` renames the files as below, and with `maxFiles`
  0 the file is deleted and starts again; the live file and its archives never hold more than `maxFiles + 1` files.
  `logfile.archiveName(path, index)` answers `<path>.<index>`, `.0` the newest. `logfile.logFileBytes(path)` answers the
  size in bytes, `-1` when there is no file.

```text
<path>.<maxFiles - 1>   deleted
<path>.<i>              becomes <path>.<i + 1>      for i from maxFiles - 2 down to 0
<path>                  becomes <path>.0
```

Every host call fails loudly: a line that cannot be appended, a size that cannot be read (other than a file that is not
there), or a rename or delete that fails raises on the host, because a log that silently stops writing is a lost log. A
file that is not there reads `-1` bytes and is skipped by a rename or a delete. The total size cap that rakun folds into
the archive count stays rakun's; it hands `fileSink` the folded count.

`log.fileSink` is one API on every target and, for now, answers a named `Error` on wasm, on both hosts, at the one call
that cannot be served (decision 405, [log_file_sink_errors_on_wasm-405](modules/decisions-taken/log_file_sink_errors_on_wasm-405/draft/main.bp)). The source still marks its four file cells
`// LANGUAGE GAP` and refuses them on wasm at their declaration; the named `Error` waits on that gap's fix.

## logging

The `Logger` (decisions 194, 195): one type, five levels, over the sink in force. `logging.Logger(name)` is built with
its logger name (`Logger(name: "invoice")`) and named in `levels` by it.

- `trace`, `debug`, `info`, `warn` and `error` take a message; `traceWith`, `debugWith`, `infoWith`, `warnWith` and
  `errorWith` take a message and a list of `#(string, string)` fields; `log(level, message, fields)` takes the level as a
  value. Every write answers what the sink's `write` answered, and 0 for a dropped record.
- The level check is the sink's `enabled`, asked before the record is built, so a record below the sink's threshold is
  dropped without its message being rendered or the host being called. `isEnabled(level)` exists so an expensive message
  is not built for a level that is off, and `lazily(level, build)` is the same guard with the builder passed in: `build`
  runs only when the level is on for the logger.
- Fields are a list, never interpolated into the sentence: a structured log whose values live inside the message is an
  unstructured log.
- `logError(module, errorClass, message, topFrames, fields)` is the one way a fault reaches the log. It writes one error
  record holding the full message, `error.module`, `error.type`, `error.stack_trace` and the caller's fields, under
  `error.digest`, and answers the digest, so the digest a fallback shows is the one on the log line. The digest is
  answered whether or not the sink takes errors. rakun-bpp's error boundary calls it; there is no render hook.

The record's facts: `millis` from std's clock, `pid` the OS process id, `node` the host name, `thread` `main`; `app` and
`trace` are empty, and a sink that knows an application name or a correlation id writes its own record. Of rakun's
logging cells only the correlation id stays rakun's (decision 349). The example of a `Logger` over an injected sink is
[log_logger_injected_sink-195](modules/decisions-taken/log_logger_injected_sink-195/draft/main.bp).

## reports

The runtime's own reports, through the sink in force (decision 349). `reports.captureRuntimeReports()` makes the faults
the runtime reports by itself reach the log as records of the logger `runtime`, at the moment they are reported, through
the sink in force then. Each record carries the field `report.kind`. It does not change what the runtime does with the
fault: the host's own report and the host's own exit stay. Calling it again keeps one capture, and it answers 0.

- On BEAM (erlang and beam) a primary filter on OTP's `logger` takes every event of the `otp` domain (crash reports,
  supervisor reports, progress reports, an unlinked process's error), formats its message on one line and writes it at
  its level (`emergency` to `error` is error, `warning` is warn, `notice` and `info` are info, `debug` is debug),
  `report.kind` the `error_logger` type or the domain. OTP's primary level still applies before it.
- On commonJS an `uncaughtExceptionMonitor` listener, which observes without handling so the process still dies as node
  decides, writes an error record: the message the error's text, `report.kind` the origin (`uncaughtException`, or
  `unhandledRejection`), and the error's stack under `error.stack_trace` when it has one.
- On wasm the binding captures nothing: a wasm program has no runtime reporting faults beside it, and a trap ends the
  instance.

`reports.writeRuntimeReport(level, kind, message, stack)` writes one such record: logger `runtime`, field
`report.kind`, and `error.stack_trace` when the host handed a stack apart from the message. A level name the host
reports that is none of `debug`, `info`, `warn` or `error` panics, naming it.
