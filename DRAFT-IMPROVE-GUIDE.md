<!--
이 문서는 "다중 에이전트 호환(Claude Code + Codex)" 개선을 진행하기 위한 작업 초안 가이드다.
실제 구현이 끝나면 이 파일은 삭제하고, 결정/근거는 docs/90-decisions/ADR-010-multi-agent-compatibility.md에,
변경된 산출물은 docs/00-meta/STRUCTURE.md에 흡수된다.
-->

# 보일러플레이트 다중 에이전트 호환 개선 가이드 (DRAFT)

## 0. 메타정보

| 항목 | 값 |
|---|---|
| 상태 | draft |
| 작성일 | 2026-05-07 |
| 폐기 시점 | ADR-010 accepted + Phase 1 산출물 머지 시 |
| 관련 ADR | ADR-004 (모델 별칭), ADR-005 (SSOT), ADR-006 (단순성), ADR-010 (신설 예정) |
| 영향받는 문서 | `CLAUDE.md`, `docs/00-meta/STRUCTURE.md`, `docs/00-meta/TEMPLATE_GUIDE.md`, `docs/00-meta/GUARDRAILS_STRATEGY.md`, `README.md`, `README_ko.md` |

---

## 1. 배경과 목적

### 1-1. 문제
현재 보일러플레이트는 Claude Code 표면(`CLAUDE.md`, `.claude/settings.json`, `.claude/agents/`, `.claude/skills/`)에만 묶여 있다. 사용자가 Claude Code의 사용량 한도에 걸리거나 다른 사정으로 OpenAI Codex CLI로 전환해야 할 때 동일 워크플로우를 이어가지 못한다.

### 1-2. 목적
**같은 저장소에서 Claude Code와 Codex CLI 양쪽 모두 동일한 워크플로우를 따라 작업할 수 있게 만든다.** 단, 다음 두 가치를 절대 위반하지 않는다.
- **단순성** (사용자 명시 + ADR-006) — 추가 표면 최소화, 실제로 쓸 때만 추가.
- **유지보수성** (사용자 명시 + ADR-005) — SSOT 패턴 유지, 정책=ADR, 정의 1곳·다른 곳은 링크.

### 1-3. 부가 가치 기준 (자체 정의)
- **가역성** — Codex 사용을 중단해도 변경의 대부분이 무해(다른 도구에도 유효).
- **Cross-platform 안정성** — Windows symlink 의존 회피.
- **공식 1차 출처 우선** — 모든 결정은 `docs.anthropic.com` / `code.claude.com` / `developers.openai.com` 1차 문서로 검증.

---

## 2. 검증된 사실 (공식 1차 출처 기준)

> 본 가이드의 모든 결정은 아래 사실에 기반한다. 추측이나 third-party 블로그 인용은 사용하지 않는다.

