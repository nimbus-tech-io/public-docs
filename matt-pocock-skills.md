---
layout: doc
title: Using Matt Pocock's Skills
subtitle: Where to learn Matt Pocock's skills, and how we install them.
updated: 2026-10-13
author: Milos
tags: [claude]
---

## What they are

[Matt Pocock's skills](https://github.com/mattpocock/skills) are a set of small skills for Claude Code. They help with real engineering work: planning a change, writing specs, test-driven development, debugging and code review.

Use them to give your work a clear structure, so the AI has a better chance to write high-quality code.

The skills change often, so this page does not copy them. Matt's own docs are always up to date:

- [Matt's skills docs on AI Hero](https://www.aihero.dev/skills): what each skill does, and the order to use them in.
- [Matt's videos on AI Hero](https://www.aihero.dev/videos): the skills in action.

### Start with these videos

- [5 Agent Skills I Use Every Day](https://www.aihero.dev/5-agent-skills-i-use-every-day): why a strict process helps AI agents, and the main flow, one skill at a time.
- [Real-world feature build with Claude Code](https://www.aihero.dev/real-world-feature-build-with-claude-code): the skills used on a real feature, from start to end.

> [!NOTE]
> The videos are older than the skills
>
> Some skill names in the videos may have changed. The videos also install the skills with `npx skills`. **Don't do that. Install them as a Claude Code plugin** (see below).

---

## How to install (updated as of 2026-10-13)

**Install the skills as a Claude Code plugin. Don't install them with `npx skills`.**

The plugin updates by itself. The `npx` installer copies the skill files into your repo, and they don't update. If you use both, you get every skill twice.

> [!NOTE]
> Use Matt's own marketplace, not the official one
>
> The Claude official marketplace (`claude-plugins-official`) updates this plugin late. It is pinned to version 1.2.3, and Matt's repo is already at 1.3.1 ([anthropics/claude-plugins-official#6360](https://github.com/anthropics/claude-plugins-official/issues/6360)). Matt's own marketplace (`mattpocock`) is always current. Install from that one.

1. If you installed from the official marketplace before, remove that copy. If you did not, skip this step:

   ```bash
   claude plugin uninstall mattpocock-skills@claude-plugins-official
   ```

2. Add Matt's marketplace. Run this once in your terminal:

   ```bash
   claude plugin marketplace add mattpocock/skills
   ```

3. Install the plugin from it:

   ```bash
   claude plugin install mattpocock-skills@mattpocock
   ```

4. Set up each repo. Open Claude Code in the repo and run this once per repo:

   ```text
   /setup-matt-pocock-skills
   ```

   It asks which issue tracker you use, which labels you put on tickets, and where to save the docs the skills create.

5. Turn on auto-update (see below). Without this, you stay on the version you installed.

### Turn on auto-update

Claude Code turns auto-update on only for official marketplaces. Matt's marketplace is a third-party one, so it is **off by default**. Turn it on once:

1. Open Claude Code and run `/plugin`.
2. Go to the **Marketplaces** tab.
3. Select `mattpocock`.
4. Select **Enable auto-update**.

After this, Claude Code checks for new versions when a session starts. When it finds one, it says `Plugin updated: mattpocock-skills · Run /reload-plugins to apply`. The next session loads the new version by itself.

To update right now, run `claude plugin update mattpocock-skills@mattpocock`, then restart Claude Code.

### Check your version

```bash
claude plugin list
```

Look for `mattpocock-skills@mattpocock`. It must say it comes from `mattpocock`, not `claude-plugins-official`. If you see both, you have every skill twice. Remove the official one with the command in step 1.

> [!TIP]
> When to go back to the official store
>
> When issue #6360 is closed and the official pin is current, the official store is fine again. Until then, use `mattpocock`.

The commands can change. If a step fails, check the [README](https://github.com/mattpocock/skills#installation-30-second-setup) and the [Claude Code plugin docs](https://code.claude.com/docs/en/discover-plugins).

> [!WARNING]
> Old copies
>
> If you copied the skills into `~/.claude/skills/` before, move those copies out. Otherwise you have every skill twice.

---

## Where to start

- **Big work with many tasks and many unknowns:** start with `/wayfinder`.
- **One single task:** start with `/grill-with-docs` or `grill-me`. It asks you questions until the plan is clear. See [here](https://www.aihero.dev/skills-grill-me#when-to-reach-for-it) when to reach for which one.

For the rest of the flow, read [Matt's skills docs on AI Hero](https://www.aihero.dev/skills).

Not sure which skill to use? Run `/ask-matt` and describe your situation. It points you to the right skill or flow.
