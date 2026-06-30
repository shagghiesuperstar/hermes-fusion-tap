# hermes-fusion-tap

> **Community Hermes Agent Skills Tap** — Fusion Orchestration System with Mixture-of-Agents Judge, Premortem Diagnostics, and Local/Cloud Hybrid support.

```
hermes skills tap add shagghiesuperstar/hermes-fusion-tap
hermes skills install fusion-orchestrator
hermes skills install fusion-judge
hermes skills install fusion-eval        # optional — testing harness
hermes skills install hermes-tweet       # optional - X/Twitter workflows
```

---

## Philosophy

Single-model inference is a single point of epistemic failure. No model — regardless of size — reliably knows what it doesn't know. The *fusion paradigm* routes a problem through multiple independent reference models, then applies a calibrated Judge to synthesize a superior answer. This mirrors how expert panels, peer review, and adversarial red-teaming work in human systems: **diversity of reasoning paths → reduced systematic bias → higher-quality outputs.**

Hermes Agent already ships a `mixture_of_agents` tool (MOA) that does exactly this at the API level. But raw MOA is a black box — the calling agent delegates and receives a blob. The Fusion Orchestrator skill wraps MOA with:

1. **Structured dispatch** — the orchestrator explicitly frames the query for each reference model's strengths.
2. **Judge arbitration** — a dedicated role evaluates and synthesizes reference outputs against a rigorous rubric.
3. **Transparency layer** — every step is logged, auditable, and surfaced to the user.
4. **Premortem gating** — known failure modes are checked *before* dispatch, not after.

---

## Genesis / Background

This tap was authored on 2026-06-14 based on the Hermes Agent Master Architecture Reference (SSOT compiled by Nous Research). The impetus: production agentic deployments running across local Apple Silicon rigs (mlx/MTPLX), private cloud (Modal, SSH), and public API providers (OpenRouter, Anthropic, OpenAI) needed a **universal, provider-agnostic fusion layer** that could run entirely local, entirely cloud, or in a hybrid configuration — with no code changes, only environment variable and config differences.

The fusion model is inspired by:
- **Wang et al. (2023)** — Mixture-of-Agents improves reasoning by aggregating diverse model outputs.
- **Liang et al. (2024)** — Cascaded LLM arbitration reduces hallucination on contested facts.
- **Constitutional AI / RLAIF patterns** — using LLMs as judges for evaluation and correction.
- The practical insight that **local models excel at speed and privacy; cloud frontier models excel at breadth and reasoning**; a well-designed fusion harness captures both.

---

## Design Intention

| Goal | Mechanism |
|------|-----------|
| Provider-agnostic | No hardcoded model strings in skill logic; all resolved via `config.yaml` + env vars |
| Local-only capable | Works with `delegatetask` + local backends (mlx, MTPLX, Ollama via OpenAI-compat) |
| Cloud-hybrid capable | Uses `mixture_of_agents` tool when OpenRouter key present |
| Privacy-preserving | Sensitive queries never leave local terminal when `MOA_DISABLED=true` |
| Auditable | Every reference response and judge decision logged to session memory |
| Composable | Skills are independent; install only what you need |
| First-run reliable | Premortem checklist + doctor diagnostics prevent silent misconfiguration |

---

## Repository Structure

```
hermes-fusion-tap/
├── README.md                          # This file
├── skills.sh.json                     # Tap catalog metadata
├── skills/
│   ├── fusion-orchestrator/
│   │   └── SKILL.md                   # Primary skill — routes queries through fusion pipeline
│   ├── fusion-judge/
│   │   └── SKILL.md                   # Judge role skill — rubric-based arbitration
│   ├── fusion-eval/
│   │   └── SKILL.md                   # Optional — test harness for calibration
│   └── hermes-tweet/
│       └── SKILL.md                   # Optional - X/Twitter workflow skill
└── references/
    ├── premortem.md                   # Failure mode analysis + mitigations
    ├── judge-system-prompt.md         # JUDGE role system prompt (Fable-5 derived)
    └── config-examples.md             # Annotated config.yaml + .env examples
```

---

## Use Case Examples

### 1. Local-Only (No Cloud Keys)

```bash
# .env — no API keys needed
MOA_DISABLED=true
FUSION_REFERENCE_MODELS=local-a,local-b   # your local model aliases

# config.yaml
model: local/your-mlx-model
provider: local
toolsets:
  - hermes-cli
  - delegation
  - codeexecution
  - file
  - terminal
```

In this mode, the orchestrator spawns two `delegatetask` subagents with different system prompt variants (creative vs. analytical), then runs the fusion-judge skill as a third pass. Entirely local, no external calls.

### 2. Cloud-Only (OpenRouter + MOA)

```bash
# .env
OPENROUTER_API_KEY=sk-or-...
MOA_DISABLED=false

# config.yaml
model: anthropic/claude-opus-4
provider: openrouter
toolsets:
  - hermes-cli
  - moa          # enables mixture_of_agents tool
  - delegation
  - file
```

The orchestrator uses `mixture_of_agents` directly for reference model fan-out, then applies the Judge skill for structured synthesis.

### 3. Hybrid (Local Fast Draft + Cloud Deep Eval)

```bash
# .env
OPENROUTER_API_KEY=sk-or-...
ANTHROPIC_API_KEY=sk-ant-...
FUSION_LOCAL_MODEL=local/mlx-fast
FUSION_CLOUD_JUDGE=anthropic/claude-opus-4

# config.yaml
model: local/mlx-fast
provider: local
toolsets:
  - hermes-cli
  - moa
  - delegation
  - file
  - terminal
```

Draft responses generated locally at low latency; Judge role escalates to cloud frontier model only for arbitration. Optimal cost/quality/privacy tradeoff.

---

## Installation

```bash
# 1. Add this tap
hermes skills tap add shagghiesuperstar/hermes-fusion-tap

# 2. Install skills
hermes skills install fusion-orchestrator
hermes skills install fusion-judge
hermes skills install fusion-eval   # optional

# 3. Run setup config migration
hermes config migrate

# 4. Verify environment
cat references/premortem.md   # review failure modes
```

---

## Premortem

See [`references/premortem.md`](references/premortem.md) for the full failure mode registry with mitigations. Key items:

- Missing `OPENROUTER_API_KEY` when `MOA_DISABLED=false` → skill detects and falls back to delegation mode
- `moa` toolset not active → orchestrator gracefully switches to `delegatetask` fan-out
- Context window exhaustion during multi-model aggregation → compression triggered automatically; Judge receives compressed summaries
- Judge prompt injection via reference model output → sanitization step built into judge procedure

---

## License

MIT — see individual skill files for per-skill licensing.

---

*Maintained by [@shagghiesuperstar](https://github.com/shagghiesuperstar). Compiled 2026-06-14.*
