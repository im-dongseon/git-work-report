---
name: git-work-report
description: Generate structured work reports from your git commit history. Analyzes commits and diffs by author to produce career-ready summaries — resumes, performance reviews, LinkedIn posts — organized by 6-month periods. Supports multi-repo, Simple Mode (log-only), and incremental updates.
---

# git-work-report

> Respond in the user's language throughout.

Turn your git commit history into structured work reports — resumes, performance reviews, and LinkedIn posts — in minutes.

> Git 커밋 히스토리와 실제 코드 변경 내용(diff)을 본인 커밋 기준으로 분석하여, 6개월 단위 기술 성과 요약서를 자동 생성하는 스킬.

---

## When to use

Use this skill when the user says:
- "Summarize my git commits"
- "Generate a work report from my commits"
- "Create a resume-ready summary of my work"
- "What did I work on this year?"
- 또는 다음 한국어 요청:
- "내 커밋 히스토리를 요약해줘"
- "어떤 작업을 했는지 정리해줘"
- "기술 성과 요약서 만들어줘"
- "이력서에 쓸 수 있게 작업 내용 정리해줘"

Or use shorthand presets like `gwr [option]` to prefill certain answers and shorten the setup flow.

Available presets:
- `gwr` — show tip and start from Q0
- `gwr summary` — prefills Q4=summary
- `gwr resume` — prefills Q4=resume
- `gwr performance` — prefills Q4=performance
- `gwr linkedin` — prefills Q4=linkedin, Q5 default=english (still asks Q5, user can change)
- `gwr simple` — prefills Q0=simple

---

## Directory Structure

```text
{output_dir}/work-summary/
├── README.md                          # 목차 (레포명 + 기간별 링크, 마지막 요약 날짜 기록)
├── report_summary.md                  # ★ 기본 최종 리포트 (항상 생성)
├── report_resume.md                   # 목적별 최종 리포트 (purposes에 resume 포함 시)
├── report_performance.md              # 목적별 최종 리포트 (purposes에 performance 포함 시)
├── report_linkedin.md                 # 목적별 최종 리포트 (purposes에 linkedin 포함 시)
├── {repo-a}/
│   ├── summary_{period_key}.md            # 중간 산출물 — 기술 성과 요약
│   ├── summary_{period_key}_resume.md     # 중간 산출물 — 이력서용
│   ├── summary_{period_key}_performance.md
│   ├── summary_{period_key}_linkedin.md
│   └── _work/
│       └── {period_key}/
│           ├── v1/
│           │   ├── config.json            # fingerprint (재사용 판단용)
│           │   ├── 00_commit_list.md
│           │   └── {hash}.json
│           └── v2/
│               ├── config.json
│               ├── 00_commit_list.md
│               └── {hash}.json
└── {repo-b}/
    ├── summary_{period_key}.md
    └── _work/
        └── {period_key}/
            └── v1/
                ├── config.json
                ├── 00_commit_list.md
                └── {hash}.json
```

**역할 분리:**
| 파일 | 역할 |
|------|------|
| `{repo-name}/summary_{period_key}*.md` | 중간 산출물 (레포별, 기간별) |
| `{repo-name}/_work/` | 커밋 단위 캐시 및 분석 근거 |
| `report_*.md` | **최종 deliverable** (루트 하위) |

**`{period_key}` 규칙:**
| 기간 선택 | period_key 예시 |
|---------|--------------|
| 전체 자동 분할 — 상반기 | `2026-H1` |
| 전체 자동 분할 — 하반기 | `2025-H2` |
| 사용자 지정 기간 | `202510_202603` |

> 단일 레포도 `{repo-name}/` 서브디렉토리를 생성한다. 단일/멀티 처리 흐름이 동일하다.

---

## Instructions

### 🔔 Step 0. User Configuration (Ask before starting)

Ask the user the following questions **in order** at skill start.
Wait for all answers before beginning analysis.

