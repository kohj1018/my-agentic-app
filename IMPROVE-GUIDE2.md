<!--
이 문서는 IMPROVE-GUIDE.md의 Phase 1 wrapper(implement/validate/repair/finalize) 위에
"core workflow wrapper 확장"(Phase 1.5)을 추가하는 실행 가이드다.

처음 합류한 사람은 IMPROVE-GUIDE.md → 본 IMPROVE-GUIDE2.md 순서로 따라간다.
DRAFT-IMPROVE-GUIDE.md는 Phase 1 의사결정 메모 — 본 가이드의 결정 근거는 §1과 부록 A에 있다.

본 가이드는 ephemeral 문서다. Phase 1.5 머지 + §3 체크리스트 통과 후 사용자가 직접 삭제한다.
결정과 근거는 ADR-010이, 산출물 위치는 STRUCTURE.md가 영속적으로 보존한다.
-->

# 보일러플레이트 다중 에이전트 호환 개선 — 실행 가이드 (Phase 1.5: core workflow wrapper)

## 0. 메타정보

| 항목 | 값 |
|---|---|
| 상태 | ready (Phase 1 머지 후 또는 병렬 진행) |
| 작성일 | 2026-05-07 |
| 폐기 시점 | Phase 1.5 산출물 머지 + §3 체크리스트 통과 후 (사용자가 직접 삭제) |
| 관련 ADR | ADR-005 (SSOT), ADR-006 (단순성), ADR-008 (Conventional Commits), ADR-010 (다중 에이전트 호환) |
| 영향받는 문서 | `README.md`, `README_ko.md`, `docs/90-decisions/ADR-010-multi-agent-compatibility.md` |
| 신설되는 디렉터리/파일 | `.agents/skills/{plan-workitem,bootstrap-project,bootstrap-stack,stabilize-milestone}/{SKILL.md, agents/openai.yaml}` (총 4 폴더 / 8 파일) |
| 선행 가이드 | [IMPROVE-GUIDE.md](IMPROVE-GUIDE.md) — Phase 1 inner-loop wrapper 4개 |

---

## 1. 시작하기 전에 (Phase 1.5 도입 근거)

### 1-1. Phase 1만으로 부족한 시나리오

IMPROVE-GUIDE.md Phase 1은 **inner-loop 4개 wrapper**(implement/validate/repair/finalize)만 박는다. 근거(DRAFT §6-6): *"inner loop은 task마다 반복되므로 wrapper 가성비가 가장 높다. bootstrap/plan/discover 등은 회당 1회라 자연어 호출로 충분."*

이 가정은 **이미 진행 중인 프로젝트**에서는 맞다. 하지만 **사용자가 보일러플레이트 fork 직후부터 Codex로 작업을 시작**하는 시나리오에서는:

README 워크플로우(Quick Start Step 0~3)의 첫 진입 순서는
`/discover-product → /bootstrap-project → /bootstrap-stack → /stack-guard → /plan-workitem → /implement-workitem → ...`

(ADR-007의 8단계 라이프사이클은 더 큰 추상화 — `discover/bootstrap/plan/implement/validate/repair/finalize/stabilize`. README 워크플로우는 그 안의 실제 skill 호출 시퀀스.)

이 중 Phase 1 wrapper가 커버하는 건 `/implement-workitem` 이후 4개뿐. 그 앞 5개 skill은 모두 자연어 호출 (`"Follow .claude/skills/bootstrap-project/SKILL.md"`)에 의존한다.

자연어 호출의 실측 비용:
- 사용자 입력 길이 ↑ (`/bootstrap-project ...` vs `Follow .claude/skills/bootstrap-project/SKILL.md ...`)
- description 매칭 흔들리면 Codex가 다른 일 시작 가능
- Codex `/skills` 목록에 안 보임 → discoverability 0
- "회당 1회"여도 그 1회가 새 프로젝트의 가장 중요한 첫 발화점

### 1-2. Phase 1.5 범위 결정

빈도 × wrapper 가성비 매트릭스로 재평가하면, **Phase 2의 8개 wrapper 중 4개가 Phase 1과 동등하거나 그에 가까운 가성비**를 가진다:

