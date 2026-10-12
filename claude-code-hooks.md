---
layout: doc
title: How to Use Claude Code Hooks
subtitle: Run your own code at fixed points in Claude Code's work, so the things that must always happen do happen.
updated: 2026-10-12
author: Milos
tags: [claude]
---

## What hooks are and why they help

Hooks make Claude Code run your code every time a specific event happens, instead of hoping the model remembers to do it. Each hook is a command you write, usually a short shell script, that Claude Code runs at a fixed point in its work.

Three settings shape how Claude behaves. They differ in how much you can count on them:

| Mechanism | What it does | How reliable it is |
|---|---|---|
| CLAUDE.md | Tells Claude what you prefer and how the project works | Claude will usually follow it, but it is guidance |
| Permissions | Allows, asks about, or denies tool use | Enforced by Claude Code |
| Hooks | Runs your command at set moments | Runs every time the event fires |

Use hooks for anything that must always happen, such as formatting code after an edit or blocking a dangerous command. Use CLAUDE.md for things Claude should simply know.

---

## Where hooks live

Hooks are declared in a settings file under a `hooks` key. The file you pick decides who the hook applies to.

| Location | Applies to | Can share with team |
|---|---|---|
| `~/.claude/settings.json` | All your projects | No, it stays on your machine |
| `.claude/settings.json` | One project | Yes, commit it to the repo |
| `.claude/settings.local.json` | One project, personal | No, keep it out of git |
| Managed policy settings | The whole organization | Yes, set by admins |
| Plugin `hooks/hooks.json` | Wherever the plugin is enabled | Yes, bundled with the plugin |

Every hook follows the same shape:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "your-command-here" }
        ]
      }
    ]
  }
}
```

The event name (`PostToolUse`), the optional `matcher` that narrows it, and the `command` to run are the three parts you will edit most. To check what is configured, type `/hooks` in Claude Code. You can also describe the hook you want and ask Claude to write the configuration for you.

---

## Lifecycle events

An event is a moment in Claude Code's workflow. Claude Code runs every matching hook when that moment arrives. The most useful events are below. The full list has more than 30 events, and the hooks reference covers all of them.

| Event | Fires when | Typical use |
|---|---|---|
| `SessionStart` | A session starts or resumes | Load project context, set environment variables |
| `UserPromptSubmit` | You submit a prompt, before Claude processes it | Add context, check the prompt |
| `PreToolUse` | Before a tool runs | Block risky commands or protected files |
| `PermissionRequest` | Claude needs a permission decision | Auto-approve a specific, safe action |
| `PostToolUse` | After a tool succeeds | Format code, run linters, log actions |
| `PostToolUseFailure` | After a tool fails | Record failures, add a hint for Claude |
| `Notification` | Claude is waiting for your input | Desktop alert, sound, or chat ping |
| `Stop` | Claude finishes a response | Check that tests pass before it stops |
| `PreCompact` / `PostCompact` | Around context compaction | Save state, restore key context |
| `ConfigChange` | A settings file changes mid-session | Audit or block configuration changes |
| `SessionEnd` | The session ends | Clean up temporary files |

Two timing details matter. `PreToolUse` can stop an action before it happens, but `PostToolUse` runs after the tool has already acted, so it cannot undo the change. `Stop` fires when Claude finishes a response, not when you interrupt it.

---

## Walkthrough: your first hook

This hook sends a desktop notification whenever Claude is waiting for you, so you can switch to other work without watching the terminal. It takes about five minutes.

1. Open `~/.claude/settings.json` and create the file if it does not exist. Add this block:

   ```json
   {
     "hooks": {
       "Notification": [
         {
           "matcher": "",
           "hooks": [
             {
               "type": "command",
               "command": "osascript -e 'display notification \"Claude Code needs your attention\" with title \"Claude Code\"'"
             }
           ]
         }
       ]
     }
   }
   ```
2. If the file already has a `hooks` key, add `Notification` next to your existing events instead of replacing the whole object.
3. On Linux, use `notify-send 'Claude Code' 'Claude Code needs your attention'` as the command. On Windows, use a PowerShell command that shows a message box.
4. Type `/hooks` in Claude Code and confirm the hook appears under `Notification`.
5. Press `Esc`, then press `Shift+Tab` until the status bar shows manual mode. Ask Claude to do something that needs permission, then switch to another window. You should get the notification.

If nothing appears on macOS, open System Settings, go to Notifications, find Script Editor, and allow notifications. The `osascript` command uses Script Editor to display the alert, so it stays silent until that permission is granted.

---

## Recipes you can copy

These patterns cover most day-to-day uses. Each one is a complete setting you can paste into the file shown.

### Format code after every edit

Run Prettier on each file Claude edits, so formatting stays consistent without anyone asking. The `jq` tool reads the file path from the hook input. Put this in `.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write" }
        ]
      }
    ]
  }
}
```

### Protect sensitive files

Block any tool call that names a secret file, such as `.env`. Claude receives the reason and adjusts its approach. A full, working example is in [Blocking Secret Reads in Claude Code](blocking-secret-reads.html).

### Restore context after compaction

When a long session is compacted, details can get lost. This `SessionStart` hook runs only after compaction and re-adds key facts, such as your conventions or the current task:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "compact",
        "hooks": [
          { "type": "command", "command": "echo 'Reminder: use Bun, not npm. Run bun test before committing.'" }
        ]
      }
    ]
  }
}
```

Text written to stdout by a `SessionStart` hook is added to Claude's context. Swap the `echo` for a command such as `git log --oneline -5` to include live information.

### Audit configuration changes

Record every change to a settings file during a session in a log, which helps with compliance reviews:

