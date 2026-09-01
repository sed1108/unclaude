# unclaude

Claude Code says a lot. This tells you what it actually said.

Under each substantial answer you get a short plain-English version of it. And whenever
you want it on purpose — posting a comment on a task, say — there is a `/unclaude` skill
you can call.

The rewrite is written by **your own Claude subscription** — no API key, no second
provider, nothing to install and run locally.

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

## What it looks like

### The block, under an answer

Claude has just finished explaining why your deploys are shipping stale dependencies.
Underneath its answer, you get this:

> Found the deployment issue: the cache key in `.github/workflows/deploy.yml` at line 34
> doesn't include the lockfile hash. This means old dependencies get reused even after the
> lockfile changes. That's why the last four deploys shipped with an outdated version of axios.
>
> The fix is to add the lockfile hash to the cache key, matching what the test job already
> does. One tradeoff: this invalidates existing caches, so the next deploy will be slower —
> around 3-4 minutes instead of 40 seconds.
>
> **Should they go ahead and make the change?**

The question at the end is the point. It was in the original too — in the last paragraph,
after four hundred words, phrased as "I wanted to flag one thing before making the change".

<details>
<summary>The original answer it was written from</summary>

> Great question! Let me take a look at that for you.
>
> I've gone ahead and traced through the deployment pipeline, and I think I've identified
> what's happening here. It appears that the issue may potentially stem from the way the
> `deploy.yml` workflow handles the caching layer — specifically, it looks like the cache
> key doesn't include the lockfile hash.
>
> Here's what I found:
>
> - `.github/workflows/deploy.yml:34` — the cache key is `node-${{ runner.os }}`, with no
>   reference to `package-lock.json`. That means a stale `node_modules` is restored even
>   after dependencies change.
> - This is why the last four deploys shipped with `axios@1.6.2` despite the lockfile
>   pinning `1.7.9` — the cache was never invalidated.
> - I should note that CI itself passes, because the test job uses a separate cache key
>   that does include the lockfile. Only the deploy job is affected.
>
> The fix would be to change the cache key to
> `node-${{ runner.os }}-${{ hashFiles('package-lock.json') }}`, which is the pattern the
> test job already uses. That said, I wanted to flag one thing before making the change:
> this will invalidate every existing cache, so the next deploy on each branch will be
> noticeably slower — probably 3-4 minutes rather than 40 seconds.
>
> Let me know if you'd like me to go ahead and make that change!

</details>

### `/unclaude`, on the way out

For text that leaves the session. You did the work; now it has to go somewhere other
people read.

```
Post the summary to the Jira ticket with /unclaude
```

What gets posted:

> Refactored the rate limiter from fixed window to sliding window log to prevent allowing
> 2x the intended rate at window boundaries.
>
> Changed `RateLimiter` in `src/middleware/rate_limit.py` to use sliding window. Added
> Redis-backed storage so the limit is shared across all 4 API pods. Wrote 12 new tests in
> `tests/test_rate_limit.py`, all passing.
>
> The burst allowance config option is currently ignored — it requires a change to the
> config schema that was out of scope for this ticket.

Instead of what Claude would have posted:

<details>
<summary>The original</summary>

> I've gone ahead and completed the work on the rate limiter! Here's a summary of what I did.
>
> I started by investigating the existing implementation and I noticed that it was using a
> fixed window, which as you probably know can allow up to 2x the intended rate at the
> window boundary. I've refactored it to use a sliding window log instead.
>
> Changes:
> - Rewrote `RateLimiter` in `src/middleware/rate_limit.py` to use a sliding window
> - Added Redis-backed storage so the limit is shared across all 4 API pods
> - Wrote 12 new tests in `tests/test_rate_limit.py` — all passing
>
> I should mention that there's one thing I wasn't able to fully resolve: the burst
> allowance config option is currently ignored. It seems like it might require a change to
> the config schema, which felt out of scope for this ticket. Happy to pick that up
> separately if you'd like!
>
> Let me know if you have any questions or if you'd like me to adjust anything!

</details>

Note what survived: every file path, the 4 pods, the 12 tests, and the unfinished burst
allowance — stated plainly instead of apologised for. Shortening is not the goal. Cutting
words while keeping every fact is.

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

## A few things worth knowing

**It cannot change Claude's answers.** The block is display-only — it never reaches
Claude's context, your transcript, or `/resume`. Not a promise; a property of the hook it
uses, and one you can check yourself. → [How it works](docs/how-it-works.md)

**It leaves nothing behind.** No transcript, no session history, no files to clean up.

**If it breaks, you lose nothing.** Not logged in, rate-limited, timed out, anything at
all — you see Claude's original text, unchanged.

**It costs a few seconds.** The answer streams at full speed as always; the block lands
5–10 seconds after it finishes. Short messages and messages that are mostly code are
skipped. → [Configuration](docs/configuration.md) if you want to tune that.

---

## Documentation

| | |
|---|---|
| [How it works](docs/how-it-works.md) | The mechanism, why it can't affect Claude, and how to verify that yourself |
| [Configuration](docs/configuration.md) | Every setting, the off switch, manual install, uninstall |
| [Troubleshooting](docs/troubleshooting.md) | It isn't showing up |

---

MIT. Issues and pull requests welcome.
