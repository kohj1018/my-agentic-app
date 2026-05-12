# IMPROVE-GUIDE

> 본 문서는 본 보일러플레이트의 개선을 *처음 보는 사람도 따라할 수 있는 단계별 행동 지침*으로 정리한 것이다.
>
> **이 가이드의 사용법** — 위에서 아래로 Phase 순서대로 진행한다. 각 Phase는 "선행 의존성"이 있고, 먼저 끝내야 다음 Phase 작업이 정합한다. Phase 안의 Step은 비교적 독립적이라 일부는 병렬 가능하다.
>
> **결과물** — 본 가이드를 모두 따라 끝내면 P0·P1 결정이 모두 ADR로 박히고, 보일러플레이트가 **단순성·SSOT·multi-tool 호환을 1순위로 박은 최소 하네스 + dogfood 검증된 8단계 lifecycle + 4 Pillars 정합 하네스**로 진화한다.
>
> **커밋 정책** — 각 Step 끝에 권장 커밋 메시지(영어 한 줄, `docs:` prefix)를 박아 두었다. 1 Step = 1 commit이 디폴트이며, 같은 ADR을 여러 Step이 박는 경우(예: Phase 5의 ADR-027) 마지막 Step에서 한 번에 commit해도 된다. Phase 단위 묶음 commit도 가능 — 본 가이드의 권장 메시지를 적절히 통합 발화하면 된다.

---

## Part 0. 시작하기 전에

### 0.1 본 가이드의 구조

본 가이드는 **12개 Phase × 약 45개 Step**으로 구성된다.

| Phase | 이름 | 우선순위 | 핵심 산출물 |
|-------|------|---------|-----------|
| 1 | P0-Gate 시뮬레이션 | **P0-Gate (단일)** | `SIMULATION_RUN.md` (Record) |
| 2 | 정책 자기 일관성 + 범위 한정 | P0-Post-Gate | ADR-022 / ADR-031 / ADR-000 |
| 3 | 단순 cleanup | P0-Post-Gate | ADR-011 / 012 / 024 |
| 4 | Schema 강화 | P0-Post-Gate | ADR-026 / 007 amend / 008 amend (1차) / 009 amend |
| 5 | 인터페이스 결정 흐름 | P0-Post-Gate | ADR-027 (인터페이스 결정 + 백엔드/프론트 sub-section 통합) |
| 6 | Milestone graduation | P0-Post-Gate | ADR-014 / 017 |
| 7 | 4 Pillars 정합 | P0-Post-Gate | ADR-019 / 021 (+ 021 amend) (+ 008 amend 2차) / ADR-018은 P1 트리거 보류 |
| 8 | 기획 PO/PM 트랙 강화 | P0-Post-Gate | ADR-035 / 036 / 037 |
| 9 | P1 보강 | P1 | ADR-020 / 025 + skill amend |
| 10 | 트리거 대기 (P1/P2) | P1·P2 보류 | (트리거 발동 전 보류) |
| 11 | 명시적 No 결정 (참고표) | — | (Phase 1~10 작업 시 참조) |
| 12 | 통합 검토 + 회귀 시뮬레이션 | 최종 | ADR-017 회차 2 |

### 0.2 Phase별 의존성 그래프

```
[Phase 1: P0-Gate] ─── (시뮬레이션 데이터로 P0 우선순위 재조정)
        │
        ▼
[Phase 2: 자기 일관성] ─── (ADR-022 Ratchet이 다른 ADR의 근거)
        │
        ▼
[Phase 3: cleanup] ─── (00-meta 흡수가 다른 PR의 surface 줄임)
        │
        ├──► [Phase 4: Schema 강화] ◄─┐
        ├──► [Phase 5: 인터페이스]    │
        │       │                     │
        │       └──► [Phase 7: 4P]    │
        │                             │
        ├──► [Phase 6: Milestone]     │
        │       │                     │
        │       └──► [Phase 8: 기획] ──┘
        │
        ▼
[Phase 9: P1] ─── [Phase 10: 트리거 대기]
        │
        ▼
[Phase 12: 회귀 시뮬레이션]
```

**핵심 규칙**:
- Phase 2가 끝나야 Phase 3~8의 ADR 본문이 *근거 표현*에서 일관성이 보장된다(Ratchet Principle 적용).
- Phase 5의 ARCHITECTURE 7-1/7-2(API/CLI) sub-section 신설은 ARCHITECTURE 7-3/7-4(백엔드/프론트) sub-section 신설과 한 흐름 → 본 가이드는 **Phase 5(ADR-027)에서 통합 처리**해 인터페이스 결정 책임 분배 의존성을 한 ADR에 묶었다.
- Phase 7의 ADR-008 amend 2차(`Refs:` footer)는 Phase 4의 ADR-008 amend 1차(monorepo scope)와 같은 ADR-008을 두 번 수정하므로, Phase 4와 Phase 7을 *순차*로 처리한다(같은 PR 안에 묶어도 됨). ADR-018(CODE_LINEAGE.md 자동 생성)은 P1 트리거 보류 — Phase 7에서는 footer 컨벤션만 P0로 박는다.

### 0.3 ADR 번호 마스터 표

> 본 가이드에서 신설·amend되는 모든 ADR 번호를 한눈에. 작업 순서상 *3번(문서 아키텍처)*이 *1번(인터페이스 결정)*보다 먼저 박히므로 ADR-011/012는 문서 아키텍처에 할당, 인터페이스 결정은 ADR-027로 배치.

| ADR | 신설/amend | 주제 | Phase |
|-----|----------|-----|------|
| ADR-000 | 신설 (메타) | Boilerplate ADR scope 명시 + supersede 정책 | 2.3 |
| ADR-007 | amend | finalize lock file 화이트리스트 11종 | 4.3 |
| ADR-008 | amend (1차) | 모노레포 scope = 패키지명 | 4.4 |
| ADR-008 | amend (2차) | Conventional Commits footer (`Refs:`) | 7.1 |
| ADR-009 | amend | AC ID 컨벤션 강화 (`AC_N`/`[AC-N]`, P1 → 데이터 트리거 시 P0) | 4.2 |
| ADR-011 | 신설 | AGENTS.md 100줄 hard cap | 3.4 |
| ADR-012 | 신설 | docs/00-meta 9→6 흡수 + Diátaxis 모드 라벨 | 3.2~3.3 |
| ADR-014 | 신설 | Milestone graduation contract | 6.1~6.3 |
| ADR-017 | 신설 | dogfood 시뮬레이션 의무 + 재실행 트리거 | 6.3 |
| ADR-018 | 신설 (P1 트리거 보류) | CODE_LINEAGE.md 자동 생성 | 7.1 (deferred) / Phase 10.9 |
| ADR-019 | 신설 | Context Packs frontmatter + JIT 로딩 | 7.2 |
| ADR-020 | 신설 | `validate --changed` (incremental) | 9.1 |
| ADR-021 | 신설 | `/stack-guard` 정적 분석 1종 권장 | 7.3 |
| ADR-021 | amend | secret scanner (gitleaks/trufflehog) | 7.3 |
| ADR-022 | 신설 | Ratchet Principle 명문화 + 적용 범위 한정 | 2.1 |
| ADR-024 | 신설 | Claude Code plan 모드 lifecycle 비범위 | 3.1 |
| ADR-025 | 신설 | `/bootstrap-stack` 외부 의존 + CI 권장 | 9.3 |
| ADR-026 | 신설 | plan-workitem 강화 (TASK_TEMPLATE schema) | 4.1 |
| ADR-027 | 신설 | 인터페이스 결정 책임 분배 (UI 시각 + API + CLI + 백엔드/프론트 sub-section 통합) | 5.1~5.5 |
| ADR-031 | 신설 | 비웹 스택을 기본 자동화 직접 지원 범위 밖으로 명시 (override 경로 제공) | 2.2 |
| ADR-035 | 신설 | Continuous discovery (DISCOVERY.md living doc) | 8.1 |
| ADR-036 | 신설 | Feature-level PRD (FEATURE_TEMPLATE 12섹션) | 8.2 |
| ADR-037 | 신설 | Spec coverage self-audit | 8.3 |
| ADR-039 | 신설 (P2 트리거) | JTBD Switch Interview 1pager | 10.2 |
| ADR-040 | 신설 (P2 트리거) | Milestone scoping (Story Map + Pitch) | 10.3 |

**빈 번호** (향후 사용): ADR-013, 015, 016, 023, 028, 029, 030, 032, 033, 034, 038. (이 중 013/015/016/023/038은 *과거 본 가이드 초기 안에 후보로 거론됐다가 효용 < 복잡도 검토로 제거된 번호* — 향후 사용 가능하나, 동일 주제(SSOT drift / Fowler quadrant / METRICS / MCP / PMF)를 다시 박을 경우 *Part 11 영구 No 결정과 정합한지* 먼저 검토.)

**ADR 개수 안내** — Phase 1~9에서 박히는 ADR 총량은 *신설 17개 + amend 6개 = 23개* (deferred ADR-018·P2 트리거 ADR-039·040 3개는 트리거 발동 전 파일 미생성, 후보로만 유지). 정확한 일람은 Part 14 부록의 SSOT 표 참조. 효용 < 복잡도인 결정(SSOT drift 자동 검출 / Fowler 4-quadrant / METRICS.md / `--apply-carryover` / architect-opus auto-escalation / 영역별 lint / PMF playbook / MCP baseline)은 *제거*했다. *결정 1개 = ADR 1개* 원칙(ADR-005 패턴 4)은 *supersede 시 부분 뒤집기 가능*이라는 정합 근거가 있어 유지.

### 0.4 작업 환경 준비

본 가이드의 작업을 시작하기 전 다음을 확인한다.

1. **저장소 상태**: `git status`로 untracked / modified가 정리되어 있는지 확인. 본 IMPROVE-GUIDE.md 외 dirty 파일 0건이 이상적.
2. **현재 ADR 번호**: `docs/90-decisions/`에 ADR-001~010이 존재. ADR-002, ADR-003은 placeholder. → 본 가이드의 신설 ADR은 ADR-011부터 시작한다.
3. **기준 브랜치**: `main` 위에서 작업한다. 각 Phase별로 별도 브랜치를 파는 것을 권장하지만, **단일 브랜치에서 Phase 순차 commit**이 git history 추적에 더 자연(ADR-008 Conventional Commits 정합).
4. **커밋 단위**: 1 Step = 1 commit이 디폴트. 같은 ADR을 여러 Step이 박는 경우(예: Phase 5의 ADR-027 — 5.1~5.5)는 마지막 Step에서 한 번에 commit하는 것을 권장. 각 Step 끝의 *Commit message* 단락에 권장 메시지가 박혀 있다.

### 0.5 모든 Phase에 적용되는 공통 규칙

- **새 ADR 작성 시**: ADR 본문은 `docs/90-decisions/_ADR_GUIDE.md`의 권장 섹션을 따른다. 작성 직후 `docs/90-decisions/README.md`에 한 줄 추가(ADR 번호·제목·상태·요약).
- **ADR 본문 amend 시**: 기존 ADR을 직접 수정하지 말고, **`## 후속 작업`** 또는 본문 끝에 `## Amendment N (날짜)` 단락을 추가하는 형태로 누적한다. ADR-005 SSOT 정합.
- **AGENTS.md 갱신 시**: 본 Phase 3.4 이후로는 100줄 cap 적용. *긴 정책 본문*은 ADR로 박고 AGENTS.md에는 한 줄 + ADR 링크만 둔다. *짧은 본문* (1~3줄, cap 안에 충분히 들어가는 *직접 지원 범위 명시 / Claude Code plan 모드 자율성 같은 자기 완결적 정책 라인*)은 AGENTS.md 본문에 직접 박아도 된다 — 단 100줄 cap을 깨지 않을 때만.
- **STRUCTURE.md 갱신 시**: 산출물 표에 행 추가/제거가 발생하면 *반드시* 본 작업 직후 갱신.
- **`Refs:` footer 컨벤션 (Phase 7.1의 ADR-008 amend 2 적용 후)** — 두 위치에서 `Refs:` 키를 사용하되 *값의 형식*으로 구분한다 (grep·자동 도구에서 정규식으로 분리 가능):
  - **Commit message footer**: `Refs: T-NNN (AC-X, AC-Y)` 또는 `Refs: T-NNN` 또는 `Refs: T-001, T-002` (task ID 형식 — CODE_LINEAGE.md derived view용)
  - **PR 본문 footer**: `Refs: ADR-NNN` 또는 `Refs: ADR-NNN, ADR-MMM` (ADR-NNN 형식 — PR과 ADR 연결용)
  - 동일 key의 다른 값 형식은 *의도된 분리*다. 키 통일이 키 분리(`Refs:` vs `ADR:`)보다 grep 패턴 단순.
- **P2 트리거 ADR 파일 정책**: ADR-039·040 같은 *P2 트리거 ADR*은 트리거 발동 전 *파일을 생성하지 않는다*. 본 가이드의 ADR 마스터 표 / Part 14 ADR 일람에 *후보*로만 유지하고, 트리거 발동 후(Step 10.2·10.3 실행 시) accepted ADR 파일을 작성한다.

---

## Part 1. Phase 1 — P0-Gate: baseline dogfood 시뮬레이션

> **이 Phase는 다른 모든 P0의 *선행 조건*이다**. 통과 못하면 Phase 2~8 작업을 *깨진 lifecycle 위에 짓는 것*이라 보류한다.
>
> 근거 — *"짓고 측정"* (Drew Breunig "Implement to learn"). 본 보일러플레이트 fork 사례 0건이라 모든 P0 결정이 [가설] 카테고리. 시뮬레이션 1회 통과가 [가설] → [관측됨] 승격의 단일 경로.
>
> **본 Phase는 *현 상태(baseline) 측정*이다** — Phase 3~8에서 *개선될* 모든 skill을 *현 상태 그대로* 사용해 마찰점을 회수한다. `/discover-product`·`/stack-guard`·`/stabilize-milestone`이 *개선 대상*이긴 하지만 현재 `.claude/skills/`에 존재한다 → 현 상태로 baseline 측정해야 *개선 전 마찰*과 *개선 후 효과*를 구분 가능. 개선 후 효과는 **Phase 12 (Round 2 v2 회귀)에서 측정**한다. baseline + v2의 *delta*가 본 가이드의 evidence base.

### Step 1.1 — todo CLI 시뮬레이션 1회 실행

**우선순위**: P0-Gate (단일)
**Evidence**: [가설→실증 변환 도구]
**의존성**: 없음 (시작점)

#### 무엇을 하는가

별도 디렉터리(예: `~/dev-practice/dogfood-todo-cli/`)에 본 보일러플레이트를 fork한 뒤, **todo CLI 미니 프로젝트**로 8단계 lifecycle을 1회 통과시킨다. 각 단계의 입출력이 다음 단계 입력으로 자연스럽게 들어가는지 체크.

#### 왜 todo CLI인가

- 4-연산 calculator는 stateless·single-function이라 lifecycle 단계를 충분히 굴리지 못한다.
- todo CLI는 CRUD 4종 + persistence가 들어가 8단계 lifecycle의 *모든 결정 포인트*(AC 분해 / E2E / 회귀)를 자연스럽게 자극한다.
- 본 보일러플레이트의 일반 사용 케이스(SaaS / API / CLI)에 더 가까움.

#### 작업 절차

**0. 사전 점검 — 필요한 skill 존재 확인**

```powershell
Get-ChildItem .\.claude\skills
```

다음 **9개 skill**이 모두 존재하는지 확인 (8단계 lifecycle을 9 skill로 통과한다 — discover / bootstrap-project + bootstrap-stack / stack-guard / plan / implement / validate / finalize / stabilize). Phase 1의 baseline 측정은 *현 상태 그대로* 사용하므로 필수:
- `discover-product/SKILL.md`
- `bootstrap-project/SKILL.md`
- `bootstrap-stack/SKILL.md`
- `stack-guard/SKILL.md`
- `plan-workitem/SKILL.md`
- `implement-workitem/SKILL.md`
- `validate-workitem/SKILL.md`
- `finalize-workitem/SKILL.md`
- `stabilize-milestone/SKILL.md`

하나라도 부재 시 Phase 1 보류 → 해당 skill 작성 후 재시도.

**1. fork 준비**

```powershell
Copy-Item -Recurse -Force `
  C:\Users\kbwdesktop\Desktop\dev-practice\my-agentic-app `
  C:\Users\kbwdesktop\Desktop\dev-practice\dogfood-todo-cli

Set-Location C:\Users\kbwdesktop\Desktop\dev-practice\dogfood-todo-cli

# 새 git 초기화
git init -b main
git add -A
git commit -m "chore: initial fork from my-agentic-app boilerplate"
```

**2. 8단계 lifecycle 통과 (9 skill 실행)**

각 skill 실행마다 *어디서 끊겼는지 / 사용자 개입 필요한 지점 / 산출물 비어 있는 지점*을 메모한다. ADR-007 lifecycle 정신상 *자동 호출 없음* — 각 skill은 사용자(또는 메인 세션)가 이전 결과를 보고 *발화*한다. 표의 `#`은 *skill 실행 순서*다 (ADR-007 8단계 lifecycle 매핑: discover=1 / bootstrap-project+bootstrap-stack=2 / stack-guard도 bootstrap 단계의 보강 / plan=3 / implement=4 / validate=5 / repair=6(필요 시) / finalize=7 / stabilize=8).

| # | skill | 입력 | 기대 산출물 | 체크포인트 |
|---|-------|-----|-----------|----------|
| 1 | `/discover-product` | "todo CLI 미니 프로젝트, 1인 개발자가 매일 쓸 작업 목록 도구" | `docs/10-charter/DISCOVERY.md` | persona·pain·MVP 채워짐 |
| 2 | `/bootstrap-project` | DISCOVERY.md 입력 | `PROJECT_CHARTER.md` + `ARCHITECTURE_OVERVIEW.md`(스택 미정) | 11/10섹션 placeholder 채워짐 |
| 3 | `/bootstrap-stack` | "Node.js + TypeScript + Vitest" | ARCHITECTURE 7번 채움 + STACK_SETUP_PLAN.md | `validate` 명령 동작 |
| 4 | `/stack-guard` | bootstrap-stack 결과 기반 사용자 발화 | `scripts/verify.{mjs,ps1}` + `.gitattributes` | `pnpm validate` 통과 |
| 5 | `/plan-workitem` | "M1: CRUD 기본" | `M1-foundation.md` + `F-001-todo-crud.md` + `T-001~T-003.md` | task의 AC가 채워짐 |
| 6 | `/implement-workitem` (T-001) | T-001 본문 | RGR 구현 + 테스트 | 테스트 통과 |
| 7 | `/validate-workitem` (T-001) | T-001 완료 후 사용자 발화 | `docs/40-validation/reports/T-001.md` Pass | report에 AC 매핑 |
| 8 | `/finalize-workitem T-001` | validate Pass 후 사용자 발화 | status `done` + commit | `git log` 1 commit |
| (5~8 반복) | T-002, T-003 | | | |
| 9 | `/stabilize-milestone M1` | M1 모든 task done 후 사용자 발화 | `QA_FINDINGS.md` + `IMPROVEMENT_GUIDE.md` 업데이트 | M1 헤더 누적 |

**3. 성공 기준 (3개 모두 충족 시 gate 통과)**

- [ ] **사용자 개입 ≤ 1회** — *개입 1회의 단위*: skill 출력 뒤 사용자가 산출물(template placeholder)에 직접 수동 편집을 가하는 1회 행위. *질문 응답*(skill이 요구한 사용자 입력)은 포함하지 않음.
- [ ] **산출물 placeholder 충원율 ≥ 80%** — *분모*: 각 skill이 박는 산출물(DISCOVERY.md / PROJECT_CHARTER.md / ARCHITECTURE_OVERVIEW.md / M1 / F-001 / T-001~003 / validation report) 안의 `## 섹션` 개수 총합. *분자*: 본문 내용이 채워진 섹션(placeholder 주석만 남은 섹션은 제외) 개수. 80% 이상 채워졌을 때 통과.
- [ ] **graduation pre-check 미통과 사유 ≤ 2개** — *사유 1개*의 단위: MILESTONE_TEMPLATE `## 5. 완료 기준` checklist의 미충족 항목 1개. 현재 Phase 6.2 박히기 전이라 사람이 수동 점검: M1의 모든 task done / 통합 validate 통과 / E2E 적용 가능 시 통과 / AC 매핑 100% (4개 중 미충족 ≤ 2개).

**4. 기록 — `docs/40-validation/SIMULATION_RUN.md` (Record)**

원 보일러플레이트(`my-agentic-app`)의 `docs/40-validation/SIMULATION_RUN.md`에 다음 형식으로 기록.

```markdown
# Simulation Run

> dogfood 시뮬레이션 회차별 누적. 회차 헤더 형식: `## Round N (YYYY-MM-DD, scenario)`.

## Round 1 (2026-05-09, todo CLI / Node+TS+Vitest)

### 단계별 마찰점
- discover-product: (예: pain 표 작성 자유도 너무 큼 → R1 가이드 부족)
- bootstrap-project: (예: 스택 미정 시 ARCHITECTURE 7 비어 있는데 8 채울 때 헷갈림)
- bootstrap-stack: ...
- stack-guard: ...
- plan-workitem: (예: AC verb whitelist 부재로 vague AC 통과)
- implement-workitem: (예: RGR Red phase 진입 직전 사고 단계 부재)
- validate-workitem: ...
- finalize-workitem: (예: lock file 매번 Needs Review로 재발화)
- stabilize-milestone: (예: graduation 기준 부재로 졸업 판정 모호)

