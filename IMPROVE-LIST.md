# Boilerplate 개선 제안 리스트 (IMPROVE-LIST)

> 작성일: 2026-04-27
> 대상 브랜치: `main` (현재 HEAD `8ae862b`)
> 근거: Claude Code 공식 문서 조사 결과 + 현재 저장소 정적 분석
> 비범위: 실제 구현(이 문서는 설계 제안만 담는다)

---

## 0. 요약

### 0.1 한 줄 흐름 비교

| 단계 | As-is | To-be |
|------|-------|-------|
| 초기화 | `/bootstrap-project` (fork된 architect-opus, 1회 입력) | `/bootstrap-project` **메인 세션이 R0~R4 라운드를 직접 운전**, 추론 무거운 단계만 architect-opus 단발 sub-call |
| 스택 세팅 | `/bootstrap-stack` (문서만 갱신) | `/bootstrap-stack` + **`/stack-guard` 호출 → 통합 `validate` 명령·검증 스크립트 자동 생성, 단일 OS 가정일 때만 PostToolUse hook** |
| 분해 | `/plan-workitem` (지침만) | `/plan-workitem` (planner agent에 위임 fork) |
| 구현 | `/implement-workitem` | (변동 없음) |
| 검증 | `/validate-workitem` (정적 판정) | `/validate-workitem` **판정 전용** + 통합 `validate` 명령 자동 실행 |
| 보수 | (없음) | `/repair-workitem` (검증 결과를 받아 수정) |
| 마감 | (없음) | `/finalize-workitem` (status `done` + 명시적 파일 목록 add + 커밋) |
| 마일스톤 마감 | (없음) | `/wrap-milestone` (E2E + 회귀 + 리팩토링 후보 제안) |

### 0.2 우선순위 / 의존성

| # | 항목 | 우선 | 난이도 | 의존 |
|---|------|------|--------|------|
| 1 | `/bootstrap-stack` 검증 hook 자동 생성 + `/validate-workitem` 연결 | P0 | 중 | — |
| 2 | `/finalize-workitem` 신설 + `/validate-workitem` 역할 분리 | P0 | 낮 | 1 |
| 3 | `/repair-workitem` 신설 | P1 | 낮 | 2 |
| 4 | 흐름상 약한 skill 정리/강화 (`/plan-workitem`·`/review-doc` 실제 agent 바인딩 포함) | P1 | 낮 | — |
| 5 | `/wrap-milestone` (마일스톤 회수 단계) | P2 | 중 | 1, 2 |
| 6 | `/bootstrap-project` 인터랙티브 기획 강화 | P0 | 중 | — |
| 11 | `/batch` 표현 명확화 + worktree 패턴 가이드 | P1 | 낮 | — |
| 12 | 모델 ID 표기 일관성과 노화 위험 해소 | P0 | 낮 | — |

> **번호 점프 안내**: 위 표에 7~10이 없는 것은 본 문서의 7(통합 흐름)·8(체크리스트)·9(공식 문서 조사)·10(재검토 메모)이 항목이 아닌 보조 섹션이기 때문이다. 또한 원 항목 11(settings.local 모순)·12(slash command 불일치)·13(자기검증 루틴)은 재검토 사이클에서 삭제·흡수되어 사라졌다(상세는 10.2 참조).

권장 적용 순서: **12 → 11 → 4 → 2 + 3 → 1(1단계만) → 5 → 6**.

