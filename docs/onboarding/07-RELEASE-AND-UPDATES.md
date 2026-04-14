# 07 — Releasing Your Adapter and Managing Updates

> **Audience**: Principal Investigators  
> **Time to complete**: 15–20 minutes (first release); 5–10 minutes (subsequent updates)  
> **Prerequisites**: Registration confirmed (see [06 — Registering Your Adapter with OMAIB](06-REGISTER.md))  
> **Next step**: [08 — Troubleshooting](08-TROUBLESHOOTING.md)

---

## What This Document Covers

After registration is confirmed, you tag an initial release to signal to the OMAIB platform that your benchmark is ready for live ingestion. This document covers:

1. How to tag and push your first release
2. How the platform polls for changes
3. When and how to version-bump your adapter
4. What changes require a new registration issue vs. an automatic update
5. The full update lifecycle

---

## Semantic Versioning in OMAIB

All four adapter files share a single `version` field that follows [Semantic Versioning](https://semver.org/): `MAJOR.MINOR.PATCH`.

| Version type | When to use | Example |
|---|---|---|
| **Patch** (`0.1.x`) | Corrections that don't change schema or evaluation behaviour | Fixing a typo in `description`, correcting `storage_gb`, adding `citation.doi` |
| **Minor** (`0.x.0`) | New capabilities that are backwards-compatible | Adding a new secondary metric, adding an optional field, adding `trust_toolkit.yaml` |
| **Major** (`x.0.0`) | Breaking changes | Changing `benchmark_id`, changing the `primary` metric, restructuring the scorecard schema |

> **Rule of thumb**: If evaluators who have already submitted scorecards would need to re-submit, it's a breaking (major) change.

When you version-bump, update the `version` field in **all four required files** to the same new value. A mismatch in version strings across files causes a Gate 1 validation error.

---

## Step 1 — Tag Your First Release

After registration is confirmed and CI is green, create the initial release tag:

```bash
# Make sure you're on main and everything is pushed
git checkout main
git pull origin main

# Create an annotated tag for the initial release
git tag -a v0.1.0 -m "Initial OMAIB DAP release — Gate 1 validated"

# Push the tag to GitHub
git push origin v0.1.0
```

The CI workflow will re-run on this release event (`release: [published]` in the workflow trigger). Check the Actions tab to confirm it passes.

### Creating a formal GitHub Release (recommended)

Tags alone work, but a formal GitHub Release provides a changelog and is more visible:

```bash
gh release create v0.1.0 \
  --title "OMAIB DAP v0.1.0 — Initial Release" \
  --notes "Initial validated release of the $(grep 'adapter_id:' omaib-adapter/benchmark_contract.yaml | head -1 | awk '{print $2}') adapter."
```

Or via the GitHub web interface: go to your repository → **Releases** → **Draft a new release** → set tag `v0.1.0` → publish.

---

## Step 2 — Platform Ingestion

After the tag is pushed, the OMAIB platform picks up your release within its next polling cycle.

### Polling schedule

The ingestion pipeline runs on a **fixed 6-hour cron schedule** for all registered adapters.

| Trigger | Frequency |
|---|---|
| Scheduled (all registered adapters) | Every 6 hours |
| Manual on-demand | Via **Actions → Adapter Ingestion Poll → Run workflow** in the `omaibench` repository (`workflow_dispatch`) |

### What happens at ingestion

1. Platform clones your repository at the tagged commit
2. Runs `omaib-validate-adapter omaib-adapter/` server-side
3. If Gate 1 passes: ingests the adapter metadata into the registry
4. Updates your leaderboard page with the new version
5. Platform updates the adapter registry and leaderboard page

If server-side validation fails on a version you've already validated locally, it usually means:
- A template or schema update was deployed between your local check and ingestion
- Run `pip install --upgrade git+https://github.com/omaib/omaib-contracts.git` and re-validate

---

## Step 3 — Making Updates

### Patch update (e.g., fix a typo in description)

```bash
# 1. Make the change in the file
# Example: fix description in benchmark_contract.yaml

# 2. Bump the version in ALL four files from 0.1.0 → 0.1.1
#    (edit benchmark_contract.yaml, data_profile.json,
#     governance_policy.yaml, scorecard_schema.json)

# 3. Validate locally
omaib-validate-adapter omaib-adapter/

# 4. Commit
git add omaib-adapter/
git commit -m "fix: correct benchmark description and bump to v0.1.1"
git push origin main

# 5. Tag the release
git tag -a v0.1.1 -m "Patch: corrected benchmark description"
git push origin v0.1.1
```

### Minor update (e.g., adding a new secondary metric)

```bash
# 1. Add the new metric to benchmark_contract.yaml (metrics.secondary)
#    and scorecard_schema.json (add a key under properties)

# 2. Bump version from 0.1.x → 0.2.0 in ALL four files

# 3. Validate locally
omaib-validate-adapter omaib-adapter/

# 4. Commit and tag
git add omaib-adapter/
git commit -m "feat: add roc_auc as secondary metric — bump to v0.2.0"
git push origin main
git tag -a v0.2.0 -m "Minor: added roc_auc secondary metric"
git push origin v0.2.0
```

### Major update (breaking change)

Major changes that affect the leaderboard (e.g., changing the primary metric) require coordination with OMAIB to avoid disrupting existing submissions. Before making a major change:

1. Open a new issue on `omaib-contracts` titled: `[PI Update] <Your-Adapter-ID> — Breaking Change`
2. Describe the change and why it's necessary
3. Wait for OMAIB team confirmation before pushing
4. Once confirmed, make the change, bump to `1.0.0`, and tag as usual

---

## Update Lifecycle Reference

```
Local edit
    │
    ▼
omaib-validate-adapter omaib-adapter/    ← Gate 1 local check
    │ passes
    ▼
git add + commit + push
    │
    ▼
GitHub Actions runs                       ← Gate 1 CI check
    │ green
    ▼
git tag -a vX.Y.Z + push tag
    │
    ▼
Platform polls (≤ 6h)                     ← ingestion
    │
    ▼
Leaderboard page updated
```

---

## Version Bump Checklist

When bumping the version, update **all four files**:

- [ ] `omaib-adapter/benchmark_contract.yaml` — `version:` field
- [ ] `omaib-adapter/data_profile.json` — `"version":` field
- [ ] `omaib-adapter/governance_policy.yaml` — `version:` field
- [ ] `omaib-adapter/scorecard_schema.json` — `"version":` field
- [ ] Validate: `omaib-validate-adapter omaib-adapter/` passes
- [ ] All changes committed and pushed to `main`
- [ ] Tag created with `git tag -a vX.Y.Z` and pushed with `git push origin vX.Y.Z`

---

## What Requires a New Registration Issue?

| Change | Process |
|---|---|
| Patch / minor changes to adapter files | Tag and push — automatic pickup |
| Adding `trust_toolkit.yaml` or `assurance_crosswalk.yaml` | Tag and push — triggers gate review |
| Changing `benchmark_id` | Open `[PI Update]` issue — breaking change |
| Changing `primary` metric | Open `[PI Update]` issue — breaking change |
| Transferring ownership (new PI) | Open `[PI Transfer]` issue |
| Taking the benchmark offline | Open `[PI Deprecation]` issue |

---

## Deprecating a Benchmark

If you need to retire a benchmark:

1. Open an issue: `[PI Deprecation] <adapter-id>` on `omaib-contracts`
2. Bump the version (patch) in **all four adapter files**, validate locally, tag, and push — this signals the final committed state of the adapter before archival
3. OMAIB will mark the leaderboard as archived but preserve historical submissions

> **Do not add a `status` field to `benchmark_contract.yaml`.** The schema uses `additionalProperties: false`; any field not defined in the schema will cause Gate 1 validation to fail. Deprecation state is managed server-side by the platform once your issue is processed.

---

## Next 
→ [08 — Troubleshooting](08-TROUBLESHOOTING.md)
