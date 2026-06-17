---
name: git-work-report
description: Generate structured work reports from your git commit history. Analyzes commits and diffs by author to produce career-ready summaries — resumes, performance reviews, LinkedIn posts — organized by 6-month periods. Supports multi-repo, Simple Mode (log-only), and incremental updates.
license: MIT
---

# git-work-report

> Respond in the user's language throughout.

Turn your git commit history into structured work reports — resumes, performance reviews, and LinkedIn posts — in minutes.

> Git 커밋 히스토리와 실제 코드 변경 내용(diff)을 본인 커밋 기준으로 분석하여, 6개월 단위 기술 성과 요약서를 자동 생성하는 스킬.

---

## When to use

Use this skill when the user says:
- "Summarize my git commits" / "Generate a work report from my commits"
- "Create a resume-ready summary of my work" / "What did I work on this year?"
- "내 커밋 히스토리를 요약해줘" / "기술 성과 요약서 만들어줘"

Or use shorthand presets:
- `gwr` — show tip and start from Q0
- `gwr summary` — prefills Q4=summary
- `gwr resume` — prefills Q4=resume
- `gwr performance` — prefills Q4=performance
- `gwr linkedin` — prefills Q4=linkedin, Q5 default=english
- `gwr simple` — prefills Q0=simple

---

## Directory Structure

```text
{output_dir}/work-summary/
├── README.md                          # TOC with repo+period links, last summary date
├── report_summary.md                  # ★ Final report (always generated)
├── report_resume.md                   # When purpose includes resume
├── report_performance.md              # When purpose includes performance
├── report_linkedin.md                 # When purpose includes linkedin
├── {repo-a}/
│   ├── summary_{period_key}.md            # Per-repo, per-period intermediate output
│   ├── summary_{period_key}_resume.md
│   ├── summary_{period_key}_performance.md
│   ├── summary_{period_key}_linkedin.md
│   └── _work/
│       └── {period_key}/
│           ├── v1/
│           │   ├── config.json            # Fingerprint for reuse detection
│           │   ├── 00_commit_list.md
│           │   └── {hash}.json
│           └── v2/ (when fingerprint changes)
└── {repo-b}/...
```

**`{period_key}` rules:**
| Period | Key example |
|--------|------------|
| Auto-split — first half | `2026-H1` |
| Auto-split — second half | `2025-H2` |
| Custom range | `202510_202603` |

---

## Instructions

### 🔔 Step 0. User Configuration (Ask before starting)

Ask the user the following questions **in order**. Wait for all answers before beginning.

```
If triggered by bare `gwr`, output this tip before Q0:

💡 Tip: Use `gwr [option]` for quick access.
   gwr summary       — technical work summary
   gwr resume        — resume format
   gwr performance   — performance review format
   gwr linkedin      — LinkedIn post format
   gwr simple        — simple mode (git log only, faster)

Q0. Choose your analysis mode:
    1) Full analysis — git log + diff analysis (accurate, takes longer)
    2) Simple mode  — git log only, fast summary (commit message based)
    [Enter] (default = 1):

Q1. Where would you like to save the output?
    A 'work-summary' folder will be created inside this path.
    [Enter] (default = current directory):

Q2. Enter the repository path(s) to analyze.
    Separate multiple repos with commas. Append `:branch-name` for non-main branches.
    [Enter] (default = current directory, main branch):

Q3. Select the time period to analyze:
    1) Full history (auto-split into 6-month periods)
    2) Custom range (e.g., 2026-01 ~ 2026-03)
    3) Only new commits since last summary (incremental update)
    [Enter]:

Q4. What is the purpose of this report? (Select multiple: 1 3 or 1,3)
    1) Technical work summary (default)
    2) Resume — action-verb focused, concise, metrics-highlighted
    3) Performance review — context·background·results, references issue numbers
    4) LinkedIn post — English, starts with action verb
    [Enter] (default = 1):

Q5. Choose the output language:
    1) Korean (default)  2) English  3) Korean + English  4) Other
    [Enter] (default = 1):
    (If Q4 includes LinkedIn, default is 2 — English)
    (If 4 is selected, ask: "Please enter the language name (e.g., Japanese, French):")
```

Build config from answers:
```json
{
  "analysis_mode": "full" | "simple",
  "output_dir": "{path}/work-summary",
  "repos": [{ "path": "...", "branch": "main" }],
  "period": "full" | "custom:{start}~{end}" | "incremental",
  "purposes": ["summary", "resume", "performance", "linkedin"],
  "language": "ko" | "en" | "both" | "{custom}"
}
```

### Step 0.5. Final Settings Confirmation

Summarize config and ask Y/n. On `n`, ask which number to edit (max 3 rounds).

