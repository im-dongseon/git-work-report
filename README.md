# git-work-report

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)

Turn your git commit history into structured work reports — resumes, performance reviews, and LinkedIn posts — in minutes.

## Language

- English (this README)
- 한국어: see [README_KO.md](./README_KO.md)

---

## What it does

- Analyzes your git commits (and optionally code diffs) by author
- Generates structured work reports organized by 6-month periods
- Supports multiple output formats: technical summary, resume, performance review, LinkedIn
- Works with single or multiple repositories
- Offers Simple Mode (log-only) for fast results
- Supports incremental updates (only new commits since last run)

Reports are produced on a 6-month cadence. If a report already exists for a period, the tool uses the report's metadata (last analysis date) and analyzes only commits after that date to incrementally update the existing report.

---

## Quick Start

### Install

```bash
# Install from GitHub (once published)
gh skill install im-dongseon/git-work-report git-work-report

# Or install locally
gh skill install ~/dev/repo/git-work-report git-work-report --from-local
```

### Run

In any AI agent (GitHub Copilot, Claude Code, etc.):

```
"Summarize my git commits"
"Generate a work report from my commits"
```

Or use shorthand presets:

| Shorthand | Effect |
|-----------|--------|
| `gwr` | Start with tip, then full setup |
| `gwr summary` | Prefills: technical summary |
| `gwr resume` | Prefills: resume format |
| `gwr performance` | Prefills: performance review |
| `gwr linkedin` | Prefills: LinkedIn post (English default) |
| `gwr simple` | Prefills: Simple Mode (log-only) |

---

## Output

Reports are saved under `{output_dir}/work-summary/`:

```
work-summary/
├── report_summary.md          # Always generated
├── report_resume.md           # When purpose includes resume
├── report_performance.md      # When purpose includes performance
├── report_linkedin.md         # When purpose includes linkedin
├── {repo-name}/
│   ├── summary_{period_key}.md
│   └── _work/{period_key}/v1/
│       ├── config.json        # Fingerprint for reuse detection
│       ├── 00_commit_list.md
│       └── {hash}.json        # Per-commit analysis cache (Full Mode only)
```

### Analysis Modes

| Mode | Data Source | Speed | Accuracy |
|------|------------|-------|----------|
| **Full** | git log + diff | Slower | High (code-based) |
| **Simple** | git log only | Fast | Medium (message-based) |

---

## Requirements

- `git` CLI
- `gh` CLI (GitHub CLI) v2.94.0+
- An AI agent: GitHub Copilot, Claude Code, Cursor, Gemini CLI, etc.

---

## Installation

```bash
# From GitHub (once published)
gh skill install im-dongseon/git-work-report git-work-report

# From local path
gh skill install ~/dev/repo/git-work-report git-work-report --from-local

# Verify
gh skill list | grep git-work-report
```

---

## License

MIT — see [LICENSE](./LICENSE)
