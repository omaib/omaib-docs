# 08 — Troubleshooting

> **Audience**: Principal Investigators
> **Reference**: Use this document when something goes wrong at any step of the onboarding process

---

## How to Use This Document

Find your error in the section that matches where you are in the onboarding process. Each entry has:

- The exact error message or symptom
- The root cause
- The fix

If your problem isn't listed here, [open an issue on omaib-contracts](https://github.com/omaib/omaib-contracts/issues/new) titled `[PI Support] <brief description>`.

---

## Installation Problems

### `omaib-init-adapter: command not found`

**Symptom:**

```text
bash: omaib-init-adapter: command not found
```

**Cause:** `omaib-contracts` is not installed, or your virtual environment is not activated.

**Fix:**

```bash
# 1. Activate your virtual environment first
source .venv/bin/activate        # macOS / Linux
.\.venv\Scripts\Activate.ps1     # Windows PowerShell

# 2. Install the package
pip install git+https://github.com/omaib/omaib-contracts.git

# 3. Verify
omaib-init-adapter --help
```

If step 1 shows `(base)` in your prompt and you're using conda, you may need to use `conda create` instead:

```bash
conda create -n omaib python=3.12 -y
conda activate omaib
pip install git+https://github.com/omaib/omaib-contracts.git
```

---

### `pip install` fails with `git clone` error

**Symptom:**

```text
ERROR: Error when trying to get requirement for URL 'git+https://github.com/omaib/omaib-contracts.git'
```

**Cause:** Git is not installed, or is not in PATH.

**Fix:**

```bash
# Check git is available
git --version

# If not installed:
# macOS:    brew install git
# Ubuntu:   sudo apt install git
# Windows:  https://git-scm.com/download/win
```

---

### Python version too old

**Symptom:**

```text
ERROR: Package 'omaib-contracts' requires a different Python: 3.9.x not in '>=3.11'
```

**Fix:** Upgrade Python to 3.11+. See the [Prerequisites guide](01-PREREQUISITES.md) for platform-specific instructions.

---

## Scaffold Problems

### `omaib-init-adapter` complains the directory already exists

**Symptom:**

```text
Error: omaib-adapter/ already exists. Use --force to overwrite.
```

**Fix:** If you want to re-scaffold from scratch:

```bash
omaib-init-adapter omaib-adapter/ --slug <your-slug> --force
```

**Warning:** `--force` overwrites existing files. If you've already filled in values, back them up first:

```bash
cp -r omaib-adapter/ omaib-adapter-backup/
omaib-init-adapter omaib-adapter/ --slug <your-slug> --force
```

---

## Validation Errors

### Gate 1 BLOCKED — placeholder text remaining

**Symptom:**

```text
Gate 1 score: 23/100
- benchmark_contract.yaml Line 8: 'title' contains placeholder text ('<placeholder>')
- governance_policy.yaml: 'custodian_boundary.controller' contains placeholder text
```

**Fix:** Find all remaining placeholders:

```bash
grep -rn "<" omaib-adapter/
```

Every line returned contains a placeholder. Replace each `<...>` with real content, then re-run validation.

---

### `adapter_id` or `benchmark_id` mismatch

**Symptom:**

```text
- Cross-file consistency: adapter_id 'dap-sonair-v2' in data_profile.json
  does not match 'dap-sonair' in benchmark_contract.yaml
```

**Fix:** List the `adapter_id` value in all files and make them identical:

```bash
grep "adapter_id\|benchmark_id" omaib-adapter/*.yaml omaib-adapter/*.json
```

Edit the mismatched file to match the canonical value in `benchmark_contract.yaml`.

---

### `access_tier` invalid or inconsistent with benchmark access model

**Symptom:**

```text
- governance_policy.yaml: 'access_tier' must be one of ['open','registered','gated','custodian-only','hybrid','federated']
```

**Fix:** Set a valid `access_tier` in `governance_policy.yaml`, and keep it consistent with your intended access model in `benchmark_contract.yaml` (`data_access.model`).

