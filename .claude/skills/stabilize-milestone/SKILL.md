---
name: stabilize-milestone
description: Stabilize a milestone — run E2E + regression + refactoring/ADR review. No code changes, no commits.
argument-hint: "[milestone id]"
disable-model-invocation: true
allowed-tools: Read Glob Grep Write Edit Bash Agent
---

이 skill은 **코드 수정·커밋·workitem status 변경을 하지 않는다.**
검증 결과를 `docs/40-validation/QA_FINDINGS.md`와 `docs/40-validation/IMPROVEMENT_GUIDE.md`에 누적 기록하는 문서 갱신은 정상 책임이다 — 그 외 변경은 금지한다.
후속 작업이 필요하면 `/repair-workitem` 또는 새 task로 텍스트 제안만 출력한다.

입력:
- `$ARGUMENTS`에는 milestone ID(예: `M1`)가 들어온다.

수행:
1. milestone 문서를 읽고 포함된 feature/task 목록을 회수한다.
2. 각 task의 status를 점검 — `done`이 아닌 항목이 있으면 명단을 출력하고 종료(완료를 강제하지 않음).
3. 통합 `validate` 명령을 실행한다 + (있으면) E2E 명령을 실행한다.
4. **qa agent에 회귀·엣지케이스 점검 위임** — qa는 보고만 한다(qa.md의 tools에 Write 없음). 반환된 보고를 본 skill이 받아 `docs/40-validation/QA_FINDINGS.md`에 누적 기록한다.
5. **reviewer agent에 리팩토링 후보·아키텍처 부채 점검 위임** — reviewer 입력에 Clean Code 6항목 체크리스트(ADR-006)를 명시 전달한다. reviewer도 보고만 한다. 반환된 보고를 본 skill이 받아 `docs/40-validation/IMPROVEMENT_GUIDE.md`에 정리.
   - reviewer 결과에 구조 변경이 필요해 보이면 메인 세션에 architect-opus 추가 호출을 텍스트로 제안.
6. 미흡한 ADR 후보 제안 — 마일스톤 중에 내려진 결정인데 ADR이 없는 것을 식별. ADR 후보 기준에 "layer 경계·의존성 규칙 변경"도 포함(ADR-006 정책).
   - layer 경계·의존성 규칙 변경(ARCHITECTURE_OVERVIEW의 ## 3-1)이 마일스톤 중에 발생했으면 ADR 후보로 표시한다(정책: ADR-006).
7. ARCHITECTURE_OVERVIEW의 `## 3-1. 레이어 경계 + 의존성 규칙` 섹션이 비어 있고 모듈 수가 3개 이상이면 채울 것을 권장 출력한다(정책: ADR-006).
8. 최종 출력:
   - 통합 `validate` 결과 + E2E 결과 (있으면)
   - P0 / P1 / P2 후속 작업
   - QA_FINDINGS / IMPROVEMENT_GUIDE 갱신 위치
   - 다음 마일스톤으로 넘기는 항목
   - architect-opus 호출 권장 (있으면)

책임 경계:
- 코드 수정·커밋·workitem status 변경 금지.
- 누적 문서 갱신만 허용 (`QA_FINDINGS.md`, `IMPROVEMENT_GUIDE.md`).

E2E 명령이 없는 스택은 3단계에서 통합 `validate`만 돌리고 E2E는 skip한다(출력에 명시).
