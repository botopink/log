# log — conformance/

- `modules/decisions-taken/<slug>-<N>/` — front 166's decision cells (decision 651: the subtree is that front's alone): one
  per code-bearing row of `../decisions-taken.md`, a `docs.md` (first line generated) and, in phase 1, `draft/main.bp` (decision
  649; no `src/`, no manifest, so `botopink cell check` finds no cell there); kept by `scripts/decision-cells.py` of the meta
  repository and its CI check 7.
- `modules/botopink.json` — the workspace manifest (`"workspaces": ["*"]`, log's targets, `log` by `"path": "../.."`) that
  `decision-cells.py compile` reads for the drafts' dependencies.
