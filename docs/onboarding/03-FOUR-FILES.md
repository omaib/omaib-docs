# 03 — The Four Required Files: A Field-by-Field Reference

> **Audience**: Principal Investigators
> **Time to read**: 20–30 minutes
> **Prerequisites**: [02 — Creating Your Domain Adapter Pack](02-CREATE-ADAPTER.md) — adapter already scaffolded
> **Next step**: [04 — Local Validation and Gate Scores](04-VALIDATION.md)

---

## Overview

Your `omaib-adapter/` directory must include exactly these four required files. Together they form a complete machine-readable contract between your dataset and the OMAIB benchmarking platform. This document provides a detailed reference for every field in every file.

| File | Purpose | Format |
|---|---|---|
| `benchmark_contract.yaml` | Defines the benchmark — what it measures, how it runs, who owns it | YAML |
| `data_profile.json` | Describes the dataset structure — modalities, splits, size, provenance | JSON |
| `governance_policy.yaml` | Defines access tier, compute location, custodian boundary, and output governance | YAML |
| `scorecard_schema.json` | Specifies the metrics the benchmark accepts and their valid ranges | JSON |

---

## File 1 — benchmark_contract.yaml

This is the central contract file. It ties everything together with a unique identifier and describes the benchmark's scientific and operational properties.

### Full annotated example

```yaml
benchmark_id: omaib-sonair              # REQUIRED — globally unique benchmark identifier
                                         # Format: omaib-<slug>. Must be lowercase, hyphens only.
                                         # This value must be identical across all 4 files.

adapter_id: dap-sonair                  # REQUIRED — globally unique adapter identifier
                                         # Format: dap-<slug>. Same slug as benchmark_id.

version: 0.1.0                          # REQUIRED — semantic version. Start at 0.1.0.
                                         # Increment patch (0.1.x) for minor corrections.
                                         # Increment minor (0.x.0) for new metrics or splits.
                                         # Increment major (x.0.0) for breaking schema changes.

project:
  name: "SONAIR Benchmark Project"       # REQUIRED — full readable project name
  pi_id: "PI-042"                                 # REQUIRED — opaque PI identifier (no personal data)
                                                   # Format: PI-NNN. Use PI-TBD until registration —
                                                   # your PI-NNN is assigned by OMAIB during registration.
  org_id: "oxford"                                # REQUIRED — short org slug (e.g. oxford, brunel, mit)
  institutions:
    - "University of Oxford"
  award_window: "2023-01 - 2026-12"              # funding period (no personal data)

license:
  data: "CC BY-NC 4.0"                  # REQUIRED — SPDX identifier or custom license name
  code: "MIT"                           # OPTIONAL — if you provide evaluation code

evaluation_modes:                       # REQUIRED — list of evaluation execution modes
  - public                              # Choose one or more from:
  - hidden-test                          #   public, hidden-test, maintainer-run,
                                         #   custodian-run, federated, restricted

data_access:                            # REQUIRED
  model: "gated"                        #   open | registered | gated | custodian-only | hybrid
  compute_location: "omaib-run"         #   omaib-run | participant-run | custodian-run | hybrid

tasks:                                  # REQUIRED — at least one task
  - task_id: acoustic_classification    #   snake_case identifier
    name: "Acoustic Event Classification"
    description: >
      Classify each 30-second underwater recording into one of 12 acoustic
      event categories using hydrophone array input.

modalities:
  - text             # include only what applies
  - image
  - audio
  - video
  - time_series
  - tabular

metrics:
  primary:                              # REQUIRED — at least one primary ranking metric
    - macro_f1
  secondary:                            # OPTIONAL — additional reported metrics
    - precision
    - recall
    - roc_auc
  acceptance_thresholds:               # OPTIONAL
    macro_f1: 0.7

baselines:                              # REQUIRED
  provided:
    - "Random (majority class)"
  reproducibility:
    release:
      - code
      - config
      - seeds
    containerised: true

scorecard_policy:                       # REQUIRED
  public_fields:
    - overall_metrics_summary
  restricted_fields:
    - raw_submission_scores             # restrict fields with re-identification risk

governance:                             # REQUIRED — summary governance constraints
  custodian_boundary_owner: "University of Oxford — Data Services"
  legal_basis: "legitimate_interests"  # contract | consent | legitimate_interests | public_task | DUA
  export_rules:
    - "No raw hydrophone recordings to leave secure data enclave"
  red_lines:
    - "No geolocation metadata attached to recordings"

dependencies:                           # REQUIRED
  platform:
    - benchmark_registry
    - scorecard_service
    - evaluator_runner
    - policy_engine
```

