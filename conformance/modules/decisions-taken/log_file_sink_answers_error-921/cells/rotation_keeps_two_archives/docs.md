# rotation_keeps_two_archives

A file of at most 10 bytes with two archives kept, written by `appendRotating` until it rotated three times, leaves
the live file, `<path>.0` and `<path>.1`; every write answered `Ok(0)`.
