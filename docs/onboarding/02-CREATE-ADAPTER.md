# 02 — Creating Your Domain Adapter Pack

> **Audience**: Principal Investigators **Time to complete**: 20–30 minutes **Previous step**:
> [01 — Prerequisites and Environment Setup](01-PREREQUISITES.md) complete **Next step**:
> [03 — The Four Required Files: A Field-by-Field Reference](03-FOUR-FILES.md)

---

## What This Document Covers

You will scaffold the `omaib-adapter/` directory inside your repository using the OMAIB CLI, then fill in the scaffolded
templates with your project's specific information. By the end of this document you will have all four required files
created with real values — ready for local validation.

---

## The omaib-adapter/ Directory

Every OMAIB-registered repository has exactly one `omaib-adapter/` folder at the repository root. This folder is the
contract between your dataset and the OMAIB platform. It contains no data — only structured metadata files that describe
your dataset's benchmark properties, governance rules, data profile, and scoring schema.

```text
your-repo/
├── omaib-adapter/              ← Everything OMAIB reads lives here
│   ├── benchmark_contract.yaml   (required)
│   ├── data_profile.json         (required)
│   ├── governance_policy.yaml    (required)
│   ├── scorecard_schema.json     (required)
│   ├── trust_toolkit.yaml        (recommended — needed for Gate 3)
│   └── assurance_crosswalk.yaml  (recommended — needed for Gate 4)
├── .github/
│   └── workflows/
│       └── validate-dap.yml      (set up in Step 05)
└── README.md
```

---

## Step 1 — Decide Your Project Slug

Before running the CLI, choose a short, lowercase identifier for your project. This becomes part of your `adapter_id`
and `benchmark_id` throughout all files.

**Rules:**

