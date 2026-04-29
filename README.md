# ARES-Ring Artifact 

This repository contains the artifact for the paper:

**ARES-Ring: Fixed-Format One-Gate Verification for Updatable Consent in Anonymous Access Control**

ARES-Ring evaluates an OGTS (One-Gate Transcript Security) verifier interface with constant-shape transcripts under consent churn, including OSN and de-identified e-health workloads, privacy/indistinguishability tests, performance/scalability, and an offline LLM proposer loop (proposer-only).

---

## 1. Artifact Contents

### 1.1. Paper and Figures
- `paper/Third_Research(3).pdf` (camera-ready draft used for artifact reference)
- `figures/OGTS.pdf` (one-gate OGTS contract workflow)
- `figures/consent-state.pdf` (consent-state transition diagram)

### 1.2. Datasets (provided as CSV)
OSN:
- `data/prime_ring_osn_twitter_users.csv`
- `data/prime_ring_osn_twitter_edges.csv`
- `data/prime_ring_osn_twitter_actions.csv`
- `data/prime_ring_osn_facebook_users.csv`
- `data/prime_ring_osn_facebook_edges.csv`
- `data/prime_ring_osn_facebook_actions.csv`

E-health:
- `data/prime_ring_ehealth_staff_graph_edges.csv`
- `data/prime_ring_ehealth_access_log.csv`

### 1.3. Evaluation Artifacts (zipped bundles)
Each bundle is self-contained and produces LaTeX tables for the corresponding metric family.

- `artifacts/Correctness-and-Safety-Under-Churn-ARES-Ring-Artifact-v2.zip`
- `artifacts/ARES-Ring-OGTS-fixed-format-compliance-AE-Artifact-v3.zip`
- `artifacts/ARES-Ring-Privacy-AE-Artifact-v5.zip`
- `artifacts/ARES-Ring-Performance-Scalability-AE-Artifact-v4.zip`
- `artifacts/ARES-Ring-Churn-Realism-AE-Artifact-v1.zip`
- `artifacts/ARES-Ring-LLM-Loop-Metrics-AE-Artifact-v1.zip`

### 1.4. Baseline Papers (for positioning only)
`baselines/` contains PDFs used to align baseline names and citations in the paper tables (not required to run code).

---

## 2. System Requirements

### 2.1. Hardware
- Tested: Desktop (Intel i7-12700, 32 GB RAM) and Raspberry Pi 4 (4 GB RAM)
- Recommended for full runs: ≥ 16 GB RAM

### 2.2. OS and Toolchain
- Ubuntu 22.04 recommended (other Linux likely works)
- R (>= 4.2)
- Optional (system-mode): `curl`, `ab` or `wrk`, and a local HTTP runtime if the Gate service is enabled

### 2.3. R Packages (typical)
The bundles include scripts that may install or check dependencies. If manual install is needed:
- `data.table`, `dplyr`, `ggplot2`, `jsonlite`, `digest`, `microbenchmark`, `boot`, `pROC`, `caret`
- For classifiers: `xgboost` or `ranger` (depending on bundle configuration)

---

## 3. Quickstart

