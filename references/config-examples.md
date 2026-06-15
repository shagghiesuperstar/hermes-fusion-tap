# Fusion Tap — Annotated Config Examples

Full working configuration examples for local-only, cloud-only, and hybrid deployments.

---

## Local-Only Configuration

### `.hermes/.env` (local-only)

```bash
# No cloud API keys required
MOA_DISABLED=true

# Optional: name your local model aliases for clarity
# (these are referenced by your local provider, not by Hermes directly)
# Your local provider endpoint is set in config.yaml custom_providers
```

### `config.yaml` (local-only)

```yaml
model: local/your-model-name   # Replace with your local model identifier
provider: local                # or 'custom' if using custom_providers

toolsets:
  - hermes-cli
  - delegation       # REQUIRED for delegate fan-out
  - file             # REQUIRED for eval output
  - terminal         # recommended
  - code_execution   # recommended for script-based skills

agent:
  max_turns: 90

fusion:
  mode: roles                    # explicit role-pinned fusion
  reference_count: "3"
  judge_mode: balanced
  log_references: "false"
  roles:
    ref1:
      provider: custom:local
      model: your-local-model-a
    ref2:
      provider: custom:local
      model: your-local-model-b
    ref3:
      provider: custom:local
      model: your-local-model-c
    judge:
      provider: custom:local
      model: your-best-local-judge-model

custom_providers:
  - name: local
    base_url: http://localhost:8000/v1   # your local server (Ollama, MTPLX, mlx, etc.)
    model: your-model-name
    api_key: none                         # most local servers don't require auth
```

---

## Cloud-Only Configuration (OpenRouter + MOA)

### `.hermes/.env` (cloud-only)

```bash
OPENROUTER_API_KEY=sk-or-your-key-here
MOA_DISABLED=false
```

### `config.yaml` (cloud-only)

```yaml
model: anthropic/claude-opus-4   # or any OpenRouter model string
provider: openrouter

toolsets:
  - hermes-cli
  - moa              # enables mixture_of_agents tool
  - delegation       # fallback if MOA fails
  - file
  - terminal

agent:
  max_turns: 90

fusion:
  mode: roles
  judge_mode: strict  # full rubric in cloud mode (more tokens affordable)
  log_references: "true"
  roles:
    ref1:
      provider: openai-codex
      model: gpt-5.5
    ref2:
      provider: openrouter
      model: anthropic/claude-sonnet-4
    ref3:
      provider: openrouter
      model: google/gemini-2.5-pro
    judge:
      provider: openai-codex
      model: gpt-5.5
```

---

## Hybrid Configuration (Local Drafts + Cloud Judge)

### `.hermes/.env` (hybrid)

```bash
OPENROUTER_API_KEY=sk-or-your-key-here
ANTHROPIC_API_KEY=sk-ant-your-key-here
MOA_DISABLED=false

# Hybrid routing hints (used by orchestrator skill logic)
FUSION_LOCAL_MODEL=local/mlx-fast
FUSION_CLOUD_JUDGE=anthropic/claude-opus-4
```

### `config.yaml` (hybrid)

```yaml
model: local/mlx-fast    # primary/default model is local
provider: local

toolsets:
  - hermes-cli
  - moa              # for cloud fan-out when beneficial
  - delegation       # for local fan-out
  - file
  - terminal

agent:
  max_turns: 90

fusion:
  mode: hybrid
  reference_count: "3"
  judge_mode: balanced
  log_references: "true"
  discovery_enabled: "true"
  # Optional: set explicit roles for deterministic model assignment.
  # Missing roles are filled by discovery/ranking.
  roles:
    ref1:
      provider: custom:local
      model: mlx-fast
    ref2:
      provider: openrouter
      model: anthropic/claude-sonnet-4
    ref3:
      provider: openai-codex
      model: gpt-5.5
    judge:
      provider: openai-codex
      model: gpt-5.5

custom_providers:
  - name: local
    base_url: http://localhost:8000/v1
    model: mlx-fast
    api_key: none

fallback_providers:
  - provider: openrouter
    model: anthropic/claude-3.5-haiku   # fast cloud fallback if local unavailable
```

---

## Multi-Profile Setup (Recommended for Production)

```bash
# Main agent profile
hermes -p main chat

# Dedicated fusion-judge profile (optional — uses judge SOUL.md)
hermes -p fusion-judge chat

# Profile directory structure:
# ~/.hermes/profiles/main/
#   config.yaml, .env, SOUL.md, MEMORY.md
# ~/.hermes/profiles/fusion-judge/
#   config.yaml, .env, SOUL.md   ← judge-system-prompt.md content
```

---

## Skill Config via `hermes config migrate`

After installing the skills, run:

```bash
hermes config migrate
```

This prompts you to configure all skill config keys interactively:
- `fusion.mode`
- `fusion.reference_count`
- `fusion.roles.ref1.provider` / `fusion.roles.ref1.model`
- `fusion.roles.ref2.provider` / `fusion.roles.ref2.model`
- `fusion.roles.ref3.provider` / `fusion.roles.ref3.model`
- `fusion.roles.judge.provider` / `fusion.roles.judge.model`
- `fusion.discovery_enabled`
- `fusion.judge_mode`
- `fusion.log_references`
- `fusion.judge_strictness`
- `fusion.judge_show_scores`
- `fusion.eval_scenarios`
- `fusion.eval_output`

---

## Toolset Quick Reference for Fusion

| Toolset | Required For | Config Key |
|---------|-------------|------------|
| `delegation` | Delegate fan-out, Judge subagent | Always required |
| `moa` | Cloud MOA dispatch | Requires `OPENROUTER_API_KEY` |
| `file` | Eval results output | Required for fusion-eval |
| `terminal` | Script-based helper tasks | Recommended |
| `code_execution` | Complex aggregation logic | Recommended |
| `skills` | Loading skill files | Always required (default) |
