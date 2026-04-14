# PI Onboarding — 00: What is OMAIB and How Does It Work for You?

**Audience**: Domain Project PIs and their teams
**Time to read**: 5 minutes
**What you will understand**: The OMAIB model, your role in it, and what you actually need to do

---

## What is OMAIB?

OMAIB (Open Multimodal AI Benchmarking) is an open benchmarking platform for AI systems applied to real-world scientific and engineering domains. It provides:

- A **standardised evaluation framework** (CDRF — Contextualised Domain Readiness Framework) to assess AI models against your domain's specific requirements
- A **public leaderboard** where benchmark results are visible to the AI research community
- An **automated ingestion pipeline** that pulls your benchmark definition from your own repository — you keep full ownership of your data and code
- A **governance layer** that enforces your data access rules — the platform does not publish data you mark as restricted

---

## The Core Model

```text
You (PI team)                  OMAIB Platform
─────────────                  ──────────────
Create adapter files     →     Platform reads them automatically
Tag a release            →     Platform ingests, validates, registers
Control data governance  →     Platform enforces your rules
Publish updates          →     Platform picks them up within 6 hours
```

Your adapter lives in **your own GitHub repository** (at your institution or personal GitHub). OMAIB never pushes to your repo — it only reads tagged releases.

---

## What You Need to Contribute

Your complete contribution is a **Domain Adapter Pack (DAP)** — a folder of four YAML/JSON files inside your repo:

| File | Purpose |
|---|---|
| `benchmark_contract.yaml` | Defines your benchmark: tasks, evaluation modes, metrics, governance |
| `data_profile.json` | Describes your dataset: size, modalities, access tier |
| `governance_policy.yaml` | Records your data governance boundary and legal basis |
| `scorecard_schema.json` | Defines what metrics are produced and which are publicly visible |

Two additional files are optional but strongly recommended for later evaluation gates:

| File | Purpose |
|---|---|
| `trust_toolkit.yaml` | Stakeholder engagement and co-design evidence (required for Gate 3) |
| `assurance_crosswalk.yaml` | Maps your metrics to regulatory frameworks (required for Gate 4) |

You do not write code. You do not access OMAIB infrastructure. You fill in structured templates.

---

## The Gate System

OMAIB evaluates adapters through a staged gate system:

| Gate | Name | What it checks |
|---|---|---|
| Gate 1 | Schema validation | All required files present, no TBD fields, schema valid |
| Gate 2 | Pilot evaluation | Evaluator has run your benchmark and confirmed it works |
| Gate 3 | Convergence | Stakeholder engagement documented, trust toolkit complete |
| Gate 4 | Assurance | Mapped to regulatory/assurance frameworks |

**You can publish at Gate 1.** Gates 2–4 are for benchmarks progressing toward formal AI assurance use.

---

## Your End-to-End Journey

| Step | What you do | Where |
|---|---|---|
| **1** | Create a GitHub repo for your adapter (can be your existing project repo) | Your GitHub |
| **2** | Install `omaib-contracts` CLI | Your local machine |
| **3** | Scaffold your adapter files | Your local machine |
| **4** | Fill in the templates | Your editor |
| **5** | Validate locally | Your local machine |
| **6** | Add the CI validation workflow | Your GitHub repo |
| **7** | Register with OMAIB | GitHub issue |
| **8** | Tag a release | Your GitHub repo |
| **9** | Platform ingests automatically | OMAIB platform |
| **10** | Publish updates by tagging new releases | Your GitHub repo |

The rest of this guide walks you through each step in detail.

---

## Next

→ [01 — Prerequisites and Environment Setup](01-PREREQUISITES.md)
