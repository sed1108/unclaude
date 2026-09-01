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
underneath it a few seconds later — **usually 5–10 seconds** (7–9s measured on typical
answers), because a second, small model has to read the finished answer and write the
rewrite.

So you read the answer, and the plain-English version lands shortly after. Nothing is
held back while it thinks.

Short messages and messages that are mostly code are skipped entirely, and cost nothing.
The threshold is tunable — see [Configuration](docs/configuration.md).

---

## What it looks like

### The block, under an answer

Claude has just spent 707 words explaining why your deploys ship stale dependencies.
Underneath, you get this:

> Should I fix the deploy cache key? The problem: stale dependencies have shipped to
> production for at least four deploys because the cache key in deploy.yml is missing the
> lockfile hash. It's just `node-${{ runner.os }}`, so every deploy restores months-old
> node_modules instead of installing fresh ones. Example: axios@1.6.2 shipped instead of
> the pinned 1.7.9. The fix is a one-line change to match what test.yml already does: add
> the lockfile hash to the cache key. Trade-off: the next deploy on each branch will be
> slow (3-4 minutes instead of 40 seconds) because it invalidates the old cache, but only
> happens once per branch. After that, performance returns to normal.

The question at the front is the point. It was in the original too — six hundred words
down, phrased as "I did want to check in with you before making this change".

<details>
<summary><b>The 707-word original it was written from</b> (click to expand)</summary>

> Great question! Let me dig into this properly for you — this is exactly the kind of thing that's worth getting to the bottom of.
>
> **Investigating the deployment pipeline**
>
> I've gone ahead and traced through the full deployment flow to understand what might be happening here. Let me walk you through what I found, step by step.
>
> First, I took a look at the workflow definitions in `.github/workflows/` to get a sense of how the pipeline is structured. There are three workflows in play:
>
> | Workflow | Trigger | Purpose |
> |---|---|---|
> | `test.yml` | pull_request | Runs the test suite |
> | `deploy.yml` | push to main | Builds and deploys |
> | `nightly.yml` | schedule | Dependency audit |
>
> After reviewing each of these, I believe I've identified what appears to be the root cause of the behaviour you're seeing.
>
> **What I found**
>
> It looks like the issue may potentially stem from the way the `deploy.yml` workflow handles its caching layer. Specifically, I noticed that the cache key doesn't appear to include a hash of the lockfile:
>
> ```yaml
> - uses: actions/cache@v4
>   with:
>     path: node_modules
>     key: node-${{ runner.os }}
> ```
>
> As you can probably see, the key here is just `node-${{ runner.os }}`, which evaluates to something like `node-Linux` on every single run. There's no reference to `package-lock.json` anywhere in it.
>
> **Why this matters:** GitHub Actions restores a cache whenever the key matches an existing entry. Since this key never changes, the very first `node_modules` that was ever cached — which could be months old at this point — gets restored on every deploy, regardless of what the lockfile actually says.
>
> **Confirming the hypothesis**
>
> To make sure I wasn't jumping to conclusions, I cross-referenced this against the actual deploy logs. Here's what I observed:
>
> - The last four deploys all shipped with `axios@1.6.2` in the bundle
> - However, `package-lock.json` has pinned `axios@1.7.9` since the dependency bump in #2841
> - The cache restore step reports `Cache restored from key: node-Linux` on all four runs
> - No `npm ci` re-install was triggered, because `node_modules` already existed
>
> So the evidence does seem to line up with the hypothesis fairly conclusively.
>
> I should also note — and I think this is important context — that CI itself passes without any issues. This is because `test.yml` uses a completely separate cache key that *does* include the lockfile hash:
>
> ```yaml
> key: node-${{ runner.os }}-${{ hashFiles('package-lock.json') }}
> ```
>
> That's why this hasn't been caught by the test suite. Only the deploy job is affected, which is arguably the worst possible place for it to happen.
>
> **Proposed fix**
>
> The fix would be to bring `deploy.yml` in line with what `test.yml` is already doing:
>
> ```yaml
> key: node-${{ runner.os }}-${{ hashFiles('package-lock.json') }}
> ```
>
> This is a one-line change and it follows the pattern that's already established elsewhere in the repository, so there shouldn't be any surprises in terms of maintainability.
>
> **One thing to flag before I proceed**
>
> That said, I did want to check in with you before making this change, because there's a trade-off worth being aware of.
>
> Changing the cache key will invalidate every existing cache entry. What that means in practice is that the next deploy on each branch will need to do a full `npm ci` from scratch rather than restoring from cache. Based on the timings I'm seeing in the logs, that's likely to take somewhere in the region of 3-4 minutes, as opposed to the roughly 40 seconds it currently takes.
>
> This is a one-time cost per branch, and after that first run the cache will be warm again and things should return to normal. But if you have a release going out today, it might be worth timing the change around that.
>
> **Summary**
>
> To summarise where things stand: the deploy cache key is missing the lockfile hash, which means stale dependencies have been shipping to production for at least the last four deploys. The fix is a one-line change matching the existing pattern in `test.yml`, with a one-off slower deploy per branch as the cost.
>
> Let me know if you'd like me to go ahead and make that change, or if you'd prefer to handle the timing differently! Happy to help either way.

</details>

### `/unclaude`, on the way out

For text that leaves the session. You did the work; now it has to go somewhere other
people read.

```
Post the summary to the Jira ticket with /unclaude
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