### Field reference table

| Field | Required | Type | Notes |
|---|---|---|---|
| `benchmark_id` | ✅ | string | `omaib-<slug>` |
| `adapter_id` | ✅ | string | `dap-<slug>` |
| `version` | ✅ | semver string | Start at `0.1.0` |
| `evaluation_modes` | ✅ | list of strings | At least one: `public`, `hidden-test`, `maintainer-run`, `custodian-run`, `federated`, `restricted` |
| `data_access.model` | ✅ | string | `open` \| `registered` \| `gated` \| `custodian-only` \| `hybrid` |
| `data_access.compute_location` | ✅ | string | `omaib-run` \| `participant-run` \| `custodian-run` \| `hybrid` |
| `metrics.primary` | ✅ | list of strings | At least one primary ranking metric name |
| `metrics.secondary` | optional | list of strings | Additional reported metrics |
| `tasks` | ✅ | list of objects | At least one task with `task_id`, `name`, `description` |
| `baselines` | ✅ | object | `provided` list and `reproducibility` sub-object required |
| `scorecard_policy` | ✅ | object | `public_fields` list required (at least one item) |
| `governance` | ✅ | object | `export_rules` list required (at least one item) |
| `dependencies` | ✅ | object | `platform` list of required platform services |
| `project.pi_id` | ✅ | string | Opaque identifier, format `PI-NNN` |
| `project.org_id` | ✅ | string | Short organisation slug |
| `license.data` | ✅ | string | SPDX identifier preferred |

---

## File 2 — data_profile.json

This file is a machine-readable description of your dataset's structure. OMAIB uses it to validate that evaluation submissions are compatible with your benchmark.

### Full annotated example

```json
{
  "_comment": "Copy to omaib-adapter/data_profile.json — remove this line before validating",

  "adapter_id": "dap-sonair",           // must match benchmark_contract.yaml
  "benchmark_id": "omaib-sonair",       // must match benchmark_contract.yaml
  "version": "0.1.0",                   // must match benchmark_contract.yaml

  "formats": ["wav", "csv"],            // REQUIRED — list of file formats in the dataset
                                         // Common values: parquet, jsonl, csv, hdf5,
                                         // rosbag, dicom, wav, mp4, png, npy

  "modalities": ["audio", "tabular"],   // REQUIRED — list of data modalities present
                                         // Common values: audio, video, image, tabular,
                                         // text, lidar, radar, imu, thermal, depth

  "sample_unit": "One 30-second recording from a single hydrophone array",
                                         // REQUIRED — plain-English description of what
                                         // constitutes one sample in your dataset.
                                         // Evaluators use this to understand the evaluation unit.

  "provenance_fields": [                // RECOMMENDED — fields present in each sample that
    "source",                           // document where the data came from. Used for
    "collection_timestamp",             // reproducibility audits.
    "site",
    "sensor_id",
    "checksum"
  ],

  "alignment": {                        // RECOMMENDED — temporal alignment information
    "time_reference": "UTC",            // UTC | GPS | NTP | N/A
    "sync_tolerance_ms": 10             // acceptable synchronisation tolerance (null if N/A)
  },

  "missingness": {                      // RECOMMENDED — how missing data is encoded
    "mask_field": "missing_mask",       // name of the field carrying the missingness mask
    "quality_flags_field": "quality_flags"  // name of field carrying quality flags
  },

  "splits": {
    "strategy": "temporal",            // REQUIRED — how the dataset is split
                                        // Values: random | temporal | site-based | stratified
    "ratios": {                         // REQUIRED — exact split proportions (must sum to 1.0)
      "train": 0.70,
      "val": 0.15,
      "test": 0.15
    },
    "leakage_controls": [              // REQUIRED — list at least one control.
      "No recording sessions overlap across splits",
      "All samples from a single deployment appear in one split only"
    ]
  },

  "size_estimate": {
    "samples": 8400,                   // REQUIRED — total number of samples (approximate OK)
    "storage_gb": 22.5,                // REQUIRED — estimated size in GB (compressed if applicable)
    "duration_hours": 70.0             // OPTIONAL — total playback/capture time (for time-series)
  }
}
```

