# senseboost CLI

Free local CLI for analyzing Qlik Cloud apps. Runs the SenseBoost analyzer on your machine and writes the report to disk — no data leaves your computer.

- **Free** for personal and commercial use.
- **Local-only**: no telemetry, no upload, no account required.
- **Self-contained reports**: JSON and HTML, ready to share or check into CI.

For team workflows, history, governance, and dashboards see the full [SenseBoost SaaS](https://senseboost.net).

---

## Install

Download the latest release archive for your platform from [Releases](https://github.com/jonasandre/senseboost-cli/releases) and place the binary on your `PATH`.

### macOS

```bash
# Apple Silicon
curl -L https://github.com/jonasandre/senseboost-cli/releases/latest/download/senseboost_Darwin_arm64.tar.gz | tar xz
# Intel
curl -L https://github.com/jonasandre/senseboost-cli/releases/latest/download/senseboost_Darwin_x86_64.tar.gz | tar xz

sudo mv senseboost /usr/local/bin/
xattr -d com.apple.quarantine /usr/local/bin/senseboost   # if Gatekeeper blocks the unsigned binary
```

### Linux

```bash
curl -L https://github.com/jonasandre/senseboost-cli/releases/latest/download/senseboost_Linux_x86_64.tar.gz | tar xz
sudo mv senseboost /usr/local/bin/
```

### Windows

Download `senseboost_Windows_x86_64.zip` from the releases page, extract `senseboost.exe`, and add its folder to your `PATH`.

### Verify

```bash
senseboost version
```

Each release publishes `checksums.txt` alongside the binaries. Verify before installing in restricted environments:

```bash
sha256sum -c checksums.txt
```

---

## Quickstart

1. Get a Qlik Cloud API key with permission to read the app you want to analyze.
2. Create a context:

   ```bash
   senseboost context create \
     --name prod \
     --base-url https://<your-tenant>.qlikcloud.com \
     --api-key <your-api-key>
   ```

   The API key is stored in your OS keychain (macOS Keychain, Windows Credential Manager, Linux Secret Service). It is never written to disk in plaintext.

3. Analyze an app:

   ```bash
   senseboost analyze \
     --app-id <qlik-app-id> \
     --out ./analysis.json \
     --html ./analysis.html
   ```

The CLI prints progress to stdout with timestamped `INFO` / `WARN` log lines and writes the JSON envelope plus an optional HTML report.

---

## Commands

```text
senseboost context create  --name <n> --base-url <url> [--api-key <key>]
senseboost context list
senseboost context use     <name>
senseboost context delete  <name>

senseboost analyze \
  [--context <name>] \
  --app-id <id> \
  --out <path> \
  [--html <path>] \
  [--timeout <duration>] \
  [--force] \
  [--app-name <n>] [--space-name <n>]

senseboost completion [shell]

senseboost version
```

Full flag reference: `senseboost <command> --help`.

---

## Shell autocomplete

`senseboost completion` tries to detect your shell automatically.

Generate a completion script and load it:

```bash
# macOS default (usually zsh)
source <(senseboost completion)

# bash or zsh explicitly
source <(senseboost completion zsh)
source <(senseboost completion bash)

# fish (different shell syntax)
senseboost completion | source

# PowerShell
senseboost completion | Out-String | Invoke-Expression
```

The CLI also suggests saved context names for `senseboost context use`, `senseboost context delete`, and `senseboost analyze --context`.

If detection fails, run `echo $SHELL` and then pass the shell explicitly, for example `senseboost completion zsh`.

For persistent setup, save the generated script in your shell's completion directory or source it from your shell startup file.

---

## CI / non-interactive use

When `SENSEBOOST_QLIK_BASE_URL` is set, the CLI ignores stored contexts entirely and reads credentials from environment variables:

```bash
export SENSEBOOST_QLIK_BASE_URL=https://<tenant>.qlikcloud.com
export SENSEBOOST_QLIK_API_KEY=...
senseboost analyze --app-id <id> --out analysis.json
```

This is the recommended path for GitHub Actions, GitLab CI, and other CI systems where the OS keychain is not available.

---

## Output format

`--out` writes a JSON envelope:

```json
{
  "schemaVersion": 1,
  "generatedAt": "2026-05-11T18:08:00Z",
  "source": {
    "tenantUrl": "https://tenant.qlikcloud.com",
    "appId": "...",
    "appName": "...",
    "spaceName": "..."
  },
  "analysis": {
    "summary": {
      /* counts, byte sizes, app file size, broken items */
    },
    "fields": [
      /* ... */
    ],
    "variables": [
      /* ... */
    ],
    "dimensions": [
      /* ... */
    ],
    "measures": [
      /* ... */
    ],
    "objects": [
      /* ... */
    ]
  }
}
```

`schemaVersion` follows semantic versioning. Additive fields are backward-compatible; breaking shape changes bump the major version.

`--html` writes a fully self-contained HTML report (no external assets, no network requests on open).

Recent output additions:

- Analysis items include explicit `used: "yes" | "no"` values alongside the existing usage counts.
- Dimensions and measures include explicit `broken: "yes" | "no"` values alongside the existing boolean fields.

---

## Privacy & security

- Reports stay on your machine. The CLI **never** sends data to SenseBoost.
- There is no telemetry, no analytics, no opt-in pings.
- Output files may contain **variable definitions and measure expressions** that encode business logic. Treat them as sensitive.
- Output files are written with `0600` permissions on Unix.
- API keys are stored only in the OS-native credential vault. If the vault is unavailable, the CLI refuses to fall back to a plaintext file and asks you to use environment variables instead.

---

## Comparison with SenseBoost SaaS

| Capability                          | Free CLI                       | SaaS              |
| ----------------------------------- | ------------------------------ | ----------------- |
| Run analysis                        | Yes (local)                    | Yes (centralized) |
| History & comparisons               | —                              | Yes               |
| Web UI, filters, dashboards         | —                              | Yes               |
| Multi-user, roles, quotas, auditing | —                              | Yes               |
| Async jobs, scheduling, automation  | —                              | Yes               |
| Shared connections across team      | —                              | Yes               |
| Output                              | Single local JSON / HTML       | Postgres + web    |
| Auth                                | API key                        | API key + OAuth   |
| Cost                                | Free                           | Paid              |

The CLI is meant for one-off, local analysis and CI scripting. The SaaS is for ongoing governance and team collaboration.

---

## Roadmap

- OAuth client-credentials (M2M) auth — currently API key only.
- Optional Homebrew tap.
- Smaller Docker image for CI use.
- Optional `--redact-expressions` flag for reports that must omit business logic.

---

## Support

- Issues, bug reports, feature requests: open an issue on this repository.
- Product questions about the SaaS: see [senseboost.net](https://senseboost.net).

This repository contains only release artifacts. The CLI source code is part of the SenseBoost monorepo and is not published here.

---

## License

The senseboost CLI is free to use under the terms of the accompanying [LICENSE](LICENSE.md). The license permits personal and commercial use of the binaries; it does **not** permit redistribution, modification, or reverse-engineering. Please read the full license before installation.

© 2026 SenseBoost. All rights reserved.