- Lowercase letters and hyphens only — no spaces, underscores, or capitals
- Globally unique within OMAIB — check the [adapter registry](https://github.com/omaib/omaib-adapter-registry) if you're
  unsure
- Descriptive but short — 2 to 4 words is typical

**Examples from existing adapters:**

| Project                           | Slug                    | adapter_id                  | benchmark_id                  |
| --------------------------------- | ----------------------- | --------------------------- | ----------------------------- |
| Autonomous driving for NAS        | `nas-self-driving`      | `dap-nas-self-driving`      | `omaib-nas-self-driving`      |
| Sonar acoustic benchmarking       | `sonair`                | `dap-sonair`                | `omaib-sonair`                |
| Carbon neutral living             | `carbon-neutral-living` | `dap-carbon-neutral-living` | `omaib-carbon-neutral-living` |
| Critical manufacturing monitoring | `critical-mm`           | `dap-critical-mm`           | `omaib-critical-mm`           |

The `adapter_id` is always `dap-<slug>` and the `benchmark_id` is always `omaib-<slug>`.

---

## Step 2 — Run omaib-init-adapter

Navigate to the root of your cloned repository (the same level as `README.md`) and run:

```bash
omaib-init-adapter omaib-adapter/ --slug <your-project-slug>
```

**Example** for a sonar acoustics project:

```bash
omaib-init-adapter omaib-adapter/ --slug my-acoustic-benchmark
```

The CLI will create the `omaib-adapter/` directory and populate it with all four required files plus the two recommended
files, with your slug pre-filled wherever it appears:

```text
Created omaib-adapter/benchmark_contract.yaml
Created omaib-adapter/data_profile.json
Created omaib-adapter/governance_policy.yaml
Created omaib-adapter/scorecard_schema.json
Created omaib-adapter/trust_toolkit.yaml     (recommended)
Created omaib-adapter/assurance_crosswalk.yaml (recommended)
```

> **If the CLI is not found**: Ensure your virtual environment is activated (`source .venv/bin/activate`) and that
> `omaib-contracts` was installed successfully. See [01 — Prerequisites and Environment Setup](01-PREREQUISITES.md).

---

## Step 3 — Review The Scaffolded Files

Open `omaib-adapter/` in your text editor. Each file has been pre-filled with:

- Your slug in the `adapter_id` and `benchmark_id` fields
- All required fields present but marked `<placeholder>` or `null` where you must supply a value
- Inline comments explaining every field

The goal is to replace every `<placeholder>` and `null` with real information before running validation.

---

## Step 4 — Fill In benchmark_contract.yaml

This is the most important file. It defines what the benchmark measures and how it operates.

Open `omaib-adapter/benchmark_contract.yaml` and work through each section:

### Identity block

```yaml
adapter_id: dap-my-acoustic-benchmark # already filled by CLI
benchmark_id: omaib-my-acoustic-benchmark # already filled by CLI
version: 0.1.0 # start here; increment when you update
```

### Project block

```yaml
project:
  name: "My Acoustic Benchmark Project" # ← full project name (readable, not an identifier)
  pi_id:
    "<PI-NNN>" # ← opaque PI identifier, no personal names or emails
    #   Leave as PI-TBD if you have not registered yet —
    #   your PI-NNN is assigned by OMAIB during registration
  org_id: "<org-slug>" # ← short organisation slug (e.g. "oxford", "brunel", "mit")
  institutions:
    - "<University / Organisation>"
  award_window: "<Start date - End date>" # ← funding period
```

### License block

```yaml
license:
  code: "MIT"
  data:
    "UK Data Protection Act 2018, DUA signed with partner NHS trust. No raw patient data may leave the custodian
    boundary."
```

### Evaluation modes block

```yaml
evaluation_modes:
  - public # ← pick one or more modes that apply
  # - hidden-test               # uncomment if the platform holds a hidden test set
  # - maintainer-run            # uncomment if OMAIB maintainers run evaluation
  # - custodian-run             # uncomment if the data custodian runs evaluation
  # - federated                 # uncomment if evaluation runs inside your network boundary
  # - restricted                # uncomment if access is restricted under a DUA or ethics agreement
```

Supported modes:

| Mode             | Meaning                                                                |
| ---------------- | ---------------------------------------------------------------------- |
| `public`         | Participants submit predictions; evaluation runs on the OMAIB platform |
| `hidden-test`    | Platform holds a hidden test set; participants do not see it           |
| `maintainer-run` | OMAIB maintainers run evaluation on behalf of participants             |
| `custodian-run`  | The data custodian runs evaluation at their site                       |
| `federated`      | Evaluation runs inside your network boundary; data never leaves        |
| `restricted`     | Access restricted under a DUA or ethics agreement                      |

### Data access block

```yaml
data_access:
  model: "gated" # ← open | registered | gated | custodian-only | hybrid
  compute_location: "omaib-run" # ← omaib-run | participant-run | custodian-run | hybrid
```

| `model` value    | Meaning                                                                 |
| ---------------- | ----------------------------------------------------------------------- |
| `open`           | Dataset is publicly downloadable; OMAIB reads it directly               |
| `registered`     | Access requires registration; approved after application                |
| `gated`          | Access requires institutional approval; OMAIB routes via the GitHub App |
| `custodian-only` | Only the data custodian can perform or authorise evaluation             |
| `hybrid`         | Multiple models apply (e.g., open training split, gated test split)     |

| `compute_location` value | Meaning                                                    |
| ------------------------ | ---------------------------------------------------------- |
| `omaib-run`              | OMAIB platform infrastructure runs the evaluation          |
| `participant-run`        | Participants run evaluation locally and submit results     |
| `custodian-run`          | Data custodian runs evaluation at their own infrastructure |
| `hybrid`                 | Evaluation is split across multiple locations              |

### Tasks block

```yaml
tasks:
  - task_id: <task_id_snake_case> # ← snake_case identifier, e.g. acoustic_classification
    name: "<Human-readable task name>"
    description: >
      One paragraph defining the task — what inputs the model receives, what the model must predict or produce, and the
      evaluation condition.
```

At least one task is required. Most adapters have one task; multi-task benchmarks list each separately.

### Modalities block

```yaml
modalities:
  - text # include only what applies
  - image
  - audio
  - video
  - time_series
  - tabular
```

### Metrics block

```yaml
metrics:
  primary:
    - macro_f1 # ← list at least one primary ranking metric
  secondary:
    - routing_accuracy
    - per_class_recall
  acceptance_thresholds: # OPTIONAL — remove if not yet defined
    macro_f1: 0.72
```

### Baselines block

```yaml
baselines:
  provided:
    - "<Baseline name>" # ← e.g., "Random (majority class)"
  reproducibility:
    release:
      - code
      - config
      - seeds
    containerised: true # ← true if you provide a Docker image
```

### Scorecard policy block

```yaml
scorecard_policy:
  public_fields:
    - overall_metrics_summary # ← fields shown on the public leaderboard
  restricted_fields:
    - "<field with re-identification risk>" # ← use empty list if none
```

### Governance block

```yaml
governance:
  custodian_boundary_owner: "<Institution / Role>"
  legal_basis: "legitimate_interests" # ← contract | consent | legitimate_interests | public_task | DUA
  export_rules:
    - "<Rule 1 — e.g., no patient-level data exported>"
  red_lines:
    - "<Hard line — e.g., no raw clinical records to leave hospital boundary>"
```

### Dependencies block

List any platform services your adapter depends on, e.g., benchmark_registry, scorecard_service, evaluator_runner,
policy_engine, etc.

```yaml
dependencies:
  platform:
    - benchmark_registry
    - scorecard_service
    - evaluator_runner
    - policy_engine
```

---

## Step 5 — Fill In data_profile.json

This file describes your dataset's structure — formats, modalities, splits, and size.

Open `omaib-adapter/data_profile.json`:

```json
{
  "adapter_id": "dap-my-acoustic-benchmark", ← refer to Step 1 — Decide Your Project Slug above
  "benchmark_id": "omaib-my-acoustic-benchmark",   ← refer to Step 1 above
  "version": "0.1.0",                       ← the version of the file
  "formats": ["wav", "csv"],               ← replace with your file formats
  "provenance_fields": [
    "source",
    "collection_timestamp",
    "site",
    "sensor_id",
    "checksum"
  ],
  "alignment": {
    "time_reference": "<UTC | GPS | NTP | N/A>",
    "sync_tolerance_ms": null
  },
  "missingness": {
    "mask_field": "missing_mask",
    "quality_flags_field": "quality_flags"
  },
  "modalities": ["audio", "tabular"],      ← replace with your data modalities
  "sample_unit": "1 recording session from a single microphone array",  ← describe one sample
  "splits": {
    "strategy": "temporal",               ← how you split train/val/test
    "ratios": { "train": 0.7, "val": 0.15, "test": 0.15 },
    "leakage_controls": ["No overlap between recording dates across splits"]
  },
  "size_estimate": {
    "samples": 12000,                     ← number of samples (can be approximate)
    "storage_gb": 45.0,                   ← estimated compressed size
    "duration_hours": null                ← for time-series / audio data
  }
}
```

---

## Step 6 — Fill In governance_policy.yaml

This file defines who can access your data, what they can do with it, and how compliance is enforced.

Open `omaib-adapter/governance_policy.yaml`:

```yaml
adapter_id: dap-benchmark
benchmark_id: omaib-benchmark
version: 0.1.0

# access_tier options: open | registered | gated | custodian-only | hybrid | federated
access_tier: registered

# compute_location options: omaib-run | participant-run | custodian-run | hybrid | site-run
compute_location: participant-run

custodian_boundary:
  controller: "UKOMAIN"
  processor: "OMAIB / University of Sheffield (evaluation infrastructure only)"
  approver_role: "Data Governance Lead, UKOMAIN/OMAIB"
  environment_notes: >
    De-identified dataset released to registered evaluators under DUA. Raw text remains within UKOMAIN boundary. OMAIB
    platform receives aggregate scores only — no raw predictions exported.

output_governance:
  export_rules:
    - "Aggregate evaluation scores (macro-F1, routing accuracy) may be published publicly."
    - "Per-class recall may only be released with written approval from the Data Governance Lead."
    - "No raw model predictions, embeddings, or attention weights may be exported."
  release_policy:
    public_scorecard: true
    restricted_scorecard: true
    human_review_gate: true
  red_lines:
    - "No raw discharge summary text may leave the UKOMAIN boundary."
    - "No patient identifier — direct or indirect — may appear in any exported artefact."
    - "No model trained solely on this dataset may be released without governance approval."
```

---

## Step 7 — Fill In scorecard_schema.json

This file defines the performance metrics your benchmark supports and their acceptable ranges. A minimal scorecard:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://github.com/<your-uni>/<your-project-name>/omaib-adapter/scorecard_schema.json",
  "title": "<Project Name> Scorecard",
  "description": "Defines the structure and public/restricted field policy for scorecard records produced by this benchmark.",
  "type": "object",
  "required": ["benchmark_id", "run_id", "model_id", "macro_f1", "routing_accuracy"],
  "properties": {
    "benchmark_id": {
      "type": "string",
      "description": "Identifies the benchmark this scorecard belongs to. Must be 'omaib-alpha-benchmark'."
    },
    "run_id": {
      "type": "string",
      "description": "Unique identifier for the evaluation run (e.g. 'run-2026-04-01-001')."
    },
    "model_id": {
      "type": "string",
      "description": "Identifier of the model being evaluated (e.g. 'clinicalbert-finetuned-v2')."
    },
    "macro_f1": {
      "type": "number",
      "minimum": 0,
      "maximum": 1,
      "description": "Macro-averaged F1 score across all seven routing categories. Public field."
    },
    "routing_accuracy": {
      "type": "number",
      "minimum": 0,
      "maximum": 1,
      "description": "Top-1 routing label accuracy across the test split. Public field."
    },
    "per_class_recall": {
      "type": "object",
      "description": "Per-category recall scores. Restricted field — requires Data Governance Lead approval to release.",
      "additionalProperties": {
        "type": "number",
        "minimum": 0,
        "maximum": 1
      }
    },
    "overall_metrics_summary": {
      "type": "string",
      "description": "Free-text summary of overall performance for the public leaderboard entry."
    }
  },
  "additionalProperties": false
}
```

Each metric in the `metrics` array of your `benchmark_contract.yaml` must have a corresponding entry here.

---

## Step 8 — Fill In trust_toolkit.yaml (Optional)

```yaml
adapter_id: dap-benchmark
benchmark_id: omaib-benchmark
version: 0.1.0

