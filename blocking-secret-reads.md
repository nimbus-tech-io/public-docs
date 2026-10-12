---
layout: doc
title: "Blocking Secret Reads in Claude Code"
subtitle: A small hook that stops Claude from touching .env files, and where it still falls short.
updated: 2026-10-12
author: Milos
tags: [claude, security]
---

## The hook

Save this as `~/.claude/hooks/block-env.py`. It blocks any tool call that names a `.env` or `.env.<suffix>` file. It allows `.env.example`. Shell commands are scanned in full. File tools are checked only for the file they touch.

```python
#!/usr/bin/env python3
"""PreToolUse hook: block tool calls that name a .env* file, except .env.example.

Bash calls are scanned in full, because a shell command can read a file in many
ways. File tools are scanned only in the fields that say which file they touch,
so writing text that mentions .env (a README, for example) is allowed.
Exit 2 blocks the call and shows the message on stderr. Any other exit lets the
call through.
"""
import json
import re
import sys

ALLOWED = {".env.example"}
# ".env" or ".env.<suffix>" that is not glued to a word (so process.env, os.environ,
# and .environment do not match).
PATTERN = re.compile(r"(?<![\w.])\.env(?![A-Za-z0-9_])(?:\.[\w.\-]+)?")

# For these tools, scan only the fields that name a file or folder.
PATH_FIELDS = {
    "Read": ("file_path",),
    "Edit": ("file_path",),
    "Write": ("file_path",),
    "NotebookEdit": ("notebook_path",),
    "Grep": ("path", "glob"),
    "Glob": ("path", "pattern"),
}


def strings(value):
    if isinstance(value, str):
        yield value
    elif isinstance(value, dict):
        for v in value.values():
            yield from strings(v)
    elif isinstance(value, list):
        for v in value:
            yield from strings(v)


def scanned_text(data):
    """Return the strings to scan. Bash and unknown tools are scanned in full."""
    tool_input = data.get("tool_input", {})
    fields = PATH_FIELDS.get(data.get("tool_name"))
    if fields is None or not isinstance(tool_input, dict):
        return strings(tool_input)
    return (v for f in fields for v in strings(tool_input.get(f)))


def main():
    try:
        data = json.load(sys.stdin)
    except Exception:
        return 0
    for text in scanned_text(data):
        for match in PATTERN.finditer(text):
            if match.group(0) not in ALLOWED:
                sys.stderr.write(
                    f"Blocked: '{match.group(0)}' is a secret env file. "
                    "Only .env.example is allowed.\n"
                )
                return 2
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

---

## Turn it on

1. Save the script above as `~/.claude/hooks/block-env.py`.
2. Add the block below to `~/.claude/settings.json`. If the file already has a `hooks` key, add the `PreToolUse` entry next to your existing events. Do not replace the whole object.
3. Type `/hooks` in Claude Code and confirm the hook appears under `PreToolUse`.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash|Read|Grep|Glob|Edit|Write|NotebookEdit",
        "hooks": [
          {
            "type": "command",
            "command": "python3 -I \"$HOME/.claude/hooks/block-env.py\"",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

The `matcher` lists the tools the hook watches. It covers the shell, the file tools, and the search tools.

Optional: add two deny rules as a second layer. They block the Read tool on `./.env` and on the `./secrets/` folder. The hook does not cover `secrets/`, so this rule is the only guard for it.

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./secrets/**)"
    ]
  }
}
```

---

## The short version

A hook is a script that Claude Code runs before a tool call. If the script exits with code `2`, Claude Code blocks the call and shows the script's message to Claude. For more on hooks, read [How to Use Claude Code Hooks](claude-code-hooks.html).

This hook looks at the whole tool call, not at one command. For shell commands, it scans the full command for a `.env` file name. For file tools, it checks the file path. So `cat -n .env`, `rg -n dummy ./.env.local`, and `cd project && cat .env` are all caught by the same check. We first tried a deny list, and it needed 127 rules and still missed cases.

This protection is **basic and partial**. It matches the text of a tool call, so it misses many ways to read a file. Treat it as a seatbelt, not a vault.

