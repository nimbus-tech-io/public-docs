---
layout: doc
title: "Saving Tokens in Claude Code: A Practical Guide"
subtitle: Habits that keep your context small, your cache warm, and your usage down.
updated: 2026-10-07
author: Milos
tags: [claude]
---

## The short version

Most wasted tokens in Claude Code come from three things: context that grows larger than the task needs, cache misses that make Claude reread that context at full price, and model or effort settings higher than the task requires. Claude Code sends your full conversation with every request, so every habit below either shrinks that conversation or keeps it cached.

The habits that save the most:

1. **Clear between unrelated tasks.** Use `/clear` when the work changes. Stale context costs tokens on every later message.
2. **Choose model and effort at the start of a session.** Switching models mid-task throws away the cache.
3. **Write specific prompts.** Name the file and the change, and say how Claude can check its own work.
4. **Plan before big changes.** Plan mode and early course-corrections prevent expensive rework.
5. **Keep CLAUDE.md short.** It loads at the start of every session, so aim for under 200 lines.
6. **Keep MCP servers lean.** Disable the ones you are not using.
7. **Send noisy work to subagents** and trim noisy output with hooks.
8. **Avoid Max effort.** It cost about seven times Medium in one real-world test for no gain.
9. **Check your cache.** Read the `Prompt cache (main)` line in `/usage` when usage looks high.

---

## Protect the prompt cache

Prompt caching lets Claude Code re-read your unchanged history at a fraction of the normal input price. The match is exact and starts at the top of the request, so a change near the top forces everything after it to be reprocessed. The cost of each miss is one slow, expensive turn, after which the new prefix is cached again.

| Action | What happens | What to do instead |
|---|---|---|
| Switch models with `/model` | Each model has its own cache, so the next request rereads the whole history with no cache hits | Pick the model at the start, or switch right after `/clear` when there is little history to reread |
| Toggle plan mode under `opusplan` | Every toggle is a model switch | Use `opusplan` only for work with few toggles |
| Run a skill or command that names another model | That turn is a model switch | Check the skill's frontmatter before using it mid-task |
| Turn on fast mode mid-session | A request header that is part of the cache key changes, so the next request rereads everything at fast mode rates | Turn it on at the start of a session |
| Change effort | On Opus 5.5, Sonnet 5.5 and Fable 5.1 with an API key or Claude subscription, the cache is kept. On other models and some providers it restarts | Check which case applies to you before changing it mid-task |
| Connect or remove an MCP server, or deny a whole tool | Invalidates the cache only when tool definitions load upfront; with tool search deferring tools, which is the default on supported models, it does not | Leave tool search on |
| Run `/compact` | Replaces your history by design, so the conversation layer rebuilds | Do it at natural breaks between tasks |
| Accumulate many screenshots | Claude Code drops the oldest images in batches, causing one slower turn per batch | Share an image again if Claude needs it after removal |
| Upgrade Claude Code | The first conversation after an upgrade builds its cache from the top | Restart at a natural break, not mid-task |

These actions keep the cache: editing files in your repository, editing CLAUDE.md (though the edit only applies after `/clear`, `/compact` or a restart), changing permission mode or output style, running skills and commands, `/recap`, and `/rewind`. Subagents get their own cache and leave the parent's intact.

> [!TIP]
> Anthropic's own tip
>
> Pick your model and effort level at the top of a session, then save `/compact` for natural breaks between tasks.

---

## Match the model and effort to the task

Model and effort are the two settings you control that change how many tokens a task burns. Thinking tokens are billed as output tokens, and the default thinking budget can reach tens of thousands of tokens per request depending on the model. In Claude Code you can't turn thinking off on Opus 5.5, Sonnet 5.5 or the Fable models, so lower effort with `/effort` instead.

Which model and effort level suits which kind of task, and the test results behind each recommendation, are covered in [Choosing a Claude Model and Effort Level: Opus 5.5 vs Sonnet 5.5](choosing-a-model.html).

One more point matters for token use:

- **Subagents inherit your model.** A switch to Opus also applies to subagents that inherit the session's model. For simple subagent work, set `model: haiku` in the subagent's configuration.

