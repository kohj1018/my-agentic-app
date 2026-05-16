# IMPROVE-LIST

이 저장소는 이미 [ADR-006](docs/90-decisions/boilerplate/ADR-006-simplicity-and-architecture.md) / [ADR-009](docs/90-decisions/boilerplate/ADR-009-tdd-default.md)로 *Simplicity First*와 *Goal-Driven Execution*을 정교하게 cover한다. 본 리스트는 진화 라운드별 누적된 *정책/구현 정밀 보강 항목*을 4 부류로 박는다 — Karpathy 4원칙 흡수에 한정되지 않고, 자기 모순 / cross-tool drift / surface 강화 audit 발견까지 포함.

- **Karpathy precision** (item 1–7, 11, 23, 24): *Surgical Changes* 4 실패 모드 + plan-time / implementation-time ambiguity 처리.
- **Internal drift fix** (item 12–15): 자기 모순 / cross-tool drift / 출력만 있고 영속화 X — 현재 박힌 *버그* 직접 교정. 모두 [관측됨] P0.
- **핵심 surface 강화** (item 16–21): stack-guard 깊이 dialing(스택별 verify 풀세트 + Codex wrapper + smoke test + Windows path + async hook 옵션 adapter) + stabilize-milestone deterministic preflight. 장기 운영의 *검증 계약*과 *통합 점검*의 신뢰성 확보.
- **별건 후속** (item 8–10, 22): Karpathy 4원칙 *적용 범위 밖*이지만 정합한 운영 정밀화 — LOC heuristic / instruction loop / 운영 기술 사실 / pattern naming.

각 항목은 다음 5필드로 박는다.

- **현재 상황**: 지금 어떤 surface에 어떻게 박혀 있는지(또는 비어 있는지). file:line 인용.
- **문제**: 발견된 drift / gap / 모순.
- **개선 방안**: 어떤 파일의 어떤 줄을 어떻게 고치는지(전/후 문구 포함).
- **근거**: 정책 ADR 링크 + [ADR-022](docs/90-decisions/boilerplate/ADR-022-ratchet-principle.md) evidence label (`[관측됨]` / `[외부실증]` / `[가설]` 또는 합성).
- **분류**: 우선순위(P0/P1/P2) + Ratchet 강도(강 = 제약 가능 / 약 = enabling 권장).

---

## 1. `builder.md` self-check의 "dead code 삭제 가능성" 문구 *방향 교정*

### 현재 상황
[.claude/agents/builder.md:40](.claude/agents/builder.md) 의 단순성 self-check 4번째 줄:

> *"삭제 가능한 dead code(쓰이지 않는 import·변수·branch)가 남았는가?"*

이 self-check는 builder가 task 구현 출력 직전에 모든 dead code를 *정리 후보*로 본다.

### 문제
Karpathy의 *Surgical Changes* 원칙은 정반대 방향이다:

> *"Remove imports/variables/functions that YOUR changes made unused. Don't remove pre-existing dead code unless asked."*

현재 builder 문구는 *pre-existing dead code*와 *내 변경이 만든 orphan*을 구분하지 않아, builder가 "이 김에 정리하자"고 인접 코드를 건드리는 길을 연다. **drive-by refactor의 의도치 않은 진입로 — 이 저장소의 "범위 밖 변경 금지" 정신 ([AGENTS.md](AGENTS.md) 핵심 행동 규율 3행)과 *내부 모순*.**

### 개선 방안
[.claude/agents/builder.md:40](.claude/agents/builder.md) 의 4번째 self-check 줄을 다음으로 교체:

```
- 이번 변경이 만든 orphan(쓰이지 않게 된 import·변수·branch)만 정리했는가?
  pre-existing dead code는 출력에 *언급*만 하고 *삭제하지 않았는가*?
```

### 근거
- 정책 SSOT: [ADR-006](docs/90-decisions/boilerplate/ADR-006-simplicity-and-architecture.md) 단순성 1순위, [AGENTS.md](AGENTS.md) "범위 밖 변경은 하지 않는다".
- Evidence label: `[관측됨]` — 내부 self-check 문구와 "범위 밖 변경 금지" 행동 규율 사이의 *방향 모순*은 직접 관측. Karpathy testimony는 framing 근거일 뿐 *evidence*로 가산하지 않는다 ([ADR-022](docs/90-decisions/boilerplate/ADR-022-ratchet-principle.md) "다중 repo 실증" 기준 미달).
- Ratchet: **제약 정책(강)** — 기존 문구의 *교정*이라 새 제약 도입이 아니라 *drift fix*다.

### 분류
- **우선순위: P0** (drift fix)
- **Ratchet: 강**

---

## 2. `builder.md` self-check에 *diff traceability* 항목 추가

### 현재 상황
[.claude/agents/builder.md:36-43](.claude/agents/builder.md) 단순성 self-check는 6항목으로 — (1) 추상화 2회+, (2) try/except 시스템 경계, (3) WHY 주석, (4) dead code(→1번 항목에서 교정 예정), (5) 가설적 예방 차단([ADR-022](docs/90-decisions/boilerplate/ADR-022-ratchet-principle.md)), (6) 인터페이스 SSOT 위반 점검.

builder.md의 "규칙"에는 "범위 밖 변경은 하지 않는다" 1줄이 있다.

### 문제
"범위 밖 변경 금지"는 추상 명령이다. Karpathy가 콕 집은 *4개 sub-failure*가 어디에도 항목화돼 있지 않다:

1. *"Don't 'improve' adjacent code, comments, or formatting."*
2. *"Don't refactor things that aren't broken."*
3. *"Match existing style, even if you'd do it differently."*
4. *"Every changed line should trace directly to the user's request."*

특히 4번 *"changed line → user request" 역추적 가능성*은 builder가 RGR 사이클 출력 직전에 self-check할 자연스러운 항목인데 빠져 있다.

### 개선 방안
[.claude/agents/builder.md](.claude/agents/builder.md) self-check 항목에 다음 1줄을 추가:

```
- 이번 변경의 모든 줄이 task의 AC 또는 명시 요청으로 거꾸로 추적 가능한가?
  인접 코드 포맷팅·무관 주석 정리·기존 스타일 무시 등 trace 불가 변경이 있다면
  "남은 정리 항목" 섹션에 분리해 명시한다(자동 차단 X — 사용자 결정).
```

### 근거
- 정책 SSOT: [ADR-006](docs/90-decisions/boilerplate/ADR-006-simplicity-and-architecture.md) + [AGENTS.md](AGENTS.md) 범위 밖 변경 금지.
- Evidence label: `[가설]` — Karpathy 단일 testimony는 [ADR-022](docs/90-decisions/boilerplate/ADR-022-ratchet-principle.md) "다중 repo 실증" 기준 미달. Chang 정리본의 커뮤니티 채택은 *adoption signal*이지 *outcome 검증 evidence*는 아님. 향후 fork 프로젝트에서 [관측됨] 회수 시 [관측됨+외부실증]로 재라벨링.
- Ratchet: **enabling 정책(약)** — *권장 텍스트*만 출력하고 *자동 차단 X*. [ADR-022](docs/90-decisions/boilerplate/ADR-022-ratchet-principle.md) "예방적 가설만 있다면 제약이 아닌 enabling" 규칙 정합.

### 분류
- **우선순위: P1** (가설 기반 enabling, 예방 surface)
- **Ratchet: 약**

---

## 3. `validate-workitem/SKILL.md`에 *diff trace audit* 검증 기준 추가

### 현재 상황
[.claude/skills/validate-workitem/SKILL.md:26-33](.claude/skills/validate-workitem/SKILL.md) 검증 기준 7항목 중 *"범위 밖 변경이 있는가"* 1줄이 있다.

[.claude/skills/validate-workitem/SKILL.md:38-61](.claude/skills/validate-workitem/SKILL.md) report 양식에는 *"실패 항목 / AC ↔ 테스트 매핑 / 다음 권장 액션"* 섹션만 있고 *diff trace audit* 결과를 박는 표준 위치가 없다.

### 문제
"범위 밖 변경이 있는가"는 builder의 self-check와 같은 추상 명령이다. validator는 이미 `Bash(git diff *)` 권한을 갖고 있는 **구조적 검출 지점**([ADR-007](docs/90-decisions/boilerplate/ADR-007-workitem-lifecycle.md) amend 2 — validator = 판정 + report 전용)인데, 이 권한이 *diff 줄 단위 추적*에 활용되지 않는다.

이 저장소의 lifecycle은 **builder = 예방, validator = 검출**이 자연스러운 분업이다. builder self-check만으로는 *권장*이지 *enforcement*가 아니므로, validator-side audit이 함께 박혀야 detection 완성된다.

### 개선 방안
[.claude/skills/validate-workitem/SKILL.md:26](.claude/skills/validate-workitem/SKILL.md) 의 "범위 밖 변경이 있는가" 줄을 다음으로 확장:

```
- 범위 밖 변경 + diff trace audit:
  `git diff` 결과의 각 변경 줄(또는 hunk)이 task의 AC-N 또는 명시 요청으로
  거꾸로 추적 가능한가? 추적 불가 줄은 다음 카테고리 중 하나로 분류 보고.

  Needs Fix 트리거 (강 constraint, P0 라벨):
    (c) pre-existing dead code 삭제 — task 범위 밖 *파괴적 변경*. Pass 차단.

  Report only + reviewer 라벨 권장 (약 enabling, P1/P2 라벨):
    (a) 인접 코드 포맷팅/주석 정리 — P1
    (b) 무관 리팩토링 (행동 미변경 코드 구조 변경) — P1
    (d) 스타일 변경 (semicolon, quote, indent 등) — P2

  의도적 (c)는 task 문서에 명시 요청으로 박혀 있을 때만 Pass 통과.
```

[.claude/skills/validate-workitem/SKILL.md:38-61](.claude/skills/validate-workitem/SKILL.md) report 양식에 다음 섹션 1개 추가:

```markdown
## Diff trace audit
- 추적 가능 변경 줄: N개 (AC-1: M개, AC-2: ...)
- 추적 불가 변경 줄: K개
  - (a) 인접 포맷팅/주석: <file:line> ... [P1]
  - (b) 무관 리팩토링: ... [P1]
  - (c) pre-existing dead code 삭제: ... [P0 — Needs Fix 트리거]
  - (d) 스타일 변경: ... [P2]
- 판정 영향: <Pass 유지 / Needs Fix 트리거 (오직 (c) 의도 외 발견 시)>
```

### 근거
- 정책 SSOT: [ADR-007](docs/90-decisions/boilerplate/ADR-007-workitem-lifecycle.md) — validator는 *판정 + report*, 구조적 enforcement 지점.
- Evidence label:
  - (c) Needs Fix 트리거 부분: `[관측됨+외부실증]` — *internal*: 항목 1의 builder.md drift fix가 같은 위반을 *예방* 측에서 막는 짝 surface (관측 가능), *external*: Karpathy *"collateral damage"* testimony.
  - (a)(b)(d) 라벨링 부분: `[가설]` — 항목 2와 동일.
- Ratchet: **혼합** — (c)만 *제약(강)*, (a)(b)(d)는 *enabling(약)*. 같은 audit surface 안에서 카테고리별 라벨 강도 분리 — [ADR-022](docs/90-decisions/boilerplate/ADR-022-ratchet-principle.md) 자기 일관성 정합.

### 분류
- **우선순위:** (c) **P0** (Needs Fix 트리거), (a)(b)(d) **P1** (라벨링)
- **Ratchet:** 혼합 — (c) 강 / 나머지 약

---

## 4. `reviewer.md`에 *Scope Discipline 별도 섹션* 추가 (Clean Code 6은 그대로)

### 현재 상황
[.claude/agents/reviewer.md:24-30](.claude/agents/reviewer.md) Clean Code 6항목 체크리스트:

1. Naming, 2. Function size + SR, 3. Duplication, 4. Premature abstraction, 5. Comment policy, 6. Layer leak.

reviewer는 각 발견 항목을 `P0/P1/P2 [카테고리] <설명>` 형식으로 라벨링한다 (예: `P1 [Duplication] auth.ts:42 — ...`).

### 문제
*Diff trace audit*에서 발견되는 *인접 코드 무관 변경 / pre-existing 정리 / 스타일 변경*은 위 6항목 어디에도 깔끔하게 들어맞지 않는다. *Premature abstraction(4)*은 *추가된 추상화*가 대상이지 *변경 scope*는 아니다. → 같은 위반이 호출 surface마다 다른 라벨로 박힐 위험 (SSOT drift).

또한 reviewer는 `/validate-workitem` 외에도 `/review-doc`, manual review, `/stabilize-milestone` 등 여러 surface에서 호출된다 — validator의 diff trace audit가 작동하지 않는 surface에서도 reviewer가 같은 위반을 잡을 수 있어야 한다.

Clean Code 6항목을 7항목으로 *확장*하면 [ADR-006](docs/90-decisions/boilerplate/ADR-006-simplicity-and-architecture.md)이 정의한 *Clean Code 6항목*의 SSOT를 흐린다. Clean Code는 *코드 스타일*, Scope는 *범위 정합* — 차원이 다르므로 별도 섹션으로 분리한다.