stakeholder_groups:
  - "Clinical informatics leads (routing decision-makers)"
  - "NHS ward administrators (end users of routing output)"
  - "Information governance officers"
  - "Clinical NLP researchers"

rubrics:
  - rubric_id: R1
    name: "Routing output clarity"
    fields:
      - clarity
      - actionability
      - handoff_burden
      - failure_criticality
    scale: "1-5"
    notes:
      "Assess whether the routing label and confidence score are interpretable by ward staff without clinical NLP
      expertise."

  - rubric_id: R2
    name: "Governance transparency"
    fields:
      - data_sharing_clarity
      - consent_explainability
      - audit_trail_legibility
    scale: "1-5"
    notes: "Assess whether the DUA terms and data flow are clearly communicated to participating institutions."

sessions:
  planned: 3
  completed: 1
  evidence_artifacts:
    - "stakeholder_workshop_2026-04-14_notes.pdf"
```

---

## Step 9 — Fill In assurance_crosswalk.yaml (Optional)

```yaml
adapter_id: dap-benchmark
benchmark_id: omaib-benchmark
version: 0.1.0

standards_targets:
  - "UK GDPR / Data Protection Act 2018"
  - "NHS Data Security and Protection Toolkit (DSPT)"
  - "MHRA AI/ML SaMD Guidance (2024)"

