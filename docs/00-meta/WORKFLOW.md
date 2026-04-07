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
- 작업을 완료(done)로 전환하기 전에 최소한 다음을 확인한다:
    - 구현 범위가 관련 workitem 문서와 일치한다
    - 관련 검증 항목(테스트 포인트, 검증 방법)이 통과했다
    - 필요한 경우 관련 문서가 함께 갱신되었다

## 문서 운영 방식

| 유형 | 예시 | 운영 방식 |
|------|------|-----------|
| Living Doc | Charter, Architecture, Design System, Workflow | 현재 기준으로 계속 갱신한다. 과거 버전은 Git 이력으로 확인한다. |
| Record | ADR, QA Findings | 기록 보존 우선. 기존 항목을 덮어쓰지 않고 추가 또는 대체한다. |

- ADR은 기존 결정을 뒤집을 때 새 ADR로 대체하는 것을 기본으로 한다.
- QA Findings는 회차 또는 날짜 기준으로 누적 기록한다.
- Improvement Guide는 Living Doc이지만, 완료된 항목은 삭제하지 않고 상태를 갱신한다.

## 문서 상태 전이

```
draft → ready → in-progress → done
                     ↓↑
                  blocked
done → deprecated (필요 시)
```

| 전이 | 최소 조건 |
|------|-----------|
| draft → ready | 필수 섹션이 채워졌고, 자기 검증 또는 리뷰를 거쳤다 |
| ready → in-progress | 실제 구현/작업이 시작됐다 |
| in-progress → blocked | 외부 의존성이나 미결 질문으로 진행 불가 |
| blocked → in-progress | 블로킹 원인이 해소됐다 |
| in-progress → done | 완료 기준을 충족했다 |
| done → deprecated | 대체되었거나 더 이상 유효하지 않다 |

이 규칙은 가이드 수준이며, 훅이나 스크립트로 강제하지 않는다.

## 문서 충돌 해결

문서 간 내용이 모순될 때:
1. 상위 문서가 하위 문서보다 권위가 높다 (charter > architecture > workitems).
2. 상위가 맞다면 하위를 수정하고, 상위가 outdated라면 상위를 먼저 갱신한 뒤 하위를 맞춘다.
3. 의도적 범위/기술 변경이라면 ADR을 남긴다.

## 6. 프로젝트별 자동화 추가
- 프로젝트의 OS/셸/런타임/프레임워크가 정해진 뒤 guardrail을 설계한다.
- shared 보일러플레이트는 구조와 원칙만 제공한다.
- 실제 scripts/hooks/CI는 프로젝트 상황에 맞게 생성한다.
- 관련 원칙은 docs/00-meta/GUARDRAILS_STRATEGY.md를 따른다.

## 단계별 에이전트 위임

각 단계에서의 에이전트 선택과 위임 조건은 [AGENT_EXECUTION_STRATEGY.md](AGENT_EXECUTION_STRATEGY.md)를 참조한다.