```bash
# In governance_policy.yaml:
access_tier: gated
```

---

### Primary metric not in scorecard_schema.json

**Symptom:**

```text
- metrics.primary 'rmse' not found in scorecard_schema.json properties
```

**Fix:** Either add `rmse` as a property in `scorecard_schema.json`:

```json
"properties": {
  "rmse": {
    "type": "number",
    "description": "Root mean square error",
    "minimum": 0.0
  }
}
```

Or change `metrics.primary` in `benchmark_contract.yaml` to match an existing `properties` key in `scorecard_schema.json`.

---

### Version mismatch across files

**Symptom:**

```text
- Cross-file consistency: version '0.2.0' in data_profile.json
  does not match '0.1.0' in benchmark_contract.yaml
```

**Fix:** All four files must use the same version string. Quick check:

```bash
grep '"version"\|^version:' omaib-adapter/*.yaml omaib-adapter/*.json
```

Update all non-matching files to the correct version.

---

### `splits.ratios` don't sum to 1.0

**Symptom:**

```text
- data_profile.json: splits.ratios values sum to 0.9500, expected 1.0000
```

**Fix:**

```json
"ratios": { "train": 0.70, "val": 0.15, "test": 0.15 }
```

Ensure the three floats sum to exactly `1.0`. Note that floating point precision matters â€” `0.7 + 0.15 + 0.15 = 1.0` but `0.33 + 0.33 + 0.34 = 1.0` as well.

---

### `required field is null`

**Symptom:**

```text
- data_profile.json: 'size_estimate.samples' is null (required field)
```

**Fix:** Replace `null` with the actual value. An approximate integer is acceptable for `samples`.

---

## CI / GitHub Actions Problems

### Workflow not triggering

**Symptom:** You pushed to `main` but no workflow run appeared in the Actions tab.

**Possible causes and fixes:**

| Cause | Fix |
|---|---|
| Workflow file is not in `.github/workflows/validate-dap.yml` exactly | Check the path â€” case-sensitive on Linux runners |
| Push was to a branch not listed in `branches:` | Add your branch to the `branches:` list in the workflow |
| No changes to `omaib-adapter/**` files | The `paths:` filter is active â€” push a change to an adapter file |
| Actions are disabled for the repository | Go to Settings â†’ Actions â†’ Allow all actions |

---

### Workflow fails: `omaib-validate-adapter: not found`

**Symptom:**

```text
Run omaib-validate-adapter omaib-adapter/ --json > validation_report.json
/home/runner/work/_temp/...: line 1: omaib-validate-adapter: command not found
```

**Cause:** The `pip install` step failed silently or was cached from a bad state.

**Fix:** Add `--no-cache-dir` and check the install step output:

```yaml
- name: Install omaib-contracts
  run: pip install --no-cache-dir git+https://github.com/omaib/omaib-contracts.git
```

---

### Workflow fails: `git clone ... network error`

**Symptom:**

```text
fatal: unable to connect to github.com
```

**Cause:** GitHub Actions runner had a transient network issue.

**Fix:** Click **Re-run failed jobs** in the Actions UI. This is a transient issue and almost always resolves on re-run.

---

### Workflow passes locally but fails in CI

**Symptom:** `omaib-validate-adapter` passes on your machine but the CI job fails.

**Most common cause:** Schema or template update deployed to `omaib-contracts` after your local install.

**Fix:**

```bash
pip install --upgrade git+https://github.com/omaib/omaib-contracts.git
omaib-validate-adapter omaib-adapter/
```

Fix any new errors, commit, and push.

---

## Registration Problems

### Bot doesn't respond to the registration issue

**Symptom:** You opened the issue but no automated checks appeared after 24 hours.

**Check:**