```
If triggered by bare `gwr` (without any option), output the following before Q0:

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

    Examples:
      ~                  →  ~/work-summary/
      ~/Documents        →  ~/Documents/work-summary/
      ~/workspace/myteam →  ~/workspace/myteam/work-summary/

    [Enter] (default = current directory):

Q2. Enter the repository path(s) to analyze.
    Separate multiple repos with commas.
    To specify a branch, append `:branch-name` after the path.
    (Default branch: main. Use the production/release branch.)

    Examples:
      (Enter only)                                  →  current directory, main branch
      ~/workspace/repo-a                            →  single repo, main branch
      ~/workspace/repo-a:release                    →  single repo, release branch
      ~/workspace/repo-a, ~/work/repo-b:release     →  multi-repo (repo-a=main, repo-b=release)

    [Enter] (default = current directory):

Q3. Select the time period to analyze:
    1) Full history (auto-split into 6-month periods)
    2) Custom range (e.g., 2026-01 ~ 2026-03)
    3) Only new commits since last summary (incremental update)

    [Enter]:

Q4. What is the purpose of this report?
    You can select multiple by space or comma (e.g., 1 3  or  1,3)

    1) Technical work summary (default)
    2) Resume                — action-verb focused, concise, metrics-highlighted
    3) Performance review    — context·background·results, references issue numbers
    4) LinkedIn post         — English, starts with action verb

    [Enter] (default = 1):

Q5. Choose the output language:
    1) Korean (default)
    2) English
    3) Korean + English (bilingual)
    4) Other — you will be asked to specify

    [Enter] (default = 1):
    (If Q4 included LinkedIn, default is 2 — English)
    (If 4 is selected, ask: "Please enter the language name (e.g., Japanese, French):")
    (Normalize the input: English→en, Korean→ko, Japanese→ja, French→fr, Spanish→es, Chinese→zh; others: lowercase+trim)
```

답변을 바탕으로 아래 config를 구성하여 이후 모든 Step에서 참조한다:
```
config = {
  analysis_mode: "full" | "simple",              // Q0
  output_dir: "{path}/work-summary",             // Q1
  repos: [                                       // Q2 — [{path, branch}] array
    { path: "~/workspace/repo-a", branch: "main" },
    { path: "~/work/repo-b",      branch: "release" }
  ],
  period: "full" | "custom:{start}~{end}" | "incremental",  // Q3
  purposes: ["summary", "resume", "performance", "linkedin"], // Q4
  language: "ko" | "en" | "both" | "{custom_normalized}"    // Q5
}
```

> 멀티레포 모드(`repos` 2개 이상)는 자동으로 Step 6까지 실행한다.
> 모든 산출물은 `config.output_dir` 하위에 저장한다 (`.agent/history/` 대신).

---

### Step 0.5. 최종 설정 확인

Q1~Q5 답변을 모두 받은 뒤, 분석 시작 전에 설정을 요약 출력하고 Y/n 확인을 받는다.

```text
📋 Analysis Settings Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
0) Analysis mode  : Full analysis (git log + diff)
1) Output path    : ~/Downloads/work-summary/
2) Repositories   : repo-a (main)
                  : repo-b (release)
3) Period         : Full history (auto-split by 6 months)
4) Purpose        : Technical summary, Resume
5) Language       : Korean
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Proceed with these settings? (Y/n)
To edit a setting, enter its number (e.g., 2)
```

- `Y` (또는 Enter) → Step 1로 진행
- `n` → 수정할 항목 번호 입력 요청 → 해당 번호의 질문만 재질문 → 설정 확인 화면 다시 출력
- 재확인 흐름은 최대 3회까지 허용

---

### Step 1. 사전 확인 — 작성자 및 커밋 범위 파악

각 레포에 대해 실행한다.

#### 1-0. `_work` fingerprint 체크

레포별로 `{config.output_dir}/{repo-name}/_work/{period_key}/`가 이미 존재하는지 확인한다.

```bash
ls "{output_dir}/{repo-name}/_work/{period_key}/" 2>/dev/null
```

존재하면 가장 큰 번호의 `vN/config.json`을 읽어 현재 설정과 비교한다.