### 3.1. Unpack all artifacts
From repo root:
```bash
mkdir -p run && cd run
for z in ../artifacts/*.zip; do unzip -o "$z" -d "$(basename "$z" .zip)"; done
````

### 3.2. Run the “must-have” evaluation set

Run these bundles first:

1. Correctness and safety under churn
2. OGTS fixed-format compliance
3. Privacy and indistinguishability
4. Performance and scalability

Each bundle includes a `README.md` (or `RUNME.md`) with exact commands. A typical pattern is:

```bash
cd run/<BUNDLE_DIR>
Rscript scripts/run_all.R
```

### 3.3. Outputs

All bundles emit LaTeX tables into:

* `out/tables/*.tex` (primary)
* `out/logs/*.log` (run logs, parameters, seeds)
* `out/csv/*.csv` (intermediate aggregates, if enabled)

---

## 4. Claims and Reproduction Map (NDSS-style)

### Claim C1: Correctness and safety under churn

**Metrics:** TAR, FAR, ValidUpdAcc, InvalidUpdRej, Churn@k, MaxStableK, ReplayRej, RollbackRej
**Artifact:** `Correctness-and-Safety-Under-Churn-ARES-Ring-Artifact-v2.zip`
**Primary outputs:**

* `tab:churn_correctness_summary`
* `tab:update_validity_by_type`
* `tab:failure_modes`

### Claim C2: OGTS fixed-format compliance

**Metrics:** mean/std/max transcript bytes, EVR_B, verify-time mean/p99, EVR_Δ, verifier path count
**Artifact:** `ARES-Ring-OGTS-fixed-format-compliance-AE-Artifact-v3.zip`
**Primary output:**

* `tab:ogts_compliance_all`

### Claim C3: Privacy and indistinguishability

**Metrics:** signer identification advantage, update-type macro-F1/macro-AUC, worst-pair advantage, linkability AUC/adv, MI, handle churn
**Artifact:** `ARES-Ring-Privacy-AE-Artifact-v5.zip`
**Primary outputs:**

* `tab:signer_adv`
* `tab:privacy_main`

### Claim C4: Performance and scalability

**Metrics:** latency breakdown (Update/Sign/Verify/Open), throughput, peak memory, proof/signature/transcript sizes, scaling with |R|
**Artifact:** `ARES-Ring-Performance-Scalability-AE-Artifact-v4.zip`
**Primary outputs:**

* `tab:perf_summary`
* `tab:footprint`
* `tab:ares_scaling`

### Claim C5: Dataset-grounded churn realism

**Metrics:** churn intensity distribution, constraint-change mix, burstiness statistics
**Artifact:** `ARES-Ring-Churn-Realism-AE-Artifact-v1.zip`
**Primary outputs:**

* `tab:churn_intensity`
* `tab:constraint_mix`
* `tab:burstiness`

### Claim C6: LLM stress-case loop usefulness (proposer-only)

**Metrics:** APR, coverage gain, worst-case uplift, exclusion of LLM runtime from protocol timings
**Artifact:** `ARES-Ring-LLM-Loop-Metrics-AE-Artifact-v1.zip`
**Primary outputs:**

* `tab:llm_loop_metrics`
* `tab:llm_uplift`
* `tab:must_have_set`

---

## 5. Reproducibility Controls

* **Seeds:** Each bundle logs RNG seeds in `out/logs/`.
* **Bootstrapping:** Confidence intervals use 10,000 bootstrap resamples where applicable and record the resampling seed.
* **Fixed envelopes:** OGTS byte/time envelopes are recorded per ring size in `out/logs/envelopes.json` (or equivalent).
* **Deterministic checking:** The LLM proposer loop is filtered by a deterministic checker. LLM time is excluded from protocol timing tables by construction.

---

## 6. System-Mode Gate Service (Optional)

Some bundles include a “system-mode” run that evaluates end-to-end gate latency under concurrency and injected root-fetch delay.

Typical workflow:

1. Start local Gate service:

```bash
Rscript services/gate_service.R --port 8080
```

2. Run clients:

```bash
Rscript scripts/run_system_mode.R --endpoint http://127.0.0.1:8080/gate --concurrency 64 --inject_delay_ms 5
```

Outputs:

* `out/system_mode_latency.tex`
* `out/cache_hit_rate.tex`
* `out/stale_root_rejects.tex`

---

## 7. Expected Runtime

* Correctness, OGTS compliance: minutes to low tens of minutes (depends on dataset slice and resampling)
* Privacy classifiers: longer (model training), typically tens of minutes
* Performance/scalability: depends on repetitions and profiling enabled

---

## 8. Safety Notes

This artifact does **not** include weaponized exploit code. It contains evaluation scripts and de-identified or public-graph-derived datasets. The Gate service is local-only by default.

---

## 9. License and Citation

### 9.1. Citation

If you use this artifact, cite the corresponding NDSS submission (BibTeX in `paper/bib/aresring.bib`, if included).

### 9.2. Licensing

* Code: MIT (recommended)
* Data: follow original dataset terms (SNAP and derived logs). De-identified e-health logs are provided for research evaluation only.

---

## 10. Contact

For questions or reproduction issues:

* Open a GitHub issue with:

  * bundle name
  * command executed
  * `out/logs/` files
  * OS/R version


