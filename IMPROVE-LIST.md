# Improve List

## 권장 적용 순서

1. **baseline / generated 경계 정리**: I-02, I-03, I-04, I-18.
2. **보일러플레이트 영역 분리**: I-14, I-16 (I-07 / I-08 인덱스 정책과 *함께 설계*).
3. **multi-tool agent 정합성**: I-15, I-19, I-20, I-13 일부(`.claude/agents/*.md`).
4. **조건부 산출물 실행 경로**: I-05, I-13 일부(`/bootstrap-design`, `/bootstrap-stack`).
5. **검증 / 누적 기록**: I-06, I-12, I-13 일부(`/stabilize-milestone`, `/review-doc`).
6. **진입 문서 / 사용자 동선**: I-01, I-09, I-10, I-11, I-17.
7. **ADR 메타 추적성 (마이그레이션 실행)**: I-07, I-08 (2단계에서 설계 완료 가정).

---

## I-01 — README_ko.md "디자인 시스템" 표현 stale

- **우선순위**: P2
- **현재 문제**: [README_ko.md:115](README_ko.md) — `│   ├── 20-system/     # 아키텍처 개요, 디자인 시스템`. ADR-027로 `DESIGN_SYSTEM.md → DESIGN.md` rename + UI 한정 결정 후 stale. [README.md](README.md) 영문판은 *"Architecture overview, UI design"*으로 정확.
- **근거**: README 상단 주석 *"구조 변경 시 README.md와 README_ko.md를 동시에 갱신한다"*가 실제 운용에서 1건 누락 → cross-language sync 신뢰도 저하. SSOT(ADR-005) 패턴 5의 표면 drift.
- **해결방안**: README_ko.md:115를 `│   ├── 20-system/     # 아키텍처 개요, UI 디자인 (UI 프로젝트 한정)`로 수정.

---

## I-02 — ADR-012 "정확히 6개 파일" ↔ 실제 docs/00-meta 7개 파일

- **우선순위**: P0
- **현재 문제**: [ADR-012:39](docs/90-decisions/ADR-012-doc-architecture-cleanup.md) — *"docs/00-meta/에 정확히 6개 파일"*. 실제 현재 트리는 7개:
  1. AGENT_EXECUTION_STRATEGY.md
  2. GLOSSARY.md
  3. GUARDRAILS_STRATEGY.md
  4. NEW_PROJECT_CHECKLIST.md
  5. **STACK_SETUP_PLAN.md** (ADR-012 이후 ADR-021/025 단계에서 추가)
  6. STRUCTURE.md
  7. WORKFLOW.md
  
  ADR-012 결정 B의 Diátaxis 모드 라벨 표에도 STACK_SETUP_PLAN.md 미포함.
- **근거**: SSOT를 핵심 가치로 박은 보일러플레이트(ADR-005)에서 *정책 ADR과 현재 구조의 직접 충돌*은 가장 강한 자기 일관성 위반. agent가 ADR-012 본문을 읽고 "/00-meta는 6개"로 인식하면 STACK_SETUP_PLAN.md를 잉여 파일로 오해할 위험.
- **해결방안**: **I-18에 위임** — I-18 방식으로 STACK_SETUP_PLAN.md를 baseline에서 빼면 ADR-012의 *"정확히 6개 파일"* 원칙이 자동 복원되어 amendment 자체가 불필요해진다.
  - 본 항목은 *문제 제기* 위치로 남기고 *해결 액션은 I-18에서 수행*.
  - 만약 I-18이 어떤 이유로 부결될 경우에만 fallback으로 ADR-012 Amendment 1(7개로 갱신 + Diátaxis 라벨 행 추가)을 진행. 1순위 권장은 *baseline 복원*(I-18 경로).
  - 채택 기록은 I-18 PR 본문에 *"본 PR이 I-02 해소"* 명시.

---

## I-03 — DESIGN.md baseline 모순 (조건부 산출물 vs 실제 baseline 파일)

- **우선순위**: P0
- **현재 문제**: [docs/20-system/DESIGN.md:8](docs/20-system/DESIGN.md) 본문 주석 — *"본 문서는 UI 프로젝트일 때만 만들어진다. /bootstrap-design이 R0~R4 라운드를 통해 채운다. 비-UI 프로젝트(API 서버 / CLI 도구)는 본 파일을 만들지 않는다."* 그러나 baseline에 **이미 존재**. [STRUCTURE.md:22](docs/00-meta/STRUCTURE.md)도 `/bootstrap-design (UI 스택 포함 시)` *생성* 어조.
- **근거**: 다음 2 모순이 누적:
  1. "만들어진다" 어조 vs baseline 존재 — fork API 서버 사용자가 *삭제 vs 비워두기*를 결정 못 함.
  2. ADR-027 결정 3 *"/bootstrap-design skill 신설"*이 *baseline placeholder 채움*인지 *부재 상태에서 생성*인지 명문화 부재.
