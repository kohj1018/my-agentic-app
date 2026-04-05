# Bootstrap Stack Output Checklist

## 필수 갱신 문서
- `docs/20-system/ARCHITECTURE_OVERVIEW.md`
- `docs/90-decisions/ADR-003-stack-selection.md`

## 선택 생성 문서
- `docs/00-meta/STACK_SETUP_PLAN.md`

## 출력 원칙
- shared 기본값에는 OS/셸 종속 hook를 강제로 넣지 않는다.
- 필요 scripts/hooks/CI를 문서로 정리한다.
- 실제 생성이 필요하다면 해당 스택에서 자연스러운 런타임을 기준으로 제안한다.
- mixed environment라면 local settings 전략을 우선 고려한다.