- **12** — stale 모델 ID(`claude-opus-4-6`)는 이미 1버전 뒤(`claude-opus-4-7`이 공식 최신 — [model-config 문서](https://code.claude.com/docs/en/model-config)). 즉시 실패는 아니지만 fork된 새 프로젝트에서 가장 빠르게 위험이 현실화되므로 최우선.
- **11** — `/batch`는 공식 bundled skill로 동작은 하므로 즉시 실패가 아니다. 다만 사용 가이드(bundled vs 가벼운 Agent 병렬 vs `isolation: "worktree"` 단일 격리) 분리가 필요해 우선순위는 P1로 둔다.
- **4** — 약한 skill agent 바인딩 정리. 삭제·바인딩 변경이 가벼운 정리 작업.
- **2 + 3** — finalize(2)와 repair(3)는 검증 결과 전달 메커니즘(`docs/40-validation/last-validate-*.md` 파일 + task 문서의 변경 예정 파일 섹션)을 공유하므로 동시 결정. validator-sonnet의 Write 권한 추가도 이 단계에서 함께.
- **1(1단계만)** — `/stack-guard`의 verify 스크립트 + 통합 `validate` 명령 생성. PostToolUse hook 자동 등록(2단계)은 prototyping 후 별도 항목으로 분리.
- **5** — wrap-milestone. 통합 명령과 last-validate 양식이 정해진 뒤(항목 1·2·3 적용 후)가 안전.
- **6** — bootstrap-project 인터랙티브 개편. 메인 컨텍스트 누적 트레이드오프 결정이 끝난 뒤 가장 마지막에. 이 시점에 항목 12의 bootstrap-stack/bootstrap-project model 라인은 frontmatter에서 통째로 제거된다.

### 0.3 항목 공통 형식

각 항목은 다음 7개 섹션으로 적는다.

1. 배경 / 동기
2. 현재 상태 (As-is)
3. 문제점
4. 개선안 (To-be)
5. 구현 스케치
6. 영향 범위
7. 열린 질문

### 0.4 횡단 원칙 (모든 항목에 공통 적용)

본 IMPROVE-LIST의 모든 항목 적용 시 다음을 공통으로 따른다.

1. **prototyping 선행 원칙** — 공식 문서로 동작이 확인되지 않은 메커니즘은 사용자 안내문이나 자동 산출물에 박기 전에 직접 띄워 확인한다. 본 문서 대부분의 메커니즘(skill frontmatter `arguments`/`paths`/`shell`/`hooks`, sub-agent `isolation: worktree`/자동 cleanup, `Bash(cmd *)` 패턴, AskUserQuestion 도구 등)은 [skills 문서](https://code.claude.com/docs/en/skills)·[sub-agents 문서](https://code.claude.com/docs/en/sub-agents)·[hooks 문서](https://code.claude.com/docs/en/hooks)에 공식 명시되어 있으므로 prototyping 대상이 아니다. 남은 prototyping 대상:

   - **항목 1 — PostToolUse hook의 실측 비용** — `acceptEdits` 모드에서 매 Edit/Write마다 lint 실행 시 비용 폭증 위험(공식 문서가 비용을 보장하지 않음). 항목 1.4 2단계의 핵심 측정.
   - **항목 2 — `allowed-tools`의 `Bash(cmd *)` 패턴 적용 범위** — 공식 예시(`Bash(git add *)`)는 확인되었으나, `Bash(pnpm validate)`처럼 와일드카드 없이 정확 일치만 두는 경우의 동작은 직접 확인이 필요.
2. **정책성 결정은 ADR 동반** — 본 문서가 새로 도입하는 정책(Conventional Commits 기본, 모델 별칭 우선, 워크아이템 라이프사이클 단계 정의, last-validate 파일 양식 등)은 적용 시 `docs/90-decisions/`에 ADR을 함께 추가한다. 6개월 뒤 fork한 사용자가 "왜 이 정책이 박혔는가"를 추적할 수 있어야 한다.
3. **skill chaining은 자동이 아니라 제안** — Claude Code skill은 다른 skill을 직접 호출하지 않는다. 본 문서의 흐름 그림(`/validate-workitem → /finalize-workitem` 같은 화살표)은 **skill이 텍스트로 다음 액션을 제안하면 사용자 또는 메인 세션이 받아 다음 skill을 발화**하는 흐름이다. "Pass → 자동으로 finalize 실행"이 아니라 "Pass → finalize 추천 출력 → 사용자/메인이 발화"임을 7번 그림 직전 단락에 한 번 명시한다.

---

## 1. `/bootstrap-stack`에서 스택별 검증 훅 자동 생성 + `/validate-workitem` 연결

### 1.1 배경 / 동기
`docs/00-meta/GUARDRAILS_STRATEGY.md`는 "shared 기본값에는 OS, 셸, 런타임에 강하게 의존하는 자동화를 넣지 않는다"를 명시한다. 그러나 스택이 정해진 **다음 단계**, 즉 어떤 자동화를 어떻게 만들지에 대한 실제 산출물은 비어 있다. 그 결과 검증이 사람의 기억에 의존한다.

### 1.2 As-is
- `.claude/skills/bootstrap-stack/SKILL.md`는 산출물로 `ARCHITECTURE_OVERVIEW.md`, `ADR-003-stack-selection.md`, 선택적으로 `STACK_SETUP_PLAN.md`만 약속한다.
- `scripts/verify.*`, `.claude/settings.json` hook 등록은 사람이 수동으로 한다.
- `/validate-workitem` → `validator-sonnet`은 `Read, Glob, Grep, Bash` 권한이 있지만, 통합 검증 명령이 없으므로 실제로는 **정적 판단만** 하고 있다.

### 1.3 문제점
- 스택이 정해진 직후 자동화 공백이 길고, 사람이 까먹으면 검증이 누락된다.
- "동작 검증" 책임이 어디에도 명시 위임되어 있지 않다.
- finalize / repair 자동화의 전제(통합 명령)가 없다.

### 1.4 To-be

본 항목은 두 단계로 나눠 적용한다 — verify 스크립트·통합 명령 생성은 즉시(P0), PostToolUse hook 자동 등록은 prototyping 후(P1).

**1단계 (즉시 적용, P0) — `/stack-guard`가 만드는 산출물**
- `/bootstrap-stack`의 산출물에 다음을 **명시적으로 추가**한다:
    1. 통합 진입점 — 이름은 **`validate`로 고정**한다(`pnpm validate` / `npm run validate` / `make validate` / `task validate` 중 스택에 자연스러운 단일 명령). 본 문서 흐름의 `/validate-workitem` 명칭과 통일해 사용자가 한 가지 단어만 기억하면 되도록 한다.
    2. `scripts/verify.{sh,ps1,mjs,py}` (스택에 가장 자연스러운 런타임 1종, 추가 셸은 사람이 원할 때만)
    3. cross-platform 차이가 큰 팀이면 `.claude/settings.local.json` 예시 동봉
    4. `docs/00-meta/STACK_SETUP_PLAN.md`에 **PostToolUse hook 등록 안내 문구**(설정 예시 + 매뉴얼 등록 절차)만 둔다 — 이 단계에서는 hook을 사용자 환경에 자동 생성하지 않는다.
- 이 산출물 생성은 새 sub-skill `/stack-guard`로 분리해, `/bootstrap-stack`이 그것을 호출한다.
    - `/stack-guard`는 첫 단계(R0)에서 운영 환경 가정을 명시적으로 받는다 — 단일 OS/셸인지, mixed env인지. 이 응답은 추후 2단계의 hook 자동 등록 여부를 결정하는 입력이 된다.
    - 이렇게 하면 스택이 바뀌었을 때 `/stack-guard`만 다시 돌릴 수 있다.
- `/validate-workitem` 흐름을 다음으로 바꾼다:
    1. (통합 `validate` 명령이 있으면) 자동 실행 — 결과를 stdout/stderr로 수집
    2. 결과를 validator-sonnet에 컨텍스트로 전달
    3. validator-sonnet은 정적 + 동적 결과를 함께 보고 `Pass / Needs Fix` 판정
- 통합 명령이 없으면(스택 미정 등) 1번을 건너뛰고 정적 판정만 한다 — 기존과 호환.

**2단계 (prototyping 후, 별도 항목으로 분리)**
- `.claude/settings.json`에 `PostToolUse` matcher `Edit|Write` hook 자동 등록은 본 항목에서 즉시 처방하지 않는다. 이유:
    - hook 입출력 JSON 스키마가 1.7 열린 질문에 남아 있다.
    - `settings.json:4`의 `defaultMode: "acceptEdits"`와 `PostToolUse=lint`가 결합되면 매 Edit/Write마다 lint가 자동 실행되어 비용이 폭증할 수 있다(사용자가 수락 프롬프트로 차단할 기회조차 없음).
    - 검증되지 않은 메커니즘으로 사용자 환경을 자동 변형하는 것은 0.4 횡단 원칙 1(prototyping 선행)과 동일한 유형의 위험이다.
- prototyping 단계에서 다음을 직접 확인한 뒤, 별도 항목(예: `1b. /stack-guard hook 자동 등록`)으로 분리해 본 IMPROVE-LIST에 추가한다:
    - hook 입출력 스키마와 settings.json patch 양식
    - `acceptEdits` 모드에서의 실측 비용(매 Edit당 lint 시간 × 빈도)
    - 단일 OS/셸 가정의 자동 감지 신뢰도(R0 응답이 충분한가)
- prototyping이 끝나기 전까지는 STACK_SETUP_PLAN.md의 안내 문구 + 사용자 매뉴얼 등록만으로 운영한다.

### 1.5 구현 스케치
- 신규 `.claude/skills/stack-guard/SKILL.md`
    - frontmatter: `disable-model-invocation: true`, `agent: builder-sonnet`, `context: fork`, `argument-hint: "[stack summary | empty to read existing docs]"`, `allowed-tools: Read Glob Grep Write Edit Bash`
    - 산출물: `scripts/verify.*`, `package.json`/`pyproject.toml` script 항목, `docs/00-meta/STACK_SETUP_PLAN.md`의 hook 등록 안내 문구(자동 등록 아님 — 위 2단계 참조)
- 수정 `.claude/skills/bootstrap-stack/SKILL.md`
    - "필수 수행"에 4번 항목으로 `/stack-guard` 호출 또는 동등 산출물 추가
    - `output-checklist.md`에 검증 스크립트 / 통합 명령 / hook 등록 안내 문구 항목 추가
- 수정 `.claude/skills/validate-workitem/SKILL.md`
    - "반드시 먼저 할 일"에 "통합 검증 명령이 있으면 먼저 실행한다" 추가
- 수정 `.claude/agents/validator-sonnet.md`
    - 규칙에 "동적 검증 실행 결과가 있으면 정적 판단보다 우선 본다" 추가
- 수정 `docs/00-meta/GUARDRAILS_STRATEGY.md`
    - `/stack-guard`의 산출물 범위 명시 (verify 스크립트 + 통합 `validate` 명령 + STACK_SETUP_PLAN.md의 hook 안내 문구). PostToolUse hook 자동 등록은 prototyping 후 별도 항목으로 분리한다는 정책도 함께 명시.

### 1.6 영향 범위
- 신규(저장소에 아직 없음): `.claude/skills/stack-guard/`, `scripts/verify.{sh,ps1,mjs,py}`, `docs/00-meta/STACK_SETUP_PLAN.md`(`/bootstrap-stack`이 선택 산출물로 약속하지만 빈 보일러플레이트엔 미존재)
- 수정: `.claude/skills/{bootstrap-stack,validate-workitem}/SKILL.md`, `.claude/skills/bootstrap-stack/output-checklist.md`, `.claude/agents/validator-sonnet.md`, `docs/00-meta/{GUARDRAILS_STRATEGY,WORKFLOW,NEW_PROJECT_CHECKLIST}.md`, `README.md`, `README_ko.md`

### 1.7 열린 질문

2단계 prototyping에서 측정해야 할 항목:
- `PostToolUse` 훅을 단일 OS 환경에서 매 Edit/Write마다 실행할 때의 실측 비용. 어디까지 가벼운 검증으로 둘지(format-only? lint subset?) — 잠정: format-only를 기본, lint는 finalize에서.
- `defaultMode: "acceptEdits"`와 결합 시 사용자가 비용을 차단할 기회가 없으므로 비용 상한을 어떻게 둘지(예: 첫 실패 시 hook 자동 비활성).
- hook의 입력/출력 JSON 스키마(공식 문서 기준).

기타:
- mixed env에서 hook을 강제하지 않더라도 line ending 차이는 남는다 — `.gitattributes`로 line ending 통일은 `/stack-guard` 산출물(1단계)에 포함한다.

---

## 2. `/finalize-workitem` 신설 + `/validate-workitem` 역할 분리

### 2.1 배경 / 동기
지금은 "검증 → 마감 → 커밋"이 한 명령에 묶이지 않아 사람이 매번 수동으로 status를 `done`으로 바꾸고 커밋한다. 동시에 `/validate-workitem`이 종종 검증 외 작업까지 떠안는 경향이 생긴다.

### 2.2 As-is
- `/validate-workitem` → `validator-sonnet`. 출력은 `Pass / Needs Fix` + 핵심 문제 5개.
- task 문서 status 갱신, 커밋은 사람의 몫.
- 결과적으로 워크아이템 라이프사이클의 "마감 시점"이 모호하다.

### 2.3 문제점
- 검증 통과인데 status가 `in-progress`로 남아 다음 작업과 혼선.
- 커밋 메시지가 매번 일관되지 않음.
- "Needs Fix"인데 사람이 잊고 finalize-like 작업을 해버릴 가능성.

### 2.4 To-be
새 skill `/finalize-workitem [task-id]`를 만든다. 동작:

1. 관련 task 문서 읽기
2. 통합 검증 명령(`pnpm validate` 등) 실행 — 사용자가 직전 `/validate-workitem`을 통과했더라도 한 번 더 실행한다(상태가 변했을 수 있음)
3. 실패 → `Needs Fix`로 종료, **커밋하지 않음**, 다음 액션으로 `/repair-workitem [task-id]` 추천
4. 통과 → task 문서의 `## 0. Status`를 `done`으로 갱신
5. `git status` / `git diff --stat` 기반으로 커밋 메시지 초안 생성 (Conventional Commits 스타일 기본)
6. **변경 파일을 명시적 목록으로 add 한다** — `git add -A` / `git add .`는 사용하지 않는다(.env, secrets, 빌드 산출물, untracked 임시 파일이 우발적으로 포함될 위험). 다음 우선순위로 파일 목록을 얻는다:
    - **(1순위) task 문서의 "변경 예정 파일/경로" 섹션** — `TASK_TEMPLATE.md`에 새로 추가되는 섹션(아래 2.5 참조). task 작성 시점에 명시되므로 가장 신뢰.
    - **(2순위) builder-sonnet 출력의 "수정 파일 목록"** — 구현 직후 가장 정확하지만 builder가 출력 양식을 일관되게 지킨다는 보장이 없다(`builder-sonnet.md`의 가이드 수준).
    - **(3순위) `git status --porcelain` 결과를 task 범위(1순위)와 교집합** — task 문서가 명시한 경로 화이트리스트와 실제 변경 경로의 교집합만 add. 1순위가 비어 있으면 이 단계는 건너뛰고 사용자에게 "task 문서에 변경 파일을 명시해 주세요"로 안내 후 종료.
   민감 경로(`./.env*`, `./secrets/**`, `node_modules/`, `dist/`, `build/`, `.next/`, `coverage/`)는 `.gitignore`로 이미 제외되지만, finalize skill 본문에도 "민감 경로가 staged 영역에 들어오면 즉시 종료" 가드를 추가한다.
7. `git commit -m "..."` 실행 — `--no-verify` 금지, `--amend` 금지, `git push` 금지(사용자 명시 요청 필요)
8. 마지막에 커밋 해시·메시지·갱신된 status 요약 반환

`/validate-workitem`은 **판정 전용**으로 명확히 좁힌다 — 자동 수정·자동 마감 금지.

### 2.5 구현 스케치
- 신규 `.claude/skills/finalize-workitem/SKILL.md`
    - frontmatter:
        - `disable-model-invocation: true`
        - `agent: builder-sonnet`
        - `context: fork`
        - `argument-hint: "[task or feature identifier]"`
        - `allowed-tools: Read Glob Grep Write Edit Bash(git add *) Bash(git status *) Bash(git diff *) Bash(git commit *) Bash(pnpm validate) Bash(npm run validate) Bash(make validate)`
    - **note**: `Bash(cmd *)` 패턴 문법은 공식 [skills 문서](https://code.claude.com/docs/en/skills)에 `Bash(git add *) Bash(git commit *)` 예시로 확정. 단, 와일드카드 없는 정확 일치(`Bash(pnpm validate)`)의 동작은 0.4 횡단 원칙 1에 따라 prototyping에서 확인. 동작이 다르면 (a) wildcard로 통일(`Bash(pnpm *)`)하거나 (b) settings.json `permissions.allow`로 옮긴다.
    - 본문: 위 8단계를 순서대로 명시. 실패 시 어떤 출력을 어떻게 요약할지 포함. add 파일 목록 우선순위(task 문서 1순위 → builder 출력 2순위 → 교집합 3순위) 명시.
    - 가드: 작업 트리에 staged/unstaged 변경이 없으면 "변경 없음" 종료. `git add -A` / `git add .` 금지. 민감 경로 staged 시 즉시 종료. `--amend` 금지. `--no-verify` 금지. `git push` 호출 금지(사용자 명시 요청 필요).
- 수정 `docs/30-workitems/_templates/TASK_TEMPLATE.md`
    - 새 섹션 추가: `## 4-1. 변경 예정 파일/경로` — 구현 시점에 작성. finalize의 add 화이트리스트로 사용된다. 처음 작성 시 비어 있을 수 있으며, builder-sonnet이 구현 후 자동 채울 수도 있다.
- 수정 `.claude/skills/validate-workitem/SKILL.md`
    - 첫 줄에 "이 skill은 **판정 전용**이다. status 변경, 코드 수정, 커밋은 하지 않는다." 명시
    - 마지막 출력에 다음 액션 제안 추가: 통과 → `/finalize-workitem`, 실패 → `/repair-workitem`
- 수정 `.claude/agents/validator-sonnet.md`
    - 규칙 첫 줄에 "구현이나 status 갱신, 커밋을 직접 수행하지 않는다." 명시
- 수정 `.claude/agents/builder-sonnet.md`
    - finalize 위임을 받았을 때의 가드(작업 트리·민감 파일·`--no-verify` 금지) 추가
    - 구현 완료 후 task 문서의 `## 4-1. 변경 예정 파일/경로` 섹션을 갱신하라는 규칙 추가.
- 신규 `docs/90-decisions/ADR-00x-commit-convention.md` (권장)
    - Conventional Commits 기본 채택을 ADR로 보존(정책성 결정).

### 2.6 영향 범위
- 신규: `.claude/skills/finalize-workitem/`, ADR-00x-commit-convention.md(권장)
- 수정: `.claude/skills/validate-workitem/SKILL.md`, `.claude/agents/{validator-sonnet,builder-sonnet}.md`, `docs/30-workitems/_templates/TASK_TEMPLATE.md`
- 문서: `WORKFLOW.md`, `AGENT_EXECUTION_STRATEGY.md`, `README.md`, `README_ko.md`

### 2.7 열린 질문
- 커밋 메시지 컨벤션을 Conventional Commits 고정으로 강제할지, charter에서 선택하게 할지(잠정: 기본 Conventional Commits, charter에서 override 가능)
- finalize 안에서 통합 명령을 한 번 더 돌리는 비용 vs `/validate-workitem` 결과를 신뢰하는 안전성 — 보수적으로 한 번 더 돌리는 쪽 추천.
- 멀티 task 묶음 커밋은 어떻게 다룰지(잠정: `/finalize-workitem T-001 T-002` 형태로 다중 ID 허용, 메시지에 모두 명시)

---

## 3. `/repair-workitem` 신설

### 3.1 배경 / 동기
검증 실패 시 "어디를 어떻게 고칠지"는 validator-sonnet이 이미 분석해 준다. 그 분석을 받아 builder-sonnet에 일관되게 위임할 진입점이 필요하다.

### 3.2 As-is
검증 실패 시 사용자가 수동으로 builder-sonnet에 다시 가거나, `/implement-workitem`을 다시 부른다. 하지만 `/implement-workitem`은 task 문서 기반의 "처음부터" 구현이므로 의미가 다르다.

### 3.3 문제점
- repair는 task 문서 + **직전 검증 결과**라는 두 입력을 가진다. 이를 표현하는 진입점이 없다.
- 사용자가 "validator의 5개 지적 중 4개만 고쳐"처럼 부분 지정하는 자연스러운 방법이 없다.

### 3.4 To-be
신규 `/repair-workitem [task-id] [optional notes]`.

**검증 결과 전달 메커니즘** — `/repair-workitem`은 `context: fork`로 위임되므로 fork된 sub-agent는 메인 세션의 prior conversation을 볼 수 없다. 즉 "최근 validate 결과를 컨텍스트에서 회수"는 동작하지 않는다. 두 메커니즘 중 하나를 명시적으로 사용한다:
- **(A) 파일 기반(기본)**: `/validate-workitem`이 판정 결과를 `docs/40-validation/last-validate-<task-id>.md`로 저장한다(파일 경로는 task-id 단위로 덮어쓴다 — 한 task의 가장 최근 검증 1회만 남긴다). `/repair-workitem`은 이 파일을 읽어 지적 목록을 받는다. 파일이 없거나 stale(파일 mtime이 task 변경보다 오래됨)하면 사용자에게 `/validate-workitem` 선행을 안내하고 종료.
- **(B) 인자 기반(선택)**: 사용자가 `/repair-workitem T-001 "validator의 P0 #1, P1 #3만 고쳐"`처럼 `$ARGUMENTS`로 직접 지적을 넘긴다. 이 경우 (A) 파일이 없어도 동작한다.

동작:
1. task 문서 읽기
2. 위 메커니즘 (A) 또는 (B)로 지적 사항 회수 — 둘 다 없으면 사용자에게 안내하고 종료
3. 지적 사항을 우선순위(P0 > P1 > P2)로 정렬
4. builder-sonnet에 fork 위임 — 입력은 task 문서 + 지적 목록(파일 경로 또는 인라인) + (있으면) 사용자 추가 메모
5. 출력은 수정 파일 목록, 어떤 지적을 어떻게 해소했는지, 미해결 지적, 다음 추천 액션(보통 `/validate-workitem` 또는 `/finalize-workitem`)

### 3.5 구현 스케치
- 신규 `.claude/skills/repair-workitem/SKILL.md`
    - frontmatter: `disable-model-invocation: true`, `agent: builder-sonnet`, `context: fork`, `argument-hint: "[task id] [optional notes]"`, `allowed-tools: Read Glob Grep Write Edit Bash`
    - 본문: 4번 단계의 입력 구조 명시. 메커니즘 (A)/(B)를 어떻게 식별하는지 명시 — 우선 `$ARGUMENTS`에 인라인 지적이 있으면 (B)로 동작, 없으면 `docs/40-validation/last-validate-<task-id>.md`를 읽어 (A)로 동작, 둘 다 없으면 `/validate-workitem` 선행 안내.
    - 가드: 범위 밖 파일 수정 금지, 새 기능 추가 금지(지적 해소만), 가능하면 한 번에 한 P0/P1만 처리하고 다음 라운드 추천.
- 수정 `.claude/skills/validate-workitem/SKILL.md`
    - 판정 결과를 표준 양식으로 `docs/40-validation/last-validate-<task-id>.md`에 저장하는 단계를 마지막에 추가(allowed-tools에 `Write` 추가 필요 — 현재 `Read Glob Grep Bash`).
    - 마지막 출력 — "Needs Fix"이면 다음 액션을 `/repair-workitem`으로 명시.
- 신규 `docs/40-validation/.gitignore` 또는 루트 `.gitignore`에 `docs/40-validation/last-validate-*.md` 추가 — 검증 산출물은 ephemeral이므로 커밋 대상이 아니다.
- 수정 `.claude/agents/validator-sonnet.md`
    - tools에 `Write` 추가(`Read, Glob, Grep, Bash` → `Read, Glob, Grep, Bash, Write`).
    - 규칙에 "판정 결과를 `docs/40-validation/last-validate-<task-id>.md`에 표준 양식으로 저장한다"는 한 줄 추가.

### 3.6 영향 범위
- 신규: `.claude/skills/repair-workitem/`
- 수정: `.claude/skills/validate-workitem/SKILL.md`, `.claude/agents/validator-sonnet.md`, `.gitignore`
- 문서: `WORKFLOW.md`, `AGENT_EXECUTION_STRATEGY.md`, `README.md`, `README_ko.md`

### 3.7 열린 질문
- 메커니즘 (A) 파일 경로가 task-id 단위로 덮어써지는데, 멀티 task 묶음 검증에선 어떻게? (잠정: `/validate-workitem`은 1 task = 1 파일을 유지, 묶음 검증은 별도 흐름으로)
- repair → validate → repair 무한 루프 방지 가드 필요(잠정: 한 task당 연속 3회 이상 repair 시 사용자 확인을 요구하는 안내 문구)

---

## 4. 흐름상 약하거나 중복인 skill 정리/강화

### 4.1 배경 / 동기
이 저장소의 정체성은 "메인 세션은 오케스트레이션, 실작업은 서브에이전트 위임"이다. 그렇다면 자주 쓰는 slash command는 실제로 agent workflow를 타야 한다. 그러나 일부 skill은 sub-agent를 호출하지 않고 단순히 "메인 세션에 지침을 주는 본문"만 갖고 있어, 실제 위임 효과 없이 컨텍스트만 차지한다. `README.md`, `WORKFLOW.md`, `AGENT_EXECUTION_STRATEGY.md`는 이런 skill들도 공식 사용 경로처럼 안내하므로, 같은 "slash command"라도 어떤 것은 실제 위임이 되고 어떤 것은 단순 안내문이라 사용 경험이 불균일하다. 특히 `/plan-workitem`은 공식 흐름의 중심 단계인데 실제 위임이 일어나지 않는다.

### 4.2 As-is
| Skill | agent 바인딩 | 본문 성격 | 중복/약점 |
|-------|-------------|-----------|----------|
| `boilerplate-context` | 없음 (auto-context, `user-invocable: false`) | 저장소 안내 | 그대로 유지 |
| `bootstrap-project` | architect-opus, fork | 강함 | (항목 6에서 강화) |
| `bootstrap-stack` | architect-opus, fork | 강함 | (항목 1에서 강화) |
| `plan-workitem` | **없음** | 짧은 지침문 | `planner` agent와 역할 중복, 위임 효과 0 |
| `implement-workitem` | builder-sonnet, fork | 강함 | OK |
| `validate-workitem` | validator-sonnet, fork | 강함 | (항목 2에서 강화) |
| `write-charter` | **없음** | 짧은 지침문 | `bootstrap-project`가 이미 charter를 만든다. 단독 의미 약함 |
| `write-architecture` | **없음** | 짧은 지침문 | `bootstrap-project`/`bootstrap-stack`이 이미 갱신. 단독 의미 약함 |
| `review-doc` | **없음** | 짧은 지침문 | `reviewer` agent와 역할 중복 |

### 4.3 문제점
- agent 바인딩이 없는 skill은 사실상 "지침 한 페이지"라서 사용자가 슬래시로 부르는 의미가 약하다.
- 동일한 일을 두 진입점(skill vs agent)에서 부를 수 있어 흐름이 흐려진다.

### 4.4 To-be
- **유지·강화** (agent 바인딩 추가) — "공식 slash command는 실제 동작을 가져야 한다"는 원칙으로 정리:
    - `plan-workitem`: `agent: planner`, `context: fork`, `argument-hint: "[milestone or feature id]"`로 강화. 본문은 분해 절차 가이드를 그대로 두되 planner에 위임되도록 한다.
    - `review-doc`: `agent: reviewer`, `context: fork`, `argument-hint: "[doc path]"`로 강화.
- **삭제**:
    - `write-charter` — `/bootstrap-project` 이후 charter 단독 갱신은 자연어 + planner subagent로 충분. 보일러플레이트의 "필수 진입점" 가치보다 혼선 비용이 큼.
    - `write-architecture` — 동일 이유. 스택 변경은 `/bootstrap-stack`(혹은 항목 1의 `/stack-guard`)으로 흡수.

**mid-project 갱신 동선** — charter/architecture는 `WORKFLOW.md`에서 Living Doc로 분류돼 있어 진행 중 재진입이 필요하다. 두 skill을 삭제한 뒤의 경로:

| 갱신 종류 | 경로 |
|----------|------|
| charter 부분 갱신 | 자연어로 메인 세션에 변경 요청 → `planner` agent에 fork 위임 |
| charter 전면 재정의 | `/bootstrap-project` 재실행 (R0~R4가 기존 charter를 입력으로 받아 변경분만 반영 — 항목 6의 R4 sub-call이 담당) |
| architecture 스택 변경 | `/bootstrap-stack` 재실행 |
| architecture 시스템 경계만 갱신 | 자연어 + `architect-opus` 단발 호출 |

이 동선은 `WORKFLOW.md`의 "5. 마일스톤 회수"(항목 5) 또는 별도 단락에 명시되어야 한다 — 사용자가 두 skill 삭제로 갱신 경로를 잃었다고 오해하지 않도록.

### 4.5 구현 스케치
- 수정 `.claude/skills/plan-workitem/SKILL.md`
    - frontmatter에 `agent: planner`, `context: fork`, `disable-model-invocation: true`, `argument-hint: "[milestone or feature id]"`, `allowed-tools: Read Glob Grep`
    - 본문 첫 줄: "입력 ID에 해당하는 charter/architecture/상위 workitem을 읽고 milestone/feature/task로 분해한다." — `context: fork` 사용 시 본문이 **명령형 절차**여야 한다는 공식 경고(skills 문서)에 따라, 분해 단계의 입력·산출물·종료 조건까지 actionable하게 적는다.
- 수정 `.claude/skills/review-doc/SKILL.md`
    - frontmatter에 `agent: reviewer`, `context: fork`, `disable-model-invocation: true`, `argument-hint: "[doc path]"`, `allowed-tools: Read Glob Grep`
    - 본문은 "입력 경로의 문서를 읽고 모순/누락/모호함을 찾아 우선순위로 보고" 같은 명령형 절차로 작성(plan-workitem과 동일한 이유).
- 삭제 `.claude/skills/write-charter/`, `.claude/skills/write-architecture/`
- 수정 `README.md`, `README_ko.md`, `docs/00-meta/WORKFLOW.md`, `docs/00-meta/AGENT_EXECUTION_STRATEGY.md` — 표에서 두 skill 제거, plan/review의 agent 바인딩을 명시

### 4.6 영향 범위
- 삭제 2 skill, 수정 2 skill, 문서 4종 갱신.

### 4.7 열린 질문
- `plan-workitem`을 fork 시키면 메인 세션이 분해 결과만 받는다 — 사용자가 회의 중 분해 과정을 보고 싶으면 fork를 끄는 옵션이 필요한가? (잠정: 기본 fork, 본문에 "메인이 직접 분해하길 원하면 fork 없이 planner agent를 호출"이라는 안내 한 줄)

---

## 5. 마일스톤 회수 단계 — `/wrap-milestone`

### 5.1 배경 / 동기
마일스톤은 여러 task의 합이다. 각 task가 finalize되었더라도, 마일스톤 단위에서 다음을 한 번 더 회수하지 않으면 누적 회귀와 부채가 쌓인다:
- end-to-end 흐름 검증
- 클린 코드 / 클린 아키텍처 관점 리팩토링 후보 도출
- 누락된 ADR / 문서 갱신
- QA 회귀 점검

### 5.2 As-is
이 단계는 어디에도 명시되어 있지 않다. 사람이 기억하면 한다.

### 5.3 문제점
- 마일스톤 종결 시점이 모호하다.
- 부채(리팩토링 후보, 문서 갱신, ADR 누락)가 상시 누적된다.
- task 단위 검증만으로는 통합 시점에 발견되는 회귀를 잡지 못한다.

### 5.4 To-be
신규 `/wrap-milestone [milestone-id]`.

동작:
1. milestone 문서를 읽고 포함된 feature/task 목록을 회수
2. 각 task의 status를 점검 — `done`이 아닌 항목이 있으면 명단을 출력하고 종료(완료를 강제하지 않음)
3. 통합 `validate` 명령 실행 + (있으면) E2E 명령 실행
4. `qa` agent에 회귀·엣지케이스 점검 위임 — qa는 **보고만** 한다(현재 `qa.md`의 tools에 Write가 없음). 반환된 보고를 `/wrap-milestone`이 받아 `docs/40-validation/QA_FINDINGS.md`에 누적 기록한다.
5. `reviewer` agent에 리팩토링 후보·아키텍처 부채 점검 위임 — 동일하게 reviewer는 보고만 하고, `/wrap-milestone`이 받아 `docs/40-validation/IMPROVEMENT_GUIDE.md`에 정리. 구조 변경이 필요해 보이면 메인 세션 가이드에 따라 architect-opus를 추가 호출.
6. 미흡한 ADR 후보 제안 (예: 마일스톤 중에 내려진 결정인데 ADR이 없는 것)
7. 최종 출력: 통합 결과 / P0·P1·P2 후속 작업 / 다음 마일스톤으로 넘기는 항목

**책임 경계**: 본 skill의 "자동 수정/커밋 금지"는 **코드 변경에 한정**한다. 검증 결과를 누적 기록하는 문서 갱신(`QA_FINDINGS.md`, `IMPROVEMENT_GUIDE.md`)은 이 skill의 정상 책임이며, 코드 수정·커밋·workitem status 변경은 금지된다. 후속 작업은 `/repair-workitem` 또는 새 task로 연결된다.

### 5.5 구현 스케치
- 신규 `.claude/skills/wrap-milestone/SKILL.md`
    - frontmatter: `disable-model-invocation: true`, `argument-hint: "[milestone id]"`, `allowed-tools: Read Glob Grep Write Edit Bash` — qa/reviewer가 Write 권한이 없으므로 누적 기록은 이 skill이 책임진다(Write/Edit 필요).
    - 본문: 위 7단계. 각 단계별로 어떤 agent/명령을 호출하는지 명시. qa/reviewer는 보고만 하고, 이 skill이 받아 누적 기록한다는 책임 경계를 본문에도 명시.
    - 비범위: 코드 수정·커밋·workitem status 변경 금지(다른 skill을 통해 후속 처리).
- 수정 `docs/00-meta/WORKFLOW.md`
    - "5. 마일스톤 회수" 섹션 신설
- 수정 `docs/40-validation/QA_FINDINGS.md`
    - 마일스톤 단위 누적을 위해 최상위 헤더 구조를 변경한다 — 기존 `## P0 / P1 / P2 / 관찰 메모` 단일 묶음을 `## M1 → ### P0/P1/P2/관찰 메모`, `## M2 → ###...` 식으로 마일스톤별로 중첩한다. 마일스톤이 정해지지 않은 초기 프로젝트는 `## 일반` 한 묶음만 둔다.
- 수정 `docs/40-validation/IMPROVEMENT_GUIDE.md`
    - 현재 `## 0. 요약 / 1. 우선순위 / 2. 즉시 수정할 항목 / 3. 권장 리팩토링 / 4. 보류 항목` 양식을 유지하되, 각 섹션 안에서 `### M1`, `### M2` 식의 마일스톤 단위 그룹핑을 권장하는 한 단락을 문서 상단에 추가한다. 양식 자체는 유지해서 기존 사용자에게 호환된다.
- **다운스트림 마이그레이션 가이드** — 본 보일러플레이트는 빈 템플릿이라 적용 시 즉시 변경 가능. 그러나 본 보일러플레이트로 시작한 다운스트림 프로젝트는 이미 누적된 데이터를 갖고 있을 수 있다. 이 경우 마이그레이션은 두 단계: (1) 기존 평면 항목들을 `## M1` 또는 `## 일반` 한 묶음으로 감싸기(편집 1회). (2) 다음 회차부터 새 마일스톤 헤더로 누적. WORKFLOW.md의 "5. 마일스톤 회수" 섹션 마지막에 이 마이그레이션 한 단락을 추가한다.

### 5.6 영향 범위
- 신규 1 skill, 문서 3종 갱신.

### 5.7 열린 질문
- 5단계의 리팩토링 후보 도출은 architect-opus(비용↑) vs reviewer(비용↓) 어느 쪽이 적절한가? (잠정: reviewer 기본, "구조 변경이 필요해 보이면 architect-opus 추가 호출"이라는 메인 세션 가이드)
- E2E 명령이 없는 스택은 4번에서 어떻게? (잠정: 통합 명령만 돌리고 E2E는 skip, 출력에 명시)
- 이 skill이 너무 많은 일을 한다 — sub-skill로 쪼갤 가치는? (잠정: 마일스톤은 자주 일어나지 않으므로 단일 skill이 OK. 비용 폭주 시 분해 검토)

---

## 6. `/bootstrap-project` 인터랙티브 기획 강화

### 6.1 배경 / 동기
현재 `/bootstrap-project`는 한 번의 자연어 입력으로 charter/architecture/M1/F-001을 한꺼번에 만든다. 입력이 짧으면 결과가 얕아지고, 페르소나·pain point·시나리오가 추측으로 채워진다. 사용자가 표현한 pain은 "기획 단계가 부자연스럽다 / 구체성이 부족하다".

### 6.2 As-is
- `bootstrap-project/SKILL.md`는 `context: fork`, `agent: architect-opus`, `effort: max`로 위임된다.
- 본문 절차: 입력 구조화 → 문서 갱신 → 초기 workitem 생성.
- 사용자에게 되묻는 단계가 없음.
- `brief-template.md`가 있지만 사람이 자발적으로 채워와야 효과가 난다.

### 6.3 문제점
- 한 번 입력으로 끝내면 페르소나·문제·시나리오가 얕다.
- 가정과 추측의 경계가 흐려져 charter의 신뢰도가 낮아진다.
- 가장 큰 pain point를 찾는 과정 없이 바로 범위가 정해지는 경향.
- **메커니즘 제약**: 현재 frontmatter는 `context: fork`로 architect-opus에 위임한다. fork된 sub-agent는 결과를 한 번만 메인에 반환하므로 라운드별 사용자 인터랙션이 sub-agent 안에서는 일어날 수 없다. 또한 [`AskUserQuestion`](https://code.claude.com/docs/en/agent-sdk/user-input) 도구는 공식 지원이지만 sub-agent의 `tools` 필드를 명시한 경우(architect-opus는 `Read, Glob, Grep, Write`로 제한) 별도로 추가하지 않으면 호출 불가. 따라서 인터랙티브 강화는 **fork 모델 자체를 바꾸거나 sub-agent tools에 `AskUserQuestion`을 추가하지 않으면 동작하지 않는다**(본 문서 6.4는 전자를 택한다).

### 6.4 To-be (메커니즘 + 라운드)

**메커니즘 변경 — 메인 세션이 라운드를 직접 운전 + 임시 파일로 컨텍스트 격리**

- `bootstrap-project` skill의 frontmatter에서 `context: fork`와 `agent: architect-opus`를 **제거**한다. 즉 이 skill은 메인 세션에 R0~R4 절차를 기재한 "절차서"로 바뀐다.
- 메인 세션이 각 라운드의 사용자 응답을 직접 받는다. **R0~R3의 라운드 산출물(페르소나 카드, pain 표, 시나리오, 범위, 가정)은 메인 컨텍스트에 누적시키지 않고 임시 파일 `docs/00-meta/.bootstrap-session-<timestamp>.md`에 누적 적재**한다. 메인 세션은 라운드 진행에 필요한 최소 요약만 유지한다.
- 무거운 추론(R0의 페르소나 후보, R1의 pain inventory)은 **architect-opus 단발 sub-call로 위임**한다 — sub-call은 임시 파일과 task 입력을 컨텍스트로 받아 결과만 메인으로 돌려준다.
- 마지막 R4(산출물 일괄 생성)는 architect-opus 단발 sub-call로 위임하되, **입력은 임시 파일 1개**다. sub-call은 임시 파일을 읽어 charter/architecture/M1/F-001을 한꺼번에 만든다. 이렇게 하면 R0~R3의 누적 컨텍스트가 R4 sub-call의 fork 격리 안에서만 다뤄지고, 메인 세션은 가벼움을 유지한다.
- R4 완료 후 임시 파일은 기본 위치(`docs/00-meta/.bootstrap-session-*.md`, `.gitignore`로 git 추적 제외)에 그대로 보존한다 — 추후 charter 리뷰 시 추적용. 사용자가 원하면 `docs/30-workitems/plans/` 같은 추적 가능 위치로 이동(이름 변경 권장)하거나 삭제한다.
- 본문 어딘가에 **트레이드오프 단락**을 명시한다 — "이 skill은 의도적으로 fork 격리를 풀고 메인 세션이 사용자 인터랙션을 직접 운전한다. R0~R3 산출물은 임시 파일에 적재해 메인 컨텍스트 부담을 최소화하지만, 라운드 요약 일부는 메인에 남는다. `AGENT_EXECUTION_STRATEGY.md`의 '메인 컨텍스트에 오래 보존하지 않는다' 원칙과의 트레이드오프이며, 종료 후 사용자가 `/clear` 또는 새 세션으로 컨텍스트를 정리할 것을 권장한다."

**라운드 구성(5단계로 압축)** — 사용자가 매 라운드 끝에 `skip` / `good` / `refine: ...`로 응답할 수 있다. 각 라운드 종료 후 결과는 임시 파일에 append된다.

1. **R0 — 문제 한 줄 + 페르소나 확인**
   - 입력 한 줄을 그대로 되돌려 "이 한 문장이 핵심 맞나" 확인.
   - 동시에 페르소나 후보 2~3개를 한 단락씩 제시(직무·맥락·일과·압력 요소). 사용자가 1개를 고르거나 합쳐달라고 한다.
   - 추론이 무거우므로 페르소나 후보 생성은 architect-opus 단발 sub-call로 위임.
2. **R1 — 핵심 pain + 시나리오**
   - 선택된 페르소나의 pain 6~10개를 brainstorm 후 빈도×고통으로 정렬, 상위 1~3개를 사용자가 고른다.
   - 가장 큰 pain의 happy path·alternate path·fail path를 5~7단계로 같이 적는다. 사용자가 끊을 지점·수용 가능 fail을 정한다.
   - architect-opus 단발 sub-call로 위임.
3. **R2 — 초기 범위 vs 비범위 / 성공 기준**
   - R1 시나리오를 충족시키는 최소 기능 묶음과 의도적으로 미루는 것을 분리.
   - 측정 가능한 성공 기준 1~3개.
4. **R3 — 핵심 가정 + 열린 질문**
   - R0~R2에서 사용자가 추측으로 답한 모든 항목을 가정으로 표시.
   - 가장 위험한 가정 1~3개에 검증 방법 1줄.
5. **R4 — 산출물 일괄 생성**
   - 임시 파일을 입력으로 charter, architecture(시스템 경계 정도), M1-foundation, F-001-core-value를 만든다. 페르소나/시나리오/가정/열린 질문을 charter에 그대로 박는다.
   - architect-opus 단발 sub-call로 위임 (effort: max).

**fast 모드** — `/bootstrap-project --fast "<설명>"`은 R0과 R3을 건너뛰고 R1+R2+R4만 도는 단축 흐름. 시간 없는 prototype 단계에서 사용한다.

**사용자 응답 수단 결정** — 라운드별 사용자 응답은 **자연어로만** 받는다. [`AskUserQuestion`](https://code.claude.com/docs/en/agent-sdk/user-input)은 공식 도구지만(9.5 참조) 본 안에선 채택하지 않는다 — 메인 세션이 직접 라운드를 운전하므로 사용자와 자연어로 직접 소통할 수 있고, 다중 선택지 UX의 추가 가치가 비용 대비 불명확하기 때문. 향후 인터랙티브 UX 개선 시 별도 항목으로 채택 검토.

### 6.5 구현 스케치
- 수정 `.claude/skills/bootstrap-project/SKILL.md`
    - frontmatter에서 `context: fork`, `agent: architect-opus`, `model: claude-opus-4-6`, `effort: max` 제거. `disable-model-invocation: true`, `argument-hint`, `allowed-tools: Read Glob Grep Write Edit`만 남긴다.
    - 본문 전면 개정: R0~R4 절차로 재작성.
    - 각 라운드의 출력 양식을 명시(페르소나 카드 포맷, pain 표, 시나리오 단계 표).
    - "메인 세션이 라운드를 직접 운전 + R0~R3 결과를 임시 파일 `docs/00-meta/.bootstrap-session-<timestamp>.md`에 적재 + 무거운 추론은 architect-opus 단발 sub-call로 위임 + R4는 임시 파일을 입력으로 sub-call" 메커니즘을 본문에 명시. 단발 sub-call 시 어떤 입력·출력 형식을 쓰는지 한 단락.
    - 트레이드오프 단락 명시(메인 컨텍스트에 라운드 요약 일부 누적, `AGENT_EXECUTION_STRATEGY.md`의 원칙과의 의도적 트레이드오프, 종료 후 `/clear` 권장).
    - 사용자 응답은 자연어로 받는다(`AskUserQuestion` 미채택 결정 명시).
- 신규 `.claude/skills/bootstrap-project/persona-template.md`
    - 페르소나 카드 양식(역할/맥락/하루 흐름/현재 해결 방법/실패 패턴/측정 가능한 성공 신호)
- 신규 `.claude/skills/bootstrap-project/pain-template.md`
    - pain 표(빈도/고통/현재 우회 방법/해결 시 효과)
- 수정 `.claude/skills/bootstrap-project/output-checklist.md`
    - "필수 갱신 문서"에 추가: charter에 페르소나/JTBD/시나리오 섹션이 채워졌는가, 가정·열린 질문이 명시되었는가
- 수정 `docs/10-charter/PROJECT_CHARTER.md` (template)
    - 섹션 추가: "2.1 페르소나", "3.1 핵심 시나리오 (happy/alt/fail)", 기존 "9. 열린 질문" 위에 "8.1 핵심 가정"
- 수정 `docs/00-meta/BOOTSTRAP_PROMPT_EXAMPLES.md`
    - 기존 예시들에 R0~R4 라운드 인터랙션이 어떻게 흐르는지 보여주는 예시 1개 추가(현재 4개 예시는 단발 입력 형식이라 새 메커니즘 흐름이 보이지 않음).
- 수정 `.gitignore`
    - `docs/00-meta/.bootstrap-session-*.md` 추가(세션 임시 파일은 커밋 대상 아님).
- 수정 `.claude/skills/bootstrap-stack/SKILL.md`
    - frontmatter에서 `context: fork`, `agent: architect-opus`, `effort: max`, `model` 라인 제거(bootstrap-project와 동일).
    - 본문은 같은 메커니즘으로 다듬는다 — 라운드 구성: R0(스택 의도 + 운영 환경 가정: 단일 OS/셸 vs mixed env) → R1(핵심 비기능 요구사항) → R2(최종 스택 + 가정/제약) → R3(`/stack-guard` 호출). 항목 1과 결합되며 메커니즘 상세는 본 항목(6.4)을 따른다.
    - **항목 12와의 정합**: frontmatter에서 `model` 라인이 사라지므로 항목 12의 별칭 처방은 이 skill에 적용되지 않는다.

### 6.6 영향 범위
- 수정 2 skill 본문 + frontmatter (`bootstrap-project`, `bootstrap-stack` — 둘 다 `context: fork`/`agent`/`model`/`effort` 라인 제거)
- 신규 보조 템플릿 2개(`persona-template.md`, `pain-template.md`), 수정 charter 템플릿, 수정 BOOTSTRAP_PROMPT_EXAMPLES, 수정 README 입력 팁 섹션
- `.gitignore`에 `docs/00-meta/.bootstrap-session-*.md` 추가
- 항목 1(`/stack-guard` 신설)과 결합 정리

### 6.7 열린 질문
- 페르소나가 2명 이상으로 늘어날 때 charter 양식을 어떻게 확장할지(잠정: 기본 1명 우선, 2명 이상이면 페르소나별 시나리오 섹션을 분리)
- 다국어 — 한국어 사용자가 많지만 영문 입력도 들어오면 산출물 언어를 어떻게 결정할지(잠정: 입력 언어를 따른다는 규칙을 본문에 명시)
- 단발 sub-call 비용 — R0/R1/R4 세 번 architect-opus를 호출하는 비용이 기존 한 번 fork보다 클 수 있음. 첫 사용 후 측정해 비용이 크면 R0/R1을 reviewer/planner로 강등하는 옵션 검토.

---

## 7. 통합 흐름 (적용 후 그림)

> 아래 화살표는 **자동 호출이 아니라 제안→발화** 흐름이다(0.4 횡단 원칙 3 참조). 각 skill은 다음 액션을 텍스트로 제안하고, 사용자 또는 메인 세션이 받아 다음 skill을 발화한다.

```
/bootstrap-project (메인 세션이 R0~R4 라운드 운전, R0~R3 결과는 임시 파일 적재, R4는 architect-opus 단발 sub-call)
    ↓
/bootstrap-stack ─→ /stack-guard (R0=운영 환경 가정 확인, 통합 `validate` 명령·검증 스크립트 자동 생성, 단일 OS 가정일 때만 PostToolUse hook)
    ↓
/plan-workitem (planner agent fork) — milestone/feature/task 분해
    ↓
/implement-workitem (builder-sonnet)
    ↓
/validate-workitem (validator-sonnet, 판정 전용; 통합 `validate` 명령 자동 실행)
    ├─ Pass → /finalize-workitem (status=done + 명시적 파일 목록 add + 커밋)
    └─ Needs Fix → /repair-workitem → /validate-workitem 재실행
                                                        ↑
                                                        └ 무한 루프 가드(연속 3회 시 사용자 확인)

마일스톤의 모든 task가 done이 되면:
/wrap-milestone (E2E + 회귀 + 리팩토링 후보 + ADR 점검)
    ↓
다음 마일스톤으로 진행 또는 /repair-workitem 후속 작업 큐에 적재
```

---

## 8. 문서 반영 체크리스트 (적용 시)

**문서 / 흐름 갱신**
- [ ] `README.md`, `README_ko.md` — 흐름 그림과 step 설명을 7번 그림 기준으로 갱신, `/batch` 사용 가이드(bundled 명시) 추가, mid-project 갱신 동선 한 단락 추가(항목 4)
- [ ] `CLAUDE.md` — 위임 트리거 표에서 병렬 패턴 3종으로 분리 (항목 11)
- [ ] `docs/00-meta/WORKFLOW.md` — `/finalize-workitem`, `/repair-workitem`, `/wrap-milestone` 단계 추가, charter/architecture mid-project 갱신 동선 단락 추가
- [ ] `docs/00-meta/AGENT_EXECUTION_STRATEGY.md` — 위임 트리거 표에 finalize/repair/wrap 행 추가, 병렬 패턴 3종(Agent 병렬 / `isolation: "worktree"` / bundled `/batch`) 단락 추가, 모델 표기 정책 단락 추가 (항목 11·12)
- [ ] `docs/00-meta/NEW_PROJECT_CHECKLIST.md` — `/bootstrap-project`가 인터랙티브로 바뀐 점 반영
- [ ] `docs/00-meta/GUARDRAILS_STRATEGY.md` — `/stack-guard`의 1단계 산출물 범위(verify 스크립트 + 통합 `validate` 명령 + STACK_SETUP_PLAN.md hook 안내), PostToolUse hook 자동 등록은 prototyping 후 별도 항목으로 분리 명시
- [ ] `docs/10-charter/PROJECT_CHARTER.md` (template) — 페르소나/시나리오/가정 섹션 추가
- [ ] `docs/30-workitems/_templates/TASK_TEMPLATE.md` — `## 4-1. 변경 예정 파일/경로` 섹션 추가(항목 2)
- [ ] `docs/40-validation/{QA_FINDINGS,IMPROVEMENT_GUIDE}.md` — 마일스톤 단위 누적 양식 반영 (항목 5)

**agents / skills**
- [ ] `.claude/agents/validator-sonnet.md` — "판정 전용" 명시, tools에 `Write` 추가, last-validate 파일 저장 규칙 추가 (항목 2·3)
- [ ] `.claude/agents/builder-sonnet.md` — finalize 위임 시 가드, 구현 후 task 문서의 `## 4-1` 섹션 갱신 규칙 추가 (항목 2)
- [ ] `.claude/agents/architect-opus.md` — `model: opus` 별칭으로 변경 (항목 12)
- [ ] `.claude/skills/bootstrap-project/SKILL.md`, `.claude/skills/bootstrap-stack/SKILL.md` — 항목 6 적용 시 `context: fork`/`agent`/`model`/`effort` 라인 통째로 제거. 항목 6 적용 전이라면 `model` 라인만 별칭(`opus`)으로 (항목 12와 항목 6의 조건부 처방).
- [ ] `.claude/settings.json` — `"model": "sonnet"` 별칭으로 변경 (항목 12). PostToolUse hook 자동 등록은 prototyping 후 별도 항목.

**ADR (정책성 결정)**
- [ ] `docs/90-decisions/ADR-00x-model-alias-policy.md` 신규(항목 12) — 모델 별칭 우선 정책
- [ ] `docs/90-decisions/ADR-00x-commit-convention.md` 신규(항목 2) — Conventional Commits 기본 채택
- [ ] (선택) `docs/90-decisions/ADR-00x-workitem-lifecycle.md` — plan→implement→validate→repair→finalize→wrap-milestone 라이프사이클 정의

**.gitignore**
- [ ] `docs/40-validation/last-validate-*.md` 추가(항목 3, ephemeral 검증 산출물)
- [ ] `docs/00-meta/.bootstrap-session-*.md` 추가(항목 6, ephemeral 세션 임시 파일)

---

## 9. 참고 — 공식 문서 조사 결과 (요약)

> 2026-04-28 시점 [code.claude.com/docs](https://code.claude.com/docs/en) 1차 출처 기반. 실제 구현 직전에 최신 문서로 한 번 더 확인 권장.

### 9.1 Skill frontmatter (공식 확인)
출처: [Skills 문서](https://code.claude.com/docs/en/skills)
- 지원 필드: `name`, `description`, `when_to_use`, `argument-hint`, `arguments`, `disable-model-invocation`, `user-invocable`, `allowed-tools`, `model`, `effort`, `context`, `agent`, `hooks`, `paths`, `shell`
- `context: fork` — fork된 sub-agent 컨텍스트로 실행. `agent` 필드로 사용할 sub-agent 지정. **공식 경고**: skill 본문이 단순 가이드라인이면 sub-agent가 actionable prompt 없이 받아 무의미한 결과 반환. 본 문서가 `context: fork`로 올리는 모든 skill(`plan-workitem`, `review-doc` 등 항목 4)은 본문이 명령형 절차여야 한다.
- `allowed-tools`의 Bash 패턴: 공식 예시 `Bash(git add *) Bash(git commit *) Bash(git status *)` — **공백 + 별표** 형식. 콜론 형식(`Bash(git status:*)`)은 문서에 없음.
- bundled skill: `/simplify`, `/batch`, `/debug`, `/loop`, `/claude-api`, `/init`, `/review`, `/security-review` 등이 기본 제공. `/batch`가 bundled라는 사실이 항목 11과 직접 관련.

### 9.2 Sub-agent frontmatter (공식 확인)
출처: [Sub-agents 문서](https://code.claude.com/docs/en/sub-agents)
- 지원 필드: `name`, `description`, `tools`, `disallowedTools`, `model`, `permissionMode`, `maxTurns`, `skills`, `mcpServers`, `hooks`, `memory`, `background`, `effort`, `isolation`, `color`, `initialPrompt`
- `isolation: worktree` — sub-agent를 임시 git worktree에서 실행. **변경이 없으면 자동 cleanup, 변경이 있으면 worktree 경로와 브랜치를 결과에 포함**(공식 동작).
- `model` 필드는 `sonnet`/`opus`/`haiku` 별칭, 전체 ID(`claude-opus-4-7` 같은), 또는 `inherit` 허용. 공식 model-config 문서의 예시가 `claude-opus-4-7`로 갱신되어 있어 4-6은 1버전 뒤 → 항목 12 근거 강화.
- `description`의 "proactively"는 관례일 뿐 공식 키워드 효과는 별도로 명시되어 있지 않다(공식은 "trigger 조건 + Use when ..." 형식의 구체성을 권장). `architect-opus.md` 등 본 보일러플레이트가 사용하는 "Use proactively for ..." 형식은 트리거 조건이 명확하므로 그대로 유지해도 무방.

### 9.3 Hooks (공식 확인)
출처: [Hooks 문서](https://code.claude.com/docs/en/hooks)
- 이벤트: `SessionStart`/`SessionEnd`, `UserPromptSubmit`/`UserPromptExpansion`, `PreToolUse`/`PostToolUse`/`PostToolUseFailure`/`PostToolBatch`, `Stop`/`StopFailure`/`SubagentStart`/`SubagentStop`, `PermissionRequest`/`PermissionDenied` 등
- `.claude/settings.json`의 `hooks` 필드에 matcher + 명령으로 등록. exit code 2로 즉시 차단. skill·sub-agent frontmatter의 `hooks` 필드도 공식 지원(스코프 격리).
- 입출력 JSON 스키마는 공통 필드(`session_id`, `transcript_path`, `cwd`, `permission_mode`, `hook_event_name`, `agent_id`, `agent_type`)와 출력(`continue`, `stopReason`, `suppressOutput`, `systemMessage`)이 공식 문서에 정의. 이벤트별 추가 필드는 [hooks-reference](https://code.claude.com/docs/en/hooks-reference) 참조.

### 9.4 모델 별칭 (공식 확인)
출처: [Model Config 문서](https://code.claude.com/docs/en/model-config)
- `sonnet` / `opus` / `haiku` 별칭은 자동 최신 매핑. settings.json의 `"model": "opus"` 같은 표기 권장. 특정 버전 핀이 필요할 때만 전체 ID(`claude-opus-4-7`).

### 9.5 AskUserQuestion (공식 확인)
출처: [user-input 문서](https://code.claude.com/docs/en/agent-sdk/user-input)
- Claude가 다중 선택지 + 자유 입력으로 사용자에게 질문할 수 있는 공식 도구. 기본 사용 가능.
- sub-agent의 `tools` 필드를 명시적으로 제한한 경우(본 보일러플레이트의 `architect-opus`는 `Read, Glob, Grep, Write`로 제한) 해당 sub-agent에서는 사용 불가. 따라서 항목 6.3의 "architect-opus에 AskUserQuestion이 없다"는 사실에 부합.
- 본 문서 항목 6.4는 보일러플레이트 단계에서는 자연어 응답으로 충분하다는 design 결정을 내렸다. 향후 인터랙티브 UX 개선 시 AskUserQuestion 채택을 별도 항목으로 검토.

### 9.6 보일러플레이트 표준 패턴 (공식 미정)
정형화된 "페르소나→pain→시나리오" 라운드(항목 6), 워크아이템 라이프사이클(plan→implement→validate→repair→finalize→wrap-milestone, 항목 1~5), 검증 결과 파일 매개 전달(항목 3.4 메커니즘 A) 등은 공식 표준이 없으며 본 보일러플레이트가 제안하는 권장 패턴이다. 공식 표준이 등장하면 그 시점에 본 문서를 갱신한다.

---

## 10. 재검토 메모

### 10.1 분류 기준
본 문서의 항목들은 두 층으로 나뉜다.

- **stale·노화 위험 (P0)** — 현재는 동작하지만 모델 버전 갱신 등으로 가까운 시점에 fork된 새 프로젝트가 실패할 위험이 있는 항목. 항목 12(모델 ID — `claude-opus-4-6`이 이미 1버전 뒤)가 이에 해당하며 적용 순서에서 최우선.
- **표현·가이드 정리 (P1)** — 즉시 동작 실패는 아니지만 사용자 혼선을 키우는 항목. 항목 11(`/batch` 사용 가이드 분리)이 이에 해당.
- **품질 향상 (P0~P2)** — 보일러플레이트의 가치(워크아이템 라이프사이클의 일관된 마감, 인터랙티브 기획)를 끌어올리는 항목. 항목 1~6이 이에 해당.

### 10.2 이번 재검토 사이클에서 제외/삭제한 후보
- **`.claude/settings.local.json` 저장소 모순 해소(원 항목 11)** — 검증 결과 `.gitignore`가 정확히 제외하고 있어 git에 트래킹되지 않음. "저장소에 들어 있다"는 전제가 사실이 아니므로 항목 자체 삭제.
- **템플릿 자기검증 스크립트(원 항목 13)** — 유일한 구체 근거가 위 settings.local 모순이었고, 자기검증 스크립트의 런타임 결정이 GUARDRAILS_STRATEGY의 stack-agnostic 원칙과 충돌. 별도 항목으로 두지 않고 향후 `/stack-guard`(항목 1) 산출물에 흡수할 여지를 남긴다.
- **slash command 실행 방식 불일치(원 항목 12)** — 항목 4와 동일한 변경이라 항목 4에 흡수.
- **README 이중 언어 운영 개선** — 드리프트 위험은 있으나 즉시 기능 오작동은 아니라 후순위.
- **문서 중복 축소** — 유지보수성 이슈는 맞지만 핵심 결함보다 후순위.

---

## 11. `/batch` 표현 명확화 + worktree 사용 가이드 추가

### 11.1 배경 / 동기
이 저장소는 메인 세션이 오케스트레이션을 맡고 실작업은 서브에이전트와 슬래시 명령에 위임하는 운영 모델을 약속한다. 그 운영 모델 안에서 "독립적인 여러 task 동시 처리"는 진입점으로 표현된다. 진입점의 정체와 사용 시점이 사용자에게 명확해야 한다.

### 11.2 현재 상태 (As-is)
- `CLAUDE.md`의 위임 트리거 항목: `독립적인 여러 task 동시 처리 → /batch 또는 worktree`.
- `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`의 위임 트리거 표(32행)와 "병렬 작업 원칙" 항목(55행 부근)에서도 `/batch` 사용을 권장.
- `.claude/skills/` 아래에 `batch` skill은 없다.
- **그러나 `/batch`는 Claude Code 공식 bundled skill이다** — `/simplify`, `/batch`, `/debug`, `/loop`, `/claude-api`와 함께 번들로 제공된다. 사용자가 `/batch`를 입력하면 작업 단위당 격리된 git worktree에서 background agent를 하나씩 띄운다(공식 동작 — [skills 문서](https://code.claude.com/docs/en/skills) 및 [commands 문서](https://code.claude.com/docs/en/commands)).
- worktree도 같은 줄에 묶여 안내되지만, 언제·어떻게(Agent 도구의 `isolation: "worktree"` 인자) 가이드가 별도로 없다.

### 11.3 문제점
- 보일러플레이트 문서가 `/batch`를 그냥 명사처럼 쓴다 — 이게 이 보일러플레이트의 custom skill인지, Claude Code bundled skill인지, 사용자가 본문만 읽고는 알기 어렵다.
- bundled `/batch`는 worktree에서 background agent를 띄우는 비교적 무거운 패턴이다. "한 메시지에서 Agent 도구를 여러 번 병렬 호출"하는 가벼운 패턴과 구분 없이 권장되어, 사용자가 어느 쪽을 골라야 할지 판단 근거가 없다.
- worktree 격리(`isolation: "worktree"`)는 sub-agent frontmatter 필드 또는 Agent 도구 호출 인자로 쓸 수 있는데, 어느 쪽인지 가이드가 없다.

### 11.4 개선안 (To-be)
- 위임 트리거 표의 `/batch 또는 worktree` 항을 다음 3개 패턴으로 분리·명시한다(가벼움 순):
    1. **한 메시지에서 Agent 호출 병렬** — 가장 가벼움. 독립적인 짧은 sub-agent 작업 여러 개를 한 turn에 띄운다. 메인이 결과만 통합.
    2. **`isolation: "worktree"` 인자로 단일 Agent 호출** — Agent 도구 호출 시 인자로 지정. sub-agent가 작업 디렉터리를 격리 git worktree에 받아 진행. 변경이 없으면 자동 cleanup, 변경이 있으면 worktree 경로와 브랜치 이름을 반환([공식 문서 확인](https://code.claude.com/docs/en/sub-agents) — sub-agent frontmatter `isolation` 필드 설명).
    3. **bundled `/batch` 호출** — 사용자가 직접 발화. Claude Code가 작업 단위당 background agent + worktree를 자동 생성. 큰 마이그레이션·codebase-wide 변경 같은 코드 단위 분리가 분명한 작업에 적합.
- 선택 기준: 가벼운 병렬 → 1, 같은 파일 충돌 가능성 있는 단일 단발 작업 → 2, 작업 단위가 분명한 codebase-wide 분산 작업 → 3.
- `AGENT_EXECUTION_STRATEGY.md`에 위 3개 패턴의 짧은 사용 가이드 단락을 추가한다 — 각 패턴의 트리거, 결과 회수 방식, 비용 차이.

### 11.5 구현 스케치
- 수정 `CLAUDE.md`
    - 위임 트리거 표의 `독립적인 여러 task 동시 처리` 행 우항을 `한 메시지에서 Agent 병렬 호출 (가벼움) / Agent 호출 시 isolation: "worktree" (단일 격리) / bundled /batch (codebase-wide 분산)` 정도로 교체. 한 줄이 길면 본문 단락으로 넘기고 표는 "병렬 패턴 3종 — 상세는 AGENT_EXECUTION_STRATEGY.md의 '병렬 패턴 3종' 단락"으로 축약.
- 수정 `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`
    - 위임 트리거 표(32행) 동일하게 수정.
    - "병렬 작업 원칙" 항목(55행 부근)의 `/batch` 단독 언급을 위 3종으로 확장.
    - 신규 단락 "병렬 패턴 3종" 추가 — 위 11.4의 1·2·3을 본문 형태로. `/batch`가 Claude Code bundled skill이라는 한 줄을 명시(보일러플레이트의 custom skill이 아님).

### 11.6 영향 범위
- 수정 2 문서 (`CLAUDE.md`, `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`)
- 신규 skill 없음(bundled `/batch`를 그대로 활용)

### 11.7 열린 질문
- `/batch`의 trigger 조건을 보일러플레이트가 별도로 wrapping할 가치가 있는가? — 잠정: 없음. bundled로 충분하고, 본 보일러플레이트의 차별점(워크아이템 라이프사이클)은 항목 1·2·3의 skill들이 담당.
- worktree 가이드를 별도 문서로 분리할지, `AGENT_EXECUTION_STRATEGY.md` 안 단락으로 둘지 — 잠정: 본문 단락(컨텍스트 비용 최소).

---

## 12. 모델 ID 표기 일관성과 노화 위험 해소

### 12.1 배경 / 동기
이 보일러플레이트의 핵심 가치는 "여러 프로젝트에서 반복 재사용"이다. 모델 ID가 시간이 지나며 deprecated되면, 수개월 뒤 새 프로젝트에 그대로 복제했을 때 메인 세션 또는 특정 서브에이전트가 즉시 실행에 실패할 수 있다. 표기 일관성이 없으면 어디를 갱신해야 하는지 사람의 기억에 의존하게 된다.

### 12.2 현재 상태 (As-is)
- `.claude/settings.json`의 `model` → `claude-sonnet-4-6` (전체 ID, 특정 버전 고정)
- `.claude/agents/architect-opus.md`의 `model` → `claude-opus-4-6` (전체 ID, 특정 버전 고정)
- `.claude/skills/bootstrap-project/SKILL.md`, `.claude/skills/bootstrap-stack/SKILL.md`의 `model` → `claude-opus-4-6` (전체 ID, 특정 버전 고정)
- `.claude/agents/builder-sonnet.md`, `validator-sonnet.md`, `qa.md`, `planner.md`, `reviewer.md`의 `model` → `sonnet` (별칭, 자동 최신 매핑)
- 모델 버전이 stale 됐을 때 어디를 갱신할지, 어떤 표기를 기본으로 둘지 안내하는 문서는 없다.

### 12.3 문제점
- 같은 종류의 정보(어떤 모델을 쓸지)가 두 가지 표기로 섞여 있어, 새 프로젝트를 시작한 사람이 어느 쪽 규칙을 따라야 할지 즉시 알기 어렵다.
- 전체 ID로 고정된 자리들(`settings.json`, `architect-opus.md`, 두 bootstrap skill)이 보일러플레이트의 "재사용 가능" 약속을 가장 먼저 깨는 stale 포인트가 된다.
- **이미 1버전 stale**: 공식 [model-config 문서](https://code.claude.com/docs/en/model-config) 예시는 `claude-opus-4-7`로 갱신되어 있어, 보일러플레이트가 박아둔 `claude-opus-4-6`은 이미 한 세대 뒤. 즉시 실행 실패는 아니지만, 다음 deprecation 사이클이 오면 fork된 새 프로젝트가 메인 세션 진입에서 실패한다.
- "별칭이 더 안전한가, 전체 ID가 더 안전한가"에 대한 의도가 어디에도 명시되지 않아, 향후 수정자가 자기 취향으로 더 어긋나게 갱신할 위험이 크다.

### 12.4 개선안 (To-be)
- 보일러플레이트가 채택할 정책을 한 곳에 명시한다.
    - **shared 기본값에서는 모델 별칭(`sonnet`, `opus`, `haiku`)만 사용한다. 특정 버전을 강제해야 하는 이유가 있으면 ADR로 남기고 그 자리에서만 전체 ID를 사용한다.**
    - 근거: ① 자동 최신 매핑이 보일러플레이트의 재사용 약속과 가장 잘 맞는다. ② 모델 버전 갱신을 사람이 잊어 발생하는 stale 실패를 원천 차단한다.
- 위 정책에 따라 stale 포인트를 정리한다.
    - `.claude/settings.json` → `"model": "sonnet"` (또는 의도적 고정 이유가 있으면 ADR 추가)
    - `.claude/agents/architect-opus.md` → `model: opus`
    - `.claude/skills/bootstrap-project/SKILL.md`, `.claude/skills/bootstrap-stack/SKILL.md`의 model 라인 처리는 **항목 6 적용 시점에 따라 다르다**:
        - **항목 6 적용 전**: 두 skill 모두 `model: claude-opus-4-6` → `model: opus`로 변경.
        - **항목 6 적용 후**: 두 skill 모두 `agent: architect-opus` / `context: fork` / `model` 라인이 통째로 사라진다(메인 세션 직접 운전 + architect-opus 단발 sub-call로 메커니즘이 바뀌므로). 즉 항목 6과 12를 동시에 적용한다면 두 skill에 대한 별칭 변경 처방은 **수행하지 않아도 된다** — 라인 자체가 제거되기 때문.
- 정책을 깨고 특정 버전을 고정해야 할 때를 위한 절차를 짧게 적어 둔다 — "ADR로 이유 기록 → 갱신 책임자 명시" 정도면 충분.

### 12.5 구현 스케치
- 수정 `.claude/settings.json`
    - `"model": "claude-sonnet-4-6"` → `"model": "sonnet"`
- 수정 `.claude/agents/architect-opus.md`
    - `model: claude-opus-4-6` → `model: opus`
- (조건부) 수정 `.claude/skills/bootstrap-project/SKILL.md`, `.claude/skills/bootstrap-stack/SKILL.md`
    - 항목 6 적용 전이면 `model: claude-opus-4-6` → `model: opus`.
    - 항목 6과 동시 적용이면 이 처방은 생략 — 항목 6.5가 두 skill의 frontmatter에서 `agent`/`context: fork`/`model` 라인을 통째로 제거한다.
- 수정 `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`
    - "중요 원칙" 또는 신규 단락에 모델 표기 정책 한 단락 추가 — 별칭 기본, 특정 버전 고정 시 ADR 명시.
- (권장) 신규 `docs/90-decisions/ADR-00x-model-alias-policy.md`
    - 별칭 우선 정책의 배경·결정·결과를 ADR 형식으로 보존. 정책성 결정이므로 선택이 아니라 권장으로 둔다.
- 항목 1의 `/bootstrap-stack` 산출물에도 "shared 설정의 model은 별칭으로 둔다"를 가이드로 포함하면 항목 6의 인터랙티브 흐름과 자연스럽게 결합된다.

### 12.6 영향 범위
- 수정 1 설정(`.claude/settings.json`), 1 agent(`.claude/agents/architect-opus.md`), 1 문서(`docs/00-meta/AGENT_EXECUTION_STRATEGY.md`)
- 항목 6 적용 전이면 추가로 2 skill(bootstrap-project, bootstrap-stack)
- 신규 ADR 1개(권장)

### 12.7 열린 질문
- architect-opus처럼 "이 작업은 반드시 가장 강한 모델로 돌려야 한다"라는 의도를 별칭만으로 표현해도 충분한가? — 잠정: 충분. 별칭 `opus`는 정의상 가장 강한 Opus 계열을 가리킨다.
- 향후 별칭이 사용자가 원하는 비용 대비 성능 곡선과 어긋나는 시점이 오면? — 잠정: 그 시점에 ADR로 "별칭 → 특정 버전" 전환을 기록한다. 현 시점에는 별칭이 운영 비용에 더 유리.
- 사용자가 보일러플레이트를 fork한 직후 자기 환경의 비용 정책을 강제해야 한다면 어디서 override가 자연스러운가? — 잠정: `.claude/settings.local.json`에서 모델 override(별도 별칭 또는 특정 ID).
