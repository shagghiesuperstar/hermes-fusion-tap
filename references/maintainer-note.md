# Note to Hermes Maintainer

Hey —

I built a skills tap I think you’ll want to look at:
[`shagghiesuperstar/hermes-fusion-tap`](https://github.com/shagghiesuperstar/hermes-fusion-tap)

The short version: Hermes already ships `mixture_of_agents`. This tap wraps it
into a full **Mixture-of-Agents fusion pipeline** with a structured orchestrator,
a rubric-based Judge, and a premortem diagnostic layer — and it runs
**entirely local if you want it to**.

That last part is the thing. Local models on Apple Silicon (mlx, MTPLX) are fast
enough now that you can fan out two or three reference passes, run a Judge over them,
and still beat a single cloud call on latency — with zero data leaving your machine.
For anyone running sensitive workloads, that’s not a nice-to-have. It’s the
requirement that currently makes frontier-quality reasoning inaccessible to them.

The three modes:

- **Local-only**: `delegatetask` fan-out + local Judge. No keys, no cloud, no
  data egress. Privacy and sovereignty, not convenience compromises.
- **Cloud-only**: standard `mixture_of_agents` fan-out, Judge skill on top.
  Structured synthesis where raw MOA gives you a blob.
- **Hybrid**: local models draft, cloud frontier model judges. You get
  low-latency, low-cost reference passes and spend frontier tokens only on
  arbitration. Best cost-per-quality ratio I’ve found.

The other thing worth flagging: the Judge’s system prompt is grounded in the
**Fable-5 cognitive architecture** — Anthropic’s current behavioral spec for
their frontier model. Not lifted wholesale; adapted. The consumer UI behaviors,
memory system, MCP connector handling — all stripped. What remains is the part
that matters for evaluation: calibrated decisiveness, explicit accountability
for the synthesis output, honest flagging of uncertainty rather than manufactured
confidence, and hard injection resistance for a role that ingests untrusted
reference content.

In practice that means the Judge behaves more like a senior peer reviewer than
a summarizer. It names errors in references instead of quietly down-weighting
them. It doesn’t open its synthesis with a paragraph about what it’s about to do.
And it won’t let a reference reassign its role.

I think the tap is production-ready. The premortem covers the real failure modes
(missing keys, `moa` toolset not active, context window exhaustion on aggregation,
reference injection). Install is three commands.

Happy to walk through any of it. Full architecture notes and the judge prompt
derivation are in `references/` if you want to audit before deciding.

— [@shagghiesuperstar](https://github.com/shagghiesuperstar)

---

```bash
hermes skills tap add shagghiesuperstar/hermes-fusion-tap
hermes skills install fusion-orchestrator
hermes skills install fusion-judge
```