### Field reference table

| Field | Required | Notes |
|---|---|---|
| `adapter_id` | ✅ | Must match `benchmark_contract.yaml` |
| `benchmark_id` | ✅ | Must match `benchmark_contract.yaml` |
| `version` | ✅ | Must match `benchmark_contract.yaml` |
| `formats` | ✅ | At least one format |
| `modalities` | ✅ | At least one modality |
| `sample_unit` | ✅ | Plain-English description |
| `splits.strategy` | ✅ | One of the four strategies |
| `splits.ratios` | ✅ | Must sum to 1.0 |
| `splits.leakage_controls` | ✅ | At least one item |
| `size_estimate.samples` | ✅ | Integer |
| `size_estimate.storage_gb` | ✅ | Float |
| `provenance_fields` | recommended | Improves reproducibility score |
| `alignment` | recommended | Required for multi-modal / time-series data |

---

## File 3 — governance_policy.yaml

This file defines who can use your data, under what conditions, and what the PI's compliance obligations are. OMAIB Gate 2 requires this file to have no `TBD` values.

### Full annotated example

```yaml
adapter_id: dap-sonair                  # must match benchmark_contract.yaml
benchmark_id: omaib-sonair             # must match benchmark_contract.yaml
version: 0.1.0                         # must match benchmark_contract.yaml

# access_tier options: open | registered | gated | custodian-only | hybrid | federated
access_tier: gated                      # REQUIRED — keep semantically consistent with data_access.model in benchmark_contract.yaml

# compute_location options: omaib-run | participant-run | custodian-run | hybrid | site-run
compute_location: omaib-run             # REQUIRED

custodian_boundary:                     # REQUIRED
  controller: "University of Nottingham"
  processor: "OMAIB / University of Sheffield (evaluation infrastructure only)"
  approver_role: "Data Governance Lead, SONAIR / OMAIB"
  environment_notes: >
    De-identified acoustic dataset released to registered evaluators under DUA.
    Raw hydrophone recordings remain within the secure data enclave.
    OMAIB platform receives aggregate scores only — no raw predictions exported.

output_governance:                      # REQUIRED
  export_rules:
    - "Aggregate evaluation scores (macro_f1, precision, recall) may be published publicly."
    - "Per-class recall may only be released with written approval from the Data Governance Lead."
    - "No raw hydrophone recordings may leave the secure data enclave."
  release_policy:
    public_scorecard: true
    restricted_scorecard: true
    human_review_gate: true
  red_lines:
    - "No geolocation metadata attached to recordings may be exported."
    - "No raw hydrophone recordings may leave the secure data enclave."
```

### Common governance mistakes

