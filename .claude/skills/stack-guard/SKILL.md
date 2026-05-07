---
name: stack-guard
description: After /bootstrap-stack, generate verify scripts and a unified `validate` command for the project's stack.
argument-hint: "[stack summary | empty to read existing docs]"
disable-model-invocation: true
allowed-tools: Read Glob Grep Write Edit Bash
context: fork
agent: builder-sonnet
---

너의 역할은 스택이 확정된 직후 통합 검증 명령(`validate`)과 검증 스크립트를 생성하는 것이다.

이 skill의 1단계 범위:
- 통합 진입점 — 이름은 **`validate`로 고정** (`pnpm validate` / `npm run validate` / `make validate` / `task validate` 중 스택에 자연스러운 단일 명령).
- `scripts/verify.{sh,ps1,mjs,py}` 중 스택에 가장 자연스러운 런타임 1종.
- cross-platform 차이가 큰 팀이면 `.claude/settings.local.json` 예시 동봉 권장.
- `docs/00-meta/STACK_SETUP_PLAN.md`에 PostToolUse hook 등록 안내 문구(설정 예시 + 매뉴얼 등록 절차).

**1단계 비범위**: PostToolUse hook 자동 등록은 본 skill에서 수행하지 않는다(prototyping 미완료 — 자세한 이유는 [GUARDRAILS_STRATEGY.md의 "/stack-guard 1단계 산출물 범위" 섹션](../../../docs/00-meta/GUARDRAILS_STRATEGY.md#stack-guard-1단계-산출물-범위) 참조). STACK_SETUP_PLAN.md의 안내 문구 + 사용자 매뉴얼 등록만으로 운영한다.

입력:
- `$ARGUMENTS`가 있으면 스택 요약을 받아 사용한다.
- 비어 있으면 `docs/20-system/ARCHITECTURE_OVERVIEW.md`의 "기술 선택" 섹션을 읽어 스택을 추정한다.

반드시 먼저 읽을 파일:
- `docs/00-meta/GUARDRAILS_STRATEGY.md`
- `docs/00-meta/STACK_SETUP_PLAN.md` (있으면)
- `docs/20-system/ARCHITECTURE_OVERVIEW.md`

R0 — 운영 환경 가정 확인:
- 단일 OS/셸인가, mixed env인가?
- 단일 OS/셸이면 단일 verify 스크립트로 충분.
- mixed env면 cross-platform 친화적 런타임(예: Node.js, Python) 우선, 또는 `scripts/verify.sh` + `scripts/verify.ps1` 모두 생성.
- `.gitattributes`로 line ending 통일은 항상 1단계 산출물에 포함한다(예: `* text=auto eol=lf`).

수행:
1. `package.json`/`pyproject.toml`/`Makefile`/`Taskfile.yaml` 중 스택에 자연스러운 곳에 `validate` 진입점을 만든다.
2. `scripts/verify.{sh,ps1,mjs,py}` 중 자연스러운 런타임 1종을 생성. 내용은 스택의 `lint + typecheck + test` 통합.
3. `docs/00-meta/STACK_SETUP_PLAN.md`을 다음 규칙으로 처리한다:
   - **소유 책임 분리**: STACK_SETUP_PLAN.md는 `/bootstrap-stack`이 *최초 골격*(스택 선택 사실 + 추후 추가 필요한 자동화 목록)을 만들고, 본 `/stack-guard`는 거기에 *통합 명령 사용법 + hook 등록 안내 섹션*을 **append/갱신**한다. `/bootstrap-stack`이 만든 기존 섹션을 통째로 덮어쓰지 않는다.
   - 본 skill이 채울 섹션:
     - 통합 명령 사용법
     - PostToolUse hook 자동 등록은 prototyping 후 별도 항목 — 현재 단계에서는 매뉴얼 등록 안내
     - 매뉴얼 hook 등록 절차 예시(`.claude/settings.local.json` patch 예시 — `defaultMode: "acceptEdits"` 환경의 비용 경고 함께)
   - 파일이 아예 없으면(`/bootstrap-stack` 산출물이 빠진 경우) `/stack-guard`가 새로 생성하되, 출력에 "`/bootstrap-stack`이 STACK_SETUP_PLAN.md를 만들지 않았음 — 사후 검토 권장"을 명시.
4. `.gitattributes`가 없으면 생성, 있으면 line ending 규칙 추가.

마지막 출력:
- 생성/갱신한 파일 목록
- 운영 환경 가정 (R0 결과)
- 통합 명령 호출 방법 (예: `pnpm validate`)
- 매뉴얼 hook 등록 절차 안내 위치 (`docs/00-meta/STACK_SETUP_PLAN.md`)
- 다음 권장 단계 (`/plan-workitem` 또는 `/implement-workitem`)