- Issue title must start with `[PI Registration]` exactly (case-sensitive)
- Confirm you opened the issue at [github.com/omaib/omaib-contracts/issues](https://github.com/omaib/omaib-contracts/issues), not your own repo

If both are correct and there is still no response after 48 hours, reply to the issue with a comment `@omaib-team ping`.

---

### Bot reports "CI workflow not found or failing"

**Symptom:**

```text
[omaib-platform-bot] ❌ Registration check failed:
  - validate-dap.yml not found in .github/workflows/
```

**Fix:** Ensure the workflow is committed and pushed:

```bash
git add .github/workflows/validate-dap.yml
git commit -m "ci: add OMAIB DAP validation workflow"
git push origin main
```

Then check the Actions tab to confirm the workflow has at least one successful run.

---

### Bot reports "Repository not accessible"

**Symptom:**

```text
[omaib-platform-bot] ❌ Registration check failed:
  - Repository not accessible at https://github.com/<your-org>/<your-repo>
```

**Fix for private repos:** First confirm the **omaib-platform-bot** GitHub App is installed on your repo with **Contents: Read** permission (see [06 — Registering Your Adapter with OMAIB](06-REGISTER.md)). Reply to your registration issue confirming App access.

If the OMAIB team explicitly asks for legacy fallback access (for org policy reasons), then add `omaib-bot` as a **Read** collaborator and reply to the issue once done.

**Fix for public repos:** Check that the repository URL in the issue body is correct and the repo is not archived.

---

## Gate System Problems

### Gate 2 takes longer than 5 working days

**Fix:** Reply to your registration issue with:

```text
@omaib-team Could you check the status of the Gate 2 pilot evaluation?
It's been more than 5 working days since registration was confirmed.
```

---

### Gate 2 fails with "evaluation error"

**Symptom:**

```text
[OMAIB Platform] Gate 2 Result â€” dap-<your-slug>
Status: FAILED
  - Evaluation error: scorecard submission failed schema validation
  - Submitted field 'macro_F1' does not match scorecard_schema metric name 'macro_f1'
```

**Fix:** Field names in `scorecard_schema.json` are case-sensitive. `macro_F1` and `macro_f1` are different. Check the exact field names and update `scorecard_schema.json` if needed, then bump the version and push.

---

## Platform Ingestion Problems

### Ingestion stuck after tagging a release

**Fix:** The ingestion poll runs every 6 hours automatically. If you need an immediate refresh, open an issue on `omaib-contracts` requesting an expedited run, or the OMAIB team can manually dispatch the ingestion workflow via **Actions â†’ Adapter Ingestion Poll â†’ Run workflow** in the `omaibench` repository. There is no bot command for on-demand refresh.

---

### Ingestion failed with schema version error

**Symptom:**

```text
[OMAIB Platform] Ingestion error â€” v0.2.0
  - benchmark_contract.yaml: unknown schema version '0.1.0' (current schema: benchmark-contract/v0.2)
```

**Cause:** A major schema version update was published. Your adapter files need to be migrated.

**Fix:** Upgrade `omaib-contracts` and re-run validation to see the current schema requirements:

```bash
pip install --upgrade git+https://github.com/omaib/omaib-contracts.git
omaib-validate-adapter omaib-adapter/   # re-run to see any updated field requirements
```

If validation reports new required fields, add them to your adapter files, bump the version in all four files, commit, and push a new tag.

---

## Getting More Help

If none of the above resolves your issue:

| Channel | Use for |
|---|---|
| [omaib-contracts Issues](https://github.com/omaib/omaib-contracts/issues) | Registration issues, platform bugs, schema questions |
| [omaib-docs Discussions](https://github.com/omaib/omaib-docs/discussions) | General onboarding questions, peer support from other PIs |
| Email: <omaib-ukomain-group@sheffield.ac.uk> | Sensitive data handling, institutional agreements, federated evaluation setup |

When filing a support issue, include:

- Your `adapter_id`
- The version of `omaib-contracts` installed (`pip show omaib-contracts`)
- The full output of `omaib-validate-adapter omaib-adapter/`
- A link to the GitHub Actions run if applicable

---

 [07 — Releasing Your Adapter and Managing Updates](07-RELEASE-AND-UPDATES.md) | [Back to index](index.md)
