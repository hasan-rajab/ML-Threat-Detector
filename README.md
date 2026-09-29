# SecureBERT Threat Analysis Lab

**A research prototype exploring how semantic command understanding and behavioral similarity can add context beyond signature-only security detection.**

This project represents an earlier stage of my AI-security work. It investigates a practical security question:

> **Can suspicious behavior remain recognizable when exact commands, IP addresses or surface-level indicators change?**

The lab combines deterministic rules, SecureBERT command classification, session embeddings, cosine similarity and DBSCAN clustering, then surfaces results through an analyst-facing API/dashboard.

For the newer production-style evolution of this work, see **[SentinelIQ](https://github.com/hasan-rajab/SentinelIQ)**.

> **Scope:** this repository is a research lab. It is not a production autonomous-blocking system.

---

## Security value

Signature and rule-based controls remain useful for known patterns, but they can miss behavior that is semantically similar while syntactically different.

This lab explores three complementary signals:

1. **known-pattern detection** through deterministic MITRE-oriented rules;
2. **semantic classification** through SecureBERT;
3. **behavioral similarity** through session embeddings and unsupervised clustering.

The intended analyst value is better context for investigation — not automatic enforcement.

---

## Architecture

```text
Honeypot / command telemetry
            ↓
       FastAPI backend
       ┌────┼──────────┐
       ↓    ↓          ↓
    rules SecureBERT session embeddings
       └────┼──────────┘
            ↓
     threat classification
       ┌────┴─────────┐
       ↓              ↓
MITRE context    similarity + DBSCAN
       └────┬─────────┘
            ↓
      WebSocket stream
            ↓
     Next.js dashboard
```

---

## Engineering evidence

### Hybrid classification
High-confidence known patterns can be handled deterministically before unmatched commands are normalized and passed into SecureBERT.

### Session fingerprinting
Session-level BERT embeddings are compared with earlier sessions through cosine similarity to explore behavior identity beyond IP address or exact command text.

### Behavioral clustering
DBSCAN groups session embeddings without requiring a fixed cluster count and treats noise points as potentially unusual behavior.

### Analyst-facing delivery
FastAPI exposes telemetry, sessions, fingerprints, clusters and report generation; a Next.js dashboard consumes the live stream.

### CI reproducibility
The repository's reference CI run #5 completed successfully on **10 September 2026**, including source checks and a successful optimized Next.js production build.

---

## Safety boundary

The prototype can emit a `BLOCK` recommendation under configured conditions, but the repository should **not** be deployed as autonomous enforcement.

A real production path would require:

- calibrated external evaluation;
- durable storage;
- enterprise identity and authorization;
- hardened APIs and rate limiting;
- human review and policy controls;
- production observability;
- rollback/incident procedures.

---

## Technology

**ML:** Python · PyTorch · Transformers · SecureBERT · scikit-learn · DBSCAN  
**API:** FastAPI · WebSockets  
**Frontend:** Next.js · TypeScript/Tailwind  
**Threat context:** MITRE ATT&CK  
**Probe:** Python socket programming

---

## Run locally

Backend:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r siem/requirements.txt
uvicorn siem.main:app --reload --host 0.0.0.0 --port 8000
```

Probe:

```bash
python agent/listener.py
```

Dashboard:

```bash
cd dashboard
npm ci
npm run dev
```

Large SecureBERT artifacts are intentionally excluded from normal source control.

---

## Portfolio progression

This project is useful because it shows the **evolution of the problem**, not because it is the newest architecture.

- **SecureBERT Threat Lab:** semantic classification, session similarity, clustering, analyst UI.
- **SentinelIQ:** streaming ingestion, durable persistence, multimodal serving, model-aligned explanations, observability, Docker and stronger ML-integrity regression tests.

That progression reflects the move from "can this ML idea work?" to "how would this capability behave inside an operational system?"
