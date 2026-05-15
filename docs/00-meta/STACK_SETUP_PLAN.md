# Stack Setup Plan

> 모드: Reference (스택 설정 절차 + 자동화 권장)

## 외부 의존 부트업 (DB / Redis / S3 등, ADR-025)
`/bootstrap-stack`이 스택 감지 시 다음 권장 출력:
- Postgres: `docker-compose.yml` 또는 `supabase start` 권장.
- Redis: `docker-compose.yml` 권장.
- S3: localstack 또는 MinIO 권장.

사용자가 채택 시 README에 1단락 + `make dev` / `pnpm dev` 등의 통합 진입점에 wiring.

## 통합 명령 사용법
스택 확정 후 `/stack-guard`가 생성하는 통합 검증 명령:
```
pnpm validate   # 또는 npm run validate / make validate / task validate
```

## CI 권장 출력 (ADR-025)
`/stack-guard`가 다음 형식의 권장 텍스트 출력 (파일 자동 생성 X — 사용자 결정):

```yaml
# .github/workflows/validate.yml (권장)
name: validate
on: [push, pull_request]
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: <stack의 validate 명령>
```

GUARDRAILS_STRATEGY *"OS/셸 종속 hook 강제 X"* 정신 — 권장만.

## PostToolUse hook 매뉴얼 등록 절차
현재 단계에서는 매뉴얼 등록. 추후 자동화 예정(GUARDRAILS_STRATEGY.md 참조).

1. `.claude/settings.local.json` 생성 또는 수정:
```json
{
  "hooks": {
    "PostToolUse": [
      { "matcher": "Edit|Write", "hooks": [{ "type": "command", "command": "pnpm validate" }] }
    ]
  }
}
```

2. 주의: `defaultMode: "acceptEdits"` 환경에서 PostToolUse hook은 매 Edit/Write마다 실행 → 비용 폭증 위험. 로컬에서만 활성화 권장.
