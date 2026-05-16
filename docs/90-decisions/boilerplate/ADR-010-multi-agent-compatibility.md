# ADR-010 다중 에이전트 도구 호환 (AGENTS.md as canonical entry)

> scope: boilerplate

## 상태
accepted

## 배경
이 보일러플레이트는 Claude Code 표면(`CLAUDE.md`, `.claude/`)에만 묶여 있어, 사용자가 Claude Code의 사용량 한도에 걸리거나 다른 사정으로 OpenAI Codex CLI로 전환할 때 동일 워크플로우를 이어가지 못한다.

본 ADR은 **같은 저장소에서 Claude Code와 Codex CLI 양쪽 모두 동일한 워크플로우를 따라 작업할 수 있게** 만든다. 단, 단순성(ADR-006)과 SSOT(ADR-005)을 절대 위반하지 않는다.

## 결정

| ID | 결정 |
|---|---|
| D1 | `AGENTS.md`를 캐노니컬 진입 페이지로 둔다 (프로젝트 루트). |
| D2 | `CLAUDE.md`는 `@AGENTS.md` import + Claude-only 추가지침 섹션만. |
| D3 | `.claude/skills/<name>/SKILL.md`가 워크플로우 본문의 canonical owner. |
| D4 | `.agents/skills/<name>/SKILL.md`는 얇은 wrapper만 — 본문은 D3 위치를 가리킴. |
| D5 | `.codex/config.toml`은 안전 baseline 최소 설정만 (sandbox, approval, secrets 차단, 모델). |
| D6 | `.codex/agents/*.toml` custom subagents는 보류 (Phase 3). |
| D7 | 본 ADR은 ADR-005 SSOT 패턴 1·4를 그대로 적용. 패턴 5("CLAUDE.md = 진입 페이지")는 본 ADR로 표현이 갱신됨 — 캐노니컬 진입 페이지가 `AGENTS.md`로 이동, `CLAUDE.md`는 `@AGENTS.md` import 한 줄. 매핑 표는 본 ADR 본문에 흡수, 별도 `AGENT_TOOL_MATRIX.md` 신설하지 않음. |
| D8 | Codex는 모델 별칭 체계 없음 — 본 ADR이 Codex 모델 ID 추적 책임. ADR-004(별칭 정책)는 본문상 Claude의 별칭만 다루므로 amend 없이 implicit scope를 본 ADR이 명문화. |

