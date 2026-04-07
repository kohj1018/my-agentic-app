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
- 에이전트의 실행 계획은 `docs/30-workitems/plans`에 자동 저장된다.

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

## 단계별 에이전트 위임

각 단계에서의 에이전트 선택과 위임 조건은 [AGENT_EXECUTION_STRATEGY.md](AGENT_EXECUTION_STRATEGY.md)를 참조한다.
