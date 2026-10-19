# log — conformance/

- `docs.md` — the library's reference, in the pattern of botopink-lang's `conformance/docs.md`: one section per public
  module of `src/`, each describing in prose what the module offers and how it fails, no fenced `botopink` (every
  example is a cell, 461), the decision cells linked where they show a point:
  - `levels` — the five levels, ranks and spellings, per-name thresholds.
  - `formats` — the `LogRecord`, the four renderers, the timestamps.
  - `digest` — the one error digest and the client error body.
  - `sink` — the `LogSink`, the sink in force, the console sink, `fanOut`.
  - `logfile` — the file sink and its rotation.
  - `logging` — the `Logger`.
  - `reports` — the capture of the runtime's own reports.
- `modules/decisions-taken/<slug>-<N>/` — front 166's decision cells (decision 651: the subtree is that front's alone): one
  per code-bearing row of `../decisions-taken.md`, a `docs.md` (first line generated) and, in phase 1, `draft/main.bp` (decision
  649; no `src/`, no manifest, so `botopink cell check` finds no cell there); kept by `scripts/decision-cells.py` of the meta
  repository and its CI check 7.
- `modules/botopink.json` — the workspace manifest (`"workspaces": ["*"]`, log's targets, `log` by `"path": "../.."`) that
  `decision-cells.py compile` reads for the drafts' dependencies.
