# BrikByteOS CLI Releases

This repository provides the **official public binary distribution** for the BrikByteOS CLI.

BrikByteOS is a deterministic release certification system that answers a critical question:

> **"Is this release safe, compliant, and ready to ship?"**

---

## 📦 What this repository contains

Each release includes:

- Verified multi-platform CLI binaries
- Release notes for each version
- SHA256 checksums for integrity verification

Supported platforms (Phase 0):

- Linux (amd64)
- macOS (amd64, arm64)
- Windows (amd64)

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

## 🚀 Installation
### Linux (amd64)
```bash
VERSION=v0.1.0
BASE_URL="https://github.com/BrikByte-Studios/brikbyteos-cli-releases/releases/download/${VERSION}"

curl -fL "${BASE_URL}/bb_linux_amd64.tar.gz" -o /tmp/bb.tar.gz
curl -fL "${BASE_URL}/checksums.txt" -o /tmp/checksums.txt

cd /tmp
grep ' bb_linux_amd64.tar.gz$' checksums.txt | sha256sum -c -

tar -xzf bb.tar.gz
sudo install bb /usr/local/bin/bb
```

---

## ⚙️ Verify installation
```bash
bb version
```

---

## ⚙️ CLI Workflow (End-to-End)

BrikByteOS is designed as a deterministic pipeline:
```
Doctor → Run → Score → Gate → Inspect
```

### 0️⃣ Doctor (Environment Readiness & Diagnostics)
Check runtime readiness

```bash
bb doctor
```
#### 📋 What bb `doctor` validates
- ✅ CLI environment integrity
- ✅ Required adapters availability:
    - Jest
    - Playwright
    - k6
    - Trivy
- ✅ Binary accessibility (PATH resolution)
- ✅ Workdir structure validity
- ✅ `.bb/` artifact directory readiness
- ✅ Permissions (read/write/execute)
- ✅ Deterministic execution prerequisites

### 1️⃣ Help

Explore available commands:
```bash
bb help
```

### 2️⃣ Run (Collect Evidence)

Execute a deterministic run:
```bash
bb run --all --workdir <path-to-root>
```
Or run a specific adapter:
```bash
bb run --adapter jest --workdir <path-to-root>
```
This produces a run under:
```
.bb/runs/<run-id>/
```

### 3️⃣ Score (Quantify Risk)

Compute a release score:
```bash
bb score --manifest .bb/runs/<run-id>/manifest.json --workdir <path-to-root>
```
Outputs:
- score (0–100)
- risk factors

### 4️⃣ Gate (Decision)

Evaluate release readiness:
```bash
bb gate --manifest .bb/runs/<run-id>/manifest.json --policy production --workdir <path-to-root>
```
Possible outcomes:
- APPROVED ✅
- REJECTED ❌
- CONDITIONAL ⚠️


### 5️⃣ Inspect (Understand Results)

Inspect run details:
```bash
bb inspect --run-id <run-id> --workdir <path-to-root>
```
This allows you to:
- view normalized outputs
- trace failures
- understand scoring decisions

### 6️⃣ Optional: Visual Inspection (Offline Inspection Bundle)

Render a human-friendly report:
```
bb inspect --run-id <run-id> --workdir <path-to-root> --ui
bb inspect --run-id <run-id> --workdir <path-to-root> --html
```
Output:

- `.bb/runs/<run-id>/ui-data.json`
- `.bb/runs/<run-id>/report.html`

Then open in browser:
```bash
open .bb/runs/<run-id>/report.html     # macOS
xdg-open .bb/runs/<run-id>/report.html # Linux
```
---

## 🔁 CI Integration (Preflight with `bb doctor`)

BrikByteOS is built on **deterministic execution**.

To guarantee reliable and reproducible results in automated environments, integrate `bb doctor` as a **mandatory preflight gate** in your CI/CD pipeline.

---

### 🧠 CI Pipeline Model

BrikByteOS pipelines follow a strict execution order:
```
Doctor → Run → Score → Gate → Inspect
```


| Stage   | Purpose |
|--------|--------|
| Doctor | Validate environment readiness |
| Run    | Collect deterministic evidence |
| Score  | Quantify release risk |
| Gate   | Enforce policy decision |
| Inspect| Explain and visualize results |

---

## ⚙️ GitHub Actions (Recommended)

### 🔹 Minimal Production Pipeline

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

### ⚠️ Why `bb doctor --strict`?
```bash
bb doctor --workdir . --strict
```
This ensures the pipeline **fails immediately** if:
- Required adapters are missing (Jest, Playwright, k6, Trivy)
- Workdir is invalid
- Permissions are incorrect
- `.bb/` artifact directory is not writable
- Environment is non-deterministic

> A pipeline that runs in a broken environment produces false confidence.

### 🧪 Optional: Machine-Readable Diagnostics

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
- Debugging CI failures
- Detecting environment drift
- Feeding dashboards (future phases)


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

## 🔚 Final CI Execution Flow
```bash
bb doctor --strict
bb run --all
bb score
bb gate
bb inspect
```

---

## 🔄 Upgrading

To upgrade, repeat the installation steps with a newer version:
```bash
VERSION=v0.1.1
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
Source (private) → Validation → Artifact → Public Distribution
```
This ensures:
- integrity of released binaries
- controlled exposure of implementation
- reproducibility of release outputs

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