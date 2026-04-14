# 06 — Registering Your Adapter with OMAIB

> **Audience**: Principal Investigators
> **Time to complete**: 15–30 minutes (plus 5 working days for platform confirmation)
> **Prerequisites**: Gate 1 passing locally + CI validation workflow green
> **Previous step**: [05 — Setting Up Continuous Validation with GitHub Action](05-CI-SETUP.md) complete
> **Next step**: [07 — Releasing Your Adapter and Managing Updates](07-RELEASE-AND-UPDATES.md)

---

## What This Document Covers

Registration is the process of telling OMAIB that your adapter repository exists and is ready for platform ingestion. It involves two actions:

1. Opening a PI registration issue at the `omaib-contracts` repository
2. Installing the OMAIB Platform Bot GitHub App on your repository

Both actions are required. The issue provides the metadata OMAIB needs; the GitHub App provides the communication channel for gate notifications.

---

## Before You Register — Checklist

| Requirement | How to verify |
|---|---|
| Gate 1 passing locally | `omaib-gate-status omaib-adapter/` shows `[PASS] Gate 1: READY` |
| CI workflow present and green | `.github/workflows/validate-dap.yml` exists; latest Actions run is ✓ |
| All four required files committed and pushed | `git status` shows clean; files visible at `github.com/<your-org>/<your-repo>/omaib-adapter/` |
| Repository is accessible | If private, OMAIB team needs read access (you'll set this up after filing the issue) |

---

## Step 1 — Open a PI Registration Issue

Go to [github.com/omaib/omaib-contracts/issues/new?template=pi-registration.yml](https://github.com/omaib/omaib-contracts/issues/new?template=pi-registration.yml) to open the PI Registration form directly.

### Issue title format

```text
[PI Registration] <Your Project Name>
```

**Examples:**

- `[PI Registration] SONAIR Acoustic Benchmark`
- `[PI Registration] NAS Self-Driving Dataset`
- `[PI Registration] Carbon Neutral Living Benchmark`

### Issue form fields

The registration form will ask for the following fields. Have them ready before you open the issue:

| Field | What to enter |
|---|---|
| **Adapter ID** | Your `adapter_id` from `benchmark_contract.yaml` (e.g. `dap-sonair`) |
| **Repository URL** | Full HTTPS URL of your GitHub repository |
| **Adapter path** | Path to `omaib-adapter/` within the repo (default: `omaib-adapter`) |
| **GitHub handle** | Your GitHub username (e.g. `@j-smith`) |
| **Benchmark ID** | Your `benchmark_id` from `benchmark_contract.yaml` (e.g. `omaib-sonair`) |
| **Tagged release** | The Git tag you have pushed that is Gate 1 ready (e.g. `v0.1.0`) |
| **Gate 1 readiness score** | Numeric score from `omaib-validate-adapter` (e.g. `93.8%`) |
| **Pre-submission checklist** | Confirm Gate 1 passes locally before submitting |

### What happens after you submit

| Day | Action |
|---|---|
| 0 | You submit the issue |
| 1 | Automated checks run — the OMAIB bot verifies the repo URL is valid, the adapter directory exists, and CI is green |
| 1–2 | OMAIB team reviews the issue for completeness |
| 3–5 | Registration confirmed — bot comments on the issue confirming your `benchmark_id` and providing instructions for Gate 2 |

If there are problems (e.g., CI not green, missing fields), the bot will comment with specific instructions. You do not need to open a new issue — fix the problem and reply to the same issue.

---

## Step 2 — Install the OMAIB Platform GitHub App

The **omaib-platform-bot** GitHub App posts gate status notifications, failure alerts, and update confirmations to your repository as issues.

### Installation steps

1. Go to [github.com/apps/omaib-platform-bot](https://github.com/apps/omaib-platform-bot)
2. Click **Install**
3. Choose the **organisation or account** that owns your adapter repository
4. Under **Repository access**, choose **Only select repositories** and pick your adapter repository
5. Review the permissions requested (see below) and click **Install**

### Permissions requested

| Permission | Scope | Why it's needed |
|---|---|---|
| **Issues: Read & Write** | Your adapter repo | Bot opens gate notification and failure issues |
| **Metadata: Read** | Your adapter repo | Bot reads repository metadata (name, visibility) |
| **Contents: Read** | Your adapter repo | Bot reads `omaib-adapter/` files at ingestion time |

The bot does not have push or admin access to your repository. It cannot write code, delete branches, or modify settings.

### Verifying installation

After installing, go to:
`https://github.com/settings/installations` (personal) or
`https://github.com/organisations/<your-org>/settings/installations`

You should see **omaib-platform-bot** listed with your repository.

---

## Step 3 — Private Repositories: Confirm App Read Access

If your repository is **private**, the **omaib-platform-bot** GitHub App you installed in Step 2 already has "Contents: Read" permission and can read your adapter files at ingestion time. No additional collaborator is needed in most cases.

After installing the App, reply to your registration issue with a brief confirmation:

```text
Private repo — omaib-platform-bot App installed on [your-repo-url]. App has Read access.
```

### Legacy fallback (omaib-bot collaborator)

If the OMAIB team explicitly requests a service-account collaborator — for example, because your organisation's GitHub App installation policy blocks App-based access — they will ask you to add `omaib-bot` as a read-only collaborator:

1. Go to your repository's **Settings → Collaborators and teams**
2. Click **Add people**
3. Search for `omaib-bot`
4. Set role to **Read** and confirm

Do not add `omaib-bot` proactively; wait for the team's instruction in the registration issue.

For **public repositories**, neither the App confirmation reply nor the collaborator step is required.

---

## What Happens After Registration is Confirmed

When OMAIB confirms your registration (via a comment on the issue), the following occurs:

1. Your adapter is added to the [OMAIB Adapter Registry](https://github.com/omaib/omaib-adapter-registry)
2. A placeholder leaderboard page is created at the OMAIB platform
3. Gate 2 evaluation is scheduled — a pilot model is run against your benchmark
4. The bot opens an issue in your repository titled: `[OMAIB Platform] Gate 2 Pilot Evaluation Scheduled`

You do not need to take any action for Gate 2 to start — the platform handles it automatically.

---

## After Gate 2 Completes

Once the pilot evaluation finishes, the bot opens another issue:

```text
[OMAIB Platform] Gate 2 Result — dap-<your-slug>

Gate 2 pilot evaluation completed.
  Primary metric (macro_f1): 0.712
  Evaluation ran on: 2024-11-20T09:14:22Z

Status: PASSED — benchmark is accepting community submissions.

Your leaderboard is now live at:
https://omaib.github.io/<your-slug>
```

If Gate 2 fails (evaluation errors, schema mismatches in submitted results), the bot lists the specific failures and you work through them by fixing adapter files or scorecard schema issues.

---

## Registration FAQ

**Q: Can I register before CI is set up?**
A: No. CI validation is a requirement for registration. Set it up first — it takes less than 10 minutes (see [05 — Setting Up Continuous Validation with GitHub Action](05-CI-SETUP.md)).

**Q: Can I register a private dataset?**
A: Yes. Set `data_access.model` to `gated`, `custodian-only`, or `hybrid` in `benchmark_contract.yaml`, and include `restricted` or `federated` in `evaluation_modes` as appropriate. The leaderboard page will display benchmark metadata but not the data itself.

**Q: Can I have multiple adapters in one repository?**
A: No. Each adapter repository contains exactly one `omaib-adapter/` folder. If you have two independent benchmarks, use two separate repositories.

**Q: How long does the review take?**
A: Typically within 5 working days. If you haven't heard within 7 days, reply to the issue to check.

**Q: I need to change my benchmark_id after registration. Is that possible?**
A: Contact the OMAIB team on the registration issue. This is a breaking change that requires coordination.

---

## Next

→ [07 — Releasing Your Adapter and Managing Updates](07-RELEASE-AND-UPDATES.md)