### 성공 기준 충족
- 사용자 개입: N회 (목표 ≤ 1)
- 충원율: N% (목표 ≥ 80%)
- graduation 미통과 사유: N건 (목표 ≤ 2)

### 결정에 미친 영향
- 통과: ✓ → Phase 2 시작
- 실패: → 발견된 깨짐 N건을 ADR 후보로 박고, 본 가이드의 결합된 P0 재평가
```

**5. 결과 처리**

| 결과 | 다음 액션 |
|-----|---------|
| **통과** | Phase 2 시작 |
| **실패** | 발견된 lifecycle 깨짐을 ADR 후보로 즉시 박고, 본 가이드의 Phase 2~8 *우선순위*만 재조정(작업 자체는 진행) |

#### 검증

- [ ] `dogfood-todo-cli/`에 8단계 lifecycle 1회 통과 흔적이 git log로 확인됨
- [ ] 원 보일러플레이트 `docs/40-validation/SIMULATION_RUN.md`에 Round 1 기록 commit 됨
- [ ] 성공 기준 3개의 충족/미충족이 명시됨

#### 결정 근거

본 단계는 *모든 P0 결정의 evidence base*. fork 사례 0건이라 [관측됨] 데이터 부재 → 시뮬레이션이 데이터 회수의 단일 경로. 본 단계 결과는 Step 6.3의 ADR-017 본문에 *재실행 트리거 정의*에 인용된다.

#### Commit message

```
docs: run dogfood simulation round 1 and record SIMULATION_RUN.md
```

---

## Part 2. Phase 2 — 정책 자기 일관성 + 범위 한정

> 본 Phase는 *다른 모든 ADR의 근거*가 되는 메타 정책을 박는다. 먼저 끝내야 Phase 3~8의 ADR 본문에서 *Ratchet 적용 범위 / 보일러플레이트 직접 지원 범위 / boilerplate vs project ADR scope*가 일관된다.

### Step 2.1 — Ratchet Principle 명문화 (ADR-022 신설)

**우선순위**: P0-Post-Gate
**Evidence**: [관측됨] (예방적 가설을 [가설] 라벨로 명시할 자리 부재)
**의존성**: Phase 1 통과

#### 현재 상태

`docs/90-decisions/_ADR_GUIDE.md`에는 Ratchet Principle 언급 없음. 새 ADR이 *예방적 가설*에 근거할 때 *제약*으로 박을지 *enabling*으로 박을지 판단 기준이 비어 있음.

#### 변경 후 상태

`_ADR_GUIDE.md` 끝에 *적용 범위 표* + 1단락 추가. builder-sonnet self-check에 1줄 추가. evidence label 3종(`[관측됨]`/`[외부실증]`/`[가설]`)이 모든 ADR 본문에 적용 시작.

#### 작업 절차

**1. ADR-022 본문 작성** — `docs/90-decisions/ADR-022-ratchet-principle.md` 신설

```markdown
# ADR-022 — Ratchet Principle 명문화 + 적용 범위 한정

## Status
accepted

## 배경
- 본 보일러플레이트는 fork 사례 0건이라 모든 *예방적 정책*이 가설 카테고리다.
- 문자 그대로의 Ratchet Principle("AGENTS.md의 모든 줄은 specific failure로 추적 가능해야 한다")을 본 보일러플레이트에 적용하면 P0 결정 다수가 [관측됨] 데이터 부재로 P2 강등된다.
- 그러나 보일러플레이트의 존재 의의가 *fork 사용자 보호*라면, 모든 정책을 [관측됨]에만 의존시키면 보호 역할을 못 한다.

## 결정
정책을 두 종류로 구분하고 Ratchet 강도를 차등 적용한다.

| 정책 종류 | 정의 | Ratchet 강도 |
|---------|------|----------------|
| **제약 정책** (constraint) | 새 reviewer 룰 / self-check / ADR 본문 / P0 분류 기준 — *agent 행동을 좁히는 정책* | **강** — [관측됨] *또는* [외부실증] 필수 |
| **enabling 정책** (enabling) | cap·budget·convention·tooling 권장 — *agent에게 도구/한계 제공* | **약** — [관측됨] / [외부실증] / [가설] 중 어느 하나라도 가능 |

## 운영 규칙
- 새 ADR의 `## 배경` 섹션은 (a) 본 보일러플레이트/fork에서 관측된 실패·발견·이슈, 또는 (b) 외부 다중 repo 실증 자료의 출처를 1~3문장으로 명시한다. 둘 다 비어 있고 *예방적 가설*만 있다면 본 ADR은 *제약*이 아닌 *enabling*(소프트 권장)이어야 한다.
- IMPROVEMENT_GUIDE·QA_FINDINGS의 모든 새 항목에 evidence label `[관측됨]`/`[외부실증]`/`[가설]` 중 하나를 박는다.
- **합성 evidence 표기**: 본 가이드는 다음 합성 표기를 사용한다. 모두 위 3종의 조합이며 Ratchet 분류는 *가장 강한 라벨*을 기준으로 한다.
  - `[관측됨+외부실증]` — 보일러플레이트/fork 관측 + 외부 실증 둘 다 있음 (제약 강하게 가능)
  - `[가설→실증]` — 현재 [가설]이지만 Phase 1·12 시뮬레이션 통과 후 [관측됨]으로 승격 예정. 본 가이드의 `[가설→실증 변환 도구]`(Step 1.1 시뮬레이션 단계 자체)와 `[가설→Gate 후 실증]`(개별 결정의 Gate 통과 후 라벨 승격)은 동일한 의미의 단일 카테고리로 본다.
  - `[가설→트리거]` — [가설] 상태로 P1·P2 보류, 트리거 발동 시 재라벨링
- builder-sonnet self-check에 1줄 추가: *"이번 추가/변경이 어떤 구체적 실패를 막는가? 관측된 실패가 없고 가설적 예방이라면 *제약 형태로 강제하지 말고 권장 형태로*."*

## 결과
- 본 가이드의 모든 P0 결정이 자기 일관성 검증을 통과한다.
- fork 사용자가 새 ADR을 박을 때 *제약 vs enabling* 결정 기준이 명확.

## 후속 작업
- (없음 — 본 ADR이 다른 ADR의 근거)