### 2-1. Claude Code 측 사실
| 사실 | 출처 |
|---|---|
| Claude Code는 `CLAUDE.md`만 읽고 `AGENTS.md`는 **읽지 않는다** | [code.claude.com/docs/en/memory](https://code.claude.com/docs/en/memory) — *"Claude Code reads `CLAUDE.md`, not `AGENTS.md`."* |
| AGENTS.md를 공유하려면 `CLAUDE.md`에서 `@AGENTS.md` import 패턴이 공식 권장 | 같은 페이지에 권장 예시로 박혀 있음 |
| `@path/to/file` 문법은 공식 — 상대/절대 경로, 5단계 재귀 가능, launch 시 인라인 로드 | 같은 페이지 |
| `CLAUDE.md`는 200줄 이하 권장 | 같은 페이지 |
| `.claude/` 위치도 유효 (`./.claude/CLAUDE.md`) | 같은 페이지 |

### 2-2. Codex CLI 측 사실
| 사실 | 출처 |
|---|---|
| `AGENTS.md`는 글로벌(`~/.codex/`), 프로젝트 루트, 모든 하위 디렉터리에서 자동 로드. `AGENTS.override.md`가 우선 | [developers.openai.com/codex/guides/agents-md](https://developers.openai.com/codex/guides/agents-md) |
| `project_doc_max_bytes` 기본값 32 KiB | 같은 페이지 |
| 프로젝트 설정은 `.codex/config.toml`, 사용자 설정은 `~/.codex/config.toml` | [developers.openai.com/codex/config-basic](https://developers.openai.com/codex/config-basic), [config-reference](https://developers.openai.com/codex/config-reference) |
| 모델 필드 공식 예시는 `gpt-5.5` | [developers.openai.com/codex/config-reference](https://developers.openai.com/codex/config-reference) — *"Model to use (e.g., `gpt-5.5`)"* |
| Skills 위치: 프로젝트는 `.agents/skills/`, 사용자는 `~/.agents/skills/` | [developers.openai.com/codex/skills](https://developers.openai.com/codex/skills) |
| Skill 호출 공식 문법: **`$skill-name`** 또는 `/skills` (점·괄호 변형은 공식 X) | 같은 페이지 — *"In CLI/IDE, run `/skills` or type `$` to mention a skill"* |
| Custom subagents: `.codex/agents/<name>.toml`. 명시 호출 시에만 spawn | [developers.openai.com/codex/subagents](https://developers.openai.com/codex/subagents) |
| `[agents]` 기본값: `max_threads=6`, `max_depth=1` | 같은 페이지 |
| Hooks: `[features] codex_hooks = true` 활성화 필요, `hooks.json` 또는 `[hooks]` 인라인 | [developers.openai.com/codex/hooks](https://developers.openai.com/codex/hooks) |
| Custom Prompts(`~/.codex/prompts/`)는 deprecated — Skills로 대체 권장 | [developers.openai.com/codex/custom-prompts](https://developers.openai.com/codex/custom-prompts) |
| Codex에는 Claude의 `opus`/`sonnet` 같은 모델 별칭 체계 없음 | config-reference 문서 분석 결과 |

### 2-3. 검증된 permissions 프로파일 스키마 (config-advanced 직접 확인)

`.codex/config.toml`의 permissions 프로파일은 **`[permissions.<name>.filesystem]` 중첩 테이블 헤더 + `":project_roots"` 키에 inline-table 값** 형태다. 차단은 deny 배열이 아니라 glob 패턴 키 → 모드 매핑(`"none"` / `"read"` / `"write"`).

```toml
default_permissions = "<name>"  # 활성화 필수

[permissions.<name>.filesystem]
":project_roots" = { "." = "write", "**/.env" = "none", "**/secrets/**" = "none" }
glob_scan_max_depth = 3
```

- 빌트인 프로파일은 leading colon (`:read-only`, `:workspace`, `:danger-no-sandbox`), 커스텀 이름은 colon 없음.
- 더 구체적인 glob이 우선(통상 glob 매칭 규칙) — `.` = "write"가 깔려도 `**/.env` = "none"이 우선.
- 출처: [config-advanced](https://developers.openai.com/codex/config-advanced) (config-basic은 빌트인 3종만 보여주고 상세 스키마는 advanced로 위임).

### 2-4. 잔여 미검증 항목
- `gpt-5.5` 사용 계정에서 실제 접근 가능 여부 — 한도/플랜별로 다를 수 있음. 적용 전 1회 확인.

---

## 3. 결정 사항 (Decisions)

> 모든 결정은 ADR-010에 정식 박는다. 본 섹션은 ADR-010 작성 입력.

| ID | 결정 | 근거 |
|---|---|---|
| D1 | `AGENTS.md`를 캐노니컬 진입 페이지로 둔다 (프로젝트 루트) | Codex 자동 로드 + Claude도 import로 동일 내용 사용 가능 |
| D2 | `CLAUDE.md`는 `@AGENTS.md` import + Claude-only 추가지침 섹션만 | 공식 권장 패턴, AGENTS.md fallback 의존 회피 |
| D3 | `.claude/skills/<name>/SKILL.md`가 workflow 본문의 **canonical owner** | 기존 자산 보존, drift 방지 |
| D4 | `.agents/skills/<name>/SKILL.md`는 **얇은 wrapper만** — 본문은 D3 위치를 가리킴 | Codex `$skill-name` UX 확보, 본문 중복 제거 |
| D5 | `.codex/config.toml`은 **안전 baseline 최소 설정만** (sandbox, approval, secrets 차단, project_doc_max_bytes, model) | 기능 포팅이 아닌 보안 격차 방지 |
| D6 | `.codex/agents/*.toml` custom subagents는 **Phase 3 보류** | Codex는 worker/explorer/default 빌트인이 있고, custom은 명시 subagent workflow 자주 쓸 때만 가치 큼 |
| D7 | 본 ADR은 ADR-005 SSOT 패턴 1·4·5를 그대로 적용 — 정책=ADR, 매핑 표는 ADR-010 본문에 흡수, 별도 `AGENT_TOOL_MATRIX.md` 신설 X | ADR-005의 자명한 적용 |
| D8 | Codex는 모델 별칭 체계 없음 — ADR-010이 모델 ID 추적 책임. ADR-004(별칭 정책)는 본문상 Claude의 별칭만 다루므로 amend 없이 implicit scope를 ADR-010이 명문화 | 검증된 사실 2-2, ADR-004 본문 분석 |

---

## 4. 비범위 (Out of Scope, 명시)

> 다음 항목은 이번 개선에서 **하지 않는다**. 추후 별도 작업으로 분리.

- [ ] `.claude/skills/` 12개를 모두 `.agents/skills/`로 wrapper 만들기. Phase 1은 inner-loop 4개만, Phase 2는 추가 workflow 6개, 유틸리티 2개(`review-doc`, `boilerplate-context`)는 보류. 한 번에 X.
- [ ] `.claude/agents/` 6개를 모두 `.codex/agents/*.toml`로 1:1 미러링 → SSOT 위반, drift 비용 큼.
- [ ] 별도 `docs/00-meta/AGENT_TOOL_MATRIX.md` 문서 신설 → ADR-010 본문에 흡수.
- [ ] Codex hooks 도입 (`hooks.json` 또는 `[features] codex_hooks`) → 본 보일러플레이트 GUARDRAILS_STRATEGY 원칙(스택 확정 후 도입)에 따라 보류.
- [ ] CI / GitHub Action 단의 Codex 통합 → 별 워크아이템.
- [ ] `~/.codex/AGENTS.md` 같은 사용자 글로벌 설정 강제 → 사용자 환경 결정.
- [ ] Plan 모드(Claude) ↔ `/plan`(Codex) 동등 매핑 — 동작 메커니즘이 다르므로 워크플로우 문서 수준에서만 안내.
- [ ] Cursor / Gemini CLI / OpenCode 등 추가 도구 호환 — AGENTS.md 도입으로 자연 확장되지만 본 작업의 책임은 아님.

---

## 5. Phase별 작업 항목

### Phase 1 — 필수 baseline (예상 1~2시간)

| ID | 작업 | 산출물 | 의존 |
|---|---|---|---|
| P1-1 | `AGENTS.md` 신설 — 현 `CLAUDE.md` 본문(목적·핵심 규율·단순성·TDD·docs 링크)을 옮김 | `AGENTS.md` | — |
| P1-2 | `CLAUDE.md`를 `@AGENTS.md` import 패턴으로 교체 | `CLAUDE.md` | P1-1 |
| P1-3 | ADR-010 작성 (배경·결정·근거·결과·후속작업 + 도구 매핑 표 본문 안 포함) | `docs/90-decisions/ADR-010-multi-agent-compatibility.md` | — (병렬 가능) |
| P1-4 | ADR 인덱스 갱신 + STRUCTURE.md Canonical Owner 표에 한 줄 추가 + 산출물 인벤토리에 `.agents/skills/`, `.codex/config.toml` 등록 | `docs/90-decisions/README.md`, `docs/00-meta/STRUCTURE.md` | P1-3 |
| P1-5 | `.codex/config.toml` 작성 — model, sandbox, approval, **및 §11 사전 점검을 통과한 permissions(`.env`, `.env.*`, `secrets/**` 차단) 포함**. 사전 검증은 §11이 단일 진입점 | `.codex/config.toml` | §11 점검 |
| P1-6 | `.agents/skills/` wrapper 4개 신설 (inner-loop). 각 폴더에 `SKILL.md` + `agents/openai.yaml` 둘 다 (implicit invocation 차단) | `.agents/skills/{implement,validate,repair,finalize}-workitem/{SKILL.md, agents/openai.yaml}` (총 8개 파일) | — |
| P1-7 | README.md / README_ko.md에 "Claude Code + Codex 호환" 5줄 추가 | `README.md`, `README_ko.md` | P1-1, P1-3 |

### Phase 2 — 추가 wrapper (실제로 Codex 사용 빈도가 자리잡으면, 사용 빈도에 따라 선별)

| ID | 작업 | 산출물 |
|---|---|---|
| P2-1 | workflow wrapper 6개 추가 | `.agents/skills/{discover-product,bootstrap-project,bootstrap-stack,stack-guard,plan-workitem,stabilize-milestone}/SKILL.md` |
| P2-2 (선택) | 유틸리티 wrapper 2개 — 실제 필요해진 뒤에만 | `.agents/skills/{boilerplate-context,review-doc}/SKILL.md` |

### Phase 3 — Custom subagents (정말 명시 subagent workflow를 자주 쓰게 된 뒤에만)

| ID | 작업 | 산출물 |
|---|---|---|
| P3-1 | 최소 3개 custom agent 포팅 | `.codex/agents/{builder,validator,reviewer}.toml` |
| P3-2 | 나머지 3개 (필요 시) | `.codex/agents/{architect,planner,qa}.toml` |

---

## 6. 각 산출물 상세 가이드

### 6-1. `AGENTS.md` (P1-1)

**위치**: 프로젝트 루트.
**길이**: Codex `project_doc_max_bytes` 기본 32 KiB 내, Claude 권장 200줄 이하 — 둘 중 더 엄격한 200줄 기준 따른다.

**본문 가이드 (현 `CLAUDE.md` 본문을 도구 중립적으로 재작성)**:
- `# 프로젝트 지침`
- `## 목적` — "Claude Code 기반의 ..." 같은 도구 종속 표현은 **재작성 필수**. 예: "Claude Code 기반의 재사용 가능한 문서 중심 개발 보일러플레이트다" → "AI coding agent 기반의 재사용 가능한 문서 중심 개발 보일러플레이트다" 또는 "여러 AI coding agent (Claude Code, OpenAI Codex 등)에서 동일하게 동작하는 문서 중심 개발 보일러플레이트다"
- `## 핵심 행동 규율` (5~7개 항목)
- `## 단순성·YAGNI` (ADR-006 요약 + 링크)
- `## TDD 기본` (ADR-009 요약 + 링크)
- `## 깊은 운영 원칙은 다음 문서를 따른다` (총 6개 링크: `docs/00-meta/*` 5종 + `docs/90-decisions/README.md` ADR 인덱스 1종)
- 그 외 단순성·TDD 단락의 ADR 본문 인라인 링크 2개(ADR-006, ADR-009) 포함 — AGENTS.md 전체에서 docs/ 링크 총 8개

**주의**:
- **HTML 주석 블록을 길게 두지 않는다** — Claude Code는 자동 스트립하지만 Codex가 같은 동작을 한다는 공식 보장이 없다. 사람용 메모가 필요하면 별도 문서로 분리.
- 도구별 분기("Claude는 X, Codex는 Y") 같은 내용은 절대 박지 않는다 — 양쪽 공용 지침만. 도구별 차이는 ADR-010에 둔다.
- "Claude Code", "Codex", "Cursor" 등 특정 도구명을 명시할 일이 있으면 일반화하거나 ADR-010에 위임한다.

### 6-2. `CLAUDE.md` (P1-2)

**전체 본문 (Claude-only 지침이 없으면 한 줄)**:
```markdown
@AGENTS.md
```

**Claude-only 추가 지침이 생기면 그때 추가**:
```markdown
@AGENTS.md

## Claude Code

(여기에 Claude Code에서만 적용할 지침)
```

**근거**: [code.claude.com/docs/en/memory](https://code.claude.com/docs/en/memory)의 권장 패턴. `@AGENTS.md`는 launch 시 인라인 로드되므로 모델 재량에 의존하지 않는다. 빈 placeholder 섹션은 두지 않는다 — 필요해질 때 추가.

### 6-3. `docs/90-decisions/ADR-010-multi-agent-compatibility.md` (P1-3)

**상태**: accepted

**섹션 가이드**:

```markdown
# ADR-010 다중 에이전트 도구 호환 (AGENTS.md as canonical entry)

## 상태
accepted

## 배경
(섹션 1 "배경과 목적" 요약, 1~2 단락)

## 결정
(섹션 3 "결정사항" 표 D1~D8 그대로 옮김)

## 근거
- 단순성 (ADR-006): 어댑터 표면 최소화
- SSOT (ADR-005): canonical owner 1곳, 다른 곳은 링크
- 공식 1차 출처: Claude Code memory 문서가 `@AGENTS.md` 패턴을 명시 권장
- 가역성: AGENTS.md는 Cursor/Gemini/OpenCode에도 유효

## 도구 표면 매핑 (Claude ↔ Codex)
(섹션 7 "표면 매핑 표" 그대로 옮김)

## 결과
- AGENTS.md 신설 — **프로젝트 루트 1곳만** 둔다(Codex가 root→cwd 누적 32 KiB cap이므로 nested AGENTS.md를 만들면 잘릴 위험). nested 지침이 필요하면 docs/ 하위 마크다운으로 분리.
- CLAUDE.md는 @AGENTS.md import
- .codex/config.toml 안전 baseline (boilerplate-secure permissions 프로파일 포함, upstream default는 박지 않음)
- .agents/skills/ wrapper 4개 (P1)
- ADR-004 본문은 Claude의 별칭만 다루므로 amend 없이 implicit scope를 본 ADR이 명문화 — Codex는 본 ADR이 모델 ID 추적
- 본 ADR은 ADR-005 SSOT 패턴 1·4·5를 그대로 적용

## 후속 작업
- Phase 2 workflow wrapper 6개 (Codex 사용 빈도 자리잡으면, 사용 빈도에 따라 선별), 유틸리티 2개는 필요 시
- Phase 3 .codex/agents/ TOML (명시 subagent workflow 자주 쓰게 되면)
- Codex 모델 ID 갱신은 본 ADR을 새 ADR로 superseding
```

### 6-4. `docs/00-meta/STRUCTURE.md` 갱신 (P1-4)

**산출물 표에 추가할 행**:
```
| AGENTS.md | `./AGENTS.md` | (수동 또는 ADR-010 fork 시) | Living |
| Codex 프로젝트 설정 | `.codex/config.toml` | 수동 | Living |
| Codex skill wrapper | `.agents/skills/<name>/{SKILL.md, agents/openai.yaml}` | 수동 | Reference |
```

**Canonical Owner 매핑 표에 추가할 행**:
```
| 도구 어댑터 매핑 (Claude ↔ Codex) | `docs/90-decisions/ADR-010-multi-agent-compatibility.md` |
| AGENTS.md 진입 페이지 정책 (왜 이 파일을 진입점으로 삼는가) | `docs/90-decisions/ADR-010-multi-agent-compatibility.md` |
| 공통 진입 지침 본문 (도구 중립 entry instructions) | `AGENTS.md` |
```

### 6-5. `.codex/config.toml` (P1-5)

> **사전 검증 위치**: 본 산출물의 사전 점검(스키마 변경 여부, `.` = "write" 트리 상속, 모델 접근성)은 §11에 단일 진입점으로 모았다. 본 절은 검증 통과 후 박을 최종 본문만 다룬다.

**최종 `.codex/config.toml`** (보일러플레이트가 진짜로 차이를 만들어야 하는 항목만, upstream 기본값 박지 않음):
```toml
# Codex CLI project config — see ADR-010
# References:
#   https://developers.openai.com/codex/config-reference
#   https://developers.openai.com/codex/config-advanced

# 모델: gpt-5.5는 config-reference의 공식 예시 값. 현행 권장 모델로 단정하지 말고
# 사용 계정에서 실제 접근 가능한지 §11 점검에서 1회 확인 후 박는다.
model = "gpt-5.5"
approval_policy = "on-request"
sandbox_mode = "workspace-write"

# Boilerplate-secure permissions 프로파일 — 빌트인 :workspace와 유사하게 project root
# write를 허용하되, .env/secrets/** 차단을 포함한 custom profile (상속 아님, 독립 정의).
# 현 .claude/settings.json의 secrets 차단(Read deny on .env, .env.*, secrets/**)과
# 동등한 결과를 Codex 측에 보장. 그 이상은 보안 baseline이 아니므로 제한하지 않는다.
default_permissions = "boilerplate-secure"

[permissions.boilerplate-secure.filesystem]
# glob 구현에 따라 "**/.env"가 루트 .env를 잡는지 애매할 수 있으므로 보수적으로 명시.
# 보안 baseline은 중복 패턴을 감수해서라도 확실히 잡는다.
":project_roots" = { "." = "write", ".env" = "none", ".env.*" = "none", "**/.env" = "none", "**/.env.*" = "none", "secrets/**" = "none", "**/secrets/**" = "none" }
glob_scan_max_depth = 3

# [permissions.boilerplate-secure.network] 블록은 의도적으로 두지 않음.
# Claude side baseline(.claude/settings.json)이 네트워크를 막지 않으므로 §7의
# "양쪽 동시 — 동일 결과" 약속을 지키려면 Codex도 막지 않아야 함.
# 네트워크 정책은 Codex sandbox_mode와 사용자 ~/.codex/config.toml 책임.
```

**의도적으로 박지 않은 항목** (upstream 기본값 — pinning 시 stale 위험, ADR-006/SSOT 패턴 1과 충돌):
- `project_doc_max_bytes` (기본 32 KiB)
- `[agents] max_threads`, `max_depth` (기본 6, 1)
- 이 값들은 사용자가 `~/.codex/config.toml`에서 개인적으로 override 가능.

**적용 후 검증**: `codex` 실행 후 `.env`/`secrets/**` 접근이 실제 차단되는지 1회 확인 (`/permissions` 또는 sandbox 로그). wrapper(`.agents/skills/...`)가 SSOT 본문(`.claude/skills/...`)을 read 가능한지도 함께 확인 — `.` = "write"가 깔려 있어 `.claude/**`은 implicit read 가능하지만 실측으로 확정.

**근거**: 현재 `.claude/settings.json`이 막고 있는 보안 baseline(.env, secrets/**)이 Codex 측에 없으면 사용자가 Codex로 한 번만 진입해도 격차 발생. 기능 포팅이 아니라 안전 baseline이므로 §9-1 완료 기준에서 반드시 통과해야 한다.

### 6-6. `.agents/skills/<name>/SKILL.md` wrapper (P1-6)

**4개 대상**: `implement-workitem`, `validate-workitem`, `repair-workitem`, `finalize-workitem`
(이유: inner loop은 task마다 반복되므로 wrapper 가성비가 가장 높다. bootstrap/plan/discover 등은 회당 1회라 자연어 호출로 충분 — Phase 2.)

**디렉터리 구조** (각 wrapper는 폴더, SKILL.md + 정책 파일 둘):
```
.agents/skills/<skill-name>/
├── SKILL.md
└── agents/openai.yaml
```

**`SKILL.md` 양식** (description도 좁게 — 보조 장치):
```markdown
---
name: <skill-name>
description: Use ONLY when the user explicitly types `$<skill-name> <task-id>`. Do not trigger implicitly from generic phrasing.
---

Source of truth: `.claude/skills/<skill-name>/SKILL.md`. Read it and follow the workflow.

Ignore Claude-specific frontmatter (`agent:`, `disable-model-invocation:`, `allowed-tools`, `context:`) — execute locally in Codex.

**Slash command translation**: 본문 안의 `/<skill-name>` 표기는 Claude 슬래시 커맨드다. Codex에서는 `$<skill-name>`으로 읽고 사용자에게 안내한다 (예: 본문 "다음 단계: `/finalize-workitem T-001`" → Codex 응답에서는 "다음 단계: `$finalize-workitem T-001`"). Codex CLI는 `/`를 빌트인 슬래시 커맨드에 쓰므로 명시적 치환이 필요.

Preserve all repo policies from `AGENTS.md` and `docs/`.

If the source path no longer exists, this wrapper is stale — see ADR-010.
```

**`agents/openai.yaml` (필수, implicit invocation 메타데이터 차단)**:
```yaml
policy:
  allow_implicit_invocation: false
```

**왜 두 단계 모두?**: Codex Skills는 description 매칭으로 implicit invocation이 default-on이다([skills](https://developers.openai.com/codex/skills)). Claude의 `disable-model-invocation: true`에 해당하는 functional equivalent는 **`agents/openai.yaml`의 `policy.allow_implicit_invocation: false`** — 이게 metadata-level 명시 차단이고 1차 방어. description 좁히기는 보조(2차 방어)로 유지. 두 단계 모두 두는 게 안전.

**참고**: `agents/openai.yaml` 파일이 없으면 `allow_implicit_invocation`은 default `true`. 누락 시 Codex가 `$implement-workitem` 같은 wrapper를 generic prompt("이 task 좀 구현해줘")에서 자동 트리거할 수 있음.

**호출 방법 안내** (README에도 5줄 추가):
- Codex CLI에서 `$implement-workitem T-001` (또는 `/skills`로 목록 확인 후 선택)

### 6-7. `README.md` / `README_ko.md` 갱신 (P1-7)

**추가할 섹션 (Quick Start 끝에)**:
```markdown
## Codex CLI에서 사용하기 (대체 진입점)

Claude Code 한도에 걸리거나 Codex를 선호할 때:

1. 같은 저장소에서 `codex` 실행 — `AGENTS.md`가 자동 로드된다.
2. 워크플로우는 동일: `docs/00-meta/WORKFLOW.md` 참조.
3. 자주 쓰는 inner-loop는 Codex skill로 호출 가능:
   - `$implement-workitem T-001`
   - `$validate-workitem T-001`
   - `$repair-workitem T-001`
   - `$finalize-workitem T-001`
4. 그 외 단계는 자연어로 호출: *"Follow `.claude/skills/bootstrap-project/SKILL.md`"*

자세한 정책은 [ADR-010](docs/90-decisions/ADR-010-multi-agent-compatibility.md).
```

---

## 7. 도구 표면 매핑 표 (ADR-010 본문에 그대로 들어감)

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

---

## 8. 작업 순서 (의존 관계)

```
§11 사전 점검 ─→ P1-5 (.codex/config.toml 작성)
P1-1 (AGENTS.md)  ──┬─→ P1-2 (CLAUDE.md @import)
                    └─→ P1-7 (README 갱신)
P1-3 (ADR-010)  ────┬─→ P1-4 (STRUCTURE.md, ADR index)
                    └─→ P1-7 (README 갱신)
P1-6 (.agents/skills × 4 폴더, 폴더당 SKILL.md + agents/openai.yaml)   ── 독립
```

**병렬화 가능 분할**:
- 그룹 A (architect-opus 위임 권장): P1-1, P1-3 — 문서 본문 작성, 단순성/SSOT 판단 필요.
- 그룹 B (builder-sonnet 위임 가능): P1-2, P1-4, P1-5, P1-6 — 그룹 A 산출물과 §11 점검 결과를 받아 기계적 적용.
- P1-7 (README): 그룹 A 완료 후.

---

## 9. 검증·완료 기준

### 9-1. Phase 1 완료 기준 (체크리스트)
- [ ] `AGENTS.md`가 200줄 이하이고 32 KiB 미만이며, **도구 종속 표현이 모두 도구 중립으로 재작성됨** ("Claude Code 기반" 등 잔재 0건)
- [ ] `AGENTS.md`의 `docs/` 상대 링크 총 8개가 모두 깨지지 않음 (00-meta 5 + 90-decisions ADR 인덱스 1 + 단순성/TDD 단락 ADR 본문 2 — CLAUDE.md → AGENTS.md 이전 후 재확인)
- [ ] `CLAUDE.md`가 `@AGENTS.md` import 형태 (Claude-only 지침이 없으면 한 줄, 있으면 `## Claude Code` 섹션 추가)
- [ ] `claude` 실행 후 `/memory`에 `CLAUDE.md`가 listed되고, 그 파일 본문에 `@AGENTS.md` import 라인이 있음. 추가로 모델에게 AGENTS.md 고유 문장 한 줄 회수시켜 본문 주입 확인 (`/memory`는 import body를 별도로 listing하지 않으므로 이 회수 단계가 필수)
- [ ] `codex` 실행 후 AGENTS.md 자동 로드 (별도 명령 불필요)
- [ ] Codex에서 `$implement-workitem` 호출 시 wrapper가 발견되고 `.claude/skills/` 본문을 read해서 따라 실행됨 (permissions가 SSOT 본문 read를 막지 않음을 동시 검증)
- [ ] **Implicit invocation 차단 실측**: generic prompt(예: "이 task 좀 구현해줘")에서 4개 wrapper(`implement/validate/repair/finalize-workitem`)가 자동 트리거되지 않음. `agents/openai.yaml`의 `policy.allow_implicit_invocation: false`가 4개 wrapper 모두에 박혀 있음
- [ ] ADR-010이 인덱스에 등록되고 STRUCTURE.md Canonical Owner 표에 매핑이 한 줄 추가됨
- [ ] `.codex/config.toml`의 sandbox/approval이 적용되고, **`.env`/`secrets/**` 접근이 Codex 측에서 실측으로 차단됨** (TODO·미검증 키 잔존 0건)

### 9-2. 회귀 검증
- [ ] 기존 `/bootstrap-project`, `/implement-workitem` 등 Claude Code 슬래시 커맨드가 그대로 동작
- [ ] 기존 sub-agent(architect-opus, builder-sonnet 등) 위임이 그대로 동작
- [ ] `docs/` 어떤 문서도 깨진 링크 없음

### 9-3. 비범위 누락 확인
- [ ] `.codex/agents/*.toml` 파일이 생성되지 않았는지 (Phase 3 비범위)
- [ ] `AGENT_TOOL_MATRIX.md` 별도 파일이 생성되지 않았는지 (D8 비범위)
- [ ] `.agents/skills/` wrapper가 4개(폴더) / 8개(파일)를 초과하지 않는지

---

## 10. 위임 권고 (실행 시)

본 보일러플레이트의 `AGENT_EXECUTION_STRATEGY.md`에 따라:

| Phase 작업 | 권장 에이전트 | 이유 |
|---|---|---|
| P1-1 (AGENTS.md 본문) | architect-opus | 보일러플레이트 진입 페이지 — 단어 선택과 SSOT 정렬이 중요 |
| P1-3 (ADR-010 작성) | architect-opus | 정책 결정 본문, tradeoff 정리 필요 |
| P1-2, P1-4, P1-5, P1-6, P1-7 | builder-sonnet | 그룹 A 결과 + §11 점검 결과를 받아 기계적 적용 |
| 전체 완료 후 | validator-sonnet | 9-1~9-3 체크리스트 검증 |
| 문서 모순·누락 점검 (선택) | reviewer | ADR-010 + STRUCTURE.md drift 점검 |

**병렬 호출 패턴 1번** (한 메시지에서 Agent 호출 병렬, AGENT_EXECUTION_STRATEGY.md "병렬 패턴 3종" 표 참조) — P1-3과 P1-1을 architect-opus 두 호출로 동시 진행 가능.

---

## 11. 적용 전 마지막 점검 사항 (TODO)

> 실행 직전에 1회씩 확인.

3개 항목.

- [ ] **모델 접근성**: `gpt-5.5`는 [config-reference](https://developers.openai.com/codex/config-reference)의 **공식 예시 값**일 뿐 "현행 권장 모델"이 아니다. 사용 계정 / OpenAI 플랜에서 실제 접근 가능한지 P1-5 적용 직전 확인. 불가하면 접근 가능한 ID로 교체하고 ADR-010 "후속 작업"에 메모. ADR-010·README에서도 "공식 예시" vs "현행 권장" 구분 유지.
- [ ] **permissions 스키마 + 트리 상속**: ① §2-3 스키마가 [config-advanced](https://developers.openai.com/codex/config-advanced)에서 본 가이드 작성 이후 변경되지 않았는지 1회 확인. ② `":project_roots"`의 `"." = "write"`가 트리 전체로 상속되는지 1회 실측 확인 (config-advanced에 명시 진술 없음. 만약 root 1단계만이면 `.claude/**`, `docs/**` 등 명시 패턴 추가 필요 → P1-5 템플릿 재작업 트리거). ③ P1-5 적용 후 `.env`/`secrets/**` 차단 실측 (§9-1과 같은 행위지만 시점이 다름 — 여기는 P1-5 직후, §9-1은 Phase 1 종료 직전).
- [ ] **wrapper 대상 목록 sync**: `.claude/skills/`의 현재 개수(12개) 변동 여부 — P1-6 / P2-1 / P2-2 목록이 최신과 일치하는지 1회 점검.

---

## 12. 참고 출처 (1차 공식)

### Claude Code
- [Memory (CLAUDE.md, @import, AGENTS.md 가이드)](https://code.claude.com/docs/en/memory)
- [Settings](https://code.claude.com/docs/en/settings)
- [Sub-agents](https://code.claude.com/docs/en/sub-agents)
- [Skills](https://code.claude.com/docs/en/skills)

### Codex CLI
- [AGENTS.md guide](https://developers.openai.com/codex/guides/agents-md)
- [Best practices](https://developers.openai.com/codex/learn/best-practices)
- [Config basics](https://developers.openai.com/codex/config-basic)
- [Config reference](https://developers.openai.com/codex/config-reference)
- [Subagents](https://developers.openai.com/codex/subagents)
- [Skills](https://developers.openai.com/codex/skills)
- [Hooks](https://developers.openai.com/codex/hooks)
- [Custom prompts (deprecated)](https://developers.openai.com/codex/custom-prompts)
- [Slash commands](https://developers.openai.com/codex/cli/slash-commands)

### 본 보일러플레이트 내부 참조
- [ADR-004 모델 별칭](docs/90-decisions/ADR-004-model-alias-policy.md)
- [ADR-005 SSOT](docs/90-decisions/ADR-005-ssot.md)
- [ADR-006 단순성·Clean Code·Clean Architecture](docs/90-decisions/ADR-006-simplicity-and-architecture.md)
- [ADR-007 워크아이템 라이프사이클](docs/90-decisions/ADR-007-workitem-lifecycle.md)
- [STRUCTURE.md 산출물 인벤토리](docs/00-meta/STRUCTURE.md)
- [AGENT_EXECUTION_STRATEGY.md](docs/00-meta/AGENT_EXECUTION_STRATEGY.md)
- [GUARDRAILS_STRATEGY.md](docs/00-meta/GUARDRAILS_STRATEGY.md)

---

## 13. 폐기 절차

이 가이드는 작업이 끝나면 삭제한다.
- ADR-010이 accepted 상태로 머지되고
- Phase 1 모든 산출물(P1-1 ~ P1-7)이 머지되고
- 9-1 체크리스트가 모두 통과하면

→ 본 파일(`DRAFT-IMPROVE-GUIDE.md`)을 git rm으로 삭제한다. 결정과 근거는 ADR-010이, 산출물 위치는 STRUCTURE.md가 영속적으로 보존한다.
