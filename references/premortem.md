# Fusion Tap — Premortem Failure Mode Registry

> A premortem is a prospective analysis: before deployment, imagine the system has failed and work backwards to identify causes. This document catalogs every failure mode identified during design, with mitigations built into the skill procedures.

---

## How to Use This Document

1. Read before first deployment
2. Check the "Env Var Checklist" section against your `.env`
3. Check the "Config Checklist" against your `config.yaml`
4. Run `fusion-eval` after setup to validate end-to-end
5. Reference this document when debugging unexpected behavior

---

## Category 1 — Environment / Configuration Failures

### FM-001: Missing OPENROUTER_API_KEY in MOA mode
**Probability**: High (most common first-run failure)
**Impact**: Fusion falls back to delegate mode silently, or crashes with auth error
**Detection**: Step 0 premortem check in fusion-orchestrator
**Mitigation**: Orchestrator detects missing key → auto-switches to delegate mode → notifies user
**Remediation**: Add `OPENROUTER_API_KEY=sk-or-...` to `.hermes/.env`

### FM-002: MOA_DISABLED not set (undefined ≠ false)
**Probability**: Medium
**Impact**: Orchestrator may attempt MOA when local-only is desired
**Detection**: Env var check in Step 0
**Mitigation**: Orchestrator treats undefined `MOA_DISABLED` as `false`; if `OPENROUTER_API_KEY` absent, auto-switches to delegate
**Remediation**: Explicitly set `MOA_DISABLED=true` in `.env` for local-only deployments

### FM-003: `delegation` toolset not enabled
**Probability**: Medium
**Impact**: Both MOA and delegate modes require some delegation capability; skill cannot function
**Detection**: Step 0 toolset check
**Mitigation**: Orchestrator aborts with clear error message listing required toolsets
**Remediation**: Add `delegation` to `toolsets` in `config.yaml`

### FM-004: fusion-judge skill not installed
**Probability**: Low-Medium (common if user installs only fusion-orchestrator)
**Impact**: Step 3 judge call fails
**Detection**: `skill_view fusion-judge` returns not found
**Mitigation**: Orchestrator falls back to Inline Judge procedure (built into Step 3)
**Remediation**: `hermes skills install fusion-judge`

### FM-005: Wrong HERMESHOME — skills installed to wrong profile
**Probability**: Low (affects multi-profile setups)
**Impact**: Skill installed but not visible to target profile
**Detection**: `hermes skills list` doesn't show fusion skills
**Mitigation**: Use `hermes -p <profile> skills install fusion-orchestrator`
**Remediation**: Verify `HERMES_PROFILE` or `-p` flag matches target profile

---

## Category 2 — Runtime / Tool Failures

### FM-006: Context window exhaustion during aggregation
**Probability**: Medium (large queries + 3–4 reference responses)
**Impact**: API call fails or response truncated
**Detection**: Token budget estimate in Step 0; compression triggered if >70% budget
**Mitigation**: Judge switches to `fast` mode; reference responses compressed before judge pass
**Remediation**: Reduce `fusion.reference_count`; use a model with larger context window

### FM-007: `mixture_of_agents` tool call fails (network/rate limit)
**Probability**: Low-Medium
**Impact**: Step 2A fails entirely
**Detection**: Exception from tool call
**Mitigation**: Auto-retry once; if second attempt fails, auto-switch to delegate mode
**Remediation**: Check OpenRouter API status; verify API key has sufficient credits

### FM-008: Delegate subagents return near-identical responses (low diversity)
**Probability**: Medium (common when same model runs all passes)
**Impact**: Judge has nothing to synthesize; output ≈ single-model quality
**Detection**: Judge step — text similarity check across references
**Mitigation**: Log diversity warning; reduce reference count for future runs; recommend different model variants
**Remediation**: Configure different models per delegate pass via custom `delegate_task` system messages; use different temperature settings

### FM-009: Subagent delegation timeout
**Probability**: Low
**Impact**: One or more reference responses missing
**Detection**: `delegate_task` returns timeout error
**Mitigation**: Proceed with available references (≥2 required for meaningful fusion)
**Remediation**: Increase `agent.max_turns` in config; check model latency

---

## Category 3 — Security / Adversarial Failures

### FM-010: Prompt injection via reference model output
**Probability**: Low (but high impact if triggered)
**Impact**: Judge role manipulated; synthesis corrupted
**Detection**: Sanitization step in fusion-judge Step 0
**Mitigation**: Strip `[JUDGE INPUT]`, `[FUSION REFERENCE REQUEST]`, and other control markers from all reference text before judge processing
**Remediation**: If sanitization bypassed, discard all reference outputs and re-run with `delegation` mode only (not MOA, which cannot be inspected mid-flight)

### FM-011: API key leakage in judge output
**Probability**: Very Low (Hermes never exposes env vars to model)
**Impact**: Credential exposure in session transcript
**Detection**: Post-synthesis scan of output
**Mitigation**: Hermes env var passthrough never injects raw key values into model context; secrets stay in `.env` only
**Remediation**: Rotate any exposed key immediately

---

## Category 4 — Quality Failures

### FM-012: All references confidently wrong (coordinated hallucination)
**Probability**: Low (but possible for obscure facts)
**Impact**: Judge synthesizes a confident wrong answer
**Detection**: Calibration score in Judge rubric; judge flags overconfidence
**Mitigation**: Judge is instructed to flag disagreements and penalize overconfidence; user warned when confidence is Medium or Low
**Remediation**: Add web search step before fusion for factual queries: `search → fuse → judge`

### FM-013: Ambiguous query produces divergent reference interpretations
**Probability**: Medium
**Impact**: References answer different questions; synthesis incoherent
**Detection**: Judge Step 2 identifies irreconcilable divergence
**Mitigation**: Judge flags ambiguity, presents both interpretations, asks user to clarify
**Remediation**: Re-run fusion with clarified query

---

## Env Var Checklist

Before first run, verify:

```bash
# Required for cloud MOA mode
OPENROUTER_API_KEY=sk-or-...       # ✅ or skip if local-only

# Required for local-only mode
MOA_DISABLED=true                  # ✅ set explicitly

# Optional — for hybrid mode
ANTHROPIC_API_KEY=sk-ant-...       # for Judge escalation to frontier model
FUSION_LOCAL_MODEL=local/mlx-fast  # local model identifier
FUSION_CLOUD_JUDGE=anthropic/claude-opus-4  # cloud judge model
```

---

## Config Checklist

```yaml
# config.yaml minimum for fusion to work
toolsets:
  - hermes-cli      # always
  - delegation      # REQUIRED for delegate mode
  - file            # REQUIRED for eval output
  - terminal        # recommended
  - moa             # REQUIRED for MOA mode (add only if OPENROUTER_API_KEY set)

agent:
  max_turns: 90     # default is fine; increase if subagents timeout
```

---

## Doctor Diagnostic Commands

Run these to verify your fusion setup:

```bash
# Verify skills installed
hermes skills list | grep fusion

# Check toolsets active
hermes config get toolsets

# Check env vars (names only — values never printed)
hermes config get terminal.env_passthrough

# Run eval harness
hermes chat
> skill_view fusion-eval
> Run the fusion eval harness
```

---

*Last updated: 2026-06-14. Update this document when adding new failure modes discovered in production.*
