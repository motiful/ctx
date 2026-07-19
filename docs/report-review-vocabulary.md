# Giving feedback ctx-report acts on precisely

`ctx-report` already tries to map your wording onto its own controlled terms before
answering. Naming the term yourself skips that guesswork and gets you a sharper,
faster fix — the same gain you get from using a domain term with a specialist
instead of describing around it.

| What you're seeing | Say this | What it triggers |
|---|---|---|
| Something's off but you can't name it | **hypocognition** | ctx-report looks for the established term instead of parroting your words back |
| The agent is just agreeing with everything you say | **sycophancy** | it re-verifies the claim instead of rubber-stamping it |
| It listed four options and didn't pick one | **argument to moderation** | decision-forcing kicks in: exhaust the research, converge to one ranked recommendation |
| It handed a decision back to you that it could have researched itself | **false graceful degradation** — specifically **graceful-stop** | this was the agent's call to make; it should not have escalated |
| It quietly did less than you asked, without saying so | **false graceful degradation** — specifically **graceful-skip** | same failure family as graceful-stop: under-delivery dressed up as done |
| The opening diagram just restates the table of contents | not a real **question-map** | should be one genuinely intriguing question per node, built from STRUCTURE's own laddering |
| You suspect it checked the easy angles and stopped | not **MECE** (梳理 incomplete) | asks for the full sub-issue enumeration, no silent gaps |
| You want the question behind the question | ask it to **ladder** (发掘) | digs from the surface question to the root one |

## Why bother

The skill and the vocabulary do different jobs. `ctx-report` governs the agent's
*output* discipline — STRUCTURE, decision-forcing, the question-map — regardless of
how you phrase your comment. Precise vocabulary reduces the agent's *input-side*
inference error: it no longer has to guess which failure mode you mean. Loading the
skill doesn't make the vocabulary redundant; they solve different halves of the same
problem.

This page is for you, not the agent — it isn't wired into any skill's Execution
Procedure, so read it whenever it's useful, not on any schedule.