| Skill | 빈도 | wrapper 가치 | 분류 |
|---|---|---|---|
| `plan-workitem` | 분해 시점마다 (자주) | 높음 | **Phase 1.5 ⬆️** |
| `bootstrap-project` | 프로젝트 시작 1회 (중요) | 중간 | **Phase 1.5 ⬆️** |
| `bootstrap-stack` | 스택 변경 1회 | 중간 | **Phase 1.5 ⬆️** |
| `stabilize-milestone` | 마일스톤마다 1회 | 중간 | **Phase 1.5 ⬆️** |
| `discover-product` | 발굴 1회 (선택적) | 낮음 | Phase 2 보류 |
| `stack-guard` | 스택 확정 직후 1회 | 낮음 | Phase 2 보류 |
| `review-doc` | 필요 시 (드물게) | 낮음 | Phase 2 보류 |
| `boilerplate-context` | 컨텍스트 인식용 (드물게) | 매우 낮음 | Phase 2 보류 |

선별 기준 4축:
1. **fork 직후 첫 진입점인가** — bootstrap-project가 가장 강함.
2. **분해 단계마다 호출되는가** — plan-workitem.
3. **워크아이템 라이프사이클의 분기점인가** — bootstrap-stack(스택 진입), stabilize-milestone(마일스톤 마감).
4. **자연어 호출이 description 매칭 흔들림 위험이 큰가** — Phase 2 보류 4개는 호출 자체가 드물어 매칭 흔들림이 발생할 빈도도 낮음.

### 1-3. Phase 3는 그대로 보류 (재확인)

