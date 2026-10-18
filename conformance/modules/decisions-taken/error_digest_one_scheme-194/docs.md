# Decision 194 — One error-digest scheme with one implementation: the first 16 hex of `strongHash` over `module|errorClass|message|top

The example computes the digest of one fault three ways. The first two calls name the same
module, class and message, and frames that differ only in what the scheme throws away: the line
numbers (`invoice.bp:42` against `invoice.bp:44`), the whitespace between frames (spaces, a tab,
a newline) and every frame past the third (`router.bp:9`, `server.bp:120`). `topFramesOf` shows
what is kept, `invoice.bp render.bp page.bp`, so both calls hash the same input and print one
16-character digest — `first == again` is `true`. The third call changes the message, which is
part of the input, and `first == other` is `false`: the digest names a fault, not an occurrence.

The second half is the one call that both logs and answers. With a sink installed that prints
each record it takes, `Logger.logError` writes one error record — the message, then
`error.digest`, `error.module`, `error.type`, `error.stack_trace` with the frames as written, and
the caller's `invoice.id` — and answers the digest. That answer equals `first`, so the digest a
fallback page shows is the one on the log line, and `clientErrorBody` is all that crosses to the
client: `{"digest":"…"}`. No other function of the program, or of any library, computes it.

Proving cells: none in this repository yet; log has no `conformance/` cells of its own, and they come with front 156's Conformance step (decision 465), which writes the cells its behaviour calls for under this `conformance/`.

[draft/main.bp](draft/main.bp)