Anthropic's cost guidance is simple: Sonnet handles most coding tasks and costs less than Opus, so reserve Opus for complex architectural decisions and multi-step reasoning. Agent teams add a multiplier. Anthropic reports about 7 times the tokens of a standard session when teammates run in plan mode, so keep teams small, use Sonnet for teammates, and shut them down when their work is done.

---

## Keep the context small

Token costs scale with context size. Each time Claude uses a tool, it sends another request carrying that batch of tool results. With caching, the history is reread at the cached rate, but a one-line question in a session open all day still draws usage for the whole conversation.

### Clear, compact and rewind

- **`/clear` between unrelated tasks.** It costs nothing and resets the session totals. Use `/rename` first so you can find the session later, then `/resume` to return to it.
- **`/compact` with instructions.** `/compact Focus on code samples and API usage` tells Claude what to keep. You can also add a "Compact instructions" heading to CLAUDE.md.
- **Compact while the cache is warm.** The summarizing request reads your whole conversation. That is cheap when the cache is warm and expensive after a long idle gap. If you plan to return tomorrow, compacting before you leave follows from how it is billed.
- **`/rewind` instead of arguing.** Rewinding truncates back to a prefix that is already cached, while compaction builds a new one. Press Escape as soon as Claude heads the wrong way, then double-tap Escape or run `/rewind`.

### What loads before you type anything

- **CLAUDE.md.** It loads at the start of every session. Keep it under 200 lines and move workflow-specific instructions, such as PR reviews or database migrations, into skills, which load only when invoked.
- **MCP servers.** Tool definitions are deferred by default, so mostly names and server instructions enter context. Run `/context` to see what takes space, and `/mcp` to disable servers you are not using. CLI tools like `gh`, `aws` and `gcloud` are more context-efficient than MCP servers because they add no per-tool listing.
- **Screenshots.** Large images hit the size cap with fewer images. Share only what Claude needs.

### Keep noisy output out of the conversation

- **Subagents for verbose work.** Running tests, fetching documentation or processing logs inside a subagent keeps the output in its context, and only a summary returns. The subagent's requests still draw on your usage, so give it a smaller model where you can.
- **Hooks that filter output.** A PreToolUse hook can grep a 10,000-line log for `ERROR` so Claude sees hundreds of tokens instead of tens of thousands. Anthropic's example hook rewrites test commands to show only failures.
- **Skills for orientation.** A "codebase-overview" skill describing architecture, key directories and naming conventions saves Claude from reading several files to learn the structure.
- **Code intelligence plugins.** For typed languages, they give Claude symbol navigation instead of text search, so one "go to definition" replaces a grep plus several file reads.

---

## Write prompts that need fewer turns

The cheapest turn is the one you don't need. Every correction loop sends the full context again, so most of the savings here come from getting it right the first time.

- **Be specific.** "Improve this codebase" triggers broad scanning. "Add input validation to the login function in auth.ts" lets Claude work with minimal file reads.
- **Give a way to check the result.** Include test cases, paste a screenshot, or define the expected output. When Claude can verify its own work, it catches issues before you ask for fixes.
- **Plan first on complex work.** Press Shift+Tab to cycle to plan mode. Claude explores the codebase and proposes an approach for your approval, which prevents expensive rework when the first direction is wrong.
- **Stop bad paths early.** Press Escape the moment Claude goes the wrong way. Then use `/rewind` or double-tap Escape to restore the conversation and code to an earlier checkpoint.
- **Test in increments.** Write one file, test it, then continue. Issues are cheapest to fix when they are caught early.
- **Add checks that catch dropped instructions.** In the 300-run test, extra effort made Opus more likely to rewrite a message and lose the instruction it carried. A test or review step catches that; more thinking did not.

---

## Idle time and long sessions

A session that has been open for hours can use far more than your activity suggests. Two things drive it: cached prefixes that expire while you are away, and background activity that keeps sending your full context.

### Cache lifetime depends on how you pay

| Your setup | Main conversation TTL | Other requests, such as subagents |
|---|---|---|
| Claude subscription, within plan usage | One hour | Five minutes |
| Usage credits, API key or cloud provider | Five minutes | Five minutes |

