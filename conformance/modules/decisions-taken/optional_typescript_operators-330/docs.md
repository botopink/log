# Decision 330 — `?T` is read with TypeScript's operators and has no methods; `result` goes; a type declared in a type is its associated type.

The example reads optional values the way a program configures `log`.
`env.read("BOTOPINK_LOG_LEVEL_NOT_SET") ?? "info"` takes `info` when the variable is absent, `??`
being the whole fallback. `setOn` answers `?Threshold` — the threshold set on exactly one logger
name, `null` when there is none — and `main` reads it with the operators alone:
`db ?? levels.root` and `app ?? levels.root` fall back to the root threshold only for `app`, and
`db!` unwraps a value the program knows is set (on a `null` it would abort with the location and
the expression, the same on every target). `levels.names.at(0)?._0 ?? "(root only)"` reads the
first configured name through `?.`, the answer `?string`, flattened, never `??string`; on an
empty list it takes the fallback.

The last line reads a `Result`, not an optional: `parseThreshold("loud")` answers `Error`, and the
program asks it with its own methods, `isError()` and `unwrapOr(Threshold.Off)` — the methods stay
on `Result`, the operators on `?T`, and no `result.*` namespace is involved.

Proving cells: none in this repository yet; log has no `conformance/` cells of its own, and they come with front 156's Conformance step (decision 465), which writes the cells its behaviour calls for under this `conformance/`.

[draft/main.bp](draft/main.bp)
