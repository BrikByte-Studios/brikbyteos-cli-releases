# BrikByteOS CLI Releases

## 🚦 Stop Bad Releases Before They Hit Production

BrikByteOS is a deterministic release certification CLI that answers one question:

> **"Is this release safe, compliant, and ready to ship?"**

It turns fragmented CI signals into a clear release decision:

```text
APPROVED ✅
REJECTED ❌
CONDITIONAL ⚠️
```

---

## ⚡ 30-Second Demo

Run BrikByteOS in your project:
```bash
bb doctor --workdir .
bb run --all --workdir .
bb score --workdir .
bb gate --workdir .
```
Example result:
```
❌ Critical vulnerabilities detected
⚠️ Performance regression detected
📉 Score: 58 → REJECTED
```
No dashboard required.  
No guesswork.  
Just a clear release decision.

---

## 🔁 CI Integration

Add BrikByteOS to your CI pipeline:
```yaml
- name: BrikByte Gate
  run: bb gate --workdir .
```
This is the conversion moment:

> BrikByteOS blocks unsafe releases before they reach production.

---

## 📊 Proof

BrikByteOS helps teams:
- 🚫 Catch critical vulnerabilities before deployment
- ⚠️ Detect performance regressions
- 📉 Reject low-confidence releases
- 🔍 Explain why a release failed
- 🧾 Preserve deterministic evidence for review

> A release can look green in CI and still be unsafe.  
> BrikByteOS makes that risk visible.

---

## ⚡ Install → Run in Under 2 Minutes
### Linux amd64
```bash
VERSION=v0.1.2
BASE_URL="https://github.com/BrikByte-Studios/brikbyteos-cli-releases/releases/download/${VERSION}"

curl -fL "${BASE_URL}/bb_linux_amd64.tar.gz" -o /tmp/bb.tar.gz
curl -fL "${BASE_URL}/checksums.txt" -o /tmp/checksums.txt

cd /tmp
grep ' bb_linux_amd64.tar.gz$' checksums.txt | sha256sum -c -

tar -xzf bb.tar.gz
sudo install bb /usr/local/bin/bb
```
Verify installation:
```bash
bb version
```
---

## 📦 What this repository contains

Each release includes:

- Verified multi-platform CLI binaries
- Release notes for each version
- SHA256 checksums for integrity verification

Supported platforms (Phase 0):

- Linux amd64
- macOS amd64
- macOS arm64
- Windows amd64

---

## 🔒 Source Code

The core source repository is maintained separately.

This repository exists as a **distribution surface only**, ensuring:

- controlled public access to binaries  
- consistent and predictable installation experience  
- separation of source code and release artifacts  

---

## ⚠️ Security

Always verify checksums before installing binaries.

```bash
sha256sum -c checksums.txt
```

Only install binaries that successfully pass verification.

---

## ⚙️ CLI Workflow (End-to-End)

BrikByteOS follows a deterministic release certification pipeline:
```
Doctor → Run → Score → Gate → Inspect
```

### 0️⃣ Doctor
Validate environment readiness:

```bash
bb doctor --workdir .
```
`bb doctor` checks:
- CLI environment integrity
- Required adapter availability
- Binary accessibility
- Workdir validity
- `.bb/` artifact directory readiness
- Read/write permissions
- Deterministic execution prerequisites

Use strict mode in CI:
```bash
bb doctor --workdir . --strict
```

### 1️⃣ Run

Collect deterministic release evidence:
```bash
bb run --all --workdir .
```
Or run a specific adapter:
```bash
bb run --adapter jest --workdir <path-to-root>
```
This produces a run under:
```
.bb/runs/<run-id>/
```

### 2️⃣ Score

Compute release risk:
```bash
bb score --manifest .bb/runs/<run-id>/manifest.json --workdir .
```
Outputs:
- score from 0–100
- risk factors
- release confidence signal

### 3️⃣ Gate

Evaluate release readiness:
```bash
bb gate --manifest .bb/runs/<run-id>/manifest.json --policy production --workdir <path-to-root>
```
Possible outcomes:
- APPROVED ✅
- REJECTED ❌
- CONDITIONAL ⚠️

### 4️⃣ Inspect
Understand what happened:
```bash
bb inspect --run-id <run-id> --workdir .
```
Use inspect to:
- view normalized outputs
- trace failures
- understand scoring decisions
- review evidence



### 5️⃣ Optional Visual Inspection

Generate an offline inspection bundle:
```bash
bb inspect --run-id <run-id> --workdir . --ui
bb inspect --run-id <run-id> --workdir . --html
```
Output:

- `.bb/runs/<run-id>/ui-data.json`
- `.bb/runs/<run-id>/report.html`

