# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A bash CLI script that verifies the SHA256 checksum of a Claude Code binary against the official release manifest hosted on Google Cloud Storage.

## Usage

```bash
./claude-verify                        # verify claude binary found in PATH
./claude-verify --path /path/to/claude # verify a specific binary
./claude-verify --os linux-x64         # override platform detection
```

## Architecture

Single-file bash script (`claude-verify`) with no dependencies beyond standard Unix tools.

**Flow:**
1. Resolve binary path (default: `which claude`)
2. Extract version via `claude --version` (parses `X.Y.Z` from output)
3. Detect platform as `{os}-{arch}[-musl]` using `uname`; musl detection checks `ldd --version` and `/lib/libc.musl-*`
4. Fetch `https://storage.googleapis.com/claude-code-dist-86c565f3-f756-42ad-8dfa-d59b1c096819/claude-code-releases/$VERSION/manifest.json`
5. Extract `.platforms[$PLATFORM].checksum` (SHA256 hex) via `jq` or `python3` fallback
6. Compute SHA256 of local binary via `sha256sum` or `shasum -a 256` fallback
7. Compare and exit 0 (match) or 1 (mismatch)

**Manifest platform keys:** `darwin-arm64`, `darwin-x64`, `linux-arm64`, `linux-x64`, `linux-arm64-musl`, `linux-x64-musl`, `win32-arm64`, `win32-x64`

**Required tools:** `curl` or `wget` for fetching; `jq` or `python3` for JSON parsing; `sha256sum` or `shasum` for hashing.