- **해결방안**: 다음 2 안 중 택 1.
  - **안 A (이동)**: `docs/20-system/DESIGN.md` → `docs/20-system/_templates/DESIGN_TEMPLATE.md`로 이동. /bootstrap-design이 *template에서 복사 후 채움* 흐름. STRUCTURE.md / ADR-027 amend로 추적성 갱신.
  - **안 B (명시화)**: 현재 위치 유지. 본 파일 1행에 `<!-- baseline placeholder. UI 프로젝트는 /bootstrap-design으로 채움. 비-UI 프로젝트는 fork 직후 본 파일 삭제. -->` 주석 추가. STRUCTURE.md presence 컬럼(I-04 참조)에 `conditional` 마킹.
  
  단순성(ADR-006) 정합은 **안 B**. SSOT 정합은 안 A. 1순위 권장은 **안 B**.

---

## I-04 — STRUCTURE.md 산출물 표에 `presence` 컬럼 부재

- **우선순위**: P1
- **현재 문제**: [STRUCTURE.md:17-36](docs/00-meta/STRUCTURE.md) 산출물 표는 `산출물 | 위치 | 생성 주체 | 라이프사이클` 4컬럼. 다음 4종 산출물이 한 컬럼에 섞임:
  - **baseline**: 보일러플레이트에 *이미 있는* 파일 (예: PROJECT_CHARTER.md 템플릿, ARCHITECTURE_OVERVIEW.md 템플릿)
  - **generated on use**: skill 호출 시 *최초 생성*되는 파일 (예: DISCOVERY.md, validation reports)
  - **conditional**: 특정 스택에서만 존재 (예: DESIGN.md UI 한정)
  - **reserved**: 번호 placeholder (예: ADR-002, ADR-003)
- **근거**: AI agent가 "누락"인지 "아직 생성 전"인지 구분 못 함 → fork 직후 *없어야 할 파일을 만들거나 / 있어야 할 파일을 건드리거나* 오인 가능.
- **해결방안**: STRUCTURE.md 산출물 표에 5번째 컬럼 `presence` 추가. 값: `baseline` / `generated` / `conditional` / `reserved`. 각 산출물 25행에 1값 박음. 정의 ADR (신규) 또는 STRUCTURE.md 본문에 4값 정의 1단락 추가.

---

## I-05 — ARCHITECTURE 7-1~7-4 비해당 섹션 자동 삭제 책임 부재

- **우선순위**: P1
- **현재 문제**: [ARCHITECTURE_OVERVIEW.md:41-138](docs/20-system/ARCHITECTURE_OVERVIEW.md) — `## 7-1 API` / `## 7-2 CLI` / `## 7-3 백엔드` / `## 7-4 프론트` sub-section이 각각 *"비-X 프로젝트는 통째 삭제"* 가이드만 명시. **agent가 실제로 삭제하는 보장 없음**. ADR-027 결정 15도 *"비대화 수용"*만 명시.
- **근거**: 비-API/CLI/백엔드/프론트 프로젝트(예: library 패키지)가 fork하면 ~80줄짜리 conditional placeholder가 그대로 잔존 → 문서 노이즈 누적.
- **해결방안**: 다음 3 변경.
  1. ARCHITECTURE_OVERVIEW.md의 conditional sub-section 주석을 **명령형**으로 강화: 현재 *"비-API 프로젝트는 통째 삭제"* → *"비-API 프로젝트는 fork 직후 본 sub-section 통째 삭제 — /bootstrap-stack이 자동 수행"*.
  2. `/bootstrap-stack` SKILL 본문에 *"스택 감지 후 비해당 7-x sub-section 삭제"* 책임 1행 추가.
  3. `/stabilize-milestone`이 conditional sub-section *잔존* 발견 시 IMPROVEMENT_GUIDE에 P2 보고.

---

## I-06 — QA_FINDINGS / IMPROVEMENT_GUIDE 항목 스키마 부재

- **우선순위**: P1
- **현재 문제**: [QA_FINDINGS.md](docs/40-validation/QA_FINDINGS.md)는 `### P0/P1/P2/관찰 메모` 헤더만, [IMPROVEMENT_GUIDE.md](docs/40-validation/IMPROVEMENT_GUIDE.md)는 `## 0~4` 섹션만. **항목 자체의 형식 미정의**. ADR-022 evidence label도 *"박는다"*만 명시되어 있고 *어떤 필드에, 어떤 형식으로*는 미정의.
- **근거**: 3~4 마일스톤 후 항목 30~50개 누적 시 검색성(*"FAC-4 unmapped 어느 마일스톤 어느 회차?"*)과 종결성(*"이 항목이 해결됐는가, 미해결인가?"*)이 모두 떨어짐.
- **해결방안**: QA_FINDINGS.md / IMPROVEMENT_GUIDE.md 본문에 다음 스키마 예시 1개 박음.
  ```markdown
  - **F-M1-001** | P1 | [관측됨] | linked: T-002 | status: open
    - 발견: FAC-4 → T-002:AC-N 매핑 누락, validate 통과인데 spec gap.
    - 결정: 다음 라운드 plan에서 T-002에 AC-3 추가.
  ```
  필수 4필드 (`ID / severity / evidence label / linked workitem`) + 권장 2필드 (`status / decision`).

