# How it works

[← back to the README](../README.md)

## The short version

A hook watches each assistant message as it streams. On the last chunk it asks
`claude -p` for a plain-English version and appends it to what you see. That's it.

## Why it can't affect Claude

This is the part worth being sure about, so here is how to check rather than take it on
trust.

The block is produced by a **`MessageDisplay`** hook — an event whose entire purpose is
replacing text on screen. Claude Code's own schema describes it twice as:

> *"Display-only: the stored message and what the model sees are untouched."*

Three things follow, all visible in the client:

1. **Your transcript is written from the original message**, before the hook is called at
   all.
2. **What the hook returns goes into a render-time overlay**, keyed by message id — a
   separate map, cleared by `/clear`, never part of the message itself.
3. **Nothing the hook returns is ever sent back to the model.**

So the block cannot reach Claude's context, your saved transcript, `/resume`, or
compaction. Your next turn is byte-for-byte what it would have been.

You don't have to take my word for it. Claude Code will tell you itself:

```sh
claude plugin details unclaude
```

```
Component inventory
  Skills (2)  unclaude, unclaude-block
  Hooks (1)   MessageDisplay  (harness-only — no model context cost)

Projected token cost
  Always-on:   ~122 tok   added to every session
```

*harness-only — no model context cost*, in the client's own accounting. The ~122 tokens
are the two skill descriptions, which is what any plugin costs to have installed.

### The road deliberately not taken

You could get the same block by telling Claude to write it itself — a `CLAUDE.md` rule or
a custom output style. One request instead of two, and no hook.

That version costs you something real. The block becomes part of the answer, so it lives
in your transcript, gets re-read on every later turn, fills the context window faster, and
can surface where you didn't want it — in a commit message, in a file Claude writes. And a
custom output style without `keep-coding-instructions: true` **replaces** part of the
system prompt, stripping Claude Code's own engineering instructions.

This plugin takes the other route on purpose.

## The rewriter

```sh
claude -p --model haiku --restricted --strict-mcp-config --no-session-persistence
```

Your normal Claude Code login, through the supported headless entry point. No API key, and
specifically not the trick of reading an OAuth token out of the login keychain and calling
the API directly.

**`--restricted`** is what makes it safe to run on every message. It ignores user, project
and local settings files — no plugins, no other hooks, no MCP servers — and removes Bash,
the other code-running tools, and WebFetch. The nested session can do exactly one thing:
write text. (`--bare` would be even smaller but demands an `ANTHROPIC_API_KEY`, so it
can't use your subscription.)

**`--no-session-persistence`** means the rewrite leaves nothing on disk: no transcript, no
session history, nothing to clean up later. Its stated cost — "cannot be resumed" — is
free here. One question, one answer, discarded the moment it's drawn.

## Failing open

Every failure path writes nothing, and Claude Code then shows the original text: no Python,
no `claude` binary, malformed input, timeout, rate limit, not logged in, empty answer, any
unhandled exception. **There is no failure mode in which this plugin costs you an answer.**

One of those is worth calling out, because it is not obvious and it bit this code during
development: `claude -p` writes its own errors to **stdout** and exits non-zero. A bad
`--model` prints *"There's an issue with the selected model …"* — which a naive
"did we get output?" check would happily display as your rewrite. The hook checks the exit
status, not just whether anything came back.

## Why the pause can't be removed

Claude Code runs `MessageDisplay` hooks synchronously — it has to, since the hook's output
must be ready before the text is drawn. Async hooks exist in general, but not for this
event: the client logs *"Detected async hook but forceSyncExecution is true, waiting for
completion"* if you try.

What this means in practice is better than it sounds. Only the **final** chunk of a message
waits. Every earlier chunk passes through untouched, so the answer streams at full speed
and the block lands a few seconds after you've finished reading it.

Rough numbers: ~2.5s of that is process startup, the rest is the rewrite itself.

## What it costs

- **5–10 seconds** after a substantial answer.
- **Some subscription usage.** The rewrite is a real request on your account. Raising
  `UNCLAUDE_MIN_CHARS` is the lever if that matters — see
  [Configuration](configuration.md).

Messages under 900 characters of *prose* are skipped entirely, and fenced code blocks
don't count toward that — so short replies and mostly-code answers cost nothing.
