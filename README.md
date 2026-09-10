# SecureBERT Threat Analysis Lab

**Research prototype for semantic command classification, session fingerprinting, behavioral clustering, and analyst-facing threat telemetry.**

This repository captures an earlier stage of my security/AI engineering work. The newer **SentinelIQ** project extends the same interest into a production-style streaming architecture with Kafka, PostgreSQL, model-aligned explanations, observability, Docker, and stronger ML-integrity tests.

## Problem

Security telemetry is noisy, repeated attacker behavior can change surface details, and purely signature-based detection misses semantic similarity. This lab explores a hybrid approach:

- deterministic MITRE-oriented rules for high-confidence patterns
- SecureBERT inference for semantic command classification
- BERT session embeddings for similarity-based identity matching
- DBSCAN clustering for behavioral grouping
- a FastAPI/WebSocket backend and Next.js analyst dashboard

The system is intentionally a **lab**, not a claim of production autonomous defense.

## Architecture

```text
Honeypot / command telemetry
            |
            v
       FastAPI backend
       /      |      \
      v       v       v
 rules    SecureBERT  session embeddings
      \       |       /
       \      v      /
        threat classification
               |
      +--------+---------+
      |                  |
      v                  v
MITRE explanation   similarity + DBSCAN
      |                  |
      +--------+---------+
               v
        WebSocket stream
               |
               v
        Next.js dashboard
```

## Engineering evidence

### Hybrid classification
Obvious patterns can be handled deterministically before invoking the model. Unmatched commands are normalized and passed to SecureBERT for semantic classification.

### Session fingerprinting
Session-level BERT embeddings are compared with prior sessions using cosine similarity. The goal is to explore whether behavior can remain recognizable even when an IP address or exact command text changes.

### Behavioral clustering
DBSCAN groups session embeddings without requiring a predefined number of clusters and surfaces noise points as unusual behavior.

### Analyst-facing application
FastAPI exposes telemetry, sessions, fingerprints, clusters, and report generation. A Next.js dashboard consumes the live stream for investigation.

## Safety boundary

The prototype can return a `BLOCK` recommendation when configured thresholds are met, but this repository should **not** be deployed as an autonomous production enforcement system. Real production use would require calibrated external evaluation, policy controls, human review where appropriate, hardened identity/access controls, durable storage, rate limiting, and operational rollback procedures.

## Stack

| Layer | Technology |
| --- | --- |
| ML | Python, PyTorch, Transformers, SecureBERT, scikit-learn, DBSCAN |
| API | FastAPI, WebSockets |
| Frontend | Next.js 14, TypeScript/Tailwind |
| Probe | Python socket programming |
| Threat context | MITRE ATT&CK data |

## Run locally

### Backend

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r siem/requirements.txt
uvicorn siem.main:app --reload --host 0.0.0.0 --port 8000
```

SecureBERT artifacts are expected under `siem/model/securebert/` and are intentionally not committed when they are too large for normal source control.

### Probe

```bash
python agent/listener.py
```

### Dashboard

```bash
cd dashboard
npm ci
npm run dev
```

## Repository structure

```text
agent/                  # telemetry/honeypot probe
siem/                   # API, classification, session analysis
siem/model/             # model integration
dashboard/              # analyst UI
mitre_attack.json       # MITRE dataset used by the lab
.github/workflows/ci.yml
```

## Verification

The repository includes a lightweight model test harness at `siem/test_engine.py`. CI performs source compilation and a production frontend build without pretending that unavailable model weights can be validated in a clean runner.

## Limitations and next steps

- model evaluation here is research-oriented rather than a production benchmark
- the in-memory session/identity stores are not durable
- thresholds require external calibration
- the MITRE data snapshot is intentionally bundled for reproducibility but increases repository size
- model artifacts must be supplied separately
- production identity, authorization, observability, persistence, deployment, and regression gates are demonstrated more completely in **SentinelIQ**

## Portfolio progression

This repository is useful as evidence of the evolution of an idea. It demonstrates semantic security analysis and full-stack experimentation; **SentinelIQ** demonstrates the later engineering step toward streaming ingestion, durable storage, observability, containerization, and serving-path integrity.

## License

MIT.
