---
description: Turn the UN-CLAUDE block under each answer on or off, or show its state
argument-hint: on | off | status
allowed-tools: Bash(touch:*), Bash(rm:*), Bash(test:*), Bash(mkdir:*)
---

The UN-CLAUDE block is controlled by one marker file: `~/.claude/unclaude-off`.
Present means off. The hook checks it on every message, so a change takes effect on
the next message of every session already running — nothing needs restarting.

Argument given: `$ARGUMENTS`

Do exactly one of these, then say in one line what the state now is:

- `off` — `mkdir -p ~/.claude && touch ~/.claude/unclaude-off`
- `on` — `rm -f ~/.claude/unclaude-off`
- `status`, or no argument — `test -e ~/.claude/unclaude-off` and report OFF if it
  exists, ON if it does not.

Do not edit any settings file, and do not offer to uninstall the plugin. Turning the
block off is a one-file operation and is meant to stay that way.