Your first message after a gap longer than the TTL misses the cache and reprocesses everything. You can set the main conversation's TTL to `1h` with the `promptCacheTtl` setting or the `CLAUDE_CODE_PROMPT_CACHE_TTL` environment variable, which needs Claude Code v2.1.242 or later. The one-hour TTL bills cache writes at a higher rate, so it pays off if you often leave sessions idle and costs more on short bursts of work. On Pro and Max plans, when you resume a large session after a long break, Claude Code offers to resume from a summary so later requests don't carry the full history.

### Things that keep spending while you are away

- **Scheduled tasks and loops** fire on their interval and send your full context each time.
- **Messages from other sessions** start a new turn when the session sits idle, sending your full context. Set `crossSessionInbound` to `hold` to queue them instead.
- **Goal check-ins** can start up to three idle turns per goal between your prompts. Set `CLAUDE_CODE_GOAL_CHECKIN_MINUTES` to `0` to turn them off.
- **Subagents, workflows and agent teammates** send their own requests. Teammates keep consuming tokens until they exit or the session ends.
- **Prompt suggestions** send a short request after each response. It is mostly cache reads, and you can turn the feature off.

Background summarization for `claude --resume` adds a small amount, typically under $0.04 per session. When a task is done, `/clear` leaves those idle requests with far less context to carry.

---

## Measure your own usage

Anthropic reports an average cost of around $13 per developer per active day across enterprise deployments, with 90% of users below $30 per active day. Your own numbers will differ with model, codebase size and how many instances you run, so check them rather than guessing.

| Tool | What it shows | Notes |
|---|---|---|
| `/usage`, Session block | Token counts by model and an estimated dollar cost at list price | Meant for API users; subscribers see plan usage bars instead. Resets on `/clear` |
| `/usage`, `Prompt cache (main)` line | Request count, share of input tokens from cache, misses, expected rebuilds, and whether the cache is warm | Needs v2.1.251 or later; the likely cause of the last miss needs v2.1.260 or later |
| `/usage`, plan breakdown | Usage attributed to skills, subagents, plugins and MCP servers, plus flags for behaviors above 10% such as long context or cache misses | Subscription plans; press `d` or `w` for 24 hours or 7 days |
| `/context` | What is consuming space in the context window | Use it before disabling MCP servers |
| Status line | Context window usage and cache fields, continuously | Configure with a status line script |
| `/insights` | A report on how you work, including friction points and suggestions | Analyzes up to 200 recent sessions and itself uses tokens |

Here is the example line from Anthropic's docs: 14 requests, 91% of input tokens from cache, 2 misses, 1 expected rebuild, and warm with a one-hour TTL. An "expected rebuild" is Claude Code rewriting the conversation itself, through compaction or clearing old tool results, and is not a problem. If cache creation stays high turn after turn, something in your prefix keeps changing, and the likely-cause text usually names it.

The session cost figure is an estimate. For billing, use the Usage page in the Claude Console.

---

## Sources

Three pages were read in full and two as search-result excerpts. Features and version numbers change often, so check them against the current Claude Code docs.

| Source | Used for | Read |
|---|---|---|
| [Manage costs effectively (Claude Code docs)](https://code.claude.com/docs/en/costs) | Reduce-token-usage advice, `/usage` fields, idle-time costs, the $13 and $30 figures | In full |
| [How Claude Code uses prompt caching (Claude Code docs)](https://code.claude.com/docs/en/prompt-caching) | Cache-invalidating actions, TTL by plan, model and effort switching | In full |
| [The Default Was Right (paddo.dev)](https://paddo.dev/blog/default-was-right/) | The 300-run test: cost by effort level, regressions, the Opus Max comparison | In full |
| [Opus 5.5 built the same app 6 times (AI Fire)](https://www.aifire.co/p/claude-opus-5-5-built-the-same-app-6-times-and-extra-won) | Build-from-scratch test where Extra gave the best balance | Excerpt |
| [Opus 5.5 at every effort level (openclawdatabase.com)](https://openclawdatabase.com/news/videos/2026-09-24-opus-55-every-effort-level-cost-comparison/) | Build-from-scratch test where Max cost about double Extra | Excerpt |