## 근거
- 단순성 (ADR-006): 어댑터 표면 최소화.
- SSOT (ADR-005): canonical owner 1곳, 다른 곳은 링크. 정책=ADR 패턴.
- 공식 1차 출처: Claude Code memory 문서가 `@AGENTS.md` 패턴을 명시 권장 ([code.claude.com/docs/en/memory](https://code.claude.com/docs/en/memory)).
- 가역성: AGENTS.md는 Codex의 공식 진입 파일이고 다른 도구와의 호환 가능성도 높아 Codex 사용을 중단해도 변경 대부분이 무해. (다른 도구 호환은 1차 출처가 아닌 기대 효과 — 검증된 사실은 Codex 측만.)
- Cross-platform 안정성: Windows symlink 의존 회피.

## 도구 표면 매핑 (Claude ↔ Codex)

| 영역 | Claude Code | Codex CLI | SSOT |
|---|---|---|---|
| 진입 페이지 | `CLAUDE.md` (← `@AGENTS.md` import) | `AGENTS.md` (자동 로드) | `AGENTS.md` |
| 정책·워크플로우·ADR | `docs/` (그대로) | `docs/` (그대로) | `docs/` |
| 도구 설정 | `.claude/settings.json` | `.codex/config.toml` | 각자 — 단 보안 baseline은 양쪽 동시 |
| 사용자 설정 | `.claude/settings.local.json` | `~/.codex/config.toml` | 각자 |
| Skill 본문 (canonical workflow) | `.claude/skills/<name>/SKILL.md` | `.claude/skills/<name>/SKILL.md` (wrapper가 가리키는 본문) | `.claude/skills/` |
| Skill discovery surface | `/<skill-name>` | `$<skill-name>` 또는 `/skills` (`.agents/skills/<name>/SKILL.md`는 wrapper body) | 도구별 진입 표면, 본문은 위 행 SSOT |
| Custom subagent | `.claude/agents/<name>.md` (markdown) | `.codex/agents/<name>.toml` (TOML) | Phase 3 보류 |
| Hooks | `.claude/settings.json` 안 | `.codex/hooks.json` 또는 `[hooks]` | 본 작업 비범위 |
| 모델 지정 | 별칭(`opus`/`sonnet`) — ADR-004 | 직접 ID(`gpt-5.5`) — 본 ADR 추적 | 도구별 다름 (구조적 차이) |
| Read 차단 (.env, .env.*, secrets/**) | `.claude/settings.json` `permissions.deny` | `.codex/config.toml` `permissions.boilerplate-secure.filesystem` | 양쪽 동시 — 동일 결과를 도구별 표면에 박는다 |

## 결과
- AGENTS.md 신설 — **프로젝트 루트 1곳만** 둔다(Codex가 root→cwd 누적 32 KiB cap이므로 nested AGENTS.md를 만들면 잘릴 위험). nested 지침이 필요하면 docs/ 하위 마크다운으로 분리.
- CLAUDE.md는 `@AGENTS.md` import.
- `.codex/config.toml` 안전 baseline (boilerplate-secure permissions 프로파일 포함, upstream default는 박지 않음).
- `.agents/skills/` wrapper 4개 (Phase 1: `implement/validate/repair/finalize-workitem`).
- ADR-004 본문은 Claude의 별칭만 다루므로 amend 없이 implicit scope를 본 ADR이 명문화 — Codex는 본 ADR이 모델 ID 추적.
- 본 ADR은 ADR-005 SSOT 패턴 1·4를 그대로 적용, 패턴 5는 본 ADR로 표현이 갱신됨 (entry page = `AGENTS.md`).
- **운영 안내 1**: `docs/` 본문(예: `docs/00-meta/WORKFLOW.md`, `DELEGATION_STRATEGY.md`)에 등장하는 `/<skill-name>` 표기는 Claude 슬래시 커맨드다. Codex 사용자는 동일 skill을 `$<skill-name>`으로 읽는다 (Step 6 wrapper와 동일 변환).
- **운영 안내 2**: `.codex/config.toml`의 `.claude/skills/**`는 (fallback 분기 시) Codex에서 read-only로 박힌다 — `.claude/skills/<name>/SKILL.md`는 D3에 의해 canonical SSOT이므로 직접 편집은 Claude Code 측에서 수행한다.

## 후속 작업
- Phase 1.5 (적용됨): plan-workitem, bootstrap-project, bootstrap-stack, stabilize-milestone 4개 wrapper 추가. 근거 — fork 직후 첫 진입 시나리오(charter → architecture → 첫 분해)에서 자연어 호출 대비 wrapper 가성비가 inner-loop와 동등.
- Phase 2 (보류): discover-product, stack-guard, review-doc, boilerplate-context 4개. 호출 빈도 낮아 자연어 호출로 충분.
- Phase 3 `.codex/agents/` TOML (명시 subagent workflow 자주 쓰게 되면).
- Codex 모델 ID 갱신은 본 ADR을 새 ADR로 superseding.
- (Step 0-1에서 `gpt-5.5` 미접근 발견 시) 본 ADR "후속 작업"에 사용된 대체 ID와 갱신 책임자 명시.
- ADR-005 SSOT 패턴 5("CLAUDE.md = 진입 페이지")의 표현을 "entry page (AGENTS.md)"로 갱신하는 후속 ADR 또는 in-place 수정 검토. 본 ADR 채택 후 캐노니컬 진입점이 AGENTS.md로 옮겨가므로 패턴 5의 단어가 어긋난다.
- (Step 0-2에서 스키마 변경 또는 `.` 비상속 발견 시) 적용된 fallback 패턴을 본 ADR "결과" 또는 "후속 작업"에 명시해 추적성 유지.

## Amendment 1 (2026-05-16) — Phase 2.5: stack-guard wrapper 승격

### 결정

기존 Phase 2 보류 4개 skill 중 **stack-guard 1개를 wrapper로 승격**한다. 나머지 3개(discover-product, review-doc, boilerplate-context)는 Phase 2 보류 유지.

### 근거

- 본 ADR Phase 2 보류 판단은 *"호출 빈도 낮음"* 기준이었으나, *영향도* 기준 재평가 결과 stack-guard만 별도 분류 필요.
- [SIMULATION_RUN.md Round 1](../../../.boilerplate/validation/SIMULATION_RUN.md) 직접 관측: 생성된 `validate` 명령이 실 환경 실패 → lifecycle 전체(validate-workitem / finalize-workitem / stabilize-milestone) 신뢰성 영향.
- 빈도는 낮으나 *실패 시 영향이 lifecycle 전체*에 미친다 — 자연어 호출이 *완전 정합 보장이 약한* surface를 inner-loop와 동등 wrapper로 박는다.

### 적용 surface

- `.agents/skills/stack-guard/SKILL.md` wrapper 신설 (기존 wrapper 8종과 동일 패턴 — `name:` 값은 `$` 없이).
- `README.md` / `README_ko.md`에서 `stack-guard`를 *Core workflow Codex wrapper* 목록으로 이동.
- 본 ADR D3·D4·D6은 변경 없음 — wrapper 본문은 여전히 `.claude/skills/<name>/SKILL.md` SSOT를 가리키는 얇은 stub.

### 후속 작업

- 향후 Phase 3에서 나머지 3개(discover-product, review-doc, boilerplate-context) 승격 여부 재평가는 *fork 데이터 회수 후* 결정 (현재 0건).
