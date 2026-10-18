# Decision 206 — `import {…} from "<name>"` resolves only to a package — std, a bundled package or a declared dependency.

The example is a package with two modules, and one of them is named `log` — the name of its
dependency. `main.bp` imports `errorDigest` and `clientErrorBody` with `from "log"`: that source
string names the package `log` and nothing else, so both come from the library's `digest`
module, not from the local module that shares the name. The local module is imported by its path
in braces with no `from` — `import {log.describeFault};` — and that is the only way to reach it.
`log.bp` itself does the same in the other direction: it takes `topFramesOf` from the dependency
with `from "log"`.

Run, the program prints the client body of a digest and the local description of the same
fault, its frames stripped by the dependency — two answers that show the two `log`s never mix.
Writing `from "<this package>"` for the local module is refused (`module-import-with-from`,
decision 309), its fix the brace form used here. Every file is a module (decision 592): the
package declares no `mod`.

Proving cells: none in this repository yet; log has no `conformance/` cells of its own, and they come with front 156's Conformance step (decision 465), which writes the cells its behaviour calls for under this `conformance/`.

[draft/main.bp](draft/main.bp)

[draft/log.bp](draft/log.bp)
