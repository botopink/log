# Decision 921 — `log`'s four choices stand: `consoleSink` writes one line per record on standard output through `terminal.print` on every target (606 (2)); `fileSink` rotates as OTP `logger_std_h` does and, on a failed host call, answers `Error` of `log`'s own error enum, never a raise (431, 486, 495); per-name levels as `levels.bp` has them; `captureRuntimeReports()` observes and never handles, a no-op binding on wasm (481).

Two cells prove the decision's second half, each a full cell over `log`'s file sink (`logfile.bp`), the four file
cells of which write `result: true` and answer `Result<_, LogError>`:

- [cells/rotation_keeps_two_archives](cells/rotation_keeps_two_archives/src/main.bp) appends to a file of `maxBytes` 10
  with `maxFiles` 2 until it has rotated three times: the live file, `<path>.0` and `<path>.1` are the files left, the
  oldest archive gone, and every `appendRotating` answered `Ok(0)`.
- [cells/unwritable_path_answers_error](cells/unwritable_path_answers_error/src/main.bp) writes under a path whose parent
  is a regular file: `appendRotating` answers `Error(AppendFailed)` instead of raising, and the sink `fileSink` builds answers `-1` for the record.

The cells run on erlang and commonJS, the targets of the workspace; wasm has no binding that reaches the file system
([decision 405](../log_file_sink_errors_on_wasm-405/docs.md)).
