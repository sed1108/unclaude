# Troubleshooting

[← back to the README](../README.md)

## The block isn't appearing

In rough order of likelihood.

**1. You haven't restarted.** Hooks are read once, at session start. The session you
installed from will never show the block. This is the answer most of the time.

**2. The message was too short.** Anything under 900 characters of prose is skipped on
purpose, and code blocks don't count toward that — so a short reply, or a long answer that
is mostly a diff, is left alone deliberately. To check, set `UNCLAUDE_MIN_CHARS=200` and
try again; it should fire on almost everything.

**3. It's switched off.** `ls ~/.claude/unclaude-off` — if that file exists, it's off.
`/unclaude-block on` removes it.

**4. The rewrite is failing.** It's a real request on your account, so being logged out or
rate-limited stops it. Check directly:

```sh
claude -p --model haiku --restricted 'say OK'
```

If that fails, the hook is failing open exactly as designed — you're seeing your original
text, which is the correct behaviour, not a bug.

## Watching it work

Run the hook by hand with a long enough message:

```sh
echo '{"message_id":"1","session_id":"1","index":0,"final":true,"delta":"<paste a long message here>"}' \
  | ~/.claude/plugins/marketplaces/unclaude/bin/unclaude-hook
```

Output means it works. **No output means one of the four things above** — and no output is
also exactly what a real session would show you: your original text, untouched. That is
what makes this hard to diagnose from the outside, and why the checklist is worth walking
through in order.

## It's too slow

Raise `UNCLAUDE_MIN_CHARS` so it fires less often, or set `UNCLAUDE_TIMEOUT` lower so it
gives up sooner and shows the original. See [Configuration](configuration.md).

The pause itself can't be removed — see
[why](how-it-works.md#why-the-pause-cant-be-removed).

## The rewrite is wrong or missed something

The block is a summary written by a small fast model, and it is not authoritative. Your
original answer is always right there above it, unchanged, and it is the one to trust.

If the rewrite is consistently poor, try `UNCLAUDE_MODEL=sonnet` — slower, better. And
please [open an issue](https://github.com/sed1108/unclaude/issues) with an example.

## Does it change what Claude does?

No, and that's structural rather than a promise — see
[How it works](how-it-works.md#why-it-cant-affect-claude). If you suspect otherwise,
`/unclaude-block off` takes effect on your very next message, so it's a one-word test.