[Codex subagents](https://developers.openai.com/codex/subagents) 제약:
- **명시 호출 시에만 spawn** — Claude의 자동 위임 트리거(`implement → builder-sonnet`) 그대로 옮길 수 없음.
- 빌트인은 worker/explorer/default 3개만 — 보일러플레이트 6개 specialized agent와 1:1 매칭 불가.

`.codex/agents/<name>.toml` 미러링해도 자동 위임 패턴이 깨지므로 사용자가 매번 명시 호출. 이건 자연어 호출(`"use builder agent for X"`)로 이미 가능. **SSOT 본문 형식 차이(.md vs .toml)에 따른 drift 비용이 가치 대비 크다** — DRAFT D6 판단 그대로 유효. Phase 1.5에서도 보류 유지.

### 1-4. 새 ADR 불필요

본 작업은 ADR-010 본문의 의사결정 범위 안이다 (§결정 D4: ".agents/skills/<name>/SKILL.md는 얇은 wrapper만"). Phase 1과 Phase 1.5는 **같은 결정의 단계적 적용**일 뿐 새 정책이 아니다. ADR-010의 "후속 작업" 섹션 중 *"Phase 2 workflow wrapper 6개 ... 사용 빈도에 따라 선별"* 메모를 in-place 갱신하는 것으로 추적성 확보.

---

## 2. 진행 방식 안내

### 2-1. 단계 구성

본 가이드는 3개 단계로 구성된다.

| 단계 | 이름 | 주요 산출물 | 커밋 |
|---|---|---|---|
| Step 1 | Phase 1.5 wrapper 4개 신설 (한 묶음) | `.agents/skills/{plan-workitem,bootstrap-project,bootstrap-stack,stabilize-milestone}/{SKILL.md, agents/openai.yaml}` (8 파일) | 1개 |
| Step 2 | README 두 종 + ADR-010 후속작업 갱신 | `README.md`, `README_ko.md`, `docs/90-decisions/ADR-010-multi-agent-compatibility.md` | 1개 |
| Step 3 | 최종 검증 (체크리스트) | (검증 메모) | (없음) |

### 2-2. 의존 관계

```
IMPROVE-GUIDE.md Phase 1 (inner-loop 4 wrapper) ─┐
                                                 │
                                                 ├─→ Step 1 (Phase 1.5 wrapper 4개)
                                                 │   ※ 독립 — Phase 1 미완료여도 시작 가능,
                                                 │     단 Step 2는 Phase 1의 README/ADR-010이 머지된 후
                                                 │
Step 1 ──→ Step 2 (README + ADR-010) ──→ Step 3 (검증)
```

처음 따라가는 사람은 **Phase 1 (IMPROVE-GUIDE.md) 완전 머지 후 본 가이드를 시작**하는 게 가장 안전하다. README·ADR-010이 Phase 1 갱신 상태여야 Step 2의 in-place 수정이 정확.

### 2-3. 위임 권고

- Step 1, 2: **builder-sonnet** — IMPROVE-GUIDE.md Step 6과 동일 패턴 (정해진 본문 박기 + 표 행 갱신).
- Step 3: **validator-sonnet** — §3 체크리스트.
- Step 3 보강 (선택): **reviewer** — Phase 1.5 추가가 ADR-010 본문(특히 §결과·후속작업)과 모순 없는지.

### 2-4. 커밋 규칙

IMPROVE-GUIDE.md §2-4와 동일.
- 영문 1줄, Conventional Commits (ADR-008).
- `git add -A` / `git add .` 금지. 명시 파일만 stage.
- `--amend`, `--no-verify`, `git push` 금지.

---

## 3. 비범위 (Phase 1.5에서도 하지 않는 것)

다음은 본 가이드 범위 **밖**이다.

- [ ] `discover-product`, `stack-guard`, `review-doc`, `boilerplate-context` 4개 wrapper. 호출 빈도가 낮아 자연어 호출로 충분 (Phase 2 보류 유지).
- [ ] `.codex/agents/*.toml` custom subagents 미러링. ADR-010 D6 + §1-3 판단 그대로 유지 (Phase 3 보류).
- [ ] 새 ADR 신설. 본 작업은 ADR-010 결정 범위 안의 단계적 적용이라 신설 ADR 없음.
- [ ] `.codex/config.toml` 보강. Phase 1에서 박은 보안 baseline이 그대로 유효.
- [ ] `.claude/skills/` 본문 수정. SSOT (D3)는 그대로.

---

## Step 1 — Phase 1.5 wrapper 4개 신설

> 권장 에이전트: **builder-sonnet** — IMPROVE-GUIDE.md Step 6 패턴 그대로 적용.
> 4개를 한 커밋에 묶는다 — 같은 패턴의 동시 적용이라 분리 가치 낮음 (단순성).

### 1-1. 4개 대상 + 각 description 표

각 wrapper는 `.claude/skills/<name>/SKILL.md`의 `argument-hint` 필드를 따라 인자 형태가 다르다. description에 정확한 인자 표기를 박아 implicit invocation 보조 차단을 강화한다.

| # | wrapper | argument-hint (Claude SSOT) | Codex description 인자 표기 |
|---|---|---|---|
| 1 | `plan-workitem` | `[milestone or feature id]` | `$plan-workitem <milestone-or-feature-id>` |
| 2 | `bootstrap-project` | `[project brief or empty (uses DISCOVERY.md)] [--apply]` | `$bootstrap-project <brief-or-empty> [--apply]` |
| 3 | `bootstrap-stack` | `[stack and runtime summary]` | `$bootstrap-stack <stack-and-runtime-summary>` |
| 4 | `stabilize-milestone` | `[milestone id]` | `$stabilize-milestone <milestone-id>` |

> **참고**: 각 skill의 정확한 argument-hint는 `.claude/skills/<name>/SKILL.md` frontmatter에서 확인 (위 표는 본 가이드 작성 시점에 SSOT를 그대로 옮긴 것). 변동 시 `.claude/skills/`가 SSOT — 작업 직전 한 번 grep으로 sync 점검 권장:
>
> ```bash
> grep -h "argument-hint" .claude/skills/{plan-workitem,bootstrap-project,bootstrap-stack,stabilize-milestone}/SKILL.md
> ```

### 1-2. 디렉터리 구조 (Phase 1과 동일)

각 skill마다 폴더 1개 + 파일 2개 = 총 4 폴더 / 8 파일.

```
.agents/skills/<skill-name>/
├── SKILL.md
└── agents/openai.yaml
```

### 1-3. `SKILL.md` 본문 (4개 모두 동일 패턴, `<skill-name>`과 인자 표기만 교체)

각 4개 폴더의 `SKILL.md`를 다음 양식으로 작성. `<skill-name>`과 description의 인자 표기는 §1-1 표를 따른다.

```markdown
---
name: <skill-name>
description: Use ONLY when the user explicitly types `$<skill-name> <argument>`. Do not trigger implicitly from generic phrasing.
---

Source of truth: `.claude/skills/<skill-name>/SKILL.md`. Read it and follow the workflow.

Treat all frontmatter keys other than `name` and `description` (e.g., `agent:`, `disable-model-invocation:`, `allowed-tools:`, `context:`, `argument-hint:`, `model:`, `effort:`) as Claude-only and ignore them — execute locally in Codex.

**Slash command translation**: 본문 안의 `/<skill-name>` 표기는 Claude 슬래시 커맨드다. Codex에서는 `$<skill-name>`으로 읽고 사용자에게 안내한다 (예: 본문 "다음 단계: `/finalize-workitem T-001`" → Codex 응답에서는 "다음 단계: `$finalize-workitem T-001`"). Codex CLI는 `/`를 빌트인 슬래시 커맨드에 쓰므로 명시적 치환이 필요.

Preserve all repo policies from `AGENTS.md` and `docs/`.

If the source path no longer exists, this wrapper is stale — see ADR-010.
```

**Description 인자 표기 — wrapper별 정확한 문자열** (Phase 1 wrapper와 동일하게 인자 부분을 backtick으로 감싼다):
- `plan-workitem`: ``Use ONLY when the user explicitly types `$plan-workitem <milestone-or-feature-id>`. Do not trigger implicitly from generic phrasing.``
- `bootstrap-project`: ``Use ONLY when the user explicitly types `$bootstrap-project <brief-or-empty> [--apply]`. Do not trigger implicitly from generic phrasing.``
- `bootstrap-stack`: ``Use ONLY when the user explicitly types `$bootstrap-stack <stack-and-runtime-summary>`. Do not trigger implicitly from generic phrasing.``
- `stabilize-milestone`: ``Use ONLY when the user explicitly types `$stabilize-milestone <milestone-id>`. Do not trigger implicitly from generic phrasing.``

### 1-4. `agents/openai.yaml` 본문 (4개 모두 동일)

```yaml
policy:
  allow_implicit_invocation: false
```

근거 — IMPROVE-GUIDE.md §6-5 그대로. metadata-level 1차 방어.

### 1-5. 만들어야 할 파일 8개 정리

| # | 파일 |
|---|---|
| 1 | `.agents/skills/plan-workitem/SKILL.md` |
| 2 | `.agents/skills/plan-workitem/agents/openai.yaml` |
| 3 | `.agents/skills/bootstrap-project/SKILL.md` |
| 4 | `.agents/skills/bootstrap-project/agents/openai.yaml` |
| 5 | `.agents/skills/bootstrap-stack/SKILL.md` |
| 6 | `.agents/skills/bootstrap-stack/agents/openai.yaml` |
| 7 | `.agents/skills/stabilize-milestone/SKILL.md` |
| 8 | `.agents/skills/stabilize-milestone/agents/openai.yaml` |

### 1-6. 적용 후 실측

```text
# Codex CLI 진입 후
> /skills
# Phase 1의 4개 + Phase 1.5의 4개 = 총 8개 wrapper가 목록에 보이는지 확인

> $plan-workitem M1
# wrapper가 .claude/skills/plan-workitem/SKILL.md를 read하고 따라 실행하는지 확인
> $bootstrap-project "내 브리프"
> $bootstrap-stack "Next.js + pnpm"
> $stabilize-milestone M1
```

또한 generic prompt(예: "이 milestone 정리해줘", "프로젝트 시작해줘")에서 4개 wrapper가 **자동 트리거되지 않는지** 확인 — `agents/openai.yaml`의 `policy.allow_implicit_invocation: false`가 4개 모두에 박혀 있으면 안 트리거되는 게 정상.

### 1-7. Step 1 완료 체크

- [ ] 4 폴더 / 8 파일 신규 존재.
- [ ] 각 `SKILL.md`의 frontmatter `name` 필드가 폴더명과 일치.
- [ ] 각 `SKILL.md`의 description이 §1-3 wrapper별 정확한 문자열로 박힘 (인자 표기 + backtick 포함).
- [ ] 각 `agents/openai.yaml`에 `policy.allow_implicit_invocation: false` 박힘.
- [ ] **repo-scoped wrapper 8개** (Phase 1 4개 + Phase 1.5 4개)가 `.agents/skills/`에 존재하며 Codex에서 `$<skill-name>`으로 명시 호출 가능. (`/skills` 목록 표시 여부는 컨텍스트 예산·user/admin/system skill 동시 표시 여부에 따라 변동 가능 — 명시 호출 동작이 1차 검증 기준.)
- [ ] `$plan-workitem M1` 등이 SSOT 본문(`.claude/skills/<name>/SKILL.md`)을 read 가능.
- [ ] generic prompt에서 4개 wrapper가 자동 트리거되지 않음.

### 1-8. 커밋

```text
feat(codex): add core workflow wrappers (plan/bootstrap/stabilize)
```

---

## Step 2 — README 두 종 + ADR-010 후속작업 갱신

> 권장 에이전트: **builder-sonnet** — 표 행 추가 + 단락 in-place 갱신.
> 선행: Step 1 완료 (또는 README/ADR 갱신 후 Step 1 작업이라도 동시 가능 — 함께 묶이는 한 커밋이 만족스럽기만 하면 무방). 처음 따라가는 사람은 Step 1 → Step 2 순서.

### 2-1. `README.md` (영문) Codex CLI 단락 갱신

기존 단락(IMPROVE-GUIDE.md Step 7-2에서 추가한 것)에서 "Inner-loop skills are callable via Codex Skills:" 항목 아래 4개 bullet에 **Phase 1.5 wrapper 4개를 추가** + 항목 제목을 일반화.

**변경 전 (3번 항목)**:
```markdown
3. Inner-loop skills are callable via Codex Skills:
   - `$implement-workitem T-001`
   - `$validate-workitem T-001`
   - `$repair-workitem T-001`
   - `$finalize-workitem T-001`
4. For other steps, invoke in natural language: *"Follow `.claude/skills/bootstrap-project/SKILL.md`"*.
```

**변경 후**:
```markdown
3. Core workflow skills are callable via Codex Skills:
   - Inner loop: `$implement-workitem T-001`, `$validate-workitem T-001`, `$repair-workitem T-001`, `$finalize-workitem T-001`
   - Planning / bootstrap / stabilize: `$plan-workitem M1`, `$bootstrap-project <brief>`, `$bootstrap-stack <stack>`, `$stabilize-milestone M1`
4. For remaining skills (`discover-product`, `stack-guard`, `review-doc`, `boilerplate-context`), invoke in natural language: *"Follow `.claude/skills/<name>/SKILL.md`"*.
```

### 2-2. `README_ko.md` (한국어) Codex CLI 단락 갱신

**변경 전 (3번 항목)**:
```markdown
3. 자주 쓰는 inner-loop는 Codex skill로 호출 가능:
   - `$implement-workitem T-001`
   - `$validate-workitem T-001`
   - `$repair-workitem T-001`
   - `$finalize-workitem T-001`
4. 그 외 단계는 자연어로 호출: *"Follow `.claude/skills/bootstrap-project/SKILL.md`"*
```

**변경 후**:
```markdown
3. 자주 쓰는 core workflow skill은 Codex skill로 호출 가능:
   - Inner loop: `$implement-workitem T-001`, `$validate-workitem T-001`, `$repair-workitem T-001`, `$finalize-workitem T-001`
   - Planning / bootstrap / stabilize: `$plan-workitem M1`, `$bootstrap-project <brief>`, `$bootstrap-stack <스택>`, `$stabilize-milestone M1`
4. 나머지 skill(`discover-product`, `stack-guard`, `review-doc`, `boilerplate-context`)은 자연어로 호출: *"Follow `.claude/skills/<name>/SKILL.md`"*
```

### 2-3. ADR-010 "후속 작업" 섹션 in-place 갱신

`docs/90-decisions/ADR-010-multi-agent-compatibility.md`의 "후속 작업" 섹션에 박힌 다음 줄을 갱신한다.

**변경 전**:
```markdown
- Phase 2 workflow wrapper 6개 (Codex 사용 빈도 자리잡으면, 사용 빈도에 따라 선별), 유틸리티 2개(`review-doc`, `boilerplate-context`)는 필요 시.
```

**변경 후** (가이드는 ephemeral이므로 ADR이 가이드를 가리키지 않게 — 근거는 ADR 본문에 inline으로 흡수):
```markdown
- Phase 1.5 (적용됨): plan-workitem, bootstrap-project, bootstrap-stack, stabilize-milestone 4개 wrapper 추가. 근거 — fork 직후 첫 진입 시나리오(charter → architecture → 첫 분해)에서 자연어 호출 대비 wrapper 가성비가 inner-loop와 동등.
- Phase 2 (보류): discover-product, stack-guard, review-doc, boilerplate-context 4개. 호출 빈도 낮아 자연어 호출로 충분.
```

### 2-4. Step 2 완료 체크

- [ ] `README.md`, `README_ko.md` Codex CLI 단락이 8개 wrapper 모두 명시 + 자연어 호출 대상이 4개로 좁혀짐.
- [ ] 두 README가 같은 정보를 다른 언어로 표현 (drift 0).
- [ ] ADR-010 "후속 작업" 섹션의 Phase 2 메모가 Phase 1.5 적용 사실 + Phase 2 보류 4개로 갱신됨.
- [ ] 두 README의 Structure 트리는 손대지 않음 (Phase 1에서 이미 갱신, 본 단계는 그대로).

### 2-5. 커밋

```text
docs: surface Phase 1.5 wrappers in READMEs and ADR-010 follow-up
```

---

## Step 3 — 최종 검증 (Phase 1.5 완료 체크리스트)

> 권장 에이전트: **validator-sonnet**.
> 옵션: **reviewer** — ADR-010 본문 ↔ Phase 1.5 산출물 drift 점검.

### 3-1. Phase 1.5 완료 체크리스트 (필수)

- [ ] `.agents/skills/` 아래 Phase 1 + Phase 1.5 = **총 8 폴더 / 16 파일** 존재.
- [ ] 각 Phase 1.5 wrapper의 `SKILL.md` description이 §1-3 wrapper별 정확한 인자 표기 + backtick으로 박힘 ("Use ONLY when ..." 패턴).
- [ ] Phase 1.5 추가된 4개(`$plan-workitem`, `$bootstrap-project`, `$bootstrap-stack`, `$stabilize-milestone`)가 Codex에서 명시 호출 가능 (`/skills` 표시 여부와 무관하게 `$...`로 트리거).
- [ ] 각 Phase 1.5 wrapper 호출 시 `.claude/skills/<name>/SKILL.md` SSOT 본문을 read해서 따라 실행됨.
- [ ] **Implicit invocation 차단 실측**: generic prompt(예: "마일스톤 정리해줘", "프로젝트 시작해줘", "스택 설정해줘", "이 feature 분해해줘")에서 4개 wrapper가 자동 트리거되지 않음.
- [ ] `README.md`/`README_ko.md`의 Codex CLI 단락이 8개 wrapper 모두 명시.
- [ ] ADR-010 "후속 작업"에 Phase 1.5 적용 + Phase 2 잔존 4개가 명시됨.

### 3-2. 회귀 검증

- [ ] 기존 Phase 1 wrapper 4개(`$implement/validate/repair/finalize-workitem`)가 그대로 동작.
- [ ] Claude Code의 `/<skill-name>` 슬래시 커맨드가 그대로 동작 — Phase 1.5는 `.claude/skills/` SSOT 본문을 손대지 않으므로 보일러플레이트 12개 skill 전체가 영향 0이 정상.
- [ ] `docs/` 어떤 문서도 깨진 링크 없음.
- [ ] AGENTS.md / CLAUDE.md / .codex/config.toml은 손대지 않음.

### 3-3. 비범위 누락 확인

- [ ] `.agents/skills/discover-product/`, `.agents/skills/stack-guard/`, `.agents/skills/review-doc/`, `.agents/skills/boilerplate-context/` 4 폴더가 **생성되지 않았는지** (Phase 2 잔존).
- [ ] `.codex/agents/*.toml` 파일이 생성되지 않았는지 (Phase 3 보류 그대로).
- [ ] `.codex/config.toml`이 본 가이드 시작 시점의 main HEAD 상태 그대로 유지됨 (Phase 1.5 작업으로 추가 변경 없음).
- [ ] 새 ADR 파일이 생성되지 않았는지 (본 작업은 ADR-010 범위 안).

### 3-4. 통과 못한 항목 처리

- §3-1 항목 미통과 → 해당 Step으로 돌아가 수정 + 새 커밋 (amend 금지).
- §3-2 회귀 발견 → 회귀 일으킨 변경 식별 + 후속 커밋.
- §3-3 비범위 위반 발견 → 위반 산출물 삭제 + 후속 커밋.

**커밋**: 없음 (검증만). 통과 못해서 수정할 때만 새 커밋.

### 3-5. 통과 후 — 본 가이드의 역할 종료

§3 체크리스트가 모두 통과되면 본 가이드의 역할은 끝난다. 이 문서(및 IMPROVE-GUIDE.md, DRAFT-IMPROVE-GUIDE.md)는 ephemeral이므로 사용자가 알아서 삭제한다.

삭제 후에도 다음 정보는 영속 보존된다:

| 정보 | 보존 위치 |
|---|---|
| Phase 1 + Phase 1.5 모든 결정과 근거 | `docs/90-decisions/ADR-010-multi-agent-compatibility.md` (후속 작업 섹션 포함) |
| 산출물 위치 | `docs/00-meta/STRUCTURE.md` (산출물 표 + Canonical Owner 매핑) |
| 도구 표면 매핑 | `docs/90-decisions/ADR-010-multi-agent-compatibility.md` 본문 |
| 사용 안내 (8개 wrapper + 자연어 호출 4개) | `README.md`, `README_ko.md`의 "Codex CLI" 단락 |
| 진입 본문 | `AGENTS.md` |

---

## 부록 A — Phase 2/3 분리 근거 + Phase 1.5 도입 분석

### A-1. DRAFT의 Phase 2/3 분리 근거 (출처 그대로)

**Phase 1만 wrapper 박은 이유** (DRAFT §6-6):
> "inner loop은 task마다 반복되므로 wrapper 가성비가 가장 높다. bootstrap/plan/discover 등은 회당 1회라 자연어 호출로 충분 — Phase 2."

**Phase 2 보류 이유** (DRAFT §5):
> "실제로 Codex 사용 빈도가 자리잡으면, 사용 빈도에 따라 선별"

**Phase 3 보류 이유** (DRAFT §3 D6 + 비범위):
> "Codex는 worker/explorer/default 빌트인이 있고, custom은 명시 subagent workflow 자주 쓸 때만 가치 큼"
> ".claude/agents/ 6개를 .codex/agents/*.toml로 1:1 미러링 → SSOT 위반·drift 비용 큼"

DRAFT의 분류 축은 **호출 빈도 × wrapper 가성비** 단 1축. "회당 1회"는 자연어 호출로 충분하다는 가정.

### A-2. Phase 1.5 도입의 분석

DRAFT 가정이 흔들리는 케이스:
- **사용자가 fork 직후부터 Codex로 작업 시작**하는 시나리오. ADR-007 라이프사이클의 첫 5단계(discover → bootstrap-project → bootstrap-stack → stack-guard → plan-workitem)가 모두 자연어 호출에 의존.
- "회당 1회"여도 그 1회가 새 프로젝트의 가장 중요한 첫 발화점이라는 측면을 DRAFT는 평가하지 않음.

선별 기준 4축 (§1-2 그대로):
1. fork 직후 첫 진입점인가 — bootstrap-project가 가장 강함.
2. 분해 단계마다 호출되는가 — plan-workitem.
3. 워크아이템 라이프사이클의 분기점인가 — bootstrap-stack, stabilize-milestone.
4. 자연어 호출의 description 매칭 흔들림 위험이 큰가 — Phase 2 보류 4개는 호출 자체가 드물어 매칭 흔들림 빈도도 낮음.

이 4축을 통과한 4개(plan-workitem / bootstrap-project / bootstrap-stack / stabilize-milestone)를 Phase 1.5로 끌어옴. 나머지 4개(discover-product / stack-guard / review-doc / boilerplate-context)는 Phase 2 잔존.

### A-3. 가성비 매트릭스 (재정리)

| Skill | 빈도 | wrapper 가치 | 분류 |
|---|---|---|---|
| `implement-workitem` | task마다 | 매우 높음 | Phase 1 |
| `validate-workitem` | task마다 | 매우 높음 | Phase 1 |
| `repair-workitem` | needs-fix 시 | 높음 | Phase 1 |
| `finalize-workitem` | pass 시 | 매우 높음 | Phase 1 |
| `plan-workitem` | 분해 시점마다 | 높음 | **Phase 1.5** |
| `bootstrap-project` | 프로젝트 시작 1회 (중요) | 중간 | **Phase 1.5** |
| `bootstrap-stack` | 스택 변경 1회 | 중간 | **Phase 1.5** |
| `stabilize-milestone` | 마일스톤마다 1회 | 중간 | **Phase 1.5** |
| `discover-product` | 발굴 1회 (선택) | 낮음 | Phase 2 보류 |
| `stack-guard` | 스택 확정 직후 1회 | 낮음 | Phase 2 보류 |
| `review-doc` | 필요 시 (드물게) | 낮음 | Phase 2 보류 |
| `boilerplate-context` | 컨텍스트 인식용 (드물게) | 매우 낮음 | Phase 2 보류 |

---

## 부록 B — Phase 3 custom subagents 보류 근거 (확장)

### B-1. Codex subagents 제약

[공식 문서 — codex/subagents](https://developers.openai.com/codex/subagents) 핵심:
- **명시 호출 시에만 spawn** — Codex 메인 세션이 task 종류를 보고 자동으로 subagent를 선택하는 패턴 없음.
- 빌트인은 worker/explorer/default 3개.
- custom subagent는 `.codex/agents/<name>.toml` 형식.

### B-2. Claude의 자동 위임 패턴과의 격차

`.claude/agents/architect-opus.md` 등 6개 sub-agent는 본문 description으로 **자동 위임 트리거**를 정의한다 (예: builder-sonnet의 description이 "scoped implementation work"이면 메인 세션이 implement 작업에서 자동 spawn).

이 자동 위임을 Codex로 옮기려면:
- 자동 spawn이 없으므로 사용자가 매번 `"use builder agent for X"` 같이 명시 호출.
- 명시 호출은 custom toml 없어도 자연어로 가능 — 즉 toml 미러링의 추가 가치가 명확하지 않음.

### B-3. SSOT 본문 형식 차이

Claude 측 sub-agent 본문: markdown (`.claude/agents/<name>.md`).
Codex 측: TOML (`.codex/agents/<name>.toml`).

같은 책임 경계·체크리스트를 두 형식으로 박으면 drift 발생 시 매번 양쪽 갱신 필요. SSOT 패턴 1·4 위반.

### B-4. 보류 결론

- **자동 위임 패턴 보존 불가** + **명시 호출은 자연어로 이미 가능** + **SSOT 형식 차이 drift 비용** → Phase 3는 그대로 보류.
- "정말 명시 subagent workflow를 자주 쓰게 된 뒤" Phase 3로 별 작업 (DRAFT D6 그대로). 그 시점 판단은 미래 사용자 또는 미래 ADR.

### B-5. Phase 1.5에서 Phase 3로 이행하는 트리거 (메모)

다음 중 하나가 충족되면 Phase 3 실 작업 검토:
1. Codex 메인 세션에서 명시 subagent 호출이 주당 ≥ 5회.
2. 같은 책임 경계의 작업이 sub-agent 분리 없이 main 컨텍스트를 오염시키는 빈도가 측정됨.
3. 위 트리거는 본 가이드 적용 후 ≥ 1개월 사용 후에 재평가.

이 트리거는 ADR-010의 "후속 작업"으로 박지 않는다 — 본 부록(IMPROVE-GUIDE2)이 폐기되면 함께 사라지지만, 그 시점 사용자는 자기 사용 패턴을 직접 평가한 뒤 새 ADR로 박는 게 맞다 (DRAFT가 일관되게 권장한 "사용 빈도 자리잡으면" 원칙과 같음).
