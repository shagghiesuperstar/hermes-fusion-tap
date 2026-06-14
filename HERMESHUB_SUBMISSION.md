# HermesHub Submission Guide

This document walks through the exact process to submit the fusion tap skills to [hermeshub.xyz](https://hermeshub.xyz) — the official skills registry for Hermes Agent.

---

## What Gets Submitted

Three skills, each as their own directory under `skills/` in the hermeshub repo:

| Skill | hermeshub path | Category |
|-------|---------------|----------|
| `fusion-orchestrator` | `skills/fusion-orchestrator/` | Agents & Swarms |
| `fusion-judge` | `skills/fusion-judge/` | Agents & Swarms |
| `fusion-eval` | `skills/fusion-eval/` | Agents & Swarms |

Each directory contains only `SKILL.md`. No additional files needed — all reference material is in the tap repo and linked from the SKILL.md Source sections.

---

## SKILL.md Frontmatter Spec (hermeshub)

The canonical format used by hermeshub (derived from [agentskills.io](https://agentskills.io)):

```yaml
---
name: skill-name              # kebab-case, matches directory name
description: "..."            # Full sentence. Describes what, when, and why.
version: "1.0.0"              # Semver string
license: MIT                  # SPDX identifier
compatibility: "..."          # What Hermes version/toolsets are required
metadata:
  author: github-username
  hermes:
    tags:                     # Array of lowercase keyword strings
      - tag1
      - tag2
    category: agents          # One of: development, research, productivity,
                              # security, data, devops, communication, agents, docs, meta
    requires_tools:           # Array of tool names; empty array [] if none
      - tool_name
    related_skills:           # Optional: array of skill names that compose well
      - other-skill
---
```

**Fields that cannot be empty:**
- `name` — must match the directory name exactly
- `description` — the text that appears on hermeshub.xyz cards. Should be a complete sentence covering: what it does, when to use it, what's distinctive.
- `version` — must be a quoted semver string
- `license` — must be a valid SPDX identifier
- `compatibility` — explicitly state required Hermes version and any required toolsets
- `metadata.author` — your GitHub username
- `metadata.hermes.tags` — minimum 5 tags for good discoverability
- `metadata.hermes.category` — must match one of the eight hermeshub categories
- `metadata.hermes.requires_tools` — use `[]` explicitly if no tools needed, don't omit

---

## Step-by-Step Submission Process

### 1. Fork hermeshub

```bash
# Fork via GitHub UI at https://github.com/amanning3390/hermeshub
# Then clone your fork:
git clone https://github.com/YOUR-USERNAME/hermeshub.git
cd hermeshub
```

### 2. Create the skill directories

```bash
mkdir -p skills/fusion-orchestrator
mkdir -p skills/fusion-judge
mkdir -p skills/fusion-eval
```

### 3. Copy the hermeshub-format SKILL.md files

Copy from this tap repo:

```bash
cp /path/to/hermes-fusion-tap/skills/fusion-orchestrator/SKILL.md skills/fusion-orchestrator/
cp /path/to/hermes-fusion-tap/skills/fusion-judge/SKILL.md skills/fusion-judge/
cp /path/to/hermes-fusion-tap/skills/fusion-eval/SKILL.md skills/fusion-eval/
```

### 4. Test locally

```bash
# Copy skills to Hermes local skill directory
cp -r skills/fusion-orchestrator ~/.hermes/skills/
cp -r skills/fusion-judge ~/.hermes/skills/
cp -r skills/fusion-eval ~/.hermes/skills/

# Verify Hermes sees them
hermes skills list

# Run the eval skill to confirm the pipeline works
hermes skills load fusion-eval
```

### 5. Open a Pull Request

```bash
git checkout -b add-fusion-skills
git add skills/fusion-orchestrator skills/fusion-judge skills/fusion-eval
git commit -m "feat: add fusion-orchestrator, fusion-judge, fusion-eval skills

Mixture-of-Agents fusion pipeline with rubric-based Judge arbitration.
Supports local-only, cloud-only, and hybrid modes. Three-skill suite:
- fusion-orchestrator: MOA pipeline with automatic mode detection
- fusion-judge: rubric-based synthesis with injection resistance
- fusion-eval: test harness and calibration diagnostics

Tap: github.com/shagghiesuperstar/hermes-fusion-tap"
git push origin add-fusion-skills
```

Then open a PR from your fork to `amanning3390/hermeshub:main`.

---

## Security Scanner

hermeshub runs `scripts/scan-skill.py` via GitHub Actions on every PR. It checks for:

- Data exfiltration patterns (curl/wget POSTs)
- Prompt injection and social engineering language
- Destructive commands
- Obfuscation (hex-encoded strings, unicode smuggling)
- Hardcoded secrets and credentials
- Suspicious network calls
- Environment variable manipulation
- Supply-chain attack vectors

**Our skills pass cleanly because:**
- No external network calls issued from skill instructions
- No credentials hardcoded (OPENROUTER_API_KEY declared as an env var requirement, not hardcoded)
- No destructive commands
- No obfuscation
- Injection resistance is a feature, not a vulnerability — the scanner won't flag defensive pattern documentation

The scanner posts results as a PR comment. Critical/high findings block merge. The main branch also has `enforce_admins` — even the repo owner can't bypass the security check.

---

## Discoverability

Once merged, skills become discoverable via:

1. **hermeshub.xyz browse UI** — listed under the Agents & Swarms category
2. **hermes CLI search**: `hermes skills search fusion` or `hermes skills search moa`
3. **hermeshub API**: `GET https://hermeshub.xyz/api/v1/skills/marketplace`
4. **Tag search**: any of the 10–12 tags on each skill

The `related_skills` frontmatter field ensures the three skills surface together when any one is found.

---

## After Merge

Installation command for any Hermes operator:

```bash
hermes skills install github:amanning3390/hermeshub/skills/fusion-orchestrator
hermes skills install github:amanning3390/hermeshub/skills/fusion-judge
hermes skills install github:amanning3390/hermeshub/skills/fusion-eval
```

Or via the tap (for users who already have this tap added):

```bash
hermes skills tap add shagghiesuperstar/hermes-fusion-tap
hermes skills install fusion-orchestrator
hermes skills install fusion-judge
```