---

## I-07 — ADR 인덱스에 reserved/parked 번호 추적성 부재

- **우선순위**: P1
- **현재 문제**: [docs/90-decisions/README.md](docs/90-decisions/README.md)의 Boilerplate ADR 표가 **000, 001, 004~012, 014, 017, 019~022, 024~027, 031, 035~037**만 나열. 누락 번호 **013, 015, 016, 018, 023, 028~030, 032~034**의 처분 미추적.
  - 결정적 증거: [ADR-008:77](docs/90-decisions/ADR-008-commit-convention.md) Amendment 2가 *"CODE_LINEAGE.md(ADR-018, P1 트리거 보류)"* 인용 → ADR-018은 reserved이나 인덱스에서 추적 불가.
- **근거**: fork 사용자가 *"왜 18이 없는가"*를 git log 외에서 답할 수 없음. ADR-000의 *"fork 번호 충돌 회피"* 정책과 정합 어긋남.
- **해결방안**: docs/90-decisions/README.md에 `### Reserved / Parked / Dropped 번호` 섹션 1개 추가. 각 누락 번호에 1행:
  - `ADR-018 | parked | CODE_LINEAGE.md (Refs footer SSOT, P1 트리거 보류)`
  - `ADR-013/015/016/... | dropped | Phase X에서 fold됨` (git log로 사유 확인 후 박음)
  
  추가로 `_ADR_GUIDE.md` "상태값"에 `reserved` 또는 `parked` 1행 추가.

---

## I-08 — ADR Amendment의 인덱스 가시성 부재

- **우선순위**: P1
- **현재 문제**: [docs/90-decisions/README.md](docs/90-decisions/README.md) 인덱스의 "한 줄 요약" 컬럼이 *base ADR만 반영*. 다음 amendment가 본문에는 있지만 인덱스에 안 보임:
  - ADR-007 Amendment 1 (lock file whitelist 11종)
  - ADR-008 Amendment 1·2 (monorepo scope, Refs footer)
  - ADR-009 Amendment 1 (AC ID 컨벤션)
  - ADR-021 Amendment 1 (secret scanner)
  - ADR-026 Amendment 1 (planner self-check, architect-opus 신호)
- **근거**: fork 사용자가 README 인덱스만 보고 ADR-008을 *"Conventional Commits 기본 채택"*으로만 인식 → `Refs:` footer 컨벤션을 놓침. SSOT 위반은 아니지만 *발견 비용 누적*.
- **해결방안**: docs/90-decisions/README.md Boilerplate ADR 표에 `Amendments` 컬럼 1개 추가. 또는 "한 줄 요약" 뒤에 `(+amend1: lock whitelist, +amend2: Refs footer)` 형식 라벨.

---

## I-09 — README "Same workflow applies" 표현 과장

- **우선순위**: P2
- **현재 문제**: [README.md:93](README.md) — *"Same workflow applies"*. 실제 Codex wrapper coverage는 8/13 (62%):
  - Wrapper 8개: implement/validate/repair/finalize/plan-workitem/bootstrap-project/bootstrap-stack/stabilize-milestone
  - 자연어 fallback 5개: discover-product/stack-guard/review-doc/boilerplate-context/bootstrap-design
- **근거**: "동등"이라기보다 *"문서·정책 동등 + 핵심 skill wrapper + 나머지 자연어 fallback"*이 정확. 사용자가 README만 보고 *완전 동등*으로 오해할 수 있음. ADR-010 D6 (Phase 2 보류) 결정과 정합 어긋남.
- **해결방안**: README.md "Using with Codex CLI" 단락 표현을 다음과 같이 정밀화.
  - 현재: *"Same workflow applies: see WORKFLOW.md"*
  - 변경: *"Documents and policies are equal. Core workflow skills have Codex wrappers ($-prefixed). Remaining skills (discover-product, stack-guard, review-doc, boilerplate-context, bootstrap-design) are invoked via natural language."*
  
  README_ko.md도 동시 갱신 (cross-language sync 정합).

---

## I-10 — NEW_PROJECT_CHECKLIST에 `/plan-workitem` 명령 박스 + Status 전환 체크 부재

- **우선순위**: P2
- **현재 문제**: [NEW_PROJECT_CHECKLIST.md:22-25](docs/00-meta/NEW_PROJECT_CHECKLIST.md) `## 2. 작업 구조 준비`가 *결과 체크*("milestone 문서가 있다")만 있고 *그것을 만드는 명령*(`/plan-workitem`) 명령 박스 부재. 1·4단계는 명령 박스 있는데 plan만 누락. 추가로 bootstrap 후 Living Doc Status를 `draft → ready`로 전환하는 체크 항목 부재.
- **근거**: 
  - 명령 박스 부재 — fork 초심자 동선에서 `/plan-workitem` 실행을 놓침. README Quick Start Step 3에는 있지만 *체크리스트와 빠른 시작 표현 불일치*.
  - Status 전환 체크 부재 — 모든 템플릿이 `## 0. Status: draft`로 박혀 있어 영구 `draft` 상태로 fork 운용될 위험.
