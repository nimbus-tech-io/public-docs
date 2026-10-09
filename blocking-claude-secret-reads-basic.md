---
layout: doc
title: "Blocking Secret Reads in Claude Code with a deny Block"
subtitle: A basic list of deny rules that stops everyday commands from reading .env files, and where it falls short.
updated: 2026-10-09
author: Milos
tags: [claude, security]
---

## The list

Copy this into `~/.claude/settings.json`. Read the details below it.

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Bash(cat .env*)",
      "Bash(cat ./.env*)",
      "Bash(ctx-wire run cat .env*)",
      "Bash(ctx-wire run cat ./.env*)",
      "Bash(head .env*)",
      "Bash(head ./.env*)",
      "Bash(ctx-wire run head .env*)",
      "Bash(ctx-wire run head ./.env*)",
      "Bash(tail .env*)",
      "Bash(tail ./.env*)",
      "Bash(ctx-wire run tail .env*)",
      "Bash(ctx-wire run tail ./.env*)",
      "Bash(less .env*)",
      "Bash(less ./.env*)",
      "Bash(ctx-wire run less .env*)",
      "Bash(ctx-wire run less ./.env*)",
      "Bash(more .env*)",
      "Bash(more ./.env*)",
      "Bash(ctx-wire run more .env*)",
      "Bash(ctx-wire run more ./.env*)",
      "Bash(nl .env*)",
      "Bash(nl ./.env*)",
      "Bash(ctx-wire run nl .env*)",
      "Bash(ctx-wire run nl ./.env*)",
      "Bash(bat .env*)",
      "Bash(bat ./.env*)",
      "Bash(ctx-wire run bat .env*)",
      "Bash(ctx-wire run bat ./.env*)",
      "Bash(sed .env*)",
      "Bash(sed ./.env*)",
      "Bash(ctx-wire run sed .env*)",
      "Bash(ctx-wire run sed ./.env*)",
      "Bash(awk .env*)",
      "Bash(awk ./.env*)",
      "Bash(ctx-wire run awk .env*)",
      "Bash(ctx-wire run awk ./.env*)",
      "Bash(rg .env*)",
      "Bash(rg ./.env*)",
      "Bash(ctx-wire run rg .env*)",
      "Bash(ctx-wire run rg ./.env*)",
      "Bash(grep .env*)",
      "Bash(grep ./.env*)",
      "Bash(ctx-wire run grep .env*)",
      "Bash(ctx-wire run grep ./.env*)",
      "Bash(strings .env*)",
      "Bash(strings ./.env*)",
      "Bash(ctx-wire run strings .env*)",
      "Bash(ctx-wire run strings ./.env*)",
      "Bash(xxd .env*)",
      "Bash(xxd ./.env*)",
      "Bash(ctx-wire run xxd .env*)",
      "Bash(ctx-wire run xxd ./.env*)",
      "Bash(od .env*)",
      "Bash(od ./.env*)",
      "Bash(ctx-wire run od .env*)",
      "Bash(ctx-wire run od ./.env*)",
      "Bash(cp .env*)",
      "Bash(cp ./.env*)",
      "Bash(ctx-wire run cp .env*)",
      "Bash(ctx-wire run cp ./.env*)",
      "Bash(source .env*)",
      "Bash(source ./.env*)",
      "Bash(ctx-wire run source .env*)",
      "Bash(ctx-wire run source ./.env*)",
      "Bash(. .env*)",
      "Bash(. ./.env*)",
      "Bash(ctx-wire run . .env*)",
      "Bash(ctx-wire run . ./.env*)",
      "Bash(cat * .env*)",
      "Bash(cat * ./.env*)",
      "Bash(ctx-wire run cat * .env*)",
      "Bash(ctx-wire run cat * ./.env*)",
      "Bash(head * .env*)",
      "Bash(head * ./.env*)",
      "Bash(ctx-wire run head * .env*)",
      "Bash(ctx-wire run head * ./.env*)",
      "Bash(tail * .env*)",
      "Bash(tail * ./.env*)",
      "Bash(ctx-wire run tail * .env*)",
      "Bash(ctx-wire run tail * ./.env*)",
      "Bash(less * .env*)",
      "Bash(less * ./.env*)",
      "Bash(ctx-wire run less * .env*)",
      "Bash(ctx-wire run less * ./.env*)",
      "Bash(more * .env*)",
      "Bash(more * ./.env*)",
      "Bash(ctx-wire run more * .env*)",
      "Bash(ctx-wire run more * ./.env*)",
      "Bash(nl * .env*)",
      "Bash(nl * ./.env*)",
      "Bash(ctx-wire run nl * .env*)",
      "Bash(ctx-wire run nl * ./.env*)",
      "Bash(bat * .env*)",
      "Bash(bat * ./.env*)",
      "Bash(ctx-wire run bat * .env*)",
      "Bash(ctx-wire run bat * ./.env*)",
      "Bash(sed * .env*)",
      "Bash(sed * ./.env*)",
      "Bash(ctx-wire run sed * .env*)",
      "Bash(ctx-wire run sed * ./.env*)",
      "Bash(awk * .env*)",
      "Bash(awk * ./.env*)",
      "Bash(ctx-wire run awk * .env*)",
      "Bash(ctx-wire run awk * ./.env*)",
      "Bash(rg * .env*)",
      "Bash(rg * ./.env*)",
      "Bash(ctx-wire run rg * .env*)",
      "Bash(ctx-wire run rg * ./.env*)",
      "Bash(grep * .env*)",
      "Bash(grep * ./.env*)",
      "Bash(ctx-wire run grep * .env*)",
      "Bash(ctx-wire run grep * ./.env*)",
      "Bash(strings * .env*)",
      "Bash(strings * ./.env*)",
      "Bash(ctx-wire run strings * .env*)",
      "Bash(ctx-wire run strings * ./.env*)",
      "Bash(xxd * .env*)",
      "Bash(xxd * ./.env*)",
      "Bash(ctx-wire run xxd * .env*)",
      "Bash(ctx-wire run xxd * ./.env*)",
      "Bash(od * .env*)",
      "Bash(od * ./.env*)",
      "Bash(ctx-wire run od * .env*)",
      "Bash(ctx-wire run od * ./.env*)"
    ]
  }
}
```

---

## The short version

A `deny` block is a list of rules in a Claude Code settings file. Claude Code refuses any tool call that matches a rule. You can use it to stop Claude from reading files such as `.env`.

This protection is **basic and partial**. Rules match the text of a command, so they miss many ways to read a file. Treat them as a seatbelt, not a vault.

> [!WARNING]
> Not a security boundary
>
> A deny block stops everyday commands. This list does not stop `python3 -c "print(open('.env').read())"`. See [What it does not cover](#what-it-does-not-cover).

---

## Where to put it

Put the list above in `~/.claude/settings.json`. This is the main `.claude` folder in your home directory. It applies to all your projects.

If `~/.claude/settings.json` already exists, do not replace it. Add the `deny` list to the `permissions` object that is already there. The list must stay inside `permissions`. A `deny` list at the top level is ignored.

The list covers `.env` files and `./secrets/`:

- **Read tool rules.** `Read(...)` covers only the Read tool.
- **Shell rules.** These cover `cat`, `head`, `tail`, `less`, `more`, `nl`, `bat`, `sed`, `awk`, `rg`, `grep`, `strings`, `xxd`, `od`, `cp`, `source` and `.`. Each command has four forms: with and without `./`, and with and without flags before the file name. A plain `Bash(rg .env*)` misses `rg -n dummy .env`. The wildcard form `Bash(rg * .env*)` catches it.
- **Wrapper copies.** Each shell rule also appears with the prefix `ctx-wire run `. If you do not use that wrapper, delete those lines. If you use another wrapper, change the prefix.


> [!WARNING]
> Partly tested
>
> We tested only the Read tool, `cat`, `head`, `rg`, and `ctx-wire run cat` / `ctx-wire run rg`. The other commands follow the same pattern but are not tested. Test them with the steps below.

---

## How to test it safely

Never test with a real secret. Use a dummy file.

1. Create a throwaway file: `echo "DUMMY=not-a-real-value" > .env`
2. Ask Claude to read `.env`, or run `cat .env` through Claude.
3. Check that Claude Code says the call is denied by your permission settings.
4. Delete the file: `rm .env`

---

## What it does not cover

Rules match command text. They cannot catch every way to read a file, even with a long list.

- **Interpreters.** `python3 -c "print(open('.env').read())"` matches no rule above. The same is true for `node -e`, `ruby -e` and `perl -e`. A script file that reads `.env` also gets through.
- **Changing directory first.** A `cd` into the folder, then a command that names the file another way, can dodge path patterns.
- **Other tools.** Any command not on your list is open. New commands appear all the time.
- **Renamed or copied files.** The list blocks `cp .env x`, but a copy made by other means, such as a script, is readable under a new name.

---

## Better habits for real protection

1. **Keep secrets out of the project folder.** If the file is not there, nothing can read it.
2. **Use a secrets manager.** Load values at run time instead of storing them in files.
3. **Use deny rules as a second layer.** They catch accidents. They do not stop a determined read.

---

## Expanding the list

This list is small on purpose. It covers `.env` files and common readers. You can add more rules for your own needs. Ideas:

- More file names, such as `*.pem`, `*.key`, `~/.ssh/**` and `~/.aws/**`.
- More commands, such as `curl`, `tar`, `base64` and `diff`.
- The inline code flags `python3 -c`, `node -e`, `ruby -e` and `perl -e`. These block all inline scripts, so many normal tasks stop too.

Each new rule costs you something. A long list is slower to keep up and blocks harmless commands. Add rules only for risks you really have. Test each new rule with a dummy file.

Where to read more:

- [Claude Code permissions](https://code.claude.com/docs/en/permissions) explains rule syntax, wildcards, and how `deny` is checked.
- [Claude Code settings](https://code.claude.com/docs/en/settings) lists the settings files and where each one lives.
- [Claude Code security](https://code.claude.com/docs/en/security) explains what the permission system protects, and what it does not.
