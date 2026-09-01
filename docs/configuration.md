# Configuration

[← back to the README](../README.md)

There is nothing you have to configure. Everything here is optional.

## Turning it off

From inside Claude Code:

```
/unclaude-block off      # stop
/unclaude-block on       # start again
/unclaude-block status   # which is it
```

From any shell:

```sh
touch ~/.claude/unclaude-off     # off
rm ~/.claude/unclaude-off        # on
```

Both do the same thing — that one file is the switch. The hook checks it on **every**
message, so a change takes effect on your next message in every session already running.
Nothing to restart, and no settings file is touched.

To turn it off for a single session only, set `UNCLAUDE=0` in that session's environment.

## Settings

All optional, all environment variables.

| Variable | Default | What it does |
|---|---|---|
| `UNCLAUDE` | — | `0` turns it off for this session |
| `UNCLAUDE_MIN_CHARS` | `900` | Messages with less prose than this are skipped |
| `UNCLAUDE_MODEL` | `haiku` | The model that writes the rewrite |
| `UNCLAUDE_TIMEOUT` | `20` | Seconds before giving up and showing the original |

### `UNCLAUDE_MIN_CHARS` is the one that matters

It counts **prose**, not bytes: fenced code blocks and whitespace are stripped before
measuring. So a 3 KB answer that is mostly a diff with one line of English is skipped,
and so is every short reply.

- **Raise it** (say `1500`) if the pause interrupts you more often than the block helps,
  or to spend less of your subscription usage on rewrites.
- **Lower it** (say `200`) to see it fire on nearly everything — useful for a few minutes
  when you first install it, less so after that.

Set it wherever you set environment variables for your shell, or per session:

```sh
UNCLAUDE_MIN_CHARS=1500 claude
```

## Uninstall

```sh
claude plugin uninstall unclaude@unclaude
```

Then restart. The only thing left behind is `~/.claude/unclaude-off`, if you ever created
it, and you can delete that.

## Installing without the plugin system

If you'd rather wire it up by hand, clone the repo and add this to
`~/.claude/settings.json`:

```json
{
  "hooks": {
    "MessageDisplay": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "/path/to/unclaude/bin/unclaude-hook",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

Then, optionally:

- copy `skills/unclaude` into `~/.claude/skills/` for the `/unclaude` skill
- put `bin/unclaude` somewhere on your `PATH` for `unclaude on|off|status` in a shell

The `/unclaude-block` command is plugin-only; the shell script and the marker file cover
the same ground.