- **해결방안**: NEW_PROJECT_CHECKLIST.md `## 2. 작업 구조 준비` 첫 줄에 다음 추가.
  ````markdown
  - [ ] `/plan-workitem [milestone-id]`를 실행해 milestone/feature/task 문서를 분해했다
    ```
    /plan-workitem M1
    ```
  - [ ] bootstrap 후 PROJECT_CHARTER.md / ARCHITECTURE_OVERVIEW.md / M1 / F-001의 `## 0. Status`를 `draft → ready`로 전환했다
  ````

---

## I-11 — WORKFLOW.md "## 4-1 작성 타이밍" + "hotfix 예외" 단락 부재

- **우선순위**: P2
- **현재 문제**: 다음 2 운영 동선이 WORKFLOW.md 본문에 미명시.
  1. TASK_TEMPLATE.md `## 4-1. 변경 예정 파일/경로`는 주석에 *"구현 시점에 채운다"*가 있지만 [WORKFLOW.md](docs/00-meta/WORKFLOW.md) 본문에는 작성 타이밍 미명시 → plan 단계에서 미리 채워야 한다는 오인 가능.
  2. [AGENTS.md:9](AGENTS.md) *"✅ 상위 문서 없이 하위 문서를 먼저 만들지 않는다"*가 강한 규칙인데 hotfix/typo/security patch 같은 예외 미정의.
- **근거**: 
  - `## 4-1` 작성 타이밍 — plan 단계에서 미리 채울 의무 없음에도 *체크리스트 강제*로 해석되면 plan 비용 증가.
  - hotfix 예외 부재 — *"상위 문서 선갱신"*을 모든 케이스에 강제하면 hotfix/typo 시 운영성 하락. AGENTS.md cap 보호 위해 본 예외는 WORKFLOW에 박는 게 정합.
- **해결방안**: WORKFLOW.md에 다음 2 단락 추가.
  1. `## 4. 구현 및 검증` 안에 1줄: *"task `## 4-1. 변경 예정 파일/경로`는 implement 중 채운다 — plan 단계에서 미리 채울 의무 없음."*
  2. 새 단락 `## 문서 선갱신 예외`: *"다음 케이스는 상위 문서 선갱신을 면제한다 — (1) 보안 hotfix (2) 단순 typo/오타 수정 (3) 명시적으로 비목표(charter ## 5)에 박힌 영역의 긴급 패치. 면제 적용 시 finalize 단계에서 IMPROVEMENT_GUIDE에 *상위 문서 후행 갱신 필요* P2 보고."*

---

## I-12 — /stabilize-milestone DISCOVERY ↔ Charter staleness 감지 메커니즘 부재

- **우선순위**: P2
- **현재 문제**: ADR-035로 *DISCOVERY=SSOT, Charter=snapshot*이 결정됐고 [AGENTS.md:43](AGENTS.md)에 1줄 명시. 그러나 PROJECT_CHARTER.md `## 2.1` / `## 3.1` / `## 9`는 DISCOVERY에서 박은 후 *drift 감지·경고 메커니즘이 없음*. 6개월 운영 후 PROJECT_CHARTER가 수동 편집되면 SSOT와 silent divergence.
- **근거**: ADR-035 잔여 모니터링(*"assumption tracker 빈 결과율 — stabilize가 미검증 가정 N건으로 보고"*)이 명시됐지만 *Charter 본문 stale 자체*는 보고 대상 미포함. mid-project pivot 시 DISCOVERY만 갱신하고 Charter는 그대로일 경우 fork 사용자가 *어느 게 진실인가*를 판단 불가.
- **해결방안**: `/stabilize-milestone`에 다음 step 1개 추가 (보고만, 자동 차단 X — validator 책임 경계 정합).
  - DISCOVERY.md mtime이 PROJECT_CHARTER.md mtime보다 최신인지 확인.
  - Assumption Tracker(`## 12`) 미검증 항목 수.
  - PROJECT_CHARTER `## 2.1 페르소나` / `## 3.1 핵심 시나리오` / `## 9 핵심 가정` 섹션 빈도/최종 갱신 시점.
  
  위 3 시그널 중 1개라도 *stale 의심* 판정 시 IMPROVEMENT_GUIDE에 P1 보고: *"DISCOVERY ↔ Charter drift 의심 — `/bootstrap-project --apply` 또는 수동 갱신 권장."*
  
  ADR-035 잔여 모니터링에 본 단락 amend로 흡수.

---

## I-13 — skill / agent 본문과 docs 정책 간 일관성 sweep 부재

