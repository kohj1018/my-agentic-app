# ADR-011 — AGENTS.md 100줄 hard cap

> scope: boilerplate

## Status
accepted

## 배경
- [외부실증] Augment의 2,500-repo 분석 — AGENTS.md / CLAUDE.md 등 agent instruction 파일이 길어질수록 agent가 중요 규칙을 누락할 확률이 높아진다. 짧고 명확한 instruction이 agent 행동 일관성을 높인다.
- 보일러플레이트 정책이 AGENTS.md에 직접 쌓이는 경향 → 장기적으로 비대화 위험.

## 결정
- AGENTS.md에 **100줄 hard cap**, **80줄 soft cap**을 적용한다.
- 새 정책은 ADR로 박고, AGENTS.md에는 한 줄 요약 + ADR 링크만 둔다.
- 짧은 본문(1~3줄, cap 안에 충분히 들어가는 자기 완결적 정책 라인)은 AGENTS.md 본문에 직접 박아도 된다 — 단 100줄 cap을 깨지 않을 때만.
- `/review-doc` skill이 AGENTS.md 길이를 점검한다:
  - 100줄 초과 → IMPROVEMENT_GUIDE에 **P0 severity** 보고
  - 80~100줄 → **P1 severity** 보고

## Ratchet 분류
enabling 정책 — Augment [외부실증] 근거. 자동 차단 아님(reviewer 보고 → 사용자 결정).

## 결과
- AGENTS.md가 간결하게 유지되어 agent instruction 파일 전체를 컨텍스트 창에 부담 없이 로드할 수 있다.
- 정책 본문은 ADR에 집중되어 SSOT(ADR-005) 유지.

## 후속 작업
- `/review-doc` 실행 시 자동 점검됨.
- AGENTS.md가 80줄 soft cap에 근접하면 리팩토링 검토.

## 참고
- ADR-022 (Ratchet Principle)
- ADR-005 (SSOT)
