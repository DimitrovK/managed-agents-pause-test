# managed-agents-pause-test

What I actually measured when running a coding agent on DigitalOcean's Managed
Agents (Harness Runtime), in public preview, on 29 September 2026.

The question was narrow: when the docs say a paused session preserves
"processes, memory, and the workspace filesystem", is that literally true?

So I started a process whose counter exists **only in memory**, never read back
from disk, paused the session, waited four and a half minutes, and resumed.

![the counter across the pause](images/pause.png)

```
47 08:05:42     <- paused here
48 08:10:10     <- resumed here, same PID
```

The counter continued from 47 to 48 with the same PID. The only trace of the
pause is a 4m28s hole in the timestamps.

![measured timings](images/timings.png)

Forking was the bigger surprise. Two children created from the running parent
both came up with the *same process at the same PID*, then diverged:

| | ticker PID | counter shortly after |
|---|---|---|
| parent | 590 | 244 |
| child 1 | 590 | 231 |
| child 2 | 590 | 228 |

## Files

- `agent.yaml` — the session manifest used for the pause test
- `locked.yaml` — the same thing with an egress allowlist, which flips the
  sandbox to deny-by-default
- `tick.log` — the raw counter log, including the gap

## Reproducing

```bash
doctl harness-runtime create -f agent.yaml
doctl harness-runtime pause <session-id>
doctl harness-runtime resume <session-id>
```

Total cost of everything here, four sessions including two forks: about 7 cents.

MIT licensed.