## 참고
- Addy Osmani — agent harness engineering ([addyosmani.com](https://addyosmani.com/blog/agent-harness-engineering/))
```

**2. `_ADR_GUIDE.md` 갱신** — `docs/90-decisions/_ADR_GUIDE.md` 본문 끝에 다음 단락 추가:

```markdown
## Ratchet Principle (ADR-022)
새 ADR을 박을 때는 [ADR-022](ADR-022-ratchet-principle.md)의 적용 범위 표를 따른다. 본 ADR의 `## 배경`은 [관측됨]·[외부실증]·[가설] 중 어디에 근거하는지 명시한다.
```

**3. builder-sonnet self-check 갱신** — `.claude/agents/builder-sonnet.md`의 *self-check* 섹션(있으면) 또는 본문 끝에 1줄 추가:

```markdown
- 이번 추가/변경이 어떤 구체적 실패를 막는가? 관측된 실패가 없고 가설적 예방이라면, 제약 형태로 강제하지 말고 권장 형태로 둔다(ADR-022).
```

**4. ADR 인덱스 갱신** — `docs/90-decisions/README.md`에 한 줄 추가:

```markdown
| 022 | Ratchet Principle | accepted | 정책의 제약 강도를 *제약(강)/enabling(약)*으로 차등 적용 |
```

#### 검증

- [ ] `git log --oneline` 마지막에 `docs: add ADR-022 ratchet principle with constraint/enabling scope` 형식 commit
- [ ] `docs/90-decisions/README.md` 표에 022 행 존재
- [ ] `docs/90-decisions/_ADR_GUIDE.md` 끝에 Ratchet Principle 단락 존재
- [ ] `.claude/agents/builder-sonnet.md`에 self-check 한 줄 추가됨

#### 결정 근거 (왜 가장 먼저인가)

다른 모든 P0 결정의 evidence label이 본 ADR의 정의를 따른다. Phase 3~8의 ADR 본문이 *제약/enabling 차등*을 미리 알아야 *과도한 강제*를 피한다. ADR-005 SSOT 정합.

#### Commit message

```
docs: add ADR-022 ratchet principle with constraint/enabling scope
```

---

### Step 2.2 — 비웹 스택을 기본 자동화 직접 지원 범위 밖으로 명시 (ADR-031 신설)

**우선순위**: P0-Post-Gate
**Evidence**: [외부실증]
**의존성**: Step 2.1 (Ratchet Principle 정합)

#### 현재 상태

본 보일러플레이트는 *모든 스택*을 암묵적으로 지원하려 한다(GUARDRAILS_STRATEGY는 stack-specific 자동화를 옵트인). 그러나 mobile/ML/embedded/game/desktop은 본 lifecycle의 산출물 자리(DESIGN_SYSTEM·ARCHITECTURE)와 정합하기 어렵다 → *암묵적 빈자리*가 발생한다.

#### 변경 후 상태

보일러플레이트의 *기본 자동화·문서 자동화 직접 지원 범위*를 5종으로 박고, 그 외 스택은 fork 사용자가 stack-guard 출력을 override하는 절차를 명시. *지원하지 않는다*가 아니라 *기본값으로 최적화하지 않는다*(override 경로 제공)가 정확한 표현. AGENTS.md에 1줄 추가.

#### 작업 절차

**1. ADR-031 본문 작성** — `docs/90-decisions/ADR-031-non-web-out-of-scope.md` 신설

```markdown
# ADR-031 — 비웹 스택을 기본 자동화 직접 지원 범위 밖으로 명시

## Status
accepted

## 배경
- 본 보일러플레이트는 단순성 1순위(ADR-006) + multi-tool 호환(ADR-010)을 정체성으로 한다.
- 모든 스택(mobile / ML / embedded / game / desktop)을 *기본 자동화의 직접 지원*으로 다루려 하면 DESIGN_SYSTEM·ARCHITECTURE 그룹이 비대화되고, 검증 도구(`validate`)·정적 분석(ADR-021) 권장이 스택별로 분기 폭증한다.
- *명시적 범위 한정 + override 경로*가 *암묵적 빈자리*보다 fork 직후 불일치 발견 시간을 단축한다.
- **"지원하지 않는다"가 아니라 "기본값으로 최적화하지 않는다"** — 본 ADR이 fork 사용자에게 *거부감*을 주지 않도록 표현을 명확히 한다.

## 결정
보일러플레이트의 *기본 자동화 + 문서 템플릿*이 직접 다루는 범위:
- web frontend (React/Next.js/Vue/Svelte/Astro 등)
- API server (FastAPI/Express/Spring/Django/Rails 등)
- CLI (Rust/Go/Python/Node 기반)
- monorepo (turbo/nx/pnpm 등으로 위 3종 결합)
- Supabase 통합 (BaaS의 대표 케이스)

기본 자동화 범위 밖 (override 경로 제공):
- mobile native (iOS Swift/Android Kotlin/RN/Flutter — RN·Flutter·SwiftUI는 Step 5.3 `/bootstrap-design` R3 표에 *override 시 시작점*만 권장)
- ML/data science (Jupyter/training pipeline)
- embedded / firmware
- game (Unity/Unreal/Godot)
- desktop (Electron 외 native — Electron은 web frontend 범주)

## fork 사용자 override 절차
fork 사용자가 기본 자동화 범위 밖 스택으로 진행하려면:
1. `/bootstrap-stack`이 본 ADR을 인용하며 *"기본 자동화 직접 지원 범위 밖입니다. 직접 보강이 필요합니다"* 출력.
2. 사용자가 `--override` 발화 시 stack-guard 출력 무시, 사용자가 ARCHITECTURE 7섹션·DESIGN_SYSTEM·검증 도구를 자유 작성.
3. 본 ADR을 인용하며 fork 프로젝트 안에 supersede ADR(예: `ADR-100-rn-stack.md`)을 박음 — *지원하지 않는다*가 아니라 *기본값 최적화 안 함*이라 override는 정상 경로다.

## 결과
- ARCHITECTURE/DESIGN_SYSTEM 자리 부재가 *결정으로 정리*된다(자리를 만들지 않는 결정).
- `validate` JS-bias 등의 비웹 마찰점이 *override 경로 + 기본 자동화 범위 명시*로 정리 — 비웹 마찰 매트릭스 컬럼이 통째 cut.
```

**2. AGENTS.md 갱신** — 본문에 *1줄* 추가 (Phase 3.4의 100줄 cap 적용 전이지만 본 줄은 *기본 자동화 범위 명시*라 cap 후에도 유지):

```markdown
## 기본 자동화 직접 지원 범위
보일러플레이트의 기본 자동화·문서 템플릿이 직접 다루는 스택은 web frontend / API server / CLI / monorepo / Supabase 통합 5종이다. 그 외(mobile / ML / embedded / game / desktop)는 fork 사용자 override 경로 제공 (ADR-031).
```

**3. STRUCTURE.md 갱신** — Canonical Owner 매핑 표에 한 줄 추가:

```markdown
| 보일러플레이트 직접 지원 스택 범위 | `docs/90-decisions/ADR-031-non-web-out-of-scope.md` |
```

**4. ADR 인덱스 갱신** — `docs/90-decisions/README.md`에 ADR-031 행 추가.

#### 검증

- [ ] `docs/90-decisions/ADR-031-non-web-out-of-scope.md` 존재
- [ ] AGENTS.md "기본 자동화 직접 지원 범위" 단락 존재 (1단락 추가)
- [ ] STRUCTURE.md Canonical Owner 표 갱신
- [ ] ADR README 표에 ADR-031 행

#### 결정 근거

Phase 3~8의 다른 결정들이 *어디에 자리를 만들지 / 만들지 않을지* 결정할 때 본 ADR의 직접 지원 범위를 따른다. 예: Phase 5의 인터페이스 결정에서 *비웹 한정 DESIGN.md 그룹은 추가하지 않는다*가 본 ADR의 자연 귀결.

#### Commit message

```
docs: add ADR-031 declaring non-web stacks out of direct support scope
```

---

### Step 2.3 — Boilerplate ADR scope 라벨링 (ADR-000 메타 신설)

**우선순위**: P0-Post-Gate (가벼움)
**Evidence**: [관측됨] (fork 사용자가 boilerplate ADR을 자기 결정과 시각적으로 구분 못 하는 마찰)
**의존성**: Step 2.1 (Ratchet 정합)

#### 현재 상태

ADR-001/004~010(8개)이 모두 *보일러플레이트 자체 정책 결정*인데, fork된 프로젝트가 이를 *자기 결정*과 시각적으로 구분 못한다. supersede 권한·번호 충돌(fork 후 새 ADR을 ADR-011부터 시작할지 ADR-100부터 시작할지) 모호.

#### 변경 후 상태

`ADR-000-boilerplate-decision-policy.md` 메타 ADR이 라벨링·supersede·번호 정책을 박는다. 기존 ADR frontmatter에 `scope: boilerplate` 필드 추가. README가 boilerplate ADR과 project ADR을 두 섹션으로 분리.

#### 작업 절차

**1. ADR-000 본문 작성** — `docs/90-decisions/ADR-000-boilerplate-decision-policy.md` 신설

```markdown
# ADR-000 — Boilerplate decision policy (메타)

## Status
accepted

## 배경
- 본 디렉터리의 ADR-001/004~010은 *이 보일러플레이트 자체*의 정책 결정이다.
- fork된 프로젝트가 자기 ADR을 박을 때 (a) 보일러플레이트 결정과 자기 결정의 시각적 구분 부재, (b) supersede 권한 모호, (c) 새 ADR 번호 시작점 모호의 마찰을 갖는다.

## 결정

### A. scope 라벨링
모든 ADR frontmatter(또는 본문 첫 줄)에 다음 표시를 둔다.
- `scope: boilerplate` — 본 보일러플레이트 자체 결정. fork 후 supersede 가능.
- `scope: project` — fork된 프로젝트의 자체 결정.

본 보일러플레이트가 박는 모든 ADR(현재 시점 박힌 ADR + 향후 보일러플레이트 영역에서 박을 ADR)은 `scope: boilerplate`로 박는다. P2 트리거 보류 ADR(039·040 등)은 트리거 발동 후 *파일 생성 시점에* 이 라벨을 박는다. **예외 — ADR-002, ADR-003**: 이 두 번호는 *보일러플레이트가 박아둔 fork-사용자용 placeholder slot*이라 `scope: project`로 표기한다 (placeholder를 fork 사용자가 채우면 그 결정은 project scope). 이 예외는 결정 C(아래)의 fork ADR 번호 정책과 정합한다.

### B. README 섹션 분리
`docs/90-decisions/README.md`를 다음 두 섹션으로 분리.
1. **Boilerplate ADR** (fork 후 supersede 가능)
2. **Project ADR** (fork된 프로젝트의 결정)

### C. fork 후 ADR 번호 정책
- fork 사용자는 ADR-011 또는 그 이상의 빈 번호를 그대로 사용하지 않는다.
- 새 프로젝트 ADR은 **ADR-100부터** 시작한다(예: `ADR-100-stack-selection.md`).
- 보일러플레이트 ADR을 supersede할 경우 *그 자리에 새 ADR을 박지 않고*, 본인 번호(ADR-100+)에서 박은 뒤 본문 첫 줄에 `Supersedes ADR-NNN (boilerplate)` 표기.
- **ADR-002, ADR-003 예외**: 이 두 번호는 *보일러플레이트가 박아둔 fork-사용자용 placeholder slot*이다. fork 사용자가 채우는 첫 결정(스택 선택·초기 결정)은 *002/003을 채워도 되고*, *ADR-100부터 새로 시작해도 된다*. 둘 중 fork 사용자가 선호하는 방식으로.

### D. supersede 권한
- fork 사용자는 boilerplate ADR을 자유롭게 supersede할 수 있다.
- supersede ADR은 *왜* 뒤집는지(프로젝트 컨텍스트·제약)를 본문에 1단락 명시.

## 결과
- fork 직후 사용자가 *어느 ADR이 내 결정인가*를 1초 안에 식별 가능.
- ADR 번호 충돌 영구 회피.
```

**2. 기존 ADR 파일에 scope 라벨링** — ADR-001/004/005/006/007/008/009/010 각 파일의 `## Status` 위에 한 줄 추가:

```markdown
> scope: boilerplate
```

(8개 파일 일괄 처리. ADR-002, ADR-003은 placeholder라 fork 사용자용 — `scope: project`로 표기.)

**3. `docs/90-decisions/README.md` 재구성**

```markdown
# ADR Index

> 이 디렉터리의 ADR을 한눈에 본다. ADR scope 정책은 [ADR-000](ADR-000-boilerplate-decision-policy.md) 참조.

## Boilerplate ADR (fork 후 supersede 가능)

| # | 제목 | 상태 | 한 줄 요약 |
|---|------|------|-----------|
| 000 | Boilerplate decision policy | accepted | scope 라벨링 + supersede + 번호 정책 |
| 001 | Doc hierarchy | accepted | docs/ 디렉터리 6분할 결정 |
| 004 | Model alias policy | accepted | shared 기본값에서 모델 별칭만 사용 |
| 005 | Single Source of Truth (SSOT) | accepted | 같은 사실은 1곳에서 정의 |
| 006 | Simplicity, Clean Code, and Clean Architecture priority | accepted | 단순성 1순위 |
| 007 | Workitem lifecycle | accepted | 8단계 lifecycle |
| 008 | Commit convention | accepted | Conventional Commits |
| 009 | TDD default + opt-out | accepted | RGR 디폴트 |
| 010 | Multi-agent compatibility (AGENTS.md as canonical entry) | accepted | Codex CLI 동등 동작 |
| (... Phase 2~9에서 추가될 ADR-011~040) | | | |

## Project ADR (fork된 프로젝트가 채움, ADR-100부터)

| # | 제목 | 상태 | 한 줄 요약 |
|---|------|------|-----------|
| 002 | Initial project decisions | (placeholder) | bootstrap 단계 초기 결정 모음 |
| 003 | Stack selection | (placeholder) | 스택 선택과 근거 |

## 신규 ADR 추가 절차
(기존 절차 유지 + ADR-000 정책 인용)
```

#### 검증

- [ ] `docs/90-decisions/ADR-000-boilerplate-decision-policy.md` 존재
- [ ] ADR-001/004~010 8개 파일 모두 `scope: boilerplate` 표시
- [ ] ADR-002, 003은 `scope: project` 표시
- [ ] README가 두 섹션으로 분리됨

#### 결정 근거

디렉터리 분리(보일러플레이트/project 별도 폴더)는 ADR-005 SSOT/단순성과 충돌해 보류. 라벨링 + 메타 ADR 결합이 최소 surface 변경으로 가장 큰 효용.

#### Commit message

```
docs: add ADR-000 meta policy with scope labels for boilerplate ADRs
```

---

## Part 3. Phase 3 — 단순 cleanup (P0-Post-Gate)

> 본 Phase는 *비용 작고 효용 큰* 정리 작업을 모은다. 여기서 docs/00-meta가 9→6로 줄어들면 다음 Phase의 cross-reference surface가 줄어든다.
>

### Step 3.1 — Claude Code plan 모드 lifecycle 비범위 (ADR-024 신설)

**우선순위**: P0-Post-Gate
**Evidence**: [관측됨] (외부 미해결 버그 [#19537](https://github.com/anthropics/claude-code/issues/19537))
**의존성**: Phase 2 완료

#### 현재 상태

- `.claude/settings.json` L5: `"plansDirectory": "./docs/30-workitems/plans"` 박혀 있음.
- `docs/30-workitems/plans/` 디렉터리 + `.gitkeep` 존재.
- `docs/00-meta/STRUCTURE.md` 산출물 표 L24에 `plan (Claude Code Plan 모드)` 행.
- `docs/30-workitems/README.md`에 `plans` 라인 존재.
- `docs/00-meta/TEMPLATE_GUIDE.md` L33~36 "설정과 경로 매핑" 단락이 plansDirectory를 가리킴.

#### 변경 후 상태

- plansDirectory 라인 삭제 → 빌트인 디폴트(`~/.claude/plans/`) 회귀.
- `docs/30-workitems/plans/` 디렉터리 통째 삭제.
- 관련 문서 4곳 정리.
- AGENTS.md에 Claude Code plan 모드 자율성 1단락.
- `/implement-workitem`에 think-before-edit 1줄 흡수.

#### 작업 절차

1. **`.claude/settings.json` 갱신** — `plansDirectory` 라인 삭제.
2. **디렉터리 삭제** — `docs/30-workitems/plans/` 통째 삭제.
3. **STRUCTURE.md 갱신** — 산출물 표에서 `plan (Claude Code Plan 모드)` 행 삭제.
4. **`docs/30-workitems/README.md` 갱신** — `plans` 관련 라인 삭제 (이 파일 자체는 Step 3.2에서 삭제 예정이지만 본 Step에서 미리 정리).
5. **`docs/00-meta/TEMPLATE_GUIDE.md` 갱신** — L33~36 "설정과 경로 매핑" 단락 삭제 (이 파일도 Step 3.2에서 흡수 후 삭제 예정).
6. **AGENTS.md 갱신** — "단순성·YAGNI" 단락 아래에 신설:

   ```markdown
   ## Claude Code plan 모드
   Claude Code의 빌트인 plan 모드(Shift+Tab)는 사용자 자율 도구다. 본 보일러플레이트의 lifecycle은 plan 모드를 의무화하지 않으며 산출물 경로(`plansDirectory`)도 강제하지 않는다. Codex 사용자도 동등한 흐름을 갖는다 (ADR-010, ADR-024).
   ```

7. **`/implement-workitem` skill 갱신** — `.claude/skills/implement-workitem/SKILL.md` Red phase 진입 직전 단락에 다음 1줄 추가:

   ```markdown
   Red phase 진입 직전, 출력의 첫 단락으로 "이 task에서 어떤 테스트를 어떤 순서로 작성할 것인가"를 1~3문장으로 명시한다(plan 모드 의존 없이 think-before-edit 규율 확보).
   ```

8. **`/plan-workitem` skill description 갱신** — `.claude/skills/plan-workitem/SKILL.md` frontmatter `description` 1줄 갱신 (rename(No) 결정의 *대체 조치* — 자동완성 표면에서 빌트인 `/plan` 모드와의 정신 모델 정합):

   ```diff
   - description: 상위 설계 문서를 기반으로 milestone, feature, task 단위 문서를 생성하거나 정리할 때 사용한다.
   + description: 상위 설계 문서를 기반으로 milestone, feature, task 단위 문서를 생성하거나 정리할 때 사용한다 (Claude Code plan 모드와 다름 — workitem 분해기).
   ```

9. **ADR-024 본문 작성** — `docs/90-decisions/ADR-024-plan-mode-out-of-lifecycle.md`. 본문 핵심:
   - scope=boilerplate
   - 결정 5가지: settings 1줄 / 디렉터리 / 문서 4곳 / AGENTS.md 1단락 / skill 2개(implement-workitem 1줄 + plan-workitem description 1줄)
   - 재검토 트리거 3종: (a) Codex 동등 plan 모드 도입 (b) plan 모드가 milestone/feature/task 분해 제공 (c) #19537 fix
   - 잔여 모니터링: fork 다운스트림 사례 0건 → [가설] 라벨. 첫 fork 사용자 피드백이 [관측됨] 승격 경로. 사용자가 plan 모드를 lifecycle에 다시 끼워 넣고 싶다는 요청 누적 시 트리거 (b) 도달로 간주

10. **ADR README 갱신**.

#### 검증

- [ ] `.claude/settings.json` 에서 plansDirectory 라인 부재
- [ ] `docs/30-workitems/plans/` 디렉터리 부재
- [ ] STRUCTURE.md / 30-workitems/README.md / TEMPLATE_GUIDE.md 정리됨
- [ ] AGENTS.md "Claude Code plan 모드" 단락 존재
- [ ] `/implement-workitem` Red phase 직전 think-before-edit 1줄 추가됨
- [ ] `/plan-workitem` skill `description`에 "Claude Code plan 모드와 다름" 명시 추가됨
- [ ] ADR-024 본문 + README 갱신

#### Commit message

```
docs: add ADR-024 removing Claude Code plan mode from lifecycle
```

---

### Step 3.2 — docs/00-meta 9→6 흡수 (ADR-012 1단계)

**우선순위**: P0-Post-Gate
**Evidence**: [외부실증] (Diátaxis + ADR-005 SSOT 정합 검증)
**의존성**: Step 3.1

#### 현재 상태

`docs/00-meta/`에 9문서 + `docs/30-workitems/README.md` 1개 = 흡수 대상 4개:
- **TEMPLATE_GUIDE.md** (52라인) — *문서 계층 정의*는 ADR-001 중복, *네이밍 규칙*은 STRUCTURE.md 흡수.
- **LOCAL_SETTINGS_EXAMPLE.md** (21라인) — GUARDRAILS_STRATEGY.md "local 자동화" 단락에 7줄 압축 흡수.
- **BOOTSTRAP_PROMPT_EXAMPLES.md** (36라인) — NEW_PROJECT_CHECKLIST.md "1. 저장소 복제 직후" 단계에 인라인 코드블록 흡수.
- **`docs/30-workitems/README.md`** (25라인) — STRUCTURE.md 산출물 표가 SSOT. 단순 삭제.

#### 변경 후 상태

`docs/00-meta/`에 6문서: STRUCTURE / WORKFLOW / AGENT_EXECUTION_STRATEGY / GUARDRAILS_STRATEGY / NEW_PROJECT_CHECKLIST / GLOSSARY.

#### 작업 절차

1. **TEMPLATE_GUIDE 흡수** — 네이밍 규칙 4줄을 STRUCTURE.md "절차" 섹션 위로 이전. 문서 연결 원칙 3줄도 함께. 원본 삭제.
2. **LOCAL_SETTINGS_EXAMPLE 흡수** — GUARDRAILS_STRATEGY.md "local 자동화 권장 원칙" 단락 아래에 7줄(개인 환경 hook / 실험적 설정 / 민감 환경 변수 / 형식은 Claude Code 공식 문서 참조) 압축. 원본 삭제.
3. **BOOTSTRAP_PROMPT_EXAMPLES 흡수** — NEW_PROJECT_CHECKLIST.md 1단계에 핵심 예시 3종(`/bootstrap-project`, `/bootstrap-stack`, `/discover-product`)만 인라인 코드블록으로 흡수. 원본 삭제.
4. **`docs/30-workitems/README.md` 단순 삭제**.
5. **cross-reference 갱신** — 다음 위치 점검:
   - `AGENTS.md` "깊은 운영 원칙" 인덱스에서 TEMPLATE_GUIDE 줄 삭제 → 5행으로 줄어듦.
   - `docs/00-meta/STRUCTURE.md` Canonical Owner 표에서 TEMPLATE_GUIDE 가리키는 행을 STRUCTURE.md(자기 자신) + ADR-001로 재매핑.
   - `.claude/skills/*/SKILL.md`의 `반드시 먼저 읽을 파일` 목록 점검.
   - 전체 grep으로 4개 파일명 인용 0건 확인.

#### 검증

- [ ] `docs/00-meta/`에 6문서만 남음
- [ ] `docs/30-workitems/README.md` 부재
- [ ] AGENTS.md "깊은 운영 원칙" 인덱스 5행 이하
- [ ] grep으로 흡수 대상 파일명 인용 0건

#### Commit message

```
docs: absorb 4 docs into core 6 under docs/00-meta per SSOT cleanup
```

---

### Step 3.3 — Diátaxis 모드 라벨 + `/review-doc` 보강 (ADR-012 2단계)

**우선순위**: P0-Post-Gate
**Evidence**: [외부실증] (Diátaxis 공식 가이드)
**의존성**: Step 3.2

#### 변경 후 상태

각 docs/00-meta/*.md 파일 첫 줄에 모드 라벨 추가. `/review-doc` skill이 모드 mismatch 검출 시 IMPROVEMENT_GUIDE에 P1 severity로 보고.

#### 작업 절차

1. **모드 라벨 추가 (6개 파일)** — 각 파일 `# 제목` 다음 줄에 다음 1줄 추가:

   | 파일 | 모드 |
   |------|------|
   | `STRUCTURE.md` | `> 모드: Reference (산출물 인벤토리)` |
   | `WORKFLOW.md` | `> 모드: Reference + How-to (워크플로우 정의 + 단계별 절차)` |
   | `AGENT_EXECUTION_STRATEGY.md` | `> 모드: Reference (위임 트리거 + 메인 세션 역할)` |
   | `GUARDRAILS_STRATEGY.md` | `> 모드: Explanation (guardrail 운영 원칙의 근거)` |
   | `NEW_PROJECT_CHECKLIST.md` | `> 모드: How-to (새 프로젝트 시작 체크리스트)` |
   | `GLOSSARY.md` | `> 모드: Reference (용어 정의)` |

2. **`/review-doc` skill 갱신** — `.claude/skills/review-doc/SKILL.md` 본문에 다음 2줄 추가:

   ```markdown
   - 본문 내용이 첫 줄의 모드 라벨(`> 모드: ...`)과 정합한지 점검.
   - mismatch 발견 시 `docs/40-validation/IMPROVEMENT_GUIDE.md`에 **P1 severity**로 보고.
   ```

3. **ADR-012 본문 작성** — `docs/90-decisions/ADR-012-doc-architecture-cleanup.md`. 본문 핵심: scope=boilerplate / 결정 A(00-meta 흡수) + 결정 B(Diátaxis 모드 라벨) / 비결정 No(00-meta 폴더명 변경 영구 No, `_templates/` 통합 영구 No).
4. **ADR README 갱신**.

#### 검증

- [ ] 6개 docs/00-meta 파일 모두 첫 줄 모드 라벨
- [ ] `/review-doc` skill 점검 2줄 추가
- [ ] ADR-012 본문 + README

#### Commit message

```
docs: add ADR-012 with diataxis mode labels and review-doc check
```

---

### Step 3.4 — AGENTS.md 100줄 hard cap (ADR-011 신설)

**우선순위**: P0-Post-Gate
**Evidence**: [외부실증] (Augment 2,500-repo 분석)
**의존성**: Step 3.3 (00-meta 흡수 후 인덱스 줄어든 상태)

#### 현재 상태

AGENTS.md 약 35~50줄(Phase 2~3.1에서 *직접 지원 범위* 1단락 + *Claude Code plan 모드* 1단락 추가됨). cap 정책 부재 → 향후 정책 추가 시 무한 비대 위험.

#### 작업 절차

1. **AGENTS.md 본문 끝에 cap 정책 1단락 추가**:

   ```markdown
   ## AGENTS.md 길이 정책
   본 문서는 **100줄 hard cap**(soft cap 80줄)을 적용한다. 새 정책은 ADR로 박고, 본 문서에는 한 줄 + 링크만 둔다 (ADR-011).
   ```

2. **`/review-doc` skill 갱신** — 다음 1줄 추가:

   ```markdown
   - AGENTS.md 길이 점검: 100줄 초과 시 IMPROVEMENT_GUIDE에 P0 severity로 보고. 80~100줄 사이는 P1.
   ```

3. **AGENTS.md trace-to-failure retrospective** — PR 본문에 기존 5개 핵심 행동 규율 각각의 ADR/사건 매핑 1회 정리:
   - "상위 문서 없이 하위 문서 먼저 만들지 않는다" → ADR-001 (문서 계층)
   - ".env, secrets/ 민감 파일 건드리지 않는다" → settings.json deny 정책
   - "작업 범위와 비범위를 명확히..." → ADR-007 (workitem lifecycle)
   - "흩어진 임시 메모보다 정해진 위치 문서 갱신" → ADR-005 (SSOT)
   - "사실, 가정, 열린 질문 구분" → ADR-022 (evidence label)

4. **ADR-011 본문 작성** — `docs/90-decisions/ADR-011-agents-md-line-cap.md`. 본문 핵심: scope=boilerplate / 결정(100줄 hard / 80줄 soft / 새 정책=ADR + 1줄 단락 / `/review-doc` cap 점검) / evidence label [외부실증] / Ratchet 약 적용(즉시 차단 X, reviewer 보고 → 사용자 결정).
5. **ADR README 갱신**.

#### 검증

- [ ] AGENTS.md 본문 ≤ 100줄
- [ ] `/review-doc` skill cap 점검 1줄
- [ ] ADR-011 본문 + README
- [ ] PR 본문 retrospective 5개 매핑

#### Commit message

```
docs: add ADR-011 enforcing AGENTS.md 100-line hard cap
```

---

## Part 4. Phase 4 — Schema 강화 (P0-Post-Gate)

> 본 Phase는 *workitem schema*에 deterministic 형식을 박는다. AC 형식·sizing 한계·lock file 처리·monorepo scope 4종이 묶음. Phase 4가 먼저 끝나야 Phase 5의 인터페이스 결정 흐름에서 task 분해가 새 schema 위에서 동작한다.
>

### Step 4.1 — TASK_TEMPLATE schema 강화 (ADR-026 신설)

**우선순위**: P0-Post-Gate
**Evidence**: [관측됨+외부실증] (외부 SDD 자료 [Fowler SDD analysis](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html))
**의존성**: Phase 3 완료

#### 현재 상태

`docs/30-workitems/_templates/TASK_TEMPLATE.md`의 `## 6. Acceptance Criteria` 주석은 *Given-When-Then 또는 명세 형태*라 "or" 옵션 → 자유도 큼. AC 개수 cap·verb whitelist·의존성 자리 없음. sizing 한계(1 task = 1 RGR / 변경 파일 수) 가이드 부재.

#### 변경 후 상태

- AC는 Given-When-Then **형식 강력 권장** (자동 차단 X, 재분해 권장 텍스트).
- measurable verb 권장 예시 (returns / displays / persists / rejects / emits / responds with / contains / matches) + 강력 금지 verb 4개("works"/"looks good"/"is correct"/"is fine"). 문맥상 허용 verb("handles"/"supports")는 *무엇을 / 어떻게*가 명시되면 통과.
- AC 3개 이하 권장(4개 이상이면 task 분해 *권장 텍스트*).
- 변경 예정 파일 5개 이하 권장 (초기 scaffolding·auth task는 초과 자연 — 사용자 결정).
- `## 9. 의존성` 신설.
- planner skill self-check 3줄(sizing / AC 형식 / 의존성).
- **quality lint 모델** — hard gate 아닌 *권장 + 재분해 텍스트*. LLM이 자주 막히지 않도록 자동 차단 없음.

#### 작업 절차

1. **TASK_TEMPLATE.md 갱신** — `docs/30-workitems/_templates/TASK_TEMPLATE.md`의 `## 6. Acceptance Criteria` 주석 교체:

   ```markdown
   ## 6. Acceptance Criteria
   <!-- AC는 Given-When-Then *형식 강력 권장*. measurable verb 사용:
        권장(좋은 예): returns, displays, persists, rejects, emits, responds with, contains, matches
        강력 금지(절대 비측정): works, looks good, is correct, is fine
        문맥상 허용: handles, supports — 단 *무엇을 / 어떻게*까지 명시되면 허용
        AC 3개 이하 권장(4개 이상이면 task 분해 *권장 텍스트*).
        위반 시 planner는 *재분해 권장 텍스트*를 출력, builder-sonnet은 *재분해 요청 텍스트*를 Red phase 직전 출력 — 자동 차단은 하지 않는다(사용자 결정). -->
   - AC-1 [Given] ... [When] ... [Then] ...
   - AC-2 [Given] ... [When] ... [Then] ...
   ```

2. **`## 9. 의존성` 신설** — `## 8. 메모` 위에 추가:

   ```markdown
   ## 9. 의존성
   <!-- 형식: `- T-002: T-001의 X 정의 후 시작 가능`. 비어 있으면 병렬 가능으로 간주. -->
   ```

3. **plan-workitem skill 갱신** — `.claude/skills/plan-workitem/SKILL.md` "반드시 수행할 일" 단락에 다음 3개 추가:

   ```markdown
   8. **분해 후 sizing self-check** — 다음 3 한계 중 하나라도 초과 시 *추가 분해 권장 텍스트*를 출력에 명시 (자동 차단 X, 사용자 결정):
      - 1 task = 1 RGR 사이클.
      - AC 3개 이하.
      - 변경 예정 파일(TASK_TEMPLATE `## 4-1`) 5개 이하.
      - 초기 scaffolding·auth 같은 task는 5개 파일 초과가 자연스럽다 — 사용자가 분해 거부 결정 가능.
   9. **AC 형식 권장 + 금지 verb 점검** — 모든 AC는 Given-When-Then + measurable verb 권장(TASK_TEMPLATE 주석 참조). 강력 금지 verb("works"/"looks good"/"is correct"/"is fine") 사용 시 *재분해 권장 텍스트* 출력. 문맥상 허용 verb("handles"/"supports")는 *무엇을 / 어떻게*가 명시되면 통과.
   10. **task 의존성 채움** — TASK_TEMPLATE `## 9. 의존성`을 분해 시 명시. 병렬 가능 task는 비워둔다.
   ```

4. **plan-workitem 출력 형식 갱신** — skill 본문에 plan 출력 마지막에 다음 매트릭스 1개 추가 안내:

   ```
   | Milestone | Feature | Task  | AC 수 | 의존성  |
   |-----------|---------|-------|-------|--------|
   | M1        | F-001   | T-001 | 2     | -      |
   | M1        | F-001   | T-002 | 3     | T-001  |
   ```

5. **builder-sonnet 갱신** — `.claude/agents/builder-sonnet.md`에 1줄 추가:

   ```markdown
   - AC가 Given-When-Then 형식이 아니거나 강력 금지 verb 사용 시 Red phase 진입 직전에 *재분해 요청 텍스트*를 출력 — 자동 차단은 하지 않고 사용자가 진행/재분해 결정 (ADR-007 lifecycle 정합 — 자동 차단 X).
   ```

6. **ADR-026 본문 작성** — `docs/90-decisions/ADR-026-plan-workitem-schema.md`. 본문 핵심:
   - scope=boilerplate
   - 결정 5종: AC Given-When-Then 권장 / verb 권장+강력 금지 / sizing 3한계 / `## 9. 의존성` / planner self-check 3줄
   - **quality lint 모델** — hard gate 아닌 *권장 + 재분해 텍스트*. ADR-007의 *판정+권장만* 책임 경계와 정합. handles/supports 같은 verb는 문맥상 허용. 자동 차단은 ADR-022 Ratchet 약 적용 정합(예방적 가설을 강제로 박지 않음).
   - 비결정 No: 2-pass planning(토큰 2배+stabilize reviewer 책임 중복) / risk·effort 추정(보일러플레이트가 정확도 보장 불가, YAGNI) / `/plan-workitem` rename(skill description 1줄로 충분) / hard gate verb whitelist 자동 차단(과강제, LLM 빈번 막힘 위험)
   - evidence label [관측됨] (vague AC가 LLM TDD 실패의 단일 최대 원인 — 외부 SDD 자료) + [외부실증]
   - 잔여 모니터링: 첫 마일스톤 *재분해 권장 텍스트 발화율* > 50% 시 verb 정의 갱신 (분해 거부율 아님 — 차단 X)

7. **ADR README 갱신**.

#### 검증

- [ ] TASK_TEMPLATE.md `## 6` 주석 + AC-1/AC-2 형식 변경
- [ ] TASK_TEMPLATE.md `## 9. 의존성` 신설
- [ ] plan-workitem skill 8/9/10 항목 추가
- [ ] builder-sonnet self-check 1줄 추가
- [ ] ADR-026 본문 + README

#### Commit message

```
docs: add ADR-026 with TASK_TEMPLATE schema and planner self-check
```

---

### Step 4.2 — AC ID 컨벤션 강화 (ADR-009 amend, P1로 시작 → 데이터 트리거 시 P0 격상)

**우선순위**: P1 (현 시점 P1 경고, Round 2 누락률 ≤ 5% 도달 시 P0로 격상)
**Evidence**: [관측됨] (ADR-009 후속 작업의 명문화)
**의존성**: Step 4.1

#### 현재 상태

ADR-009 본문에 *후속 작업*으로 AC ID 컨벤션 명시되어 있지만 *권장*에 그침. validator-sonnet이 자연어 매칭에 의존 → false positive 위험.

#### 변경 후 상태

validator-sonnet이 AC ↔ 테스트 매핑 시 테스트 이름에 `AC_N` 또는 `[AC-N]` 식별자 누락 시 **P1 경고**. Phase 12 (Round 2) 또는 후속 실 마일스톤에서 누락률 ≤ 5% 도달 시 P0로 격상.

#### 작업 절차

1. **ADR-009 amend** — `docs/90-decisions/ADR-009-tdd-default.md` 본문 끝에 단락 추가:

   ```markdown
   ## Amendment 1 (2026-05-XX) — AC ID 컨벤션 강화 (P1 경고, 데이터 트리거 시 P0 격상)
   ### 결정
   - 테스트 이름은 `AC_N` 또는 `[AC-N]` 식별자를 포함한다 (예: `test_AC_1_unauthenticated_returns_401`).
   - validator-sonnet이 매핑 시 식별자 누락을 발견하면 IMPROVEMENT_GUIDE에 **P1 severity**로 보고.
   - Phase 12 (Round 2) 또는 후속 실 마일스톤에서 누락률 ≤ 5% 도달 시 P0로 격상 (재amend).

   ### 근거
   - ADR-009 후속 작업의 명문화.
   - 자연어 매칭 false positive 차단.
   ```

2. **validator-sonnet 갱신** — `.claude/agents/validator-sonnet.md` 본문에 1줄 추가:

   ```markdown
   - 테스트 이름에 `AC_N` 또는 `[AC-N]` 식별자 누락 시 IMPROVEMENT_GUIDE에 P1 severity로 보고. ADR-009 amend 정합.
   ```

3. **TASK_TEMPLATE.md 주석 갱신** — `## 6-1. 테스트 시나리오 (TDD Red)` 주석 예시를 식별자 포함 형태로 명시:

   ```markdown
   <!-- 각 AC에 대응하는 테스트 파일·테스트 이름. 사람이 미리 채우거나 builder-sonnet이 Red phase 시작 전에 채운다.
        테스트 이름에 `AC_N` 또는 `[AC-N]` 식별자 포함 강제 (ADR-009 amend).
        예:
        - AC-1 → tests/auth/me.spec.ts > test_AC_1_unauthenticated_returns_401
        - AC-2 → tests/auth/me.spec.ts > test_AC_2_authenticated_returns_user -->
   ```

#### 검증

- [ ] ADR-009 본문 끝에 Amendment 1 단락
- [ ] validator-sonnet 본문에 식별자 점검 1줄
- [ ] TASK_TEMPLATE.md 6-1 주석 갱신

#### Commit message

```
docs: amend ADR-009 to enforce AC ID convention in test names
```

---

### Step 4.3 — finalize lock file 화이트리스트 (ADR-007 amend)

**우선순위**: P0-Post-Gate
**Evidence**: [가설→Gate 후 실증] (Phase 1 시뮬레이션에서 발견될 가능성 큼)
**의존성**: Phase 3 완료

#### 현재 상태

`/finalize-workitem`이 task `## 4-1. 변경 예정 파일/경로`에 명시되지 않은 staged 파일 발견 시 `Needs Review`로 종료. lock 파일(`pnpm-lock.yaml` 등)은 task와 무관한 자동 생성물인데도 매번 차단 → 마찰점.

#### 작업 절차

1. **ADR-007 amend** — `docs/90-decisions/ADR-007-workitem-lifecycle.md` 본문 끝에 단락 추가:

   ```markdown
   ## Amendment 1 (2026-05-XX) — finalize lock file 화이트리스트

   ### 결정
   `/finalize-workitem`의 우선순위 (3) 제외 규칙에 다음 11종 lock 파일을 자동 화이트리스트로 추가한다 (TASK_TEMPLATE `## 4-1`에 명시되지 않아도 자동 add 허용):
   - `pnpm-lock.yaml`
   - `package-lock.json`
   - `yarn.lock`
   - `bun.lockb`
   - `Cargo.lock`
   - `Gemfile.lock`
   - `composer.lock`
   - `go.sum`
   - `Pipfile.lock`
   - `poetry.lock`
   - `uv.lock`

   그 외 신규 dependency 변경(예: `package.json` 의 `dependencies`/`devDependencies` 키 추가)은 reviewer P1로 보고.

   ### 근거
   - lock file은 task 단위 변경의 부산물 → `## 4-1` 강제는 단순성 위반.

   ### 잔여 모니터링
   - 11종 외 신규 패키지 매니저 등장 시 누락 위험. stabilize가 staged된 `*.lock` / `*-lock.*` 패턴 중 화이트리스트 미일치 1건 발견 시 P1로 보고.
   ```

2. **`/finalize-workitem` skill 갱신** — `.claude/skills/finalize-workitem/SKILL.md` 본문에서 우선순위 (3) 제외 규칙에 lock file 11종 화이트리스트 명시 + ADR-007 amend 인용.

#### 검증

- [ ] ADR-007 본문 Amendment 1 단락
- [ ] finalize-workitem skill 화이트리스트 명시

#### Commit message

```
docs: amend ADR-007 with finalize lock file whitelist
```

---

### Step 4.4 — monorepo Conventional Commits scope (ADR-008 amend 1차)

**우선순위**: P0-Post-Gate
**Evidence**: [외부실증] (Conventional Commits 표준)
**의존성**: Phase 3 완료

#### 현재 상태

ADR-008은 Conventional Commits를 박았지만 *모노레포 scope 정책* 부재. fork 사용자가 모노레포에서 scope를 즉흥 결정 → 일관성 위반.

#### 작업 절차

1. **ADR-008 amend** — `docs/90-decisions/ADR-008-commit-convention.md` 본문 끝에 단락 추가:

   ```markdown
   ## Amendment 1 (2026-05-XX) — 모노레포 scope 컨벤션

   ### 결정
   모노레포 감지 시 Conventional Commits scope = 패키지명. 예:
   - `feat(api): /me 엔드포인트 추가`
   - `feat(web): 로그인 폼 검증`
   - `feat(shared): Date 유틸 함수`

   `/bootstrap-stack`의 monorepo 라운드(Step 9.4)가 scope vocabulary 목록을 박는다.

   ### 근거
   - 모노레포에서 scope 즉흥 결정은 git log 가독성 저하 + 자동 changelog 분기 어려움.
   ```

2. **단일-repo는 변화 없음** — ADR-008 본문이 단일 repo 케이스 그대로 유지.

#### 검증

- [ ] ADR-008 본문 Amendment 1 단락

#### Commit message

```
docs: amend ADR-008 to add monorepo scope as package name
```

---

## Part 5. Phase 5 — 인터페이스 결정 흐름 (ADR-027)

> 본 Phase는 *인터페이스 결정 책임 분배*(UI 시각·API·CLI) + *ARCHITECTURE 백엔드/프론트 sub-section*을 한 ADR(ADR-027)로 묶어 처리한다. 둘 다 *ARCHITECTURE 7-1/7-2 자리*를 만들고 채우는 한 흐름이다.
>

### Step 5.1 — DESIGN_SYSTEM.md → DESIGN.md rename + UI 한정

**우선순위**: P0-Post-Gate
**Evidence**: [외부실증] + [관측됨]
**의존성**: Phase 4 완료

#### 현재 상태

- `docs/20-system/DESIGN_SYSTEM.md`가 UI / API/백엔드 / CLI 3 그룹 placeholder (광의 SSOT 의도).
- 백엔드 그룹의 "API 로깅/관측성"이 `ARCHITECTURE_OVERVIEW.md`의 "운영성"과 책임 중복.
- 백엔드 개발자에게 *DESIGN*이 *시각 디자인*으로 misnomer.

#### 변경 후 상태

- `DESIGN_SYSTEM.md` → `DESIGN.md` rename + UI 한정.
- 본문은 UI 그룹만 남김. API/백엔드·CLI 그룹은 ARCHITECTURE 7-1/7-2로 이전(Step 5.2).

#### 작업 절차

1. **파일 rename** — `docs/20-system/DESIGN_SYSTEM.md` → `docs/20-system/DESIGN.md`.

   ```powershell
   Move-Item C:\Users\kbwdesktop\Desktop\dev-practice\my-agentic-app\docs\20-system\DESIGN_SYSTEM.md C:\Users\kbwdesktop\Desktop\dev-practice\my-agentic-app\docs\20-system\DESIGN.md
   ```

2. **본문 재구성** — `docs/20-system/DESIGN.md` 전체를 다음 [Stitch DESIGN.md canonical 섹션 순서](https://github.com/google-labs-code/design.md/blob/main/docs/spec.md)에 정렬해 placeholder만 둔다:

   ```markdown
   # 디자인 (UI)
   > 모드: Reference + How-to (UI 시각 결정의 SSOT)

   ## 0. Status
   draft

   <!-- 본 문서는 UI 프로젝트일 때만 만들어진다. /bootstrap-design이 R0~R4 라운드를 통해 채운다.
        비-UI 프로젝트(API 서버 / CLI 도구)는 본 파일을 만들지 않는다. -->

   ## 1. Overview
   <!-- 디자인 원칙 3~5개 (actionable verb. "modern/clean/sleek" 같은 모호어 금지) -->

   ## 2. Colors
   <!-- 3-tier 토큰 (DTCG): primitive(blue-100..900) → semantic(color/text/primary) → component(button/bg/primary) -->

   ## 3. Typography
   <!-- 1~2 family, 4~5 size scale, modular ratio (1.125/1.25/1.333), weight pair -->

   ## 4. Layout
   <!-- 4 또는 8 단위 base spacing, t-shirt scale 또는 numeric -->

   ## 5. Elevation & Depth
   <!-- shadow scale + radius scale -->

   ## 6. Shapes
   <!-- 컴포넌트 모서리 / 컨테이너 형태 -->

   ## 7. Components
   <!-- primitives (Button/Input/Text/Icon), composites (Card/Modal/Toast), patterns (Form/EmptyState/ErrorState/LoadingState).
        각 컴포넌트마다 상태 매트릭스 강제: default / hover / active / focus / disabled / loading / error / empty. -->

   ## 8. Motion
   <!-- duration/easing + `prefers-reduced-motion` 분기. Material 3 기준: 라우팅 UI 160~240ms, entrance/exit 240~360ms -->

   ## 9. Do's and Don'ts
   <!-- explicit prohibition (designproject.io 합의):
        - 색 5색 이내 / raw hex 금지
        - Inter·Roboto·Arial 디폴트 금지
        - 3-column icon grid 디폴트 금지
        - hierarchy는 size+weight+color 중 2축 이상
        - 한 화면 primary CTA 2개 이상 금지
        - 모든 motion에 `prefers-reduced-motion` 분기
        - 모든 컴포넌트에 empty/loading/error 상태 정의 -->
   ```

3. **STRUCTURE.md 갱신** — 산출물 표 갱신:

   ```diff
   - | design system | `docs/20-system/DESIGN_SYSTEM.md` | `/bootstrap-project` (필요 시 사용자가 수동 보강) | Living |
   + | design (UI only) | `docs/20-system/DESIGN.md` | `/bootstrap-design` (UI 스택 포함 시) | Living |
   ```

4. **`/bootstrap-project` output checklist 갱신** — `.claude/skills/bootstrap-project/output-checklist.md`에서 `DESIGN_SYSTEM.md` 라인 제거 (`/bootstrap-design`이 별도로 채움).

5. **AGENTS.md "깊은 운영 원칙" 인덱스에 한 줄 추가** (UI 프로젝트일 때만):

   ```markdown
   - [시각 디자인](docs/20-system/DESIGN.md) (UI 프로젝트 한정)
   ```

#### 검증

- [ ] `docs/20-system/DESIGN_SYSTEM.md` 부재
- [ ] `docs/20-system/DESIGN.md` 존재 + Stitch canonical 9 섹션
- [ ] STRUCTURE.md 산출물 표 갱신
- [ ] bootstrap-project output checklist 갱신

---

### Step 5.2 — ARCHITECTURE 7-1 (API), 7-2 (CLI), 7-3 (백엔드 결정), 7-4 (프론트 결정) 신설

**우선순위**: P0-Post-Gate
**Evidence**: [외부실증] + [가설] (인터페이스 결정 + 백엔드/프론트 sub-section 통합)
**의존성**: Step 5.1

#### 현재 상태

`docs/20-system/ARCHITECTURE_OVERVIEW.md`는 1~10 섹션. API envelope·error code·CLI 출력 포맷·DB migration·라우팅 등의 자리 부재.

#### 변경 후 상태

7번 "기술 선택" 아래에 4개 sub-section 신설:
- `## 7-1. API 컨벤션` (envelope / error code / 네이밍 / 페이지네이션 / Don'ts)
- `## 7-2. CLI 컨벤션` (출력 포맷 / 플래그·명령어 / TTY/ANSI / Don'ts)
- `## 7-3. 백엔드 결정` (8 prompt: DB migration / 인증 / 트랜잭션 / Idempotency / Rate limit / Async / Caching / API versioning)
- `## 7-4. 프론트 결정` (7 prompt: 라우팅 / 상태관리 / SSR-CSR / i18n / SEO / 인증 / 폼 validation)

각 sub-section은 *질문 prompt만*. 답은 사용자(또는 `/bootstrap-stack`이 architect-opus 단발 호출로 채움). 비-API/비-CLI/비-백엔드/비-프론트 프로젝트는 통째로 삭제 가능.

#### 작업 절차

1. **ARCHITECTURE_OVERVIEW.md 갱신** — `## 7. 기술 선택` 아래에 다음 4 sub-section 추가:

   ```markdown
   ## 7-1. API 컨벤션
   <!-- API 스택일 때만 채운다. /bootstrap-stack이 architect-opus 단발 호출로 채울 수 있다.
        비-API 프로젝트는 통째 삭제. -->

   ### 응답 envelope
   <!-- 예: `{ data, error, meta }` 또는 RFC 7807 problem+json 등 -->

   ### HTTP 상태 코드 매핑
   <!-- 비즈니스 에러 ↔ HTTP 상태 매핑 표 -->

   ### error 레지스트리
   <!-- 도메인 에러 코드 일람. 예: `USER_NOT_FOUND` (404), `INVALID_INPUT` (400) -->

   ### 네이밍
   <!-- 단/복수형, snake_case vs camelCase, 자원 vs 액션 -->

   ### 페이지네이션
   <!-- offset / cursor / keyset, 응답 형식 -->

   ### Don'ts
   <!-- 예:
        - envelope 변경 금지
        - error code ad-hoc 추가 금지
        - endpoint 단/복수형 혼용 금지
        - 비차단 fail이 200 OK로 가는 패턴 금지 -->

   ## 7-2. CLI 컨벤션
   <!-- CLI 라이브러리 사용 시만 채운다. /bootstrap-stack이 architect-opus 단발 호출로 채울 수 있다.
        비-CLI 프로젝트는 통째 삭제. -->

   ### 출력 포맷
   <!-- text / JSON / table 모드 + 기본 모드 / TTY 감지 정책 -->

   ### 플래그·명령어
   <!-- 명령어 트리, 플래그 컨벤션(`--`/`-`), 도움말 포맷 -->

   ### TTY/ANSI 정책
   <!-- 색상 / progress bar / TTY 미감지 시 fallback -->

   ### Don'ts
   <!-- 예:
        - JSON/표 출력 모드 일관성 위반 금지
        - TTY 감지 없는 ANSI 색 금지
        - interactive prompt가 `--yes`로 우회되지 않는 패턴 금지 -->

   ## 7-3. 백엔드 결정
   <!-- 백엔드 스택일 때만 채운다. /bootstrap-stack이 채움 권장.
        비-백엔드 프로젝트는 통째 삭제. -->

   ### DB migration
   <!-- 도구 / 버전 관리 / rollback 정책 -->

   ### 인증·인가
   <!-- 세션 vs JWT / OAuth / RBAC vs ABAC -->

   ### 트랜잭션 경계
   <!-- 어디서 begin/commit / nested 처리 -->

   ### Idempotency
   <!-- key 정책 / TTL / 중복 응답 -->

   ### Rate limit
   <!-- per-user / per-endpoint / 응답 헤더 -->

   ### Async job
   <!-- queue / worker / retry 정책 -->

   ### Caching
   <!-- HTTP cache / app cache / invalidation -->

   ### API versioning
   <!-- header / URL / breaking 정책 -->

   ## 7-4. 프론트 결정
   <!-- 프론트 스택일 때만 채운다. /bootstrap-design 또는 /bootstrap-stack이 채움 권장.
        비-프론트 프로젝트는 통째 삭제. -->

   ### 라우팅
   <!-- file-based vs config-based / 동적 라우트 / 가드 -->

   ### 상태관리
   <!-- 글로벌 store 사용 여부 / 서버 상태(React Query 등) / 폼 상태 -->

   ### SSR-CSR
   <!-- 페이지별 렌더링 모드 / 데이터 로딩 정책 -->

   ### i18n
   <!-- 라이브러리 / 사용자 언어 감지 / RTL -->

   ### SEO
   <!-- meta / sitemap / 구조화 데이터 / canonical URL -->

   ### 인증
   <!-- 토큰 저장(쿠키/storage) / refresh / OAuth 콜백 -->

   ### 폼 validation
   <!-- 라이브러리 / async validation / 에러 표시 정책 -->
   ```

2. **`/bootstrap-stack` skill 갱신** — `.claude/skills/bootstrap-stack/SKILL.md`에 본문 한 줄 + 행동 단락 추가:

   ```markdown
   API/CLI 인터페이스 컨벤션은 ARCHITECTURE_OVERVIEW.md의 7-1/7-2에 박는다. 백엔드/프론트 결정은 7-3/7-4에 박는다.
   ```

   추가 행동 단락:

   ```markdown
   ## sub-section 자동 채움
   - API 스택 감지 시 architect-opus 단발 sub-call로 ARCHITECTURE 7-1 + 7-3 채움.
   - CLI 스택 감지 시 같은 방식으로 7-2 채움.
   - 프론트 스택 감지 시 7-4 채움 (7-4의 *시각* 부분은 `/bootstrap-design`이 별도 처리).
   ```

3. **STRUCTURE.md Canonical Owner 매핑 갱신**:

   ```markdown
   | UI 시각 디자인 | `docs/20-system/DESIGN.md` |
   | API/CLI 인터페이스 컨벤션 | `docs/20-system/ARCHITECTURE_OVERVIEW.md` `## 7-1`, `## 7-2` |
   | 백엔드 핵심 결정 | `docs/20-system/ARCHITECTURE_OVERVIEW.md` `## 7-3` |
   | 프론트 핵심 결정 | `docs/20-system/ARCHITECTURE_OVERVIEW.md` `## 7-4` |
   ```

#### 검증

- [ ] ARCHITECTURE_OVERVIEW.md에 7-1, 7-2, 7-3, 7-4 sub-section 신설
- [ ] bootstrap-stack skill 본문 갱신
- [ ] STRUCTURE.md Canonical Owner 매핑 갱신

---

### Step 5.3 — `/bootstrap-design` skill 신설 (UI 한정, R0~R4 라운드)

**우선순위**: P0-Post-Gate
**Evidence**: [외부실증]
**의존성**: Step 5.1, 5.2

#### 현재 상태

`/bootstrap-project` / `/bootstrap-stack`만 존재. UI 시각 결정을 발굴하는 skill 부재 → 첫 컴포넌트 구현 시점에 LLM이 즉흥 결정 (purple gradient SaaS 회귀 위험).

#### 작업 절차

1. **`.claude/skills/bootstrap-design/SKILL.md` 신설** — 본문 핵심:

   ```markdown
   ---
   name: bootstrap-design
   description: UI 시각 결정 발굴 라운드 (R0~R4). DESIGN.md 채움.
   disable-model-invocation: true
   allowed-tools: Read Glob Grep Write Edit Agent
   ---

   # /bootstrap-design

   > 모드: How-to (UI 시각 결정 라운드)
   > 패턴: `discover-product` 차용 — `context: fork`를 명시하지 않아 메인 세션이 R0~R4를 직접 운전한다. R0(레퍼런스 분해)과 R1(원칙 추출)의 무거운 추론은 `Agent` 도구로 architect-opus를 단발 sub-call로 위임. 종료 후 사용자가 `/clear` 권장 (R0~R4 인터랙션이 다음 task 컨텍스트에 잡음).

   ## 트리거
   - `/bootstrap-stack` 종료 출력에 "frontend 감지됨. `/bootstrap-design` 권장" 텍스트 한 줄. 사용자 발화로 시작.
   - 비-UI 프로젝트는 호출되지 않음.

   ## 모드
   - `--fast`: R0(레퍼런스 1개) + R1(원칙 1줄로 압축) + R2(토큰). R3·R4 생략. R1은 *완전 생략 금지* — R2 토큰 결정의 근거가 되므로 *minimal 1줄*(예: "monochrome + 1 accent")이라도 채운다.
   - 기본: R0~R4 모두.

   ## 반드시 먼저 읽을 파일
   - `docs/10-charter/PROJECT_CHARTER.md` (페르소나·시나리오)
   - `docs/20-system/ARCHITECTURE_OVERVIEW.md` (스택)
   - `docs/20-system/DESIGN.md` (현재 placeholder)

   ## R0 — 레퍼런스 추출 + 안티-레퍼런스
   - 좋아하는 제품 1~3개 (예: Linear / Notion / Stripe / Vercel / Arc / Things)의 시각 메커니즘 분해.
     - color signature
     - typography pairing
     - density
     - motion 톤
   - **안티-레퍼런스 1~2개 필수**: "purple gradient generic SaaS 같지 말 것", "indigo-on-slate Tailwind 디폴트 회피".
   - architect-opus 단발 sub-call로 분해 가능.

   ## R1 — 디자인 원칙 3~5개
   - actionable verb. 모호어("modern/clean/sleek") 금지.
   - 예: "정보 밀도 우선", "monochrome + 1 accent", "motion은 의미 전달용만".

   ## R2 — 디자인 토큰 (W3C DTCG + Stitch 정렬)
   - 3-tier 토큰: primitive → semantic → component.
   - color: brand 1 + neutral 1 + accent 1 + semantic 4 (success/warning/error/info), 12~16 hex.
   - typography: 1~2 family, 4~5 size scale, modular ratio, weight pair.
   - spacing: 4 or 8 base, t-shirt scale 또는 numeric.
   - radius / shadow / motion (duration·easing·`prefers-reduced-motion`).
   - WCAG 4.5:1 텍스트 대비 검증.

   ## R3 — 컴포넌트 인벤토리 + 상태 매트릭스
   - primitives, composites, patterns.
   - 각 컴포넌트마다 상태 매트릭스 강제: default / hover / active / focus / disabled / loading / error / empty.
   - 스택별 시작점:

     | 스택 | 시작점 |
     | --- | --- |
     | React/Next.js | shadcn/ui (Radix + CSS 변수 + March 2026 shadcn-skills로 에이전트 컨텍스트 자동 주입) |
     | Vue | shadcn-vue |
     | Svelte | shadcn-svelte |
     | Astro | shadcn 패턴 + Astro 어댑터 |
     | RN/Expo *(ADR-031 override 시)* | Tamagui |
     | Flutter *(ADR-031 override 시)* | ShadCN-Flutter 또는 Material 3 |
     | SwiftUI *(ADR-031 override 시)* | Apple HIG 토큰 직접 정의 |

     본 표의 *기본 자동화 직접 지원 스택*은 React/Vue/Svelte/Astro (web frontend). RN·Flutter·SwiftUI는 *기본 자동화 범위 밖*(ADR-031)이며, fork 사용자가 override 발화 시 본 시작점을 참조 가능.

   ## R4 — `docs/20-system/DESIGN.md` 저장
   - 섹션 순서를 Stitch DESIGN.md canonical에 정렬: Overview / Colors / Typography / Layout / Elevation & Depth / Shapes / Components / Motion / Do's and Don'ts.
   - 토큰: frontmatter YAML 또는 fenced ```yaml``` 블록.

   ## 종료 후
   - 사용자가 `/clear` 권장. R0~R4가 인터랙션 길어지면 다음 task의 컨텍스트에 잡음.
   ```

2. **STRUCTURE.md 산출물 표 갱신** — `bootstrap-design` 행 추가.

3. **`/bootstrap-stack` 종료 출력 갱신** — UI 스택 감지 시 텍스트 1줄 권장 출력.

#### 검증

- [ ] `.claude/skills/bootstrap-design/SKILL.md` 존재
- [ ] STRUCTURE.md 산출물 표에 행 추가
- [ ] bootstrap-stack 종료 출력에 권장 텍스트 추가됨

---

### Step 5.4 — agent self-check 보강 (UI/API/CLI Don'ts 점검)

**우선순위**: P0-Post-Gate
**Evidence**: [외부실증]
**의존성**: Step 5.1, 5.2

#### 작업 절차

1. **builder-sonnet 갱신** — 다음 1줄 추가:

   ```markdown
   - 이번 task의 인터페이스 요소(컴포넌트/엔드포인트/명령어/스택 결정)가 해당 SSOT(DESIGN.md / ARCHITECTURE 7-1 API / 7-2 CLI / 7-3 백엔드 / 7-4 프론트)의 토큰·컨벤션·Don'ts를 위반하지 않는가?
   ```

2. **validator-sonnet 갱신** — 다음 1줄 추가:

   ```markdown
   - UI: 컴포넌트가 R3 인벤토리 등록 + 상태 매트릭스 충족? / API: 7-1 envelope·error 컨벤션 준수? / CLI: 7-2 출력 포맷 컨벤션 준수? / 백엔드: 7-3 DB migration·인증·트랜잭션 결정 정합? / 프론트: 7-4 라우팅·상태관리·SSR-CSR 결정 정합?
   ```

#### 검증

- [ ] builder-sonnet self-check 1줄 추가
- [ ] validator-sonnet 점검 1줄 추가

---

### Step 5.5 — ADR-027 본문 작성

**우선순위**: P0-Post-Gate
**의존성**: Step 5.1~5.4 모두 완료

#### 작업 절차

1. **ADR-027 본문 작성** — `docs/90-decisions/ADR-027-interface-decision-allocation.md`. 본문 핵심:
   - **Status**: accepted, scope=boilerplate
   - **배경**: AI 평균 미감(purple gradient SaaS) 회귀 + DESIGN_SYSTEM 광의 SSOT의 misnomer + ARCHITECTURE 운영성 중복
   - **결정 15개**:
     1. `DESIGN_SYSTEM.md` → `DESIGN.md` rename + UI 한정.
     2. ARCHITECTURE 7-1(API), 7-2(CLI), 7-3(백엔드), 7-4(프론트) sub-section 신설.
     3. `/bootstrap-design` skill 신설 (UI 한정).
     4. `/bootstrap-stack`이 7-1/7-2/7-3/7-4 채움 책임.
     5. UI DESIGN.md는 Stitch DESIGN.md canonical 섹션 순서 채택.
     6. 3-tier DTCG 토큰 모델.
     7. 영역별 Don'ts 섹션 필수.
     8. **repo root `DESIGN.md` 두지 않음** (외부 도구 자동 발견 마찰은 사용자가 root stub으로 ad-hoc 해결).
     9. **운영성 ↔ 7-1 경계 가이드**: trace ID/log 포맷/관측 stack=운영성. 응답 envelope/error 레지스트리/네이밍=7-1. 흐릿한 영역(예: error 응답에 trace ID)은 *주된 결정 맥락*으로 판단 — error 일관성 핵심이면 7-1, 관측 인프라 핵심이면 운영성.
     10. **charter 비목표 가이드**: "미적 트렌드 추종(neumorphism, glassmorphism, 그날의 dribbble)"을 비목표로 박을 것 권장.
     11. **API/CLI에 R0~R4 같은 라운드 도입 안 함** (YAGNI, ADR-006). API/CLI는 스택·도메인 강결합이라 `/bootstrap-stack`의 architect-opus 단발 sub-call로 충분. ADR-017 시뮬레이션이 가치 입증 시 후속 ADR로 `/bootstrap-stack --kind=api|cli` 모드 분기 가능.
     12. **shadcn/ui 권장이지 강제 아님** — shadcn은 ownership 모델(코드 복사)이라 lock-in 약함. `/bootstrap-design` skill의 R3 스택별 시작점 테이블 7개(React/Vue/Svelte/RN-Expo/Flutter/SwiftUI/Astro)가 대안 커버.
     13. **`/bootstrap-design --fast` 도입** — R0(레퍼런스 1개) + R2(토큰)만 → R3·R4 생략. discover-product의 단계별 출구 보장 패턴.
     14. **fork 격리 풀고 메인 세션이 R0~R4 운전** (discover-product 패턴) — `context: fork` 미명시. R0/R1 무거운 추론은 architect-opus 단발 sub-call(`Agent` 도구). 종료 후 사용자가 `/clear` 권장.
     15. **ARCHITECTURE 11→14 섹션 비대화 수용** — 7-1~7-4는 모두 "있을 때만 채움" 가이드라 비-API/비-CLI/비-백엔드/비-프론트 프로젝트는 통째 삭제. 분리 유지(B 옵션)의 misnomer·운영성 중복 비용보다 net win.
   - **비결정 No**: DESIGN_SYSTEM 광의 SSOT 유지 / 영역별 3개 파일 분리 / UI까지 ARCHITECTURE 흡수. 모두 단순성 1순위 위반으로 cut.
   - **마이그레이션**: 9 surface 변경 (DESIGN.md rename, ARCHITECTURE 4 sub-section, STRUCTURE 산출물·Canonical Owner, bootstrap-project output checklist, AGENTS.md 인덱스 1줄, /bootstrap-design 신설, /bootstrap-stack 본문 갱신, agent self-check 2개)
   - **시나리오 검증**: 본 가이드 부록의 시나리오 검증 표를 ADR 본문에 박는다 (Next.js SaaS / FastAPI 백엔드 / Rust CLI / 풀스택 / RN+Expo / 라이브러리 / UI+root stub / `--fast` prototype 8 시나리오).
   - **외부 근거** (결정 권위 추적용):
     - [Stitch DESIGN.md spec (Google Labs, 2026-04 오픈소스화)](https://github.com/google-labs-code/design.md/blob/main/docs/spec.md) — canonical 섹션 순서 채택 근거 (결정 5).
     - [W3C DTCG 2025.10 stable](https://www.w3.org/community/design-tokens/2025/10/28/design-tokens-specification-reaches-first-stable-version/) — 3-tier 토큰 모델 표준 (결정 6).
     - [designproject.io — How to write a design.md AI agents follow](https://designproject.io/blog/design-md-file/) — Don'ts가 LLM 정확도 향상의 *단일 최대 기여* (결정 7).
     - [Brad Frost — Agentic Design Systems in 2026](https://bradfrost.com/blog/post/agentic-design-systems-in-2026/) — *deliberate constraint* 안에서 AI 출력 품질 상한 (결정 1·11 근거).
     - [Material 3 — Motion easing/duration](https://m3.material.io/styles/motion/easing-and-duration) — 라우팅 UI 160~240ms / entrance·exit 240~360ms / `prefers-reduced-motion` (DESIGN.md `## 8. Motion` 섹션 근거).
     - [prg.sh — Why Your AI Keeps Building the Same Purple Gradient](https://prg.sh/ramblings/Why-Your-AI-Keeps-Building-the-Same-Purple-Gradient-Website) — LLM median 회귀 진단 (배경).
     - [VoltAgent/awesome-design-md](https://github.com/VoltAgent/awesome-design-md) (70+) / [getdesign.md](https://getdesign.md/) (423+) — R0 레퍼런스·안티-레퍼런스 입력 라이브러리.
     - [Smashing — Naming best practices for tokens](https://www.smashingmagazine.com/2024/05/naming-best-practices/) — 3-tier kebab-case 컨벤션.
     - [shadcn/ui March 2026 update — CLI v4 + AI Agent Skills + Design System Presets](https://dev.to/codedthemes/shadcnui-march-2026-update-cli-v4-ai-agent-skills-and-design-system-presets-1gp1) — 결정 12의 "AI-ready" 근거.

2. **ADR README 갱신**.

#### 검증

- [ ] ADR-027 본문 + README
- [ ] Phase 5 전체 commit 묶음 (또는 Step별 분할 commit) 완료

#### Commit message (Phase 5 통합)

```
docs: add ADR-027 reorganizing interface decisions across DESIGN/ARCHITECTURE
```

---

## Part 6. Phase 6 — Milestone graduation contract

> 본 Phase는 *stabilize-milestone*의 P0 결정을 박는다. milestone-level *sprint contract*가 핵심. 3 Step으로 단순화 — graduation checklist + 회고 / pre-check + `--dry-run` / ADR-014·017 합본.

### Step 6.1 — MILESTONE_TEMPLATE graduation checklist + 회고 (ADR-014 1단계)

**우선순위**: P0-Post-Gate
**Evidence**: [관측됨] (현재 placeholder 자체가 증거)
**의존성**: Phase 3 완료

#### 현재 상태

`docs/30-workitems/_templates/MILESTONE_TEMPLATE.md`의 `## 5. 완료 기준`은 빈 placeholder. graduation contract 부재 → milestone 졸업 판정 모호.

#### 작업 절차

1. **MILESTONE_TEMPLATE.md 갱신** — `## 5. 완료 기준` 본문을 graduation checklist로 교체:

   ```markdown
   ## 5. 완료 기준 (graduation checklist)
   > sprint contract: 본 마일스톤이 "done"이라고 합의되는 외부 검증 가능한 기준.
   - [ ] 모든 task status: done
   - [ ] 통합 validate Pass
   - [ ] E2E Pass (스택에 정의된 경우)
   - [ ] AC 매핑 100% (validation report 기준)
   - [ ] P0 severity finding 0건 (QA_FINDINGS의 본 마일스톤 헤더 기준)
   - [ ] (선택) 본 마일스톤 한정 추가 기준
   ```

2. **`## 8. 회고` 신설** — `## 7. 열린 질문` 아래에 추가:

   ```markdown
   ## 8. 회고 (stabilize 자동 채움)
   - 목표 달성도: <정량/정성 1줄>
   - scope creep 사례: <있으면 1줄, 없으면 "없음">
   - 비목표(charter ## 5) 위반 사례: <있으면 1줄>
   - 핵심 학습 3개 이내
   ```

3. **`/plan-workitem` skill 갱신** — milestone 생성 시점에 위 default를 자동 채움. 사용자가 추가 기준을 협상.

   ```markdown
   ## milestone 생성 시 default
   - `## 5. 완료 기준`은 ADR-014 graduation checklist 5+1 항목 default 사용.
   - `## 8. 회고`는 stabilize가 자동 채움 — plan 단계에서는 비워둠.
   ```

#### 검증

- [ ] MILESTONE_TEMPLATE.md `## 5` placeholder가 5+1 checklist로 교체
- [ ] MILESTONE_TEMPLATE.md `## 8. 회고` 신설
- [ ] plan-workitem skill default 단락 추가

#### Commit message

```
docs: add milestone graduation checklist and retrospective sections
```

---

### Step 6.2 — `/stabilize-milestone` graduation pre-check + `--dry-run` (ADR-014 2단계)

**우선순위**: P0-Post-Gate
**Evidence**: [관측됨]
**의존성**: Step 6.1

#### 작업 절차

1. **`/stabilize-milestone` skill 갱신** — 단계 1과 2 사이에 graduation pre-check 단계 신설:

   ```markdown
   ### 1.5. Graduation pre-check
   - MILESTONE의 `## 5. 완료 기준` 각 항목 자동 체크.
   - 미충족 항목 발견 시 `졸업 가능: NO, <미충족 항목>` 출력 후 *조기 종료 옵션*.
   - `--dry-run` 플래그가 켜져 있으면 pre-check만 돌리고 종료(P0 검증 도구).
   ```

2. **`--dry-run` 플래그 명시** — skill frontmatter 또는 본문에 옵션 명시.

#### 검증

- [ ] stabilize-milestone skill 단계 1.5 추가
- [ ] `--dry-run` 플래그 명시

#### Commit message

```
docs: add stabilize graduation pre-check step with --dry-run flag
```

---

### Step 6.3 — ADR-014 본문 작성 + ADR-017 시뮬레이션 dogfood ADR화

**우선순위**: P0-Post-Gate
**의존성**: Step 6.1·6.2 완료 + Phase 1 (baseline 시뮬레이션 1회 실행 완료). ADR-014/017이 Phase 6의 모든 결정을 한 ADR로 묶어 박는 *합본 시점*.

#### 작업 절차

1. **ADR-014 본문 작성** — `docs/90-decisions/ADR-014-milestone-graduation.md`. 본문 핵심:
   - scope=boilerplate
   - 결정 3종: graduation checklist 5+1 / 회고 4 항목 / pre-check 단계 + `--dry-run`
   - 비결정 No: Release-level DoD (stabilize 출력에 자연 흡수 — carry-over 0건 + ADR 후보 0건 = release-ready)
   - 기타 *milestone graduation 관련 직접 cut된 결정들*(Fowler 4-quadrant / METRICS.md / `--apply-carryover` / architect-opus auto-escalation 신호)은 IMPROVE-GUIDE Part 11 영구 No 표에 사유 명시 — 본 ADR이 직접 부정하지 않고 상위 결정으로 위임.
   - 근거: Anthropic 3-agent harness sprint contract + Atlassian multi-level DoD (story/sprint/release)
   - 잔여 모니터링: graduation pre-check 미통과 사유 패턴 (3회 이상 반복 시 lifecycle 단계 결함 신호)

2. **ADR-017 본문 작성** — `docs/90-decisions/ADR-017-dogfood-simulation.md`. 본문 핵심:
   - scope=boilerplate
   - 결정: 시뮬레이션 dogfood 1회 의무 + 재실행 트리거 3종 (새 ADR / lifecycle 단계 변경 / skill 본문 큰 변경)
   - 시나리오: todo CLI (CRUD + persistence) — calculator는 stateless라 lifecycle 자극 불충분
   - 성공 기준 3개: 사용자 개입 ≤ 1회 / 충원율 ≥ 80% / pre-check 미통과 사유 ≤ 2개
   - 산출물: `docs/40-validation/SIMULATION_RUN.md` (Record, 회차별 누적)
   - 실패 처리: 발견된 깨짐을 ADR 후보로 박고, 결합 P0 재평가

3. **STRUCTURE.md 산출물 표 갱신** — SIMULATION_RUN.md 행 추가 (Record).

4. **ADR README 갱신** — ADR-014, 017 행 추가.

#### 검증

- [ ] ADR-014 본문 + README
- [ ] ADR-017 본문 + README
- [ ] STRUCTURE.md 산출물 표 갱신 (SIMULATION_RUN.md)

#### Commit message (두 ADR을 한 commit으로 묶음 권장)

```
docs: add ADR-014 graduation contract and ADR-017 dogfood simulation
```

---

## Part 7. Phase 7 — 4 Pillars 정합 (하네스 엔지니어링)

> 본 Phase는 *하네스 엔지니어링* P0 결정을 박는다. Anthropic 4 Pillars(system prompt / tools / context / subagents)에 본 보일러플레이트 자산을 매핑할 때 **Context pillar가 가장 약하다** — `Refs:` footer 컨벤션 / JIT 로딩 / Context Packs가 P0 핵심. CODE_LINEAGE.md 자동 생성은 P1 트리거 보류(Step 7.1 참조).
>
> **Anthropic 3-agent vs 본 보일러플레이트 6-agent — 정직한 정당화**: Anthropic 3-agent 패턴(planner / generator / evaluator)을 본 보일러플레이트로 매핑하면 *evaluator만 3-way 분리*(validator-sonnet / qa / reviewer). 분리 정당성은 *증거 베이스 차이*: validator=AC 텍스트+테스트 함수 이름 / qa=코드 실행 결과(Playwright 등) / reviewer=PR diff 패턴. 통합 가능성은 *데이터 트리거*(Step 10.7) — 시뮬레이션이 출력 중복률 ≥30% 보고 시 통합 ADR 검토. 추측이 아니라 데이터 기반.

### Step 7.1 — Conventional Commits `Refs:` footer 컨벤션 (ADR-008 amend 2차) + CODE_LINEAGE.md (P1 트리거 보류, ADR-018 신설 *deferred*)

**우선순위**: ADR-008 amend 2차 = **P0-Post-Gate** / CODE_LINEAGE.md 자동 생성 = **P1 (트리거 보류)**
**Evidence**: [외부실증] (Conventional Commits footer) + [가설→트리거] (CODE_LINEAGE.md 자동 생성)
**의존성**: Phase 4.4 (ADR-008 amend 1차 — monorepo scope)

#### 현재 상태

- 코드와 task ID의 lineage 추적 자리 부재.
- task `## 4-1. 변경 예정 파일/경로`는 본문 명시상 *drift 가능*("git 실제 변경과 어긋나면 finalize는 차이를 출력에 명시").
- `/finalize-workitem`이 명시적 file add를 강제하므로 git이 1차 SSOT.
- ADR-008은 Conventional Commits를 박았지만 *footer 컨벤션* 부재.

#### 변경 후 상태 — 2단계 도입

**P0 (즉시 박음)** — ADR-008 amend 2차로 `Refs:` footer 컨벤션. 사용자가 `git log --grep="Refs: T-XXX"`로 ad-hoc lineage 추적 가능.

**P1 (트리거 보류)** — `docs/40-validation/CODE_LINEAGE.md` 자동 생성은 *footer 누적 + sticky question 발생 시* 도입. 이유:
- 자동 생성 도구 작성 비용 + 첫 회 정확도 검증 필요.
- *git history와 중복*되는 view라 SSOT 분산 우려.
- 갱신 실패 시 오히려 신뢰도 하락 위험.

**트리거 조건** (3개 중 1개):
1. 첫 마일스톤 stabilize 후 *"이 파일 왜 있지?"* sticky question 1회+ 발생.
2. fork 사용자가 CODE_LINEAGE.md를 요청.
3. 6개월 footer 누적 후 *git log grep 만으로 부족*하다는 [관측됨] 데이터 회수.

#### 작업 절차 (P0 부분만)

1. **ADR-008 amend 2차** — `docs/90-decisions/ADR-008-commit-convention.md` 본문 끝에 Amendment 2 단락 추가:

   ```markdown
   ## Amendment 2 (2026-05-XX) — `Refs:` footer 컨벤션

   ### 결정 (컨벤션 — 단정형)
   commit 메시지 footer는 `Refs:` 라인 1개를 포함한다 (CODE_LINEAGE.md 트리거 도달 시 derived view의 SSOT):

   ```
   feat(auth): implement /me endpoint

   Refs: T-003 (AC-2, AC-3)
   ```

   - `Refs:` 값은 `T-NNN (AC-X, AC-Y)` 형식.
   - 다중 task 묶음 commit은 `Refs: T-001, T-002` 형식.
   - lock file 화이트리스트 commit(ADR-007 amend 1)은 `Refs: chore` 또는 생략 가능.

   ### 실행 (skill — 권장형)
   `/finalize-workitem` skill은 footer 누락 발견 시 *footer 추가 권장 텍스트* 출력. 자동 차단은 하지 않음 (사용자 결정 — ADR-007 책임 경계 정합).

   ### 근거
   - CODE_LINEAGE.md가 git log + footer 기반 derived view → footer가 SSOT.
   ```

2. **`/finalize-workitem` skill 갱신** — commit 메시지 footer 검증 1줄 추가:

   ```markdown
   - 모든 commit 메시지에 `Refs: T-NNN (AC-X, AC-Y)` 형식 footer 존재 확인. 누락 시 *footer 추가 권장 텍스트* 출력 (자동 차단 X, 사용자 결정).
   ```

#### 작업 절차 (P1 트리거 발동 시 활성화)

3. **ADR-018 본문 작성 (deferred)** — `docs/90-decisions/ADR-018-code-lineage.md`. 본 ADR은 *트리거 발동 후 박음* — 본 가이드는 트리거 도달 전까지 ADR-018 미작성. 본문 예정 내용:
   - scope=boilerplate
   - 결정 5종 (트리거 도달 시):
     1. 소스 = git log + Conventional Commits footer (task `## 4-1` *아님*)
     2. 형식 = 4 필드 표 (path / first / last / touches / active-AC)
     3. regeneration = stabilize가 매 회 *전량 재생성* (idempotent)
     4. deleted file = 행 보존 + last 필드에 `(deleted in M3, last T-021)` 토큰
     5. git 추적함 (.gitignore *아님*). 본문 첫 줄에 `<!-- GENERATED by /stabilize-milestone -->` 마커
   - 비결정 No (트리거 도달 전 보류 이유): git history와 *view 중복*이라 SSOT 분산 우려 / 자동 생성 도구 갱신 실패 시 신뢰도 하락 / footer 만으로 ad-hoc 추적 충분 (사용자 [관측됨] 데이터 부재)

4. **Phase 10 트리거 대기 표에 행 추가** — Phase 10.9 (CODE_LINEAGE.md 자동 생성)을 트리거 매트릭스에 등록.

#### 검증 (P0 부분만)

- [ ] ADR-008 본문 Amendment 2 단락 (`Refs:` footer 컨벤션)
- [ ] `/finalize-workitem` skill footer 검증 권장 1줄 추가
- [ ] Phase 10 트리거 매트릭스에 CODE_LINEAGE.md 행 등록 (트리거 발동 전 대기 상태)

#### Commit message

```
docs: amend ADR-008 with Refs footer convention (CODE_LINEAGE.md deferred to P1 trigger)
```

#### 결정 근거 (왜 자동 생성 보류인가)

- footer 컨벤션 1줄만으로 *git log --grep="Refs: T-XXX"* ad-hoc lineage 추적 가능 — 즉시 효용 90% 확보.
- CODE_LINEAGE.md 자동 생성은 *나머지 10% 효용* (이 파일 왜 있지? 같은 sticky question에 즉시 답) — 도구 작성·정확도 검증 비용 대비 ROI 의문.
- ADR-022 Ratchet 약 적용 — [가설]만 있는 정책은 *enabling* 형태로 도입. CODE_LINEAGE.md는 *제약 정책* 형태(stabilize가 매 회 재생성 의무)였으나 Ratchet 강 적용 영역 → 트리거 보류.
- 트리거 발동 시 본 가이드 Step 7.1을 *활성화*해 작업 절차 3·4를 실행.

---

### Step 7.2 — JIT 로딩 + Context Packs frontmatter (ADR-019 신설)

**우선순위**: P0-Post-Gate
**Evidence**: [외부실증] (Anthropic effective context engineering)
**의존성**: Phase 3 완료

#### 현재 상태

각 skill SKILL.md에 *반드시 먼저 읽을 파일* 목록은 있지만 *최소 충분*인지 점검 없음. 매 task마다 모든 ADR fork-load하는 패턴 위험.

#### 변경 후 상태

- JIT 로딩 정책 명문화: *반드시 먼저 읽을 파일*은 *최소 충분*. 추가 ADR/architecture 섹션은 task 본문에서 발화 시 인용 — 사전 fork-load 금지.
- skill frontmatter `context-pack` 필드: `minimal` (default, 일반 skill) / `full` (architect-opus 전용).
- frontend·backend 같은 *영역별 pack은 사전 정의하지 않음* — 사용자가 필요 시 fork 프로젝트에서 자체 정의 (과설계 회피).

#### 작업 절차

1. **모든 skill SKILL.md 본문에 1줄 추가**:

   ```markdown
   ## Context 정책 (ADR-019)
   `반드시 먼저 읽을 파일`은 *최소 충분*. 추가 ADR/architecture 섹션은 task 본문에서 발화 시 인용 — 사전 fork-load 금지.
   ```

2. **각 skill frontmatter에 `context-pack` 필드 추가**:

   ```yaml
   ---
   name: ...
   description: ...
   context-pack: minimal  # default. architect-opus만 full.
   ---
   ```

   pack 정의:

   | pack | 포함 | 용도 |
   |------|------|------|
   | **minimal** *(default)* | AGENTS.md + task 본문 | 모든 일반 skill |
   | full | 모든 docs/ | architect-opus 디폴트 |

3. **`architect-opus` agent 갱신** — frontmatter `context-pack: full`.

4. **ADR-019 본문 작성** — `docs/90-decisions/ADR-019-context-packs-and-jit.md`. 본문 핵심:
   - scope=boilerplate
   - 결정 2종: JIT 로딩 정책 명문화 + Context Packs 2종(minimal / full)
   - 비결정 No: frontend/backend 영역별 pack 사전 정의 (과설계 — 사용자가 필요 시 fork에서 자체 정의)
   - 토큰 절감 추정: minimal ~5K vs full ~30K (호출당 5~25K 절감)

5. **ADR README 갱신**.

#### 검증

- [ ] 모든 skill SKILL.md에 Context 정책 1줄
- [ ] 모든 skill frontmatter `context-pack: minimal`
- [ ] architect-opus `context-pack: full`
- [ ] ADR-019 본문 + README

#### Commit message

```
docs: add ADR-019 with JIT context loading and context-pack frontmatter
```

---

### Step 7.3 — `/stack-guard` 정적 분석 권장 + secret scanner (ADR-021 신설 + amend 1)

**우선순위**: P0-Post-Gate
**Evidence**: [외부실증] (dependency-cruiser / import-linter / gitleaks)
**의존성**: Phase 3 완료

#### 현재 상태

`/stack-guard`가 정적 분석 도구를 권장하지 않음. layer 경계 위반·secret hardcode 검출 자리 없음.

#### 변경 후 상태

- 스택별 *1종* 정적 분석 권장 (paralysis 차단 — 대안 나열 X).
- secret scanner (gitleaks/trufflehog) 권장.
- *강제 X, 권장만* (ADR-010 multi-tool 호환).
- 영역별 lint(UI=design tokens / API=Pact·Schemathesis / CLI=snapshot test)는 *과설계로 cut* — Step 5.4의 builder/validator self-check가 동등 효용을 더 가벼운 비용으로 확보 (Pact/Schemathesis는 학습 곡선 큼).

#### 작업 절차

1. **`/stack-guard` skill 갱신** — 본문에 다음 표 추가:

   ```markdown
   ## 정적 분석 도구 권장 (스택별 1종)

   | 스택 | 도구 | 비고 |
   |------|------|------|
   | TypeScript / JS | `dependency-cruiser` | layer 위반 룰을 ARCHITECTURE_OVERVIEW `## 3-1` 채움 시 함께 권장. madge·skott은 사용자 결정 옵션. |
   | Python | `import-linter` | 동일 layer 룰 패턴 |
   | Go | `go vet` (built-in) | 후속 보강 가능 |
   | Rust | `cargo deny` + `cargo udeps` | unused deps + license/advisory 동시 점검 |

   ## Secret scanner 권장 (전 스택)
   - `gitleaks` 또는 `trufflehog`. 둘 중 1종 선택.
   - finalize 직전 staged 파일에 secret 패턴 검출 시 차단.
   - *강제 X, 권장만*.

   `validate` 명령에 lint 단계로 통합 → CI fail로 잡힘.
   ```

2. **ADR-021 본문 작성** — `docs/90-decisions/ADR-021-static-analysis-recommendation.md`. 본문 핵심:
   - scope=boilerplate
   - 결정: 스택별 1종 + secret scanner. *강제 X, 권장만*.
   - paralysis 방지: 대안 나열 X.
   - GUARDRAILS_STRATEGY *"OS/셸 종속 hook 강제 X"* 정신 정합.

3. **ADR-021 amend Amendment 1 (secret scanner)** — 본문 끝에 단락 추가:

   ```markdown
   ## Amendment 1 (2026-05-XX) — secret scanner 추가
   - `gitleaks` / `trufflehog` 1종 권장 (전 스택).
   - finalize 직전 staged 파일 점검.
   - 근거: secret hardcode 검출 부재 빈자리 보강.
   ```

4. **ADR README 갱신**.

#### 검증

- [ ] stack-guard skill에 정적 분석 표 + secret scanner 단락
- [ ] ADR-021 본문 + Amendment 1 + README

#### Commit message

```
docs: add ADR-021 with stack-specific static analysis and secret scanner
```

---

### Step 7.4 — 코드 organization 가이드 (skill amend)

**우선순위**: P0-Post-Gate
**Evidence**: [외부실증]
**의존성**: Phase 5 (ARCHITECTURE 7-1~7-4)

#### 작업 절차

1. **`/bootstrap-stack` skill 갱신** — 다음 표 추가:

   ```markdown
   ## 스택별 디폴트 디렉터리 구조 (권장 출력)

   | 스택 | 디폴트 트리 |
   |------|-----------|
   | Next.js | `app/`, `components/`, `lib/`, `tests/` |
   | FastAPI | `app/{api,core,domain,infra}/`, `tests/` |
   | Express | `src/{routes,services,domain,infra}/`, `tests/` |
   | Rust CLI | `src/{cli,core,...}/`, `tests/` |
   | Go CLI | `cmd/`, `internal/{cli,core,...}/`, `tests/` |
   | Python CLI | `src/<pkg>/{cli,core,...}/`, `tests/` |

   ARCHITECTURE_OVERVIEW.md `## 3-1` 채움 시 함께 박음.
   사용자 즉흥 결정 → 스파게티 차단.
   ```

#### 검증

- [ ] bootstrap-stack skill 디폴트 디렉터리 표 추가

#### Commit message

```
docs: add stack-specific default directory layout to bootstrap-stack
```

---

### Step 7.5 — sub-agent 출력 cap 1~2K 명문화 (agent amend)

**우선순위**: P1
**Evidence**: [외부실증] (Anthropic 가이드)
**의존성**: Phase 3 완료

#### 작업 절차

1. **각 agent 본문에 1줄 추가** — `.claude/agents/*.md` 6개 모두:

   ```markdown
   ## 출력 cap
   반환 요약은 1,000~2,000 토큰. 긴 reasoning은 본 sub-agent 안에 둔다(메인 컨텍스트 토큰 경합 방지 — Anthropic 가이드).
   ```

#### 검증

- [ ] 6개 agent 모두 출력 cap 단락 추가

#### Commit message

```
docs: add 1-2K output token cap to all sub-agents
```

---

## Part 8. Phase 8 — 기획 PO/PM 트랙 강화

> 본 Phase는 *기획(PO 트랙: discovery·PMF / PM 트랙: feature 분해·spec)* 의 P0/P1 결정을 박는다. *build it right*(품질·lifecycle·SSOT)는 깊지만 *build the right it*(PMF·iteration·AI-readable spec)가 얕은 빈자리를 메운다.
>
> **PO/PM 트랙 분리는 1인 개발자에게 과잉 아닌가** — *역할 분리가 아니라 트랙(시간) 분리*. Cagan dual-track Agile의 원래 정의도 *time-split for one person*. 본 Phase의 P0 두 항목(8.1·8.2)은 *문서 구조*만 바꾸므로 1인 비용 0.

### Step 8.1 — DISCOVERY.md 진짜 living doc + Assumption Tracker (ADR-035 신설)

**우선순위**: P0-Post-Gate
**Evidence**: [관측됨+외부실증] (Cagan/Torres dual-track + DISCOVERY.md "Living Doc" 분류했지만 절차 비어 있음)
**의존성**: Phase 3 완료

#### 현재 상태

`docs/10-charter/_templates/DISCOVERY_TEMPLATE.md`는 11섹션 placeholder. STRUCTURE.md는 "Living Doc"으로 분류했지만 *어떻게 살아 있는가* 정의 부재. mid-project pivot 시 재호출 절차 부재. DISCOVERY → Charter *1방향 박기*만 정의 → SSOT 모호.

#### 변경 후 상태

11섹션 → 13섹션. 신설 2개(`## 12 Assumption Tracker` / `## 13 Opportunity Backlog`). `/discover-product`에 `--update` 모드 추가. **DISCOVERY.md = persona/scenario/assumption SSOT, Charter는 snapshot view**(역방향 SSOT 아님)을 ADR-035에 명문화.

#### 작업 절차

1. **DISCOVERY_TEMPLATE.md 갱신** — `## 11. 열린 질문` 아래에 신설:

   ```markdown
   ## 12. Assumption Tracker
   <!-- ## 10 핵심 가정의 *검증 결과 누적*. 빈 결과 = "미검증 - 행동 차단", stabilize가 보고. -->
   | ID  | 가정                        | 검증 방법    | 검증 결과   | 검증일      | 다음 행동       |
   |-----|---------------------------|------------|----------|-----------|---------------|
   | A-1 | 타겟이 매주 이력서 갱신     | 5명 인터뷰   | true (4/5) | 2026-05-15  | F-002 진행      |
   | A-2 | 무료→$5 전환 ≥ 10%         | landing A/B  | false (3%) | 2026-06-01  | pivot 필요      |

   ## 13. Opportunity Backlog
   <!-- 기각·검증실패 후보까지 보존 (Torres OST opportunity space 정신). -->
   | Pain                       | 빈도×고통  | 현재 상태       | 비고                    |
   |---------------------------|----------|---------------|----------------------|
   | 매일 이력서 버전 관리       | 매일×중   | active (F-001) | 핵심                    |
   | 협업자 권한 관리           | 가끔×하   | parked         | M3 이후 재평가          |
   | AI 자동 작성               | -          | rejected       | A-2 검증 결과 false     |
   ```

2. **`/discover-product` skill 갱신** — `.claude/skills/discover-product/SKILL.md`에 다음 모드 추가:

   ```markdown
   ## --update 모드 (mid-project pivot)
   기존 DISCOVERY.md 있으면:
   - R0 (페르소나 재확인) → R1·R2 (opportunity backlog 갱신·새 pain 추가) → R3 (assumption tracker 갱신) → R4 저장.
   - **`--fast --update`**: assumption tracker만 갱신 (가장 빈번한 mid-project use case).

   ## Idempotency
   ID 매칭 — 기존 ID(A-1·A-2)면 *검증일·다음 행동만 갱신*, 새 가정이면 새 ID 부여.
   ```

3. **AGENTS.md "깊은 운영 원칙" 인덱스에 1줄 추가** (Phase 3.4 cap 적용 후라 본 1줄도 cap 안에 들어가는지 확인):

   ```markdown
   - **DISCOVERY=SSOT, Charter=snapshot** — DISCOVERY.md 갱신 시 Charter는 자동 sync 안 됨. 사용자가 `/bootstrap-project --apply`로 갱신 제안을 받거나 직접 편집. (ADR-035)
   ```

4. **PROJECT_CHARTER.md 본문 끝에 안내 추가**:

   ```markdown
   <!-- DISCOVERY.md 가 SSOT, 본 Charter는 snapshot view (ADR-035).
        DISCOVERY 갱신 시 본 Charter는 자동 sync 안 됨 — `/bootstrap-project --apply` 또는 수동 갱신. -->
   ```

5. **ADR-035 본문 작성** — `docs/90-decisions/ADR-035-continuous-discovery.md`. 본문 핵심:
   - scope=boilerplate
   - 결정 4종: `## 12 Assumption Tracker` 신설 / `## 13 Opportunity Backlog` 신설 / `--update` 모드 추가 / DISCOVERY=SSOT/Charter=snapshot 명문화
   - 근거: Cagan dual-track Agile + Torres continuous discovery + DISCOVERY.md "Living Doc" 분류 비어 있음
   - 잔여 모니터링: assumption tracker 빈 결과율 (stabilize 보고)

6. **ADR README 갱신**.

#### 검증

- [ ] DISCOVERY_TEMPLATE.md `## 12`, `## 13` 신설
- [ ] discover-product skill `--update` 모드 + Idempotency 단락
- [ ] AGENTS.md 인덱스 1줄 (cap 안)
- [ ] PROJECT_CHARTER.md 본문 끝 안내
- [ ] ADR-035 본문 + README

#### Commit message

```
docs: add ADR-035 with DISCOVERY.md as living doc and assumption tracker
```

---

### Step 8.2 — FEATURE_TEMPLATE PRD 강화 (ADR-036 신설)

**우선순위**: P0-Post-Gate
**Evidence**: [관측됨+외부실증] (Osmani 6 core + ChatPRD 6 sections)
**의존성**: Phase 4 (TASK_TEMPLATE schema)

#### 현재 상태

`docs/30-workitems/_templates/FEATURE_TEMPLATE.md`는 10섹션. User Story / 시나리오 / Feature-level AC / NFR 자리 부재. AI agent가 implement하기엔 *who·why·시나리오 측정 기준* 부족.

#### 변경 후 상태

10섹션 → 12섹션. **신설 3개**(`## 3 핵심 시나리오 (Feature-level)` / `## 7 Feature-level AC` / `## 8 NFR`) + **흡수 1개**(기존 `## 8 검증 방법` → `## 7 FAC`로 흡수, 측정 가능 형식화) + **강화 1개**(기존 `## 2 사용자 가치` → User Story 형식 강제).

#### 작업 절차

1. **FEATURE_TEMPLATE.md 갱신** — 본문 재구성:

   ```markdown
   # F-xxx-이름

   ## 0. Status
   draft

   ## 1. 요약

   ## 2. 사용자 가치 (User Story)
   <!-- "As a <persona>, I want to <goal>, so that <benefit>." 1개 이상.
        persona는 PROJECT_CHARTER.md `## 2.1` ID 인용 — 자체 발명 X. -->

   ## 3. 핵심 시나리오 (Feature-level)
   <!-- happy / alternate / fail 각 3~5단계.
        Charter `## 3.1`(제품 전체)과 다른 *이 feature 한정* 시나리오. -->

   ## 4. 범위

   ## 5. 비범위

   ## 6. 요구사항

   ## 7. Feature-level Acceptance Criteria
   <!-- FAC-1, FAC-2 ... 시나리오 수준 측정 가능 기준.
        task `## 6 AC`는 FAC를 만족시키는 구현 단위.
        구 `## 8 검증 방법`을 흡수. -->

   ## 8. Non-functional Requirements
   <!-- 성능·접근성·보안·i18n. 해당 없으면 "(해당 없음)" 명시. -->

   ## 9. 엣지 케이스

   ## 10. 의존성

   ## 11. 관련 문서
   - Milestone:
   - Charter:
   - Architecture:
   - ADR:

   ## 12. 열린 질문
   ```

2. **`/plan-workitem` skill 본문에 1줄 추가**:

   ```markdown
   feature 분해 시 12섹션 모두 채운다. `## 7 FAC`는 task `## 6 AC`로 분해되며 매핑 누락 시 plan 출력의 "남은 미결정 사항"에 명시.
   ```

3. **`--fast` 회피 보장** — plan-workitem skill에 추가:

   ```markdown
   ## --fast 모드
   prototype은 `## 3 핵심 시나리오` / `## 7 FAC` / `## 8 NFR` 신설 3섹션을 1줄씩만 채워도 OK ("해당 없음" / "M2 이후 검토").
   YAGNI 정합 — Phase 6의 graduation contract *시작 시점 budget*과 동등 정신.
   ```

4. **AGENTS.md "핵심 행동 규율"에 boundaries 3-tier 라벨링** — 기존 5~7개 규율에 Osmani 3-tier(✅Always/⚠️Ask/🚫Never) 라벨만 도입(추가 항목 0개, 100줄 cap 보호):

   ```markdown
   ## 핵심 행동 규율
   - ✅ 상위 문서 없이 하위 문서를 먼저 만들지 않는다.
   - 🚫 `.env`, `secrets/` 같은 민감 파일은 건드리지 않는다.
   - ✅ 작업 범위와 비범위를 명확히 적고, 범위 밖 변경은 하지 않는다.
   - ✅ 흩어진 임시 메모보다 정해진 위치의 문서를 갱신한다.
   - ⚠️ 사실, 가정, 열린 질문을 구분해서 적는다. 검증 가능한 표현을 우선한다.
   - ✅ 커밋은 작고 논리적인 단위로 나눈다. 커밋 전에 관련 workitem 문서와 구현 범위가 일치하는지 확인한다.
   ```

5. **ADR-036 본문 작성** — `docs/90-decisions/ADR-036-feature-level-prd.md`. 본문 핵심:
   - scope=boilerplate
   - 결정 3종: FEATURE_TEMPLATE 12섹션 / plan-workitem FAC↔AC 매핑 강제 / AGENTS.md boundaries 3-tier 기존 항목 라벨링
   - 근거: Osmani 6 core + ChatPRD 6 sections + spec-kit/Kiro/ChatPRD가 *feature 단위 user story+AC를 spec 단계에서* 박는데 본 보일러플레이트는 task 단계만
   - `--fast` 회피 보장: 신설 3섹션 1줄씩만 OK
   - 비결정 No 1: 자체 발명 PRD 양식 (외부 표준 학습 비용 0, ADR-005 SSOT 위반)
   - 비결정 No 2: spec-kit `constitution.md` 별도 파일 추가 (AGENTS.md + ADR-006 단순성 정책이 동등 매핑 → ADR-005 진입 페이지 1개 정책 + ADR-010 IDE lock-in 회피와 정합. spec-kit의 4-phase는 본 보일러플레이트의 8단계 lifecycle과 동치 매핑이며 별도 파일 추가 시 SSOT 분산)

6. **ADR README 갱신**.

#### 검증

- [ ] FEATURE_TEMPLATE.md 12섹션
- [ ] plan-workitem skill 1줄 + --fast 모드 단락
- [ ] AGENTS.md 핵심 행동 규율 ✅⚠️🚫 라벨링 (cap 안)
- [ ] ADR-036 본문 + README

#### Commit message

```
docs: add ADR-036 expanding FEATURE_TEMPLATE to 12-section PRD
```

---

### Step 8.3 — Spec coverage self-audit (ADR-037 신설)

**우선순위**: P1
**Evidence**: [외부실증] (Osmani self-audit)
**의존성**: Step 8.2 (FEATURE FAC가 있어야 self-audit 가능)

#### 작업 절차

1. **validator-sonnet 갱신** — 다음 1줄 추가:

   ```markdown
   - feature `## 7 FAC`의 각 항목이 task `## 6 AC`로 매핑됐는가? 매핑 안 된 FAC가 있으면 report에 `Spec Gap: FAC-N → unmapped` 명시 + 미커버 task 추가 권장 (자동 차단 X — ADR-007 책임 경계 정합).
   ```

2. **`/plan-workitem` skill 출력 형식 갱신** — 다음 섹션 추가:

   ```markdown
   ## 8. FAC ↔ AC 매핑표
   - 형식: `FAC-1 → T-001:AC-1, T-002:AC-2`.
   - 미커버 FAC는 `unmapped`.
   ```

3. **ADR-037 본문 작성** — `docs/90-decisions/ADR-037-spec-coverage-audit.md`. 본문 핵심:
   - scope=boilerplate
   - 결정: validator self-audit 1 step + plan-workitem 매핑표 출력
   - **자동 차단 X (제안만)** — ADR-007 validator 책임 경계 정합
   - 토큰 비용: ~1K/task (6개월 뒤 누락 발견 비용보다 작음)

4. **ADR README 갱신**.

#### 검증

- [ ] validator-sonnet self-audit 1줄 추가
- [ ] plan-workitem 출력 형식에 `## 8. FAC ↔ AC 매핑표` 추가
- [ ] ADR-037 본문 + README

#### Commit message

```
docs: add ADR-037 with FAC to AC coverage self-audit in validator
```

---

## Part 9. Phase 9 — P1 보강 (다음 마일스톤)

> 본 Phase의 항목은 *Phase 1~8이 끝난 다음 마일스톤* 시점에 박는다. P0 작업 직후 한 번에 묶지 않는 이유: P0 결과로 우선순위 재조정 가능, P1은 *enabling 정책* 다수라 Ratchet 약 적용.

### Step 9.1 — `validate --changed` (incremental, ADR-020 신설)

**우선순위**: P1
**Evidence**: [외부실증] (Nx affected / Turbo affected 패턴)
**의존성**: Phase 7.3 (정적 분석 통합)

#### 작업 절차

1. **`/stack-guard` skill 갱신** — `validate --changed` 옵션 추가 권장:

   ```markdown
   ## validate --changed (incremental)
   - git diff 기반 변경 파일만 lint/typecheck/test.
   - Nx affected / Turbo affected 패턴 차용.
   - **사용 시점**:
     - `/finalize-workitem` 직전 → `--changed`만 (빠른 회전).
     - `/stabilize-milestone` → full validate (누락 차단).
   ```

2. **`/finalize-workitem` skill 갱신** — finalize 직전에 `--changed` 호출 권장 1줄.

3. **ADR-020 본문 작성** — `docs/90-decisions/ADR-020-incremental-validate.md`. 본문 핵심:
   - scope=boilerplate
   - 결정: `validate --changed` (incremental)
   - finalize는 `--changed`만, stabilize는 full validate
   - 근거: 큰 codebase에서 full validate 비용 폭증 → AI가 검증 skip 위험

4. **ADR README 갱신**.

#### 검증

- [ ] stack-guard skill `validate --changed` 단락
- [ ] finalize-workitem skill 1줄
- [ ] ADR-020 본문 + README

#### Commit message

```
docs: add ADR-020 with validate --changed for incremental check
```

---

### Step 9.2 — Dependency hygiene (skill amend)

**우선순위**: P1
**Evidence**: [외부실증]
**의존성**: Phase 3 완료

#### 작업 절차

1. **`/stabilize-milestone` skill 갱신** — 매 회 1회 실행:

   ```markdown
   ## Dependency hygiene
   - `npm audit` / `pip-audit` (스택별 대응) 1회 실행.
   - 결과를 IMPROVEMENT_GUIDE.md에 P1 severity로 보고.
   - 6개월 unused deps는 P2로 자동 등록.
   ```

#### 검증

- [ ] stabilize-milestone skill dependency hygiene 단락

#### Commit message

```
docs: add stabilize dependency hygiene scan
```

---

### Step 9.3 — `/bootstrap-stack` 외부 의존 권장 + `/stack-guard` CI 권장 (ADR-025 신설)

**우선순위**: P1
**Evidence**: [외부실증]
**의존성**: Phase 3 완료

#### 작업 절차

1. **`docs/00-meta/STACK_SETUP_PLAN.md` 신설 (또는 갱신)** — 외부 의존 부트업 절차 1단락:

   ```markdown
   ## 외부 의존 부트업 (DB / Redis / S3 등)
   `/bootstrap-stack`이 스택 감지 시 다음 권장 출력:
   - Postgres: `docker-compose.yml` 또는 `supabase start` 권장.
   - Redis: `docker-compose.yml` 권장.
   - S3: localstack 또는 MinIO 권장.

   사용자가 채택 시 README에 1단락 + `make dev` / `pnpm dev` 등의 통합 진입점에 wiring.
   ```

2. **`/stack-guard` skill 갱신** — `.github/workflows/validate.yml` 권장 출력 (생성 X):

   ```markdown
   ## CI 권장 출력
   `/stack-guard`가 다음 형식의 권장 텍스트 출력 (파일 자동 생성 X — 사용자 결정):

   ```yaml
   # .github/workflows/validate.yml (권장)
   name: validate
   on: [push, pull_request]
   jobs:
     validate:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - run: <stack의 validate 명령>
   ```

   GUARDRAILS_STRATEGY *"OS/셸 종속 hook 강제 X"* 정신 — 권장만.
   ```

3. **ADR-025 본문 작성** — `docs/90-decisions/ADR-025-external-deps-and-ci-recommendation.md`. 본문 핵심:
   - scope=boilerplate
   - 결정 2종 한 흐름: `/bootstrap-stack` 외부 의존 권장 출력 + `/stack-guard` CI workflow 권장 출력
   - 강제 X, 권장만 (GUARDRAILS_STRATEGY 정신)

4. **ADR README 갱신**.

#### 검증

- [ ] STACK_SETUP_PLAN.md 외부 의존 부트업 단락
- [ ] stack-guard skill CI 권장 단락
- [ ] ADR-025 본문 + README

#### Commit message

```
docs: add ADR-025 with external deps bootup and CI workflow recommendations
```

---

### Step 9.4 — `/bootstrap-stack` monorepo 라운드 (skill amend)

**우선순위**: P1
**Evidence**: [외부실증]
**의존성**: Phase 4.4 (ADR-008 amend monorepo scope)

#### 작업 절차

1. **`/bootstrap-stack` skill 갱신** — monorepo 라운드 단락 추가:

   ```markdown
   ## monorepo 라운드 (감지 시 자동)
   1. **orchestrator 결정**: turbo / nx / pnpm workspaces only / lerna 등 1종.
   2. **shared 패키지 위치 + 버전 정책**: `packages/shared`, semver vs fixed.
   3. **publish 정책**: 외부 publish vs internal-only.
   4. **scope vocabulary**: 패키지명 목록을 ADR-008 amend의 scope 컨벤션과 정합화.
   ```

#### 검증

- [ ] bootstrap-stack skill monorepo 라운드 단락

#### Commit message

```
docs: add bootstrap-stack monorepo round to skill body
```

---

### Step 9.5 — `/plan-workitem` sizing/분리 가이드 (skill amend)

**우선순위**: P1
**Evidence**: [외부실증] (Nx/Turbo monorepo 패턴) — [관측됨]은 Phase 12 Round 2(monorepo 시나리오)에서 회수 예정
**의존성**: Phase 4.1 (ADR-026 sizing 한계), Step 9.4 (monorepo 라운드 결과의 scope vocabulary 참조)

#### 작업 절차

1. **`/plan-workitem` skill 갱신** — monorepo·백엔드 sizing 가이드 단락 추가:

   ```markdown
   ## monorepo·백엔드 sizing 가이드
   - **monorepo**: 1 task = 단일 패키지 5 파일 이하 (cross-package 변경은 task 분리).
   - **백엔드**: OpenAPI 변경·DB migration·코드 구현은 *별도 task*로 분리. 한 task에 묶지 않는다.
   - Phase 4.1의 sizing 휴리스틱(1 RGR / AC 3 / 변경 5)이 monorepo·백엔드에서 깨지는 문제는 *외부실증*(Nx affected / Turbo affected 패턴 + monorepo 컨벤션) 기반. 본 보일러플레이트의 [관측됨] 데이터는 Phase 12 Round 2(monorepo 시나리오)에서 회수.
   - **SSOT 노트**: 본 sizing 가이드는 본 skill 본문이 SSOT다. 운영 가이드라 ADR로 박지 않음 — ADR-005 패턴 4(정책=ADR)의 *경계 영역*. 추적성은 ADR-026 Amendment 1(Step 9.6)에서 명시.
   ```

#### 검증

- [ ] plan-workitem skill sizing 가이드 단락 (SSOT 노트 포함)

#### Commit message

```
docs: add monorepo and backend sizing guide to plan-workitem skill
```

---

### Step 9.6 — planner charter 정합 + architect-opus 신호 (ADR-026 본문 amend)

**우선순위**: P1
**Evidence**: [관측됨]
**의존성**: Phase 4.1 (ADR-026)

#### 작업 절차

1. **`/plan-workitem` skill 갱신** — 다음 두 단락 추가:

   ```markdown
   ## 정합성 self-check (분해 직후 1회 실행)
   - charter `## 5. 비목표` 단락 키워드와 분해된 feature/task를 매칭. 위반 의심 시 출력의 "남은 미결정 사항"에 명시.
   - feature 범위가 상위 milestone `## 3. 포함되는 기능`에 매핑되는지 확인. 매핑 실패 시 동일 위치에 명시.

   ## architect-opus 호출 권장 신호 (감지 시 텍스트 제안만, 자동 호출 금지 — ADR-007)
   다음 4 신호 중 하나라도 감지되면 출력 마지막에 `architect-opus 호출 권장: <이유>` 1줄 추가:
   1. 새 모듈 디렉터리 생성 (`src/<new>/` 또는 동등 경로).
   2. charter `## 7. 제약 조건`에 없는 새 외부 의존 (npm/pip/cargo) 추가.
   3. ARCHITECTURE_OVERVIEW.md `## 3-1. 레이어 경계` 변경.
   4. "패턴 변경" / "새 boundary" / "도메인 경계" 키워드 등장.
   ```

2. **ADR-026 본문 amend** — `docs/90-decisions/ADR-026-plan-workitem-schema.md` 본문 끝에 단락 추가:

   ```markdown
   ## Amendment 1 (2026-05-XX) — planner self-check + architect-opus 신호 + sizing SSOT
   - planner skill에 charter 정합 self-check 단락(2문장).
   - architect-opus 호출 권장 신호 4종 (텍스트 제안만, 자동 호출 X — ADR-007 정합).
   - **monorepo·백엔드 sizing 가이드의 SSOT는 plan-workitem skill 본문**(Step 9.5에서 추가됨). ADR-005 패턴 4(정책=ADR)의 *경계 영역* — 운영 가이드는 정책이 아니라 skill 본문에 둔다. 추적성을 본 amend에서 명시해 git history만으로 추적 안 되는 *추적성 손실* 위험 보강.
   - 잔여 모니터링: 첫 마일스톤 stabilize에서 architect-opus 신호 4종의 false positive 비율 측정. 50% 초과 시 신호 정밀도 강화 (예: 1번 "새 모듈 디렉터리 생성" 신호는 키워드 매칭 → AST 기반 디렉터리 감지로 대체).
   ```

#### 검증

- [ ] plan-workitem skill 두 단락 추가
- [ ] ADR-026 Amendment 1 단락 (sizing SSOT 명시 + false positive 모니터링 포함)

#### Commit message

```
docs: amend ADR-026 with planner charter check and architect-opus signals
```

---

## Part 10. Phase 10 — 트리거 대기 (P1/P2)

> 본 Phase의 항목은 *조건이 충족될 때까지 대기*. P1(ADR-018 같은 deferred)과 P2(ADR-039·040 같은 트리거 보류)를 모두 포함 — 본 가이드를 따라가며 *지금* 작업하지 않는다. 트리거 도달 시 본 가이드의 해당 Step을 *실행 가능 상태*로 활성화.

### 트리거 매트릭스

| Step | 항목 | 트리거 |
|------|------|--------|
| 10.1 | ADR 카테고리화 (`category: policy/arch/stack/tooling/misc`) | ADR ≥ 15 도달 시 (Phase 1~9 완료 후 자연 발동) |
| 10.2 | Switch Interview 1pager (ADR-039) | 사용자 인터뷰 시도 1회 발생 시 |
| 10.3 | Story Map + Shape Up Pitch (ADR-040) | milestone 3+ 잡힌 프로젝트 |
| 10.4 | 가벼운 enabling 항목 (charter language / PII 안내) | 해당 시나리오 등장 시 (관측됨 데이터 회수) |
| 10.5 | codebase 복잡도 예산 | 모듈 폭증 사건 1건 (Ratchet 강 적용 영역) |
| 10.6 | agent concurrency guard | 동일 task multi-fork 충돌 사건 1건 |
| 10.7 | validator + reviewer 통합 | 시뮬레이션이 출력 중복률 ≥ 30% 보고 시 |
| 10.8 | 00-meta 폴더명 변경 (operations/) | (영구 보류 — Diátaxis 가이드상 폴더명보다 모드 라벨이 강한 신호) |
| 10.9 | CODE_LINEAGE.md 자동 생성 (ADR-018) | (1) sticky question 1회+ 발생 / (2) fork 사용자 요청 / (3) 6개월 footer 누적 후 grep 부족 [관측됨] |

### Step 10.1 — ADR 카테고리화 (트리거: ADR ≥ 15)

**현재**: ADR 10개. Phase 1~9 완료 시 기존 10 + 신설 17 = ADR 약 27개 → 카테고리화 트리거(ADR ≥ 15) 발동.

**작업**: ADR frontmatter에 `category` 필드 추가, README 카테고리별 섹션 분리. *진짜 ratchet*(자라난 후 정리).

### Step 10.2 — Switch Interview 1pager (ADR-039 신설, 트리거: 인터뷰 1회)

**작업 절차** (트리거 도달 후):
1. `docs/10-charter/_templates/SWITCH_INTERVIEW.md` 신설 — JTBD 4-forces (push/pull/anxiety/habit) + 5단계 timeline 질문 (*첫 생각 → 검색 → 평가 → 선택 → 사용*).
2. `/discover-product` R0/R1에 1줄 추가: *"페르소나가 AI 추론만으로 생성됐다면 *가정*으로 표시. 인터뷰 1~3건 후 `--update`."*
3. **결과 환류 경로 명시** — SWITCH_INTERVIEW.md 본문 끝에 1단락: *"인터뷰 결과는 DISCOVERY.md `## 13. Opportunity Backlog`(신규 pain 행) + `## 12. Assumption Tracker`(검증 결과 행)로 환류. 별도 인터뷰 기록 파일 만들지 않음 — DISCOVERY.md가 PO 트랙 SSOT(ADR-035 정합)."*
4. ADR-039 본문 작성.

### Step 10.3 — Story Map + Shape Up Pitch (ADR-040 신설, 트리거: milestone 3+)

**작업 절차** (트리거 도달 후):
1. MILESTONE_TEMPLATE.md 7섹션 → 9섹션. `## 1-1 Appetite`, `## 2-1 Story Map`, `## 4-1 Rabbit holes` 모두 *옵션*.
2. ADR-040 본문 작성.

### Step 10.4 — 가벼운 enabling 항목 (트리거: 해당 시나리오 등장)

| 항목 | 트리거 | 작업 |
|------|-------|------|
| charter language 1줄 | 한국어가 아닌 fork 사용자 등장 | charter `## 0` 위 `> 작성 언어: <언어>` 1줄 + AGENTS.md 1줄 |
| PII/privacy 안내 | 개인 정보 처리 프로젝트 등장 | ARCHITECTURE `## 8.보안` 안내문 1줄 |

### Step 10.5 — codebase 복잡도 예산 (영구 No / 트리거 시 재검토)

*영구 No*. *측정-경고 only*는 ROI 의문, 관측된 실패 0건 (Ratchet 강 적용). fork 사례에서 모듈 폭증 사건 등장 시만 재검토.

### Step 10.6 — agent concurrency guard (영구 No / 트리거 시 재검토)

*영구 No*. AGENT_EXECUTION_STRATEGY 병렬 패턴 3종이 이미 worktree 권장. 동일 task multi-fork 충돌 사건 1건 시만 재검토.

### Step 10.7 — validator + reviewer 통합 (데이터 트리거 보류)

*데이터 트리거*. 시뮬레이션 dogfood가 validator/reviewer 출력 중복률 ≥30% 보고 시 통합 ADR 검토. 30% 미만이면 분리 유지가 데이터로 정당화.

### Step 10.8 — 00-meta 폴더명 변경 (영구 No)

*영구 No*. Diátaxis 공식 가이드 *"폴더명보다 내용의 mode가 강한 신호"* + cross-reference 일괄 변경 비용 큼. 효용은 *처음 fork 시 1회* 직관 향상에 한정.

### Step 10.9 — CODE_LINEAGE.md 자동 생성 (ADR-018 트리거 도달 시 활성화)

**작업 절차** (트리거 도달 후 — Step 7.1의 deferred 본문 활성화):
1. `docs/40-validation/CODE_LINEAGE.md` 신설 — 4 필드 표 (path / first / last / touches / active-AC) + 자동 생성 마커.
2. `/stabilize-milestone` skill에 *매 회 전량 재생성* 단계 추가 (idempotent, merge conflict 1회 재실행 해소, 삭제 파일 행 보존).
3. ADR-018 본문 작성 — Step 7.1의 *deferred* 본문 그대로 활성화.
4. STRUCTURE.md 산출물 표에 CODE_LINEAGE.md 행 추가 (Living, 자동 재생성).

**왜 트리거 도달 전 보류인가**: `Refs:` footer 1줄 컨벤션(Step 7.1 P0)만으로 `git log --grep="Refs: T-XXX"` ad-hoc 추적 가능 — 효용 90% 즉시 확보. 나머지 10%(자동 생성 view)는 *도구 작성·정확도 검증 비용*보다 *git history와 view 중복*의 SSOT 분산 우려가 크다. 트리거 발동 시 비용/효용 역전 → 활성화.

---

## Part 11. Phase 11 — 명시적 No 결정 (참고표)

> 본 Phase는 작업이 아니다. **하지 말아야 할 것** 명시 — Phase 1~10 작업 시 헷갈릴 때 본 표 참조.

### 영구 No (작업 안 함)

| 영역 | 항목 | 근거 |
|------|------|------|
| 인터페이스 | UI까지 ARCHITECTURE 흡수 | UI는 시스템 구조와 직교 → 잡음비 폭락 |
| 인터페이스 | 영역별 3개 파일 분리 (DESIGN_UI / DESIGN_API / DESIGN_CLI) | 단순성 1순위 위반 |
| 인터페이스 | repo root `DESIGN.md` 두기 | ADR-001 문서 계층 + ADR-005 진입 페이지 1개 + ADR-010 AGENTS.md 캐노니컬 진입 위반 |
| 인터페이스 | 외부 도구 root DESIGN.md 발견 마찰 자동 해소 | 사용자가 root에 1줄 stub 직접 추가 (보일러플레이트는 sync 책임 안 짐) |
| 문서 아키텍처 | 00-meta 폴더명 변경 (operations/) | Diátaxis 가이드 + cross-ref 비용 |
| 문서 아키텍처 | `_templates/` 분산 통합 | fork 사용자가 처음 1회만 보는 영역 |
| plan-workitem | plan 모드 lifecycle 통합 (정합 정책 4축) | ADR-010 위반 + ADR-006 단순성 위반 |
| plan-workitem | 2-pass planning (reviewer pass) | 토큰 2배 + stabilize reviewer 책임 중복 |
| plan-workitem | TASK_TEMPLATE risk·effort 추정 | 보일러플레이트가 정확도 보장 불가, YAGNI |
| plan-workitem | `/plan-workitem` → `/decompose-workitem` rename | cross-ref 일괄 변경 비용 > 효용 |
| stabilize | Release-level DoD | stabilize 출력에 자연 흡수 |
| stabilize | Fowler 4-quadrant 매트릭스 (severity × quadrant 2축) | severity(P0/P1/P2)만으로 충분, 매 항목 quadrant 라벨링 부담 |
| stabilize | METRICS.md 별도 파일 + 4지표 자동 누적 | git log + QA_FINDINGS로 ad-hoc 추출 가능, 시각화는 외부 도구 |
| stabilize | `--apply-carryover` 자동화 | default OFF + 명시 발화만 동작 → 사용 빈도 낮음, 수동 carry-over로 충분 |
| stabilize | architect-opus auto-escalation 자동 호출 / 신호 텍스트 제안 | false positive 우려, agent 본문 부담 |
| stabilize | SSOT drift 자동 검출 (5단어 동일 시퀀스) | false positive 큼, 사람 review로 충분 |
| 4 Pillars | codebase 복잡도 예산 | 측정-경고 only는 ROI 의문 |
| 4 Pillars | agent concurrency guard | 병렬 패턴 3종이 이미 worktree 권장 |
| 4 Pillars | Hindsight 동적 memory layer | Claude Code 전용 → ADR-010 위반 |
| 4 Pillars | 영역별 lint 자동화 (UI=design tokens / API=Pact·Schemathesis / CLI=snapshot test) | Pact/Schemathesis 학습 곡선 큼. Step 5.4 self-check로 동등 효용 |
| 4 Pillars | Context Packs 영역별 사전 정의 (frontend/backend pack) | 과설계 — minimal/full 2종으로 충분. 사용자 필요 시 fork에서 자체 정의 |
| 4 Pillars | MCP integration baseline (settings.json `mcpServers: {}` + 권한·secret 분류) | Claude Code 특이 → ADR-010 약충돌. 비-MCP 사용자에게 placeholder 무의미 |
| 스택 범위 | 비웹 스택의 *기본 자동화 직접 지원* (mobile/ML/embedded/game/desktop) | ADR-031 (Step 2.2). override 경로 제공 — *지원 안 함*이 아니라 *기본값 최적화 안 함* |
| 스택 범위 | finalize `--pr` 플래그 | finalize 책임 경계가 commit까지 (ADR-007) |
| 스택 범위 | Supabase RLS 자동 검증 | 도구 중립(ADR-010) 깸 |
| 기획 | spec-kit `constitution.md` 별도 파일 추가 | AGENTS.md + ADR-006 단순성 정책이 동등 매핑 → ADR-005 진입 페이지 1개 + ADR-010 IDE lock-in 회피와 정합 (ADR-036 비결정 No 2) |
| 기획 | PMF playbook + 측정 권장 정책 (Sean Ellis 40% test / Superhuman 4-step) | fork 대부분이 PMF 비목표(CLI/내부 도구/API 서버) — 보일러플레이트 정체성과 약함. SaaS는 fork 사용자가 직접 도입 |
| 기획 | 자체 발명 PRD 양식 (외부 표준 무시) | 외부 표준 학습 비용 0 + ADR-005 SSOT 위반 (ADR-036 비결정) |

### 조건부 No (트리거 시 재검토)

조건부 No 항목들은 *Phase 10 트리거 매트릭스*에서 SSOT로 관리한다 (ADR-005 SSOT 정합). Phase 10.1~10.9 참조.

### 데이터 트리거 보류

| 영역 | 항목 | 데이터 |
|------|------|------|
| 4 Pillars | AC ID 컨벤션 P0 격상 | 시뮬레이션 통과 + 누락률 ≤5% (현재 P1) |
| Phase 1 | 본 가이드 P0 우선순위 재조정 | dogfood 시뮬레이션 결과 |

---

## Part 12. Phase 12 — 통합 검토 + 회귀 시뮬레이션

> 본 Phase는 Phase 1~9 완료 후 *마지막 검증*. 모든 결정이 한 시스템으로 정합한지 점검.

### Step 12.1 — 통합 정합성 체크

#### 체크 항목

- [ ] **ADR 번호 충돌 0건** — 0.3 ADR 마스터 표 vs 실제 `docs/90-decisions/` 디렉터리 일치.
- [ ] **scope 라벨링** — 모든 ADR (002, 003 placeholder 제외)에 `scope: boilerplate`.
- [ ] **AGENTS.md ≤ 100줄** — 본 가이드 작업으로 누적된 모든 추가 합산해 cap 안. 누적 항목: Phase 2.2 *기본 자동화 직접 지원 범위* 1단락 + Phase 3.1 *Claude Code plan 모드* 1단락 + Phase 3.4 *AGENTS.md 길이 정책* 1단락 + Step 5.1 *시각 디자인* 인덱스 1줄 + Phase 8.1 *DISCOVERY=SSOT* 1줄 + Phase 8.2 boundaries 3-tier 라벨링(0줄 — 기존 항목 라벨링만). 시작 ~35줄 + 누적 ~10~15줄 = 약 45~50줄 (cap 안전).
- [ ] **STRUCTURE.md 산출물 표 정합** — 신설 산출물 (DESIGN.md, SIMULATION_RUN.md) 모두 행 존재. CODE_LINEAGE.md는 트리거 도달 전이라 부재 정상 (도달 시 Phase 10.9에서 추가).
- [ ] **STRUCTURE.md Canonical Owner 매핑 정합** — Phase 5/8에서 추가된 행 모두 존재.
- [ ] **Diátaxis 모드 라벨** — docs/00-meta 6문서 모두 첫 줄 라벨.
- [ ] **evidence label** — IMPROVEMENT_GUIDE / QA_FINDINGS 신규 항목에 `[관측됨]`/`[외부실증]`/`[가설]` 박힘.
- [ ] **`Refs:` footer 컨벤션 적용** — Phase 7.1 이후 모든 commit에 `Refs: T-NNN` 또는 `Refs: chore` footer 박힘 (CODE_LINEAGE.md 자동 재생성은 트리거 도달 후 점검).
- [ ] **`/review-doc` 통합** — 모드 mismatch / AGENTS.md cap 점검 모두 동작.

### Step 12.2 — 회귀 시뮬레이션 (ADR-017 회차 2)

**트리거**: 본 가이드 모든 P0 작업 완료 (신설 17개 + amend 6개) → ADR-017의 *재실행 트리거*("새 ADR 도입 / lifecycle 단계 변경 / skill 본문 큰 변경") 발동.

#### 시나리오 선택 — *Round 1과 다른 시나리오 권장*

Round 2는 Round 1(todo CLI / Node+TS+Vitest)과 *다른 스택 시나리오*를 선택해 신설 결정의 폭넓은 동작 검증을 권장한다. 같은 시나리오 반복은 Round 1에서 이미 [관측됨]으로 회수된 영역만 다시 측정 — *Round 2의 marginal value*가 작다.

| 옵션 | 시나리오 | 검증되는 결정 (Round 1로는 못 잡음) |
|------|---------|-------------------------------|
| **권장 A** | Express API + Postgres (`docker compose up postgres`) | ADR-025 외부 의존 부트업 / ADR-007 lock file(npm) / ARCHITECTURE 7-3 백엔드 결정 / OpenAPI·migration task 분리 (Step 9.5) |
| **권장 B** | turbo monorepo (api + web + shared 3 패키지) | ADR-008 amend 모노레포 scope / Step 9.4 monorepo 라운드 / Step 9.5 패키지당 5 파일 sizing |

**최소 권장**: Round 2 = 옵션 A 또는 B 중 1종. 두 옵션 모두 Round 1의 todo CLI에서는 작동 자체가 불가능했던 결정을 검증한다.

#### 작업 절차

1. **dogfood-<scenario>/ 디렉터리 별도 fork** (예: `dogfood-express-api/` 또는 `dogfood-turbo-monorepo/`).
2. **Step 1.1과 동일 절차** — 8단계 lifecycle 1회 통과.
3. **`docs/40-validation/SIMULATION_RUN.md`에 Round 2 추가** — Step 1.1의 형식과 동일. 회차 헤더에 *선택한 시나리오* 명시 (예: `## Round 2 (2026-XX-XX, Express API / Node+Postgres+Vitest)`).
4. **Round 2 vs Round 1 비교** — 다음 데이터 회수:
   - 사용자 개입 횟수 비교 (목표: Round 2 ≤ Round 1, 단 시나리오 복잡도 차이 감안)
   - 충원율 비교 (목표: Round 2 ≥ Round 1)
   - graduation pre-check 미통과 사유 비교 (목표: Round 2 ≤ Round 1)
   - 신설 결정 (FAC↔AC 매핑 / lock file 화이트리스트 / Given-When-Then 강제 / 시나리오별 신규 결정 — 위 표의 *검증되는 결정* 컬럼) 동작 확인

5. **Round 2 미충족 시 후속 ADR 후보** — 발견된 깨짐을 IMPROVEMENT_GUIDE에 **P0 severity**로 보고. Round 2는 P0/P1 결정이 박힌 후의 회귀 시뮬레이션이므로 깨짐은 *lifecycle 단절* 시그널 — 본 가이드를 근거로 추가 ADR 작성.

#### Commit message

```
docs: record dogfood simulation round 2 in SIMULATION_RUN.md
```

### Step 12.3 — Phase 9 (P1) 결정의 데이터 트리거 점검

Round 2 결과를 기반으로 다음 *데이터 트리거*가 충족됐는지 점검:

- **ADR-009 amend AC ID P1→P0 격상** — Round 2의 누락률 ≤ 5%인가?
- **6→3 agent 통합** — Round 2의 validator/reviewer 출력 중복률 ≥ 30%인가?

각 트리거 충족 시 후속 ADR 작성 (Phase 10에서 활성화 예정 항목).

### Step 12.4 — 본 가이드 archive

본 가이드의 모든 작업이 완료되면, 본 IMPROVE-GUIDE.md는 *역사 기록*으로 보존하거나 archive 디렉터리(예: `docs/99-archive/`)로 이동할 수 있다. ADR-005 SSOT 정합상 모든 결정 본문은 각 ADR이 소유하며, 본 가이드는 *작업 절차의 역사*로만 의미를 갖는다.

트리거 대기 항목(P1·P2)은 IMPROVEMENT_GUIDE.md에 *대기 큐*로 유지하고, 본 가이드 Part 10(Phase 10)을 참조 인덱스로 활용한다.

#### Commit message

```
docs: archive IMPROVE-GUIDE after all P0/P1 decisions ratified
```

---

## Part 13. 부록 — 빠른 진행 체크리스트

> 본 가이드를 따라가며 진행 상태를 빠르게 추적할 때 사용.

### Phase 1 (P0-Gate)
- [ ] Step 1.1 — dogfood 시뮬레이션 1회 실행 + SIMULATION_RUN.md Round 1

### Phase 2 (자기 일관성 + 범위 한정)
- [ ] Step 2.1 — ADR-022 Ratchet Principle
- [ ] Step 2.2 — ADR-031 비웹 스택 기본 자동화 범위 밖 명시 (override 경로 제공)
- [ ] Step 2.3 — ADR-000 Boilerplate ADR scope 라벨링

### Phase 3 (단순 cleanup)
- [ ] Step 3.1 — ADR-024 plan 모드 lifecycle 비범위
- [ ] Step 3.2 — ADR-012 1단계 (00-meta 9→6 흡수)
- [ ] Step 3.3 — ADR-012 2단계 (Diátaxis 모드 라벨)
- [ ] Step 3.4 — ADR-011 AGENTS.md 100줄 hard cap

### Phase 4 (Schema 강화)
- [ ] Step 4.1 — ADR-026 TASK_TEMPLATE schema
- [ ] Step 4.2 — ADR-009 amend AC ID 컨벤션 (P1, Round 2 누락률 ≤ 5% 도달 시 P0 격상)
- [ ] Step 4.3 — ADR-007 amend lock file 화이트리스트
- [ ] Step 4.4 — ADR-008 amend 1차 monorepo scope

### Phase 5 (인터페이스 결정 흐름)
- [ ] Step 5.1 — DESIGN_SYSTEM → DESIGN.md rename
- [ ] Step 5.2 — ARCHITECTURE 7-1/7-2/7-3/7-4 신설
- [ ] Step 5.3 — `/bootstrap-design` skill 신설
- [ ] Step 5.4 — agent self-check 보강
- [ ] Step 5.5 — ADR-027 본문 작성

### Phase 6 (Milestone graduation)
- [ ] Step 6.1 — graduation checklist + 회고 (MILESTONE_TEMPLATE)
- [ ] Step 6.2 — pre-check + `--dry-run`
- [ ] Step 6.3 — ADR-014 + ADR-017 본문 작성

### Phase 7 (4 Pillars 정합)
- [ ] Step 7.1 — ADR-008 amend 2차 (Refs footer, P0) / CODE_LINEAGE.md 자동 생성은 P1 트리거 보류 (Phase 10.9)
- [ ] Step 7.2 — ADR-019 JIT 로딩 + Context Packs (minimal/full 2종)
- [ ] Step 7.3 — ADR-021 정적 분석 권장 + amend secret scanner
- [ ] Step 7.4 — 코드 organization 가이드
- [ ] Step 7.5 — sub-agent 출력 cap 명문화

### Phase 8 (기획 PO/PM 트랙)
- [ ] Step 8.1 — ADR-035 DISCOVERY.md living doc
- [ ] Step 8.2 — ADR-036 FEATURE_TEMPLATE PRD 강화
- [ ] Step 8.3 — ADR-037 Spec coverage self-audit

### Phase 9 (P1 보강)
- [ ] Step 9.1 — ADR-020 `validate --changed`
- [ ] Step 9.2 — Dependency hygiene
- [ ] Step 9.3 — ADR-025 외부 의존 + CI 권장
- [ ] Step 9.4 — bootstrap-stack monorepo 라운드
- [ ] Step 9.5 — plan-workitem sizing 가이드
- [ ] Step 9.6 — planner charter 정합 + architect-opus 신호

### Phase 12 (통합 검토)
- [ ] Step 12.1 — 통합 정합성 체크
- [ ] Step 12.2 — Round 2 회귀 시뮬레이션 (Round 1과 다른 시나리오 1종 권장 — Express API+Postgres 또는 turbo monorepo)
- [ ] Step 12.3 — Phase 9 데이터 트리거 점검
- [ ] Step 12.4 — 본 가이드 archive

### 권장 commit 메시지 일람

> 각 Step 끝에도 박혀 있지만, 한눈에 보기 위해 한 번 더 모은다. 영어 한 줄 + `docs:` prefix.

```
# Phase 1
docs: run dogfood simulation round 1 and record SIMULATION_RUN.md

# Phase 2
docs: add ADR-022 ratchet principle with constraint/enabling scope
docs: add ADR-031 declaring non-web stacks out of direct support scope
docs: add ADR-000 meta policy with scope labels for boilerplate ADRs

# Phase 3
docs: add ADR-024 removing Claude Code plan mode from lifecycle
docs: absorb 4 docs into core 6 under docs/00-meta per SSOT cleanup
docs: add ADR-012 with diataxis mode labels and review-doc check
docs: add ADR-011 enforcing AGENTS.md 100-line hard cap

# Phase 4
docs: add ADR-026 with TASK_TEMPLATE schema and planner self-check
docs: amend ADR-009 to enforce AC ID convention in test names
docs: amend ADR-007 with finalize lock file whitelist
docs: amend ADR-008 to add monorepo scope as package name

# Phase 5
docs: add ADR-027 reorganizing interface decisions across DESIGN/ARCHITECTURE

# Phase 6
docs: add milestone graduation checklist and retrospective sections
docs: add stabilize graduation pre-check step with --dry-run flag
docs: add ADR-014 graduation contract and ADR-017 dogfood simulation

# Phase 7
docs: amend ADR-008 with Refs footer convention (CODE_LINEAGE.md deferred to P1 trigger)
docs: add ADR-019 with JIT context loading and context-pack frontmatter
docs: add ADR-021 with stack-specific static analysis and secret scanner
docs: add stack-specific default directory layout to bootstrap-stack
docs: add 1-2K output token cap to all sub-agents

# Phase 8
docs: add ADR-035 with DISCOVERY.md as living doc and assumption tracker
docs: add ADR-036 expanding FEATURE_TEMPLATE to 12-section PRD
docs: add ADR-037 with FAC to AC coverage self-audit in validator

# Phase 9
docs: add ADR-020 with validate --changed for incremental check
docs: add stabilize dependency hygiene scan
docs: add ADR-025 with external deps bootup and CI workflow recommendations
docs: add bootstrap-stack monorepo round to skill body
docs: add monorepo and backend sizing guide to plan-workitem skill
docs: amend ADR-026 with planner charter check and architect-opus signals

# Phase 12
docs: record dogfood simulation round 2 in SIMULATION_RUN.md
docs: archive IMPROVE-GUIDE after all P0/P1 decisions ratified
```

---

## Part 14. 부록 — ADR 일람 (역참조)

> 본 가이드의 Phase·Step에서 박히는 ADR을 ADR 번호 순으로 일람. 신규 산출물·skill amend·agent amend 포함.

### 신설 ADR

| ADR | 주제 | Phase / Step | 핵심 산출물 |
|-----|-----|------------|-----------|
| ADR-000 | Boilerplate decision policy (메타) | Step 2.3 | scope 라벨링 + supersede + 번호 정책 |
| ADR-011 | AGENTS.md 100줄 hard cap | Step 3.4 | AGENTS.md 길이 정책 1단락 + `/review-doc` cap 점검 |
| ADR-012 | docs/00-meta 9→6 흡수 + Diátaxis 모드 라벨 | Step 3.2~3.3 | 4 파일 흡수 + 6 파일 첫 줄 라벨 + `/review-doc` mismatch |
| ADR-014 | Milestone graduation contract | Step 6.1~6.3 | MILESTONE_TEMPLATE checklist + 회고 + pre-check + `--dry-run` |
| ADR-017 | Dogfood 시뮬레이션 의무 + 재실행 트리거 | Phase 1, Step 6.3 | SIMULATION_RUN.md (Record) |
| ADR-018 | CODE_LINEAGE.md 자동 생성 (P1 트리거 보류, deferred) | Step 7.1 (footer만 P0) / Phase 10.9 (자동 생성 활성화) | 4 필드 표 + 전량 재생성 — 트리거 도달 시 활성화 |
| ADR-019 | Context Packs frontmatter + JIT 로딩 | Step 7.2 | minimal (default) / full (architect-opus) 2종 |
| ADR-020 | `validate --changed` (incremental) | Step 9.1 | finalize 직전 `--changed`만 |
| ADR-021 | `/stack-guard` 정적 분석 1종 권장 (+ amend 1) | Step 7.3 | dependency-cruiser / import-linter / go vet / cargo deny + gitleaks/trufflehog |
| ADR-022 | Ratchet Principle 명문화 + 적용 범위 한정 | Step 2.1 | 제약(강) / enabling(약) 차등 + evidence label |
| ADR-024 | Claude Code plan 모드 lifecycle 비범위 | Step 3.1 | plansDirectory 삭제 + plans/ 디렉터리 + 4 문서 정리 |
| ADR-025 | `/bootstrap-stack` 외부 의존 + `/stack-guard` CI 권장 | Step 9.3 | STACK_SETUP_PLAN.md + GitHub workflow 권장 출력 |
| ADR-026 | plan-workitem 강화 (TASK_TEMPLATE schema + amend) | Step 4.1, 9.6 | sizing/AC verb/Given-When-Then/의존성 + planner self-check |
| ADR-027 | 인터페이스 결정 책임 분배 | Step 5.1~5.5 | DESIGN.md(UI) + ARCHITECTURE 7-1/7-2/7-3/7-4 + `/bootstrap-design` |
| ADR-031 | 비웹 스택을 기본 자동화 직접 지원 범위 밖으로 명시 | Step 2.2 | 직접 지원 5종 + override 경로 제공 |
| ADR-035 | Continuous discovery (DISCOVERY.md living doc) | Step 8.1 | Assumption Tracker + Opportunity Backlog + `--update` |
| ADR-036 | Feature-level PRD (FEATURE_TEMPLATE 12섹션) | Step 8.2 | User Story + 시나리오 + FAC + NFR + boundaries 3-tier |
| ADR-037 | Spec coverage self-audit | Step 8.3 | validator FAC↔AC 매핑 점검 |
| ADR-039 | JTBD Switch Interview 1pager (P2 트리거) | Step 10.2 | (인터뷰 시도 1회 시 활성화) |
| ADR-040 | Milestone scoping (Story Map + Pitch, P2 트리거) | Step 10.3 | (milestone 3+ 시 활성화) |

### 기존 ADR amend

| ADR | Amendment | Phase / Step | 내용 |
|-----|-----------|------------|------|
| ADR-007 | Amendment 1 | Step 4.3 | finalize lock file 화이트리스트 11종 |
| ADR-008 | Amendment 1 | Step 4.4 | 모노레포 scope = 패키지명 |
| ADR-008 | Amendment 2 | Step 7.1 | Conventional Commits `Refs:` footer |
| ADR-009 | Amendment 1 | Step 4.2 | AC ID 컨벤션 강화 (`AC_N` / `[AC-N]`, P1 → 데이터 트리거 시 P0) |
| ADR-021 | Amendment 1 | Step 7.3 | Secret scanner (gitleaks / trufflehog) 권장 |
| ADR-026 | Amendment 1 | Step 9.6 | planner charter 정합 self-check + architect-opus 신호 4종 |

### 신설 산출물 (ADR 외 — STRUCTURE.md 산출물 표 갱신 대상)

| 산출물 | 위치 | 라이프사이클 | 박힌 Phase / Step |
|--------|-----|-------------|----------------|
| SIMULATION_RUN.md | `docs/40-validation/SIMULATION_RUN.md` | Record | Phase 1, Step 6.3 |
| CODE_LINEAGE.md | `docs/40-validation/CODE_LINEAGE.md` | Living (자동 재생성) | Step 7.1 (deferred) / Phase 10.9 (트리거 활성화) |
| DESIGN.md | `docs/20-system/DESIGN.md` (rename) | Living | Step 5.1 |
| `/bootstrap-design` | `.claude/skills/bootstrap-design/SKILL.md` | Reference | Step 5.3 |
| `_ADR_GUIDE.md` Ratchet 단락 | `docs/90-decisions/_ADR_GUIDE.md` | Reference | Step 2.1 |
| STACK_SETUP_PLAN.md | `docs/00-meta/STACK_SETUP_PLAN.md` | Reference | Step 9.3 |

---

> **본 가이드 끝**.
> 작업 완료 후 본 IMPROVE-GUIDE.md는 *역사 기록*으로 보존하거나 archive 디렉터리로 이동할 수 있다. ADR-005 SSOT 정합상 모든 결정 본문은 각 ADR이 소유하며, 본 가이드는 *작업 절차의 역사*로만 의미를 갖는다.


