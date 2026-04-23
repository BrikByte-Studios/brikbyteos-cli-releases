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
Run → Score → Gate → Inspect
```

### 1️⃣ Help

Explore available commands:
```bash
bb --help
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