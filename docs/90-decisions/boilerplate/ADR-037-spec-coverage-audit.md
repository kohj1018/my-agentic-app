# ADR-037 — Spec Coverage Self-audit

> scope: boilerplate

## Status
accepted

## 배경
- [외부실증] Osmani self-audit — feature FAC(Feature-level Acceptance Criteria)와 task AC 매핑 누락이 *spec gap*의 핵심 원인.
- ADR-036으로 FEATURE_TEMPLATE에 `## 7 FAC`가 생겼지만 FAC→AC 매핑을 추적하는 메커니즘 부재.
- 매핑 없이 구현하면 feature 수준 품질이 검증되지 않은 채 마일스톤이 종료된다.

## 결정

### 1. validator self-audit 1 step
validate 시 feature `## 7 FAC` 각 항목이 task `## 6 AC`로 매핑됐는지 확인. 매핑 안 된 FAC가 있으면 report에 `Spec Gap: FAC-N → unmapped` 명시 + 미커버 task 추가 권장.

**자동 차단 X (제안만)** — ADR-007 validator 책임 경계 정합.

### 2. plan-workitem 출력 형식에 FAC ↔ AC 매핑표 추가
feature 분해 후 출력에 `FAC-N → T-xxx:AC-N` 형식 매핑표. 미커버 FAC는 `unmapped` 표시.

## 토큰 비용
~1K/task. 6개월 뒤 spec gap 발견 비용보다 작음.

## 결과
- feature 단위 spec coverage가 plan 단계와 validate 단계 양쪽에서 자동 추적됨.
- `Spec Gap` 보고로 미구현 스펙을 조기 발견.

## 후속 작업
없음

## 참고
- ADR-036 (FEATURE_TEMPLATE 12섹션)
- ADR-026 (TASK_TEMPLATE schema)
- ADR-007 (validator 책임 경계 — 판정+권장만)
