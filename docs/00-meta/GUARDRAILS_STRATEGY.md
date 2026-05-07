# Guardrails Strategy

## 목적
이 보일러플레이트는 cross-platform 재사용성을 우선한다.
따라서 shared 기본값에는 OS, 셸, 런타임에 강하게 의존하는 자동화를 넣지 않는다.

## 기본 원칙
- shared 설정에는 플랫폼 중립적인 항목만 둔다.
- local 자동화는 `.claude/settings.local.json`에서 활성화한다.
- 프로젝트의 스택이 정해진 뒤 그 스택에 맞는 scripts/hooks/CI를 생성한다.
- 문서 구조와 운영 원칙은 shared로 유지한다.
- 환경 종속적인 guardrail은 optional로 둔다.

## shared 기본값에 포함하는 것
- `CLAUDE.md`
- `.claude/agents`
- `.claude/skills`
- `docs/` 문서 구조
- `.claude/settings.json`의 최소 공통 설정
- 민감 파일 접근 제한

## shared 기본값에 포함하지 않는 것
- PowerShell 전용 hook
- Bash 전용 hook
- Python 런타임 전제를 가진 검증 스크립트
- Node.js 전제를 가진 검증 스크립트
- 특정 프레임워크 lint/test/build hook

## local 자동화 권장 원칙
- 개인 환경에서만 필요한 hook는 `.claude/settings.local.json`에 둔다.
- 실험적인 자동화도 local에 둔다.
- 팀 전체에 강제할 검증은 스택이 확정된 뒤 repo 차원에서 추가한다.

## stack-specific 생성 시점
다음이 정해진 후 생성한다.
- 운영체제 전제
- 셸 전제
- 런타임 전제
- 언어/프레임워크
- package manager
- 테스트 도구
- lint/typecheck 도구

## /stack-guard 1단계 산출물 범위
스택이 확정된 후 사용자가 `/stack-guard`를 발화하면 다음을 생성한다.

**1단계 산출물 (자동 생성)**:
- 통합 진입점 — 이름은 `validate`로 고정 (`pnpm validate` / `npm run validate` / `make validate` / `task validate` 중 스택에 자연스러운 1종).
- `scripts/verify.{sh,ps1,mjs,py}` 중 스택에 자연스러운 런타임 1종.
- `.gitattributes` (line ending 통일).
- `docs/00-meta/STACK_SETUP_PLAN.md`에 PostToolUse hook **매뉴얼** 등록 안내 문구.

**1단계 비범위 (prototyping 후 분리)**:
- PostToolUse hook 자동 등록 — `acceptEdits` 모드에서 매 Edit/Write마다 lint를 자동 실행하면 비용이 폭증할 수 있다(사용자가 수락 프롬프트로 차단할 기회조차 없음). hook 입출력 JSON 스키마와 settings.json patch 양식이 본 시점에 직접 검증되지 않았다. prototyping 단계에서 (1) hook 입출력 스키마와 settings.json patch 양식 (2) `acceptEdits` 모드의 실측 비용 (3) 단일 OS/셸 가정의 자동 감지 신뢰도를 측정한 뒤 별도 항목(`5b. /stack-guard hook 자동 등록`)으로 분리한다.

## 권장 예시
- Next.js + pnpm + Playwright 프로젝트
  - `scripts/verify.ps1` 또는 `scripts/verify.mjs`
  - lint / typecheck / test / e2e hook
- Python + pytest + ruff 프로젝트
  - `scripts/verify.py`
  - format / lint / test hook

## 결정 이유
- 템플릿 자체는 어디서든 clone해서 써야 한다.
- 환경이 다른 팀원에게 동일한 hook를 강제하면 실패 확률이 높다.
- 공통 템플릿은 구조와 원칙을 제공하고,
  실제 자동화는 프로젝트 상황에 맞게 생성하는 편이 유지보수에 유리하다.
