---
layout: doc
title: "Choosing a Claude Model and Effort Level: Opus 5.5 vs Sonnet 5.5"
subtitle: Which model and effort level to use for which task, and the evidence behind it.
updated: 2026-10-07
author: Milos
tags: [claude]
---

## The short version

Use Opus 5.5 at Medium for work in existing code where judgment matters, and Sonnet 5.5 at Low or Medium for small, well-tested changes. Save Extra for long builds from scratch, and avoid Max: when Medium falls short, narrow the task or add context first.

Three rules cover most situations:

1. **Existing code, judgment needed:** Opus at Medium. Drop to Opus Low to save money.
2. **Small, well-specified change with tests:** Sonnet at Low or Medium.
3. **Tempted by Sonnet High or Extra:** use Opus instead. In a 300-run test on a real codebase, Opus Low matched Sonnet High's results for about 36% less cost, and Sonnet Extra cost twice as much as Opus Medium while scoring lower.

> [!NOTE]
> Why more effort is not always better
>
> Effort controls how much a model spends, not how careful it is. On real code, raising Opus above Medium made it break more existing tests.

---

## Which model and effort to use

Pick by the kind of task first, then by how much the result can be checked. "Extra" is the name Claude Code's `/effort` slider uses for the API's xhigh level.

| Situation | Model and effort | Why |
|---|---|---|
| Small, well-specified change with tests (bug fix, UI tweak, templated edit) | {% include badge.html m="sonnet" %} Sonnet 5.5, Low or Medium | Zero broken tests at both levels in the 300-run test, and the cheapest careful setting there |
| Change in existing code where judgment matters | {% include badge.html m="opus" %} Opus 5.5, Medium (Low to save money) | Medium was the best Opus setting on two separate days; Low got 11 clean runs for 38% of Medium's cost |
| Change with many edge cases in existing code | {% include badge.html m="opus" %} Opus 5.5, Medium | At Low, Opus solved one edge-case-heavy change once in three runs, against three at Medium |
| Architecture, unfamiliar repository, ambiguous requirements | {% include badge.html m="opus" %} Opus 5.5, Medium, then step up only if it falls short | Anthropic positions Opus for complex work requiring careful judgment |
| Long build from scratch (30+ minutes) | {% include badge.html m="opus" %} Opus 5.5, Extra | Two build-from-scratch tests found Extra best; Max roughly doubled cost and time for a worse result |
| A Medium run fails its tests | Don't raise effort: split the task into smaller steps or add context, then rerun at Medium | Sonnet's only gain at Max came from edge cases, at 35x the Medium price; narrower tasks and better context are the cheaper fix |
| High-volume triage or latency-sensitive chat | {% include badge.html m="sonnet" %} Sonnet 5.5, Low | Speed and price matter most, and output is easy to check |

Two habits sit underneath the table. First, add a test or a review step to catch dropped instructions, because more effort made that failure more likely, not less. Second, in Claude Code the `opusplan` alias uses Opus for planning and Sonnet for execution. Each plan-mode toggle counts as a model switch that starts a fresh cache, so it suits work with few toggles.

---

## What effort actually changes

Effort is a spend dial. It changes how many tokens a model writes and how far it searches, and it applies to every output token: text, tool calls and thinking. Lower effort also means fewer, terser tool calls, which is part of why Low feels fast in Claude Code.

| | Opus 5.5 | Sonnet 5.5 |
|---|---|---|
| Default effort, Claude API | Medium | High |
| Default effort, Claude Code and Claude apps | Medium | Medium |
| Thinking | Always on, cannot be disabled | Adaptive; up-front thinking can be skipped at Low, Medium and High via the API |
| Anthropic's starting advice | Start at Medium; reserve Extra and Max for work where you have measured a quality gain | Medium for well-specified agentic coding, High for harder or longer work; Extra and Max only where a quality gain is measured |

Three details change how you should read older advice:

- **Levels are recalibrated per model.** Opus 5.5 at Medium matches or exceeds Opus 5 at High on coding and knowledge-work evaluations. Settings from earlier models do not carry over.
- **Opus 5.5 thinks more per turn at a given level**, especially at Extra and Max, so an old Extra setting now costs more than it used to.
- **Defaults differ by surface.** A comparison run without setting effort pits Sonnet at High against Opus at Medium on the API, which is not a fair test.

In Claude Code, Max applies to the current session only, and its docs warn that Max may show diminishing returns and is prone to overthinking.

---

## The evidence

