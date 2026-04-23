# BrikByteOS CLI Releases

This repository provides the **official public binary distribution** for the BrikByteOS CLI.

BrikByteOS is a deterministic release certification system that answers a critical question:

> "Is this release safe, compliant, and ready to ship?"

---

## 📦 What this repository contains

- Verified multi-platform CLI binaries
- Release notes for each version
- SHA256 checksums for integrity verification

---

## 🔒 Source Code

The core source repository is maintained separately.

This repository exists as a **distribution surface only**, ensuring:

- controlled public access to binaries
- consistent installation experience
- separation of source and release artifacts

---

## ⚠️ Security

Always verify checksums before installing binaries:

```bash
sha256sum -c checksums.txt
```

---

## 🚀 Installation

Example (Linux amd64):
```bash
VERSION=v0.1.0
BASE_URL="https://github.com/BrikByte-Studios/brikbyteos-cli-releases/releases/download/${VERSION}"

curl -fL "${BASE_URL}/bb_linux_amd64.tar.gz" -o /tmp/bb.tar.gz
tar -xzf /tmp/bb.tar.gz -C /tmp
sudo install /tmp/bb /usr/local/bin/bb
```

---

## 🎯 Purpose

This repository exists to:
- distribute **validated release artifacts**
- support installation workflows (`bb-install`)
- enable adoption without exposing private implementation details