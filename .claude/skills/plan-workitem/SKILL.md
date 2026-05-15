---
name: plan-workitem
description: 상위 설계 문서를 기반으로 milestone, feature, task 단위 문서를 생성하거나 정리할 때 사용한다 (Claude Code plan 모드와 다름 — workitem 분해기).
argument-hint: "[milestone or feature id]"
disable-model-invocation: true
allowed-tools: Read Glob Grep Write Edit
context: fork
agent: planner
context-pack: minimal
---

너의 역할은 입력으로 받은 milestone/feature/task ID에 대한 workitem 문서를 분해·생성·갱신하는 것이다.

입력:
- `$ARGUMENTS`에는 milestone ID(예: `M1`), feature ID(예: `F-001`), 또는 자연어 분해 요청이 들어온다.

반드시 먼저 읽을 파일:
- `docs/10-charter/PROJECT_CHARTER.md`
- `docs/20-system/ARCHITECTURE_OVERVIEW.md`
- 입력 ID에 해당하는 상위 workitem 문서(있으면)
- `docs/30-workitems/_templates/MILESTONE_TEMPLATE.md`, `FEATURE_TEMPLATE.md`, `TASK_TEMPLATE.md`

반드시 수행할 일:
1. 입력 ID에 해당하는 상위 문서를 읽어 범위와 비범위를 파악한다.
2. 작업을 milestone, feature, task 중 적절한 레벨로 나눈다.
3. 각 문서의 범위와 비범위를 명확히 적는다.
4. 관련 문서 링크를 함께 기록한다.
5. 검증 포인트와 완료 기준을 포함한다.
6. **task 단위 분해 시**: TASK_TEMPLATE의 `## 6. Acceptance Criteria`에 측정 가능한 AC를 최소 1개 이상 채운다. Given-When-Then 형식을 *강력 권장*하며 자세한 점검은 아래 9번 항목과 TASK_TEMPLATE 주석을 참조한다. AC가 비면 `/implement-workitem`이 RGR 사이클을 시작할 수 없다(정책: [ADR-009](../../../docs/90-decisions/ADR-009-tdd-default.md), [ADR-026](../../../docs/90-decisions/ADR-026-plan-workitem-schema.md)).
7. 새 문서를 만들 때는 해당 레벨의 템플릿을 복사해 채운다.
8. **분해 후 sizing self-check** — 다음 3 한계 중 하나라도 초과 시 *추가 분해 권장 텍스트*를 출력에 명시 (자동 차단 X, 사용자 결정):
   - 1 task = 1 RGR 사이클.
   - AC 3개 이하.
   - 변경 예정 파일(TASK_TEMPLATE `## 4-1`) 5개 이하.
   - 초기 scaffolding·auth 같은 task는 5개 파일 초과가 자연스럽다 — 사용자가 분해 거부 결정 가능.
9. **AC 형식 권장 + 금지 verb 점검** — 모든 AC는 Given-When-Then + measurable verb 권장(TASK_TEMPLATE 주석 참조). 강력 금지 verb("works"/"looks good"/"is correct"/"is fine") 사용 시 *재분해 권장 텍스트* 출력. 문맥상 허용 verb("handles"/"supports")는 *무엇을 / 어떻게*가 명시되면 통과.
10. **task 의존성 채움** — TASK_TEMPLATE `## 9. 의존성`을 분해 시 명시. 병렬 가능 task는 비워둔다.

## feature 분해 시 (ADR-036)
feature 분해 시 12섹션 모두 채운다. `## 7 FAC`는 task `## 6 AC`로 분해되며 매핑 누락 시 plan 출력의 "남은 미결정 사항"에 명시.

## --fast 모드
prototype은 `## 3 핵심 시나리오` / `## 7 FAC` / `## 8 NFR` 신설 3섹션을 1줄씩만 채워도 OK ("해당 없음" / "M2 이후 검토").
YAGNI 정합 — Phase 6의 graduation contract *시작 시점 budget*과 동등 정신.