Then open the report:
```bash
open .bb/runs/<run-id>/report.html     # macOS
xdg-open .bb/runs/<run-id>/report.html # Linux
```

---

## 🔁 CI Integration Guide

BrikByteOS should run as a release confidence gate in CI.

### ⚙️ GitHub Actions (Recommended)
```yaml
name: brikbyteos-pipeline

on:
  push:
    branches: [main]
  pull_request:

jobs:
  release-certification:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install bb CLI
        run: |
          set -euo pipefail

          VERSION=v0.1.2
          BASE_URL="https://github.com/BrikByte-Studios/brikbyteos-cli-releases/releases/download/${VERSION}"

          curl -fL "${BASE_URL}/bb_linux_amd64.tar.gz" -o bb.tar.gz
          tar -xzf bb.tar.gz
          sudo mv bb /usr/local/bin/bb

      - name: Verify installation
        run: bb version

      # 🩺 Stage 0: Doctor (FAIL FAST)
      - name: Environment validation
        run: |
          set -euo pipefail
          bb doctor --workdir . --strict

      # 🚀 Stage 1: Run
      - name: Execute deterministic run
        run: |
          set -euo pipefail
          bb run --all --workdir .

      # 📊 Stage 2: Score
      - name: Compute score
        run: |
          set -euo pipefail
          RUN_ID=$(ls -t .bb/runs | head -n1)
          bb score --manifest .bb/runs/$RUN_ID/manifest.json --workdir .

      # 🚦 Stage 3: Gate
      - name: Evaluate gate
        run: |
          set -euo pipefail
          RUN_ID=$(ls -t .bb/runs | head -n1)
          bb gate --manifest .bb/runs/$RUN_ID/manifest.json --policy production --workdir .

      # 🔍 Stage 4: Inspect
      - name: Generate report
        run: |
          set -euo pipefail
          RUN_ID=$(ls -t .bb/runs | head -n1)
          bb inspect --run-id $RUN_ID --workdir . --html

      - name: Upload report
        uses: actions/upload-artifact@v4
        with:
          name: brikbyteos-report
          path: .bb/runs/*/report.html
```

---

## 🧪 Optional: Machine-Readable Diagnostics

Export doctor results for observability:
```yaml
- name: Export doctor report
  run: |
    bb doctor --workdir . --json > doctor-report.json

- name: Upload doctor report
  uses: actions/upload-artifact@v4
  with:
    name: doctor-report
    path: doctor-report.json
```
Use cases:
- CI debugging
- environment drift detection
- future dashboards
- audit trails


### 🚫 Anti-Pattern (Avoid This)
```yaml
# ❌ Incorrect pipeline
- run: bb run --all
```
Why this is dangerous:
- Missing tools → false failures
- Skipped adapters → incomplete evidence
- Silent drift → unreliable scoring
- Green pipeline → invalid confidence


### ✅ Best Practice

Always enforce:
```bash
bb doctor --strict
```
before any execution step.

---

## 🔄 Upgrading

To upgrade, repeat the installation steps with a newer version:
```bash
VERSION=v0.1.3
```

---

## 📌 Versioning

Releases follow semantic versioning within the Phase 0 scope:
```bash
v0.1.x
```
Each release is:
- deterministically built
- validated through automated pipelines
- published only after passing release certification checks

---

## 🎯 Purpose

This repository exists to:
- distribute **validated and reproducible release artifacts**
- support installation workflows (e.g., `bb-install`)
- enable adoption without exposing private implementation details
- provide a stable, trusted entry point for using BrikByteOS

---

## 🧠 Design Principle

BrikByteOS separates:
```
Source → Validation → Artifact → Public Distribution
```
This ensures:
- integrity of released binaries
- controlled implementation exposure
- reproducible release outputs
- trusted public installation

---

## ⚡ Notes
- This repository does not contain source code
- All binaries are produced from a controlled, validated build pipeline
- Do not rely on unverified binaries from other sources

---

## 📣 Feedback
If you encounter issues with installation or binaries:
- open an issue in the appropriate public repository (if available)
- or contact the maintainer directly

---

## 📜 License

Binary usage is governed by the license defined in the source repository.

Refer to the source project for full licensing terms.

---

## 🙏 Closing

BrikByteOS is built on a simple but powerful belief:

> **Releases should not rely on guesswork — they should be proven.**

Every binary in this repository represents a release that has been:

- deterministically built  
- systematically validated  
- objectively evaluated  

Whether you're exploring, integrating, or building on top of BrikByteOS:

> May your releases be **clear, confident, and correct.**

---

**Build with confidence. Ship with certainty.**