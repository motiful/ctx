# Reviewer vocabulary — say the precise word, get the precise fix

This is for you, not the agent. `ctx-report`'s STEP 0 already tries to map a rough
description back to the right term on its own — nothing here is required reading.
But naming the term directly skips that inference step entirely, so if you want the
sharpest, fastest fix, reach for the word instead of describing around it.

| What it feels like | Say this | What it triggers |
|---|---|---|
| "Something's off but I can't name it" | **hypocognition** | agent looks for the established term instead of parroting your words back |
| "You're just agreeing with everything I say" | **sycophancy** | agent re-verifies the claim instead of rubber-stamping it |
| "You listed four options and didn't pick one" | **argument to moderation** | decision-forcing: exhaust the research, converge to one ranked recommendation |
| "You gave up on this and handed it back to me without really trying" | **graceful-stop** (a *false* graceful degradation) | this was researchable — the agent should not have escalated it |
| "You quietly did less than I asked, without saying so" | **graceful-skip** (a *false* graceful degradation) | same family as graceful-stop — under-delivery dressed up as done |
| "This opening diagram is just your table of contents redrawn" | "this isn't a **question-map**" | should be STRUCTURE's laddering — one genuinely intriguing question per node, not a section label |
| "Did you check every angle, or just the easy ones?" | "this isn't **MECE**" (梳理 not exhaustive) | asks for the full sub-issue enumeration, no silent gaps |
| "What's the real question underneath this?" | "**ladder** this" (发掘) | asks the agent to dig from the surface question to the root one |

## Why this exists

This list came out of round-26 of the meta-report-craft thread: having `ctx-report`
loaded doesn't make precise vocabulary redundant. The skill governs the agent's
*output* discipline (STRUCTURE, decision-forcing, the question-map); precise
vocabulary reduces the agent's *inference* burden on the input side — same effect as
using a domain term with a specialist instead of describing around it. The two are
complementary, not substitutes.

This file is deliberately **not** wired into any skill's Execution Procedure — it is
read by a human, on their own schedule, not loaded by an agent mid-task. That is the
whole point: a reference an agent is supposed to load needs an explicit EP call or it
risks silent skip (see `skill-forge`'s own `references/skill-format.md` — "no EP call
= 100% skip"); a reference meant for a human doesn't have that failure mode, because
nothing depends on an agent noticing it.