The strongest data comes from one practitioner's test on real work, not a launch benchmark. He ran both models at all five effort levels on six changes that were actually merged into a large production TypeScript monorepo: 20 runs per model per level, 300 in all, graded by the tests the original changes shipped with. A clean run passes every test, old and new.

| Effort | Opus 5.5 clean runs (of 20) | Opus 5.5 cost | Sonnet 5.5 clean runs (of 20) | Sonnet 5.5 cost |
|---|---|---|---|---|
| Low | 11 | $16.97 | 10 | $8.38 |
| Medium | 13 | $44.10 | 11 | $11.76 |
| High | 12 | $65.16 | 11 | $26.60 |
| Extra (xhigh) | 12 | $139.79 | 10 | $96.04 |
| Max | 11 | $307.16 | 13 | $415.18 |

Costs are API list prices for all 20 runs. Opus broke an already-passing test in 0, 0, 1, 1 and 2 runs from Low to Max. Sonnet broke none at any level. Every Opus regression was the same one: it rewrote an existing user-facing message and dropped the instruction that message carried.

**Benchmarks tell a similar cost story.** On the Artificial Analysis index, Sonnet 5.5 scores 36 at Low ($0.41 per task) and 47 at High ($1.08). Opus 5.5 scores 42 at Low ($0.55) and 54 at High ($1.82). At Max, Sonnet costs $7.60 per index task against $5.98 for Opus. Anthropic's own FrontierCode results show Opus at Medium (54.6%) slightly above Opus at Max (54.4%), and Sonnet scoring lower at Max (46.2%) than at Extra (52.1%).

**Build-from-scratch tests point somewhere else.** Two creators built the same app with Opus 5.5 at every level in Claude Code. Both found Extra gave the best balance. In one, Max took 2 hours 28 minutes and about $50 for a worse result, and Extra finished faster than High in the other.

**Why they disagree.** In the monorepo test, more effort meant bigger diffs: Opus's diff for one change grew from about 480 added lines at Medium to about 830 at Max. In code that already works, more rewriting means more chances to break something. In a new project there is nothing to break, so extra thoroughness pays. This is a plausible reading of the data, not a proven cause.

---

## The best case for the alternatives

The recommendations above are not the only defensible choices. Each alternative has a real argument behind it.

**For Sonnet at High or Extra.** Sonnet was the most careful model in the test: it broke no passing test and dropped no instruction at any level. It also streams faster than Opus (85 to 139 tokens per second against 74 to 93), and when a task's difficulty is genuinely reasoning depth, more effort can pay off. Its one gain, 13 clean runs at Max against 11 at Medium, came entirely from a set of edge cases it had missed at lower levels. Anthropic also reports Sonnet 5.5 at Max performing comparably to Opus 5.5 on several evaluations.

**For Opus at Low.** It buys the stronger model's judgment cheaply: 11 clean runs and zero regressions for 38% of Opus Medium's bill. Anthropic notes that on several coding evaluations Opus 5.5 at Low comes close to Opus 5 at High. The risk is edge cases: Low missed the edge-case-heavy change in two of three runs.

**For leaving everything on defaults.** The title of the monorepo study is "The Default Was Right", and the data supports it for Opus. Anthropic also ships Opus 5.5 with Medium as its default. The cost of overriding them is time spent re-testing, and the risk of carrying an old setting forward after a model update.

The case against the alternatives is mostly about cost and variance. At High and above, Sonnet's price advantage narrows or disappears, so the cheaper model stops being cheaper, and Anthropic itself says Sonnet complements Opus best at lower effort settings.

---

## Switching effort mid-session and prompt caching

Changing effort partway through a conversation can throw away the prompt cache, so the cost depends on where you change it.

| Where | What happens | What to do |
|---|---|---|
| Claude API, top-level effort | Changing it between requests invalidates cached prefixes. Anthropic's docs show Opus 5.5 going from a full cache read to zero cache reads when effort moved from Medium to Low. | Pick one level per conversation and vary it across workloads |
| Claude API, per-message effort (beta) | Preserves the cache on Opus 5.5, Opus 5 and Sonnet 5.5. It needs the beta header `mid-conversation-output-config-2026-07-01`. It is not available with Sonnet's `between_tools` thinking mode, which returns a 400 error if effort differs. | Use this when one conversation needs different effort per turn |
| Claude Code, `/effort` | On Opus 5.5 and Sonnet 5.5 with an API key or a Claude subscription, changing effort keeps the cache and applies without a prompt. On other models, and on Amazon Bedrock, Google Cloud's Agent Platform or a Claude apps gateway, it restarts the cache, and Claude Code asks you to confirm while the cache is warm. | Change effort freely on the 5.5 models; elsewhere, set it at the start of a session |