mapping:
  - evidence_field: overall_metrics_summary
    control_id: "DSPT-DSP9"
    standard: "NHS Data Security and Protection Toolkit"
    gate: "pilot"
    notes:
      "Aggregate performance metrics (macro-F1, routing accuracy) demonstrate model utility without exposing
      patient-level data."

  - evidence_field: macro_f1
    control_id: "MHRA-SaMD-4.2"
    standard: "MHRA AI/ML SaMD Guidance 2024"
    gate: "pilot"
    notes:
      "Primary metric for clinical performance threshold — must meet acceptance threshold (0.72) before limited
      deployment."

  - evidence_field: governance_policy
    control_id: "GDPR-Art30"
    standard: "UK GDPR"
    gate: "pilot"
    notes:
      "Governance policy documents the processing activity record (controller, processor, legal basis, data flows)
      required under Article 30."

decision_gates:
  pilot_to_limited: >
    macro_f1 >= 0.72 on held-out test split; Data Governance Lead sign-off on per-class recall release; at least one
    completed stakeholder session with ward administrators.
  limited_to_scale: >
    External clinical audit completed; MHRA SaMD classification confirmed; DUA renewed for multi-site deployment;
    reproducibility manifests (code, config, seeds) published.
```

---

## Step 10 — Commit the Scaffolded Files

Once you've filled in all placeholders, stage and commit:

```bash
git add omaib-adapter/
git commit -m "feat: add OMAIB adapter files for dap-<your-slug>"
git push origin main
```

Do **not** push yet if you have data files — only the metadata files in `omaib-adapter/` should be committed. Data lives
separately.

---

## What a Completed Adapter Looks Like

For reference, here is the `dap-sonair` adapter's directory listing (a real registered adapter):

```text
omaib-adapter/
├── benchmark_contract.yaml
├── data_profile.json
├── governance_policy.yaml
└── scorecard_schema.json
```

All four required files, no extra files, no data. The `trust_toolkit.yaml` and `assurance_crosswalk.yaml` files are
added when the PI is working toward Gate 3 and Gate 4 respectively.

---

## Common Mistakes at This Stage

| Mistake                                                                                                                      | How to avoid it                                                                                                    |
| ---------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| Leaving `<placeholder>` text in any field                                                                                    | Run `grep -r "<" omaib-adapter/` — any output means placeholders remain                                            |
| `adapter_id` in one file doesn't match another                                                                               | All files must use the identical `adapter_id` string                                                               |
| `data_access.model` in `benchmark_contract.yaml` is semantically inconsistent with `access_tier` in `governance_policy.yaml` | Keep them aligned — e.g., `model: gated` pairs with `access_tier: gated`                                           |
| Typo in `evaluation_modes`                                                                                                   | Only these values are valid: `public`, `hidden-test`, `maintainer-run`, `custodian-run`, `federated`, `restricted` |
| Committing data files into `omaib-adapter/`                                                                                  | This folder is metadata-only                                                                                       |

---

## Next

→ [03 — The Four Required Files: A Field-by-Field Reference](03-FOUR-FILES.md)
