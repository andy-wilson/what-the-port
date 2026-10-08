# Final fix report

## Finding 1: zero memory and CPU in wtp list
Change: WhatThePort/Sources/WhatThePort/Terminal/TerminalCommand.swift, in `list(json:)` after the two scans. If any server has a container and no container row has memory above 0, it polls with `usleep(500_000)` and `scanNow()` for up to 3 s. No change when there are no containers.
Manual check: `wtp list` against live containers (about 3 s total) prints non-zero memory and CPU, for example `:5432 m2a-postgres 46 MB 0.2%` and `:6379 m2a-redis 10 MB 0.4%`.

## Finding 2: pid 0 in wtp list --json
Change: extracted `TerminalCommand.jsonObject(_:status:)` in TerminalCommand.swift. For container rows it omits `pid` and emits `"processes": []`, and it keeps the `container` object. content/whattheport.md line 46 now lists `container` (`id`, `name`, `image`) and says container rows have no `pid` and are stopped with `docker stop <id>`.
Test: `testContainerJSONHasNoPid` in Tests/WhatThePortTests/TerminalTests.swift.
RED: the first run failed to compile with `type 'TerminalCommand' has no member 'jsonObject'` (log at /tmp/claude-501/wtp-red.log).
GREEN: 50 tests in 6 suites passed.
Manual check: `wtp list --json` printed `pid_present=False`, `processes=[]` and a non-zero `memoryBytes` for all 11 container rows, for example `5432 False [] 49010442 m2a-postgres`.

## Finding 3: pid 0 in the process lists
Changes: UI/ServerDetailView.swift skips the pid Text when `server.container != nil`. Terminal/TerminalApp.swift renders an empty padded pid cell for container rows, so the columns still line up. The change was not run in the GUI or the TUI, only compiled.

## Full suite
50 tests passed, up from the baseline of 49.
