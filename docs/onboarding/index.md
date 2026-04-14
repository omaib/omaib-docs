# PI Onboarding Guide

This directory contains the complete onboarding guide for Principal Investigators (PIs) contributing Domain Adapter
Packs (DAPs) to the OMAIB benchmarking platform.

Each document covers one stage of the onboarding journey and can be read independently. Together they form a complete
walkthrough from first contact with OMAIB to a live, maintained benchmark.

---

## Document Index

| #   | Document                                          | What it covers                                    | Estimated time |
| --- | ------------------------------------------------- | ------------------------------------------------- | -------------- |
| 00  | [Overview: What is OMAIB?](00-OVERVIEW.md)        | Platform model, gate system, end-to-end journey   | 10 min read    |
| 01  | [Prerequisites and Setup](01-PREREQUISITES.md)    | GitHub, Python, CLI tools, environment            | 15–20 min      |
| 02  | [Creating Your Adapter](02-CREATE-ADAPTER.md)     | Scaffolding and filling the adapter files         | 20–30 min      |
| 03  | [The Four Required Files](03-FOUR-FILES.md)       | Field-by-field reference for all four files       | 20–30 min read |
| 04  | [Local Validation](04-VALIDATION.md)              | Running validation, understanding gate scores     | 10–15 min      |
| 05  | [CI Setup](05-CI-SETUP.md)                        | GitHub Actions validation workflow                | 10 min         |
| 06  | [Registration](06-REGISTER.md)                    | Submitting registration issue, installing the bot | 15–30 min      |
| 07  | [Releases and Updates](07-RELEASE-AND-UPDATES.md) | Tagging, versioning, the update lifecycle         | 15–20 min      |
| 08  | [Troubleshooting](08-TROUBLESHOOTING.md)          | Common errors and fixes at every stage            | Reference      |

---

## Quick-Start Path

If you're in a hurry:

1. **[Prerequisites](01-PREREQUISITES.md)** — install Python and `omaib-contracts`
2. **[Create Adapter](02-CREATE-ADAPTER.md)** — run `omaib-init-adapter` and fill in the files
3. **[Validation](04-VALIDATION.md)** — run `omaib-validate-adapter` until Gate 1 passes
4. **[CI Setup](05-CI-SETUP.md)** — copy the workflow to `.github/workflows/`
5. **[Register](06-REGISTER.md)** — open the registration issue

Total time for an experienced PI with a well-documented dataset: ~2 hours.

---

## Key Concepts

**Domain Adapter Pack (DAP):** The metadata bundle describing your benchmark. Lives in `omaib-adapter/` in your
repository. Contains four required YAML/JSON files.

**Gate system:** Four progressive validation levels. Gate 1 is local/CI schema validation. Gates 2–4 are run by the
platform after registration.

**adapter_id:** Your unique identifier — always `dap-<your-slug>`.

**benchmark_id:** The benchmark's unique identifier — always `omaib-<your-slug>`.

---

## Getting Help

- [omaib-contracts Issues](https://github.com/omaib/omaib-contracts/issues) — platform support
- [omaib-docs Discussions](https://github.com/omaib/omaib-contracts/discussions) — peer questions
- Email: <omaib-ukomain-group@sheffield.ac.uk> — institutional/federated queries