- **우선순위**: P1
- **현재 문제**: 본 Improve List는 `docs/`와 README/AGENTS 표면의 drift를 주로 다루지만, 실제 실행 규칙의 상당 부분은 `.claude/skills/<name>/SKILL.md`, `.claude/agents/*.md`, `.agents/skills/<name>/SKILL.md` wrapper에 있다. 현재 분석에서 확인된 drift 후보(예: DESIGN.md 조건부 정책, ARCHITECTURE 7-x 삭제 책임, Codex wrapper coverage, QA schema, DISCOVERY ↔ Charter staleness)가 skill/agent 본문에 반영됐는지 점검하는 항목이 없음.
- **근거**: ADR-005는 정책 본문을 ADR/docs에 두고 agent/skill에는 링크 + 행동 규율만 두는 패턴을 채택한다. 그러나 실제 작업은 skill/agent 본문이 수행하므로 docs만 고치고 skill/agent를 놓치면 *문서상 정책은 갱신됐지만 실행 경로는 예전 규칙을 따르는* drift가 생긴다. 특히 `/bootstrap-stack`, `/bootstrap-design`, `/stabilize-milestone`, `/review-doc`, Codex wrapper는 이번 개선 후보와 직접 연결된다.
- **해결방안**: 구조 변경 항목(I-02~I-12, I-14~I-20) 반영 후 별도 sweep task를 둔다.
  1. `.claude/skills/bootstrap-design/SKILL.md` — DESIGN.md baseline/conditional 정책 반영.
  2. `.claude/skills/bootstrap-stack/SKILL.md` — ARCHITECTURE 7-x 비해당 섹션 삭제 책임 반영.
  3. `.claude/skills/stabilize-milestone/SKILL.md` — QA schema, DISCOVERY ↔ Charter staleness, conditional placeholder 잔존 보고 반영.
  4. `.claude/skills/review-doc/SKILL.md` — ADR amendment visibility, reserved/parked 번호, AGENTS.md cap, 00-meta 파일 수 drift 점검 반영.
  5. `.agents/skills/*/SKILL.md` — README의 Codex fallback 표현과 wrapper coverage가 실제 wrapper 목록과 일치하는지 점검.
  6. `.claude/agents/*.md` — AGENTS.md / ADR-006 / ADR-022 self-check와 중복·충돌하는 규칙이 없는지 점검.

---

## I-14 — 보일러플레이트 ADR vs 프로젝트 ADR이 같은 폴더에 물리적 혼재

- **우선순위**: P1
- **현재 문제**: [docs/90-decisions/](docs/90-decisions/)에 보일러플레이트 ADR(`scope: boilerplate`)과 fork 사용자가 박을 프로젝트 ADR(`scope: project`)이 *같은 폴더*에 누적 예정. [ADR-000](docs/90-decisions/ADR-000-boilerplate-decision-policy.md)이 첫 줄 라벨 + README 섹션 분리로 *논리 구분*은 했지만 *물리적 분리*는 안 됨. 6개월 운영 후 fork 사용자는 보일러플레이트 ADR ~25개 + 자기 ADR 수십 개가 *시각적으로 섞인* 폴더를 마주함.
- **근거**: 
  - ADR-005 SSOT는 *컨텐츠 분리*가 핵심 가치이지만 *물리적 영역 분리*도 같은 원리의 연장. 라벨 수준 분리는 *읽어야 알 수 있는* signal이고 폴더 분리는 *읽지 않고도 보이는* signal.
  - fork 사용자가 *자기 ADR을 어디 박을지*, *어떤 ADR이 supersede 대상인지* 매번 첫 줄 확인 필요 → 발견 비용 누적.
  - ADR-000 D의 supersede 흐름(`Supersedes ADR-NNN (boilerplate)`)은 폴더 분리 후에도 cross-folder 참조로 유지 가능.
- **해결방안**: `docs/90-decisions/`를 다음 2 sub-folder로 분리.
  ```
  docs/90-decisions/
    README.md                # 두 인덱스 허브 (Boilerplate / Project 링크)
    boilerplate/             # fork 후 read-only. ADR-000~099.
      ADR-000-...
      ADR-001-...
      ...
    project/                 # fork 사용자가 자기 ADR 박는 주축. ADR-100부터.
      README.md              # 빈 인덱스 placeholder
  ```
  - 마이그레이션: 25 ADR 파일 이동 + 모든 ADR 본문 내부 cross-link 갱신 + STRUCTURE.md / skill/agent 본문의 ADR 경로 갱신 (약 50~80 surface).
  - ADR-000 amend로 본 결정 박음. README.md(인덱스 허브)에 *"fork 사용자는 `project/`에 자기 ADR을 박는다"* 명시.
  - I-07(reserved/parked 번호 추적), I-08(amendment 가시성) 둘 다 `boilerplate/README.md` 안에서 처리.
  - **선후 관계**: 본 항목은 I-07 / I-08의 인덱스 정책과 *같이 설계*해야 함 — 새 폴더 구조에서 두 정책이 박힐 위치(`boilerplate/README.md`)와 표 컬럼 형식이 본 결정의 직접 영향이라, 별도 PR로 진행하면 두 번 마이그레이션 위험. 권장 적용 순서 2단계 묶음 안에서 설계 → 7단계에서 마이그레이션 실행.

---

