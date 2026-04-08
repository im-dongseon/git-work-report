# git-work-report

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)

Git 커밋 히스토리를 이력서, 성과보고서, LinkedIn 포스팅으로 자동 변환합니다.

## 언어

- 한국어 (이 README_KO.md)
- English: see [README.md](./README.md)

---

## 기능

- 작성자 기준으로 커밋과 코드 변경(diff)을 분석
- 6개월 단위 기술 성과 요약서 자동 생성
- 다양한 출력 형식 지원: 기술 성과 요약, 이력서, 성과보고서, LinkedIn
- 단일 레포 및 멀티 레포 지원
- 간편 분석(Simple Mode): 커밋 메시지 기반 빠른 요약
- 점진적 업데이트: 마지막 실행 이후 신규 커밋만 분석

보고서는 6개월 단위로 생성됩니다. 이미 해당 기간에 대한 리포트가 존재하는 경우, 리포트의 메타데이터(마지막 분석일)를 기준으로 그 이후의 커밋만 분석하여 기존 리포트를 갱신합니다.

---

## 빠른 시작

### 설치

```bash
# GitHub에서 설치 (공개 배포 후)
npx skills add {username}/git-work-report -g -y

# 로컬 경로에서 설치
npx skills add ~/git-work-report -g -y
```

### 실행

AI 에이전트(GitHub Copilot, Claude Code 등)에서 아래와 같이 요청:

```
"내 커밋 히스토리를 요약해줘"
"어떤 작업을 했는지 정리해줘"
"기술 성과 요약서 만들어줘"
"이력서에 쓸 수 있게 작업 내용 정리해줘"
```

또는 단축어(shorthand) 사용:

| 단축어 | 동작 |
|--------|------|
| `gwr` | 팁 안내 후 전체 질문 시작 |
| `gwr summary` | 기술 성과 요약 모드로 바로 시작 |
| `gwr resume` | 이력서용 모드로 바로 시작 |
| `gwr performance` | 성과보고서용 모드로 바로 시작 |
| `gwr linkedin` | LinkedIn용 모드 (기본 언어: 영어) |
| `gwr simple` | 간편 분석 모드 (git log 기반, 빠름) |

---

## 산출물

결과물은 `{output_dir}/work-summary/` 아래에 저장됩니다:

```
work-summary/
├── report_summary.md          # 기본 최종 리포트 (항상 생성)
├── report_resume.md           # 이력서용 (purposes에 resume 포함 시)
├── report_performance.md      # 성과보고서용 (purposes에 performance 포함 시)
├── report_linkedin.md         # LinkedIn용 (purposes에 linkedin 포함 시)
├── {repo-name}/
│   ├── summary_{period_key}.md
│   └── _work/{period_key}/v1/
│       ├── config.json        # 재사용 판단용 fingerprint
│       ├── 00_commit_list.md
│       └── {hash}.json        # 커밋별 분석 캐시 (상세 분석 모드만)
```

### 분석 방식 비교

| 방식 | 데이터 소스 | 속도 | 정확도 |
|------|------------|------|--------|
| **상세 분석 (Full)** | git log + diff | 느림 | 높음 (코드 기반) |
| **간편 분석 (Simple)** | git log만 | 빠름 | 중간 (메시지 기반) |

---

## 요구사항

- `git` CLI
- AI 에이전트: GitHub Copilot, Claude Code, Cursor, Gemini CLI 등

---

## 설치

```bash
# GitHub에서 설치 (공개 배포 후)
npx skills add {username}/git-work-report -g -y

# 로컬 경로에서 설치
npx skills add ~/git-work-report -g -y

# 설치 확인
npx skills ls
```

---

## 라이선스

MIT — [LICENSE](./LICENSE) 참조