**비교 항목**: `repo_path`, `branch`, `author_email`(또는 `author_name`), `author_match_rule`, `period_range`, `language`, `analysis_version`

**판단 규칙:**
1. `_work/{period_key}/`가 처음 생성되는 경우 → `v1/` 생성
2. fingerprint가 **일치** → 기존 `vN/` 재사용
   - 이미 있는 `{hash}.json`은 스킵
   - 없는 커밋만 신규 분석
   - `00_commit_list.md`만 갱신
3. fingerprint가 **불일치** → 새 버전 디렉토리 생성
   - 예: 기존 `v1/`이 있으면 `v2/` 생성
   - `{repo-name}/summary_*.md`는 덮어쓰기

> **분석 방식에 따른 실행 흐름:**
> - **Full Mode** (`analysis_mode: "full"`): Step 2 → Step 3 (diff 분석) → Step 4 → Step 5 → Step 6 → Step 7
> - **Simple Mode** (`analysis_mode: "simple"`): Step 2-S → Step 4-S → Step 5 → Step 6 → Step 7  
>   (Step 2, 3 생략. `_work/`에는 `config.json`과 `00_commit_list.md`만 저장)

**`config.json` 스펙:**
```json
{
  "repo_path": "~/workspace/repo-a",
  "branch": "main",
  "author_email": "user@example.com",
  "author_name": "Your Name",
  "author_match_rule": "email",
  "period_key": "2026-H1",
  "period_range": "2026-01-01~2026-06-30",
  "language": "ko",
  "analysis_mode": "simple",
  "analysis_version": "1"
}
```

> `purposes`는 fingerprint 비교 대상에서 **제외**한다.  
> `{hash}.json`의 내용은 purposes와 무관하므로, purposes가 변경돼도 기존 `{hash}.json`은 재사용 가능하다.  
> `analysis_mode`는 fingerprint 비교 대상에 **포함**한다. (simple ↔ full 변경 시 새 vN/ 생성)

```bash
# 현재 git 설정 작성자 확인
git config user.name && git config user.email

# 전체 작성자 목록 확인
git --no-pager log --pretty=format:"%an | %ae" | sort | uniq -c | sort -rn
```

**작성자 식별 규칙 (우선순위 순):**

1. `git config user.email` 값이 있으면 → `--author="{EMAIL}"` 사용
2. `user.email`이 비어있고 `user.name`이 있으면 → `--author="{NAME}"` fallback 사용
   - 단, 동명이인 위험 확인: 전체 작성자 목록에서 동일 이름 여러 개이면 사용자에게 확인 요청
   - 결과물 서두에 `> ⚠️ 작성자 매칭 기준: user.name="{NAME}" (email 미설정)` 명시
3. 둘 다 없으면 → 사용자에게 직접 작성자명/이메일 입력 요청

```bash
# 1순위: email 기반
AUTHOR=$(git config user.email)

# 2순위: name fallback
if [ -z "$AUTHOR" ]; then
  AUTHOR=$(git config user.name)
fi

# 본인 커밋 날짜 범위 확인
git --no-pager log {branch} --author="$AUTHOR" --pretty=format:"%ad" --date=short | sort -u
```

- 반드시 `--author` 옵션으로 **본인 커밋만** 필터링한다.
- 커밋이 없는 기간은 스킵한다.
- **점진적 업데이트 모드**: `{config.output_dir}/work-summary/README.md`에서 마지막 요약 날짜를 읽어 그 이후 커밋만 대상으로 한다.

---

### Step 2. 커밋 목록 생성 + Revert 쌍 제거 + 그룹핑 (Full Mode)

#### 2-1. 커밋 해시 목록 추출 (Merge 커밋 제외)

```bash
# Step 1에서 결정된 $AUTHOR 변수 사용
git --no-pager log {branch} --author="$AUTHOR" --after="{START}" --before="{END}" \
  --pretty=format:"%H | %ad | %s" --date=short --no-merges
```

