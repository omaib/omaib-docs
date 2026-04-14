# 04 — Local Validation and Gate Scores

> **Audience**: Principal Investigators **Time to complete**: 10–15 minutes **Prerequisites**: All four required files
> in `omaib-adapter/` with no `<placeholder>` values **Previous step**:
> [03 — The Four Required Files: A Field-by-Field Reference](03-FOUR-FILES.md) **Next step**:
> [05 — Setting Up Continuous Validation with GitHub Action](05-CI-SETUP.md)

---

## What This Document Covers

Before registering with OMAIB, you validate your adapter locally using the CLI tools provided by `omaib-contracts`. This
document explains:

1. How to run validation and read the output
2. What the gate score means and how it's calculated
3. How to interpret each gate level
4. How to fix the most common validation failures
5. How to check your gate status as you iterate

---

## The Two Validation Commands

### omaib-validate-adapter

Validates all files in `omaib-adapter/` against OMAIB schemas and reports every error and warning:

```bash
omaib-validate-adapter omaib-adapter/
```

**Typical successful output:**

```text
[OK] benchmark_contract.yaml
[OK] data_profile.json
[OK] governance_policy.yaml
[OK] scorecard_schema.json
Optional files:
  [--] trust_toolkit.yaml        not present (optional)
Gate Readiness Score: 100.0%
Gate 1 Ready: YES
```

**Typical output with errors:**

```text
[FAIL] benchmark_contract.yaml
         - 'title' contains placeholder text ('<placeholder>')
         - 'metrics.primary' value 'rmse' not found in scorecard_schema.json properties
[OK]   data_profile.json
[FAIL] governance_policy.yaml
         - 'access_tier' (open) does not match 'data_access.model' in benchmark_contract.yaml (restricted)
[OK]   scorecard_schema.json
Optional files:
  [--] trust_toolkit.yaml        not present (optional)
Gate Readiness Score: 41.0%
Gate 1 Ready: NO
```

### omaib-gate-status

A shorter command that shows your current gate readiness and what's needed to progress further:

```bash
omaib-gate-status omaib-adapter/
```

**Example output (Gate Readiness Score 100%):**

```text
Gate Status for: omaib-adapter/

  [PASS] Gate 1: READY  (All required DAP files valid; governance boundary and export policy recorded.)
  [PASS] Gate 2: READY  (Evaluator runnable end-to-end; scorecard produced.)
  [----] Gate 3: NOT READY  (Cross-project scorecard alignment + external review evidence.) Requires platform-level evidence beyond adapter files
  [----] Gate 4: NOT READY  (v1.0 packaging: docs, schema versions, reproducibility manifests, adoption assets.) Requires platform-level evidence beyond adapter files
```

Gate 2 passes automatically when Gate 1 is ready **and** the Gate Readiness Score is ≥ 80%. Gates 3 and 4 always require
platform-level evidence that cannot be inferred from local adapter files.

### Saving The Report As JSON

For CI integration, scripting, or archiving:

```bash
omaib-validate-adapter omaib-adapter/ --json > validation_report.json
```

The JSON report includes the full error list, gate readiness score, and a machine-readable `gate_1_ready` boolean. Key
fields in the output object:

```json
{
  "adapter_path": "omaib-adapter/",
  "file_reports": [
    {
      "filename": "benchmark_contract.yaml",
      "exists": true,
      "valid_schema": true,
      "errors": [],
      "tbd_fields": [],
      "total_required_fields": 12,
      "non_tbd_required_fields": 12
    }
  ],
  "gate_readiness_score": 100.0,
  "gate_1_ready": true,
  "missing_required_files": [],
  "optional_files_present": [],
  "optional_files_missing": []
}
```

---

## Understanding The Gate Readiness Score

The Gate Readiness Score (`gate_readiness_score`) is a percentage (0–100) reflecting how complete and valid your adapter
files are. `Gate 1 Ready: YES` is shown when all required files are present, all pass schema validation, and have no
`TBD` values in required fields.

The score is computed by the validator at runtime based on file completeness. The exact formula may evolve — always
treat the printed `Gate 1 Ready: YES / NO` as the definitive result.

### What The Output Markers Mean

| Marker   | Meaning                                                                                                                                                                                                    |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `[OK]`   | File is present and schema-valid. TBD fields, if any, are printed under the file entry and reduce the Gate Readiness Score — a file can show `[OK]` while Gate 1 still fails if the score drops below 60%. |
| `[FAIL]` | File has schema errors or missing required fields — blocks Gate 1                                                                                                                                          |
| `[MISS]` | Required file is missing entirely — blocks Gate 1                                                                                                                                                          |
| `[--]`   | Optional file is absent — does not block Gate 1                                                                                                                                                            |

