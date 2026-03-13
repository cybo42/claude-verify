# claude-verify

A bash script that verifies the integrity of a [Claude Code](https://claude.ai/code) binary by comparing its SHA256 checksum against Anthropic's official release manifest.

## Why

Verifying a binary's checksum after download confirms it hasn't been corrupted in transit or tampered with. This is especially useful when installing Claude Code in CI/CD pipelines, on shared systems, or after manual downloads.

## Requirements

| Purpose | Tools checked (in order) |
|---|---|
| Fetch manifest | `curl`, `wget` |
| Parse JSON | `jq`, `python3` |
| Compute checksum | `sha256sum`, `shasum` |

All of these are available by default on macOS and most Linux distributions.

## Installation

```bash
curl -O https://raw.githubusercontent.com/cybo42/claude-verify/main/claude-verify
chmod +x claude-verify
```

Or clone the repo:

```bash
git clone https://github.com/cybo42/claude-verify.git
cd claude-verify
```

## Usage

```bash
# Verify the claude binary found in PATH
./claude-verify

# Verify a binary at a specific path
./claude-verify --path /usr/local/bin/claude

# Override platform detection (useful in containers or cross-checks)
./claude-verify --os linux-x64-musl

# Silent mode — no output on success, useful for scripts and CI
./claude-verify --silent

# Combine options
./claude-verify --path ./claude --os darwin-arm64 --silent
```

### Options

| Flag | Short | Description |
|---|---|---|
| `--path PATH` | `-p` | Path to the binary to verify (default: `which claude`) |
| `--os PLATFORM` | `-o` | Override platform key instead of auto-detecting |
| `--silent` | `-s` | Suppress all output on success; errors still go to stderr |
| `--help` | `-h` | Show usage information |

### Supported platforms

`darwin-arm64` · `darwin-x64` · `linux-arm64` · `linux-x64` · `linux-arm64-musl` · `linux-x64-musl` · `win32-arm64` · `win32-x64`

musl variants are for Alpine Linux and other musl-based distributions. The script detects musl automatically by inspecting `ldd --version` and `/lib/libc.musl-*`.

## Example output

```
Binary:   /Users/you/.local/bin/claude
Version:  2.1.58
Platform: darwin-arm64
Manifest: https://storage.googleapis.com/.../2.1.58/manifest.json

Expected: 182c20c6080d042e4e08a6b2a2ce8258c1a50e53c01d36ddf20a82b4693395ea
Actual:   182c20c6080d042e4e08a6b2a2ce8258c1a50e53c01d36ddf20a82b4693395ea

OK  Checksum verified successfully.
```

A checksum mismatch prints an error to stderr and exits with code `1`. In silent mode (`-s`), all normal output is suppressed — only errors are printed.

## How it works

1. Resolves the binary path and runs `--version` to extract the version number.
2. Auto-detects the platform from `uname -s` and `uname -m` (with musl detection on Linux).
3. Fetches the release manifest from Anthropic's distribution bucket at `https://storage.googleapis.com/claude-code-dist-86c565f3-f756-42ad-8dfa-d59b1c096819/claude-code-releases/$VERSION/manifest.json`.
4. Extracts the expected SHA256 checksum for the detected platform.
5. Computes the SHA256 of the local binary and compares.

## License

MIT