```text
📋 Analysis Settings Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
0) Analysis mode  : Full analysis (git log + diff)
1) Output path    : ~/Downloads/work-summary/
2) Repositories   : repo-a (main), repo-b (release)
3) Period         : Full history (auto-split by 6 months)
4) Purpose        : Technical summary, Resume
5) Language       : Korean
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Proceed with these settings? (Y/n)
To edit a setting, enter its number (e.g., 2)
```

---

### Step 1. Pre-check — Author & Commit Range

For each repo:

1. **Fingerprint check**: If `_work/{period_key}/` exists, read latest `vN/config.json` and compare: `repo_path`, `branch`, `author_email/name`, `author_match_rule`, `period_range`, `language`, `analysis_mode`. Match → reuse existing `vN/` (skip cached hashes). Mismatch → create `vN+1/`.

2. **Author detection** (priority order):
   - `git config user.email` → `--author="{EMAIL}"`
   - `git config user.name` → `--author="{NAME}"` fallback (warn if duplicate names exist)
   - Neither → ask user directly

```bash
# Check author
AUTHOR=$(git config user.email) || AUTHOR=$(git config user.name)
# Verify commit dates
git --no-pager log {branch} --author="$AUTHOR" --pretty=format:"%ad" --date=short | sort -u
```

3. **Incremental mode**: Read last summary date from `work-summary/README.md`, analyze only commits after that date.

---

### Step 2. Commit List + Revert Removal + Grouping (Full Mode)

#### 2-1. Extract commit hashes (no merges)
```bash
git --no-pager log {branch} --author="$AUTHOR" --after="{START}" --before="{END}" \
  --pretty=format:"%H | %ad | %s" --date=short --no-merges
```

#### 2-2. Auto-detect and remove revert pairs
```bash
git --no-pager log {branch} --author="$AUTHOR" --grep="^Revert" \
  --pretty=format:"%H %s" --no-merges
```
Remove both the revert commit and its original. Keep reverted-then-rewritten commits.

#### 2-3. Group similar commits (clustering)
Criteria (priority order): same file patterns → common keywords in messages → temporal proximity (within 3 days).

`00_commit_list.md` format:
```markdown
# {YYYY}-H1 Commit List
## [Group A] feature_pipeline
| # | Hash | Date | Message | Status |
|---|------|------|---------|--------|
| A1 | dc820de | 2026-03-06 | feat: feature_pipeline added | ⬜ |
| A2 | 7d1eddc | 2026-03-06 | fix: moving path bug fix | ⬜ |
```

### Step 2-S. Commit List (Simple Mode)

Same as Step 2 but message-based grouping only (no diff analysis). Group by `feat:`/`fix:`/`refactor:`/`chore:` prefix, then keyword, then date proximity. All status = `⬜` (no hash JSON).

---

### Step 3. Per-Commit Diff Analysis → JSON (Full Mode)

Process commits in group order. Skip if `{hash}.json` already exists. Update `00_commit_list.md` status to `✅` on completion.

#### 3-0. Size check
```bash
git --no-pager show {hash} --stat --no-patch
git --no-pager show {hash} --patch -- '*.yaml' '*.py' '*.sql' '*.md' '*.json' \
  ':!*.lock' ':!*.png' ':!*.jpg' ':!*.gif' | grep '^+' | grep -v '^+++' | wc -l
```
**Large commit**: 300+ added lines OR 10+ changed files → use 3-1B path. Otherwise use 3-1A.

#### 3-1A. Normal commit — read full diff
```bash
git --no-pager show {hash} --patch -- '*.yaml' '*.py' '*.sql' '*.md' '*.json' \
  ':!*.lock' ':!*.png' ':!*.jpg' ':!*.gif' | grep -v '^-' | grep -v '^@@' | grep -v '^diff' | grep -v '^index'
```

#### 3-1B. Large commit — staged extraction (4 steps)
1. **Metadata**: `git show --stat`, `git show --format="%B"` for file list + commit body
2. **Priority files**: inline docs (`doc_md:`, `description:`, `GOAL:`), SQL/HiveQL, DAG task defs, new files
3. **High-signal patterns**: grep for `FROM|INTO|INSERT|CREATE TABLE`, `def |task_id|dag_id`, `JIRA-|#[0-9]+`, `schedule_interval|retries`
4. **Structure JSON** from extracted signals only (no full diff read)

#### 3-2. Extraction priorities
| Priority | Item | Pattern |
|----------|------|---------|
| 1 | Inline docs, JIRA numbers | `doc_md:`, `description:`, `JIRA-1234` |
| 2 | File header comments | `# GOAL:`, pipeline descriptions |
| 3 | Table/DB names | `FROM`, `INTO`, `database_name` |
| 4 | Tech keywords | `TEZ`, `TriggerDagRun`, `HiveQL` |