## I-15 — agent 이름에 Anthropic 모델 별칭 박힘 (multi-tool 정신 어긋남)

- **우선순위**: P1
- **현재 문제**: [.claude/agents/](.claude/agents/) 6개 agent 중 3개 이름에 모델 별칭 박힘 — `architect-opus`, `builder-sonnet`, `validator-sonnet`. 나머지(`planner`, `reviewer`, `qa`)는 역할 이름만. **일관성 깨짐** + ADR-010 *"multi-tool 호환"* 정신과 직접 충돌.
- **근거**: 다음 3 문제 누적.
  1. **Codex 사용자 의미 혼선**: `$builder-sonnet`은 Codex CLI에서 *어떤 모델*인지 자명하지 않음. `.codex/agents/builder-sonnet.toml`에 매핑이 필요한데 *이름 자체가 거짓말*이 됨.
  2. **모델 갱신 비용**: [ADR-004](docs/90-decisions/ADR-004-model-alias-policy.md)는 *별칭 자동 갱신*을 의도했는데 *이름에 별칭 박힘*은 그 의도와 어긋남. Opus 5.0 도입 시 이름 변경 surface 발생.
  3. **역할 vs 비용 융합**: 이름이 *복수 의미*(역할 + 모델 hint) 운반 → ADR-006 단순성 1순위 위반. frontmatter `model: opus`만으로 같은 효과 달성 가능.
- **해결방안**: 3개 agent rename. 모델 선택은 frontmatter `model:` 필드로만 표기.
  - `architect-opus` → `architect`
  - `builder-sonnet` → `builder`
  - `validator-sonnet` → `validator`
  - 나머지 3개(`planner`/`reviewer`/`qa`) 그대로.
  - 마이그레이션 surface: `.claude/agents/*.md` 3 rename + 모든 SKILL.md `agent:` 필드(약 10 surface) + WORKFLOW.md / AGENT_EXECUTION_STRATEGY.md / ADR-004/006/007/026 본문 참조(약 30 surface) + `.codex/agents/*.toml` 매핑 갱신 + `.agents/skills/*/SKILL.md` wrapper 갱신.
  - ADR-004 또는 ADR-010 amend로 본 결정 박음 ("agent 이름은 역할 중심, 모델은 frontmatter").

---

## I-16 — SIMULATION_RUN.md가 fork 사용자 영역(docs/40-validation/)에 동거

- **우선순위**: P1
- **현재 문제**: [docs/40-validation/SIMULATION_RUN.md](docs/40-validation/SIMULATION_RUN.md)는 ADR-017의 *"보일러플레이트 자체 dogfood 기록"*. fork 사용자의 *프로젝트 산출물이 아님*. 그러나 `docs/40-validation/`에 fork 사용자의 QA_FINDINGS.md / IMPROVEMENT_GUIDE.md와 *동거* → fork 사용자가 *"이건 내 책임 산출물인가?"* 오인. [STRUCTURE.md](docs/00-meta/STRUCTURE.md) 산출물 표에 등록까지 되어 있어 *책임 산출물*로 해석될 위험.
- **근거**: 
  - ADR-017 본문 *"본 보일러플레이트를 개선할 때"*로 한정 명시했지만 *그 한정 정보가 폴더 위치/STRUCTURE에는 반영 안 됨*.
  - fork 사용자가 *새 제품을 만들 때 todo CLI Round 1 / Express API Round 2 기록*을 마주하면 *맥락 오염*.
  - 그러나 *fork 사용자가 보일러플레이트 신뢰성 증명*용으로는 가치 있음 → 삭제 X, 위치 이동 O.
- **해결방안**: `docs/40-validation/SIMULATION_RUN.md`를 보일러플레이트 메타 영역으로 이동.
  - 1순위 권장 위치: `.boilerplate/validation/SIMULATION_RUN.md` (디렉터리 root noise 회피, `docs/` 내부 fork 사용자 시야에서 격리).
  - 2순위 위치: `docs/_boilerplate-meta/SIMULATION_RUN.md` (docs/ 안에 유지하되 _ 접두사로 보조 자료 표시).
  - STRUCTURE.md 산출물 표에서 본 행 제거 + 별도 *"보일러플레이트 메타 산출물"* 표 신설(또는 presence 컬럼(I-04)에서 `boilerplate-only` 값으로 표시).
  - ADR-017 본문에 위치 경로 갱신.
  - 본 항목은 I-14(보일러플레이트 ADR 분리)와 묶어 *"보일러플레이트 영역 분리"* 한 PR로 처리 권장.
  - **`.boilerplate/` 영역 도입 시 1줄 설명 필수**: 새 top-level 디렉터리는 fork 사용자에게 *"이게 뭐지"* 첫 질문을 발생시킴. STRUCTURE.md *"보일러플레이트 메타 산출물"* 표 헤더에 1줄 — *".boilerplate/ = 보일러플레이트 자체 검증·메타 자료. fork 후 read-only. 프로젝트 산출물 아님."* — 추가. README.md / README_ko.md 디렉터리 트리 단락에도 동일 1줄.

---

