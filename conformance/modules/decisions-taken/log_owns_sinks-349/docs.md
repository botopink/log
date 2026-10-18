# Decision 349 — `log` owns the sinks; rakun and onze only integrate.

The example is what a framework does with `log` at boot, and nothing more. `boot` chooses two of
the sinks `log` ships and configures them: `consoleSink` in the `plain` format for errors only,
and `fileSink` in the `ecs` format, rotating at 1 MiB with two archives, under per-name levels —
`info` for every logger, `db` (and every `db.*` name) off, `http.access` from `warn`. `fanOut`
joins them into one sink, and `boot` answers it or the first refusal.

`main` first asks the joined sink what it takes: an error from `db.pool` (`true`: the console
takes every error, though the file's `db` threshold is off), an info from `http.access`
(`false` on both), a warning from `http.access` and an info from `app` (`true`, the file takes
them). It then installs the sink with `setSink` and calls `captureRuntimeReports()`, so the
runtime's own fault reports reach the same sink as records of the logger `runtime`. A `db` info
is dropped and the file is still absent (`-1` bytes); an `http.access` warning is appended to it.
A levels value with `db` set twice is refused by the sink that takes it, naming the reason, before
any line is written. The program removes its directory at the end.

Every sink, the per-name levels and the capture are `log`'s: the program writes no host cell and
no handler, and the same source runs on every target.

Proving cells: none in this repository yet; log has no `conformance/` cells of its own, and they come with front 156's Conformance step (decision 465), which writes the cells its behaviour calls for under this `conformance/`.

[draft/main.bp](draft/main.bp)
