# unclaude

Claude Code says a lot. This tells you what it actually said.

Under each substantial answer you get a short plain-English version of it. And whenever
you want it on purpose — posting a comment on a task, say — there is a `/unclaude` skill
you can call.

The rewrite is written by **your own Claude subscription** — no API key, no second
provider, nothing to install and run locally.

![A long Claude answer beside its plain-English rewrite](docs/comparison.png)

---

## It cannot affect Claude

This matters more than anything else here, so it comes first.

The block is **display-only**. It never reaches Claude's context, your transcript, or
`/resume`. Claude's reasoning, its answers, and everything it does next are byte-for-byte
what they would have been without this plugin installed. Nothing is added to a prompt,
nothing is fed back to the model.

That isn't a promise — it's a property of the hook it uses, and Claude Code will confirm
it for you:

```
$ claude plugin details unclaude

  Hooks (1)  MessageDisplay  (harness-only — no model context cost)
```

*harness-only — no model context cost*, in the client's own accounting.
→ [How it works](docs/how-it-works.md) for why, and how to verify it yourself.

---

## Install

Paste this repo's URL to Claude Code and say "install it". Or run it yourself:

```sh
claude plugin marketplace add https://github.com/sed1108/unclaude
claude plugin install unclaude@unclaude --yes
```

Then **restart Claude Code**. Hooks load at session start, so the session you installed
from won't show anything.

Needs Claude Code logged in with a subscription, and Python 3. Nothing to configure.

---

## The block takes a few seconds

Worth knowing before you install, because it is the one thing you will notice.

Claude's answer streams at full speed exactly as it always has. The block then appears
underneath it — **usually 8–15 seconds later**, because a second, small model has to read
the finished answer and write the rewrite. It is faster when Anthropic's API is quiet
(7–9s) and slower when it is busy; 20s+ happens.

So you read the answer, and the plain-English version lands shortly after. Nothing is held
back while it thinks.

If the rewrite takes longer than 25 seconds it is abandoned and Claude's original text
stands on its own. You lose the block, never the answer.

Short messages and messages that are mostly code are skipped entirely, and cost nothing.
The threshold is tunable — see [Configuration](docs/configuration.md).

---

## `/unclaude`, for text on its way out

For text that leaves the session. You did the work; now it has to go somewhere other
people read.

```
Post the summary to the ticket with /unclaude
```

What gets posted:

> Rewrote the rate limiter to use sliding window log with Redis backing; 12 new tests
> pass, full suite is green.
>
> The previous implementation used a fixed window algorithm, which has a known weakness: a
> client can issue a full quota of requests immediately before the reset and another full
> quota immediately after, allowing up to 2x the intended rate to slip through in a short
> burst.
>
> Changed the RateLimiter class in src/middleware/rate_limit.py to keep a timestamped log
> of recent requests and count only those inside the trailing window, eliminating the
> boundary problem.
>
> The previous implementation kept counters in process memory. Since the API runs on 4
> pods behind the load balancer, each pod independently enforced the limit, making the
> effective rate roughly 4x the configured rate. Added Redis-backed storage so all pods
> share a single view of the counters, using the existing Redis connection pool.
>
> Wrote 12 new tests in tests/test_rate_limit.py covering basic allow/deny behaviour at
> and around the limit, the boundary case that fixed-window failed on, concurrent access
> from multiple simulated pods, Redis connection failure (fails open, doesn't take down
> the API), and window expiry and log trimming. All tests pass and the full existing suite
> is green.
>
> The burst_allowance config option is currently ignored by the new implementation.
> Supporting it would require the config schema to change so that burst size and burst
> window can be expressed independently. This touches shared config validation code that
> other services depend on, so it's tracked separately as a follow-up.

Note what **survived**: every file path, the 4 pods, the 12 tests, the 2x and 4x rate
leaks, and the unfinished `burst_allowance` — stated plainly instead of apologised for.
Shortening is not the goal. Cutting words while keeping every fact is.

<details>
<summary><b>What Claude would have posted</b> (click to expand)</summary>

