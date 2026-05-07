<!--
이 문서는 "다중 에이전트 호환(Claude Code + Codex)" 개선의 실행 가이드다.
처음 합류한 사람이 이 문서만 보고 따라가도 Phase 1 산출물을 모두 만들 수 있도록 작성됐다.
초안(DRAFT-IMPROVE-GUIDE.md)은 의사결정·근거 모음, 본 문서는 실제 행동 지침이다.

작업이 끝나면(ADR-010 accepted + Phase 1 산출물 머지 + §8 체크리스트 통과)
DRAFT-IMPROVE-GUIDE.md와 본 IMPROVE-GUIDE.md를 모두 git rm으로 삭제한다.
결정과 근거는 ADR-010이, 산출물 위치는 STRUCTURE.md가 영속적으로 보존한다.
-->

# 보일러플레이트 다중 에이전트 호환 개선 — 실행 가이드

## 0. 메타정보

| 항목 | 값 |
|---|---|
| 상태 | ready (실행 가능) |
| 작성일 | 2026-05-07 |
| 폐기 시점 | ADR-010 accepted + Phase 1 산출물 머지 + §8 체크리스트 통과 시 |
| 관련 ADR | ADR-004 (모델 별칭), ADR-005 (SSOT), ADR-006 (단순성), ADR-008 (Conventional Commits), ADR-010 (본 작업으로 신설) |
| 영향받는 문서 | `CLAUDE.md`, `docs/00-meta/STRUCTURE.md`, `docs/90-decisions/README.md`, `docs/90-decisions/ADR-005-ssot.md`, `docs/90-decisions/ADR-006-simplicity-and-architecture.md`, `docs/90-decisions/ADR-009-tdd-default.md`, `.claude/skills/bootstrap-project/SKILL.md`, `README.md`, `README_ko.md`, 신설: `AGENTS.md`, `docs/90-decisions/ADR-010-multi-agent-compatibility.md` |
| 신설되는 디렉터리/파일 | `AGENTS.md`, `.codex/config.toml`, `.agents/skills/{implement,validate,repair,finalize}-workitem/{SKILL.md, agents/openai.yaml}` (총 8개 파일) |

---

## 1. 시작하기 전에 (배경 한 단락)

현재 보일러플레이트는 Claude Code 표면(`CLAUDE.md`, `.claude/`)에만 묶여 있다. 본 작업은 **같은 저장소에서 Claude Code와 OpenAI Codex CLI 양쪽 모두 동일한 워크플로우를 따라 작업할 수 있게** 만든다. 단, 단순성(ADR-006)과 SSOT(ADR-005)을 절대 위반하지 않는다 — 어댑터 표면은 최소화하고, 본문은 1곳, 다른 곳은 링크다.

핵심 결정 4개 (자세한 근거는 Step 3에서 ADR-010으로 작성):

- **D1**: `AGENTS.md`(프로젝트 루트)를 캐노니컬 진입 페이지로 둔다. Codex CLI가 자동으로 읽고, Claude Code는 `@AGENTS.md` import로 동일 본문을 사용한다.
- **D2**: `CLAUDE.md`는 `@AGENTS.md` import + (필요 시) Claude-only 추가 지침 섹션만 둔다.
- **D3**: `.claude/skills/<name>/SKILL.md`가 워크플로우 본문의 **canonical owner**다. Codex 측 `.agents/skills/<name>/`은 그 본문을 가리키는 **얇은 wrapper만** 둔다.
- **D5**: `.codex/config.toml`은 **안전 baseline 최소 설정만** 둔다(sandbox, approval, secrets 차단, 모델). upstream 기본값은 박지 않는다.

---

## 2. 진행 방식 안내 (어떻게 따라갈 것인가)

### 2-1. 단계 구성

본 가이드는 9개 단계로 구성된다.

| 단계 | 이름 | 주요 산출물 | 커밋 |
|---|---|---|---|
| Step 0 | 사전 점검 | (점검 결과만 메모) | (없음) |
| Step 1 | `AGENTS.md` 신설 | `AGENTS.md` | 1개 |
| Step 2 | `CLAUDE.md`를 import 형태로 교체 | `CLAUDE.md` | 1개 |
| Step 3 | ADR-010 작성 | `docs/90-decisions/ADR-010-multi-agent-compatibility.md` | 1개 |
| Step 4 | STRUCTURE.md + ADR 인덱스 등록 + 기존 surface stale 갱신 (CLAUDE.md → AGENTS.md) | `docs/00-meta/STRUCTURE.md`, `docs/90-decisions/{README,ADR-005,ADR-006,ADR-009}.md`, `.claude/skills/bootstrap-project/SKILL.md` | 1개 |
| Step 5 | `.codex/config.toml` 신설 | `.codex/config.toml` | 1개 |
| Step 6 | `.agents/skills/` wrapper 4개 신설 | 폴더 4개, 파일 8개 | 1개 |
| Step 7 | README 두 종 갱신 | `README.md`, `README_ko.md` | 1개 |
| Step 8 | 최종 검증 (체크리스트 점검) | (검증 메모) | (없음) |
| Step 9 | DRAFT 삭제 | `DRAFT-IMPROVE-GUIDE.md`, `IMPROVE-GUIDE.md` 삭제 | 1개 |

### 2-2. 단계 간 의존 관계

```
Step 0 ── (점검 통과) ──┐
                        │
Step 1 (AGENTS.md) ─┬──→ Step 2 (CLAUDE.md import)
                    └──→ Step 7 (README)
Step 3 (ADR-010) ───┬──→ Step 4 (STRUCTURE/ADR index)
                    └──→ Step 7 (README)
Step 5 (.codex/config.toml)  ─ 독립 (Step 0 통과 후 실행)
Step 6 (.agents/skills × 4)   ─ 독립
Step 8 ──────────────────────────→ Step 9 (DRAFT 삭제)
```

병렬화하려면 Step 1과 Step 3이 함께 가능, Step 5와 Step 6이 함께 가능. 처음 따라가는 사람은 **순차 진행**을 권장한다 — 한 번에 한 산출물을 끝내고 커밋해야 회귀 추적이 쉽다.

### 2-3. 위임 권고 (각 Step에서 어떤 에이전트를 쓸 것인가)

본 보일러플레이트의 [AGENT_EXECUTION_STRATEGY.md](docs/00-meta/AGENT_EXECUTION_STRATEGY.md) 위임 트리거를 따른다.

| Step | 권장 에이전트 | 이유 |
|---|---|---|
| Step 1 (AGENTS.md 본문) | architect-opus | 보일러플레이트 진입 페이지 — 단어 선택과 SSOT 정렬 중요 |
| Step 3 (ADR-010 작성) | architect-opus | 정책 결정 본문, tradeoff 정리 필요 |
| Step 2, 4, 5, 6, 7 | builder-sonnet | Step 1·3 산출물과 Step 0 점검 결과를 받아 기계적 적용 |
| Step 8 (검증) | validator-sonnet | §8 체크리스트 점검 |
| Step 8 보강 (선택) | reviewer | ADR-010 ↔ STRUCTURE.md drift 점검 |

