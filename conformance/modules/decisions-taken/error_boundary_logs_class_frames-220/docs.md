# Decision 220 — The runtime hands the error boundary the caught error's class and frames (erlang: `Class` and the stacktrace; node: `err

The example is an error boundary as jhonstart writes one. The runtime hands it what it caught as
`Caught`: the class and the frames — on erlang the `Class` of the exception and its stacktrace,
on node `err.name` and `err.stack` — and `boundary` calls `log`'s `logError` with the route
segment `app/blog/[slug]/page` as the module, the class, the message, the frames and its own
field `http.status`. A sink installed by the program prints each record.

The erlang fault is hit twice, its frames carrying other line numbers and one frame fewer: each
hit writes one error record with the frames as the runtime handed them, and both answer one
digest, the one `errorDigest` gives for the three frames with their line numbers stripped —
`log`'s rule, never the boundary's. The node fault (`TypeError`, a JavaScript stack whose frames
hold `:line:column`) goes through the same call and answers a digest of its own. The boundary
never computes a digest and renders nothing from the error but the digest it is answered.

Proving cells: none in this repository yet; log has no `conformance/` cells of its own, and they come with front 156's Conformance step (decision 465), which writes the cells its behaviour calls for under this `conformance/`.

[draft/main.bp](draft/main.bp)