| Mistake | Correct approach |
|---|---|
| `access_tier` in `governance_policy.yaml` isn't semantically consistent with `data_access.model` in `benchmark_contract.yaml` | Keep them aligned (e.g. both `gated`, or `custodian-only` ↔ `custodian-run`) |
| `custodian_boundary` block missing or contains `TBD` values | Required — must name `controller`, `processor`, and `approver_role` before Gate 2 |
| `output_governance.red_lines` is empty or omitted | List at least one hard constraint; use `[]` only if the dataset is fully open with no restrictions |
| `compute_location` not specified or left as `TBD` | Required — one of: `omaib-run`, `participant-run`, `custodian-run`, `hybrid`, `site-run` |

---

## File 4 — scorecard_schema.json

This file defines the structure of a valid evaluation result submission. Evaluators submit scorecards that are validated against this schema before they appear on the leaderboard.

### Full annotated example

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://github.com/your-org/your-sonair-repo/omaib-adapter/scorecard_schema.json",
  "title": "SONAIR Scorecard",
  "description": "Defines the structure and public/restricted field policy for scorecard records produced by the SONAIR benchmark.",
  "type": "object",
  "required": ["benchmark_id", "run_id", "model_id", "macro_f1"],   // list primary metric(s) here
  "properties": {
    "benchmark_id": {
      "type": "string",
      "description": "Identifies the benchmark this scorecard belongs to. Must be 'omaib-sonair'."
    },
    "run_id": {
      "type": "string",
      "description": "Unique identifier for the evaluation run (e.g. 'run-2026-04-01-001')."
    },
    "model_id": {
      "type": "string",
      "description": "Identifier of the model being evaluated."
    },
    "macro_f1": {                                                    // primary metric — must match metrics.primary in benchmark_contract.yaml
      "type": "number",
      "minimum": 0,
      "maximum": 1,
      "description": "Macro-averaged F1 score across all acoustic event classes. Public field."
    },
    "precision": {
      "type": "number",
      "minimum": 0,
      "maximum": 1,
      "description": "Macro-averaged precision across all classes. Public field."
    },
    "recall": {
      "type": "number",
      "minimum": 0,
      "maximum": 1,
      "description": "Macro-averaged recall across all classes. Public field."
    },
    "roc_auc": {
      "type": "number",
      "minimum": 0.5,
      "maximum": 1,
      "description": "Area under the ROC curve (one-vs-rest, macro average). Public field."
    },
    "overall_metrics_summary": {
      "type": "string",
      "description": "Free-text summary of overall performance for the public leaderboard entry."
    }
  },
  "additionalProperties": false
}
```

### Metric naming convention

Use snake_case. Common metric names recognised by OMAIB's leaderboard renderer:

| Metric | `name` value | `lower_is_better` |
|---|---|---|
| Root mean square error | `rmse` | `true` |
| Mean absolute error | `mae` | `true` |
| R-squared | `r2_score` | `false` |
| F1 (macro) | `macro_f1` | `false` |
| Accuracy | `accuracy` | `false` |
| Precision (macro) | `precision` | `false` |
| Recall (macro) | `recall` | `false` |
| AUC-ROC | `roc_auc` | `false` |
| Log loss | `log_loss` | `true` |
| Mean IoU | `mean_iou` | `false` |

Custom metric names are accepted — just keep them consistent across `benchmark_contract.yaml` and `scorecard_schema.json`.

---

## Cross-file Consistency Rules

All four files must be internally consistent. These values must be **identical** across all files that contain them:

| Value | Where it appears |
|---|---|
| `adapter_id` | All 4 files |
| `benchmark_id` | All 4 files |
| `version` | All 4 files |
| `data_access.model` / `access_tier` | `data_access.model` in `benchmark_contract.yaml` should be semantically consistent with `access_tier` in `governance_policy.yaml` |
| Primary metric names | Each name in `metrics.primary` in `benchmark_contract.yaml` must appear as a key under `properties` in `scorecard_schema.json` |

Running `omaib-validate-adapter omaib-adapter/` checks all of these automatically. Inconsistencies cause a Gate 1 block.

---

## Next

→ [04 — Local Validation and Gate Scores](04-VALIDATION.md)