> All done! 🎉 I've gone ahead and completed the work on the rate limiter, and I'm happy to report that everything is passing. Here's a comprehensive summary of what I did and why.
>
> **Background**
>
> Before making any changes, I wanted to make sure I properly understood the existing implementation, so I spent some time reading through the current code. What I found was that the rate limiter was using a **fixed window** algorithm.
>
> As you may already know, fixed window rate limiting has a well-documented weakness: because the counter resets at a hard boundary, a client can issue a full quota of requests immediately before the reset and another full quota immediately after. In the worst case this allows up to **2x the intended rate** to slip through in a short burst. Given that this limiter is protecting our public API, that seemed worth addressing properly rather than papering over.
>
> **What I changed**
>
> I've refactored the implementation to use a **sliding window log** approach instead. Here's a breakdown of the changes:
>
> **1. Core algorithm**
>
> I rewrote the `RateLimiter` class in `src/middleware/rate_limit.py`. The new implementation keeps a timestamped log of recent requests and counts only those falling inside the trailing window, which eliminates the boundary problem entirely.
>
> **2. Shared state across pods**
>
> I also noticed — and I think this is worth flagging as a somewhat separate issue — that the previous implementation kept its counters **in process memory**. Since we run 4 API pods behind the load balancer, this meant each pod was independently enforcing the limit, so the effective rate was roughly 4x what was configured.
>
> To address this, I've added Redis-backed storage so that all pods share a single view of the counters. I used the existing Redis connection pool rather than introducing a new dependency, so there's nothing new to configure or deploy.
>
> **3. Tests**
>
> I wrote **12 new tests** in `tests/test_rate_limit.py` covering:
>
> - Basic allow/deny behaviour at and around the limit
> - The boundary case that the old fixed-window implementation failed
> - Concurrent access from multiple simulated pods
> - Redis connection failure (fails open, so a Redis outage doesn't take down the API)
> - Window expiry and log trimming
>
> All 12 pass, and I also ran the full existing suite to make sure nothing regressed — **the whole suite is green**.
>
> **One thing I wasn't able to finish**
>
> I do want to be upfront about one item that I wasn't able to fully resolve.
>
> The `burst_allowance` config option is currently **ignored** by the new implementation. Supporting it properly would require a change to the configuration schema — specifically, it would need to become a nested object rather than a scalar, so that the burst size and the burst window can be expressed independently.
>
> That felt like it was probably out of scope for this particular ticket, and it also touches the shared config validation code which other services depend on, so I didn't want to make that call unilaterally. I'd suggest we track it separately. Happy to pick it up as a follow-up if that would be helpful!
>
> **Summary**
>
> To summarise: the rate limiter now uses a sliding window log backed by Redis, which fixes both the 2x boundary leak and the 4x multi-pod leak. 12 new tests, full suite green. The `burst_allowance` option is not yet wired up and will need a config schema change.
>
> Let me know if you have any questions about any of this, or if you'd like me to adjust the approach anywhere! Always happy to iterate. 😊

</details>

Works for anything outbound: a PR description, a review comment, a commit message, a Slack
message, an email.

---

## Turning it off

```
/unclaude-block off
```

Takes effect on your next message, in every session already running. `/unclaude-block on`
brings it back.

---

## Two more things

**If it breaks, you lose nothing.** Not logged in, rate-limited, timed out, anything at
all — you see Claude's original text, unchanged.

**It leaves nothing behind.** No transcript, no session history, no files to clean up.

---

## Documentation

| | |
|---|---|
| [How it works](docs/how-it-works.md) | The mechanism, why it can't affect Claude, and how to verify that yourself |
| [Configuration](docs/configuration.md) | Every setting, the off switch, manual install, uninstall |
| [Troubleshooting](docs/troubleshooting.md) | It isn't showing up |

---

## Credits

The idea comes from [claudish-to-english](https://github.com/gvzdv/claudish-to-english) by
Mike Gvozdev, which does the same thing through a separate model — a local one via ollama
by default, or the Anthropic or an OpenAI-compatible API.

**The difference here is where the rewrite comes from.** unclaude uses the same Claude you
are already paying for, through `claude -p`, so there is no second provider to install,
no API key to manage, and nothing running locally. If you would rather keep the rewriting
off your Claude usage entirely, or run it on a local model, use theirs.

---

MIT. Issues and pull requests welcome.