Model switches are different: each model has its own cache, so a `/model` switch rereads the whole conversation with no cache hits, and the cost applies once per switch. Pick the model at the start of a session. Two GitHub issues from earlier Claude Code versions report conflicting cache behavior on effort changes; the current documentation above supersedes them.

If only one turn needs more thinking, Claude Code's `ultrathink` keyword raises effort for a single prompt. This document does not cover how the claude.ai apps handle the cache.

---

## Confidence and how to check it on your own work

Overall confidence is medium. Every source agrees on the direction: Sonnet is cheap at Low and Medium, its price advantage fades from High upward, and Opus at Medium is a safe default. The exact crossover between Sonnet High and Opus Low is within noise.

Known limits of the evidence:

- **Small samples.** The monorepo test used three or four runs per change, and most gaps in its table are one or two runs.
- **One codebase, one author's prompts.** A different codebase or a vaguer ticket could move every number. The author also inferred the Claude effort setting rather than reading it from request logs.
- **Costs are list prices.** The test ran on subscriptions, so real bills differ. On a subscription, the same relative token spend shows up as usage limits instead of dollars.
- **Vendor and aggregator blogs.** Several comparison pages restate Anthropic's published figures, and none of the build-from-scratch tests are controlled studies.
- **The new-versus-existing-code split is a pattern.** It fits the diff-size explanation, but three small studies do not prove it.

> [!TIP]
> Check it on your own work
>
> Take five to ten recent changes with good tests, run each at Opus Low, Opus Medium and Sonnet Medium, and record three numbers per setting: runs that pass every test, existing tests broken, and cost. If Opus Low and Sonnet Medium tie on quality, choose by cost; if either one misses edge cases, move that task type up a level.

---

## Sources

Figures are as of the update date at the top. Two pages were read in full; the rest were read as search-result excerpts, so check them before quoting numbers elsewhere.

| Source | Used for | Read |
|---|---|---|
| [The Default Was Right (paddo.dev)](https://paddo.dev/blog/default-was-right/) | 300-run monorepo test, costs, regressions, diff size | In full |
| [Effort, Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/effort) | Defaults, per-model guidance, per-message effort, caching | In full |
| [Introducing Claude Sonnet 5.5 (Anthropic)](https://www.anthropic.com/claude-sonnet-5-5) | Positioning, Low and Medium cost claims | Excerpt |
| [Prompting Claude Opus 5.5 (Claude Platform Docs)](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5-5) | Opus 5.5 effort calibration | Excerpt |
| [Steering thinking (Claude Platform Docs)](https://platform.claude.com/docs/en/build-with-claude/thinking-steering-and-cost) | Medium-to-Low cache miss example | Excerpt |
| [How Claude Code uses prompt caching](https://code.claude.com/docs/en/prompt-caching) | Effort and model switching, other cache-invalidating actions | In full |
| [Claude Code issue 63962](https://github.com/anthropics/claude-code/issues/63962) and [issue 61984](https://github.com/anthropics/claude-code/issues/61984) | Conflicting reports on cache behavior | Excerpt |
| [Sonnet 5.5 vs Opus 5.5 (emergent.sh)](https://emergent.sh/learn/sonnet-5-5-vs-opus-5-5) | Artificial Analysis index scores and cost per task | Excerpt |
| [Sonnet 5.5 vs Opus 5.5 (myclaw.ai)](https://myclaw.ai/blog/sonnet-5-5-vs-opus-5-5) | FrontierCode figures as reported by Anthropic | Excerpt |
| [Opus 5.5 best practices (claudefa.st)](https://claudefa.st/blog/guide/development/opus-5-5-best-practices) | Opus Medium default, Claude Code Max warning | Excerpt |
| [Opus 5.5 at every effort level (openclawdatabase.com)](https://openclawdatabase.com/news/videos/2026-09-24-opus-55-every-effort-level-cost-comparison/) | Build-from-scratch test, Max cost and time | Excerpt |
| [Opus 5.5 built the same app 6 times (AI Fire)](https://www.aifire.co/p/claude-opus-5-5-built-the-same-app-6-times-and-extra-won) | Build-from-scratch test, Extra naming | Excerpt |
