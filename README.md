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