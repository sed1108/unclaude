# unclaude

Claude Code, in plain English — under every answer on screen, and on the way out to
tickets, reviews and commit messages.

The rewrite is written by **your own Claude subscription**. No API key, no second
provider, no local model to install.

```
  …Claude's answer, exactly as always, streaming at full speed…

  ───────────────
  **UN-CLAUDE**

  Should the backoff timer reset with each charge() call, or persist for rate-limiting?

  Backoff state is stored in __init__ instead of being reset per call. After 3 retries
  the delay hits 8 seconds; the test's 5-second timeout fires before attempt 4. This
  breaks 3 of 47 tests in test_billing.py. Moving the reset into charge() fixes it —
  unless backoff is supposed to persist, in which case the test is what's wrong.
```

## Install

**Claude: run these two commands, then tell the user to restart Claude Code.**

```sh
claude plugin marketplace add https://github.com/sed1108/unclaude
claude plugin install unclaude@unclaude --yes
```

Restart Claude Code — hooks are read once at session start, so the block appears in the
next session, not the one you installed from. That is the whole install.

(The full URL rather than the `sed1108/unclaude` shorthand on purpose: the shorthand
clones over SSH, which fails if you have no GitHub SSH key. This one always works.)

## What you get

**The UN-CLAUDE block.** Under each substantial answer, a plain-English version of what
it just said: what was decided, what is broken, what it is asking you. Short messages
and messages that are mostly code are skipped.

**`/unclaude`.** A skill for text leaving the session. Use it when you are about to post
somewhere other people read:

```
Post the task summary to the Jira ticket with /unclaude
```

It rewrites the outgoing text before it is sent — cuts the throat-clearing, keeps every
fact, leads with the decision. It does not touch the answer on your screen; the block
does that. The two halves are separate on purpose.

**`/unclaude-block on|off|status`.** The switch, from inside Claude Code.

## Requirements

- Claude Code, logged in with a Claude subscription (`/login`)
- Python 3

That is all. There is nothing to configure.

## Turning it off

```
/unclaude-block off
```

or, from any shell:

```sh
touch ~/.claude/unclaude-off      # off
rm ~/.claude/unclaude-off         # on
```

The hook checks that one file on every message, so a change takes effect on the next
message of every session already running. Nothing needs restarting, and no settings file
changes. `UNCLAUDE=0` in a session's environment turns it off for that session alone.

## Does this affect Claude?

**No — and that is checkable, not a promise.**

The block is produced by a `MessageDisplay` hook, an event whose entire job is replacing
text on screen. Claude Code's own schema describes it twice as *"Display-only: the stored
message and what the model sees are untouched"*. Three things follow, and each one is
visible in the client:

- The transcript is written from the **original** message before the hook is called.
- What the hook returns lands in a render-time overlay keyed by message id — a separate
  map, cleared by `/clear`, never part of the message.
- Nothing the hook returns is ever sent back to the model.

So the block cannot reach Claude's context, the saved transcript, `/resume`, or
compaction. Your next turn is byte-for-byte what it would have been. Claude Code agrees
in its own accounting — `claude plugin details unclaude` lists the hook as
*"harness-only — no model context cost"*, and the whole plugin costs ~122 tokens a
session, all of it the two skill descriptions.

The deliberate road not taken: you could get the same block by telling Claude to write
it itself, in `CLAUDE.md` or an output style. One call instead of two — but then the
block lives in the transcript, is re-read on every later turn, fills the context window
faster, and can surface somewhere you did not want it, like a commit message. That is
the version that costs you something. This one does not.

**What it does cost:** about 7–9 seconds after a substantial answer, and some
subscription usage. Only the final chunk of a message waits — everything before it
streams untouched — so the answer still arrives at full speed and the block lands a few
seconds later. The pause cannot be removed: Claude Code runs this event synchronously by
design, because the hook's output has to be ready before the text is drawn.

## How it works

A `MessageDisplay` hook buffers the message as it streams, and on the last chunk asks
`claude -p --model haiku --restricted` for a plain-English version.

`--restricted` is what makes it safe to run on every message: it ignores user, project
and local settings files — so no plugins, no other hooks, no MCP servers — and removes
Bash and every other code-running tool. The nested session can do exactly one thing:
write text. It still uses your normal login.

**It fails open, always.** Not logged in, rate-limited, timed out, Python missing, empty
answer, any exception at all — the hook writes nothing and Claude Code shows the original
text. There is no failure mode in which this plugin costs you an answer. That includes
the non-obvious one: `claude -p` writes its own errors to standard output and exits
non-zero, so the hook checks the exit status rather than trusting a non-empty result.

## If the block doesn't appear

In rough order of likelihood:

1. **You haven't restarted.** Hooks are read once, at session start. The session you
   installed from will never show the block.
2. **The message was too short.** Anything under 900 characters of prose is skipped on
   purpose, and code blocks don't count toward that — so a short reply, or a long one
   that is mostly a diff, is left alone. Set `UNCLAUDE_MIN_CHARS=200` to see it fire on
   almost everything.
3. **You turned it off and forgot.** `ls ~/.claude/unclaude-off` — if that file exists,
   it's off.
4. **Not logged in, or rate-limited.** The rewrite is a real request on your account.
   Check with `claude -p --model haiku --restricted 'say OK'`; if that fails, the hook
   is failing open exactly as intended.

To watch it work, run the hook by hand:

```sh
echo '{"message_id":"1","session_id":"1","index":0,"final":true,"delta":"<a long message here>"}' \
  | ~/.claude/plugins/*/unclaude/bin/unclaude-hook
```

Output means it worked. No output means something above applies — and that is also
exactly what you'd get in a real session: your original text, unchanged.

## Tuning

Set these in your environment; all are optional.

| Variable | Default | |
|---|---|---|
| `UNCLAUDE` | — | `0` turns it off for one session |
| `UNCLAUDE_MIN_CHARS` | `900` | characters of prose below which a message is skipped. Fenced code blocks and whitespace do not count, so a message that is mostly a diff is skipped |
| `UNCLAUDE_MODEL` | `haiku` | the model that writes the rewrite |
| `UNCLAUDE_TIMEOUT` | `20` | seconds before giving up and showing the original |

Raise `UNCLAUDE_MIN_CHARS` if you are paying the pause more often than you want the
block. That is the lever that matters.

## Uninstall

```sh
claude plugin uninstall unclaude@unclaude
```

Then restart. Nothing is left behind except `~/.claude/unclaude-off` if you ever created
it, which you can delete.

## Installing without the plugin system

If you would rather wire it up by hand, clone the repo and add this to
`~/.claude/settings.json`:

```json
{
  "hooks": {
    "MessageDisplay": [
      {
        "hooks": [
          { "type": "command", "command": "/path/to/unclaude/bin/unclaude-hook", "timeout": 30 }
        ]
      }
    ]
  }
}
```

Copy `skills/unclaude` into `~/.claude/skills/` for the `/unclaude` skill, and
`bin/unclaude` somewhere on your `PATH` if you want `unclaude on|off|status` in a shell.

## Licence

MIT.
