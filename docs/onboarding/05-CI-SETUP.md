# 05 — Setting Up Continuous Validation with GitHub Actions

> **Audience**: Principal Investigators
> **Time to complete**: 10 minutes
> **Prerequisites**: Gate 1 passing locally (see [04 — Local Validation and Gate Scores](04-VALIDATION.md))
> **Next step**: [06 — Registering Your Adapter with OMAIB](06-REGISTER.md)

---

## What This Document Covers

Once your adapter passes Gate 1 locally, you set up a GitHub Actions workflow so that OMAIB validation runs **automatically** on every push and pull request to your repository. This protects you from accidentally committing invalid adapter files and gives OMAIB a continuous signals channel for your benchmark's health.

---

## Why CI Validation Matters

| Benefit | Detail |
|---|---|
| Prevents regressions | A schema break in your adapter files fails the workflow before it merges |
| Visible to OMAIB | The platform monitors your workflow badge; a persistent failure pauses leaderboard ingestion |
| Pre-registration check | OMAIB reviewers will check that this workflow is present and green before confirming registration |
| Audit trail | Every push produces a downloadable `omaib-validation-report` artifact retained for 30 days |

---

## Step 1 — Create the Workflow Directory

In the root of your adapter repository:

```bash
mkdir -p .github/workflows
```

If `.github/workflows/` already exists (common if you have other CI), skip this step.

---

## Step 2 — Create the Workflow File

Create `.github/workflows/validate-dap.yml` with the following content:

```yaml
# OMAIB DAP Validation — GitHub Actions workflow for PI adapter repos
#
# Validates your omaib-adapter/ directory on every push, PR, and release.
# No secrets required — omaib-contracts is a public pip package.

name: OMAIB DAP Validation

on:
  push:
    branches: ["main", "develop"]
    paths:
      - "omaib-adapter/**"
      - ".github/workflows/validate-dap.yml"
  pull_request:
    paths:
      - "omaib-adapter/**"
  release:
    types: [published]

jobs:
  validate-adapter:
    name: Validate DAP (Gate 1)
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install omaib-contracts
        run: pip install git+https://github.com/omaib/omaib-contracts.git

      - name: Validate adapter (Gate 1)
        id: validate
        run: |
          omaib-validate-adapter omaib-adapter/ --json > validation_report.json
          EXIT=$?
          cat validation_report.json
          echo "exit_code=$EXIT" >> "$GITHUB_OUTPUT"

      - name: Gate status summary
        run: omaib-gate-status omaib-adapter/

      - name: Upload validation report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: omaib-validation-report
          path: validation_report.json
          retention-days: 30

      - name: Fail if Gate 1 not reached
        if: steps.validate.outputs.exit_code != '0'
        run: |
          echo "::error::Adapter did not pass Gate 1 validation."
          echo "::error::Run 'omaib-validate-adapter omaib-adapter/' locally to see details."
          exit 1

      - name: Gate 1 passed
        if: steps.validate.outputs.exit_code == '0'
        run: echo "::notice::Gate 1 PASSED — adapter is ready for OMAIB platform ingestion."
```

---

## Step 3 — Understand What the Workflow Does

The workflow runs in six steps:

| Step | What it does |
|---|---|
| **Checkout** | Clones your repository into the runner |
| **Set up Python** | Installs Python 3.12 on the runner |
| **Install omaib-contracts** | Fetches the latest CLI tools from GitHub |
| **Validate adapter (Gate 1)** | Runs `omaib-validate-adapter` and saves the JSON report; captures the exit code |
| **Gate status summary** | Prints a human-readable gate summary to the job log |
| **Upload validation report** | Uploads `validation_report.json` as a downloadable artifact (30-day retention) |

The workflow **passes** if the exit code from `omaib-validate-adapter` is `0` (Gate 1 reached). It **fails** with a clear error annotation if Gate 1 is not reached, and the pipeline is blocked from merging.

---

## Step 4 — Customise the Trigger Branches (Optional)

The template triggers on pushes to `main` and `develop`. If your repository uses a different branch naming convention, update the `branches` list:

```yaml
on:
  push:
    branches: ["main", "master", "release/*"]   ← edit this list
```

If you want validation to run on **every push** (not just adapter file changes), remove the `paths` filter:

```yaml
on:
  push:
    branches: ["main", "develop"]
    # paths filter removed — runs on all pushes
```

---

## Step 5 — Commit and Push the Workflow

```bash
git add .github/workflows/validate-dap.yml
git commit -m "ci: add OMAIB DAP validation workflow"
git push origin main
```

After pushing, navigate to your repository on GitHub, click the **Actions** tab, and watch the `OMAIB DAP Validation` workflow run. A green tick means Gate 1 passed in CI.

---

## Step 6 — Verify the Artifact

After a successful run:

1. Go to the workflow run on GitHub Actions
2. Click **Summary** tab
3. Scroll to **Artifacts** at the bottom
4. Click `omaib-validation-report` to download `validation_report.json`

The JSON report contains:

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

## Adding the Status Badge to Your README

Copy this Markdown snippet into your `README.md`, replacing `<your-org>` and `<your-repo>`:

```markdown
[![OMAIB DAP Validation](https://github.com/<your-org>/<your-repo>/actions/workflows/validate-dap.yml/badge.svg)](https://github.com/<your-org>/<your-repo>/actions/workflows/validate-dap.yml)
```

This badge shows the live validation status and is visible to anyone viewing your repository.

---

## When the Workflow Fails

If the workflow fails after you've made changes to your adapter files:

1. Click the failed run to see which step failed
2. Expand the **Validate adapter (Gate 1)** step to see the error output
3. The errors printed there are identical to what you'd see running `omaib-validate-adapter omaib-adapter/` locally
4. Fix the errors, commit, and push — the workflow re-runs automatically

If you see an error in the **Install omaib-contracts** step (e.g., network timeout), re-run the job using the **Re-run failed jobs** button. If it fails consistently, check the [omaib-contracts repository](https://github.com/omaib/omaib-contracts) for known issues.

---

## What OMAIB Sees

When your registration is reviewed, OMAIB checks for the presence of this workflow in `.github/workflows/validate-dap.yml` and that the most recent run is green. This is a checklist item in the registration review process — it's not optional.

---

## Next

→ [06 — Registering Your Adapter with OMAIB](06-REGISTER.md)
