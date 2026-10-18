# Decision 594 — Every library runs at build on the wasm comptime runtime (434), jhonstart first; a function only Erlang or Node can run is a located error there.

`log`'s digest, level and format functions are pure botopink — `topFramesOf`, `errorDigest`,
`clientErrorBody`, `levelName`, `otpLevel` and `formatName` read nothing but their arguments
and std — so they need no wasm form of their own: written under `comptime`, each call in the
example runs at build on the wasm comptime runtime, whatever `--target` the build has, and the
program carries string literals. It prints the kept frames `invoice.bp render.bp page.bp`, the
client body `{"digest":"…"}`, and `warn warning logstash`; the last line shows the build-time
digest equal to the one the same call answers at run time.

The other half of the rule is `log`'s host cells. A call under `comptime` that reaches one with no
wasm form — `logFileBytes`, which reaches `fileBytes` (the file system), or a renderer reaching
std's clock — is refused at build, located, naming the cell it reaches, and is never run on the
target's VM instead.

Proving cells: none in this repository yet; log has no `conformance/` cells of its own, and they come with front 156's Conformance step (decision 465), which writes the cells its behaviour calls for under this `conformance/`.

[draft/main.bp](draft/main.bp)
