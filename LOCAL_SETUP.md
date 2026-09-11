# Local setup (fork notes)

Personal setup notes for running this fork on macOS (Apple Silicon).

## Why a prebuilt binary

Upstream needs Go 1.26+. Rather than install a toolchain, the local setup uses
the upstream release binary, which is dropped into `bin/` (gitignored):

```sh
gh release download v7.2.158 --repo router-for-me/CLIProxyAPI \
  --pattern 'CLIProxyAPI_*_darwin_aarch64.tar.gz' --dir bin
tar -C bin -xzf bin/CLIProxyAPI_*_darwin_aarch64.tar.gz
xattr -d com.apple.quarantine bin/cli-proxy-api   # clear Gatekeeper flag
```

To build from source instead, install Go 1.26+ and run `go build ./cmd/server`.

## Config

`config.yaml` is gitignored and holds the local API key + management secret.
It binds loopback only:

```yaml
host: "127.0.0.1"
port: 8317
auth-dir: "~/.cli-proxy-api"
api-keys:
  - "<local key>"
remote-management:
  allow-remote: false
  secret-key: "<mgmt secret>"
```

## Run

```sh
./bin/cli-proxy-api --config config.yaml
```

- OpenAI-compatible API: `http://127.0.0.1:8317/v1/...`
- Management UI: `http://127.0.0.1:8317/management.html`

## Accounts

OAuth tokens live in `~/.cli-proxy-api/*.json`, one file per account. Add more
with `./bin/cli-proxy-api --claude-login` (also `--codex-login`,
`--antigravity-login`, `--xai-login`, `--kimi-login`). Multiple accounts for the
same provider are round-robined automatically.

## Staying current with upstream

```sh
git fetch upstream && git merge upstream/main
```
