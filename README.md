# 🔍 Praven

> **Explain any git diff in plain English. Get a PR description, commit message, and risk flags from your terminal.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)

---

## Why Praven?

You just finished a chunk of work. Now you need a commit message, a PR description, and to remember whether you touched anything risky. Praven reads your diff and gives you all three in seconds — no context-switching, no copy-pasting into a browser. It runs entirely from your terminal and works with local models for full privacy.

## Demo

<!-- Replace with a terminal recording (e.g., asciinema or VHS) -->
<!--
![Praven demo](docs/demo.gif)
-->

**Example output:**

```
📝 Summary
──────────────────────────────────────────────────
Added rate limiting to the /api/login endpoint using a sliding-window
counter backed by Redis. The limit is 5 attempts per minute per IP.

💬 Commit Message
──────────────────────────────────────────────────
feat(auth): add rate limiting to login endpoint

📋 PR Description
──────────────────────────────────────────────────
## What
Added rate limiting to the login API endpoint.

## Why
Prevent brute-force attacks on user accounts.

## How
- Added Redis-backed sliding window counter
- Configured to 5 attempts/minute/IP
- Returns 429 with Retry-After header when exceeded

Risk Flags
────────────────────────────────────────────────────────────
  ⚠ [WARN] Auth/login/session code changed but no test files were modified.
  ℹ [INFO] Dependency files changed: requirements.txt.
```

## Installation

### With pipx (recommended)

```bash
pipx install praven
```

### From source

```bash
git clone https://github.com/praven-dev/praven.git
cd praven
pip install .
```

### For development

```bash
pip install -e ".[dev]"
```

## Quick Start

1. **Install Praven** (see above)

2. **Set up a provider** — pick one:

   ```bash
   # Option A: Cloud (Anthropic)
   export ANTHROPIC_API_KEY="your-key"

   # Option B: Cloud (OpenAI)
   export OPENAI_API_KEY="your-key"

   # Option C: Local (Ollama) — fully private
   ollama pull llama3
   ollama serve
   ```

3. **Run Praven:**

   ```bash
   # In any git repo with staged changes:
   git add -A
   praven

   # Or with Ollama:
   praven --provider ollama
   ```

## Commands & Flags

| Command / Flag | Description |
|---|---|
| `praven` | Explain staged changes (default) |
| `praven --unstaged` | Explain unstaged working-tree changes |
| `praven --last N` | Explain the last N commits |
| `praven --branch main` | Compare current branch to a base branch |
| `praven --style STYLE` | Output style (see below) |
| `praven --provider NAME` | LLM provider: `anthropic`, `openai`, `ollama` |
| `praven --model NAME` | Override the model name |
| `praven --copy` | Copy the result to the clipboard |
| `praven --json` | Machine-readable JSON output |
| `praven --yes` / `-y` | Skip the privacy consent prompt |
| `praven --version` | Show version |
| `praven init` | Create `praven.toml` and `.pravenignore` in the repo |

## Output Styles

| Style | Flag | What you get |
|---|---|---|
| **Full** (default) | `--style full` | Summary + commit message + PR description + risk flags |
| **Commit** | `--style commit` | Conventional Commits message only |
| **PR** | `--style pr` | Pull request description in markdown |
| **Changelog** | `--style changelog` | Changelog entry grouped by type |
| **Slack** | `--style slack` | Short, non-technical team update |

## Configuration

Praven reads configuration from multiple sources in this priority order:

1. CLI flags (highest)
2. Environment variables (`PRAVEN_PROVIDER`, `PRAVEN_MODEL`, `PRAVEN_STYLE`)
3. `praven.toml` in the repo root
4. `~/.config/praven/config.toml`
5. Built-in defaults

Run `praven init` to create a starter `praven.toml`:

```toml
provider = "anthropic"     # anthropic | openai | ollama
model = ""                 # leave empty for the provider default
style = "full"
max_file_chars = 4000
max_total_chars = 30000
```

### .pravenignore

Create a `.pravenignore` file in your repo root to exclude files from analysis. Uses gitignore-style glob patterns.

**Always excluded by default:** lock files (`*.lock`, `package-lock.json`), minified files (`*.min.js`), environment files (`.env`), key/certificate files (`*.pem`, `*.key`), images, and binary files.

```gitignore
# .pravenignore
*.generated.ts
vendor/
dist/
*.snap
```

## Providers

### Anthropic (default)

```bash
export ANTHROPIC_API_KEY="your-key"
praven
```

Default model: `claude-sonnet-4-20250514`

### OpenAI

```bash
export OPENAI_API_KEY="your-key"
praven --provider openai
```

Default model: `gpt-4o`

### Ollama (local — fully private)

Nothing leaves your machine. Install [Ollama](https://ollama.ai), pull a model, and go:

```bash
ollama pull llama3
ollama serve
praven --provider ollama
```

Default model: `llama3`. Override with `--model codellama` or any model you've pulled.

The Ollama URL defaults to `http://localhost:11434` and can be changed in `praven.toml` or via the `PRAVEN_OLLAMA_URL` environment variable.

## Privacy

- **Cloud providers:** Before the first API call, Praven shows a one-time notice that your diff will be sent to the provider. Use `--yes` to skip it, or set `consent_given = true` in your user config.
- **Ignored files** are never sent to any provider.
- **Detected secrets** are flagged but never printed or sent.
- **API keys** are never logged or printed.
- **For full privacy**, use `--provider ollama` — everything stays on your machine.

## Risk Flags

Praven scans your diff for common risk patterns (no AI, pure rules):

| Level | Rule |
|---|---|
| ⚠ WARN | Auth/login/session code changed with no test changes |
| ⚠ WARN | Database migration or schema files changed |
| ⚠ WARN | Large diff (500+ changed lines) |
| ℹ INFO | Dependency files changed |
| ℹ INFO | CI/CD or deployment config changed |
| ℹ INFO | Source files changed with no test files |
| 🚨 HIGH | Possible secret or credential in added lines |

## Roadmap

- [ ] Git hook mode (`praven hook install` for pre-commit / prepare-commit-msg)
- [ ] GitHub Action for automated PR descriptions
- [ ] More providers (Gemini, Mistral, Groq)
- [ ] Custom prompt templates
- [ ] Interactive mode (approve/edit before committing)
- [ ] Diff filtering by directory or file type
- [ ] Token usage tracking and cost estimates

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for setup instructions, how to run tests, and how to add a new provider (a great first contribution!).

## License

[MIT](LICENSE)