> **Gate 1 Ready: YES means you can register.** A score of 100% is not required — optional files (e.g.
> `trust_toolkit.yaml`) are not gating.

---

## The Four Gates

| Gate       | Name                   | What it checks                                                                                   | Who runs it                                   |
| ---------- | ---------------------- | ------------------------------------------------------------------------------------------------ | --------------------------------------------- |
| **Gate 1** | Schema validation      | File structure, field types, cross-file consistency                                              | You (local CLI) and OMAIB CI                  |
| **Gate 2** | Pilot evaluation       | A pilot model is evaluated against your benchmark — checks that the evaluation runs end-to-end   | OMAIB platform (automated, post-registration) |
| **Gate 3** | Convergence & trust    | Leaderboard results are converging; `trust_toolkit.yaml` filled in; stakeholder rubrics complete | OMAIB platform + PI                           |
| **Gate 4** | Assurance & regulatory | `assurance_crosswalk.yaml` mapped to compliance standards; external audit if required            | OMAIB platform + PI                           |

Local validation only covers Gate 1. Gates 2–4 are run by the OMAIB platform after registration.

---

## Fixing Common Validation Errors

### Error: "contains placeholder text"

```text
- 'title' contains placeholder text ('<placeholder>')
```

**Fix**: Open the file and replace every `<...>` string with real content. Search for all remaining placeholders with:

```bash
grep -r "<" omaib-adapter/ --include="*.yaml" --include="*.json"
```

Any match means a placeholder remains. Replace it with your actual value and re-run validation.

---

### Error: "required field is null"

```text
- 'size_estimate.samples' is null (required field)
```

**Fix**: Replace `null` with a real value. For `samples`, an approximate integer is acceptable:

```json
"samples": 8000
```

---

### Error: "metrics.primary not found in scorecard_schema.json"

```text
- 'metrics.primary' value 'rmse' not found in scorecard_schema.json properties
```

**Fix**: Add the missing metric as a property in `scorecard_schema.json`:

```json
"rmse": {
  "type": "number",
  "minimum": 0,
  "description": "Root mean square error"
}
```

Alternatively, change `metrics.primary` in `benchmark_contract.yaml` to match an existing key under `properties` in
`scorecard_schema.json`.

---

### Error: "access_tier mismatch"

```text
- 'access_tier' (open) does not match 'data_access.model' in benchmark_contract.yaml (restricted)
```

**Fix**: Make both files use the same value. Decide which is correct and update the other:

```yaml
# benchmark_contract.yaml
data_access:
  model: gated

# governance_policy.yaml
access_tier: gated # ← must be semantically consistent with data_access.model above
```

---

### Error: "adapter_id mismatch across files"

```text
- adapter_id 'dap-sonair-v2' in data_profile.json does not match 'dap-sonair' in benchmark_contract.yaml
```

**Fix**: Make `adapter_id` identical in all files. A quick check:

```bash
grep "adapter_id" omaib-adapter/*.yaml omaib-adapter/*.json
```

All lines should show the same value.

---

### Error: "ratios do not sum to 1.0"

```text
- splits.ratios values sum to 0.95, expected 1.0
```

**Fix**: Make the three ratios add up to exactly 1.0:

```json
"ratios": { "train": 0.70, "val": 0.15, "test": 0.15 }
```

---

### Warning (not a blocker): "recommended field missing"

```text
⚠  citation.doi not set — adapter discoverability will be reduced
⚠  provenance_fields not set
```

Warnings don't prevent Gate 1 from passing. Fill them in when your dataset has a DOI and you have documented provenance
fields.

---

## Iterative Validation Workflow

The recommended local iteration loop:

```text
1. Edit a file
2. Run:  omaib-validate-adapter omaib-adapter/
3. Fix any errors
4. Repeat until:  Gate 1 Ready: YES
5. Run:  omaib-gate-status omaib-adapter/   to confirm [PASS] Gate 1: READY
6. Commit and push
7. Proceed to register
```

Expected number of iterations: 2–5 for a first-time PI. Most errors are typos, placeholder text, or mismatched IDs.

---

## After Gate 1 Passes

Once you see `Gate 1 Ready: YES`, you have two parallel next steps:

1. **Set up CI** — copy the validation workflow to `.github/workflows/validate-dap.yml` so validation runs automatically
   on every push (see [05-CI-SETUP.md](05-CI-SETUP.md))
2. **Register** — open a PI registration issue with OMAIB (see [06-REGISTER.md](06-REGISTER.md))

You can do both in any order, but setting up CI first means you get automatic validation before registration is
confirmed.

---

## Next

→ [05 — Setting Up Continuous Validation with GitHub Action](05-CI-SETUP.md)
