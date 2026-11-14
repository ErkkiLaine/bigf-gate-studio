# BIGF v2.4 Gate Compiler & Studio

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.24215776.svg)](https://doi.org/10.5281/zenodo.24215776)
[![License: CC BY-NC 4.0](https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc/4.0/)

**Live Browser Studio:** [https://erkkilaine.github.io/bigf-gate-studio/](https://erkkilaine.github.io/bigf-gate-studio/)

A zero-dependency, air-gapped browser compiler and deterministic simulator for the **Business Impact Governance Framework (BIGF v2.4)**. It compiles canonical **BIA Anchor Specifications** into deterministic Open Policy Agent (OPA / Rego v1) runtime authorization gates (`Authority = f(B, S, T)`), pure-integer $10^{-4}$ fixed-point arithmetic (zero IEEE-754 floating-point parsing), RFC 8785 JCS SHA-256 cryptographic anchor digests, unit tests, and telemetry collectors.

> *"BIA leads; SLA lags. If governance requires a meeting, it is already too late."*

---

## Core Architecture & Generated Bundle

Running the studio (either online via GitHub Pages or locally by opening `index.html` offline) generates a complete 5-file deployment bundle:

1. **`bia_anchor.json`** — Canonical governance contract binding financial ceilings, telemetry thresholds, TTL bounds, and the RFC 8785 SHA-256 anchor digest.
2. **`bigf_gate.rego`** — Fail-closed OPA / Rego v1 policy enforcing `200 ALLOW`, `200 SHADOW_WOULD_HALT`, `200 OVERRIDE_ACTIVE`, or `403 HARD HALT` with Canonical Decision Record (CDR) logging.
3. **`bigf_gate_test.rego`** — Deterministic unit test suite covering nominal execution, anchor tampering, B/S/T breaches, and Break-Glass expiration.
4. **`collect_biii.py`** — Zero-dependency Python 3 telemetry collector computing `BIII = min(S_telemetry, S_dependency, S_capacity, S_drift)` in pure integer arithmetic.
5. **`DEPLOY_GUIDE.md`** — Operational sizing, calibration, and incident containment runbook.

---

## Quick Sizing Guide (B, S, T & Break-Glass)

All numeric parameters use exact 4-decimal fixed-point formatting (`0.0000`) to guarantee deterministic integer evaluation across runtimes.

### 1. B-Axis: Business Impact Ceiling (`max_exposure_eur_hr_fp4`)
Calculate maximum acceptable hourly exposure using:  
`Hourly Exposure = Lost Revenue + SLA Penalties + Incident Recovery Cost`

| Criticality Tier | Example Systems | Recommended Ceiling (EUR/hr) | Typical Allowed Actions |
| :--- | :--- | :--- | :--- |
| **Tier 1: Mission-Critical Core** | Payment rails, core routing, identity/auth | `250000.0000` | `canary_deploy`, `rollback_config` |
| **Tier 2: Customer-Facing** | E-commerce, booking, customer portals | `50000.0000` | `route_shift`, `canary_deploy`, `rollback_config` |
| **Tier 3: Internal Operations** | Reporting, internal tooling, staging | `5000.0000` | `route_shift`, `canary_deploy`, `rollback_config`, `scale_out` |

### 2. S-Axis: System Integrity Threshold (`min_biii_fp4`)
The **Business Impact Integrity Index (BIII)** $\in [0.0000, 1.0000]$ is determined by the weakest of four live telemetry signals:  
$$\text{BIII} = \min(S_{\text{telemetry}},\; S_{\text{dependency}},\; S_{\text{capacity}},\; S_{\text{drift}})$$

* **`0.8000` (Canonical Baseline):** Standard BIGF default; permits normal operational variance while halting cascading failures.
* **`0.8500` (High Integrity):** Recommended for customer-facing services sensitive to latency spikes.
* **`0.9500` (Zero-Tolerance):** Core financial and authentication infrastructure.

### 3. T-Axis: Temporal Bounds & Freeze Windows
* **`max_ttl_seconds`:** `60–300 s` for autonomous remediation bots; `900–1800 s` for CI/CD deployment pipelines.
* **`block_friday_weekend_freeze`:** `true` to automatically block non-emergency routing and configuration mutations during high-risk change freeze windows.

### 4. Break-Glass Emergency Override
When an active outage or zero-day containment requires intervention *while* `BIII < min_biii_fp4`, operators issue a time-bounded Break-Glass override rather than disabling the gate:
* Must be explicitly bound to a single `change_id` and attributed to a named `commander_id`.
* Strictly capped at `break_glass_max_ttl_seconds <= 900` (15 minutes), automatically reverting to `ENFORCE` upon expiry.
* Emits `200 OVERRIDE_ACTIVE (BREAK-GLASS CDR LOGGED)` into the immutable audit trail.

---

## Three-Phase Production Rollout

1. **Weeks 1–2 (`SHADOW` Mode):** Deploy the compiled anchor in `SHADOW` mode and monitor `SHADOW_WOULD_HALT` events in CDR logs during normal peak traffic.
2. **Threshold Calibration:** Tune `min_biii_fp4` or sensor polling intervals if normal load bursts trigger false positives.
3. **Production Lock (`ENFORCE` Mode):** Recompute the SHA-256 anchor digest and switch `enforcement_mode` to `ENFORCE` (`403 HARD HALT` on invariant breach).

---

## Author & Citation

**Author:** Erkki Laine (VITI Labs)  
**Foundational Specification:** *Business Impact Governance Framework (BIGF)* — DOI: [`10.5281/zenodo.22658623`](https://doi.org/10.5281/zenodo.22658623)  
**Executive Simulator:** *BIGF Executive Search Simulator* — DOI: [`10.5281/zenodo.23667357`](https://doi.org/10.5281/zenodo.23667357)  
**Reference Framework:** *BIGF Framework* — DOI: [`10.5281/zenodo.23667573`](https://doi.org/10.5281/zenodo.23667573)  
**License:** Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)
