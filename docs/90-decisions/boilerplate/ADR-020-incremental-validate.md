# ADR-020 — `validate --changed` (incremental)

> scope: boilerplate

## Status
accepted

## 배경
- [외부실증] Nx affected / Turbo affected 패턴 — monorepo 대용량 codebase에서 full validate 비용이 폭증하면 AI agent가 검증 단계를 skip하거나 timeout 위험.
- 현재 `/finalize-workitem`이 full validate를 실행 — task 단위로는 과도한 비용.

## 결정
`validate --changed` incremental 옵션 도입 패턴:
- **finalize 직전**: `--changed`만 (변경 파일 범위만 lint/typecheck/test). 빠른 회전.
- **stabilize 시**: full validate (누락 차단). 마일스톤 단위 종합 검증.

## 적용 원칙
- 스택이 `--changed` 옵션을 지원하지 않으면 full validate로 fallback.
- `/stack-guard`가 스택 설정 시 `--changed` 지원 여부를 출력에 명시.
- `/finalize-workitem`은 `--changed` 사용을 *권장 텍스트*로 안내 (강제 X).

## 결과
- finalize 속도 향상 → AI agent의 검증 skip 위험 감소.
- stabilize에서 full validate로 누락 차단.

## 후속 작업
없음

## 참고
- ADR-007 (workitem lifecycle — finalize / stabilize 분리)
- ADR-022 (Ratchet Principle — [외부실증] 라벨)
