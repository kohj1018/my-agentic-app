---
name: implement-workitem
description: Implement one scoped workitem using builder-sonnet, following Red→Green→Refactor TDD cycle.
argument-hint: "[task or feature identifier] [--fast]"
disable-model-invocation: true
allowed-tools: Read Glob Grep Write Edit Bash
context: fork
agent: builder-sonnet
context-pack: minimal
---

너의 역할은 지정된 workitem을 Red → Green → Refactor 3 phase 사이클로 구현하는 것이다.

입력:
- `$ARGUMENTS`에는 task ID 또는 feature ID가 들어온다.
- `--fast` 플래그가 있으면 RGR 사이클을 1회만 돌려 첫 AC만 완료하고 종료한다(prototype용).

반드시 먼저 할 일:
1. 관련 task 문서를 읽는다.
2. 필요하면 상위 feature/milestone/architecture 문서를 읽는다.
3. task 문서의 `## 6. Acceptance Criteria`(AC-1, AC-2 ...)를 회수한다.
4. `## 6-2. TDD opt-out`을 점검한다 — 사유와 follow-up이 모두 있으면 opt-out 모드로 진행, 둘 중 하나만 비어 있으면 형식 위반으로 표시하고 종료(사용자에게 보강 요청).

opt-out 흐름 (사유와 follow-up 모두 채워졌을 때만):
- 테스트 작성을 건너뛴다.
- 출력에 "TDD opt-out 사유: <사유> / Follow-up: <task ID>"를 명시.
- 다른 흐름은 동일.

기본 흐름 — Red → Green → Refactor (각 AC마다 반복):

Red phase 진입 직전, 출력의 첫 단락으로 "이 task에서 어떤 테스트를 어떤 순서로 작성할 것인가"를 1~3문장으로 명시한다(plan 모드 의존 없이 think-before-edit 규율 확보).

**1. Red**
- task의 `## 6. Acceptance Criteria` 항목을 1개 골라 그것을 위반하는 실패 테스트를 작성한다.
- 테스트 이름에 `AC_N` 식별자를 포함하는 것을 권장(예: `test_AC_1_user_can_login`). 강제 아님.
- 테스트 실행 → "원하는 이유로" 실패하는지 확인 후 phase 종료.

**2. Green**
- 그 테스트를 통과시키는 **최소 코드**만 작성한다.
- 다른 AC를 미리 만족시키지 않는다(YAGNI 강화 — ADR-006).
- 테스트 통과 확인 후 phase 종료.

**3. Refactor**
- 단순성 self-check 4항목 + Clean Code 6항목(ADR-006)에 따라 정리한다.
- 외부 행동을 바꾸지 않는다.
- 테스트 통과 유지 확인 후 phase 종료.

위 사이클을 task의 모든 AC가 소진될 때까지 반복.
`--fast`면 첫 AC만 완료하고 종료, 나머지 AC는 후속 호출 권장.

마지막에 task 문서의 `## 4-1. 변경 예정 파일/경로`를 갱신한다(finalize의 add 참조 목록).

마지막 출력:
- 수정 파일 목록
- AC별 진행 상태 (완료/미완료, 예: `AC-1 ✅, AC-2 ✅, AC-3 ❌(다음 호출)`)
- 핵심 변경 사항
- 단순성 self-check 결과 (남은 정리 항목 N건, 있으면 명시)
- 남은 리스크
- 다음 추천 단계 (보통 `/validate-workitem <task-id>`)

정책 근거:
- TDD: [ADR-009-tdd-default.md](../../../docs/90-decisions/ADR-009-tdd-default.md)
- 단순성·Clean Code: [ADR-006-simplicity-and-architecture.md](../../../docs/90-decisions/ADR-006-simplicity-and-architecture.md)

## Context 정책 (ADR-019)
`반드시 먼저 읽을 파일`은 *최소 충분*. 추가 ADR/architecture 섹션은 task 본문에서 발화 시 인용 — 사전 fork-load 금지.