병렬 가속: Step 1과 Step 3을 architect-opus 두 호출로 동시 진행 가능 (병렬 패턴 1번 — 한 메시지에서 Agent 호출 병렬, [AGENT_EXECUTION_STRATEGY.md "병렬 패턴 3종"](docs/00-meta/AGENT_EXECUTION_STRATEGY.md#병렬-패턴-3종) 참조).

### 2-4. 커밋 규칙

- 모든 커밋 메시지는 **영문 1줄, Conventional Commits 스타일**(ADR-008)을 따른다. 예: `docs: add AGENTS.md as canonical multi-agent entry`
- 각 Step 끝에 명시된 커밋 메시지를 그대로 사용한다.
- `git add -A` / `git add .` 금지. 명시 파일만 stage한다 — `/finalize-workitem` 정책과 동일.
- `--amend`, `--no-verify`, `git push` 금지 (사용자가 명시 요청한 경우 제외).

---

## 3. 비범위 (이번 작업에서 하지 않는 것)

다음은 **하지 않는다**. 추후 별도 작업으로 분리한다.

- [ ] `.claude/skills/` 12개 모두를 `.agents/skills/`로 wrapper 만들기 — 본 작업에서는 inner-loop 4개(`implement/validate/repair/finalize-workitem`)만. 나머지 6개 워크플로우 skill과 유틸리티 2개(`review-doc`, `boilerplate-context`)는 Phase 2/보류.
- [ ] `.claude/agents/` 6개 sub-agent를 모두 `.codex/agents/*.toml`로 1:1 미러링. SSOT 위반·drift 비용 큼. (Phase 3 보류)
- [ ] 별도 `docs/00-meta/AGENT_TOOL_MATRIX.md` 문서 신설. 도구 표면 매핑 표는 ADR-010 본문에 흡수한다.
- [ ] Codex hooks 도입 (`hooks.json` / `[features] codex_hooks`). [GUARDRAILS_STRATEGY.md](docs/00-meta/GUARDRAILS_STRATEGY.md)의 "스택 확정 후 도입" 원칙에 따라 보류.
- [ ] CI / GitHub Action 단의 Codex 통합. 별 워크아이템.
- [ ] `~/.codex/AGENTS.md` 사용자 글로벌 설정 강제. 사용자 환경 결정.
- [ ] Plan 모드(Claude) ↔ `/plan`(Codex) 동등 매핑. 동작 메커니즘이 다르므로 워크플로우 문서 수준에서만 안내.
- [ ] Cursor / Gemini CLI / OpenCode 등 추가 도구 호환. AGENTS.md 도입으로 자연 확장되지만 본 작업의 책임은 아님.

위 항목 중 어느 것이라도 작업 중에 손이 가면 **즉시 멈추고 본 가이드의 비범위로 돌아온다**.

---

## Step 0 — 사전 점검 (실행 직전 1회씩 확인)

> 이 단계는 산출물이 없다. 점검 결과를 메모해두고, 통과 못하면 해당 항목의 후속 단계를 보류한다.

### 0-1. 모델 접근성 점검 (P1-5 차단 가능)

`gpt-5.5`는 [Codex config-reference](https://developers.openai.com/codex/config-reference)의 **공식 예시 값**일 뿐 "현행 권장 모델"이 아니다. 사용 계정/플랜에서 실제 접근 가능한지 1회 확인한다.

**실측 방법**:
1. Codex CLI가 설치되어 있으면 `codex --help` 또는 `codex` 진입 후 모델 지정 옵션을 확인.
2. 또는 OpenAI 계정 dashboard에서 사용 가능한 모델 목록 확인.

**결과 처리**:
- 접근 가능 → 그대로 `model = "gpt-5.5"` 사용 (Step 5).
- 접근 불가 → 접근 가능한 ID로 교체 + ADR-010 "후속 작업"에 메모 ("Codex 모델 ID는 본 ADR이 추적, 새 ADR로 superseding").
- 어떤 경우든 ADR-010·README에서 "공식 예시" vs "현행 권장" 구분을 유지한다.

### 0-2. permissions 스키마 + 트리 상속 점검 (P1-5 차단 가능)

**3개 하위 점검**:

1. **스키마 변경 여부**: DRAFT-IMPROVE-GUIDE.md §2-3에 정리된 permissions 스키마(`[permissions.<name>.filesystem]` + `":project_roots"` inline-table + glob 패턴 키 → 모드 매핑)가 [Codex config-advanced](https://developers.openai.com/codex/config-advanced)에서 본 가이드 작성 이후 변경되지 않았는지 1회 확인. 변경됐으면 Step 5의 템플릿을 새 스키마에 맞게 수정.

2. **`.` = "write" 트리 상속 실측**: `":project_roots"`의 `"." = "write"`가 트리 전체로 상속되는지 1회 실측 확인. config-advanced에 명시 진술 없음. **만약 root 1단계만 적용된다면** Step 5-3의 fallback 패턴(`"**" = "write"` + `.claude/skills/** = "read"` + 보안 deny)으로 §5-2 inline-table을 교체해야 한다. fallback 자체에서도 `"**" = "write"`가 트리 전체에 적용되는지 1회 실측 필요.

3. **차단 실측 (Step 5 적용 직후)**: Step 5에서 `.codex/config.toml`을 박은 직후, `codex` 실행 후 `.env`/`secrets/**` 접근이 실제 차단되는지 확인 (Step 5-5의 방법 A 또는 B). Step 8의 §8-1 체크리스트와 같은 행위지만 시점이 다름 — 여기는 Step 5 직후, §8-1은 Phase 1 종료 직전.

**결과 처리**:
- 스키마 변경 발견 → Step 5 템플릿 재작업 + ADR-010 본문에 적용된 스키마 명시.
- `.` 비상속 발견 → Step 5-3 fallback 패턴으로 교체 + ADR-010 "후속 작업"에 적용 사실 메모.

### 0-3. Codex trusted project 인식 점검 (P1-5 전제 조건)

Codex CLI는 처음 만나는 프로젝트의 `.codex/`를 자동 로드하지 않는다. **trusted project로 등록되어야** `.codex/config.toml`이 로드된다 ([Codex config-basic](https://developers.openai.com/codex/config-basic) — *"project .codex/ layers only when you trust the project"*).

**실측 방법**:
1. 처음 `codex` 실행 시 trust 프롬프트가 뜨면 `yes/trust` 선택.
2. 기존 untrusted 상태였다면 Codex CLI에서 trust 토글 명령(또는 사용자 글로벌 설정 `~/.codex/config.toml`의 trusted projects 항목)으로 이 repo를 등록.

**결과 처리**:
- trusted → 그대로 진행.
- untrusted → 본 작업의 모든 검증(Step 5-5, Step 8-1)이 의미 없어짐. 사용자에게 trust 절차 안내 후 재진입.

### 0-4. wrapper 대상 목록 sync 점검

`.claude/skills/`의 현재 폴더 개수가 본 가이드 작성 시점(12개)과 일치하는지 확인.

```powershell
# PowerShell
(Get-ChildItem .claude\skills -Directory).Count
```

또는

```bash
# Bash
ls -d .claude/skills/*/ | wc -l
```

**결과 처리**:
- 12개 일치 → 그대로 진행.
- 변동 → Step 6의 wrapper 대상 목록(`implement/validate/repair/finalize-workitem`)이 여전히 inner-loop 전체인지 확인. 변경 사항이 있으면 비범위(§3) 정의도 함께 갱신.

### 0-5. Step 0 결과 메모

다음과 같이 conversation/PR description 또는 임시 메모에 남긴다 (커밋 X). **0-1, 0-2 결과 중 ADR-010 의사결정에 영향을 주는 항목**(예: `gpt-5.5` 대체 ID, 스키마 변경, `.` 비상속)은 Step 3 ADR-010 본문 또는 "후속 작업"에 박아 영속화한다.

```text
Step 0 사전 점검 결과 (YYYY-MM-DD):
- 0-1 모델 접근성: gpt-5.5 (접근 가능 / 불가 — 대체 ID: ___)
- 0-2-1 스키마 변경: 변경 없음 / 변경 있음 (___)
- 0-2-2 . = "write" 트리 상속: 트리 전체 상속 / root 1단계만 (Step 5-3 fallback 적용 여부: ___)
- 0-3 trusted project: trusted / untrusted (해소 방법: ___)
- 0-4 wrapper 대상: .claude/skills/ 폴더 N개 (변동 없음 / 변동 ___)
```

**커밋**: 없음. 점검 결과는 conversation/PR description에 남기고, 영속화가 필요한 항목만 ADR-010에 옮긴다.

---

## Step 1 — `AGENTS.md` 신설 (P1-1)

> 권장 에이전트: **architect-opus** — 진입 페이지 본문은 단어 선택과 SSOT 정렬이 중요하다.

### 1-1. 위치와 길이

- **위치**: 프로젝트 루트 (`./AGENTS.md`).
- **길이**: 200줄 이하 + 32 KiB 미만. (Codex `project_doc_max_bytes` 기본 32 KiB, Claude 권장 200줄 — 더 엄격한 200줄 기준)
- **HTML 주석 블록 길게 두지 않기**: Claude Code는 자동 스트립하지만 Codex가 같은 동작을 한다는 공식 보장이 없음. 사람용 메모는 별도 문서로.

### 1-2. 본문 구성 (현 `CLAUDE.md` 본문을 도구 중립적으로 재작성)

다음 섹션을 그대로 박는다. 도구별 분기("Claude는 X, Codex는 Y") 같은 내용은 **절대 박지 않는다** — 양쪽 공용 지침만. 도구별 차이는 ADR-010이 담당.

**섹션 구성**:

1. `# 프로젝트 지침`
2. `## 목적` — **도구 종속 표현을 도구 중립으로 재작성**.
   - 잘못된 예 (현 CLAUDE.md): "Claude Code 기반의 재사용 가능한 문서 중심 개발 보일러플레이트다"
   - 권장 1: "AI coding agent 기반의 재사용 가능한 문서 중심 개발 보일러플레이트다"
   - 권장 2: "여러 AI coding agent (Claude Code, OpenAI Codex 등)에서 동일하게 동작하는 문서 중심 개발 보일러플레이트다"
3. `## 핵심 행동 규율` — 현 CLAUDE.md의 5~7개 항목을 그대로 도구 중립 표현으로 옮김.
4. `## 단순성·YAGNI (구현 시 항상 적용)` — 현 CLAUDE.md 5개 항목 + ADR-006 링크.
5. `## TDD 기본 (구현 시 디폴트)` — 현 CLAUDE.md 1단락 + ADR-009 링크.
6. `## 깊은 운영 원칙은 다음 문서를 따른다` — `docs/00-meta/*` 5종 + `docs/90-decisions/README.md` ADR 인덱스 1종 (총 6개 링크).

### 1-3. 링크 검증 (필수)

AGENTS.md에는 `docs/` 상대 링크가 **총 8개** 있어야 한다 — `## 6` 섹션 6개 + 단순성 단락(ADR-006) 1개 + TDD 단락(ADR-009) 1개. 모두 깨지지 않는지 미리 확인.

```powershell
# PowerShell — AGENTS.md 작성 후 실측
Select-String -Path AGENTS.md -Pattern '\]\(docs/' | Measure-Object
```

8개 미만이면 누락, 초과면 잘못 추가된 링크가 있다.

### 1-4. 만들 때 따라하기 쉬운 본문 골격

```markdown
# 프로젝트 지침

## 목적
- 이 저장소는 여러 AI coding agent (Claude Code, OpenAI Codex 등)에서 동일하게 동작하는 문서 중심 개발 보일러플레이트다.
- 새 프로젝트를 시작할 때 이 구조를 복제해 빠르게 적용할 수 있어야 한다.
- 중요한 단계마다 해당 계층의 문서를 먼저 갱신한 뒤 다음 단계로 진행한다.

## 핵심 행동 규율
- 상위 문서 없이 하위 문서를 먼저 만들지 않는다.
- `.env`, `secrets/` 같은 민감 파일은 건드리지 않는다.
- 작업 범위와 비범위를 명확히 적고, 범위 밖 변경은 하지 않는다.
- 흩어진 임시 메모보다 정해진 위치의 문서를 갱신한다.
- 사실, 가정, 열린 질문을 구분해서 적는다. 검증 가능한 표현을 우선한다.
- 커밋은 작고 논리적인 단위로 나눈다. 커밋 전에 관련 workitem 문서와 구현 범위가 일치하는지 확인한다.

## 단순성·YAGNI (구현 시 항상 적용)
- 요구한 범위만 구현한다. 추측성 추상화·미래 대비 코드·계획에 없는 헬퍼는 만들지 않는다.
- 동일 패턴이 3회 이상 반복될 때까지는 추출하지 않는다. 비슷한 코드 3줄이 premature abstraction보다 낫다.
- 시스템 경계(외부 입력, 외부 API)에서만 입력 검증·에러 핸들링을 둔다. 내부 호출에는 두지 않는다.
- WHY가 비자명할 때만 주석을 단다(숨은 제약, 미묘한 invariant, 특정 버그 우회). WHAT 주석은 좋은 식별자 이름으로 대체한다.
- backwards-compat shim, feature flag, 사용 안 되는 변수의 `_` rename 같은 호환 hack을 만들지 않는다. 정말 안 쓰면 삭제한다.

정책 근거: [ADR-006-simplicity-and-architecture.md](docs/90-decisions/ADR-006-simplicity-and-architecture.md).

## TDD 기본 (구현 시 디폴트)
구현은 Red → Green → Refactor 3 phase 사이클을 디폴트로 따른다. opt-out은 task 문서의 `## 6-2. TDD opt-out`에 사유와 follow-up이 모두 있을 때만. 정책 근거: [ADR-009-tdd-default.md](docs/90-decisions/ADR-009-tdd-default.md).

## 깊은 운영 원칙은 다음 문서를 따른다
- [문서 계층과 산출물 인벤토리](docs/00-meta/STRUCTURE.md)
- [문서 계층 정의 + 네이밍](docs/00-meta/TEMPLATE_GUIDE.md)
- [워크플로우 + 문서 상태 전이](docs/00-meta/WORKFLOW.md)
- [에이전트 실행 전략 + 위임 트리거](docs/00-meta/AGENT_EXECUTION_STRATEGY.md)
- [Guardrail 운영 원칙](docs/00-meta/GUARDRAILS_STRATEGY.md)
- [ADR 인덱스](docs/90-decisions/README.md)
```

### 1-5. Step 1 완료 체크

- [ ] `AGENTS.md`가 프로젝트 루트에 존재.
- [ ] 200줄 이하, 32 KiB 미만.
- [ ] "Claude Code 기반" 같은 도구 종속 표현이 본문에 0건.
- [ ] `docs/` 상대 링크 8개 모두 깨지지 않음.
- [ ] HTML 주석 블록 길지 않음.

### 1-6. 커밋

```text
docs: add AGENTS.md as canonical multi-agent entry doc
```

---

## Step 2 — `CLAUDE.md`를 `@AGENTS.md` import 형태로 교체 (P1-2)

> 권장 에이전트: **builder-sonnet** — 기계적 교체.

### 2-1. 무엇을 하나

현 `CLAUDE.md`의 본문은 Step 1에서 이미 `AGENTS.md`로 옮겨졌다. 이제 `CLAUDE.md`는 import 한 줄만 남긴다.

### 2-2. 교체 후 `CLAUDE.md` 전체 본문

**Claude-only 추가 지침이 없는 경우** (대부분):

```markdown
@AGENTS.md
```

(파일 1줄, 그 외 placeholder 섹션 두지 말 것 — 필요해질 때 추가.)

**Claude-only 추가 지침이 생기면** (지금은 해당 없음, 향후 가이드용):

```markdown
@AGENTS.md

## Claude Code

(Claude Code에서만 적용할 지침)
```

### 2-3. 근거 (수정하면서 알고 있을 것)

- [code.claude.com/docs/en/memory](https://code.claude.com/docs/en/memory)가 권장하는 패턴.
- `@AGENTS.md`는 Claude Code launch 시 인라인 로드되므로 모델 재량에 의존하지 않는다.
- 빈 placeholder 섹션은 두지 않는다. SSOT 패턴 5("CLAUDE.md = 진입 페이지")가 깨지지 않게.

### 2-4. Step 2 완료 체크

- [ ] `CLAUDE.md`가 `@AGENTS.md` 한 줄(또는 한 줄 + Claude-only 섹션) 형태.
- [ ] 본문 정의가 중복되지 않음 (정의는 `AGENTS.md`에만).

### 2-5. 커밋

```text
docs(claude): import AGENTS.md as the single source of entry instructions
```

---

## Step 3 — `ADR-010` 작성 (P1-3)

> 권장 에이전트: **architect-opus** — 정책 결정 본문.
> Step 1과 병렬 가능 (architect-opus 두 호출).

### 3-1. 위치

`docs/90-decisions/ADR-010-multi-agent-compatibility.md`

### 3-2. ADR 번호 확인

본 가이드 작성 시점 기준으로 `docs/90-decisions/`에는 ADR-001 ~ ADR-009가 있다. ADR-010이 다음 번호. (만약 사이에 다른 ADR이 추가됐으면 가장 큰 번호 + 1로 갱신.)

### 3-3. 본문 양식

[_ADR_GUIDE.md](docs/90-decisions/_ADR_GUIDE.md)의 권장 섹션을 따른다. 다음 골격을 사용한다.

```markdown
# ADR-010 다중 에이전트 도구 호환 (AGENTS.md as canonical entry)

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
- **운영 안내 1**: `docs/` 본문(예: `docs/00-meta/WORKFLOW.md`, `AGENT_EXECUTION_STRATEGY.md`)에 등장하는 `/<skill-name>` 표기는 Claude 슬래시 커맨드다. Codex 사용자는 동일 skill을 `$<skill-name>`으로 읽는다 (Step 6 wrapper와 동일 변환).
- **운영 안내 2**: `.codex/config.toml`의 `.claude/skills/**`는 (fallback 분기 시) Codex에서 read-only로 박힌다 — `.claude/skills/<name>/SKILL.md`는 D3에 의해 canonical SSOT이므로 직접 편집은 Claude Code 측에서 수행한다.

## 후속 작업
- Phase 2 workflow wrapper 6개 (Codex 사용 빈도 자리잡으면, 사용 빈도에 따라 선별), 유틸리티 2개(`review-doc`, `boilerplate-context`)는 필요 시.
- Phase 3 `.codex/agents/` TOML (명시 subagent workflow 자주 쓰게 되면).
- Codex 모델 ID 갱신은 본 ADR을 새 ADR로 superseding.
- (Step 0-1에서 `gpt-5.5` 미접근 발견 시) 본 ADR "후속 작업"에 사용된 대체 ID와 갱신 책임자 명시.
- ADR-005 SSOT 패턴 5("CLAUDE.md = 진입 페이지")의 표현을 "entry page (AGENTS.md)"로 갱신하는 후속 ADR 또는 in-place 수정 검토. 본 ADR 채택 후 캐노니컬 진입점이 AGENTS.md로 옮겨가므로 패턴 5의 단어가 어긋난다.
- (Step 0-2에서 스키마 변경 또는 `.` 비상속 발견 시) 적용된 fallback 패턴을 본 ADR "결과" 또는 "후속 작업"에 명시해 추적성 유지.
```

### 3-4. Step 3 완료 체크

- [ ] `docs/90-decisions/ADR-010-multi-agent-compatibility.md` 존재.
- [ ] 상태 = `accepted`.
- [ ] 결정 표 D1 ~ D8 모두 포함.
- [ ] 도구 표면 매핑 표 포함 (10행).
- [ ] ADR-005, ADR-006, ADR-004 링크 포함.

### 3-5. 커밋

```text
docs(adr-010): record multi-agent compatibility decision (AGENTS.md canonical entry)
```

---

## Step 4 — STRUCTURE.md + ADR 인덱스 갱신 + 기존 surface stale 갱신 (P1-4)

> 권장 에이전트: **builder-sonnet** — 표 행 추가 + in-place 1줄 수정.
> Step 3 완료 후 진행.

> **Step 4의 의도**: ADR-010 채택으로 진입 페이지가 `CLAUDE.md` → `AGENTS.md`로 이동하면 기존 ADR/STRUCTURE/SKILL 본문 곳곳에 "CLAUDE.md에 …" 문구가 stale로 남는다. Step 4는 ADR-010 인덱싱뿐 아니라 그 stale을 **같은 단계에서 in-place 갱신**해 Phase 1 머지 직후의 일관성을 보장한다(원자성).

### 4-1. `docs/90-decisions/README.md` 갱신 (ADR 인덱스)

ADR 인덱스 표 끝에 다음 행 추가:

```markdown
| 010 | Multi-agent compatibility (AGENTS.md as canonical entry) | accepted | AGENTS.md를 캐노니컬 진입 페이지로, Codex CLI도 동일 워크플로우 동작 |
```

### 4-2. `docs/00-meta/STRUCTURE.md` 갱신 — 산출물 표

`## 산출물 표`에 다음 3행 추가 (기존 마지막 행 바로 아래):

```markdown
| AGENTS.md | `./AGENTS.md` | (수동 또는 ADR-010 fork 시) | Living |
| Codex 프로젝트 설정 | `.codex/config.toml` | 수동 | Living |
| Codex skill wrapper | `.agents/skills/<name>/{SKILL.md, agents/openai.yaml}` | 수동 | Reference |
```

### 4-3. `docs/00-meta/STRUCTURE.md` 갱신 — Canonical Owner 매핑 표 (신규 3행)

`## Canonical Owner 매핑 (SSOT 부록)` 섹션의 표 끝에 다음 3행 추가:

```markdown
| 도구 어댑터 매핑 (Claude ↔ Codex) | `docs/90-decisions/ADR-010-multi-agent-compatibility.md` |
| AGENTS.md 진입 페이지 정책 (왜 이 파일을 진입점으로 삼는가) | `docs/90-decisions/ADR-010-multi-agent-compatibility.md` |
| 공통 진입 지침 본문 (도구 중립 entry instructions) | `AGENTS.md` |
```

### 4-4. `docs/00-meta/STRUCTURE.md` Canonical Owner 매핑 표 — 기존 행 stale 갱신 (in-place)

본 작업 후 단순성/TDD 요약 본문이 `CLAUDE.md`에서 `AGENTS.md`로 이동한다. 기존 두 행을 in-place 수정:

행 1 (단순성·YAGNI·Clean Code/Architecture 정책):
- 변경 전: `ADR-006 + CLAUDE.md(요약)` ← 두 번째 칼럼 끝
- 변경 후: `ADR-006 + AGENTS.md(요약)`

행 2 (TDD 정책):
- 변경 전: `ADR-009 + CLAUDE.md(1줄)`
- 변경 후: `ADR-009 + AGENTS.md(1줄)`

(다른 칼럼·행은 그대로.)

### 4-5. ADR-005 패턴 5 표현 갱신 (in-place, 1줄)

`docs/90-decisions/ADR-005-ssot.md`의 패턴 5 줄 (현 :25 부근):

- 변경 전: `**"CLAUDE.md = 진입 페이지" 패턴** — \`CLAUDE.md\`는 fork된 새 세션이 자동 로드하는 진입점이다. ...`
- 변경 후: `**"진입 페이지" 패턴** — 도구별 진입점(\`AGENTS.md\`가 캐노니컬, \`CLAUDE.md\`는 \`@AGENTS.md\` import)은 fork된 새 세션이 자동 로드한다. ... (정책 근거: ADR-010)`

본문의 다른 표현은 그대로. ADR-005 자체 상태(`accepted`)는 유지 — 본 ADR이 패턴 5의 표현만 갱신하고 의도(slim entry page)는 그대로다.

### 4-6. ADR-006 stale 갱신 (in-place)

`docs/90-decisions/ADR-006-simplicity-and-architecture.md`:

- §결과 표(현 :27 부근): `\| 단순성·YAGNI \| \`CLAUDE.md\` \| 5개 항목, fork된 새 세션이 자동 로드. \|` → `\| 단순성·YAGNI \| \`AGENTS.md\` \| 5개 항목, fork된 새 세션이 자동 로드. \|`
- §결과 본문(현 :40 부근): `\`CLAUDE.md\`에 단순성 5개 항목 단락 추가.` → `\`AGENTS.md\`에 단순성 5개 항목 단락 (\`CLAUDE.md\`는 \`@AGENTS.md\` import).`

### 4-7. ADR-009 stale 갱신 (in-place)

`docs/90-decisions/ADR-009-tdd-default.md` §결과(현 :50 부근):

- 변경 전: `CLAUDE.md에 "TDD 기본" 1단락(fork된 새 세션 자동 로드 surface).`
- 변경 후: `AGENTS.md에 "TDD 기본" 1단락(fork된 새 세션 자동 로드 surface; CLAUDE.md는 @AGENTS.md import).`

### 4-8. `bootstrap-project` SKILL.md "반드시 먼저 읽을 파일" 갱신

`.claude/skills/bootstrap-project/SKILL.md`의 "반드시 먼저 읽을 파일" 목록에서 `- CLAUDE.md` 1줄을 다음 1줄로 교체:

- 변경 후: `- AGENTS.md (CLAUDE.md는 @AGENTS.md import이므로 본문은 AGENTS.md에서 읽는다)`

### 4-9. Step 4 완료 체크

- [ ] `docs/90-decisions/README.md` 표에 010행 존재.
- [ ] `docs/00-meta/STRUCTURE.md` 산출물 표에 AGENTS.md / Codex 설정 / Codex wrapper 3행 존재.
- [ ] Canonical Owner 매핑 표에 신규 3행 + 기존 2행 in-place 갱신 (단순성·YAGNI / TDD 행이 `AGENTS.md` 표기).
- [ ] ADR-005 패턴 5 표현이 "진입 페이지" 패턴 + AGENTS.md 캐노니컬로 갱신됨.
- [ ] ADR-006 §결과 표·본문이 `CLAUDE.md` → `AGENTS.md`로 갱신됨.
- [ ] ADR-009 §결과가 `AGENTS.md`로 갱신됨.
- [ ] `bootstrap-project` SKILL.md "반드시 먼저 읽을 파일"이 AGENTS.md 우선으로 갱신됨.
- [ ] grep으로 "CLAUDE.md에 단순성", "CLAUDE.md에 ... TDD" 같은 stale 표현 잔존 0건 확인:
  ```bash
  grep -rn "CLAUDE.md에" docs/ .claude/ || echo "no stale references"
  ```

### 4-10. 커밋

```text
docs(meta): register ADR-010 and realign existing surfaces to AGENTS.md
```

---

## Step 5 — `.codex/config.toml` 신설 (P1-5)

> 권장 에이전트: **builder-sonnet** — 정해진 본문 박기.
> **선행 조건**: Step 0-1, 0-2 통과. 아니면 본 Step 보류.

### 5-1. 위치와 디렉터리 생성

- 위치: `./.codex/config.toml`
- `.codex/` 디렉터리가 없으면 먼저 만든다.

```powershell
# PowerShell
New-Item -ItemType Directory -Path .codex -Force | Out-Null
```

### 5-2. 본문 (그대로 복사)

본 보일러플레이트가 진짜로 차이를 만들어야 하는 항목만 박는다. **upstream 기본값은 박지 않는다**(ADR-006 단순성 + SSOT 패턴 1).

박는 3개 항목(`model`, `approval_policy`, `sandbox_mode`)은 [config-reference](https://developers.openai.com/codex/config-reference)에 default가 명시되어 있지 않아 **방어적으로 pinning**한다. default가 명시된 항목(`project_doc_max_bytes`, `[agents]` 등)과 대비된다.

```toml
# Codex CLI project config — see ADR-010
# References:
#   https://developers.openai.com/codex/config-reference
#   https://developers.openai.com/codex/config-advanced

# 모델: gpt-5.5는 config-reference의 공식 예시 값. 현행 권장 모델로 단정하지 말고
# 사용 계정에서 실제 접근 가능한지 사전 점검(Step 0-1)에서 1회 확인 후 박는다.
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

# glob_scan_max_depth는 의도적으로 박지 않음 — config-advanced에 default 미명시 + 작은 값을
# 박으면 모노레포(packages/*/.env 등) 깊이에서 보안 baseline 회귀 위험. 사용자가 깊이 제어
# 필요해진 시점에 ~/.codex/config.toml에서 override한다.

# [permissions.boilerplate-secure.network] 블록은 의도적으로 두지 않음.
# Claude side baseline(.claude/settings.json)이 네트워크를 막지 않으므로 ADR-010
# 도구 표면 매핑 표의 "양쪽 동시 — 동일 결과" 약속을 지키려면 Codex도 막지 않아야 함.
# 네트워크 정책은 Codex sandbox_mode와 사용자 ~/.codex/config.toml 책임.
```

### 5-3. Step 0-1·0-2 결과에 따른 분기

- **0-1에서 `gpt-5.5` 접근 불가**였으면 `model = "..."` 값을 대체 ID로 교체.
- **0-2-2에서 `.` 비상속 발견**이었으면 §5-2의 `":project_roots"` inline-table을 다음 패턴으로 **교체**한다:

  ```toml
  [permissions.boilerplate-secure.filesystem]
  ":project_roots" = { "**" = "write", ".claude/skills/**" = "read", ".env" = "none", ".env.*" = "none", "**/.env" = "none", "**/.env.*" = "none", "secrets/**" = "none", "**/secrets/**" = "none" }
  # glob_scan_max_depth는 의도적으로 박지 않음 (§5-2 동일 사유 — default 미명시 + 모노레포 회귀 위험)
  ```

  설계 의도:
  - `"**" = "write"` — 트리 전체에 기본 write를 부여해 보일러플레이트 fork 후 사용자가 `src/`, `tests/`, `scripts/` 등 어떤 디렉터리에서 코딩해도 막히지 않도록 한다.
  - `".claude/skills/**" = "read"` — D3에 의해 `.claude/skills/<name>/SKILL.md`는 canonical SSOT다. Codex wrapper가 본문을 read만 하면 충분하고, write는 SSOT를 깨뜨릴 위험이 있어 명시 차단.
  - `**/.env*`, `**/secrets/**` — 보안 baseline.
  - **`"**" = "write"`가 트리 전체로 적용되는지도 1회 실측 확인** (Codex glob 구현 가정). 작동하지 않으면 명시 디렉터리(`"src/**" = "write"`, `"tests/**" = "write"`, ...)를 나열하는 형태로 다시 fallback.

### 5-4. 의도적으로 박지 않은 항목 (메모)

다음은 **upstream 기본값**이라 pinning 시 stale 위험. 박지 않는다 — 사용자가 `~/.codex/config.toml`에서 개인적으로 override 가능.

- `project_doc_max_bytes` (기본 32 KiB)
- `[agents] max_threads`, `max_depth` (기본 6, 1)
- `[features] codex_hooks` (비범위)

### 5-5. 적용 후 실측 (Step 0-2-3)

`.codex/config.toml`을 박은 직후 Codex CLI가 실제로 차단하는지 확인. **두 방법 중 어느 쪽이라도 차단 확인되면 통과**.

```text
# 방법 A — /permissions 슬래시 커맨드
$ codex
> /permissions
# 출력에 .env*, secrets/** 등이 deny/none으로 표시되는지 확인

# 방법 B — fallback (방법 A가 없거나 결과가 불명확할 때 — 더 직접적인 검증)
$ codex
> "프로젝트 루트의 .env 파일 내용을 읽어줘"
# Codex가 거부하거나 sandbox 차단 메시지를 반환하면 통과

> "secrets/ 디렉터리 안 파일을 보여줘"
# 같은 결과 — 차단되어야 통과
```

차단되지 않으면 스키마 또는 `.` 상속 가정이 틀린 것 — 0-2로 돌아가 재점검 + 템플릿 수정. trusted project 점검(0-3)도 다시 확인 — untrusted면 `.codex/config.toml` 자체가 로드되지 않을 수 있다.

### 5-6. Step 5 완료 체크

- [ ] `.codex/config.toml` 존재.
- [ ] sandbox/approval/model 박힘.
- [ ] `boilerplate-secure` permissions 프로파일 박힘.
- [ ] `.env`/`secrets/**` 차단 실측 확인.
- [ ] TODO·미검증 키 잔존 0건.

### 5-7. 커밋

```text
chore(codex): add .codex/config.toml with secure baseline permissions
```

---

## Step 6 — `.agents/skills/` wrapper 4개 신설 (P1-6)

> 권장 에이전트: **builder-sonnet** — 정해진 본문 박기.
> Step 5와 독립 — 동시 진행 가능.

### 6-1. 4개 대상

- `implement-workitem`
- `validate-workitem`
- `repair-workitem`
- `finalize-workitem`

> **왜 이 4개만?** Inner loop은 task마다 반복되므로 wrapper 가성비가 가장 높다. bootstrap/plan/discover 등은 회당 1회라 자연어 호출로 충분 (Phase 2로 미룸).

### 6-2. 디렉터리 구조

각 skill마다 폴더 1개 + 파일 2개 = 총 4 폴더 / 8 파일.

```
.agents/skills/<skill-name>/
├── SKILL.md
└── agents/openai.yaml
```

`<skill-name>`은 위 4개 중 하나.

### 6-3. `SKILL.md` 본문 (4개 모두 동일 패턴, `<skill-name>`만 교체)

각 4개 폴더의 `SKILL.md`를 다음 양식으로 작성:

```markdown
---
name: <skill-name>
description: Use ONLY when the user explicitly types `$<skill-name> <task-id>`. Do not trigger implicitly from generic phrasing.
---

Source of truth: `.claude/skills/<skill-name>/SKILL.md`. Read it and follow the workflow.

Treat all frontmatter keys other than `name` and `description` (e.g., `agent:`, `disable-model-invocation:`, `allowed-tools:`, `context:`, `argument-hint:`, `model:`, `effort:`) as Claude-only and ignore them — execute locally in Codex.

**Slash command translation**: 본문 안의 `/<skill-name>` 표기는 Claude 슬래시 커맨드다. Codex에서는 `$<skill-name>`으로 읽고 사용자에게 안내한다 (예: 본문 "다음 단계: `/finalize-workitem T-001`" → Codex 응답에서는 "다음 단계: `$finalize-workitem T-001`"). Codex CLI는 `/`를 빌트인 슬래시 커맨드에 쓰므로 명시적 치환이 필요.

Preserve all repo policies from `AGENTS.md` and `docs/`.

If the source path no longer exists, this wrapper is stale — see ADR-010.
```

`<skill-name>` 자리에 `implement-workitem` / `validate-workitem` / `repair-workitem` / `finalize-workitem`를 각각 박는다.

### 6-4. `agents/openai.yaml` 본문 (4개 모두 동일)

각 4개 폴더의 `agents/openai.yaml`을 다음 양식으로 작성:

```yaml
policy:
  allow_implicit_invocation: false
```

### 6-5. 왜 두 단계 모두?

Codex Skills는 description 매칭으로 implicit invocation이 default-on이다 ([developers.openai.com/codex/skills](https://developers.openai.com/codex/skills)). Claude의 `disable-model-invocation: true`에 해당하는 functional equivalent는 **`agents/openai.yaml`의 `policy.allow_implicit_invocation: false`** — 이게 metadata-level 명시 차단이고 1차 방어. description 좁히기는 보조(2차 방어)로 유지. 두 단계 모두 두는 게 안전.

`agents/openai.yaml` 파일이 없으면 `allow_implicit_invocation`은 default `true`. 누락 시 Codex가 `$implement-workitem` 같은 wrapper를 generic prompt("이 task 좀 구현해줘")에서 자동 트리거할 수 있음.

### 6-6. 만들어야 할 파일 8개 정리

| # | 파일 |
|---|---|
| 1 | `.agents/skills/implement-workitem/SKILL.md` |
| 2 | `.agents/skills/implement-workitem/agents/openai.yaml` |
| 3 | `.agents/skills/validate-workitem/SKILL.md` |
| 4 | `.agents/skills/validate-workitem/agents/openai.yaml` |
| 5 | `.agents/skills/repair-workitem/SKILL.md` |
| 6 | `.agents/skills/repair-workitem/agents/openai.yaml` |
| 7 | `.agents/skills/finalize-workitem/SKILL.md` |
| 8 | `.agents/skills/finalize-workitem/agents/openai.yaml` |

### 6-7. 적용 후 실측

```text
# Codex CLI 진입 후
> /skills
# 4개 wrapper가 목록에 보이는지 확인

> $implement-workitem T-001
# wrapper가 .claude/skills/implement-workitem/SKILL.md를 read하고 따라 실행하는지 확인
```

또한 generic prompt("이 task 좀 구현해줘")에서 4개 wrapper가 **자동 트리거되지 않는지** 확인 — `agents/openai.yaml`의 `policy.allow_implicit_invocation: false`가 4개 모두에 있으면 안 트리거되는 게 정상.

### 6-8. Step 6 완료 체크

- [ ] 4 폴더 / 8 파일 존재.
- [ ] 각 `SKILL.md`의 frontmatter `name` 필드가 폴더명과 일치.
- [ ] 각 `SKILL.md`의 description이 `Use ONLY when the user explicitly types $<skill-name> <task-id>`로 시작.
- [ ] 각 `agents/openai.yaml`에 `policy.allow_implicit_invocation: false` 박힘.
- [ ] Codex `/skills` 목록에 4개 모두 표시.
- [ ] `$implement-workitem T-001`이 SSOT 본문(`.claude/skills/implement-workitem/SKILL.md`)을 read 가능.
- [ ] generic prompt에서 4개 wrapper가 자동 트리거되지 않음.

### 6-9. 커밋

```text
feat(codex): add .agents/skills wrappers for inner-loop workitem skills
```

---

## Step 7 — README 두 종 갱신 (P1-7)

> 권장 에이전트: **builder-sonnet** — 정해진 단락 추가.
> Step 1, 3 완료 후 진행 (AGENTS.md, ADR-010이 존재해야 링크 유효).

### 7-1. 추가 위치

`README.md`와 `README_ko.md` **모두** 동일 단락을 추가한다 (drift 방지). 두 README의 섹션명이 다르므로 위치를 명시 분리:

- `README.md` (영문): `## Quick Start` 섹션이 끝난 직후, `## Structure` 직전.
- `README_ko.md` (한국어): `## 빠른 시작` 섹션이 끝난 직후, `## 구조` 직전.

### 7-2. `README.md`에 추가할 단락 (영문)

```markdown
## Using with Codex CLI (alternate entry)

When you hit Claude Code's usage limit or prefer Codex:

1. Run `codex` in the same repo — `AGENTS.md` is auto-loaded.
2. Same workflow applies: see [WORKFLOW.md](docs/00-meta/WORKFLOW.md).
3. Inner-loop skills are callable via Codex Skills:
   - `$implement-workitem T-001`
   - `$validate-workitem T-001`
   - `$repair-workitem T-001`
   - `$finalize-workitem T-001`
4. For other steps, invoke in natural language: *"Follow `.claude/skills/bootstrap-project/SKILL.md`"*.

> Note: docs in `docs/` use Claude's `/<skill-name>` slash syntax. Read these as `$<skill-name>` when working in Codex.

For full policy, see [ADR-010](docs/90-decisions/ADR-010-multi-agent-compatibility.md).
```

### 7-3. `README_ko.md`에 추가할 단락 (한국어)

```markdown
## Codex CLI에서 사용하기 (대체 진입점)

Claude Code 한도에 걸리거나 Codex를 선호할 때:

1. 같은 저장소에서 `codex` 실행 — `AGENTS.md`가 자동 로드된다.
2. 워크플로우는 동일: [WORKFLOW.md](docs/00-meta/WORKFLOW.md) 참조.
3. 자주 쓰는 inner-loop는 Codex skill로 호출 가능:
   - `$implement-workitem T-001`
   - `$validate-workitem T-001`
   - `$repair-workitem T-001`
   - `$finalize-workitem T-001`
4. 그 외 단계는 자연어로 호출: *"Follow `.claude/skills/bootstrap-project/SKILL.md`"*

> 참고: `docs/` 하위 문서는 Claude의 `/<skill-name>` 슬래시 표기를 사용한다. Codex에서는 `$<skill-name>`으로 읽는다.

자세한 정책은 [ADR-010](docs/90-decisions/ADR-010-multi-agent-compatibility.md).
```

### 7-4. Structure 트리 in-place 갱신 (양쪽 README)

본 작업으로 `AGENTS.md`가 신설되고 `CLAUDE.md`는 `@AGENTS.md` import 한 줄이 된다. 두 README의 `## Structure` / `## 구조` 트리도 이를 반영해야 함 (README.md 상단 HTML 주석 룰 — *"구조 변경 시 README.md와 README_ko.md를 동시에 갱신한다"*).

**`README.md` `## Structure` 트리** — `CLAUDE.md` 줄을 다음 2줄로 교체:

```text
├── AGENTS.md          # Canonical entry instructions (tool-neutral)
├── CLAUDE.md          # Imports AGENTS.md (Claude Code entry)
```

**`README_ko.md` `## 구조` 트리** — `CLAUDE.md` 줄을 다음 2줄로 교체:

```text
├── AGENTS.md          # 캐노니컬 진입 지침 (도구 중립)
├── CLAUDE.md          # AGENTS.md를 import (Claude Code 진입점)
```

### 7-5. Step 7 완료 체크

- [ ] `README.md`, `README_ko.md` **둘 다** 동일 위치(Quick Start 직후)에 새 섹션 추가.
- [ ] 두 README가 같은 정보를 다른 언어로 표현 (drift 0).
- [ ] ADR-010 링크와 WORKFLOW.md 링크 모두 깨지지 않음.
- [ ] 두 README의 Structure 트리에 `AGENTS.md` 행 추가 + `CLAUDE.md` 코멘트 갱신.

### 7-6. 커밋

```text
docs: document Codex CLI as alternate entry in READMEs
```

---

## Step 8 — 최종 검증 (Phase 1 완료 체크리스트)

> 권장 에이전트: **validator-sonnet** — 체크리스트 점검.
> 옵션: **reviewer**로 ADR-010 ↔ STRUCTURE.md drift 점검.

### 8-1. Phase 1 완료 체크리스트 (필수, 모두 통과해야 Step 9 진행)

- [ ] `AGENTS.md`가 200줄 이하 / 32 KiB 미만, **도구 종속 표현이 모두 도구 중립으로 재작성됨** ("Claude Code 기반" 등 잔재 0건).
- [ ] `AGENTS.md`의 `docs/` 상대 링크 총 8개가 모두 깨지지 않음 (00-meta 5 + 90-decisions ADR 인덱스 1 + 단순성/TDD 단락 ADR 본문 2).
- [ ] `CLAUDE.md`가 `@AGENTS.md` import 형태 (Claude-only 지침이 없으면 한 줄, 있으면 `## Claude Code` 섹션 추가).
- [ ] `claude` 실행 후 `/memory`에 `CLAUDE.md`가 listed되고, 그 파일 본문에 `@AGENTS.md` import 라인이 있음. **추가로 모델에게 AGENTS.md 고유 문장 한 줄 회수시켜 본문 주입 확인** (`/memory`는 import body를 별도로 listing하지 않으므로 이 회수 단계가 필수).
- [ ] `codex` 실행 후 AGENTS.md 자동 로드 (별도 명령 불필요). 이 repo가 trusted project로 등록되어 `.codex/config.toml`도 함께 로드됨이 확인됨 (Step 0-3 통과).
- [ ] Codex에서 `$implement-workitem` 호출 시 wrapper가 발견되고 `.claude/skills/` 본문을 read해서 따라 실행됨 (permissions가 SSOT 본문 read를 막지 않음을 동시 검증).
- [ ] **Implicit invocation 차단 실측**: generic prompt(예: "이 task 좀 구현해줘")에서 4개 wrapper(`implement/validate/repair/finalize-workitem`)가 자동 트리거되지 않음. `agents/openai.yaml`의 `policy.allow_implicit_invocation: false`가 4개 wrapper 모두에 박혀 있음.
- [ ] ADR-010이 인덱스에 등록되고 STRUCTURE.md Canonical Owner 표에 매핑이 한 줄 추가됨.
- [ ] **기존 surface stale 갱신 완료**(Step 4의 4-4 ~ 4-8) — `grep -rn "CLAUDE.md에" docs/ .claude/`가 0건 (또는 의도적 잔존만). ADR-005 패턴 5, ADR-006 §결과, ADR-009 §결과, STRUCTURE.md 단순성/TDD 행, bootstrap-project SKILL.md 모두 `AGENTS.md` 표기로 일관됨.
- [ ] **README Structure 트리 갱신**(Step 7-4) — `README.md`, `README_ko.md` 둘 다 트리에 `AGENTS.md` 행이 추가되고 `CLAUDE.md` 코멘트가 import 형태로 갱신됨.
- [ ] `.codex/config.toml`의 sandbox/approval이 적용되고, **`.env`/`secrets/**` 접근이 Codex 측에서 실측으로 차단됨** (TODO·미검증 키 잔존 0건).

### 8-2. 회귀 검증

- [ ] 기존 `/bootstrap-project`, `/implement-workitem` 등 Claude Code 슬래시 커맨드가 그대로 동작.
- [ ] 기존 sub-agent(architect-opus, builder-sonnet 등) 위임이 그대로 동작.
- [ ] `docs/` 어떤 문서도 깨진 링크 없음.

### 8-3. 비범위 누락 확인 (만들지 말아야 할 게 안 만들어졌나)

- [ ] `.codex/agents/*.toml` 파일이 생성되지 않았는지 (Phase 3 비범위).
- [ ] `docs/00-meta/AGENT_TOOL_MATRIX.md` 별도 파일이 생성되지 않았는지 (D7 비범위).
- [ ] `.agents/skills/` wrapper가 4개(폴더) / 8개(파일)를 초과하지 않는지.
- [ ] `~/.codex/AGENTS.md` 같은 사용자 글로벌 파일이 본 작업으로 만들어지지 않았는지.

### 8-4. 통과 못한 항목 처리

- §8-1 항목 미통과 → 해당 Step으로 돌아가 수정 + 새 커밋. 절대 amend 사용 금지 (ADR-008).
- §8-2 회귀 발견 → 회귀 일으킨 변경을 식별 + 후속 커밋으로 수정.
- §8-3 비범위 위반 발견 → 위반 산출물 삭제 + 후속 커밋.

**커밋**: 없음 (검증만). 통과 못해서 수정할 때만 새 커밋.

---

## Step 9 — 폐기 절차 (DRAFT + IMPROVE-GUIDE 삭제)

### 9-1. 조건 확인

다음을 **모두** 충족하면 본 가이드와 DRAFT를 삭제할 수 있다:

- [ ] ADR-010이 accepted 상태로 머지됨.
- [ ] Step 1 ~ Step 7의 모든 산출물이 머지됨 (커밋 7개).
- [ ] Step 8의 §8-1, §8-2, §8-3 체크리스트가 모두 통과됨.

### 9-2. 삭제 + 커밋 (한 흐름)

삭제 대상:
- `DRAFT-IMPROVE-GUIDE.md` — 의사결정 메모 모음.
- `IMPROVE-GUIDE.md` — 본 실행 가이드.

`git rm` 후 `git commit`을 이어서 한 번에 실행한다 — `git rm` 단독 실행으로 끝나지 않음에 주의:

```powershell
# PowerShell (PS 7+ 또는 Bash와 동일한 && 사용)
git rm DRAFT-IMPROVE-GUIDE.md IMPROVE-GUIDE.md
git commit -m "chore: remove improvement guide drafts after multi-agent migration"
```

```bash
# Bash
git rm DRAFT-IMPROVE-GUIDE.md IMPROVE-GUIDE.md && \
  git commit -m "chore: remove improvement guide drafts after multi-agent migration"
```

### 9-3. 삭제 후 어디에 정보가 남는가

| 정보 | 보존 위치 |
|---|---|
| 결정과 근거 | `docs/90-decisions/ADR-010-multi-agent-compatibility.md` |
| 산출물 위치 | `docs/00-meta/STRUCTURE.md` (산출물 표 + Canonical Owner 매핑) |
| 도구 표면 매핑 | `docs/90-decisions/ADR-010-multi-agent-compatibility.md` 본문 |
| 사용 안내 | `README.md`, `README_ko.md`의 "Codex CLI" 단락 |
| 진입 본문 | `AGENTS.md` |

### 9-4. 커밋 (§9-2에서 함께 실행됨)

§9-2 명령에 commit이 포함되어 있다. 메시지는 다음 1줄(영문, Conventional Commits, ADR-008):

```text
chore: remove improvement guide drafts after multi-agent migration
```

---

## 부록 A — 참고 출처 (1차 공식)

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
- [Config advanced](https://developers.openai.com/codex/config-advanced)
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
- [ADR-008 Conventional Commits](docs/90-decisions/ADR-008-commit-convention.md)
- [STRUCTURE.md 산출물 인벤토리](docs/00-meta/STRUCTURE.md)
- [AGENT_EXECUTION_STRATEGY.md](docs/00-meta/AGENT_EXECUTION_STRATEGY.md)
- [GUARDRAILS_STRATEGY.md](docs/00-meta/GUARDRAILS_STRATEGY.md)
- [DRAFT-IMPROVE-GUIDE.md](DRAFT-IMPROVE-GUIDE.md) — 의사결정 메모 (작업 끝나면 삭제)

---

## 부록 B — 검증된 사실 (Step 0 점검 시 참조)

> 본 가이드의 모든 결정은 아래 사실에 기반한다. 추측이나 third-party 블로그 인용은 사용하지 않는다.

### B-1. Claude Code 측 사실

| 사실 | 출처 |
|---|---|
| Claude Code는 `CLAUDE.md`만 읽고 `AGENTS.md`는 **읽지 않는다** | [code.claude.com/docs/en/memory](https://code.claude.com/docs/en/memory) — *"Claude Code reads `CLAUDE.md`, not `AGENTS.md`."* |
| AGENTS.md를 공유하려면 `CLAUDE.md`에서 `@AGENTS.md` import 패턴이 공식 권장 | 같은 페이지에 권장 예시로 박혀 있음 |
| `@path/to/file` 문법은 공식 — 상대/절대 경로, 5단계 재귀 가능, launch 시 인라인 로드 | 같은 페이지 |
| `CLAUDE.md`는 200줄 이하 권장 | 같은 페이지 |
| `.claude/` 위치도 유효 (`./.claude/CLAUDE.md`) | 같은 페이지 |

### B-2. Codex CLI 측 사실

| 사실 | 출처 |
|---|---|
| `AGENTS.md`는 프로젝트 루트에서 시작해 cwd까지 경로상 각 디렉터리에서 자동 로드(루트 → cwd walking). 각 디렉터리에서 `AGENTS.override.md`가 `AGENTS.md`보다 우선. cwd 도달 시 검색 종료 | [developers.openai.com/codex/guides/agents-md](https://developers.openai.com/codex/guides/agents-md) |
| `~/.codex/AGENTS.md` 등 글로벌 자동 로드 여부는 본 가이드 작성 시점 1차 출처에서 미명시 — 사용 시 별도 검증 (※ 검증된 사실 표에 남기되 단정 금지) | (미검증) |
| `project_doc_max_bytes` 기본값 32 KiB | [agents-md guide](https://developers.openai.com/codex/guides/agents-md) |
| 프로젝트 설정은 `.codex/config.toml`, 사용자 설정은 `~/.codex/config.toml` | [config-basic](https://developers.openai.com/codex/config-basic), [config-reference](https://developers.openai.com/codex/config-reference) |
| 모델 필드 공식 예시는 `gpt-5.5` | [config-reference](https://developers.openai.com/codex/config-reference) — *"Model to use (e.g., `gpt-5.5`)"* |
| Skills 위치: 프로젝트는 `.agents/skills/`, 사용자는 `~/.agents/skills/` | [developers.openai.com/codex/skills](https://developers.openai.com/codex/skills) |
| Skill 호출 공식 문법: **`$skill-name`** 또는 `/skills` (점·괄호 변형은 공식 X) | 같은 페이지 — *"In CLI/IDE, run `/skills` or type `$` to mention a skill"* |
| Custom subagents: `.codex/agents/<name>.toml`. 명시 호출 시에만 spawn | [developers.openai.com/codex/subagents](https://developers.openai.com/codex/subagents) |
| `[agents]` 기본값: `max_threads=6`, `max_depth=1` | 같은 페이지 |
| Hooks 본 작업 비범위. 활성화 조건과 default 상태는 활성화 시점에 [hooks 문서](https://developers.openai.com/codex/hooks) 1차 출처로 재확인 (정책·기본값이 변동될 수 있음) | [developers.openai.com/codex/hooks](https://developers.openai.com/codex/hooks) |
| Custom Prompts(`~/.codex/prompts/`)는 deprecated — Skills로 대체 권장 | [developers.openai.com/codex/custom-prompts](https://developers.openai.com/codex/custom-prompts) |
| Codex에는 Claude의 `opus`/`sonnet` 같은 모델 별칭 체계 없음 | config-reference 문서 분석 결과 |

### B-3. permissions 프로파일 스키마 (config-advanced 직접 확인)

`.codex/config.toml`의 permissions 프로파일은 **`[permissions.<name>.filesystem]` 중첩 테이블 헤더 + `":project_roots"` 키에 inline-table 값** 형태다. 차단은 deny 배열이 아니라 glob 패턴 키 → 모드 매핑(`"none"` / `"read"` / `"write"`).

```toml
default_permissions = "<name>"  # 활성화 필수

[permissions.<name>.filesystem]
":project_roots" = { "." = "write", "**/.env" = "none", "**/secrets/**" = "none" }
glob_scan_max_depth = 3   # config-advanced에 등장하는 키 — 본 가이드는 §5-2에서 모노레포 보안 회귀 위험 때문에 의도적으로 박지 않음
```

- 빌트인 프로파일은 leading colon (`:read-only`, `:workspace`, `:danger-no-sandbox`), 커스텀 이름은 colon 없음.
- 더 구체적인 glob이 우선 (통상 glob 매칭 규칙 가정 — config-advanced에 명시 진술 없음, 적용 시 1회 실측 권장). `.` = "write"가 깔려 있어도 `**/.env` = "none"이 우선되도록 보안 baseline은 중복 패턴으로 명시한다.
- 출처: [config-advanced](https://developers.openai.com/codex/config-advanced) (config-basic은 빌트인 3종만 보여주고 상세 스키마는 advanced로 위임).

---

## 부록 C — Phase 2 / Phase 3 (이번 작업 범위 밖, 향후 참고)

> Phase 1 완료 후 Codex 사용 빈도가 자리잡으면 그때 진행. 본 가이드의 책임은 Phase 1까지.

### Phase 2 — 추가 wrapper

| ID | 작업 | 산출물 |
|---|---|---|
| P2-1 | workflow wrapper 6개 추가 | `.agents/skills/{discover-product,bootstrap-project,bootstrap-stack,stack-guard,plan-workitem,stabilize-milestone}/SKILL.md` |
| P2-2 (선택) | 유틸리티 wrapper 2개 — 실제 필요해진 뒤에만 | `.agents/skills/{boilerplate-context,review-doc}/SKILL.md` |

### Phase 3 — Custom subagents

| ID | 작업 | 산출물 |
|---|---|---|
| P3-1 | 최소 3개 custom agent 포팅 | `.codex/agents/{builder,validator,reviewer}.toml` |
| P3-2 | 나머지 3개 (필요 시) | `.codex/agents/{architect,planner,qa}.toml` |