## I-17 — NEW_PROJECT_CHECKLIST.md 이름이 다른 00-meta 파일과 컨벤션 깨짐

- **우선순위**: P2
- **현재 문제**: [docs/00-meta/NEW_PROJECT_CHECKLIST.md](docs/00-meta/NEW_PROJECT_CHECKLIST.md) — `NEW_` 접두사가 다른 6개 파일(STRUCTURE/WORKFLOW/AGENT_EXECUTION_STRATEGY/GUARDRAILS_STRATEGY/GLOSSARY/STACK_SETUP_PLAN)과 *컨벤션 깨짐*. 다른 파일들은 모두 *개념명*인데 본 파일만 *시점 명명*(`NEW_`).
- **근거**:
  - 시점 명명은 stale 위험 — *"새 프로젝트"*는 fork 직후 한 번이고, 그 후에는 영구히 *"새가 아님"*. 6개월 뒤 fork 사용자가 *"이게 새 프로젝트인가 기존인가"* 의미 혼선.
  - 표면 일관성 약화 — ADR-005 SSOT 패턴 1의 *"같은 사실은 같은 표현"* 원칙과 정합 어긋남.
- **해결방안**: `NEW_PROJECT_CHECKLIST.md` → `PROJECT_START_CHECKLIST.md` rename.
  - "bootstrap"은 lifecycle 단계 이름이라 파일명에 박으면 *해당 skill 호출 시 보는 체크리스트*로 오인 가능 → 회피.
  - "STARTUP_"은 *Startup 회사*와 혼동 가능 → 회피.
  - "PROJECT_START_"가 *프로젝트 시작 시점*을 자연스럽게 의미 + 다른 파일들의 개념명 컨벤션과 정합.
  - 마이그레이션: 1 파일 rename + README.md / README_ko.md / AGENT_EXECUTION_STRATEGY.md / bootstrap-project SKILL.md / bootstrap-stack SKILL.md / WORKFLOW.md 참조 갱신 약 10 surface.
  - I-10(`/plan-workitem` 명령 박스 + Status 전환 체크)과 묶어 한 PR로 처리 자연스러움.

---

## I-18 — STACK_SETUP_PLAN.md가 baseline에 있는 게 모호 (meta vs generated 경계 흐림)

- **우선순위**: P1
- **현재 문제**: [docs/00-meta/STACK_SETUP_PLAN.md](docs/00-meta/STACK_SETUP_PLAN.md)가 baseline에 존재. 그러나 본문은 *외부 의존 부트업 권장 출력 가이드*(보일러플레이트 정책) + *스택 확정 후 채워질 영역*(generated 산출물)이 **혼재**. STRUCTURE.md는 *"`/bootstrap-stack`, `/stack-guard`"* 생성 주체로 등록 — *generated* 어조와 *baseline 존재*가 충돌. I-02(ADR-012 *"6개 파일"* 충돌)와 같은 뿌리 + I-03(DESIGN.md baseline 모순)와 같은 패턴.
- **근거**: 
  - *meta 문서인지 생성 산출물인지 흐림* → AI agent가 *"이 파일은 이미 있으니 생성 skip"*과 *"이 파일은 placeholder니 갱신해야"* 사이에서 매번 판단 필요.
  - 보일러플레이트 정책(예: PostToolUse hook 매뉴얼 등록 절차)은 *영구 reference*인데 generated 산출물(스택별 통합 진입점, CI workflow YAML)은 *프로젝트별*. 두 성격이 같은 파일에 섞이면 fork 사용자의 stack 변경 시 *어디까지 덮어쓸지* 모호.
  - GUARDRAILS_STRATEGY.md의 *원칙*과 STACK_SETUP_PLAN.md의 *프로젝트별 실행 계획*의 경계도 함께 불명확.
- **해결방안**: 다음 2 변경.
  1. 본 파일을 `docs/00-meta/_templates/STACK_SETUP_PLAN_TEMPLATE.md`로 이동. `/bootstrap-stack`이 *template에서 복사*해 `docs/00-meta/STACK_SETUP_PLAN.md`를 *생성*. baseline에는 template만 존재.
  2. 본문 분리:
     - *보일러플레이트 정책 부분*(PostToolUse hook 매뉴얼 절차) → `GUARDRAILS_STRATEGY.md`로 이동 (원칙 SSOT).
     - *프로젝트별 실행 계획*(외부 의존 부트업, 통합 명령, CI workflow YAML) → template에 placeholder로 두고 /bootstrap-stack이 채움.
  - 마이그레이션 surface: 1 파일 이동/rename + GUARDRAILS_STRATEGY.md 단락 추가 + /bootstrap-stack SKILL.md *"template에서 생성"* 흐름 명시 + STRUCTURE.md 산출물 표 갱신.
  - I-02(ADR-012 amend) + I-04(presence 컬럼) + 본 항목을 *"baseline vs generated 경계 정리"* 한 PR로 묶음 권장.

---

## I-19 — AGENT_EXECUTION_STRATEGY.md가 Claude subagent 표현에 종속됨

