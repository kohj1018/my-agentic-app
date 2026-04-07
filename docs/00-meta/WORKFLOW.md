# 워크플로우

## 1. 범위 정의
- `docs/10-charter/PROJECT_CHARTER.md`에서 문제, 목표, 비목표, 제약을 정리한다.

## 2. 시스템 설계
- `docs/20-system/ARCHITECTURE_OVERVIEW.md`에서 시스템 구조를 정리한다.
- 필요 시 `docs/20-system/DESIGN_SYSTEM.md`를 보강한다.

## 3. 작업 단위 분해
- 마일스톤 단위 목표를 `docs/30-workitems/milestones`에 만든다.
- 기능 단위 문서를 `docs/30-workitems/features`에 만든다.
- 실제 구현 단위 문서를 `docs/30-workitems/tasks`에 만든다.

## 4. 구현 및 검증
- 구현 후 `docs/40-validation/QA_FINDINGS.md`에 문제를 정리한다.
- `docs/40-validation/IMPROVEMENT_GUIDE.md`에 수정 우선순위를 반영한다.

## 5. 의사결정 기록
- 중요한 기술적 선택은 `docs/90-decisions`에 ADR로 남긴다.

## 기본 원칙
- 상위 문서 없이 하위 문서만 먼저 만들지 않는다.
- 흩어진 메모보다 정해진 위치의 문서를 갱신한다.
- 애매한 사항은 문서에 가정과 열린 질문으로 남긴다.

## 6. 프로젝트별 자동화 추가
- 프로젝트의 OS/셸/런타임/프레임워크가 정해진 뒤 guardrail을 설계한다.
- shared 보일러플레이트는 구조와 원칙만 제공한다.
- 실제 scripts/hooks/CI는 프로젝트 상황에 맞게 생성한다.
- 관련 원칙은 docs/00-meta/GUARDRAILS_STRATEGY.md를 따른다.

## 단계별 추천 에이전트

| 단계 | 주요 작업 | 추천 에이전트/명령 |
|------|-----------|-------------------|
| 프로젝트 초기화 | charter + architecture 초안 생성 | `/bootstrap-project` |
| 중요한 기획/설계 | 제품 방향, 아키텍처, 큰 tradeoff | `architect-opus` |
| 일반 문서 정리 | 요구사항 정제, workitem 분해 | `planner` |
| 구현 | task 단위 코드 변경, 테스트 추가 | `builder-sonnet` |
| 구현 검증 | 문서-구현 일치, 범위 점검 | `validator-sonnet` |
| 비판 리뷰 | 문서/코드의 누락, 모순 검토 | `reviewer` |
| QA | 엣지 케이스, 회귀 위험, 사용자 영향 점검 | `qa` |
| 큰 병렬 변경 | 독립 task 병렬 처리 | `/batch` + worktree |