## milestone 생성 시 default (ADR-014)
- `## 5. 완료 기준`은 ADR-014 graduation checklist 5+1 항목 default 사용 (MILESTONE_TEMPLATE 그대로 복사). 사용자가 추가 기준을 협상해 "(선택)" 행을 채운다.
- `## 8. 회고`는 `/stabilize-milestone`이 자동 채움 — plan 단계에서는 비워둔다.

반드시 지킬 원칙:
- 코드를 구현하지 않는다.
- 서로 다른 추상화 레벨을 한 문서에 섞지 않는다(milestone은 큰 목표, feature는 사용자 가치, task는 구현 단위).
- 하위 문서는 상위 문서를 링크한다.
- 확실하지 않은 내용은 가정으로 표시한다.
- 열린 질문이 남으면 문서에 명시한다.

마지막 출력:
- 생성·갱신한 문서 목록(상대 경로)
- 분해 결과 매트릭스 (아래 형식):
  ```
  | Milestone | Feature | Task  | AC 수 | 의존성  |
  |-----------|---------|-------|-------|--------|
  | M1        | F-001   | T-001 | 2     | -      |
  | M1        | F-001   | T-002 | 3     | T-001  |
  ```
- `## 8. FAC ↔ AC 매핑표` (feature 분해 시 — plan 출력에 섹션 헤딩으로 박힘, ADR-037):
  ```
  ## 8. FAC ↔ AC 매핑표
  FAC-1 → T-001:AC-1, T-002:AC-2
  FAC-2 → T-003:AC-1
  FAC-3 → unmapped  ← 미커버 task 필요
  ```
- 핵심 가정
- 남은 미결정 사항
- 다음 추천 단계(보통 `/implement-workitem [task-id]`)

## monorepo·백엔드 sizing 가이드
- **monorepo**: 1 task = 단일 패키지 5 파일 이하 (cross-package 변경은 task 분리).
- **백엔드**: OpenAPI 변경·DB migration·코드 구현은 *별도 task*로 분리. 한 task에 묶지 않는다.
- Phase 4.1의 sizing 휴리스틱(1 RGR / AC 3 / 변경 5)이 monorepo·백엔드에서 깨지는 문제는 *외부실증*(Nx/Turbo 패턴) 기반. [관측됨] 데이터는 Phase 12 Round 2에서 회수.
- **SSOT 노트**: 본 sizing 가이드는 본 skill 본문이 SSOT다. 운영 가이드라 ADR로 박지 않음 — 추적성은 ADR-026 Amendment 1에서 명시.

## 정합성 self-check (분해 직후 1회 실행, ADR-026 amend 1)
- charter `## 5. 비목표` 단락 키워드와 분해된 feature/task를 매칭. 위반 의심 시 출력의 "남은 미결정 사항"에 명시.
- feature 범위가 상위 milestone `## 3. 포함되는 기능`에 매핑되는지 확인. 매핑 실패 시 동일 위치에 명시.

## architect-opus 호출 권장 신호 (감지 시 텍스트 제안만, 자동 호출 금지 — ADR-007)
다음 4 신호 중 하나라도 감지되면 출력 마지막에 `architect-opus 호출 권장: <이유>` 1줄 추가:
1. 새 모듈 디렉터리 생성 (`src/<new>/` 또는 동등 경로).
2. charter `## 7. 제약 조건`에 없는 새 외부 의존 (npm/pip/cargo) 추가.
3. ARCHITECTURE_OVERVIEW.md `## 3-1. 레이어 경계` 변경.
4. "패턴 변경" / "새 boundary" / "도메인 경계" 키워드 등장.

## Context 정책 (ADR-019)
`반드시 먼저 읽을 파일`은 *최소 충분*. 추가 ADR/architecture 섹션은 task 본문에서 발화 시 인용 — 사전 fork-load 금지.
