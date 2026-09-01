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

Same job, one step later. The change above got made — and now it has to go on the ticket,
where people who were not in the session will read it.

```
Post the summary to the ticket with /unclaude
```

What gets posted:

> Extended password reset expiry window from 15 minutes to 2 hours and added dashboard
> visibility for expired-link attempts.
>
> Over the last 30 days, 340 out of 4,200 password reset requests failed (about 8%), all
> showing "This link has expired" even when clicked immediately. The reset link expired
> after 15 minutes, but the email notification queue took up to 22 minutes to deliver at
> weekday morning peak. Failures clustered between 08:45 and 10:15.
>
> Changed the reset expiry window from 15 minutes to 2 hours in settings/auth.py. This
> duration is standard across most products.
>
> Added a counter for expired-link attempts to the existing dashboard. Previously this
> required log digging to discover.
>
> Wrote 6 new tests in tests/test_password_reset.py: link used within window works, link
> used after 2 hours is rejected, link used twice is rejected, and expired-link counter
> increments correctly. All 6 passed. Full auth suite passed all tests.
>
> The notification queue remains slow at peak, and password reset emails are still queued
> behind marketing sends. The proper fix is a priority lane for password resets, which
> needs queue infrastructure work and should be tracked separately.
>
> The 340 affected customers who saw the expiry error have not been contacted. They would
> succeed on retry, but no one has told them that. Created a list ready if Support wants
> to reach out.

Note what **survived**: the 340 failures, the 08:45–10:15 window, the file paths, the 6
tests — and both unfinished things, left as the last thing you read rather than buried
under an apology. Shortening is not the goal. Cutting words while keeping every fact is.

<details>
<summary><b>What Claude would have posted</b> (click to expand)</summary>

> All done! 🎉 I've gone ahead and made the password reset change we discussed, and I'm happy to report everything is passing. Here's a comprehensive summary of what I did and why.
>
> **Background**
>
> Just to recap where we started: roughly one in twelve people who asked to reset their password couldn't complete it. Over the last 30 days that was 340 out of 4,200 requests — about 8% — and they were all seeing "This link has expired" even though they'd clicked the link the moment it landed in their inbox.
>
> As we established, the cause was a timing mismatch rather than a bug as such. The reset link was set to expire after 15 minutes, but the email sits on the notification queue first, and at the weekday morning peak that queue was taking up to 22 minutes to deliver. So the link was already dead on arrival, and the failures clustered almost entirely between 08:45 and 10:15.
>
> **What I changed**
>
> **1. The expiry window**
>
> I've extended the reset window from 15 minutes to 2 hours. This is the one-line change in `settings/auth.py` we talked about, and 1-2 hours is well within what most products use.
>
> **2. Visibility**
>
> I also noticed — and I think this is worth flagging as a somewhat separate improvement — that we had no way of seeing this problem without going and looking for it. Nobody would have known the failure rate had crept up.
>
> So I've added a counter for expired-link attempts, which means the rate now shows up on the existing dashboard rather than needing someone to go digging through logs. If it climbs again, we'll see it.
>
> **3. Tests**
>
> I wrote **6 new tests** in `tests/test_password_reset.py` covering:
>
> - A link used within the window works
> - A link used after 2 hours is rejected
> - A link that has already been used once is rejected the second time
> - The expired-link counter increments correctly
>
> All 6 pass, and I ran the full existing auth suite as well — **everything is green**.
>
> **Two things I wasn't able to finish**
>
> I do want to be upfront about a couple of items that are still outstanding.
>
> **The root cause is still there.** This change treats the symptom. The notification queue is still slow at peak, and password reset emails are still queued behind marketing sends. Giving them their own priority lane is the proper fix, but that's a few days of work and it really needs someone who knows the queue infrastructure to look at it properly. I'd suggest tracking that separately rather than folding it into this ticket.
>
> **The 340 people who already failed haven't been told.** They tried to reset their password, were told the link had expired, and as far as I can tell most of them just gave up rather than trying again. They'd now succeed if they tried, but nobody has told them that. I didn't want to send anything to 340 customers on my own initiative — that felt like it needed a human decision, and probably wants Support to look at the wording. I've put the list together and it's ready if someone wants it.
>
> **Summary**
>
> To summarise: the reset window is now 2 hours instead of 15 minutes, and the expired-link rate is now visible on the dashboard. 6 new tests, full auth suite green. The queue delay that actually caused this is unfixed, and 340 affected customers have not been contacted.
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