#### 2-2. Revert 쌍 자동 감지 및 제거

Revert 커밋과 그 원본 커밋을 쌍으로 감지하여 목록에서 모두 제거한다.
(최종 결과에 영향 없는 노이즈 제거. 단, Revert 후 재작성된 커밋은 유지)

```bash
git --no-pager log {branch} --author="$AUTHOR" --grep="^Revert" \
  --pretty=format:"%H %s" --no-merges
```

#### 2-3. 유사 커밋 그룹핑 (클러스터링)

같은 파일·모듈을 건드린 커밋을 사전에 그룹핑한다.
이후 diff 분석 시 그룹 단위로 처리하면 "구축 → 안정화" 흐름이 자연스럽게 드러난다.

그룹핑 기준 (우선순위 순):
1. 동일 파일명 패턴 (예: `feature_pipeline*.yaml`)
2. 커밋 메시지의 공통 키워드
3. 날짜 근접성 (3일 이내 연속 커밋)

`00_commit_list.md` 형식:
```markdown
# {YYYY}-H1 커밋 목록

## [그룹 A] feature_pipeline 구축
| # | 해시 | 날짜 | 메시지 | 처리 |
|---|------|------|--------|------|
| A1 | dc820de | 2026-03-06 | feat: feature_pipeline 추가 | ⬜ |
| A2 | 7d1eddc | 2026-03-06 | fix: moving path 버그 수정 | ⬜ |

## [그룹 B] batch_pipeline 구축
| # | 해시 | 날짜 | 메시지 | 처리 |
|---|------|------|--------|------|
| B1 | 7ba3c59 | 2026-03-24 | feat: batch_pipeline step1 추가 | ⬜ |
...
```

---

### Step 2-S. 커밋 목록 생성 (Simple Mode)

> `config.analysis_mode == "simple"` 일 때만 실행. Full Mode는 Step 2를 사용한다.

```bash
# 커밋 목록 추출 (Merge 커밋 제외)
git --no-pager log {branch} --author="$AUTHOR" --after="{START}" --before="{END}" \
  --pretty=format:"%H | %ad | %s" --date=short --no-merges
```

**Revert 쌍 제거**: Step 2와 동일한 방식으로 Revert 커밋과 원본 커밋 쌍을 감지하여 제거한다.

**메시지 기반 그룹핑**: diff 분석 없이 커밋 메시지 패턴으로만 그룹핑한다.
1. `feat:` / `fix:` / `refactor:` / `chore:` prefix 기준 1차 분류
2. 커밋 메시지의 공통 키워드 기준 그룹핑
3. 날짜 근접성 (3일 이내 연속 커밋) 보조 기준

`00_commit_list.md` 형식은 Step 2와 동일하다. 단, `처리` 컬럼 값은 모두 `⬜` (hash JSON 없음).

> Simple Mode에서 `_work/{period_key}/vN/`에는 `config.json`과 `00_commit_list.md`만 저장한다.

---

### Step 3. 커밋별 diff 분석 → JSON 저장 (Full Mode)

그룹 순서대로 커밋을 처리한다.

- **이미 `{hash}.json`이 존재하면 스킵** (재실행 안전)
- 처리 완료 시 `00_commit_list.md` 해당 행 `처리` 컬럼을 `✅`로 업데이트

#### 3-0. 커밋 규모 판별 (일반 vs 대형)

```bash
# 필터링 후 추가 줄 수 + 변경 파일 수 확인
git --no-pager show {commit_hash} --stat --no-patch
git --no-pager show {commit_hash} --patch \
  -- '*.yaml' '*.py' '*.sql' '*.md' '*.json' \
  ':!*.lock' ':!*.png' ':!*.jpg' ':!*.gif' \
  | grep '^+' | grep -v '^+++' | wc -l
```