### 개선 방안
[.claude/agents/reviewer.md](.claude/agents/reviewer.md) Clean Code 6항목 체크리스트는 *그대로 유지*하고, 그 아래에 *별도* 섹션 1개 신설:

```markdown
## Scope Discipline 체크 (별도 차원 — Clean Code와 독립)

변경 줄이 task의 AC 또는 명시 요청으로 거꾸로 추적 가능한가.
다음 4 카테고리의 *범위 정합 위반*을 발견 시 라벨링.

- (a) 인접 코드 포맷팅/주석 정리 — `[Scope-format]`
- (b) 무관 리팩토링 — `[Scope-refactor]`
- (c) pre-existing dead code 삭제 — `[Scope-purge]` (P0 권장)
- (d) 기존 스타일 무시·변경 — `[Scope-style]`

reviewer 출력 라벨링 예: `P0 [Scope-purge] auth.ts:120 — 무관 dead function 삭제`.
```

### 근거
- 정책 SSOT: [ADR-006](docs/90-decisions/boilerplate/ADR-006-simplicity-and-architecture.md) (단순성 1순위, 범위 밖 변경 금지의 라벨링 SSOT).
- Evidence label: `[가설]` — 항목 2와 동일 근거.
- Ratchet: **enabling 정책(약)** — *라벨링*은 reviewer가 발견을 분류해 출력하는 기능이지 *Pass 차단*은 아니다 (차단은 항목 3 validator의 (c) 카테고리만).

### 분류
- **우선순위: P1** (라벨링 SSOT, 검출 보조)
- **Ratchet: 약**

---

## 5. `implement-workitem/SKILL.md` Red phase 직전에 *ambiguity surfacing protocol* 추가

### 현재 상황
[.claude/skills/implement-workitem/SKILL.md:31](.claude/skills/implement-workitem/SKILL.md):

> *"Red phase 진입 직전, 출력의 첫 단락으로 '이 task에서 어떤 테스트를 어떤 순서로 작성할 것인가'를 1~3문장으로 명시한다(plan 모드 의존 없이 think-before-edit 규율 확보)."*

plan-workitem이 AC 형식·금지 verb를 *plan 단계*에서 점검한다 ([plan-workitem/SKILL.md:36](.claude/skills/plan-workitem/SKILL.md)).

### 문제
*Plan 단계의 AC 점검*과 *implementation 시점의 ambiguity*는 다른 차원이다. AC가 GWT 형식·measurable verb를 만족해도 builder가 *2+ 해석 가능한 부분*을 silent하게 한 가지로 선택해 RGR을 돌릴 위험이 남는다.

Karpathy의 *Think Before Coding* 원칙:

> *"If multiple interpretations exist, present them - don't pick silently."*

현재 흐름은 silent pick → 잘못된 RGR → repair 사이클 → 비용 폭증으로 이어질 수 있다.

### 개선 방안
[.claude/skills/implement-workitem/SKILL.md:31](.claude/skills/implement-workitem/SKILL.md) Red phase 직전 plan 단락 *바로 뒤*에 1줄 추가:

```
AC가 2+ 해석이 가능하다고 판단되면 해석안을 1~3개 나열하고
*자기가 선택한 해석*을 표시한다(예: "해석 A를 따른다 — 이유: ...").
자동 차단 X — 사용자가 출력 보고 차단/수정 결정.
```

### 근거
- 정책 SSOT: [ADR-006](docs/90-decisions/boilerplate/ADR-006-simplicity-and-architecture.md) + [ADR-007](docs/90-decisions/boilerplate/ADR-007-workitem-lifecycle.md) 자동 차단 X 원칙.
- Evidence label: `[가설]` — Karpathy testimony에 기반한 예방적 가설. [ADR-022](docs/90-decisions/boilerplate/ADR-022-ratchet-principle.md) 다중 repo 실증 기준 미달. 향후 fork 프로젝트의 repair 비율 회수 후 [관측됨] 승격 검토.
- Ratchet: **enabling 정책(약)** — 자동 차단 없이 *권장 텍스트 출력*만. [ADR-022](docs/90-decisions/boilerplate/ADR-022-ratchet-principle.md) "가설만 있다면 enabling" 정합.

### 분류
- **우선순위: P1** (예방적 enabling, 디폴트 흐름 내장)
- **Ratchet: 약**

---

## 6. `implement-workitem/SKILL.md` Red phase plan을 *Step → verify* 형식 권장

### 현재 상황
[.claude/skills/implement-workitem/SKILL.md:31](.claude/skills/implement-workitem/SKILL.md) Red phase 직전 plan은 *"어떤 테스트를 어떤 순서로 작성할 것인가" 1~3문장 자유 텍스트*.

### 문제
Karpathy의 *Goal-Driven Execution* 원칙은 *"For multi-step tasks, state a brief plan: [Step] → verify: [check]"* 형식을 권한다. 이 형식이 *self-loop 가능한 success criteria*를 강제하는 핵심.

현재 자유 텍스트 plan은 *RGR 자체가 verify를 강제*하므로 큰 갭은 아니지만, *plan 출력 자체*에 verify 표지가 없으면 builder가 plan을 *느슨하게* 쓰고 RGR로 정합 책임을 미루는 경향이 생긴다.

### 개선 방안
[.claude/skills/implement-workitem/SKILL.md:31](.claude/skills/implement-workitem/SKILL.md) Red phase plan 단락을 다음과 같이 확장:

```
Red phase 진입 직전, 출력의 첫 단락으로 plan을 다음 형식으로 명시
(plan 모드 의존 없이 think-before-edit 규율 확보):

  1. <Step> → verify: <어떤 테스트/조건으로 확인>
  2. <Step> → verify: <...>
  3. <Step> → verify: <...>

자유 텍스트 1~3문장도 허용하지만 Step → verify 형식이 권장.
*AC-N과 Step의 대응*은 plan 단계에서 명시.
```

### 근거
- 정책 SSOT: [ADR-009](docs/90-decisions/boilerplate/ADR-009-tdd-default.md) TDD Red phase + measurable verify.
- Evidence label: `[가설]` — Karpathy *"Step → verify"* 형식은 그의 권장이지 outcome 검증이 아님. RGR cycle이 이미 per-AC verify를 enforce하므로 *형식 보강*에 그침.
- Ratchet: **enabling 정책(약)** — RGR이 이미 verify를 강제하므로 *권장 형식*만 추가, 중복 가능성 존재.

### 분류
- **우선순위: P2** (RGR 중복, 형식 보강에 그침)
- **Ratchet: 약**

---

## 7. `AGENTS.md` 단순성 단락에 *Surgical Changes* 한 줄 추가

### 현재 상황
[AGENTS.md:16-21](AGENTS.md) 단순성·YAGNI 단락은 5줄이고, 전체 문서는 46줄 (hard cap 100줄, [ADR-011](docs/90-decisions/boilerplate/ADR-011-agents-md-line-cap.md)).

### 문제
[AGENTS.md](AGENTS.md) 핵심 행동 규율(11행)에 *"범위 밖 변경은 하지 않는다"*가 있지만, 위 항목 1~4에서 *구체적 실패 모드 4가지*로 쪼개진 내용은 어떤 *기존 ADR 외부의 진입 SSOT 1줄*도 갖고 있지 않다.

[ADR-005](docs/90-decisions/boilerplate/ADR-005-ssot.md) SSOT 패턴 5("진입 페이지" 패턴): fork된 새 세션이 자동 로드하는 [AGENTS.md](AGENTS.md)에 *핵심 행동 규율*은 박혀 있어야 한다. 항목 1~4가 모두 *Surgical Changes 한 원칙의 4가지 표현*이라 *진입 SSOT 1줄*이 필요.

### 개선 방안
[AGENTS.md:21](AGENTS.md) 단순성·YAGNI 단락 마지막에 1줄 추가:

```
- 변경한 모든 줄은 task의 AC 또는 명시 요청으로 거꾸로 추적 가능해야 한다.
  인접 코드 개선·무관 포맷팅·기존 스타일 무시·pre-existing dead code 삭제는 금지.
```

추가 후 [AGENTS.md](AGENTS.md) 줄 수: 46 → 48 (hard cap 100 여유 충분, [ADR-011](docs/90-decisions/boilerplate/ADR-011-agents-md-line-cap.md) 정합).

**순서 제약**: 이 1줄은 *반드시* [ADR-006](docs/90-decisions/boilerplate/ADR-006-simplicity-and-architecture.md) Amendment(항목 11)가 박힌 *후*에 추가한다. [AGENTS.md](AGENTS.md)는 진입 페이지이고 정책 본문이 아니다 ([ADR-005](docs/90-decisions/boilerplate/ADR-005-ssot.md) SSOT 패턴 5) — *ADR이 정책의 canonical, AGENTS.md는 1줄 요약*. 순서가 바뀌면 진입 페이지가 정책 본문처럼 보이는 문제 발생.

### 근거
- 정책 SSOT: [ADR-005](docs/90-decisions/boilerplate/ADR-005-ssot.md) 패턴 5 + [ADR-006](docs/90-decisions/boilerplate/ADR-006-simplicity-and-architecture.md).
- Evidence label: `[가설]` — 항목 2~6 정책의 *SSOT 1줄 propagation*이라 evidence는 underlying 가설을 따른다.
- Ratchet: **enabling 정책(약)** — SSOT 1줄 자체는 강하게 보이지만, underlying 규칙들이 [가설]/enabling이라 *실질 강도는 약*. constraint 효과는 항목 1·3 (c) 카테고리에서만 발생.

### 분류
- **우선순위: P1** (항목 11이 박힌 *후*에 SSOT 1줄)
- **Ratchet: 약**

---

## 8. LOC sanity heuristic을 *권장 텍스트* 형태로만 추가

### 현재 상황
[.claude/agents/builder.md:36-43](.claude/agents/builder.md) self-check에는 *추상화 2회+, try/except 위치, 주석 종류, dead code(→교정), 가설적 예방 차단, 인터페이스 SSOT*가 있지만 *전체 변경 LOC 규모*에 대한 sanity check는 없다.

[.claude/skills/plan-workitem/SKILL.md:31-35](.claude/skills/plan-workitem/SKILL.md) sizing self-check는 *task 단위* (1 RGR / AC 3 / 파일 5).

### 문제
Karpathy heuristic: *"If you write 200 lines and it could be 50, rewrite it."*

이건 *task 단위 sizing*과 다른 차원의 *code 단위 sanity*. 현재 어디에도 없다. 다만 강한 숫자 규칙은 false trigger 위험이 크다(initial scaffolding 등은 자연스럽게 큼).

### 개선 방안
[.claude/agents/builder.md](.claude/agents/builder.md) self-check 끝에 1줄을 *권장 텍스트 출력* 형태로 추가:

```
- 이번 task의 총 변경 LOC가 task 범위에 비해 큰 편인가?
  체감 200줄 이상 + 단순화 여지 있으면 "단순화 후보" 1~3개를
  *권장 텍스트*로 출력(자동 차단 X, 사용자 결정).
  initial scaffolding·auth 등 자연스럽게 큰 task는 면제.
```

*수치는 hard cap이 아니라 휴리스틱*임을 명시.

### 근거
- 정책 SSOT: [ADR-006](docs/90-decisions/boilerplate/ADR-006-simplicity-and-architecture.md).
- Evidence label: `[가설→트리거]` — [관측됨] 데이터 없음. 향후 fork 프로젝트에서 false trigger 비율 측정 후 trigger.
- Ratchet: **enabling 정책(약)** — 강제 숫자 X, 권장 텍스트만.

### 분류
- **우선순위: P2**
- **Ratchet: 약**

---

## 9. (별건 후속) `/stabilize-milestone`에 *instruction improvement 후보* 보고 추가

### 현재 상황
[.claude/skills/stabilize-milestone/SKILL.md](.claude/skills/stabilize-milestone/SKILL.md)는 *milestone 단위 통합 점검* — E2E + 회귀 + 리팩토링 후보 + ADR 후보. 결과는 [docs/40-validation/QA_FINDINGS.md](docs/40-validation/QA_FINDINGS.md), [docs/40-validation/IMPROVEMENT_GUIDE.md](docs/40-validation/IMPROVEMENT_GUIDE.md)에 누적.

### 문제
현재 stabilize 출력은 *코드·문서 내용*에 대한 개선 제안이지 *agent/skill body의 문구 자체*에 대한 개선 제안은 표준 항목이 아니다.

