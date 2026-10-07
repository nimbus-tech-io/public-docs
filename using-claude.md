---
layout: doc
title: Using Claude Effectively
subtitle: A friendly guide to choosing a model, planning a change, and managing context.
mermaid: true
updated: 2026-10-07
---

## Model overview

We work with three models, each with its own price and job. A good rule of thumb is to start with the most affordable one that can handle the task well.

| Model | Model ID | Context | Input&nbsp;$/1M | Output&nbsp;$/1M | Role |
|---|---|---|---|---|---|
| Claude Opus 5.5 {% include badge.html m="opus" %} | `claude-opus-5-5` | 1M | $4.00 | $20.00 | Planning, architecture, and anything open-ended |
| Claude Sonnet 5.5 {% include badge.html m="sonnet" %} | `claude-sonnet-5-5` | 1M | $2.00 | $10.00 | Implementation and everyday work |
| Claude Haiku 4.5 {% include badge.html m="haiku" %} | `claude-haiku-4-5` | 200K | $1.00 | $5.00 | Fast, light, simple tasks |

Prices are [Anthropic API list prices](https://platform.claude.com/docs/en/about-claude/pricing) per million tokens, as of October 2026. We use Claude through a subscription, so the same price differences show up as how quickly you use your limits rather than as dollars. Opus costs about twice as much as Sonnet, so it uses your allowance about twice as fast.

You'll need a recent version of Claude Code for the newest models: v2.1.280 or later for Opus 5.5, and v2.1.284 or later for Sonnet 5.5. If a model isn't available, run `claude update`.

> A good default: plan with Opus, build with Sonnet.
>
> Opus shines when a problem is vague or open-ended, because that's where its thinking pays off. Once you have a clear plan, Sonnet can carry it out for about half the price and gets you to the same place. Opus can absolutely write code too. To decide which model and effort level suit a particular piece of work, see our [model and effort guide](https://claude.ai/artifact/Lc9sQEngP7jNFW8SRbXrVe).
{: .callout .callout-tip}

---

## How to approach a change

Take a moment to pick a workflow before you write your first prompt. These three questions will get you there:

1. **Is the change simple?** Describe what you want directly to Sonnet or Haiku. No planning needed.
2. **Is it complex, with no domain knowledge you need to preserve?** Use [grill-me](https://www.aihero.dev/my-grill-me-skill-has-gone-viral) in plan mode, then implement with Sonnet.
3. **Is it complex, with domain knowledge worth preserving?** Use [OpenSpec](https://github.com/Fission-AI/OpenSpec/). Plan and write the spec in Opus, then implement in Sonnet.
{: .flow-steps}

```mermaid
flowchart TD
    start(["<span style='color:#f5f0e8;font-weight:700;letter-spacing:.5px'>START</span>"]) --> q1{Is the change<br/>complex?}
    q1 -- NO --> direct[Describe the change directly to<br/>Sonnet or Haiku —<br/>no planning needed.]
    q1 -- YES --> q2{Domain knowledge<br/>to preserve?}
    q2 -- NO --> grill["<a href='https://www.aihero.dev/my-grill-me-skill-has-gone-viral'><i>grill-me</i></a><br/>+ plan mode, then<br/>implement with Sonnet."]
    q2 -- YES --> openspec["<a href='https://github.com/Fission-AI/OpenSpec/'><i>OpenSpec</i></a><br/>Plan & spec in Opus,<br/>implement in Sonnet."]

    classDef startNode fill:#3a3020,stroke:#3a3020,color:#f5f0e8,font-weight:700
    classDef green fill:#e3ead9,stroke:#9cc4a6,color:#1a4a2e
    classDef blue fill:#e3e5e5,stroke:#93a9d0,color:#1a3570
    classDef purple fill:#e9e0e5,stroke:#ad98cf,color:#38107a
    class start startNode
    class direct green
    class grill blue
    class openspec purple
```

---

## Medium complexity: `opusplan` + `grill-me`

For changes that are too involved to describe in one go but don't need a lasting record of the reasoning, `opusplan` with `grill-me` is a lovely place to start. It's also a great default if you're newer to Claude: a simple plan-then-implement flow that works well most of the time, with no manual model switching.

Select the `opusplan` model and invoke [`grill-me`](https://www.aihero.dev/my-grill-me-skill-has-gone-viral) from plan mode. Opus explores the codebase and talks the problem through with you, then produces a plan. Once you approve it, `opusplan` hands off to Sonnet for the implementation.

1. Select the `opusplan` model and enter plan mode.
2. Invoke `grill-me`. Opus looks through the codebase and asks clarifying questions, surfacing edge cases, hidden assumptions, and possible failure modes. Your starting input can be rough.
3. Answer the questions. When the conversation wraps up, Opus writes a well-informed plan for you to review.
4. Approve the plan. `opusplan` switches to Sonnet to implement it.
{: .flow-steps}

As you get more comfortable with Claude, feel free to mix and match: choose your own model for each step, adjust effort levels, or skip planning entirely for small tasks. The [model and effort guide](https://claude.ai/artifact/Lc9sQEngP7jNFW8SRbXrVe) is there when you're ready.

If the work involves domain knowledge, architectural trade-offs, or decisions people will need to understand months from now, a plan on its own isn't quite enough. That's where OpenSpec comes in.

---

## OpenSpec: domain knowledge and architecture

Some work has a strong domain component, such as custom business logic, non-obvious architectural constraints, or decisions with long-term consequences. For that kind of work, it helps to capture the *why* alongside the plan, so that future implementers (human or AI) understand the reasoning and not just the outcome.

[OpenSpec](https://github.com/Fission-AI/OpenSpec/) is a workflow built for this. A spec document is committed alongside the code, which makes the rationale a first-class part of the repo.

### When OpenSpec is a good fit

- The feature touches domain rules that aren't obvious from the code.
- There are real trade-offs between approaches that are worth recording.
- A future developer would otherwise have to reverse-engineer the intent.
- You expect to revisit this area and want the context to still make sense in six months.

### Recommended workflow

1. Open a fresh conversation with Opus. Its 1M context gives you plenty of room for background docs, existing code, and a long planning discussion.
2. Work through the problem with Opus, weigh the options, and arrive at a plan.
3. Ask Opus to write the spec. It should capture the goal, the options you considered, the decision you made, and the reasons why.
4. Open a new window with Sonnet and hand it the spec. Sonnet carries out the plan, and you save your Opus usage for the thinking.
{: .flow-steps}

---

## Context management: stay or start fresh?

Every conversation builds up context, so at some point you'll choose between carrying on in the same window and starting a new one. One thing worth knowing: each model keeps its own cache, so switching models with `/model` in the middle of a session means the new model has to reread the whole conversation from scratch. You pay that cost once per switch, and it can eat into the savings from moving to a cheaper model.

**After a short planning session.** If the session was short and the plan is modest, there's no need to switch. Stay with the model you started on, or use `opusplan` and let it do the handoff for you.

**After a long planning session.** If planning and exploration have used up most of your context, end the session with a handover document and open a fresh window on Sonnet. A new window has no cache to lose, and Sonnet gets the distilled insight without wading through the whole conversation. This is the same approach we recommend after an OpenSpec session.

A good handover document includes:

- the decision that was made
- the reasoning behind it
- any constraints you uncovered
- the exact next steps

Paste it into the new window as your opening prompt. Even when two models have the same context window, a long planning conversation is noisy, so ask the current session to write the handover document before you move on. It only takes a minute and makes the next step much smoother.

---

## Quick reference

| Situation | Approach | Model |
|---|---|---|
| Simple change, no planning needed | Describe the change directly | {% include badge.html m="sonnet" %} or {% include badge.html m="haiku" %} |
| Vague or open-ended request | Think it through with Opus first | {% include badge.html m="opus" %} |
| Complex, no domain knowledge to preserve | `opusplan` + `grill-me` in plan mode | `opusplan` |
| Complex, domain knowledge worth preserving | OpenSpec: plan and spec in Opus, implement in Sonnet | {% include badge.html m="opus" %} → {% include badge.html m="sonnet" %} |
| You have a good plan | Implement it | {% include badge.html m="sonnet" %} |

Not sure which setup fits your task? Our [model and effort guide](https://claude.ai/artifact/Lc9sQEngP7jNFW8SRbXrVe) goes deeper.
