---
name: fusion-eval
description: Test harness for calibrating the fusion pipeline. Runs benchmark scenarios and reports drift, consistency, and quality metrics. Use to validate your fusion setup before production use.
version: 1.0.0
author: shagghiesuperstar
license: MIT
default_metadata:
  hermes:
    tags: Eval, Testing, Calibration, Benchmark, Quality, Fusion
    related_skills:
      - fusion-orchestrator
      - fusion-judge
    requires_toolsets:
      - delegation
      - file
    config:
      - key: fusion.eval_scenarios
        description: "Path to a JSONL file of eval scenarios (query + expected answer). Defaults to built-in benchmark."
        default: ""
        prompt: "Path to custom eval scenarios JSONL (leave blank for built-in)"
      - key: fusion.eval_output
        description: "Path to write eval results JSON"
        default: "fusion-eval-results.json"
        prompt: "Output file for eval results"
---

# Fusion Eval Harness

Validates your fusion pipeline configuration by running a battery of benchmark queries and measuring consistency, quality, and calibration. Run this before deploying fusion in production.

---

## When to Use

- First-time setup: verify fusion pipeline works end-to-end
- After changing models, providers, or config
- Periodically to detect quality drift
- When debugging unexpected fusion behavior

---

## Built-in Benchmark Scenarios

The harness includes these test classes:

| Class | Description | Pass Criteria |
|-------|-------------|---------------|
| **Factual consensus** | Query with a clear correct answer | All refs agree; synthesis correct |
| **Contested facts** | Query where common wrong answers exist | Judge identifies correct; synthesis accurate |
| **Multi-part query** | Query with 3+ distinct sub-questions | Synthesis addresses all parts |
| **Ambiguous query** | Intentionally underspecified query | Judge flags ambiguity; doesn't hallucinate scope |
| **Adversarial reference** | One injected reference with injection markers | Judge sanitizes; fusion not compromised |
| **Local-only mode** | Runs with MOA_DISABLED=true | Delegate fan-out functions correctly |

---

## Procedure

### Step 1 — Preflight Checks

1. Verify `fusion-orchestrator` skill is installed: `skill_view fusion-orchestrator`
2. Verify `fusion-judge` skill is installed: `skill_view fusion-judge`
3. Check `delegation` toolset is active
4. Check `file` toolset is active (for writing results)
5. Determine eval mode: MOA or delegate (mirror current fusion config)

Report preflight status. Abort if orchestrator or judge skills missing.

---

### Step 2 — Run Scenarios

For each scenario:
1. Load `fusion-orchestrator`
2. Submit scenario query
3. Capture full fusion output (synthesis + judge notes)
4. Compare synthesis against expected answer using these criteria:
   - Factual match (contains expected key facts?)
   - Completeness (addresses all required elements?)
   - No hallucinations (contains no known-false claims?)
5. Score: PASS / PARTIAL / FAIL with notes

---

### Step 3 — Report

Write results to `fusion.eval_output` path (default: `fusion-eval-results.json`):

```json
{
  "run_date": "<ISO timestamp>",
  "fusion_mode": "<moa|delegate>",
  "judge_mode": "<strict|balanced|fast>",
  "scenarios_run": N,
  "pass": N,
  "partial": N,
  "fail": N,
  "pass_rate": "X%",
  "results": [
    {
      "class": "factual_consensus",
      "query": "...",
      "expected": "...",
      "synthesis": "...",
      "verdict": "PASS",
      "notes": "..."
    }
  ]
}
```

Also print a summary table to the user:

```
✅ Fusion Eval Results
Mode: delegate | Judge: balanced
Scenarios: 6 | Pass: 5 | Partial: 1 | Fail: 0
Pass Rate: 83%

Partial:
  - multi-part query: synthesis missed sub-question 3

Recommendation: Increase fusion.reference_count to 4 to improve completeness.
```

---

## Pitfalls & Mitigations

| Failure Mode | Detection | Mitigation |
|-------------|-----------|------------|
| Eval runs but orchestrator not installed | preflight fails | Install fusion-orchestrator first |
| Custom JSONL malformed | JSON parse error | Validate JSONL; each line must be `{"query": "...", "expected": "...", "class": "..."}` |
| Eval exhausts context window | Token budget hit | Reduce scenarios per run; run in batches |
| Pass rate < 70% | Results show | Review config-examples.md; check model quality; increase reference_count |
| Adversarial reference test fails | Judge not sanitizing | Verify fusion-judge is up to date; check for skill version mismatch |

---

## Verification

- [ ] All 6 built-in scenario classes ran
- [ ] Results JSON written to output path
- [ ] Summary table printed
- [ ] Pass rate ≥ 70% before declaring pipeline production-ready
