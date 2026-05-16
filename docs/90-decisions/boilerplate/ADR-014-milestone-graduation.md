# ADR-014 — Milestone Graduation Contract

> scope: boilerplate

## Status
accepted

## 배경
- [관측됨] `MILESTONE_TEMPLATE.md`의 `## 5. 완료 기준`이 빈 placeholder → milestone 졸업 판정 모호. stabilize 실행 시 "완료인가"를 매번 사람이 주관적으로 판단.
- [외부실증] Atlassian multi-level DoD (story/sprint/release) — sprint 단위의 외부 검증 가능한 완료 기준이 "릴리즈 품질"과 "구현 완료"를 분리한다.
- [외부실증] Anthropic 3-agent harness sprint contract — planner·builder·evaluator 분리 패턴에서 sprint 단위 완료 기준이 evaluator의 판정 근거.

## 결정

### 1. Graduation checklist 5+1 항목
`## 5. 완료 기준`을 다음 5개 필수 + 1개 선택으로 교체:
1. 모든 task status: done
2. 통합 validate Pass
3. E2E Pass (스택에 정의된 경우)
4. AC 매핑 100% (validation report 기준)
5. P0 severity finding 0건 (QA_FINDINGS의 본 마일스톤 헤더 기준)
6. (선택) 본 마일스톤 한정 추가 기준

### 2. 회고 4 항목
`## 8. 회고` 신설 (`## 7. 열린 질문` 아래):
- 목표 달성도: 정량/정성 1줄
- scope creep 사례: 있으면 1줄, 없으면 "없음"
- 비목표(charter ## 5) 위반 사례: 있으면 1줄
- 핵심 학습 3개 이내

### 3. Graduation pre-check + `--dry-run`
`/stabilize-milestone`의 단계 1.5에 graduation pre-check 신설:
- `## 5. 완료 기준` 각 항목 자동 체크.
- 미충족 발견 시 `졸업 가능: NO` + 미충족 목록 출력 + 조기 종료 옵션.
- `--dry-run` 플래그 = pre-check만 돌리고 종료 (P0 검증 도구).

## 비결정 (영구 No)
- Release-level DoD — stabilize 출력에 자연 흡수 (carry-over 0건 + ADR 후보 0건 = release-ready).
- Fowler 4-quadrant test classification — 보일러플레이트가 정확도 보장 불가, YAGNI.
- METRICS.md — 메트릭 정의는 프로젝트별 결정, boilerplate 강제 불가.
- `--apply-carryover` 자동 이월 — 사용자 명시적 결정 필요 (ADR-007 책임 경계 정합).
- architect auto-escalation 신호 — 트리거 기준 정의 불가 (프로젝트별).

## 결과
- `/stabilize-milestone --dry-run [M1]`으로 전체 QA 없이 졸업 가능 여부를 빠르게 확인.
- 회고가 milestone 문서에 누적되어 다음 마일스톤 계획에 재사용.

## 잔여 모니터링
graduation pre-check 미통과 사유 패턴 — 3회 이상 반복 시 lifecycle 단계 결함 신호 → ADR 후보.

## 참고
- ADR-007 (workitem lifecycle)
- ADR-009 (TDD default)
- ADR-022 (Ratchet Principle — [관측됨] 라벨)