Anthropic case-study PDF ([www-cdn.anthropic.com/58284b19e702b49db9302d5b6f135ad8871e7658.pdf](https://www-cdn.anthropic.com/58284b19e702b49db9302d5b6f135ad8871e7658.pdf))에서 *세션 종료 시 Claude가 세션 요약과 함께 instruction 개선 제안을 만들어 CLAUDE.md/workflow 문서를 지속 개선하는 루프*가 보고된다. 단 단일 case-study라 [외부실증] 기준의 *다중 repo* 요건은 미달. 이 저장소의 *Living Doc* 정책 + [IMPROVEMENT_GUIDE.md](docs/40-validation/IMPROVEMENT_GUIDE.md) 스키마와는 *구조적 정합*은 분명.

### 개선 방안
[.claude/skills/stabilize-milestone/SKILL.md](.claude/skills/stabilize-milestone/SKILL.md) 출력 항목에 다음 1개 추가:

```
- instruction improvement 후보:
  본 마일스톤 동안 builder/validator/reviewer가 반복적으로 막힌 패턴,
  AGENTS.md 또는 agent/skill body 문구가 *비자명하거나 모호*했던 지점,
  새로 박을 만한 *self-check 항목* 후보를
  [IMPROVEMENT_GUIDE.md](docs/40-validation/IMPROVEMENT_GUIDE.md)에 보고.
  각 항목에 [ADR-022](docs/90-decisions/boilerplate/ADR-022-ratchet-principle.md) evidence label 부착.
  *AGENTS.md / agent / skill body는 자동 수정 X — 보고만*.
```

### 근거
- 정책 SSOT: [ADR-022](docs/90-decisions/boilerplate/ADR-022-ratchet-principle.md) 자동 적용 금지 + [IMPROVEMENT_GUIDE.md 스키마](docs/40-validation/IMPROVEMENT_GUIDE.md).
- Evidence label: `[가설]` — Anthropic 단일 case-study는 [ADR-022](docs/90-decisions/boilerplate/ADR-022-ratchet-principle.md) "다중 repo 실증" 기준 미달. 향후 fork 프로젝트에서 *문구 개선이 실제로 채택된 비율* 회수 시 [관측됨] 승격.
- Ratchet: **enabling 정책(약)** — 보고만, 자동 적용 X.

### 분류
- **우선순위: P2** (별건 후속, Karpathy 4원칙 적용 범위 *밖*)
- **Ratchet: 약**

---

## 10. (별건 후속) `bootstrap-stack` 산출물의 ARCHITECTURE_OVERVIEW *구체적 기술 사실* 체크리스트 강화

### 현재 상황
[.claude/skills/bootstrap-stack/SKILL.md](.claude/skills/bootstrap-stack/SKILL.md)와 [.claude/skills/bootstrap-stack/output-checklist.md](.claude/skills/bootstrap-stack/output-checklist.md)는 스택 결정·자동화 설계를 담당. [docs/20-system/ARCHITECTURE_OVERVIEW.md](docs/20-system/ARCHITECTURE_OVERVIEW.md)는 *시스템 구조 설명* + 7-1~7-4 인터페이스 결정을 담지만, *프로젝트 진입 시 자주 묻는 구체적 기술 사실*(ports / 실행 명령 / 환경변수 이름 / 주요 파일 역할 / known gotchas)에 대한 *명시 체크리스트*는 없다.

### 문제
Karpathy 본인 GitHub [karpathy/llm-council/CLAUDE.md](https://github.com/karpathy/llm-council/blob/master/CLAUDE.md)는 일반 행동 강령보다 *프로젝트별 기술 노트*(아키텍처·포트·환경변수·파일별 역할)를 담은 *실전 프로젝트 메모리* 성격. 이건 *Karpathy 4원칙*과 별개로 *CLAUDE.md를 어떻게 쓰는가*에 대한 그의 실제 사용 패턴.

이 저장소의 [ARCHITECTURE_OVERVIEW.md](docs/20-system/ARCHITECTURE_OVERVIEW.md) 7-1~7-4 섹션(API/CLI/백엔드/프론트)은 이미 *인터페이스 결정*을 담지만, *bootstrap 직후 자동으로 채워야 할 fact 체크리스트*는 없다 — fork 사용자가 직접 채워야.

surface는 `bootstrap-project`(charter + architecture 구조, 스택 미정 단계까지)가 아니라 `bootstrap-stack`(스택 확정 후 자동화 설계)이 자연 위치 — ports / 실행 명령 / 환경변수는 스택 결정 후에야 확정된다.

### 개선 방안
[.claude/skills/bootstrap-stack/output-checklist.md](.claude/skills/bootstrap-stack/output-checklist.md) 에 다음 체크리스트 항목 추가:

```
스택 확정 후 ARCHITECTURE_OVERVIEW.md에 *구체적 기술 사실*도 함께 채울 것:
- [ ] 실행 명령 (`pnpm dev`, `make run`, `task validate` 등) — 스택 정합
- [ ] 주요 포트 (개발/스테이징/프로덕션 분리)
- [ ] 환경변수 이름 (값은 비워둠 — secrets는 .env, `.gitignore` 정합)
- [ ] 주요 파일/디렉터리 역할 (3~5개 핵심만)
- [ ] known gotcha / 자주 막히는 지점 (있으면, 없으면 생략)

비-UI 프로젝트는 7-4(프론트) 섹션 생략, CLI 프로젝트는 7-2 강화 등 스택 정합.
```

대안 위치: [docs/20-system/ARCHITECTURE_OVERVIEW.md](docs/20-system/ARCHITECTURE_OVERVIEW.md) `## 7. 기술 선택` 아래 `## 7-5. 운영 기술 사실` 신설하고 `/bootstrap-stack`이 채움. → 이 경우 [ADR-027](docs/90-decisions/boilerplate/ADR-027-interface-decision-allocation.md) "인터페이스 결정 책임 분배" amend 필요.

### 근거
- 정책 SSOT: [ADR-027](docs/90-decisions/boilerplate/ADR-027-interface-decision-allocation.md) 인터페이스 결정 책임 분배.
- Evidence label: `[가설]` — Karpathy 본인 *실 사용 패턴*은 *individual practice*이지 [ADR-022](docs/90-decisions/boilerplate/ADR-022-ratchet-principle.md) "다중 repo 실증" 기준의 *외부실증*은 아님. 단 본 저장소의 *실용성* 관점에서는 [가설→실증]으로 곧장 들어가도 OK.
- Ratchet: **enabling 정책(약)** — 체크리스트는 권장, 강제 X.

### 분류
- **우선순위: P2** (별건 후속, Karpathy 4원칙 *적용 범위 밖*이나 *주변 통찰*)
- **Ratchet: 약**

---

## 11. (메타) [ADR-006](docs/90-decisions/boilerplate/ADR-006-simplicity-and-architecture.md) Amendment로 위 1~7의 *SSOT 정착*

### 현재 상황
[ADR-005](docs/90-decisions/boilerplate/ADR-005-ssot.md) 패턴 4: *"정책 = ADR"* — 보일러플레이트가 도입하는 모든 정책은 ADR로 박고, agent/skill 본문에는 정책 설명을 길게 박지 않는다.

[ADR-006](docs/90-decisions/boilerplate/ADR-006-simplicity-and-architecture.md)은 단순성·Clean Code·Clean Architecture 3층 우선순위를 정의하지만, *Surgical Changes 4가지 sub-failure*와 *implementation-time ambiguity*는 명시되어 있지 않다.

### 문제
위 항목 1~7을 그대로 적용하면 agent/skill body 7곳에 변경이 흩뿌려진다. SSOT가 한 곳에 모이지 않으면 향후 갱신/supersede 시 4~7곳을 빠뜨릴 위험.

### 개선 방안
[ADR-006](docs/90-decisions/boilerplate/ADR-006-simplicity-and-architecture.md)에 *Amendment 1*을 박는다 (예시 형식):

```markdown
## Amendment 1 (2026-05-16) — Surgical Changes 명시 + ambiguity surfacing protocol

### 결정
단순성 1순위에 다음 2개 sub-원칙을 명시한다.

**Surgical Changes (sub-원칙 1)**
- 변경 줄은 모두 task의 AC 또는 명시 요청으로 거꾸로 추적 가능.
- 인접 코드 개선·무관 포맷팅·기존 스타일 무시·pre-existing dead code 삭제 금지.
- pre-existing dead code는 *언급*만, *삭제는 명시 요청 시*.

**Ambiguity surfacing (sub-원칙 2)**
- AC가 2+ 해석 가능 시 builder는 해석안을 나열하고 자기 선택을 표시.
- 자동 차단 X — 사용자가 출력 보고 차단/수정 결정 ([ADR-007](docs/90-decisions/boilerplate/ADR-007-workitem-lifecycle.md) 정합).

### 강도 분류 (ADR-022 정합)
- **제약(강)** — pre-existing dead code 의도 외 *삭제*는 validator의 Pass 차단 트리거. 근거 라벨: `[관측됨+외부실증]` (item 1의 drift 관측 + Karpathy testimony).
- **enabling(약)** — 나머지 sub-원칙(diff trace 라벨링 / 인접 포맷팅 / 스타일 무시 / ambiguity surfacing 권장 텍스트)은 *report only* / *권장 텍스트 출력*. 근거 라벨: `[가설]`. 향후 fork 프로젝트에서 [관측됨] 회수 시 [가설→실증] 승격 검토.

### 적용 surface
- [AGENTS.md](AGENTS.md) 단순성 단락 1줄 (Surgical Changes 진입 SSOT).
- [.claude/agents/builder.md](.claude/agents/builder.md) self-check 2항목 (1: dead code wording 교정, 2: diff trace 추가).
- [.claude/agents/reviewer.md](.claude/agents/reviewer.md) *별도 Scope Discipline 섹션* 신설 (Clean Code 6은 그대로 유지).
- [.claude/skills/validate-workitem/SKILL.md](.claude/skills/validate-workitem/SKILL.md) diff trace audit + report 섹션 (카테고리 (c) Needs Fix 트리거).
- [.claude/skills/implement-workitem/SKILL.md](.claude/skills/implement-workitem/SKILL.md) Red phase plan + ambiguity gate.

본 amendment의 적용은 단일 task로 박지 않고 surface별로 분할 — 각 적용 task가 *amendment 본문의 적용 surface 완성*임을 amendment 자체에 명시.

### 근거
- Karpathy 직접 testimony (2026-01-26 X post): "silent assumptions", "collateral damage" 실패 모드 명명.
- Chang의 4-원칙 정리본의 커뮤니티 widespread adoption (단 star 수는 *adoption signal*이지 *outcome 검증 evidence*는 아님 — [ADR-022](docs/90-decisions/boilerplate/ADR-022-ratchet-principle.md) 정합).
- 내부 [관측됨]: [.claude/agents/builder.md](.claude/agents/builder.md) self-check 4번 "dead code 정리" 문구와 핵심 행동 규율 "범위 밖 변경 금지" 사이의 방향 모순 (item 1 drift).

### 후속 작업
- 본 Amendment 적용 후 builder self-check 비용 측정. 항목 추가가 builder 출력 토큰을 크게 늘리면 축약 검토.
- fork 프로젝트에서 [관측됨] 데이터 회수 후 enabling 부분을 [가설→실증]로 승격 검토.
- Anthropic instruction-improvement loop (item 9)는 본 amendment의 *자기 적용*: amendment 1 자체가 향후 라운드에서 supersede될 가능성 추적.
```

[docs/90-decisions/boilerplate/README.md](docs/90-decisions/boilerplate/README.md)의 ADR-006 행에 *(+amend1: Surgical Changes + ambiguity surfacing)* 박음.

### 근거
- 정책 SSOT: [ADR-005](docs/90-decisions/boilerplate/ADR-005-ssot.md) 패턴 4 ("정책 = ADR").
- Evidence label: 혼합 — 제약 부분은 `[관측됨+외부실증]`, enabling 부분은 `[가설]`. amend 본문에서 sub-원칙별로 분리 라벨.
- Ratchet: **혼합** — Surgical Changes (c) 카테고리만 강, 나머지 전체 약.

### 분류
- **우선순위: P0** (정책 SSOT 정착, item 1과 같은 task)
- **Ratchet: 혼합 (강 + 약)**

---

## 12. (drift fix) `validate-workitem/SKILL.md`에 FAC → AC spec coverage audit 추가

### 현재 상황

[.claude/skills/validate-workitem/SKILL.md:26-33](.claude/skills/validate-workitem/SKILL.md) 검증 기준 7항목: 문서 범위 정합 / 범위 밖 변경 / 검증 포인트 / regression risk / 통합 검증 명령 결과 / AC ↔ 테스트 매핑 / 테스트 선행 휴리스틱.

[.claude/agents/validator.md:45](.claude/agents/validator.md): *"feature `## 7 FAC`의 각 항목이 task `## 6 AC`로 매핑됐는가? 매핑 안 된 FAC가 있으면 report에 `Spec Gap: FAC-N → unmapped` 명시 + 미커버 task 추가 권장 (자동 차단 X — ADR-007 책임 경계 정합)."*

### 문제

같은 audit 책무가 **validator agent body에는 있으나 validate-workitem skill body에는 없다**. [ADR-010](docs/90-decisions/boilerplate/ADR-010-multi-agent-compatibility.md) D3에 의해 *SSOT는 skill body* — 즉:

- Claude 호출 시 validator subagent로 위임되면 agent body가 적용됨 → Spec Gap 검출 가능.
- **Codex wrapper(`$validate-workitem`) 또는 skill 본문만 따르는 호출에서는 누락됨**.

이건 **ADR-010 multi-tool parity 정면 위반** + [ADR-037](docs/90-decisions/boilerplate/ADR-037-spec-coverage-audit.md) "Spec coverage self-audit" 정책의 *cross-tool 불균등 작동*.

### 개선 방안

[.claude/skills/validate-workitem/SKILL.md:26-33](.claude/skills/validate-workitem/SKILL.md) 검증 기준 7항목 마지막에 1줄 추가:

```
- FAC → AC spec coverage audit (ADR-037):
  feature `## 7 FAC`의 각 항목이 본 task의 `## 6 AC` 또는 *연관 task의 AC*에
  매핑되는가? 매핑 안 된 FAC가 있으면 report의 "Spec coverage" 섹션에
  `Spec Gap: FAC-N → unmapped` 명시 + 미커버 task 추가 권장.
  자동 차단 X — ADR-007 책임 경계 정합.
```

[.claude/skills/validate-workitem/SKILL.md:38-61](.claude/skills/validate-workitem/SKILL.md) report 양식의 "## AC ↔ 테스트 매핑" 섹션 *바로 뒤*에 새 섹션 1개 추가:

```markdown
## Spec coverage (FAC ↔ AC, ADR-037)
- FAC-1: ✅ T-001:AC-1
- FAC-2: ✅ T-001:AC-2
- FAC-3: ❌ unmapped — 미커버 task 추가 권장 (예: T-XXX [Given]...[When]...[Then]...)
```

### 근거

- 정책 SSOT: [ADR-010](docs/90-decisions/boilerplate/ADR-010-multi-agent-compatibility.md) D3 (skill body = canonical SSOT) + [ADR-037](docs/90-decisions/boilerplate/ADR-037-spec-coverage-audit.md).
- Evidence label: `[관측됨]` — validator.md ↔ validate-workitem SKILL.md *직접 비교 확인*. 동일 책무가 한 surface에만 박혀 cross-tool drift.
- Ratchet: **제약 정책(강)** — 기존 책무의 *skill body 동등 복제*. 새 정책 도입 X.

### 분류
- **우선순위: P0** (cross-tool drift fix)
- **Ratchet: 강**

---

## 13. (drift fix) `review-doc/SKILL.md` 권한 ↔ 책무 내부 모순 해소

### 현재 상황

[.claude/skills/review-doc/SKILL.md:9](.claude/skills/review-doc/SKILL.md) frontmatter:

```yaml
allowed-tools: Read Glob Grep
```

본문 line 28-32 (4곳):
- *"mismatch 발견 시 `docs/40-validation/IMPROVEMENT_GUIDE.md`에 **P1 severity**로 보고."*
- *"AGENTS.md 길이 점검: 100줄 초과 시 IMPROVEMENT_GUIDE에 P0 severity로 보고. 80~100줄 사이는 P1."*
- *"새 dropped 번호 발견 시 P2 보고."*
- *"누락 발견 시 P1 보고."*
- *"docs/00-meta/ 파일 수가 ... 위반 시 P0 보고."* (line 32)

### 문제

`allowed-tools`에 Write/Edit가 없는데 본문이 *"IMPROVEMENT_GUIDE.md에 보고"*를 5곳에서 명령. **물리적 실행 불가** — `Read Glob Grep`만으론 파일 쓸 수 없음.

해석 가능성 두 가지:
- (A) "보고"는 *출력만* — 본문이 동사를 잘못 씀.
- (B) "보고"는 *실제 기록* — `allowed-tools`가 잘못 정의됨.

현재 본문 동사가 명백히 *기록* 의미라 (B)가 의도. [ADR-007](docs/90-decisions/boilerplate/ADR-007-workitem-lifecycle.md) 책임 경계 정합 — validator도 `Write` 권한으로 report 기록함.

### 개선 방안

[.claude/skills/review-doc/SKILL.md:9](.claude/skills/review-doc/SKILL.md) frontmatter 갱신:

```yaml
# 전
allowed-tools: Read Glob Grep

# 후
allowed-tools: Read Glob Grep Write Edit
```

Write/Edit를 **unscoped로 추가** + 본문 강한 제한 텍스트로 SSOT 분리. *path-scoped `Write(<path>)` syntax는 보일러플레이트의 다른 어디에도 사용되지 않아 frontmatter 동작 미검증* — false safety 회피 차원에서 unscoped + 본문 제약 패턴 채택 (validate-workitem이 `Bash(<cmd>)` path-scoped만 사용하는 것과 정합).

[.claude/skills/review-doc/SKILL.md:34](.claude/skills/review-doc/SKILL.md) "마지막 출력" 섹션 *직전*에 1단락 추가 (강제 제한):

```
본 skill은 발견 항목을 **`docs/40-validation/IMPROVEMENT_GUIDE.md` 단일 파일에만** 누적 기록한다.
그 외 어떤 파일도 Write/Edit 금지 — 본 제약은 frontmatter가 아닌 본문이 SSOT다.
본문 외 변경이 필요해 보이면 출력에 "후속 task 권장" 텍스트만 남긴다 — `/plan-workitem` 또는 사용자가 후속 발화.
위반 발견 시 IMPROVEMENT_GUIDE.md에 *self-report* (예: `P1 [Self-violation] review-doc edited <file>`)
+ 다음 라운드 stabilize가 회수.
```

본문 텍스트가 frontmatter보다 강하지만 *validator agent가 호출 시 본문 정합 점검* + reviewer가 stabilize에서 self-violation 발견 시 P1 보고 → 다층 방어.

### 근거

- 정책 SSOT: [ADR-007](docs/90-decisions/boilerplate/ADR-007-workitem-lifecycle.md) 책임 경계 (validator 패턴 정합).
- Evidence label: `[관측됨]` — frontmatter ↔ 본문 *직접 모순* 직접 확인.
- Ratchet: **제약 정책(강)** — 자기 모순 해소. 새 정책 도입 X.

### 분류
- **우선순위: P0** (내부 모순 fix)
- **Ratchet: 강**

---

## 14. (drift fix) `stabilize-milestone/SKILL.md` 회고 책임 경계 명시

### 현재 상황

[.claude/skills/stabilize-milestone/SKILL.md:10-12](.claude/skills/stabilize-milestone/SKILL.md):

> 이 skill은 **코드 수정·커밋·workitem status 변경을 하지 않는다.**
> 검증 결과를 `docs/40-validation/QA_FINDINGS.md`와 `docs/40-validation/IMPROVEMENT_GUIDE.md`에 누적 기록하는 문서 갱신은 정상 책임이다 — 그 외 변경은 금지한다.

[.claude/skills/stabilize-milestone/SKILL.md:57](.claude/skills/stabilize-milestone/SKILL.md):

> milestone 문서의 `## 8. 회고` 섹션을 stabilize 종료 시점에 자동 채운다 (ADR-014). 목표 달성도 / scope creep / 비목표 위반 / 핵심 학습 3개 이내.

### 문제

milestone 문서는 *workitem* ([ADR-007](docs/90-decisions/boilerplate/ADR-007-workitem-lifecycle.md) 1단계 산출물). 따라서:

- line 10–12: *"workitem status 변경 X"* + *"그 외 변경 금지"*.
- line 57: *milestone 문서의 한 섹션을 *자동 채운다***.

**같은 skill 내부에서 정면 모순**. 정확한 해석은 "status 변경 X / 본문 회고 섹션 갱신 O"인데, 책임 경계 단락이 *"그 외 변경 금지"*라 명시 예외 없으면 모호.

[ADR-014](docs/90-decisions/boilerplate/ADR-014-milestone-graduation.md)에는 회고 자동 채움이 박혀 있지만, **skill 본문 책임 경계 단락에 *예외 명시*가 없으면** Codex wrapper 또는 자연어 호출에서 *책임 경계만 읽고 회고 갱신을 건너뛸* 위험.

### 개선 방안

[.claude/skills/stabilize-milestone/SKILL.md:10-12](.claude/skills/stabilize-milestone/SKILL.md) 책임 경계 단락을 다음으로 갱신:

```
# 전
이 skill은 **코드 수정·커밋·workitem status 변경을 하지 않는다.**
검증 결과를 `docs/40-validation/QA_FINDINGS.md`와 `docs/40-validation/IMPROVEMENT_GUIDE.md`에 누적 기록하는 문서 갱신은 정상 책임이다 — 그 외 변경은 금지한다.

# 후
이 skill은 **코드 수정·커밋·workitem status 변경을 하지 않는다.**
다음 세 종류의 문서 갱신만 정상 책임이다:
1. `docs/40-validation/QA_FINDINGS.md` 누적 기록 (qa 위임 결과).
2. `docs/40-validation/IMPROVEMENT_GUIDE.md` 누적 기록 (reviewer 위임 결과 + item 21 deterministic preflight 결과).
3. milestone 문서의 `## 8. 회고` 섹션 자동 채움 ([ADR-014](../../../docs/90-decisions/boilerplate/ADR-014-milestone-graduation.md) graduation contract — status 변경 X, 본문 단락 갱신만).

그 외 변경은 금지한다 — milestone 문서의 `## 0. Status` / `## 1~7` 섹션 / 다른 workitem 문서 / 코드 일체.
```

[.claude/skills/stabilize-milestone/SKILL.md:54-57](.claude/skills/stabilize-milestone/SKILL.md) 본문 끝의 "책임 경계" 단락은 *재확인*만 남기고 본문은 위 도입부로 SSOT 이동:

```
# 후
책임 경계:
- 코드 수정·커밋·workitem status 변경 금지.
- 누적 문서 갱신 + milestone `## 8. 회고` 자동 채움 (상세는 본 skill 도입부 책임 경계 단락 참조).
```

### 근거

- 정책 SSOT: [ADR-007](docs/90-decisions/boilerplate/ADR-007-workitem-lifecycle.md) 책임 경계 + [ADR-014](docs/90-decisions/boilerplate/ADR-014-milestone-graduation.md) 회고.
- Evidence label: `[관측됨]` — skill 본문 내부 *직접 모순* 직접 확인.
- Ratchet: **제약 정책(강)** — 자기 모순 해소. 새 정책 도입 X.

### 분류
- **우선순위: P0** (내부 모순 fix)
- **Ratchet: 강**

---

## 15. (drift fix) `plan-workitem` FAC↔AC 매핑표 문서 영속화

### 현재 상황

[.claude/skills/plan-workitem/SKILL.md:66-72](.claude/skills/plan-workitem/SKILL.md) "마지막 출력" 섹션:

```
- `## 8. FAC ↔ AC 매핑표` (feature 분해 시 — plan 출력에 섹션 헤딩으로 박힘, ADR-037):
  FAC-1 → T-001:AC-1, T-002:AC-2
  FAC-2 → T-003:AC-1
  FAC-3 → unmapped  ← 미커버 task 필요
```

**plan 출력 텍스트에만** 박힘 — feature 문서 본문 또는 task 본문에 영속 저장 X.

[FEATURE_TEMPLATE.md](docs/30-workitems/_templates/FEATURE_TEMPLATE.md)에도 FAC↔AC 매핑 자리 부재.

### 문제

`/plan-workitem`의 출력 텍스트는 **세션 종료 시 사라짐**. 다음 라운드에서 `/validate-workitem` (item 12) 또는 `/stabilize-milestone` (item 21)이 FAC unmapped를 점검하려 해도 plan 출력은 이미 없음.

→ [ADR-037](docs/90-decisions/boilerplate/ADR-037-spec-coverage-audit.md)의 "Spec coverage self-audit" 정책이 *plan 단계에서만 surface*되고 *workitem 진화 동안 추적 끊김*. validator와 stabilize-milestone이 재점검할 surface가 *없는 상태*.

### 개선 방안

[FEATURE_TEMPLATE.md](docs/30-workitems/_templates/FEATURE_TEMPLATE.md)에 새 **subsection** 추가 (`## 7. FAC` 섹션의 *하위 7-1로* — 12 main sections + 1 mapping subsection 형식, ADR-036 12-섹션 구조 유지):

```markdown
## 7-1. FAC ↔ AC 매핑표 (subsection of ## 7)
<!-- /plan-workitem이 task 분해 시 본 subsection을 채운다 (영속 SSOT — plan 출력은 echo).
     형식: FAC-N → T-NNN:AC-N, T-MMM:AC-M (다대다 허용)
     unmapped 항목은 미커버 task 추가 권장 — validator(ADR-037) 및 stabilize preflight가 재점검.
     본 subsection은 ## 7 FAC와 한 묶음 — ADR-036 12-섹션 구조에 *추가 main section 신설 X*. -->
- FAC-1 →
- FAC-2 →
- FAC-3 →
```

[.claude/skills/plan-workitem/SKILL.md:39-40](.claude/skills/plan-workitem/SKILL.md) "feature 분해 시 (ADR-036)" 단락 갱신:

```
# 전
feature 분해 시 12섹션 모두 채운다. `## 7 FAC`는 task `## 6 AC`로 분해되며 매핑 누락 시 plan 출력의 "남은 미결정 사항"에 명시.

# 후
feature 분해 시 12 main sections + `## 7-1` mapping subsection 모두 채운다.
`## 7 FAC`는 task `## 6 AC`로 분해되며 매핑 결과는 **feature 문서의 `## 7-1. FAC ↔ AC 매핑표` subsection에 영속 저장** (출력만 X — drift 차단).
매핑 누락(unmapped FAC)은 plan 출력의 "남은 미결정 사항"에 *추가*로 명시.
다음 라운드의 [validate-workitem](../validate-workitem/SKILL.md) Spec coverage audit (item 12)
및 [stabilize-milestone deterministic preflight](../stabilize-milestone/SKILL.md) (item 21)이
본 영속 표를 참조해 cross-round 추적.
```

**Legacy fallback 명시** — 기존 feature 문서(템플릿 변경 *전*에 생성된 것)에는 `## 7-1` subsection이 부재할 수 있다. validator(item 12) / stabilize preflight(item 21)는 다음 순서로 회수:

1. `## 7-1` subsection 존재 → 본문 매핑 표 직접 점검.
2. 부재 → `## 7 FAC` 본문에서 *inline 매핑 표기*(예: `- FAC-1 → T-001:AC-1`) 휴리스틱 검출.
3. 둘 다 부재 → `Spec Gap: <feature> 매핑표 부재 — legacy 문서 보강 권장` 라벨로 IMPROVEMENT_GUIDE에 P1 기록 + 다음 plan 라운드에서 `## 7-1` 보강.

[.claude/skills/plan-workitem/SKILL.md:66-72](.claude/skills/plan-workitem/SKILL.md) "마지막 출력" 섹션의 매핑표 부분 갱신:

```
# 전
- `## 8. FAC ↔ AC 매핑표` (feature 분해 시 — plan 출력에 섹션 헤딩으로 박힘, ADR-037):

# 후
- feature 분해 시: 매핑표를 *feature 문서의 `## 7-1`에 직접 기록* + plan 출력 요약에 동일 표 echo
  (`## 7-1` 본문이 SSOT, plan 출력은 사람 확인용).
```

### 근거

- 정책 SSOT: [ADR-037](docs/90-decisions/boilerplate/ADR-037-spec-coverage-audit.md) (Spec coverage audit이 *cross-round 추적 surface 필요*) + [ADR-005](docs/90-decisions/boilerplate/ADR-005-ssot.md) (출력 X 영속 문서 O).
- Evidence label: `[관측됨]` — plan-workitem 출력만 있고 영속 자리 부재 직접 확인.
- Ratchet: **제약 정책(강)** — ADR-037 정책 surface drift fix.

### 분류
- **우선순위: P0** (cross-round 추적 surface 부재)
- **Ratchet: 강**

---

## 16. (핵심 강화) `/stack-guard` 생성 후 `validate` 1회 smoke test

### 현재 상황

[.claude/skills/stack-guard/SKILL.md:37-48](.claude/skills/stack-guard/SKILL.md) 수행 단계: `validate` 진입점 생성 + verify 스크립트 생성 + STACK_SETUP_PLAN.md append + `.gitattributes` 생성. **생성 후 실제로 `validate` 명령을 1회 실행해 통과/실패를 확인하는 단계 없음**.

[.boilerplate/validation/SIMULATION_RUN.md:12](.boilerplate/validation/SIMULATION_RUN.md): Round 1에서 *"pnpm validate 기본 가정이 Node.js v22.12.0 환경에서 pnpm v11.1.2 버전 비호환으로 실패 → npm run validate로 대체"* — **stack-guard가 만든 명령이 실 환경에서 실패한 사례 직접 관측**.

### 문제

`/stack-guard`는 *장기 운영의 핵심 검증 명령 생성기*이다. 생성된 `validate` 명령이 실제로 통과/실패를 *반환*하지 않으면 [validate-workitem](.claude/skills/validate-workitem/SKILL.md), [finalize-workitem](.claude/skills/finalize-workitem/SKILL.md), [stabilize-milestone](.claude/skills/stabilize-milestone/SKILL.md) 세 lifecycle 단계가 *모두 같은 잘못된 가정 위에서 작동*. 사용자가 발견하는 시점은 [validate-workitem](.claude/skills/validate-workitem/SKILL.md) 첫 실행 후 — 이미 builder가 RGR 1+ 사이클 돈 후.

### 개선 방안

[.claude/skills/stack-guard/SKILL.md:37-48](.claude/skills/stack-guard/SKILL.md) 수행 단계 끝(4번 뒤)에 새 단계 1개 추가:

```
5. **Smoke test (필수)**: 생성된 `validate` 명령을 1회 실행한다 (`allowed-tools`의 Bash 권한 활용 — 신규 권한 추가 불필요).
   - 통과: 그대로 진행, 마지막 출력에 `validate smoke test: PASS` 명시.
   - 실패 + *명령 자체의 실패*(예: 명령 없음 / 패키지 매니저 비호환): 사용자에게 *생성된 명령 + 실패 stderr + 제안 대체*를 출력하고 **수정 안내 후 종료**.
     - 예: pnpm 버전 비호환 발견 시 npm run validate로 대체 권장.
   - 실패 + *빈 프로젝트 정상 케이스*(예: 비어있는 lint 룰 / 테스트 0건): 정상으로 간주, 출력에 `validate smoke test: PASS (empty rules/tests warning)` 명시.
   - 실패 + *린트/테스트 자체의 실 위반*: 빈 프로젝트가 아니라 실제 에러 — 사용자에게 직접 수정 안내 후 종료.
```

[.claude/skills/stack-guard/SKILL.md:49-54](.claude/skills/stack-guard/SKILL.md) "마지막 출력" 단락에 1줄 추가:

```
- validate smoke test 결과 (PASS / PASS with warning / FAIL with stderr 요약)
```

### 근거

- 정책 SSOT: [ADR-007](docs/90-decisions/boilerplate/ADR-007-workitem-lifecycle.md) lifecycle 신뢰성 + [ADR-006](docs/90-decisions/boilerplate/ADR-006-simplicity-and-architecture.md) (자기 검증 가능한 산출물만 박는다).
- Evidence label: `[관측됨]` — SIMULATION_RUN.md Round 1에서 *생성된 명령이 실 환경 실패* 직접 관측. 가설 X.
- Ratchet: **제약 정책(강)** — 생성된 산출물의 *최소 자기 검증* 필수.

### 분류
- **우선순위: P0** ([관측됨] 마찰점 직접 해소)
- **Ratchet: 강**

---

## 17. (핵심 강화) `/stack-guard` Codex wrapper 승격

### 현재 상황

[README.md:94](README.md): *"Remaining skills (discover-product, stack-guard, review-doc, boilerplate-context, bootstrap-design) are invoked via natural language."*

[ADR-010](docs/90-decisions/boilerplate/ADR-010-multi-agent-compatibility.md) "후속 작업" Phase 2: *"discover-product, stack-guard, review-doc, boilerplate-context 4개. 호출 빈도 낮아 자연어 호출로 충분."*

### 문제

`/stack-guard`는 **장기 운영에서 *검증 계약*의 SSOT**다. 호출 빈도는 낮지만 *실패 시 영향이 lifecycle 전체*에 미친다 (item 16의 SIMULATION_RUN 관측이 직접 증거). [ADR-010](docs/90-decisions/boilerplate/ADR-010-multi-agent-compatibility.md) Phase 2 보류 판단은 *"빈도"*를 기준으로 했으나, *"영향도"* 기준으로 재평가 필요.

Codex wrapper 부재의 실질 영향:
- Codex 사용자가 자연어로 호출 → skill 본문이 *완전 정합으로 적용된다는 보장 약함* (자연어 → SKILL.md 매칭은 LLM 휴리스틱).
- inner-loop의 다른 skill 4개(`implement/validate/repair/finalize`) + planning/bootstrap/stabilize 4개는 wrapper로 박혀 *동등 진입점*인데 **stack-guard만 자연어** — Codex 사용자가 *왜 이것만 다른가* 혼란.

### 개선 방안

**(a) Wrapper 신설** — [.agents/skills/stack-guard/SKILL.md](.agents/skills/stack-guard/SKILL.md) 신설. 기존 wrapper 패턴(`.agents/skills/plan-workitem/SKILL.md` 등)과 **동일 형식**:

```markdown
---
name: stack-guard
description: Use ONLY when the user explicitly types `$stack-guard [stack summary]`. Do not trigger implicitly from generic phrasing.
---

Source of truth: `.claude/skills/stack-guard/SKILL.md`. Read it and follow the workflow.

Treat all frontmatter keys other than `name` and `description` (e.g., `agent:`, `disable-model-invocation:`, `allowed-tools:`, `context:`, `argument-hint:`, `model:`, `effort:`) as Claude-only and ignore them — execute locally in Codex.

**Slash command translation**: 본문 안의 `/stack-guard` 표기는 Claude 슬래시 커맨드다. Codex에서는 `$stack-guard`으로 읽고 사용자에게 안내한다. Codex CLI는 `/`를 빌트인 슬래시 커맨드에 쓰므로 명시적 치환이 필요.

Preserve all repo policies from `AGENTS.md` and `docs/`.

If the source path no longer exists, this wrapper is stale — see ADR-010.
```

**주의**: `name:` 값은 `$` *없이* (`stack-guard`) 박는다 — Codex가 `$` prefix를 사용자 입력 surface로만 쓰고 wrapper 정의는 plain name. 기존 wrapper 8종(`plan/implement/validate/repair/finalize/bootstrap-project/bootstrap-stack/stabilize-milestone`)과 동일 패턴.

**(b) README 갱신** — [README.md:94](README.md):

```
# 전
Core workflow skills have Codex wrappers ($-prefixed): $implement-workitem, $validate-workitem, $repair-workitem, $finalize-workitem, $plan-workitem, $bootstrap-project, $bootstrap-stack, $stabilize-milestone. Remaining skills (discover-product, stack-guard, review-doc, boilerplate-context, bootstrap-design) are invoked via natural language.

# 후
Core workflow skills have Codex wrappers ($-prefixed): $implement-workitem, $validate-workitem, $repair-workitem, $finalize-workitem, $plan-workitem, $bootstrap-project, $bootstrap-stack, $stabilize-milestone, **$stack-guard**. Remaining skills (discover-product, review-doc, boilerplate-context, bootstrap-design) are invoked via natural language.
```

README_ko.md도 동등 갱신.

**(c) ADR-010 Amendment 1** — Phase 2 보류 결정은 *정책*이므로 inline patch가 아닌 [Amendment 1로 박는다](docs/90-decisions/boilerplate/_ADR_GUIDE.md) ([ADR-005](docs/90-decisions/boilerplate/ADR-005-ssot.md) 패턴 4 정합). item 11이 ADR-006 Amendment 1을 박은 패턴과 동일:

```markdown
## Amendment 1 (2026-05-16) — Phase 2.5: stack-guard wrapper 승격

### 결정

기존 Phase 2 보류 4개 skill 중 **stack-guard 1개를 wrapper로 승격**한다. 나머지 3개(discover-product, review-doc, boilerplate-context)는 Phase 2 보류 유지.

### 근거

- 본 ADR Phase 2 보류 판단은 *"호출 빈도 낮음"* 기준이었으나, *영향도* 기준 재평가 결과 stack-guard만 별도 분류 필요.
- [SIMULATION_RUN.md Round 1](../../.boilerplate/validation/SIMULATION_RUN.md) 직접 관측: 생성된 `validate` 명령이 실 환경 실패 → lifecycle 전체(validate-workitem / finalize-workitem / stabilize-milestone) 신뢰성 영향.
- 빈도는 낮으나 *실패 시 영향이 lifecycle 전체*에 미친다 — 자연어 호출이 *완전 정합 보장이 약한* surface를 inner-loop와 동등 wrapper로 박는다.

### 적용 surface

- `.agents/skills/stack-guard/SKILL.md` wrapper 신설 (기존 wrapper 8종과 동일 패턴 — `name:` 값은 `$` 없이).
- `README.md` / `README_ko.md`에서 `stack-guard`를 *Core workflow Codex wrapper* 목록으로 이동.
- 본 ADR D3·D4·D6은 변경 없음 — wrapper 본문은 여전히 `.claude/skills/<name>/SKILL.md` SSOT를 가리키는 얇은 stub.

### 후속 작업

- 향후 Phase 3에서 나머지 3개(discover-product, review-doc, boilerplate-context) 승격 여부 재평가는 *fork 데이터 회수 후* 결정 (현재 0건).
```

[docs/90-decisions/boilerplate/README.md](docs/90-decisions/boilerplate/README.md)의 ADR-010 행에 *(+amend1: Phase 2.5 stack-guard wrapper 승격)* 박음.

### 근거

- 정책 SSOT: [ADR-010](docs/90-decisions/boilerplate/ADR-010-multi-agent-compatibility.md) D4 (Codex wrapper 패턴).
- Evidence label: `[관측됨+가설]` — SIMULATION_RUN.md Round 1 마찰점이 *영향도 [관측됨]* evidence (단순 가설 X). 단 *wrapper 도입이 영향 완화*는 [가설] — 종합.
- Ratchet: **enabling 정책(약)** — wrapper는 진입점 표면화일 뿐 새 정책 도입 X.

### 분류
- **우선순위: P1** (영향도는 P0급이나 변경 surface는 enabling)
- **Ratchet: 약**

---

## 18. (핵심 강화) `/stack-guard` `${CLAUDE_PROJECT_DIR}` + exec form 표준화

### 현재 상황

[GUARDRAILS_STRATEGY.md:80-92](docs/00-meta/GUARDRAILS_STRATEGY.md) "PostToolUse hook 매뉴얼 등록 절차"의 예시:

```json
{
  "hooks": {
    "PostToolUse": [
      { "matcher": "Edit|Write", "hooks": [{ "type": "command", "command": "pnpm validate" }] }
    ]
  }
}
```

`command: "pnpm validate"` — **bare command, CWD 가정 의존**. 또한 [.claude/skills/stack-guard/SKILL.md:38](.claude/skills/stack-guard/SKILL.md)의 verify 스크립트 호출 경로도 동등 가정.

### 문제

Anthropic 공식 hooks docs + open issue #50960 (다중 reproducer 보고, 2026년 기준 미해결): Windows 환경에서 sub-agent fork 후 CWD가 phantom 경로(예: `.claude\claims\.claude\hooks\...`)로 drift. bare `pnpm validate`는 *전혀 다른 디렉터리에서 실행*돼 *조용히 실패*.

또한 `.cmd`/`.bat` shim(npm, npx, eslint 등)은 exec form (`type: "command"` + `args` 배열)으로 spawn 불가 — `shell: "powershell"` 또는 `node <스크립트>` 직접 호출 필요.

이 결함은 **`/stack-guard`가 만드는 매뉴얼 등록 예시 + verify 스크립트 호출 경로 둘 다 영향**.

### 개선 방안

[GUARDRAILS_STRATEGY.md:80-92](docs/00-meta/GUARDRAILS_STRATEGY.md) "PostToolUse hook 매뉴얼 등록 절차" 예시 본문을 다음 2 OS 예시로 갱신.

**Unix/macOS 예시:**

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "if": "Edit(**/*.{ts,tsx,js,mjs,py})|Write(**/*.{ts,tsx,js,mjs,py})",
        "command": "${CLAUDE_PROJECT_DIR}/scripts/verify.sh",
        "args": ["--changed"]
      }]
    }]
  }
}
```

**Windows 예시 (PowerShell 또는 `.cmd` shim 대응):**

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "if": "Edit(**/*.{ts,tsx,js,mjs,py})|Write(**/*.{ts,tsx,js,mjs,py})",
        "command": "powershell",
        "args": ["-File", "${CLAUDE_PROJECT_DIR}/scripts/verify.ps1", "--changed"]
      }]
    }]
  }
}
```

**Schema 주의 (defensive 패턴)**: `matcher`는 *outer event entry*에 (도구 이름 필터), `if`는 *inner command handler*에 (확장자 등 fine-grained 필터) 둔다. `if`가 outer에서 작동하는 Anthropic docs 변형이 있을 수 있으나, 본 패턴은 *둘 다 받는 schema variant*에 대해 안전 — 첫 fork 적용 시 hooks docs(`code.claude.com/docs/en/hooks-guide`) 직접 확인 권장.

3 변경 사항:
1. `${CLAUDE_PROJECT_DIR}` 절대 경로 — CWD drift 회피.
2. `args` 배열 (exec form) — shell escaping 회피, `.cmd` shim 대응 (Windows는 `powershell` 또는 `node` 직접 호출).
3. `if` 필드 (matcher 대신) — 파일 확장자 필터링으로 hook 호출 빈도 감소.

[.claude/skills/stack-guard/SKILL.md:31-35](.claude/skills/stack-guard/SKILL.md) R0 단계에 다음 분기 박음:

```
- 단일 OS/셸이 Windows로 판정되면 `scripts/verify.ps1` 우선 + 매뉴얼 hook 예시는 PowerShell exec form (위 GUARDRAILS_STRATEGY Windows 예시).
- macOS/Linux 판정 또는 mixed env면 `scripts/verify.sh` 우선 + exec form 그대로 (Unix/macOS 예시).
- mixed env면 *두 verify 스크립트 모두 생성* (`.sh` + `.ps1`) + 두 hook 예시 모두 출력.
- 두 OS 모두 매뉴얼 hook 예시 본문은 `${CLAUDE_PROJECT_DIR}` + `args` 배열로 박는다 (Anthropic open issue #50960 다중 reproducer 대응).
```

### 근거

- 정책 SSOT: [GUARDRAILS_STRATEGY.md](docs/00-meta/GUARDRAILS_STRATEGY.md) PostToolUse hook 절차.
- Evidence label: `[관측됨+외부실증]` — *내부*: 본 저장소가 Windows 환경에서 작동 (사용자 환경 직접 관측). *외부*: Anthropic open issue #50960의 *다중 reproducer 보고*가 [ADR-022](docs/90-decisions/boilerplate/ADR-022-ratchet-principle.md) "다중 repo 실증" 기준 충족.
- Ratchet: **제약 정책(강)** — 산출 hook 예시의 *정확성*. 새 정책 도입 X (기존 예시의 정밀화).

### 분류
- **우선순위: P1** (산출물 정확성)
- **Ratchet: 강**

---

## 19. (핵심 강화) `/stack-guard` 스택별 verify 풀세트 — TS-first depth

### 현재 상황

[.claude/skills/stack-guard/SKILL.md:39](.claude/skills/stack-guard/SKILL.md): *"`scripts/verify.{sh,ps1,mjs,py}` 중 자연스러운 런타임 1종을 생성. 내용은 스택의 `lint + typecheck + test` 통합."*

→ "lint + typecheck + test 통합"이라는 **추상 문구만**. 스택별 도구 조합 default 미명시.

[ADR-021](docs/90-decisions/boilerplate/ADR-021-static-analysis-recommendation.md): 정적 분석 1종만 권장 (dependency-cruiser / import-linter 등). lint/format/typecheck 도구 조합은 비결정.

[ADR-031](docs/90-decisions/boilerplate/ADR-031-non-web-out-of-scope.md): 직접 지원 스택 5종 (web frontend / API / CLI / monorepo / Supabase). 사용자 의견(2026-05-XX 발언) — TS web app이 가장 큰 ratio + TS면 `biome 또는 prettier+linter + tsc` 조합.

### 문제

`/stack-guard` 출력이 *추상*이라:
- 사용자가 직접 verify.{ext}에 lint + typecheck + test 도구를 채워야.
- TS 사용자가 Biome / ESLint+Prettier / oxlint 중 선택 paralysis.
- *"validate가 정확히 무엇을 검증하는가"*의 표준 부재 → 다음 라운드의 [validate-workitem](.claude/skills/validate-workitem/SKILL.md)이 *각 fork마다 다른 의미*의 validate 결과를 받음.
- [SIMULATION_RUN.md:14](.boilerplate/validation/SIMULATION_RUN.md) Round 1: *"ESM 모듈 캐시 초기화 패턴이 skill 본문에 명시되어 있지 않아 직접 판단 필요"* — *스택별 default 부재*의 직접 관측 사례 (TS 영역).

### 개선 방안

[.claude/skills/stack-guard/SKILL.md](.claude/skills/stack-guard/SKILL.md) "정적 분석 도구 권장" 표 *바로 뒤*에 새 단락 추가:

```markdown
## 스택별 verify 풀세트 (default template, ADR-031 5 스택 정합)

| 스택 | format | lint | typecheck | unit test | e2e test |
|------|--------|------|-----------|-----------|----------|
| TS web (Next/Vite) | Biome (또는 Prettier) | Biome (또는 ESLint) | `tsc --noEmit` | Vitest | Playwright |
| TS API (Express/Fastify/Hono) | Biome (또는 Prettier) | Biome (또는 ESLint) | `tsc --noEmit` | Vitest | supertest 또는 동등 |
| TS CLI | Biome (또는 Prettier) | Biome (또는 ESLint) | `tsc --noEmit` | Vitest | (선택, snapshot) |
| Python | `ruff format` | `ruff` | `mypy --strict` (또는 pyright) | pytest | (선택, 스택별) |
| Go | `gofmt -l` | `golangci-lint` | `go vet` (built-in) | `go test` | (선택) |
| Rust | `cargo fmt --check` | `clippy` | `cargo check` | `cargo test` | (선택) |

생성된 `validate` 명령은 위 5단계를 *순서대로* 묶는다. 어느 한 단계라도 빠지면 출력에 *"missing: <단계>"* 명시. e2e는 `validate:e2e` 별도 명령으로 분리 (task 단위 finalize는 e2e 제외, milestone 단위 stabilize만 실행).

도구 선택은 **첫 fork에서 결정 + ARCHITECTURE_OVERVIEW.md `## 7-X`에 박힌다** — 이후 변경 시 [/bootstrap-stack](../bootstrap-stack/SKILL.md) 재실행 또는 수동 갱신.

**TS-first depth 권고**: TS 스택은 본 보일러플레이트 직접 지원 ratio가 가장 큼 → default를 `Biome (format+lint 통합) + tsc + Vitest + Playwright`로 박는다 (`Biome` 단일 선택으로 paralysis 차단). 사용자가 ESLint+Prettier 분리 선호 시 ARCHITECTURE_OVERVIEW.md에 명시 후 verify 갱신.

**도구 감지 우선 순서** (기존 프로젝트에 fork되는 경우):

1. **감지**: `package.json` 의존성·devDependencies / `.eslintrc*` / `.prettierrc*` / `biome.json` / `vitest.config.*` / `jest.config.*` / `playwright.config.*` 등 *기존 도구 흔적* 먼저 확인.
2. **존재 → 그대로 사용**: 위 도구 중 어느 것이 *이미 박혀 있으면* default로 *덮어쓰지 않는다*. 예: ESLint+Prettier+Jest가 박힌 프로젝트에 Biome+Vitest를 강제 install 금지. 발견 도구를 ARCHITECTURE_OVERVIEW.md `## 7-X`에 기록.
3. **부재 → Biome+tsc+Vitest+Playwright default 박음**: green-field 또는 도구 미정 프로젝트에만 적용.
4. **충돌(Biome ↔ ESLint+Prettier 둘 다 박힘 등)**: 사용자에게 출력으로 보고 + 결정 요청. 자동 선택 X.
```

`/stack-guard` "마지막 출력" 섹션에 1줄 추가:

```
- 스택별 default verify template은 본 skill의 "스택별 verify 풀세트" 표 기준. 도구 변경 시 ARCHITECTURE_OVERVIEW.md ## 7-X 갱신.
```

### 근거

- 정책 SSOT: [ADR-021](docs/90-decisions/boilerplate/ADR-021-static-analysis-recommendation.md) (paralysis 방지 — 스택별 1종 권장) + [ADR-031](docs/90-decisions/boilerplate/ADR-031-non-web-out-of-scope.md) (직접 지원 스택 5종).
- Evidence label: `[가설→실증]` — 사용자 의견 (TS web 중심 + Biome/Prettier+linter+tsc 조합)은 단일 testimony라 *outcome 검증*은 아님. 단 SIMULATION_RUN.md Round 1의 마찰점이 *추상 문구 cause*를 뒷받침 → 합성으로 [가설→실증] (Round 3 시뮬레이션 시 [관측됨] 회수 가능).
- Ratchet: **enabling 정책(약)** — default template은 시작점 권장, 사용자 변경 가능.

### 분류
- **우선순위: P1** ([관측됨] 마찰 완화 + 사용자 의견 정합)
- **Ratchet: 약**

---

## 20. (핵심 강화) PostToolUse hook async adapter — 옵션 출력으로 박기

### 현재 상황

[GUARDRAILS_STRATEGY.md:50-60](docs/00-meta/GUARDRAILS_STRATEGY.md) "1단계 비범위" 사유:
> PostToolUse hook 자동 등록 — `acceptEdits` 모드에서 매 Edit/Write마다 lint를 자동 실행하면 비용이 폭증할 수 있다 ... prototyping 단계에서 ... 측정한 뒤 별도 항목으로 분리한다.

[.claude/skills/stack-guard/SKILL.md:20](.claude/skills/stack-guard/SKILL.md) 동등 명시: *"1단계 비범위: PostToolUse hook 자동 등록은 본 skill에서 수행하지 않는다."*

### 문제

Anthropic 공식 hooks docs (2026)가 다음 3 패턴 명시:
- `async: true` — 비차단 백그라운드 실행.
- `asyncRewake: true` — 백그라운드 실행 + 실패 시(exit 2)만 Claude 깨워서 stderr를 system reminder로 주입.
- `if: "Edit(*.ts)|Write(*.ts)"` — 파일 패턴 필터링.

이 3 패턴이 *비용 폭증 사유*를 해결. 단 **async hook은 *차단형 게이트가 아닌 보조 루프***. canonical 검증 책임은 여전히 [validate-workitem](.claude/skills/validate-workitem/SKILL.md), [finalize-workitem](.claude/skills/finalize-workitem/SKILL.md), [stabilize-milestone](.claude/skills/stabilize-milestone/SKILL.md)의 동기 `validate` 호출이 가짐.

또한 [ADR-010](docs/90-decisions/boilerplate/ADR-010-multi-agent-compatibility.md) multi-tool parity: `.claude/settings.json`에 hook을 *shared로 박으면 Codex parity 깨짐*. canonical 검증은 `validate` 스크립트(도구 중립), hook은 *Claude-only adapter*.

### 개선 방안

[GUARDRAILS_STRATEGY.md:50-60](docs/00-meta/GUARDRAILS_STRATEGY.md) "1단계 비범위" 사유 단락 갱신:

```
# 전
**1단계 비범위 (prototyping 후 분리)**:
- PostToolUse hook 자동 등록 — `acceptEdits` 모드에서 ... 비용이 폭증할 수 있다 ... 별도 항목으로 분리한다.

# 후
**1단계 비범위 (사용자 옵션 — shared 자동 등록 X)**:
- PostToolUse hook은 본 1단계에서 **`.claude/settings.json` shared에 자동 박지 않는다** ([ADR-010](../90-decisions/boilerplate/ADR-010-multi-agent-compatibility.md) multi-tool parity 정합 — canonical 검증은 `validate` 스크립트, hook은 Claude-only adapter).
- Anthropic 2026 hooks docs의 `async: true` / `asyncRewake: true` / `if: "..."` 3 패턴이 *비용 폭증 우려*를 해결하므로, `/stack-guard`는 **이 패턴 예시를 *옵션 출력*으로 박는다** (사용자가 채택 시 [.claude/settings.local.json](../../.claude/settings.local.json)에 복사).
- 비범위 → 옵션 surface 이동: 기존 *"prototyping 후 자동화 예정"*은 더 이상 명시 X — Anthropic API가 이미 해결한 영역.
- **canonical 검증은 hook 도입 여부와 무관하게 작동** — `/validate-workitem`, `/finalize-workitem`, `/stabilize-milestone` 각각이 동기 `validate` 호출을 가짐 (ADR-007 lifecycle 정합).
```

[.claude/skills/stack-guard/SKILL.md:53](.claude/skills/stack-guard/SKILL.md) "마지막 출력" 단락에 *옵션 출력* 항목 추가:

```
- **옵션: Claude PostToolUse async adapter 예시** (사용자가 채택 시 `.claude/settings.local.json`에 복사):

```jsonc
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "if": "Edit(**/*.{ts,tsx,js,mjs,py})|Write(**/*.{ts,tsx,js,mjs,py})",
        "command": "${CLAUDE_PROJECT_DIR}/scripts/verify.sh",
        "args": ["--changed"],
        "async": true,
        "asyncRewake": true
      }]
    }]
  }
}
```

**Schema 주의**: item 18과 동일 defensive 패턴 — `matcher` outer (도구 이름 필터, 항상 작동) + `if` inner (확장자 필터, schema variant). 첫 fork 적용 시 Anthropic hooks docs 직접 확인 권장.

도입은 사용자 결정. 본 hook은 *조기 피드백 adapter* — 실패 시 Claude를 깨워 stderr를 system reminder로 주입. **차단형 게이트 아님** (완료 판정은 동기 `validate-workitem` / `finalize-workitem` / `stabilize-milestone`이 책임).
```

### 근거

- 정책 SSOT: [GUARDRAILS_STRATEGY.md](docs/00-meta/GUARDRAILS_STRATEGY.md) + [ADR-010](docs/90-decisions/boilerplate/ADR-010-multi-agent-compatibility.md) (multi-tool parity).
- Evidence label: `[가설]` — Anthropic 공식 docs의 *API 존재*는 factual 사실이지만 *async hook이 본 보일러플레이트 fork에서 outcome 개선*은 fork 데이터 부재. ADR-022 정합으로 [가설]. *비범위 사유 갱신* 자체는 factual (Anthropic API change).
- Ratchet: **enabling 정책(약)** — 옵션 출력만, 자동 적용 X. ADR-022 "가설만 있다면 enabling" 정합.

### 분류
- **우선순위: P1** (장기 운영 UX 개선 옵션, Codex parity 유지)
- **Ratchet: 약**

---

## 21. (핵심 강화) `/stabilize-milestone` deterministic pre-flight 신설

### 현재 상황

[.claude/skills/stabilize-milestone/SKILL.md:18-19](.claude/skills/stabilize-milestone/SKILL.md) 수행 step 1: milestone 문서 읽고 feature/task 목록 회수. step 1.5: graduation pre-check.

step 3–6: qa/reviewer 위임 (LLM 비용 큰 경로).

### 문제

LLM 위임 *전에* deterministic하게 검증 가능한 cheap mechanical check가 별 surface 없이 LLM에 위임됨:
- docs/ 내부 markdown link 유효성 — `markdown-link-check`로 충분.
- ADR 참조(`[ADR-NNN]`)의 실제 파일 존재 여부 — `glob` + 정규식으로 충분.
- feature `## 7-1. FAC ↔ AC 매핑표` (item 15에서 영속화)의 unmapped 항목 — `grep` 1회.
- 모드 라벨 ↔ 본문 정합 (ADR-012 Diátaxis) — `head` + heuristic.

이들은 *LLM reasoning 비용이 들 이유 없는 영역*. 또한 [review-doc](.claude/skills/review-doc/SKILL.md)이 *단일 문서* 검토 skill이라 cross-doc 일관성은 별 surface 필요 — 그 자리가 본 단계 (review-doc을 `--all`/`--milestone`으로 확장하지 않고 stabilize에 흡수해야 review-doc 비대화 차단).

### 개선 방안

[.claude/skills/stabilize-milestone/SKILL.md:18-19](.claude/skills/stabilize-milestone/SKILL.md) step 1.5 *직전*에 새 step 1.0 신설:

```markdown
### 1.0. Deterministic pre-flight (LLM 위임 전 cheap mechanical check)

LLM 호출 전 다음을 순서대로 점검 (모두 deterministic, fail-fast X — 보고만):

1. **docs/ 내부 markdown link 유효성**: `markdown-link-check docs/**/*.md` 또는 동등 도구.
   - 깨진 link 발견 시 IMPROVEMENT_GUIDE.md에 `P1 [Doc-link] <file:line> — <broken link>` 라벨 기록.
   - 도구 미설치 환경은 본 항목만 skip + 출력에 명시.
2. **ADR 참조 유효성**: `[ADR-NNN]` 패턴 grep + 실제 파일 존재 여부 매칭.
   - 누락 발견 시 IMPROVEMENT_GUIDE에 `P1 [ADR-ref] <file:line> — ADR-NNN 본문 부재` 기록.
3. **FAC ↔ AC unmapped 검출** (item 15 영속화 정합):
   - 본 마일스톤의 모든 feature 문서 `## 7-1. FAC ↔ AC 매핑표`에서 *unmapped* 또는 *비어 있음* 항목 회수.
   - 발견 시 IMPROVEMENT_GUIDE에 `P0 [Spec-gap] F-NNN:FAC-N → unmapped` 기록 + 미커버 task 추가 권장.
4. **모드 라벨 ↔ 본문 정합 휴리스틱** (ADR-012): 모든 `docs/00-meta/` 파일의 `> 모드: ...` 라벨이 본문과 명백히 어긋나는지 점검 (휴리스틱 한계 명시).
   - mismatch 시 P2 `[Doc-mode] <file>` 기록.

본 단계는 모두 *보고만* — 발견이 있어도 stabilize 후속 단계 차단 X (LLM 위임 단계로 계속). 다음 라운드의 `/plan-workitem`이 후속 task로 회수.

**review-doc 책임 분담**: [review-doc](../review-doc/SKILL.md)은 *단일 문서 ad-hoc 검토*에 한정 (item 13 권한 모순 fix 후). cross-doc / link / FAC↔AC는 본 deterministic preflight가 담당 — review-doc을 `--all`/`--milestone` 모드로 확장하지 않는다.
```

`/stabilize-milestone` `allowed-tools` frontmatter는 이미 `Read Glob Grep Write Edit Bash Agent`라 `markdown-link-check` Bash 호출 가능 (도구 설치는 사용자 환경 결정).

### 근거

- 정책 SSOT: [ADR-014](docs/90-decisions/boilerplate/ADR-014-milestone-graduation.md) (graduation pre-check 정합 — 1.5 단계 *앞에* deterministic 추가) + [ADR-019](docs/90-decisions/boilerplate/ADR-019-context-packs-and-jit.md) (LLM 비용 최소화).
- Evidence label: `[가설]` — *mechanical-suitable 영역 분류*는 외부 출처가 있으나 *본 보일러플레이트 fork outcome 개선*은 fork 데이터 부재. ADR-022 정합으로 [가설].
- Ratchet: **enabling 정책(약)** — 보고만, 차단 X.

### 분류
- **우선순위: P1** (LLM 비용 절감 + cross-doc surface 확보)
- **Ratchet: 약**

---

## 22. (별건 후속) `/stabilize-milestone` evaluator-optimizer 패턴 명명

### 현재 상황

[ADR-014](docs/90-decisions/boilerplate/ADR-014-milestone-graduation.md)는 graduation checklist 5+1 + 회고 + pre-check + `--dry-run`을 정의. `/stabilize-milestone`의 *전체 패턴 명명*은 어디에도 없음.

[.claude/skills/stabilize-milestone/SKILL.md](.claude/skills/stabilize-milestone/SKILL.md) 본문은 *"E2E + 회귀 + 리팩토링 후보 + ADR 후보"* 식으로 *기능 나열*만.

### 문제

Anthropic "Building Effective AI Agents" 가이드가 *evaluator-optimizer pattern*을 공식 endorse: generator + skeptical evaluator + iterative loop. `/stabilize-milestone`은 정확히 이 패턴인데 *명명되지 않음* → fork 사용자가 *왜 이 분할인가*를 추적 어려움.

### 개선 방안

[ADR-014](docs/90-decisions/boilerplate/ADR-014-milestone-graduation.md)에 *Amendment 1* 추가:

```markdown
## Amendment 1 (2026-05-16) — Evaluator-Optimizer 패턴 명명

### 결정

`/stabilize-milestone`이 instantiate하는 패턴을 Anthropic "Building Effective AI Agents" 가이드의 **evaluator-optimizer pattern**으로 명명한다.

- **Generator** = [/implement-workitem](../../../.claude/skills/implement-workitem/SKILL.md) (이전 lifecycle 단계).
- **Evaluator** = qa + reviewer agent + deterministic preflight (본 skill이 위임/실행).
- **Optimizer** = [/repair-workitem](../../../.claude/skills/repair-workitem/SKILL.md) (다음 단계, 사용자 발화).

본 skill은 evaluator 단계의 *orchestration* — 코드 수정 X, 평가 + 보고만 (책임 경계 IMPROVE-LIST item 14 정합).

### 근거

- Anthropic 단일 source의 패턴 명명은 [ADR-022](ADR-022-ratchet-principle.md) "다중 repo 실증" 기준의 *외부실증* X — *명명 자체는 행동 변화 없는 citation*이라 evidence 부담 적음.
- ADR-007 lifecycle의 책임 분할(builder = 구현, validator = 판정 + report)은 이미 패턴 정합이지만 *milestone scope*의 명명이 빠짐.

### 적용 surface

- [.claude/skills/stabilize-milestone/SKILL.md](../../../.claude/skills/stabilize-milestone/SKILL.md) 본문 첫 단락에 *"본 skill은 evaluator-optimizer pattern의 evaluator orchestration이다 (ADR-014 amend 1)"* 1줄 추가.
- [DELEGATION_STRATEGY.md](../../00-meta/DELEGATION_STRATEGY.md) 스킬 실행 순서 가이드 단락에 동일 1줄.

### 후속 작업

없음 — citation 추가만.
```

[docs/90-decisions/boilerplate/README.md](docs/90-decisions/boilerplate/README.md)의 ADR-014 행에 *(+amend1: evaluator-optimizer pattern 명명)* 박음.

### 근거

- 정책 SSOT: [ADR-014](docs/90-decisions/boilerplate/ADR-014-milestone-graduation.md).
- Evidence label: `[가설]` — Anthropic 단일 source. ADR-022 정합. 단 *명명 자체는 행동 변화 없음 (legitimacy citation)*이라 evidence 부담 가벼움.
- Ratchet: **enabling 정책(약)** — 행동 변화 없음.

### 분류
- **우선순위: P2** (legitimacy citation, 별건 후속)
- **Ratchet: 약**

---

## 23. (Karpathy precision) `plan-workitem` AC interpretation diversity self-check

### 현재 상황

[.claude/skills/plan-workitem/SKILL.md:36](.claude/skills/plan-workitem/SKILL.md) "9. AC 형식 권장 + 금지 verb 점검" — *형식*만 점검 (GWT 형식, measurable verb, 금지 verb).

[item 5](#5-implement-workitemskillmd-red-phase-직전에-ambiguity-surfacing-protocol-추가) — implement-workitem 시점의 ambiguity surfacing.

### 문제

AC가 GWT 형식 + measurable verb 통과해도 *2+ 해석 가능한 부분이 silent하게 한 가지로 결정될 위험*은 *implement 시점에 잡으면 RGR 1회 이미 시작*. **plan 시점에 더 싸게 잡는 surface가 부재**.

예시: `[Given] user is logged in [When] clicks save [Then] data persists` — *measurable verb 통과*. 하지만:
- 해석 A: "persists in IndexedDB (offline)"
- 해석 B: "persists via API call (server)"
- 해석 C: "persists in localStorage"

planner가 이 차이를 *명시*하지 않으면 builder가 silent pick 후 RGR 사이클. **item 5는 builder 자리 보강이지 planner 자리 보강이 아니다** — 2-layer defense 필요.

### 개선 방안

[.claude/skills/plan-workitem/SKILL.md:36](.claude/skills/plan-workitem/SKILL.md) "9. AC 형식 권장 + 금지 verb 점검" 단락 *직후* 9-1 단락 신설:

```
9-1. **AC interpretation diversity self-check** (분해 직후 1회 실행):

각 AC를 *2+ 합리적 해석이 가능한지* self-check.
가능 시 plan 출력의 "남은 미결정 사항" 섹션에 다음 형식으로 박음:

- AC-N (T-NNN): 해석 A=<...>, 해석 B=<...>, 권장 선택=<...>
  (이유: charter ## 7. 제약 조건 또는 ## 5. 비목표 정합 / 비용 정합 등)

자동 차단 X — 사용자가 plan 검토 시 *해석 결정 협상*.

본 self-check가 plan 단계에서 발화하면 [item 5의 implement-workitem ambiguity surfacing](../implement-workitem/SKILL.md)은
*재확인 surface*가 됨 — 2-layer defense (plan에서 잡으면 RGR 1회 절감).
```

### 근거

- 정책 SSOT: [ADR-006](docs/90-decisions/boilerplate/ADR-006-simplicity-and-architecture.md) + [ADR-026](docs/90-decisions/boilerplate/ADR-026-plan-workitem-schema.md) "quality lint 모델 — *권장 + 재분해 텍스트*".
- Evidence label: `[가설]` — item 5와 동일 evidence (Karpathy testimony). ADR-022 정합으로 [가설]. *plan 시점이 더 싸다*는 보조 가정.
- Ratchet: **enabling 정책(약)** — 권장 텍스트만, 자동 차단 X.

### 분류
- **우선순위: P1** (예방 surface, plan-시점)
- **Ratchet: 약**

---

## 24. (Karpathy precision) `reviewer.md` Document Consistency 별도 섹션

### 현재 상황

[.claude/agents/reviewer.md:24-30](.claude/agents/reviewer.md) Clean Code 6항목 체크리스트 — *코드 시각*만.

[item 4](#4-reviewermd에-scope-discipline-별도-섹션-추가-clean-code-6은-그대로) — *Scope Discipline* 별도 섹션 (1~4 carry-over, 변경 diff 시각).

[review-doc](.claude/skills/review-doc/SKILL.md)이 *reviewer agent를 위임* (frontmatter `agent: reviewer`) — 즉 reviewer는 *코드 + 문서* 양쪽 surface에서 호출됨.

### 문제

review-doc이 reviewer를 호출할 때:
- Clean Code 6항목은 *코드 시각*이라 문서에 부적합 (Function size + SR이 ADR 본문에 적용?).
- Scope Discipline 4 카테고리 (item 4)는 *변경 diff 시각*이라 단일 문서 검토에 부적합.

→ reviewer가 문서 검토 시 *적용 가능한 라벨 셋이 없음* → 막연한 라벨링 또는 코드 시각 강제 적용. 같은 위반이 surface마다 다른 라벨로 박힐 위험 (SSOT drift).

또한 item 21 (stabilize-milestone deterministic preflight)의 결과를 reviewer가 후속 LLM 라벨링할 때도 본 차원이 필요.

### 개선 방안

[.claude/agents/reviewer.md](.claude/agents/reviewer.md) "Scope Discipline 체크" 단락 *바로 뒤*에 새 단락 신설 (Clean Code 6 + Scope Discipline 4 + Doc Consistency 4 = 3 차원 분리, 호출 surface에 따라 선택 적용):

```markdown
## Document Consistency 체크 (별도 차원 — 문서 review 시 호출)

review-doc (item 13 권한 모순 fix 후) 또는 stabilize-milestone (item 21 deterministic preflight)이
reviewer를 호출하면 다음 4 카테고리의 *문서 일관성 위반*을 발견 시 라벨링.

- (e) 모드 라벨(`> 모드: ...`)과 본문 정합 불일치 ([ADR-012](../../docs/90-decisions/boilerplate/ADR-012-doc-architecture-cleanup.md) Diátaxis) — `[Doc-mode]`
- (f) cross-reference link 유효성 (특히 `[ADR-NNN]` 참조와 실제 파일 매칭) — `[Doc-link]`
- (g) 인용된 ADR 본문과 *현재 ADR 본문* 정합 (citation drift — ADR이 amend된 후 인용자 미갱신) — `[Doc-adr-drift]`
- (h) Terminology consistency — 같은 개념이 다른 용어로 부르는 경우 (e.g., "Acceptance Criteria" vs "완료 조건") — `[Doc-term]`

reviewer 출력 라벨링 예: `P1 [Doc-link] AGENTS.md:38 — broken ADR link to ADR-XX`.

**호출 surface 명시**: 본 agent가 호출될 때 입력에 *"review surface: code | doc | mixed"*를 명시받는다. surface에 따라 적용 차원:
- `code`: Clean Code 6 + Scope Discipline 4.
- `doc`: Doc Consistency 4 + (해당 시) Scope Discipline 4 (변경 diff가 있을 때만).
- `mixed`: 3 차원 모두.
```

또한 [stabilize-milestone](../../.claude/skills/stabilize-milestone/SKILL.md) step 5 reviewer 위임 호출 입력에 *"review surface: code"* 명시 (default = code).
[review-doc](../../.claude/skills/review-doc/SKILL.md)이 reviewer 위임 시 *"review surface: doc"* 명시 (item 13 권한 fix와 함께 박음).

### 근거

- 정책 SSOT: [ADR-006](docs/90-decisions/boilerplate/ADR-006-simplicity-and-architecture.md) (Clean Code SSOT) + [ADR-007](docs/90-decisions/boilerplate/ADR-007-workitem-lifecycle.md) (책임 경계).
- Evidence label: `[관측됨+가설]` — *review-doc surface와 reviewer agent의 불일치*는 [관측됨] (직접 확인), *Doc Consistency 4 카테고리가 효과적*은 [가설]. 종합.
- Ratchet: **enabling 정책(약)** — 라벨링 추가, Pass 차단 X.

### 분류
- **우선순위: P1** (라벨링 SSOT — code/doc/scope 3 차원 분리)
- **Ratchet: 약**

---

## 우선순위 요약

| # | 항목 | P | Ratchet | Evidence |
|---|------|---|---------|----------|
| 1 | builder.md dead code 문구 방향 교정 | **P0** | 강 (drift fix) | [관측됨] |
| 11 | ADR-006 Amendment 1 (정책 SSOT) | **P0** | 혼합 (강+약) | 혼합 |
| 3 | validate-workitem diff trace audit (4 카테고리) | P0 (c) / P1 (a)(b)(d) | (c) 강 / 나머지 약 | (c) [관측됨+외부실증] / 나머지 [가설] |
| 4 | reviewer.md Scope Discipline 별도 섹션 | P1 | 약 | [가설] |
| 2 | builder.md diff trace self-check 추가 | P1 | 약 | [가설] |
| 5 | implement-workitem ambiguity surfacing | P1 | 약 | [가설] |
| 7 | AGENTS.md Surgical Changes 1줄 SSOT | P1 | 약 | [가설] |
| 6 | implement-workitem Step → verify plan | P2 | 약 | [가설] |
| 8 | LOC sanity heuristic (권장 텍스트) | P2 | 약 | [가설→트리거] |
| 9 | stabilize-milestone instruction improvement 보고 | P2 | 약 | [가설] |
| 10 | bootstrap-stack ARCHITECTURE 구체 기술 사실 | P2 | 약 | [가설] |
| 12 | validate-workitem FAC→AC drift fix (skill body ↔ agent body) | **P0** | 강 (drift fix) | [관측됨] |
| 13 | review-doc 권한 모순 fix (Write/Edit 허용 + 쓰기 범위 제한) | **P0** | 강 (drift fix) | [관측됨] |
| 14 | stabilize-milestone 회고 책임 경계 명시 (예외 박기) | **P0** | 강 (drift fix) | [관측됨] |
| 15 | plan-workitem FAC↔AC 영속화 (FEATURE_TEMPLATE `## 7-1`) | **P0** | 강 (drift fix) | [관측됨] |
| 16 | stack-guard 생성 후 validate smoke test (필수 단계) | **P0** | 강 | [관측됨] |
| 17 | stack-guard Codex wrapper 승격 (`$stack-guard`) | P1 | 약 | [관측됨+가설] |
| 18 | stack-guard `${CLAUDE_PROJECT_DIR}` + exec form (2 OS) | P1 | 강 | [관측됨+외부실증] |
| 19 | stack-guard TS-first depth verify 풀세트 (스택별 표) | P1 | 약 | [가설→실증] |
| 20 | PostToolUse hook async adapter 옵션 출력 (shared 자동 등록 X) | P1 | 약 | [가설] |
| 21 | stabilize-milestone deterministic pre-flight (1.0 단계 신설) | P1 | 약 | [가설] |
| 22 | stabilize-milestone evaluator-optimizer 패턴 명명 (ADR-014 amend) | P2 | 약 | [가설] |
| 23 | plan-workitem AC interpretation diversity (2-layer w/ item 5) | P1 | 약 | [가설] |
| 24 | reviewer.md Document Consistency 별도 섹션 (3 차원 분리) | P1 | 약 | [관측됨+가설] |

## 적용 시 가드레일 (적용하지 *말 것*)

- Karpathy CLAUDE.md를 그대로 복붙해 새 file로 두지 않는다 — [AGENTS.md](AGENTS.md)와 SSOT 충돌.
- *plugin marketplace install* / *`curl >> CLAUDE.md`* 적용하지 않는다 — [ADR-010](docs/90-decisions/boilerplate/ADR-010-multi-agent-compatibility.md) `AGENTS.md` 캐노니컬 진입 정책 위반.
- *"bias toward caution over speed" 메타-룰* 박지 않는다 — `--fast` 옵션이 이미 사용자 선택 trade-off surface ([ADR-009](docs/90-decisions/boilerplate/ADR-009-tdd-default.md)).
- *Product layer 7원칙* 박지 않는다 — [ADR-035](docs/90-decisions/boilerplate/ADR-035-continuous-discovery.md) DISCOVERY=SSOT + [ADR-036](docs/90-decisions/boilerplate/ADR-036-feature-level-prd.md) 12-섹션 PRD가 이미 cover.
- star 수를 *outcome 검증 evidence*로 사용하지 않는다 — *adoption signal*로만 인용. [ADR-022](docs/90-decisions/boilerplate/ADR-022-ratchet-principle.md) 정합.
- Clean Code 6항목을 7로 확장하지 않는다 — Scope Discipline은 별도 차원.
- 단일 case-study / 단일 testimony를 `[외부실증]`으로 라벨링하지 않는다 — [ADR-022](docs/90-decisions/boilerplate/ADR-022-ratchet-principle.md) "다중 repo 실증" 기준.
- PostToolUse hook 자동 등록을 **P0 강제 정책으로 박지 않는다** — `async`/`asyncRewake`는 *차단형 게이트가 아닌 보조 루프*. canonical 완료 판정은 동기 `validate-workitem` / `finalize-workitem` / `stabilize-milestone`이 담당 (item 20 정합).
- `.claude/settings.json` *shared*에 hook을 박지 않는다 — Codex parity 위반 ([ADR-010](docs/90-decisions/boilerplate/ADR-010-multi-agent-compatibility.md)). hook은 Claude-only adapter, canonical 검증은 `validate` 스크립트 (도구 중립).
- review-doc을 *`--all` / `--milestone`* 모드로 확장하지 않는다 — surface 비대화. cross-doc / link / FAC↔AC 검토는 stabilize-milestone deterministic preflight (item 21)에 흡수.
- stack-guard가 *smoke test 미통과* `validate` 명령을 산출하지 않는다 — item 16 필수. 사용자가 발견하는 시점이 validate-workitem 첫 실행 후면 이미 RGR 1+ 사이클 소비.
- 보일러플레이트 직접 지원 스택을 *TS web 단일*로 좁히지 않는다 — [ADR-031](docs/90-decisions/boilerplate/ADR-031-non-web-out-of-scope.md)의 5 스택 (web / API / CLI / monorepo / Supabase) 유지. *TS-first depth* (item 19)는 *기본 verify template*의 깊이 dialing이지 *범위 축소*가 아니다.
- 두 번째 evaluator agent로 *architect를 일상 호출*하지 않는다 — architect는 큰 tradeoff / 큰 설계만 ([DELEGATION_STRATEGY.md](docs/00-meta/DELEGATION_STRATEGY.md)). multi-verifier 패턴은 ADR-class 문서 검토 등 *고비용 영역에 한정*해 고려 (현재 본 리스트에는 도입 X).