# Decision 405 — For now, `log.fileSink` is one API on every target and answers a named `Error` on wasm — on both hosts.

The example is the decision's own program, written for the wasm target on both of its hosts.
`fileSink` is asked for a file sink on `app.log` (10 MiB, seven archives, the `ecs` format):
on wasm it answers `Error("log.fileSink: a wasm program has no file system - use consoleSink")`
at that call, at run time, writing nothing; the program prints the message and takes the
fallback the message names, `consoleSink` in the `plain` format. The record `log` writes next,
`started without a file`, reaches standard output through the console sink, so the program shows
both halves: `log` imports and logs on wasm, and the one call that cannot be served says so.

On erlang and commonJS the same program gets a real file sink and appends the line to `app.log`;
the cell's answer is the wasm one, so at promote its `"targets"` is `["wasm"]`.

Proving cells: none in this repository yet; log has no `conformance/` cells of its own, and they come with front 156's Conformance step (decision 465), which writes the cells its behaviour calls for under this `conformance/`.

[draft/main.bp](draft/main.bp)