```json
{
  "hooks": {
    "ConfigChange": [
      {
        "matcher": "",
        "hooks": [
          { "type": "command", "command": "jq -c '{timestamp: now | todate, source: .source, file: .file_path}' >> ~/claude-config-audit.log" }
        ]
      }
    ]
  }
}
```

---

## Controlling Claude: exit codes, feedback, and matchers

A hook talks back to Claude Code through its exit code and its output. Exit codes are the simplest tool, and they cover most needs.

| Exit code | What Claude Code does |
|---|---|
| `0` | Continues. The hook has no objection, but this does not approve the action by itself. |
| `2` | Blocks the action. Anything written to stderr goes to Claude as the reason. |
| Any other code | Non-blocking error. Claude Code shows a warning and carries on. |

> [!WARNING]
> Exit 1 does not block
>
> Exit `1` is the usual Unix failure code, but Claude Code treats it as a warning, not a block. A policy hook must exit with `2`.

Here is a complete blocking hook. It stops Claude from editing protected files. Save it as `.claude/hooks/protect-files.sh`, run `chmod +x` on it, and register it under `PreToolUse` with the matcher `Edit|Write`:

```bash
#!/bin/bash
INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')
PROTECTED_PATTERNS=(".env" "package-lock.json" ".git/")

for pattern in "${PROTECTED_PATTERNS[@]}"; do
  if [[ "$FILE_PATH" == *"$pattern"* ]]; then
    echo "Blocked: $FILE_PATH matches protected pattern '$pattern'" >&2
    exit 2
  fi
done
exit 0
```

For finer control, exit `0` and print JSON instead. A `PreToolUse` hook can deny a call and explain why, which lets Claude adjust:

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "Use rg instead of grep for better performance"
  }
}
```

> [!WARNING]
> Use one method per hook
>
> Put `permissionDecision` inside `hookSpecificOutput`. If you put it at the top level, Claude Code ignores it without an error.

### Matchers

A matcher limits a hook to certain tools or events. Without one, the hook fires on every occurrence of its event. Matchers are case-sensitive. `Edit|Write` matches either tool, `Bash` matches shell commands, and `mcp__github__.*` matches every tool from one MCP server. To run a hook only for certain shell commands, add an `if` field such as `"if": "Bash(git *)"`.

---

## Other hook types

Command hooks run a shell script. Three other types cover situations where a script is awkward. Start with commands and add the others only when you need them.

| Type | How it decides | Good for | Default timeout |
|---|---|---|---|
| `command` | Runs your shell command and reads its exit code and output | Formatting, file checks, notifications, logging | 10 minutes |
| `http` | POSTs the event data to a URL and reads the JSON reply | A shared audit service used by several teams | 10 minutes |
| `prompt` | Sends the event data to a Claude model that returns `ok` or not | Judgment calls a script cannot express | 30 seconds |
| `agent` | Starts a sub-agent that can read files and run tools before deciding | Checks that need the codebase, such as "do the tests pass?" | 60 seconds |

A `prompt` hook is useful as a stop check. It asks whether every requested task is complete, and if not, Claude keeps working. Agent hooks are experimental and may change, so prefer command hooks for production workflows.

---

## Troubleshooting

Most hook problems fall into a few patterns. Work through the table from top to bottom.

| Symptom | Likely cause | Fix |
|---|---|---|
| Hook never runs | Matcher spelling or case is wrong, or the event is wrong | Run `/hooks` to confirm placement. Matchers are case-sensitive. |
| Hook missing from `/hooks` | Invalid JSON, such as a trailing comma | Validate the file. Restart the session if the watcher missed the edit. |
| "command not found" | Relative path or script not executable | Use `"$CLAUDE_PROJECT_DIR"/...` and run `chmod +x` on the script. |
| "jq: command not found" | `jq` is not installed | Install it, or parse the JSON with Python or Node.js. |
| JSON output does nothing | Shell startup files print text before your JSON | Wrap `echo` lines in your shell config with a check for interactive shells. |
| JSON output does nothing | Fields sit at the top level | Move `permissionDecision` and similar fields inside `hookSpecificOutput`. |
| Claude keeps working after a stop hook | The stop hook keeps blocking | Exit early when `stop_hook_active` is `true` in the input. |

To test a hook without starting a session, pipe sample input into it and check the exit code:

```bash
echo '{"tool_name":"Bash","tool_input":{"command":"ls"}}' | ./my-hook.sh
echo $?
```

For a full trace, start Claude Code with `claude --debug-file /tmp/claude.log` and follow the file with `tail -f`. Run `/debug` inside a session if you did not start with the flag.

---

## Security and team rollout

Hooks run as your user with full permissions, so a hook can do anything you can do in a shell. Treat them as production code. Read every hook in a repository before you trust it, especially a `.claude/settings.json` that came with a clone or a plugin you did not write.

For a team, follow these rules:

- Commit shared hooks to `.claude/settings.json` so everyone gets the same behavior. Keep personal hooks in `~/.claude/settings.json`.
- Use managed policy settings for rules that must apply across the organization. Set `allowManagedHooksOnly` if only admin-approved hooks should run.
- Keep hooks fast. A slow hook delays every matching event, so move heavy work to a background job or narrow the matcher.
- Remember the limits. A `PreToolUse` deny blocks an action even in bypass-permissions mode. A hook can make Claude stricter, but it cannot grant permissions that the permission rules do not allow. A `PostToolUse` hook runs after the change, so use `PreToolUse` when you need prevention.

If you start with one hook, make it the protected-files check or the auto-format hook. Both save real time, and the failures they cause are easy to see.

---

## Sources

- [Claude Code hooks guide](https://code.claude.com/docs/en/hooks-guide): setup steps, exit codes, matchers, hook types, and troubleshooting.
- [Claude Code hooks reference](https://code.claude.com/docs/en/hooks): the complete event list and every JSON field.