- **우선순위**: P2
- **현재 문제**: [docs/00-meta/AGENT_EXECUTION_STRATEGY.md](docs/00-meta/AGENT_EXECUTION_STRATEGY.md) 본문에 *"한 메시지에서 Agent 호출 병렬"*, *"isolation: worktree"*, *"bundled `/batch` 호출"* 등 Claude Code 도구별 표현 다수. ADR-010이 *"multi-tool 호환"* + *AGENTS.md 캐노니컬 진입*을 박았는데 본 문서는 *위임 전략의 SSOT*임에도 Claude 종속.
- **근거**:
  - ADR-005 SSOT 패턴 5 *"진입 페이지 도구 중립"*과 *위임 전략 SSOT*는 같은 *도구 중립* 원칙 적용 대상.
  - 파일명 `AGENT_EXECUTION_STRATEGY.md`도 *AGENT*가 Claude Code 용어로 강하게 연상 — Codex CLI에는 *sub-agent* 동등 개념이 있지만 *agent*라는 용어는 도구별 의미 차이 존재.
  - Codex 사용자가 본 문서를 읽으면 *"이건 Claude 전용 문서인가?"* 첫 인상.
- **해결방안**: 다음 2 변경 (rename은 선택).
  1. **본문 일반화** (필수): Claude 도구별 표현을 *역할 기반*으로 일반화.
     - 예: *"한 메시지에서 Agent 호출 병렬"* → *"한 turn에 독립 sub-agent 다중 호출 (Claude: Agent tool, Codex: $-prefixed skill)"*.
     - 예: *"`isolation: worktree`"* → *"격리 git worktree (도구별 지원 표는 ADR-010 매핑 표 참조)"*.
     - 도구별 매핑은 ADR-010 표로 이양 (SSOT 패턴 1).
  2. **파일명 rename** (선택): `AGENT_EXECUTION_STRATEGY.md` → **`DELEGATION_STRATEGY.md`** (1순위 권장). *위임 전략*이 본 문서의 실제 책임이라 가장 정확. `EXECUTION_STRATEGY.md`는 구현/검증 *실행 전략*까지 포함하는 어조라 범위 과대 — 회피.
  - 마이그레이션: 본문 약 5~10 단락 일반화 + (rename 시) README/AGENTS.md / skill 본문 참조 약 10 surface + STRUCTURE.md Canonical Owner 표 행 갱신.
  - I-15(agent 이름 모델 별칭 제거) + 본 항목을 *"multi-tool 정합 강화"* 묶음으로 함께 처리 자연스러움.

---

## I-20 — validator / reviewer / qa 판정 범위 경계가 명시 부재

- **우선순위**: P2
- **현재 문제**: [.claude/agents/](.claude/agents/) 세 agent — `validator-sonnet` / `reviewer` / `qa` — 의 *판정 범위 경계*가 [AGENT_EXECUTION_STRATEGY.md:26-36](docs/00-meta/AGENT_EXECUTION_STRATEGY.md) 위임 트리거 표에 *한 줄*씩만 있고 *세 agent의 차이*가 직접 명문화되지 않음. SIMULATION_RUN Round 2가 validator/reviewer 중복률 ~10~15%로 분리 유지 정당화했지만 *경계 규칙* 자체는 명시 부재.
- **근거**:
  - fork 사용자가 *milestone stabilize 시 qa vs reviewer 어느 쪽 호출인가*를 매번 판단해야 함.
  - 보일러플레이트의 *판정 일관성* 유지하려면 세 agent의 *판정 범위 경계*가 ADR 또는 위임 트리거 표에 명문화 필요.
  - ADR-007 lifecycle 8단계 책임 경계 표는 *skill 단위*. *agent 단위 경계*는 별도 SSOT 필요.
- **해결방안**: AGENT_EXECUTION_STRATEGY.md(또는 I-19로 rename된 DELEGATION_STRATEGY.md) 위임 트리거 표에 다음 3행을 *경계 규칙*으로 명확화.
  - **`validator`**: workitem 단위. *판정 + report* 전용. AC ↔ 테스트 매핑, 문서 범위 정합. **코드/문서 수정 금지** (ADR-007).
  - **`reviewer`**: 코드/문서 단위. *구조적 모순 + 숨은 복잡도 + 정책 drift*. Clean Code 6항목(ADR-006). **수정 권장만, 직접 수정 X**.
  - **`qa`**: milestone / user-flow 단위. *회귀 + 엣지 케이스 + 사용자 위험*. **보고만, Write 권한 없음** (stabilize-milestone이 받아 적음).
  - 세 행을 단순 1줄 설명에서 *판정 단위 / 판정 종류 / 책임 제약* 3 sub-field로 확장.
  - ADR-007 amend로 본 *agent 단위 경계 SSOT* 명시 또는 신규 ADR로 박음.
  - 마이그레이션: AGENT_EXECUTION_STRATEGY.md 표 갱신 + STRUCTURE.md Canonical Owner 표에 *"agent 단위 책임 경계"* 행 추가.

---