#### 3-3. Save analysis (lightweight JSON)
```json
{
  "hash": "dc820de", "date": "2026-03-06", "group": "A",
  "category": "feat", "files": ["yamls/feature_pipeline_step1.yaml"],
  "representative_files": ["feature_pipeline_step1.yaml"],
  "tables": ["myproject.feature_table"],
  "tech": ["HiveQL", "TriggerDagRun", "TEZ"],
  "business_logic": "Feature pipeline Step1 new build",
  "jira": ["JIRA-1234"],
  "scale": "new_file", "large_commit": false,
  "summary": "feature_pipeline Step1 DAG new build"
}
```
`scale`: `new_file` | `hotfix` | `refactor` | `config_change`

---

### Step 4. Final Summary (Full Mode)

Read all `_work/{period_key}/*.json`, aggregate by group, generate summary:

1. **Period overview**: total commits, active period, group count, key flow
2. **Work breakdown** (per group): commit count, representative files, tables, business logic, tech stack, "build → stabilize" narrative
3. **Tech highlights**: new tech adoption, external system integration
4. **Performance summary (3-5 items)**: purpose-specific format

### Step 4-S. Message-Based Summary (Simple Mode)

Same structure as Step 4 but from commit messages only. **Must include warning**: `> ⚠️ This summary is based on commit messages only (Simple Mode). Actual code changes are not reflected.`

---

### Step 5. Purpose-Specific Report Files

Generate per-repo intermediate files (`summary_{period_key}*.md`) by reformatting existing JSON/summary data. No re-reading of diffs.

| Purpose | Per-repo file | Final report | JIRA? |
|---------|--------------|-------------|-------|
| `summary` | `summary_{period_key}.md` | `report_summary.md` | Excluded |
| `resume` | `summary_{period_key}_resume.md` | `report_resume.md` | Excluded |
| `performance` | `summary_{period_key}_performance.md` | `report_performance.md` | **Included** |
| `linkedin` | `summary_{period_key}_linkedin.md` | `report_linkedin.md` | Excluded |

**Format guidelines:**
- **Summary**: Group flows + tech stack, commit counts, table names
- **Resume**: Action-verb focused, 1-2 lines, metrics emphasis. e.g. `Designed and built Feature pipeline (HiveQL, TEZ, Airflow K8s)`
- **Performance**: Detailed with context/background/results, JIRA numbers. e.g. `[2026.03] Feature pipeline new build (JIRA-1234): DAG development for data-driven route exploration. TEZ engine migration improved processing stability.`
- **LinkedIn**: Present tense, action verb start, English. e.g. `Designed and built feature pipeline (HiveQL, TEZ Engine, Airflow K8s).`

---

### Step 6. Final Report Generation (`report_*.md`)

Aggregate per-repo `summary_*.md` and `_work/*.json` into final reports. **Token economy**: JSON → summary → raw diff (never re-read diffs).

`report_summary.md` template:
```markdown
# Technical Work Report
> Author: {author} | Date: {date} | Projects: {N} | Purpose: {purposes}

## Key Achievements
- {achievement 1}
- {achievement 2}
- {achievement 3}

---

## Project: {repo-name}
> Repo: {path} | Branch: {branch} | Period: {period}

### {period_key}
{summary}
```

**Sorting**: Repos in user input order. Periods: year descending → H2 > H1.

**Multi-repo bonus** (2+ repos): Contribution table with repo, period, commits, tech stack, weight.

### Step 7. Cleanup

1. Update `work-summary/README.md` TOC with last summary date (for incremental updates)
2. Ask user whether to delete `_work/` directory

---

## Output Format

Adapt to `config.language`. Performance summary sentence formats:

**Korean:**
- **[시스템명] 신규 설계 및 구축**: [기술 스택·핵심 로직] 포함, [이슈] 연계
- **[기술명] 전환/도입**: [배경] → [개선 효과] 달성
- **[작업명] 구현 및 운영 안정화**: [반복 수정 횟수]회 수정 끝에 안정화

**English:**
- **Designed and built [System Name]**: including [tech stack · core logic], linked to [issue]
- **Introduced/migrated to [Technology]**: [background] → achieved [improvement]
- **Implemented and stabilized [Feature]**: stabilized after [N] iterative fixes

---

## Notes

- Merge commits excluded via `--no-merges`
- Revert pairs auto-removed in Step 2. Reverted-then-rewritten commits preserved.
- Ambiguous messages (`bug fix`, `no message`): infer from inline docs, comments, table names
- Large commits (300+ lines or 10+ files): use staged extraction (Step 3-1B), never truncate
- Deleted lines (`-`) excluded from analysis to save tokens
- `user.email` unset → fallback to `user.name`. Warn on duplicate names.
