---
name: plan-workitem
description: 상위 설계 문서를 기반으로 milestone, feature, task 단위 문서를 생성하거나 정리할 때 사용한다 (Claude Code plan 모드와 다름 — workitem 분해기).
argument-hint: "[milestone or feature id]"
disable-model-invocation: true
allowed-tools: Read Glob Grep Write Edit
context: fork
agent: planner
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
6. **task 단위 분해 시**: TASK_TEMPLATE의 `## 6. Acceptance Criteria`에 측정 가능한 AC를 최소 1개 이상 채운다(Given-When-Then 또는 명세 형태). AC가 비면 `/implement-workitem`이 RGR 사이클을 시작할 수 없다(정책: [ADR-009](../../../docs/90-decisions/ADR-009-tdd-default.md)).
7. 새 문서를 만들 때는 해당 레벨의 템플릿을 복사해 채운다.

반드시 지킬 원칙:
- 코드를 구현하지 않는다.
- 서로 다른 추상화 레벨을 한 문서에 섞지 않는다(milestone은 큰 목표, feature는 사용자 가치, task는 구현 단위).
- 하위 문서는 상위 문서를 링크한다.
- 확실하지 않은 내용은 가정으로 표시한다.
- 열린 질문이 남으면 문서에 명시한다.

마지막 출력:
- 생성·갱신한 문서 목록(상대 경로)
- 핵심 가정
- 남은 미결정 사항
- 다음 추천 단계(보통 `/implement-workitem [task-id]`)