> [!WARNING]
> Not a security boundary
>
> The hook stops everyday reads. It does not stop a command that builds the file name at run time. See [What it does not cover](#what-it-does-not-cover).

---

## What it blocks

We ran the script on sample tool input. It returned exit code `2` (blocked) or `0` (allowed):

| Tool call | Result |
|---|---|
| `cat .env` | Blocked |
| `rg -n dummy ./.env.local` | Blocked |
| `cd project && cat .env` | Blocked |
| `python3 -c "print(open('.env').read())"` | Blocked |
| Read tool on `app/.env.production` | Blocked |
| Read tool on `app/.env.example` | Allowed |
| Write tool with the target `app/.env` | Blocked |
| Edit or Write on `README.md`, with text that mentions `.env` | Allowed |
| Grep with `path` or `glob` that names `.env` | Blocked |
| Grep with a search pattern that only mentions `.env` | Allowed |
| `echo process.env.HOME` and `ls .environment` | Allowed |

The script also blocked a real command in a live session. The command only contained the file name inside a test string.

> [!NOTE]
> Shell commands are still strict
>
> The hook cannot tell a file read from a file name in a string, so a shell command that only mentions `.env` is blocked. File tools are different: Claude can edit a README that mentions `.env`. If the shell rule is too strict, run that command yourself in your own terminal.

---

## How to test it safely

Never test with a real secret. Use a dummy file.

1. In your own terminal, create a throwaway file: `echo "DUMMY=not-a-real-value" > .env`
2. Ask Claude to read the file, or to run `cat` on it.
3. Check that Claude Code shows the "Blocked" message from the hook.
4. Delete the file: `rm .env`

You can also test the script without a session. Pipe sample input into it and check the exit code:

```bash
echo '{"tool_name":"Bash","tool_input":{"command":"cat .env"}}' | python3 -I ~/.claude/hooks/block-env.py
echo $?
```

The exit code should be `2`.

---

## What it does not cover

The hook matches text. It cannot catch every way to read a file. We ran each of these through the script, and it let them through:

- **A name built at run time.** `python3 -c "print(open('.'+'e'+'nv').read())"` never contains the file name as one piece of text.
- **Wildcards.** `cat .e*` and `cat .en?` match the file, but not the pattern.
- **Other secret files.** The pattern only knows `.env` names. Files such as `*.pem`, `~/.ssh/id_rsa` and `~/.aws/credentials` are open. Add patterns for them if you need them.

These are gaps we did not test, but they follow from how the hook works:

- **A script that is written, then run.** File tools are not scanned for content. Claude can write a script that reads the file, then run it with `python3 read.py`. The script text is not scanned, and the command does not name the file.
- **A broken hook lets calls through.** Exit code `2` is the only block. If `python3` is missing, or the script crashes, the call goes ahead.
- **Tools outside the matcher.** The hook only watches the tools in `matcher`. Other tools, such as MCP tools, are not checked.
- **Claude can edit its own settings.** A call that edits `~/.claude/settings.json` or the script does not name a `.env` file. Managed policy settings can lock hooks. See the security section of the hooks doc below.
- **Secrets already in the environment.** `printenv` prints variables that are already loaded.

---

## Better habits for real protection

- **Keep secrets out of the project folder.** If the file is not there, nothing can read it.
- **Use a secrets manager.** Load values at run time instead of storing them in files.
- **Use the hook as a second layer.** It catches accidents. It does not stop a determined read.

---

## Learn more

- [How to Use Claude Code Hooks](claude-code-hooks.html) explains events, matchers, exit codes, and team rollout.
- [Claude Code hooks reference](https://code.claude.com/docs/en/hooks) lists every event and JSON field.
- [Claude Code permissions](https://code.claude.com/docs/en/permissions) explains rule syntax and how `deny` is checked.
- [Claude Code settings](https://code.claude.com/docs/en/settings) lists the settings files and where each one lives.
- [Claude Code security](https://code.claude.com/docs/en/security) explains what the permission system protects, and what it does not.
