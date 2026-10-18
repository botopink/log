# Decision 337 — The shorthand import goes; `mod m;` binds the namespace `m` in the module that declares it.

The example is refused. `main.bp` writes two shorthand imports, items with no module path and no
`from`. `import {errorDigest};` names a function the dependency `log` exports, but a bare name
never reaches a dependency's modules (nor std's): no module of this package declares
`errorDigest`, so the item is refused as an unresolved import, located at it — the program has to
write `import {digest.errorDigest} from "log";`. `import {loggerName};` names a function one
module of this package declares `pub`, so the refusal at that item writes its fix,
`import {naming.loggerName};`.

Both refusals are `shorthand-import`, one per item, and the program builds once each item names
where its name comes from. The decision's second and third parts (`mod m;` binding a namespace)
are not shown: every file is a module and no `mod` is written (decision 592).

Proving cells: none in this repository yet; log has no `conformance/` cells of its own, and they come with front 156's Conformance step (decision 465), which writes the cells its behaviour calls for under this `conformance/`.

[draft/main.bp](draft/main.bp)

[draft/naming.bp](draft/naming.bp)
