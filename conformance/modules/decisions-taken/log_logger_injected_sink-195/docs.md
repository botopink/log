# Decision 195 — `log` — `import {errorDigest, Logger} from "log";` — levels, `LogRecord`, four renderers, `errorDigest` and a `Logger` w

The decision writes `import {errorDigest, Logger} from "log";`; under decision 592 that leaf form is
refused as written, so the example imports each name from the module that declares it
(`errorDigest` from `log/digest`, `Logger` from `log/logging`): 195's spelling is an example of its
time. The record, the levels and the sink slot are imported by their modules. It installs a sink of its own with
`setSink`: `enabled` takes debug and above, and `write` prints each record on one line (level,
logger, message, digest, fields) and answers `1`.

The logger `app.billing` then writes through that sink. `trace` is below the sink's threshold, so
the sink's `enabled` answers `false` before any record is built and the call answers `0`;
`infoWith` passes, the sink prints `info app.billing charging invoice.id=7`, and the call answers
the write's `1`. `isEnabled` asks the same question without writing (`false` for trace, `true`
for debug), and `lazily` never runs its builder for a level that is off. `logError` writes one
error record through the injected sink and answers its digest, the same one `errorDigest`
computes from the same four strings. Last, `setSink(defaultSink())` puts back the sink in force
when none is set — the host's logger, OTP `logger` on erlang and `console` on node — which
receives every record a program writes before installing its own.

The program reads the same on every target: nothing in it names a host.

Proving cells: none in this repository yet; log has no `conformance/` cells of its own, and they come with front 156's Conformance step (decision 465), which writes the cells its behaviour calls for under this `conformance/`.

[draft/main.bp](draft/main.bp)