**대형 커밋 기준**: 필터링 후 추가 줄 **300줄 이상** OR 변경 파일 **10개 이상**  
→ 해당하면 [3-1B 대형 커밋 경로](#3-1b)로 처리, 아니면 [3-1A 일반 경로](#3-1a)로 처리

#### 3-1A. 일반 커밋 — diff 전체 읽기

```bash
# 추가된 줄만, 소스 파일만, 바이너리·lock 제외
git --no-pager show {commit_hash} --patch \
  -- '*.yaml' '*.py' '*.sql' '*.md' '*.json' \
  ':!*.lock' ':!*.png' ':!*.jpg' ':!*.gif' \
  | grep -v '^-' | grep -v '^@@' | grep -v '^diff' | grep -v '^index'
```

#### 3-1B. 대형 커밋 — 단계적 추출 (4단계)

**1단계: 메타데이터 수집**
```bash
# 파일 목록, 변경 규모, 메시지, 커밋 본문 확인
git --no-pager show {commit_hash} --stat --no-patch
git --no-pager show {commit_hash} --format="%B" --no-patch
```

**2단계: 우선순위 파일 선별 및 상세 조회**  
아래 우선순위로 파일을 선별해 `--patch`로 재조회한다:
1. 프로젝트에서 사용하는 inline 문서 패턴 (예: `doc_md:`, `description:`, `GOAL:` 등) 이 담긴 파일
2. `.sql`, HiveQL 쿼리 파일
3. DAG 태스크 정의, 설정값이 있는 파일
4. 신규 추가 파일 (`new file mode`)

```bash
# 특정 파일만 상세 조회
git --no-pager show {commit_hash} -- {priority_file_path} \
  | grep -v '^-' | grep -v '^@@' | grep -v '^diff' | grep -v '^index'
```

**3단계: 의미 밀도 높은 패턴만 추출**  
선별된 파일에서 다음 패턴을 grep으로 추출한다:
```bash
# 프로젝트 inline 문서 패턴 (예: doc_md:, description:, GOAL: 등)
grep -A5 'doc_md:\|description:\|GOAL:'

# 상단 주석 (파이프라인 목적 설명)
grep -E '^\+\s*#.*'

# SQL/HiveQL 핵심 구문
grep -E '^\+.*(FROM|INTO|INSERT|CREATE TABLE|database_name)'

# 함수·태스크 정의
grep -E '^\+.*(def |task_id|dag_id|BashOperator|PythonOperator)'

# 설정값 변화
grep -E '^\+.*(JIRA-|#[0-9]+|[A-Z]+-[0-9]+|schedule_interval|retries)'
```

**4단계: 추출 결과로 JSON 구조화**  
3단계에서 수집한 신호만으로 JSON을 작성한다. 전체 diff를 그대로 읽지 않는다.

#### 3-2. 우선 추출 항목 (정확도 향상)

| 우선순위 | 항목 | 예시 패턴 |
|---------|------|----------|
| 1순위 | 프로젝트에서 사용하는 inline 문서 패턴 (예: `doc_md:`, `description:`, `GOAL:` 등) | 비즈니스 목적 명문화 |
| 1순위 | JIRA·이슈 번호 | `JIRA-1234`, `PROJ-123` |
| 2순위 | 파일 상단 `#` 주석 | `GOAL:`, 파이프라인 설명 |
| 3순위 | 테이블·DB명 | `FROM`, `INTO`, `database_name` |
| 4순위 | 기술 키워드 | `TEZ`, `TriggerDagRun`, `HiveQL` |

#### 3-3. 분석 결과 저장 (경량 JSON)

```json
{
  "hash": "dc820de",
  "date": "2026-03-06",
  "group": "A",
  "category": "feat",
  "files": ["yamls/hive_query/feature/feature_pipeline_step1.yaml"],
  "representative_files": ["feature_pipeline_step1.yaml"],
  "tables": ["myproject.feature_table"],
  "tech": ["HiveQL", "TriggerDagRun", "TEZ"],
  "business_logic": "Feature pipeline Step1 신규 구축",
  "jira": ["JIRA-1234"],
  "scale": "new_file",
  "large_commit": false,
  "summary": "feature_pipeline Step1 DAG 신규 구축"
}
```

`scale` 값: `new_file` | `hotfix` | `refactor` | `config_change`  
`large_commit`: 대형 커밋 기준 해당 시 `true` (단계적 추출 적용 여부 기록)

---

### Step 4. 최종 요약서 작성 (Full Mode)

`_work/{YYYY}-H1/*.json`을 모두 읽어 그룹 단위로 집계 후 요약서를 생성한다.

요약서 구성:
1. **기간 개요**: 총 커밋 수, 활동 기간, 그룹 수, 주요 흐름
2. **주요 작업 분류** (카테고리 × 그룹): 각 그룹마다 아래 항목 포함
   - 근거 커밋 수 (해당 그룹의 커밋 수)
   - 대표 파일 (그룹 내 `representative_files` 상위 3개)
   - 테이블명·비즈니스 로직·기술 스택
   - "구축 → 안정화" 흐름 서술
3. **기술적 하이라이트**: 신기술 도입, 외부 시스템 연동
4. **성과 요약 (3~5개)**: 목적별 포맷으로 출력

> Note: Include JIRA/issue references only in performance review output (`report_performance.md`). Exclude from summary, resume, and LinkedIn outputs.

---

### Step 4-S. 커밋 메시지 기반 요약서 작성 (Simple Mode)

> `config.analysis_mode == "simple"` 일 때만 실행.

`00_commit_list.md`의 커밋 메시지만을 바탕으로 요약서를 작성한다.

요약서 구성:
1. **기간 개요**: 총 커밋 수, 활동 기간, 그룹 수, prefix 분포 (feat/fix/refactor 등)
2. **주요 작업 분류** (그룹별):
   - 그룹 커밋 수 + 날짜 범위
   - 커밋 메시지에서 추론 가능한 기술 스택·시스템명
   - "구축 → 안정화" 흐름 (feat 다음 fix 패턴 감지)
3. **성과 요약 (3~5개)**: 목적별 포맷으로 출력 (Step 4와 동일 구조)

> **출력 주의사항**: 결과물 서두에 반드시 아래 문구를 포함한다:
> ```
> > ⚠️ This summary is based on commit messages only (Simple Mode). Actual code changes are not reflected.
> ```

---

### Step 5. 목적별 리포트 파일 생성

`config.purposes`에 따라 레포별 중간 산출물(`summary_*.md`)을 생성한다.  
목적별 리포트는 **기존 JSON/summary를 재가공**하여 생성한다. diff를 다시 읽어 재분석하지 않는다.

**파일명 매핑:**
| purposes 값 | 레포 중간 산출물 | 최종 리포트 | 생성 조건 |
|------------|----------------|-----------|---------|
| `summary` (기본) | `summary_{period_key}.md` | `report_summary.md` | **항상 생성** — JIRA 제외 |
| `resume` | `summary_{period_key}_resume.md` | `report_resume.md` | purposes에 포함 시 — JIRA 제외 |
| `performance` | `summary_{period_key}_performance.md` | `report_performance.md` | purposes에 포함 시 — **JIRA 포함** |
| `linkedin` | `summary_{period_key}_linkedin.md` | `report_linkedin.md` | purposes에 포함 시 — JIRA 제외 |

#### 기술 성과 요약 (`summary_{period_key}.md`)
- 그룹별 작업 흐름과 기술 스택 서술
- 커밋 수, 테이블명 포함 (JIRA 제외)

#### 이력서용 (`summary_{period_key}_resume.md`)
- 동사 중심, 간결 (1~2줄), 수치 강조 (JIRA 제외)
- 예: `Feature pipeline 설계·구축 (HiveQL, TEZ, Airflow K8s)`

#### 성과보고서용 (`summary_{period_key}_performance.md`)
- 기간·배경·결과 포함 상세 서술, JIRA 번호 병기
- 예: `[2026.03] Feature pipeline 신규 구축 (JIRA-1234): 데이터 기반 이동 경로 탐색 DAG 개발. TEZ 엔진 전환으로 처리 안정성 개선.`

#### LinkedIn용 (`summary_{period_key}_linkedin.md`, 영문)
- Present tense, action verb 시작 (JIRA 제외)
- 예: `Designed and built feature pipeline (HiveQL, TEZ Engine, Airflow K8s). Resolved partition scan range issues through iterative debugging.`

---

### Step 6. 최종 리포트 생성 (`report_*.md`)

레포별 `summary_*.md`와 `_work/*.json`을 집계하여 최종 리포트를 생성한다.  
**토큰 절약 원칙**: JSON → summary → raw diff 순으로 우선 활용. diff 재읽기 금지.

> Use the appropriate language for all report headings and content based on `config.language`.

#### `report_summary.md` (항상 생성)

```markdown
# Technical Work Report
> (Korean: # 기술 성과 리포트)

> 분석자: {author} | 생성일: {date} | 프로젝트: {N}개 | 작성 목적: {purposes}

## 핵심 성과 요약
- {전체 성과 1}
- {전체 성과 2}
- {전체 성과 3}

---

## Project: {repo-name}
> 레포: {repo-path} | 브랜치: {branch} | 분석 기간: {period}

### {period_key}
{summary}

---
```

**정렬 규칙:**
- 프로젝트(레포) 섹션 순서: 사용자 입력 순서 유지
- 기간 섹션 순서: 연도 내림차순 → H2 > H1

**멀티레포 전용 — 레포별 기여 비중 표** (레포 2개 이상인 경우에만 추가):
| 레포 | 기간 | 커밋 수 | 주요 기술 스택 | 비중 |
|------|------|--------|--------------|------|
| repo-a | 2026-H1 | 33 | HiveQL, TEZ, Airflow | 90% |
| repo-b | 2026-H1 | 4 | Python, FastAPI | 10% |

#### 목적별 리포트 (`report_resume.md` 등)

purposes에 해당하는 레포별 `summary_{period_key}_{purpose}.md`를 통합하여 각 `report_{목적}.md`를 생성한다.  
구조는 `report_summary.md`와 동일하며, 내용만 목적에 맞게 조정한다.

> 멀티레포 모드(`repos` 2개 이상)는 자동으로 Step 6까지 실행한다.

---

### Step 7. 마무리

1. `{config.output_dir}/work-summary/README.md` 목차 업데이트 (마지막 요약 날짜 기록 — 점진적 업데이트용)
2. `_work/` 디렉토리 삭제 여부를 사용자에게 확인 후 처리

---

## Output Format

Output format adapts to `config.language`. Use the appropriate language for section headings and report content.

Performance summary sentence format (Korean):
- **[시스템명] 신규 설계 및 구축**: [기술 스택·핵심 로직] 포함, [이슈] 연계
- **[기술명] 전환/도입**: [배경] → [개선 효과] 달성
- **[작업명] 구현 및 운영 안정화**: [반복 수정 횟수]회 수정 끝에 안정화

Performance summary sentence format (English):
- **Designed and built [System Name]**: including [tech stack · core logic], linked to [issue]
- **Introduced/migrated to [Technology]**: [background] → achieved [improvement]
- **Implemented and stabilized [Feature]**: stabilized after [N] iterative fixes

---

## Notes

- Merge 커밋은 `--no-merges` 옵션으로 제외한다.
- Revert 쌍(원본+Revert)은 Step 2에서 자동 제거한다. Revert 후 재작성 커밋은 유지한다.
- 커밋 메시지가 모호한 경우(`bug fix`, `no message`) diff의 inline 문서 패턴·주석·테이블명으로 실제 내용을 파악한다.
- **대형 커밋**(필터 후 추가 줄 300줄 이상 OR 변경 파일 10개 이상)은 Step 3-1B 단계적 추출을 적용한다. 임의 라인 상한으로 자르지 않는다.
- 삭제 줄(`-`)은 분석에서 제외하여 토큰을 절감한다.
- `user.email` 미설정 시 `user.name`으로 fallback. 동명이인 위험 시 사용자에게 확인 요청한다.
