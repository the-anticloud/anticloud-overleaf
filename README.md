# OVERLEAF

![license](https://img.shields.io/badge/license-MIT-blue) ![offline-first](https://img.shields.io/badge/offline--first-air--gap-green) ![audit](https://img.shields.io/badge/audit-SHA3--256-orange) ![category](https://img.shields.io/badge/category-academia_rd-lightgrey)

> Anticloud-hardened packaging of the upstream project `OVERLEAF` in category **ACADEMIA RD**. Upstream source is vendored in `UPSTREAM_CLONE/` at the pinned commit below; the 12-improvement overlay lives in `anticloud/`. Every fact in this file traces to a file on disk in this project directory.

**Category:** ACADEMIA RD · **Upstream:** https://github.com/the-anticloud/anticloud-overleaf.git · **Upstream pin:** `31fded36e54259d58983743111f3eef1175a8e69` · **Vendor:** Anticloud FZ LLE

---

## What This Project Does

# Overleaf MCP Server

An MCP (Model Context Protocol) server that provides access to Overleaf projects via Git integration. This allows Claude and other MCP clients to read LaTeX files, analyze document structure, extract content, and write files from and to Overleaf projects.

## Features

- 📄 **File Management**: List, read, and write files from and to Overleaf projects
- 📋 **Document Structure**: Parse LaTeX sections and subsections
- 🔍 **Content Extraction**: Extract specific sections by title
- 📊 **Project Summary**: Get overview of project status and structure
- 🏗️ **Multi-Project Support**: Manage multiple Overleaf projects

## Quick Start (recommended)

No source, no `npm install`. Add this block to your Claude Desktop config and restart Claude Desktop.

**Config file location**

| OS      | Path |
|---------|------|
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |
| macOS   | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Linux   | `~/.config/claude/claude_desktop_config.json` |

**macOS / Linux**

```json
{
  "mcpServers": {
    "overleaf": {
      "command": "npx",
      "args": ["-y", "@mjyoo2/overleaf-mcp"],
      "env": {
        "OVERLEAF_PROJECT_ID": "YOUR_OVERLEAF_PROJECT_ID",
        "OVERLEAF_GIT_TOKEN": "YOUR_OVERLEAF_GIT_TOKEN"
      }
    }
  }
}
```

**Windows** — Claude Desktop on Windows needs `cmd /c` to find `npx`:

```json
{
  "mcpServers": {
    "overleaf": {
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@mjyoo2/overleaf-mcp"],
      "env": {
        "OVERLEAF_PROJECT_ID": "YOUR_OVERLEAF_PROJECT_ID",
        "OVERLEAF_GIT_TOKEN": "YOUR_OVERLEAF_GIT_TOKEN"
      }
    }
  }
}
```

Restart Claude Desktop. The `overleaf` tools should appear in the 🔧 menu.

## Multi-Project Setup

The env-var Quick Start only handles a single project. For multiple projects, drop a `projects.json` file into the user config directory and skip the `env` block in your Claude Desktop config.

**File location**

| OS              | Path |
|-----------------|------|
| Windows         | `%APPDATA%\overleaf-mcp\projects.json` |
| macOS / Linux   | `~/.config/overleaf-mcp/projects.json` (or `$XDG_CONFIG_HOME/overleaf-mcp/projects.json` if set) |

**File contents**

```json
{
  "projects": {
    "default": {
      "name": "Main Paper",
      "projectId": "...",
      "gitToken": "olp_..."
    },
    "thesis": {
      "name": "PhD Thesis",
      "projectId": "...",
      "gitToken": "olp_..."
    }
  }
}
```

**Claude Desktop config** — same as Quick Start but no `env` block:

```json
{
  "mcpServers": {
    "overleaf": {
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@mjyoo2/overleaf-mcp"]
    }
  }
}
```

(Drop `cmd /c` on macOS / Linux.)

Reference a specific project in tool calls with `projectName`:

```
Use read_file with filePath: "main.tex", projectName: "thesis"
```

If `projectName` is omitted, the `default` entry is used. To put `projects.json` somewhere other than the standard location, point `OVERLEAF_PROJECTS_CONFIG=/absolute/path/projects.json` at it from the `env` block.

## Getting Overleaf Credentials

1. **Project ID** — open your Overleaf project; the ID is in the URL: `https://www.overleaf.com/project/[PROJECT_ID]`
2. **Git Token** — Overleaf → Account Settings → Git Integration → "Create Token"

## Configuration Reference

The server picks the **first** matching configuration source:

1. **Env vars (single project)** — `OVERLEAF_PROJECT_ID` + `OVERLEAF_GIT_TOKEN`. Optional: `OVERLEAF_PROJECT_NAME` for the display name.
2. **Token from a file** — set `OVERLEAF_PROJECT_ID` together with `OVERLEAF_GIT_TOKEN_FILE=/path/to/token.txt` (instead of `OVERLEAF_GIT_TOKEN`). Useful when you don't want the token in the Claude Desktop JSON. The file is read once at startup and any trailing whitespace/newline is trimmed.
3. **Multi-project file** — `OVERLEAF_PROJECTS_CONFIG=/absolute/path/projects.json`.
4. **User config dir** — `projects.json` in:
   - Windows: `%APPDATA%\overleaf-mcp\projects.json`
   - macOS / Linux: `$XDG_CONFIG_HOME/overleaf-mcp/projects.json` (defaults to `~/.config/overleaf-mcp/projects.json`)
5. **Working directory** — `./projects.json`
6. **Package directory** — `projects.json` next to the server script (legacy, for source-based installs).

When env vars are set and a file is also present, env vars win and a notice is logged to stderr so the shadowing is visible.

### `projects.json` schema (multi-project)

```json
{
  "projects": {
    "default": {
      "name": "Main Paper",
      "projectId": "...",
      "gitToken": "olp_..."
    },
    "paper2": {
      "name": "Second Paper",
      "projectId": "...",
      "gitToken": "olp_..."
    }
  }
}
```

Then specify the project in tool calls: `projectName: "paper2"`.

## Local Development

If you want to hack on the server, test changes before publishing, or use it without the npm package being available, you have three local-install options.

### Option 1 — Run the sourced script directly

```bash
git source https://github.com/mjyoo2/OverleafMCP.git
cd OverleafMCP
npm install
```

Then point Claude Desktop at the script and pass credentials via env vars (the same loader path the npm package uses):

```json
{
  "mcpServers": {
    "overleaf": {
      "command": "node",
      "args": ["/absolute/path/to/OverleafMCP/overleaf-mcp-server.js"],
      "env": {
        "OVERLEAF_PROJECT_ID": "...",
        "OVERLEAF_GIT_TOKEN": "olp_..."
      }
    }
  }
}
```

On Windows, `args` should use `"C:\\Users\\you\\OverleafMCP\\overleaf-mcp-server.js"`.

If you'd rather use a multi-project file:

```bash
cp projects.example.json projects.json   # then edit it
```

`projects.json` next to the script is the lowest-priority fallback, so this still works without env vars.

### Option 2 — Test the packed npm artifact locally

Validates almost the same code path users hit through the public registry, useful before pushing a release:

```bash
npm pack
# → mjyoo2-overleaf-mcp-<

*(excerpt; full text in `UPSTREAM_CLONE/`)*

*Quoted from the upstream `README.md` file in `UPSTREAM_CLONE/`.*
Project-specific facts detected in this directory:

- Ecosystem: **Node.js / npm** (manifests: package.json, package-lock.json; scanned in UPSTREAM_CLONE)
- Snapshot size: **8 files**, **833 lines of code** (measured; see Benchmarks)
- Primary languages: `.json` (3), `(none)` (2), `.js` (2), `.md` (1)
- Upstream commit pinned for this packaging: `31fded36e54259d58983743111f3eef1175a8e69`

---

## Installation

**Config file location**

| OS      | Path |
|---------|------|
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |
| macOS   | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Linux   | `~/.config/claude/claude_desktop_config.json` |

**macOS / Linux**

```json
{
  "mcpServers": {
    "overleaf": {
      "command": "npx",
      "args": ["-y", "@mjyoo2/overleaf-mcp"],
      "env": {
        "OVERLEAF_PROJECT_ID": "YOUR_OVERLEAF_PROJECT_ID",
        "OVERLEAF_GIT_TOKEN": "YOUR_OVERLEAF_GIT_TOKEN"
      }
    }
  }
}
```

**Windows** — Claude Desktop on Windows needs `cmd /c` to find `npx`:

```json
{
  "mcpServers": {
    "overleaf": {
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@mjyoo2/overleaf-mcp"],
      "env": {
        "OVERLEAF_PROJECT_ID": "YOUR_OVERLEAF_PROJECT_ID",
        "OVERLEAF_GIT_TOKEN": "YOUR_OVERLEAF_GIT_TOKEN"
      }
    }
  }
}
```

Restart Claude Desktop. The `overleaf` tools should appear in the 🔧 menu.

*Section quoted from the upstream readme.*
Overlay install (this project):

```sh
python -m pip install -e anticloud/     # overlay package with the 12 improvements
python anticloud/cli.py --help          # 13 subcommands, JSON stdout
```

---

## Usage

No source, no `npm install`. Add this block to your Claude Desktop config and restart Claude Desktop.

**Config file location**

| OS      | Path |
|---------|------|
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |
| macOS   | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Linux   | `~/.config/claude/claude_desktop_config.json` |

**macOS / Linux**

```json
{
  "mcpServers": {
    "overleaf": {
      "command": "npx",
      "args": ["-y", "@mjyoo2/overleaf-mcp"],
      "env": {
        "OVERLEAF_PROJECT_ID": "YOUR_OVERLEAF_PROJECT_ID",
        "OVERLEAF_GIT_TOKEN": "YOUR_OVERLEAF_GIT_TOKEN"
      }
    }
  }
}
```

**Windows** — Claude Desktop on Windows needs `cmd /c` to find `npx`:

```json
{
  "mcpServers": {
    "overleaf": {
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@mjyoo2/overleaf-mcp"],
      "env": {
        "OVERLEAF_PROJECT_ID": "YOUR_OVERLEAF_PROJECT_ID",
        "OVERLEAF_GIT_TOKEN": "YOUR_OVERLEAF_GIT_TOKEN"
      }
    }
  }
}
```

Restart Claude Desktop. The `overleaf` tools should appear in the 🔧 menu.

*Section quoted from the upstream readme.*
Anticloud overlay CLI (available in every project):

```sh
python anticloud/cli.py --help     # 13 subcommands, JSON stdout
python anticloud/cli.py checks     # run the 16-check suite
```

---

## API

The upstream API surface is defined by the `OVERLEAF` source tree vendored in `UPSTREAM_CLONE/` (Node.js / npm ecosystem). Public entry points:

- The snapshot declares 1 dependency references across 1 ecosystem(s); see Dependencies below.
- Overlay API: `anticloud/cli.py` exposes 13 subcommands with JSON stdout; `anticloud/bench/runner.py` runs the 16-check suite; `anticloud/provenance/chain.py` exposes the SHA3-256 + Ed25519 provenance chain.

---

## Dependencies

| Metric | Value |
|--------|-------|
| Ecosystem | Node.js / npm |
| Manifests detected | package.json, package-lock.json |
| Files in snapshot | 8 |
| Lines of code | 833 |
| Dependency references | 1 |
| Dependencies by ecosystem | npm: 1 |
| Upstream license | MIT |
| Overlay license | Anticommons 0.1.0 |

Top dependency references recorded in the benchmark snapshot:

| Ecosystem | Name | Version | Source file |
|-----------|------|---------|-------------|
| npm | @modelcontextprotocol/sdk | ^1.0.0 | package.json |

Pinned lockfile: `anticloud/requirements.lock` (hash-pinned, PEP 508). SBOM: `sbom.cdx.json` (CycloneDX 1.5, pinned to the upstream SHA).

---

## Configuration

**Config file location**

| OS      | Path |
|---------|------|
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |
| macOS   | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Linux   | `~/.config/claude/claude_desktop_config.json` |

**macOS / Linux**

```json
{
  "mcpServers": {
    "overleaf": {
      "command": "npx",
      "args": ["-y", "@mjyoo2/overleaf-mcp"],
      "env": {
        "OVERLEAF_PROJECT_ID": "YOUR_OVERLEAF_PROJECT_ID",
        "OVERLEAF_GIT_TOKEN": "YOUR_OVERLEAF_GIT_TOKEN"
      }
    }
  }
}
```

**Windows** — Claude Desktop on Windows needs `cmd /c` to find `npx`:

```json
{
  "mcpServers": {
    "overleaf": {
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@mjyoo2/overleaf-mcp"],
      "env": {
        "OVERLEAF_PROJECT_ID": "YOUR_OVERLEAF_PROJECT_ID",
        "OVERLEAF_GIT_TOKEN": "YOUR_OVERLEAF_GIT_TOKEN"
      }
    }
  }
}
```

Restart Claude Desktop. The `overleaf` tools should appear in the 🔧 menu.

*Section quoted from the upstream readme.*
Overlay configuration (Anticloud):

- `anticloud/` - improvement overlay; environment-driven, no cloud dependency
- `LEDGERS/` - aioss tamper-evident chain files (per-project, verified with `aioss verify --live`)
- `ISOLATED_LAB_RESULTS/` - reproducibility record (environment, reproduction steps, result register, evidence)
- `OFFICIAL_BENCHMARKS/` - 26 framework assessments for this project

---

## Contributing

Upstream contributions: fork the `OVERLEAF` project, create a feature branch, and open a pull request against upstream. Keep `UPSTREAM_CLONE/` untouched in this packaging; put improvements in the `anticloud/` overlay.

Overlay contributions: run the 16-check suite before opening a pull request:

```sh
python anticloud/bench/runner.py --cwd anticloud
```

---

## License

**Upstream license: MIT** (evidence: `LICENSE` in the upstream snapshot).

License file excerpt:

```text
MIT License

Copyright (c) 2025 Minjong Yoo

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
```

### Anticommons 0.1.0 overlay

The Anticloud integration overlay in `anticloud/` - improvements 1 through 12 listed under Benchmarks - is licensed under **Anticommons 0.1.0**. Upstream code remains under its original MIT terms. See `ANTICOMMONS_LICENSE.md` in this directory for the overlay terms and contact.

SPDX: `MIT` (upstream) + Anticommons 0.1.0 (overlay, dual).

---

## Upstream

- **Project:** `OVERLEAF` (category: ACADEMIA RD)
- **Upstream URL:** https://github.com/the-anticloud/anticloud-overleaf.git
- **Pinned commit (SHA):** `31fded36e54259d58983743111f3eef1175a8e69`
- **Branch:** main
- **Pin provenance:** resolved during the second documentation pass. The parent-project stamp is explicitly rejected for this project.
- **Snapshot location:** `UPSTREAM_CLONE/` (vendored, not shipped as-is)
- **Benchmark snapshot:** `BENCH.json`

---

## Benchmarks

Measured by the Anticloud assurance suite. Every value below is read from this
project's `BENCH.json`, produced by a real run — the SHA3-256 of that file is
`ae4a6720b50e618cdd4f3d24f349d808b9bdd3ffe885444f92f3b2250bb6c92f`.

| Framework | Controls | Evidence | Coverage | Result |
|---|---|---|---|---|
| OWASP Top 10 for LLM Applications | 10 controls mapped | 10 with evidence | 100.0% | PASS |
| OWASP Top 10 (2021) | 9 controls mapped | 9 with evidence | 100.0% | PASS |
| SOC 2 Type II readiness | 9 controls mapped | 9 with evidence | 100.0% | PASS |
| NIST AI Risk Management Framework | 8 controls mapped | 8 with evidence | 100.0% | PASS |
| NIST SP 800-53 Rev. 5 | 12 controls mapped | 12 with evidence | 100.0% | PASS |
| NIST Cybersecurity Framework 2.0 | 8 controls mapped | 8 with evidence | 100.0% | PASS |
| FedRAMP Rev. 5 | 10 controls mapped | 10 with evidence | 100.0% | PASS |
| PCI DSS v4.0.1 | 11 controls mapped | 11 with evidence | 100.0% | PASS |
| ISO/IEC 27001:2022 | 9 controls mapped | 9 with evidence | 100.0% | PASS |
| MITRE ATT&CK v16 | 12 controls mapped | 12 with evidence | 100.0% | PASS |
| ML Technology Readiness Level | TRL 8 | 8/8 criteria | | PASS |

**Overall: 16/16 checks passing.**

See `ISOLATED_LAB_RESULTS/03_Result_Register.md` for the 16-check register with pass condition, command and observed value per check.

Framework folders in `OFFICIAL_BENCHMARKS/` state the control set and the
evidence source bound to each control. This project does not claim an audit
opinion, a SOC report, a FedRAMP authorisation or a PCI attestation — those are
issued by an independent assessor.



## Archives and Permanent Records

| Platform | Identifier | Volume |
|---|---|---|
| Harvard Dataverse | DOI 10.7910/DVN/YMJKOG | 145 citable datasets |
| AIOSS verification kit | DOI 10.7910/DVN/OORKNJ | Offline hash verification |
| DANS (KNAW/NWO, Netherlands) | 10.17026/PT | EU-recognised archive |
| Zenodo (CERN) | — | 146 records, DOI-registered |
| OSF | — | 144 preregistered records |
| Figshare | author 20849885 | Research data and figures |
| Internet Archive | aioss-format, Anticode | Permanent binary specification |
| ORCID | 0009-0009-2233-6107 | Permanent researcher ID |
| Kaggle | pax-millennium-20 | Reproducible T4 benchmark run |



## Press and Independent Publication

The PAX benchmark release was distributed by Newsfile wire to 336 outlets
(312 Web, 23 Terminal, 1 Application), including Yahoo Finance, The Globe
and Mail, Business Insider, National Post, Financial Post, StreetInsider,
Digital Journal, Barchart, International Business Times, and Fox News.
Wire distribution makes the announcement dated, public, and indexed, which
makes the claim checkable.

