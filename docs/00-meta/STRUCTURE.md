# 산출물 인벤토리

## 목적
이 보일러플레이트가 운영하는 모든 산출물(문서, skill 산출물, agent 산출물 등)의 위치, 생성 주체, 라이프사이클을 단일 표로 관리한다.
새 산출물이 도입되면 이 표에 한 줄을 추가하는 것이 기본 절차다.

## 라이프사이클 정의
- **Living**: 현재 기준으로 계속 갱신한다. 과거 버전은 git 이력으로 확인한다.
- **Reference**: 보조 자료. 갱신 빈도가 낮다.
- **Record**: 기록 보존이 우선. 덮어쓰지 않고 추가 또는 대체한다.
- **ephemeral**: 임시 산출물. 회차마다 덮어쓴다.

## 산출물 표

| 산출물 | 위치 | 생성 주체 | 라이프사이클 |
|--------|------|-----------|--------------|
| project charter | `docs/10-charter/PROJECT_CHARTER.md` | `/bootstrap-project` | Living |
| discovery | `docs/10-charter/DISCOVERY.md` | `/discover-product` | Living |
| architecture overview | `docs/20-system/ARCHITECTURE_OVERVIEW.md` | `/bootstrap-project`, `/bootstrap-stack` | Living |
| design system | `docs/20-system/DESIGN_SYSTEM.md` | `/bootstrap-project` (필요 시 사용자가 수동 보강) | Living |
| milestone | `docs/30-workitems/milestones/M*-*.md` | `/plan-workitem` | Living |
| feature | `docs/30-workitems/features/F-*-*.md` | `/plan-workitem` | Living |
| task | `docs/30-workitems/tasks/T-*-*.md` | `/plan-workitem`, `/implement-workitem` | Living |
| validation report | `docs/40-validation/reports/<task-id>.md` | `/validate-workitem` | ephemeral |
| qa findings | `docs/40-validation/QA_FINDINGS.md` | `/stabilize-milestone` (mile별 누적) | Record |
| improvement guide | `docs/40-validation/IMPROVEMENT_GUIDE.md` | `/stabilize-milestone` | Living |
| ADR | `docs/90-decisions/ADR-*.md` (인덱스: `docs/90-decisions/README.md`) | architect-opus, `/bootstrap-project` 등 | Record |
| stack setup plan | `docs/00-meta/STACK_SETUP_PLAN.md` | `/bootstrap-stack`, `/stack-guard` | Reference |
| verify scripts | `scripts/verify.{sh,ps1,mjs,py}` | `/stack-guard` | Reference |
| AGENTS.md | `./AGENTS.md` | (수동 또는 ADR-010 fork 시) | Living |
| Codex 프로젝트 설정 | `.codex/config.toml` | 수동 | Living |
| Codex skill wrapper | `.agents/skills/<name>/{SKILL.md, agents/openai.yaml}` | 수동 | Reference |

## Canonical Owner 매핑 (SSOT 부록)

각 사실은 단 하나의 canonical 문서에 정의되고, 다른 문서는 한 줄 + 링크만 둔다.

| 사실 | Canonical Owner |
|------|-----------------|
| 문서 계층 정의 (`docs/00-meta`, ...) | `docs/00-meta/TEMPLATE_GUIDE.md`의 "문서 계층" 섹션 |
| 위임 트리거 + 메인 세션 역할 | `docs/00-meta/AGENT_EXECUTION_STRATEGY.md` |
| 상태값 + 전이 규칙 (workitem 일반) | `docs/00-meta/WORKFLOW.md`의 "문서 상태 전이" |
| ADR 전용 상태값 (`proposed`/`accepted`/`superseded`/`deprecated`) | `docs/90-decisions/_ADR_GUIDE.md` |
| 워크플로우 단계 흐름 (한 줄 그림) | `docs/00-meta/WORKFLOW.md` |
| Guardrail 원칙 | `docs/00-meta/GUARDRAILS_STRATEGY.md` |
| 새 프로젝트 시작 절차 (체크리스트) | `docs/00-meta/NEW_PROJECT_CHECKLIST.md` |
| Bootstrap 입력 예시 | `docs/00-meta/BOOTSTRAP_PROMPT_EXAMPLES.md` |
| 모델 별칭 정책 | `docs/90-decisions/ADR-004-model-alias-policy.md` |
| 단순성·YAGNI·Clean Code/Architecture 정책 | `docs/90-decisions/ADR-006-simplicity-and-architecture.md` + `AGENTS.md`(요약) |
| TDD 정책 | `docs/90-decisions/ADR-009-tdd-default.md` + `AGENTS.md`(1줄) |
| 워크아이템 라이프사이클 | `docs/90-decisions/ADR-007-workitem-lifecycle.md` |
| Conventional Commits | `docs/90-decisions/ADR-008-commit-convention.md` |
| 산출물 위치 인벤토리 | 본 문서(`docs/00-meta/STRUCTURE.md`) |
| ADR 인덱스 | `docs/90-decisions/README.md` |
| 도구 어댑터 매핑 (Claude ↔ Codex) | `docs/90-decisions/ADR-010-multi-agent-compatibility.md` |
| AGENTS.md 진입 페이지 정책 (왜 이 파일을 진입점으로 삼는가) | `docs/90-decisions/ADR-010-multi-agent-compatibility.md` |
| 공통 진입 지침 본문 (도구 중립 entry instructions) | `AGENTS.md` |
| 보일러플레이트 직접 지원 스택 범위 | `docs/90-decisions/ADR-031-non-web-out-of-scope.md` |

## 절차

### 새 산출물 도입 시
1. 산출물 표에 한 줄 추가(위치, 생성 주체, 라이프사이클).
2. 라이프사이클이 Record/ephemeral이면 `.gitignore` 처리 여부도 함께 판단.

### 새 정책 도입 시
1. ADR을 만든다 — 정책 본문은 ADR이 SSOT.
2. `docs/90-decisions/README.md`에 한 줄 추가.
3. 관련 agent/skill 본문에는 정책 설명 대신 ADR 링크 + 자기 영역 행동 규율(self-check 등)만 둔다.
4. canonical owner 매핑이 변하면 본 문서의 Canonical Owner 표 갱신.
