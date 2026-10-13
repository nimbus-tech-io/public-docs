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

1. Install the plugin. Run this once in your terminal:

   ```bash
   claude plugin install mattpocock-skills@claude-plugins-official
   ```

2. Set up each repo. Open Claude Code in the repo and run this once per repo:

   ```text
   /setup-matt-pocock-skills
   ```

   It asks which issue tracker you use, which labels you put on tickets, and where to save the docs the skills create.

The commands can change. If a step fails, check the [README](https://github.com/mattpocock/skills#installation-30-second-setup).

> [!WARNING]
> Old copies
>
> If you copied the skills into `~/.claude/skills/` before, move those copies out. Otherwise you have every skill twice.

---

## Where to start

- **Big work with many tasks and many unknowns:** start with `/wayfinder`.
- **One single task:** start with `/grill-me`. It asks you questions until the plan is clear.

For the rest of the flow, read [Matt's skills docs on AI Hero](https://www.aihero.dev/skills).

Not sure which skill to use? Run `/ask-matt` and describe your situation. It points you to the right skill or flow.
