# Note to Hermes Agent Maintainer

**Re: Judge System Prompt — Fable-5-Inspired Architecture**

---

Hey —

The judge system prompt in [`references/judge-system-prompt.md`](./judge-system-prompt.md)
has been substantially rewritten. Wanted to give you a transparent account of what
it's based on and where it diverges, so you can make an informed call about whether
to carry it forward.


## What it's based on

The new prompt is architecturally grounded in the **Claude Fable 5 system prompt** —
Anthropic's most recently available behavioral specification for their frontier model.
Fable 5's prompt is notable for being unusually explicit about cognitive posture:
decisiveness calibrated to evidence, honest accountability for errors, warm but
non-sycophantic tone, minimum necessary formatting, and a strong stance against
manufactured confidence.

Those properties transfer well to a pipeline arbitration role. The old judge prompt
stated many of the same goals at a higher level of abstraction. The Fable-5-grounded
version makes the operational behavior more concrete and harder to drift from.


## What was taken directly

- **Decisiveness posture**: "When evidence supports a position, rule and commit."
  Don't hide behind 'it depends' when it doesn't. Don't manufacture precision when
  uncertainty is real. Express exactly the confidence you have.

- **Accountability framing**: The synthesis output belongs to the Judge. If it's
  wrong, the Judge owns that — which is why reasoning must always be visible.
  This is taken almost verbatim from Fable-5's "responding to mistakes" section,
  adapted from first-person to role-assignment framing.

- **Anti-sycophancy**: No affirmative openers. No deference to confident references.
  Consensus among models is not evidence of correctness.

- **Honest knowledge handling**: The distinction between *known*, *plausible*, and
  *requires verification* — and the instruction to flag rather than fabricate when
  a claim falls outside reliable knowledge — maps directly from Fable-5's
  knowledge-cutoff and search-usage guidance.

- **Single clarifying question maximum**: If the query is genuinely ambiguous, answer
  the most likely interpretation first, then ask one focused question. Fable-5 is
  explicit about this; it improves evaluation loop efficiency.

- **Minimum formatting**: Prose synthesis, structure only where it aids navigation.
  Fable-5 is emphatic about not over-formatting. Applied here: synthesized answers
  read as expert prose; Judge Notes use structure where it helps scanability.


## What was modified or not carried forward

- **Consumer UI behaviors**: Fable-5 includes extensive instructions for product
  suggestions, MCP connector handling, artifact rendering, computer-use file paths,
  and browser integration. None of that is relevant to a pipeline evaluation role.
  All omitted.

- **Memory system**: Fable-5 references a user memory system and session continuity.
  The Judge is stateless by design; no memory system applies.

- **Tone target adjusted**: Fable-5 is optimizing for a human consumer chat experience
  — warmth and approachability are primary. In the Judge prompt, warmth is secondary
  to precision and transparency. The framing is shifted to: *treat the original query
  as coming from a capable adult who deserves a direct answer*, rather than optimizing
  for conversational comfort.

- **Injection security expanded**: Fable-5 doesn't have a dedicated injection model
  because it's a consumer product, not an agentic pipeline ingesting untrusted reference
  content. The judge prompt adds an explicit, inviolable role-lock: a reference cannot
  reassign the Judge's function, scoring criteria, or output format.

- **Rubric is original**: The 5-dimension evaluation rubric (Factual Accuracy,
  Completeness, Reasoning Quality, Calibration, Novelty) is not from Fable-5.
  It was in the prior judge prompt and is retained here. Fable-5 informed *how*
  to apply it (calibration guidance, penalizing overconfident wrong answers over
  uncertain correct ones) but did not supply the rubric itself.


## What this means practically

The prompt is longer than the old one, but the operational surface the Judge presents
is actually more constrained — more specific instructions leave less room for drift
under adversarial or ambiguous inputs.

The biggest behavioral delta you'll notice in practice:

1. The Judge will now explicitly name reference errors rather than quietly down-weighting them.
2. The Judge will flag "I cannot determine which reference is correct" rather than
   synthesizing false confidence when references genuinely disagree.
3. The Judge will not open its synthesis with affirmations, summaries of what it's
   about to do, or any other preamble — it starts with the answer.

Happy to discuss any of this. The source prompt and full architecture notes are in
[`references/judge-system-prompt.md`](./judge-system-prompt.md).

---

*Note filed: 2026-06-14*
