# Boilerplate 개선 제안 리스트 (IMPROVE-LIST)

> 작성일: 2026-04-27 (초안), 2026-04-29 정리 (우선순위·이름·표현 조정, `/discover-product` 분리)
> 대상 브랜치: `main` (현재 HEAD `37d9dd0`)
> 근거: Claude Code 공식 문서 조사 결과 + 현재 저장소 정적 분석
> 비범위: 실제 구현(이 문서는 설계 제안만 담는다)

---

## 0. 요약

### 0.1 한 줄 흐름 비교

| 단계 | As-is | To-be |
|------|-------|-------|
| 발굴 | (없음) | `/discover-product` — persona·pain·JTBD·핵심 시나리오·MVP 범위·가정·열린 질문을 산출 |
| 초기화 | `/bootstrap-project` (architect-opus fork, 1회 입력) | `/bootstrap-project` — discovery 산출물을 읽어 charter / architecture / M1 / F-001 생성 또는 갱신 |
| 스택 세팅 | `/bootstrap-stack` (문서만 갱신) | `/bootstrap-stack` 후 사용자가 `/stack-guard` 이어 실행 — stack-specific `validate` 명령과 검증 스크립트 생성 |
| 분해 | `/plan-workitem` (지침문) | `/plan-workitem` (planner agent + fork) |
| 구현 | `/implement-workitem` | `/implement-workitem` — Red → Green → Refactor 3 phase 명시; AC ↔ 테스트 매핑 |
| 검증 | `/validate-workitem` (정적 판정) | `/validate-workitem` — 판정 + `docs/40-validation/reports/<task-id>.md`에 결과 기록 + AC↔테스트 매핑 점검 |
| 보수 | (없음) | `/repair-workitem` — report의 실패 항목만 수정 |
| 마감 | (없음) | `/finalize-workitem` — 최종 검증 + status `done` + 명시적 파일 add + 커밋 |
| 마일스톤 안정화 | (없음) | `/stabilize-milestone` — E2E + 회귀 + 리팩토링 후보 점검(코드 수정·커밋 없음) |
| 코딩 규율 | (암묵, 약함) | `CLAUDE.md` 단순성·YAGNI 정책 + `reviewer` Clean Code 6항목 체크리스트 + `ARCHITECTURE_OVERVIEW`의 레이어 경계·의존성 규칙 섹션 |
| 문서 중복·SSOT | (정의가 5곳까지 흩어짐, drift) | canonical owner 매핑 + 다른 곳은 한 줄 + 링크. ADR 인덱스(`docs/90-decisions/README.md`)·산출물 인벤토리(`docs/00-meta/STRUCTURE.md`) 신설. 정책=ADR 패턴. |

### 0.2 우선순위 / 의존성

| # | 항목 | 우선 | 난이도 | 의존 |
|---|------|------|--------|------|
| 1 | 모델 별칭 정책 | P0 | 낮 | — |
| 2 | 약한 skill 정리 + agent 바인딩 | P1 | 낮 | — |
| 3 | validation report 구조 + `/repair-workitem` 신설 | P0 | 낮 | — |
| 4 | `/finalize-workitem` 신설 + `/validate-workitem` 역할 분리 | P0 | 중 | 3 |
| 5 | `/stack-guard` 1단계(스크립트·통합 명령 생성) | P1 | 중 | — |
| 6 | `/stabilize-milestone` | P2 | 중 | 3, 4, 5 |
| 7 | `/discover-product` 신설 | P1 | 중 | — |
| 8 | `/bootstrap-project`가 discovery 산출물을 읽도록 조정 | P1 | 낮 | 7 |
| 9 | `/batch` + worktree 가이드 정리 | P2 | 낮 | — |
| 10 | 에이전트 TDD 기본 흐름 | P1 | 중 | 3, 4, 5 |
| 11 | 에이전트 단순성·Clean Code·Clean Architecture 가드레일 | P1 | 낮~중 | 6 (부분) |
| 12 | 단일 출처(SSOT) 구조 — 중복 제거 + cross-ref 표준화 | P0 | 중 | 1, 2 |

권장 적용 순서: **1 → 2 → 12 → 11a → 3 → 4 → 5 → 10 → 6 → 11b → 7 → 8 → 9**.

- `11a`/`11b` — 항목 11은 두 단계로 적용한다. **11a**: `CLAUDE.md` 단순성 단락 + agent 규칙 갱신(즉시 적용 가능, 의존 없음). **11b**: `ARCHITECTURE_OVERVIEW` 레이어 섹션 + `/stabilize-milestone` 통합(항목 6 적용 후).
- 항목 12는 항목 3~11이 만드는 신규 ADR·산출물·정책의 **전제 인프라**다. 항목 1·2 직후, 다른 신규 작성 항목 전에 적용한다.

- **1** — `claude-opus-4-6`은 [공식 model-config 문서](https://code.claude.com/docs/en/model-config) 기준으로 1버전 뒤(`claude-opus-4-7`이 최신). 즉시 실행 실패는 아니지만, 다음 deprecation 사이클이 오면 fork된 새 프로젝트에서 가장 먼저 stale이 된다. 별칭으로 바꾸는 비용은 거의 0이고 fork 안정성이 즉시 올라간다. 가장 먼저 정리한다.
- **2** — agent 바인딩이 빠진 skill(`plan-workitem`, `review-doc`)을 정리하고, 흐름상 약한 skill(`write-charter`, `write-architecture`)의 거취를 정한다. 가벼운 정리.
- **3 + 4** — validation report 양식과 finalize는 `task 문서 변경 예정 파일/경로` 섹션과 `docs/40-validation/reports/<task-id>.md` 양식을 공유한다. 동시 결정. validator-sonnet의 Write 권한 추가도 이 단계에서.
- **5** — `/stack-guard`의 verify 스크립트 + 통합 `validate` 명령 생성. PostToolUse hook 자동 등록은 prototyping 후 별도 항목으로 분리(본 항목에 즉시 포함하지 않는다).
- **6** — `/stabilize-milestone`. 통합 `validate` 명령(항목 5)과 report 양식(항목 3)이 정해진 뒤가 안전.
- **7 + 8** — discovery skill 신설과 `/bootstrap-project`의 입력 변경은 같은 PR로 묶어 적용. discovery만 단독으로 도입하면 charter 생성 경로가 두 갈래(discovery 결과 사용 vs 기존 단발 입력)로 갈라져 흐름이 흐려진다.
- **9** — `/batch` 표현 정리. 즉시 실패가 아니고 핵심 병목도 아니므로 마지막.
- **10** — 에이전트 TDD 기본 흐름. AC를 task 단위로 명문화하고 `/implement-workitem`을 Red→Green→Refactor 3 phase로 분해. validation report에 AC↔테스트 매핑이 기록되므로 항목 3의 양식, 항목 4의 finalize 가드, 항목 5의 통합 `validate` 명령이 모두 선행되는 편이 자연스럽다. 항목 11과 함께 fork된 미래 프로젝트의 에이전트 작업 규율을 결정한다.
- **11** — 에이전트 단순성·Clean Code·Clean Architecture 가드레일. 사용자가 강조한 핵심 가치(요구한 것만 딱, 오버엔지니어링 회피)를 fork 시 그대로 복사되는 surfaces(`CLAUDE.md`, `.claude/agents/*`, `ARCHITECTURE_OVERVIEW` 템플릿)에 박는다. 두 단계로 나눠 적용 — **11a** `CLAUDE.md` 단순성 단락과 agent 규칙은 의존 없이 즉시 적용 가능. **11b** Clean Architecture 의존성 규칙 섹션과 `/stabilize-milestone` 점검 통합은 항목 6 적용 후.
- **12** — 단일 출처(SSOT) 구조. 동일 사실이 5곳까지 흩어진 현 상태를 정리해 canonical owner 매핑·ADR 인덱스·산출물 인벤토리·"정책=ADR" 패턴을 도입한다. **항목 3~11이 만드는 신규 ADR·산출물·정책의 전제 인프라**이므로 항목 1·2 직후 적용한다. 적용 후 모든 신규 작성은 SSOT 패턴을 따른다.

### 0.3 항목 공통 형식

각 항목은 다음 7개 섹션으로 적는다.

1. 배경 / 동기
2. 현재 상태 (As-is)
3. 문제점
4. 개선안 (To-be)
5. 구현 스케치
6. 영향 범위
7. 열린 질문

### 0.4 횡단 원칙 (모든 항목에 공통 적용)

본 IMPROVE-LIST의 모든 항목 적용 시 다음을 공통으로 따른다.

1. **prototyping 선행 원칙** — 공식 문서로 동작이 확인되지 않은 메커니즘은 사용자 안내문이나 자동 산출물에 박기 전에 직접 띄워 확인한다. 본 문서 대부분의 메커니즘(skill frontmatter `arguments`/`paths`/`shell`/`hooks`, sub-agent `isolation: worktree`/자동 cleanup, `Bash(cmd *)`/`Bash(cmd:*)`/정확 일치 `Bash(cmd)` 패턴, AskUserQuestion 도구 등)은 [skills 문서](https://code.claude.com/docs/en/skills)·[sub-agents 문서](https://code.claude.com/docs/en/sub-agents)·[permissions 문서](https://code.claude.com/docs/en/permissions)·[hooks 문서](https://code.claude.com/docs/en/hooks)에 공식 명시되어 있으므로 prototyping 대상이 아니다. 남은 prototyping 대상:
   - **항목 5 — PostToolUse hook의 실측 비용** — `acceptEdits` 모드에서 매 Edit/Write마다 lint를 실행하면 비용이 폭증할 위험. 본 항목의 1단계에서는 hook 자동 등록을 처방하지 않고, prototyping 후 별도 항목으로 분리한다.
2. **정책성 결정은 ADR 동반** — 본 문서가 새로 도입하는 정책(Conventional Commits 기본, 모델 별칭 우선, 워크아이템 라이프사이클 단계 정의, validation report 파일 양식 등)은 적용 시 `docs/90-decisions/`에 ADR을 함께 추가한다. 6개월 뒤 fork한 사용자가 "왜 이 정책이 박혔는가"를 추적할 수 있어야 한다.
3. **skill chaining은 자동이 아니라 제안** — Claude Code skill은 다른 skill을 함수처럼 직접 호출하지 않는다. 본 문서의 흐름 그림(`/validate-workitem → /finalize-workitem` 같은 화살표)은 **앞 skill이 텍스트로 다음 액션을 제안하면 사용자 또는 메인 세션이 받아 다음 skill을 발화**하는 흐름이다. "Pass → 자동으로 finalize 실행"이 아니라 "Pass → finalize 추천 출력 → 사용자/메인이 발화"임을 통합 흐름 그림(섹션 13) 직전에 한 번 명시한다.
4. **단순성·YAGNI 우선 원칙** (항목 11과 직접 연결) — 본 문서가 도입하는 모든 정책은 fork된 미래 프로젝트의 에이전트가 "요구한 범위만 구현"하도록 보강하는 것이 목표다. 어떤 항목이 추측성 추상화(미래의 어떤 기능을 위한 인터페이스, 1~2회밖에 쓰지 않을 helper)를 만든다면 그 항목 자체가 본 원칙을 위반한 것이다. 적용 시 이 충돌이 보이면 항목을 단순화하거나 적용을 미룬다.
5. **SSOT 원칙** (항목 12와 직접 연결) — 본 문서의 모든 항목 적용 시 **각 사실은 단 하나의 canonical 문서에 정의되고 다른 곳은 한 줄 + 링크**여야 한다. 신규 정책은 ADR로 박고 agent/skill 본문에는 행동 규율과 ADR 링크만 둔다. 신규 산출물은 `docs/00-meta/STRUCTURE.md` 인벤토리에 등록한다. 신규 ADR은 `docs/90-decisions/README.md` 인덱스에 행을 추가한다. 본 IMPROVE-LIST의 항목별 본문(섹션 1~12)이 SSOT이고, 섹션 14 체크리스트는 인덱스 역할이다 — 본문과 충돌하면 본문이 권위.

### 0.5 항목 번호 변경 매핑 (이전 버전 대비)

본 문서는 2026-04-27 초안의 항목 번호를 새 우선순위에 맞춰 재배열했다. 기존 PR 코멘트/링크 추적용 매핑:

| 신 # | 구 # | 항목 |
|------|------|------|
| 1 | 12 | 모델 별칭 정책 |
| 2 | 4 | 약한 skill 정리 + agent 바인딩 |
| 3 | 3 | validation report + `/repair-workitem` |
| 4 | 2 | `/finalize-workitem` |
| 5 | 1 | `/stack-guard` 1단계 |
| 6 | 5 | `/stabilize-milestone` (구 `/wrap-milestone`) |
| 7 | 신규 | `/discover-product` |
| 8 | 6의 일부 | `/bootstrap-project` discovery 입력 |
| 9 | 11 | `/batch` + worktree 가이드 |
| 10 | 신규 | 에이전트 TDD 기본 흐름 |
| 11 | 신규 | 에이전트 단순성·Clean Code·Clean Architecture 가드레일 |
| 12 | 신규 | 단일 출처(SSOT) 구조 — 중복 제거 + cross-ref 표준화 |

원 항목 7(통합 흐름)·8(체크리스트)·9(공식 문서 조사)·10(재검토 메모)은 본문 보조 섹션으로 위치만 이동했다(섹션 13·14·15·16). 본 재배열에서 새 항목 10·11·12가 본문 항목으로 들어오면서 보조 섹션 번호가 3 증가했다.

원 항목 11(settings.local 모순)·12(slash command 불일치)·13(자기검증 루틴)은 재검토 사이클에서 삭제·흡수되어 사라졌다(상세는 섹션 16.2 참조).

---

## 1. 모델 별칭 정책

### 1.1 배경 / 동기
이 보일러플레이트의 핵심 가치는 "여러 프로젝트에서 반복 재사용"이다. 모델 ID가 시간이 지나며 deprecated되면 수개월 뒤 새 프로젝트에 그대로 복제했을 때 stale로 보이는 면이 생긴다. 표기 일관성이 없으면 어디를 갱신해야 하는지 사람의 기억에 의존하게 된다.

### 1.2 As-is
- `.claude/settings.json`의 `model` → `claude-sonnet-4-6` (전체 ID, 특정 버전 고정)
- `.claude/agents/architect-opus.md`의 `model` → `claude-opus-4-6` (전체 ID, 특정 버전 고정)
- `.claude/skills/bootstrap-project/SKILL.md`, `.claude/skills/bootstrap-stack/SKILL.md`의 `model` → `claude-opus-4-6`
- `.claude/agents/builder-sonnet.md`, `validator-sonnet.md`, `qa.md`, `planner.md`, `reviewer.md`의 `model` → `sonnet` (별칭, 자동 최신 매핑)
- 표기 정책을 안내하는 문서는 없다.

### 1.3 문제점
- 같은 종류의 정보(어떤 모델을 쓸지)가 두 가지 표기로 섞여 있어, 새 프로젝트를 시작한 사람이 어느 쪽 규칙을 따라야 할지 즉시 알기 어렵다.
- 전체 ID로 고정된 자리들이 보일러플레이트의 "재사용 가능" 약속을 가장 먼저 깎아내리는 staleness 지점이다.
- 공식 [model-config 문서](https://code.claude.com/docs/en/model-config) 기준 현재 별칭 매핑은 `opus → claude-opus-4-7`, `sonnet → claude-sonnet-4-6`. 즉:
    - `architect-opus.md`/`bootstrap-project`/`bootstrap-stack`의 `claude-opus-4-6`은 **현 시점 1세대 뒤**다 — 즉시 실행 실패는 아니지만, 다음 deprecation 사이클에서 가장 먼저 noise가 된다.
    - `settings.json`의 `claude-sonnet-4-6`은 **현 시점 최신과 일치**하지만 다음 deprecation 사이클에서 같은 stale 위험을 갖는다. staleness보다 *표기 일관성*과 *fork 후 자동 최신 매핑* 가치를 위해 별칭으로 통일한다.

### 1.4 To-be
- 보일러플레이트가 채택할 정책을 한 곳에 명시한다.
    - **shared 기본값에서는 모델 별칭(`sonnet`, `opus`, `haiku`)만 사용한다. 특정 버전을 강제해야 하는 이유가 있으면 ADR로 남기고 그 자리에서만 전체 ID를 사용한다.**
    - 근거: ① 자동 최신 매핑이 보일러플레이트의 재사용 약속과 가장 잘 맞는다. ② 사람이 모델 갱신을 잊어 staleness가 누적되는 것을 저비용으로 막는다.
- 위 정책에 따라 staleness 지점을 정리한다.
    - `.claude/settings.json` → `"model": "sonnet"`
    - `.claude/agents/architect-opus.md` → `model: opus`
    - `.claude/skills/bootstrap-project/SKILL.md`, `.claude/skills/bootstrap-stack/SKILL.md`의 model 라인 처리는 **항목 8 적용 시점에 따라 다르다**:
        - **항목 8 적용 전**: 두 skill 모두 `model: claude-opus-4-6` → `model: opus`로 변경.
        - **항목 8 적용과 동시**: `bootstrap-project`의 frontmatter 자체가 단순화되며 `model` 라인이 사라질 가능성이 있다(상세는 항목 8). 이 경우 별칭 변경은 생략 가능.
- 정책을 깨고 특정 버전을 고정해야 할 때를 위한 절차를 짧게 적어 둔다 — "ADR로 이유 기록 → 갱신 책임자 명시" 정도면 충분.

### 1.5 구현 스케치
- 수정 `.claude/settings.json` — `"model": "claude-sonnet-4-6"` → `"model": "sonnet"`
- 수정 `.claude/agents/architect-opus.md` — `model: claude-opus-4-6` → `model: opus`
- (조건부) 수정 `.claude/skills/bootstrap-project/SKILL.md`, `.claude/skills/bootstrap-stack/SKILL.md` — 항목 8 적용 전이면 `model: opus`로. 동시 적용이면 항목 8이 처리.
- 수정 `docs/00-meta/AGENT_EXECUTION_STRATEGY.md` — 모델 표기 정책 한 단락 추가(별칭 기본, 특정 버전 고정 시 ADR).
- (권장) 신규 `docs/90-decisions/ADR-00x-model-alias-policy.md` — 정책성 결정이므로 ADR로 보존.

### 1.6 영향 범위
- 수정 1 설정, 1 agent, 1 문서, (조건부) 2 skill
- 신규 ADR 1개(권장)

### 1.7 열린 질문
- architect-opus처럼 "이 작업은 가장 강한 모델로 돌려야 한다"는 의도를 별칭만으로 충분히 표현할 수 있는가? — 잠정: 충분. 별칭 `opus`는 정의상 가장 강한 Opus 계열을 가리킨다.
- 사용자가 fork 직후 자기 환경의 비용 정책을 강제해야 한다면 어디서 override가 자연스러운가? — 잠정: `.claude/settings.local.json`에서 model override.

---

## 2. 약한 skill 정리 + agent 바인딩

### 2.1 배경 / 동기
이 저장소의 정체성은 "메인 세션은 오케스트레이션, 실작업은 서브에이전트 위임"이다. 그렇다면 자주 쓰는 slash command는 실제로 agent workflow를 타야 한다. 그러나 일부 skill은 sub-agent를 호출하지 않고 단순히 "메인 세션에 지침을 주는 본문"만 갖고 있어, 실제 위임 효과 없이 컨텍스트만 차지한다.

### 2.2 As-is
| Skill | agent 바인딩 | 본문 성격 | 중복/약점 |
|-------|-------------|-----------|----------|
| `boilerplate-context` | 없음 (auto-context) | 저장소 안내 | 그대로 유지 |
| `bootstrap-project` | architect-opus, fork | 강함 | (항목 8에서 조정) |
| `bootstrap-stack` | architect-opus, fork | 강함 | (항목 5와 결합) |
| `plan-workitem` | **없음** | 짧은 지침문 | planner agent와 역할 중복, 위임 효과 0 |
| `implement-workitem` | builder-sonnet, fork | 강함 | OK |
| `validate-workitem` | validator-sonnet, fork | 강함 | (항목 3·4에서 강화) |
| `write-charter` | **없음** | 짧은 지침문 | `bootstrap-project`가 charter를 만든다. 단독 의미 약함 |
| `write-architecture` | **없음** | 짧은 지침문 | `bootstrap-project`/`bootstrap-stack`이 갱신. 단독 의미 약함 |
| `review-doc` | **없음** | 짧은 지침문 | reviewer agent와 역할 중복 |

### 2.3 문제점
- agent 바인딩이 없는 skill은 사실상 "지침 한 페이지"라서 사용자가 슬래시로 부르는 의미가 약하다.
- 동일한 일을 두 진입점(skill vs agent)에서 부를 수 있어 흐름이 흐려진다.

### 2.4 To-be
- **유지·강화** (agent 바인딩 추가) — "공식 slash command는 실제 동작을 가져야 한다"는 원칙으로 정리:
    - `plan-workitem`: `agent: planner`, `context: fork`, `argument-hint: "[milestone or feature id]"`. 본문은 분해 절차 가이드를 그대로 두되 planner에 위임된다.
    - `review-doc`: `agent: reviewer`, `context: fork`, `argument-hint: "[doc path]"`.
- **삭제 또는 refine 계열로 축소**:
    - `write-charter` — `/bootstrap-project` 이후 charter 단독 갱신은 자연어 + planner subagent로 충분. 보일러플레이트의 "필수 진입점" 가치보다 혼선 비용이 큼. 두 가지 처분안:
        - (A) 삭제 + 대체 경로 안내(아래 mid-project 동선) — 본 문서의 권장.
        - (B) `refine-charter`로 이름·역할을 좁혀 "기존 charter의 부분 갱신만" 담당하도록 축소 — 사용자가 charter 단독 갱신 진입점을 원할 때.
    - `write-architecture` — 동일 처분(A 권장, B는 `refine-architecture`로 축소).

**mid-project 갱신 동선** — charter/architecture는 `WORKFLOW.md`에서 Living Doc로 분류돼 진행 중 재진입이 필요하다. 두 skill을 삭제했을 때의 경로:

| 갱신 종류 | 경로 |
|----------|------|
| charter 부분 갱신 | 자연어로 메인 세션에 변경 요청 → `planner` agent에 fork 위임 |
| charter 전면 재정의 | `/discover-product` 재실행(또는 산출물만 갱신) → `/bootstrap-project`로 charter 재생성 |
| architecture 스택 변경 | `/bootstrap-stack` 재실행 후 `/stack-guard` 이어 실행 |
| architecture 시스템 경계만 갱신 | 자연어 + `architect-opus` 단발 호출 |

이 동선은 `WORKFLOW.md`의 mid-project 단락에 명시되어야 한다 — 사용자가 두 skill 삭제로 갱신 경로를 잃었다고 오해하지 않도록.

### 2.5 구현 스케치
- 수정 `.claude/skills/plan-workitem/SKILL.md`
    - frontmatter에 `agent: planner`, `context: fork`, `disable-model-invocation: true`, `argument-hint: "[milestone or feature id]"`, `allowed-tools: Read Glob Grep`
    - 본문은 명령형 절차로 작성 — 입력 ID에 해당하는 charter/architecture/상위 workitem을 읽고 milestone/feature/task로 분해. `context: fork` 사용 시 본문은 actionable해야 한다는 공식 경고(skills 문서) 준수.
- 수정 `.claude/skills/review-doc/SKILL.md`
    - frontmatter에 `agent: reviewer`, `context: fork`, `disable-model-invocation: true`, `argument-hint: "[doc path]"`, `allowed-tools: Read Glob Grep`
    - 본문은 "입력 경로의 문서를 읽고 모순/누락/모호함을 찾아 우선순위로 보고" 같은 명령형 절차.
- 삭제 `.claude/skills/write-charter/`, `.claude/skills/write-architecture/` (대체 경로는 위 mid-project 동선)
- 수정 `README.md`, `README_ko.md`, `docs/00-meta/WORKFLOW.md`, `docs/00-meta/AGENT_EXECUTION_STRATEGY.md` — 표에서 두 skill 제거, plan/review의 agent 바인딩 명시, mid-project 동선 단락 추가

### 2.6 영향 범위
- 삭제 2 skill, 수정 2 skill, 문서 4종 갱신.

### 2.7 열린 질문
- `plan-workitem`을 fork 시키면 메인 세션이 분해 결과만 받는다 — 회의 중 분해 과정을 보고 싶으면 fork를 끄는 옵션이 필요한가? (잠정: 기본 fork, 본문에 "메인이 직접 분해하길 원하면 fork 없이 planner agent를 호출"이라는 안내 한 줄)

---

## 3. validation report 구조 + `/repair-workitem` 신설

### 3.1 배경 / 동기
검증 결과는 다음 단계(repair 또는 finalize)의 입력이다. 그러나 현재는 결과가 메인 세션 메시지로만 흘러 fork된 sub-agent가 회수할 수 없다. 결과 회수 메커니즘을 파일로 표준화하면 사용자도, 다음 skill도 동일한 위치를 보면 된다.

### 3.2 As-is
- `/validate-workitem` → validator-sonnet. 결과는 메인 세션에 메시지로 남는다.
- 사용자가 검증 실패 후 "어디를 어떻게 고칠지"를 builder-sonnet에 다시 풀어 설명하거나, `/implement-workitem`을 다시 부른다 — 후자는 task 문서 기반의 처음부터 구현이라 의미가 다르다.
- 결과 회수의 표준 위치가 없으므로 sub-agent fork 환경에서는 직전 검증을 회수할 방법이 없다.

### 3.3 문제점
- repair는 task 문서 + **직전 검증 결과**라는 두 입력을 가진다. 이를 표현하는 진입점이 없다.
- 검증 결과의 보존 위치가 명시되어 있지 않으므로, 사용자가 결과를 다시 보려면 메인 세션 스크롤백을 의존한다.
- 사용자가 "validator의 5개 지적 중 4개만 고쳐"처럼 부분 지정하는 자연스러운 방법이 없다.

### 3.4 To-be

**검증 결과 파일 양식 통일** — `/validate-workitem`은 판정 결과를 `docs/40-validation/reports/<task-id>.md`에 기록한다(파일 경로는 task-id 단위로 덮어쓴다 — 한 task의 가장 최근 검증 1회만 남긴다). 양식은 다음 섹션을 포함한다:

- 검증 시각, task-id, 판정(`Pass` / `Needs Fix`)
- 실패 항목 목록(P0/P1/P2 우선순위, 짧은 설명, 관련 파일·라인)
- 통합 명령(`pnpm validate` 등) 실행 결과 요약(있으면)
- 다음 권장 액션 — Pass면 `/finalize-workitem` 안내, Needs Fix면 `/repair-workitem` 안내(skill chaining 자동 아님, 텍스트 제안임을 명시)

**`/repair-workitem [task-id] [optional notes]` 신설** — 동작:

1. task 문서 읽기
2. `docs/40-validation/reports/<task-id>.md` 읽기
   - 파일이 없거나 stale(파일 mtime이 task 변경보다 오래됨)하면 사용자에게 `/validate-workitem` 선행을 안내하고 종료
   - 파일이 `Pass`이면 `/finalize-workitem`을 안내하고 종료(repair 대상 없음)
   - 사용자가 인자로 부분 지정(`/repair-workitem T-001 "P0 #1, P1 #3만"`)을 줬으면 그 부분만 대상
3. 실패 항목을 우선순위(P0 > P1 > P2)로 정렬
4. builder-sonnet에 fork 위임 — 입력은 task 문서 + report 파일 경로 + 사용자 추가 메모
5. **repair 책임 경계** (이 skill 본문에 명시):
   - 새 기능을 추가하지 않는다.
   - task 범위 밖 파일을 수정하지 않는다.
   - 자동 커밋하지 않는다 — 결과만 반환하고, 커밋은 `/finalize-workitem` 또는 사용자가 별도로.
   - 가능하면 한 라운드에 P0/P1만 처리하고 P2 이하는 다음 라운드 추천.
6. 출력은 수정 파일 목록, 어떤 실패 항목을 어떻게 해소했는지, 미해결 항목, 다음 권장 액션(보통 `/validate-workitem` 재실행).

### 3.5 구현 스케치
- 수정 `.claude/skills/validate-workitem/SKILL.md`
    - 첫 줄에 "이 skill은 **판정 + report 기록 전용**이다. status 변경, 코드 수정, 커밋은 하지 않는다." 명시.
    - 마지막 단계로 "판정 결과를 표준 양식으로 `docs/40-validation/reports/<task-id>.md`에 기록"을 추가.
    - allowed-tools에 `Write` 추가(현재 `Read Glob Grep Bash`).
    - 마지막 출력에서 다음 액션을 제안 — Pass면 `/finalize-workitem`, Needs Fix면 `/repair-workitem`. 자동 호출이 아니라 텍스트 제안임을 한 줄 명시.
- 수정 `.claude/agents/validator-sonnet.md`
    - tools에 `Write` 추가(`Read, Glob, Grep, Bash` → `Read, Glob, Grep, Bash, Write`).
    - 규칙에 "구현이나 status 갱신, 커밋을 직접 수행하지 않는다", "판정 결과를 `docs/40-validation/reports/<task-id>.md`에 표준 양식으로 기록한다"를 추가.
- 신규 `.claude/skills/repair-workitem/SKILL.md`
    - frontmatter: `disable-model-invocation: true`, `agent: builder-sonnet`, `context: fork`, `argument-hint: "[task id] [optional notes]"`, `allowed-tools: Read Glob Grep Write Edit Bash`
    - 본문: 위 동작 4단계 + 책임 경계(새 기능 금지, 범위 밖 금지, 자동 커밋 금지). report 부재/Pass/Stale 시 종료 흐름 명시.
- 신규 `docs/40-validation/reports/.gitkeep` (디렉터리 생성용)
- 수정 `.gitignore` — `docs/40-validation/reports/*.md` 추가(검증 산출물은 ephemeral). `.gitkeep`는 예외로 트래킹.

### 3.6 영향 범위
- 신규 1 skill, 1 디렉터리
- 수정 `.claude/skills/validate-workitem/SKILL.md`, `.claude/agents/validator-sonnet.md`, `.gitignore`
- 문서: `WORKFLOW.md`, `AGENT_EXECUTION_STRATEGY.md`, `README.md`, `README_ko.md`

### 3.7 열린 질문
- 멀티 task 묶음 검증에서는 어떻게? (잠정: `/validate-workitem`은 1 task = 1 파일을 유지, 묶음 검증은 별도 흐름으로)
- repair → validate → repair 무한 루프 방지 가드 필요(잠정: 한 task당 연속 3회 이상 repair 시 사용자 확인을 요구하는 안내 문구)
- report 양식을 markdown으로 둘지, JSON으로 둘지 — 잠정: markdown(사람이 직접 읽기 쉬움, fork된 sub-agent도 Read로 처리 가능). JSON이 필요하면 후속 항목으로.

---

## 4. `/finalize-workitem` 신설 + `/validate-workitem` 역할 분리

### 4.1 배경 / 동기
지금은 "검증 → 마감 → 커밋"이 한 명령에 묶이지 않아 사람이 매번 수동으로 status를 `done`으로 바꾸고 커밋한다. 동시에 `/validate-workitem`이 종종 검증 외 작업까지 떠안는 경향이 생긴다.

### 4.2 As-is
- `/validate-workitem` → validator-sonnet. 출력은 `Pass / Needs Fix` + 핵심 문제 5개.
- task 문서 status 갱신, 커밋은 사람의 몫.
- 결과적으로 워크아이템 라이프사이클의 "마감 시점"이 모호하다.

### 4.3 문제점
- 검증 통과인데 status가 `in-progress`로 남아 다음 작업과 혼선.
- 커밋 메시지가 매번 일관되지 않음.
- "Needs Fix"인데 사람이 잊고 finalize-like 작업을 해버릴 가능성.

### 4.4 To-be
새 skill `/finalize-workitem [task-id]`. 동작:

1. 관련 task 문서 읽기
2. 통합 검증 명령(`pnpm validate` 등) 실행 — 사용자가 직전 `/validate-workitem`을 통과했더라도 한 번 더 실행한다(상태가 변했을 수 있음). 통합 명령이 없으면(스택 미정 등) 이 단계는 건너뛴다.
3. 실패 → `Needs Fix`로 종료. 커밋하지 않음. 다음 액션으로 `/repair-workitem [task-id]`를 텍스트로 제안.
4. 통과 → task 문서의 `## 0. Status`를 `done`으로 갱신
5. `git status` / `git diff --stat` 기반으로 커밋 메시지 초안 생성 (Conventional Commits 스타일 기본)
6. **변경 파일을 명시적 목록으로 add 한다** — `git add -A` / `git add .`는 사용하지 않는다(.env, secrets, 빌드 산출물, untracked 임시 파일이 우발적으로 포함될 위험).
   파일 목록 산출 우선순위(엄격한 가드가 아니라 우선 참조 순):
    - **(1) task 문서의 "변경 예정 파일/경로" 섹션** — `TASK_TEMPLATE.md`에 추가되는 섹션(아래 4.5 참조). 있으면 우선 참조.
    - **(2) `git status --porcelain` / `git diff --name-only`로 실제 변경 파일 회수**.
    - **(3) 제외 규칙** — task 범위와 명백히 무관한 파일, 민감 경로(`./.env*`, `./secrets/**`), 빌드 산출물(`node_modules/`, `dist/`, `build/`, `.next/`, `coverage/`)은 add 대상에서 제외. 민감 경로는 `.gitignore`로 이미 제외돼야 하지만, finalize 본문에도 "민감 경로가 staged 영역에 들어오면 즉시 종료" 가드를 둔다.

   (1)이 없거나 (2)와 어긋나면 사용자에게 차이를 보여주고 확인을 받는다(엄격하게 종료하지 않는다 — 첫 회 사용자가 task 문서에 변경 예정 파일을 안 적었을 때 흐름이 막히지 않도록).
7. `git commit -m "..."` 실행 — `--no-verify` 금지, `--amend` 금지, `git push` 금지(사용자 명시 요청 필요)
8. 마지막에 커밋 해시·메시지·갱신된 status 요약 반환

`/validate-workitem`은 **판정 + report 기록 전용**으로 좁힌다 — 자동 수정·자동 마감 금지. (이 줄은 항목 3과 같은 변경에 속한다.)

### 4.5 구현 스케치
- 신규 `.claude/skills/finalize-workitem/SKILL.md`
    - frontmatter:
        - `disable-model-invocation: true`
        - `agent: builder-sonnet`
        - `context: fork`
        - `argument-hint: "[task or feature identifier]"`
        - `allowed-tools: Read Glob Grep Write Edit Bash(git add *) Bash(git status *) Bash(git diff *) Bash(git commit *) Bash(pnpm validate) Bash(npm run validate) Bash(make validate)`
    - **note**: `Bash(cmd *)`/`Bash(cmd:*)`(trailing wildcard 동등 표기) 패턴 및 정확 일치(`Bash(pnpm validate)`, `Bash(npm run build)`)는 공식 [permissions 문서](https://code.claude.com/docs/en/permissions)에 모두 명시. 정확 일치는 별도 prototyping 없이 그대로 사용 가능.
    - 본문: 위 8단계. 실패 시 어떤 출력을 어떻게 요약할지. add 파일 목록 산출 우선순위(task 문서 참조 → git 실제 변경 → 제외 규칙) 명시. (1)이 비어 있어도 강제 종료하지 않고 사용자 확인을 받는다는 운영 원칙도 본문에.
    - 가드: 작업 트리에 변경이 없으면 "변경 없음" 종료. `git add -A` / `git add .` 금지. 민감 경로 staged 시 즉시 종료. `--amend` 금지. `--no-verify` 금지. `git push` 호출 금지.
- 수정 `docs/30-workitems/_templates/TASK_TEMPLATE.md`
    - 새 섹션 추가: `## 4-1. 변경 예정 파일/경로` — 구현 시점에 작성. finalize의 add 참조 목록으로 사용된다(엄격한 화이트리스트가 아니라 참조).
- 수정 `.claude/skills/validate-workitem/SKILL.md`
    - 첫 줄 "이 skill은 **판정 + report 기록 전용**이다" — 항목 3과 동일.
- 수정 `.claude/agents/validator-sonnet.md`
    - 항목 3과 동일.
- 수정 `.claude/agents/builder-sonnet.md`
    - finalize 위임을 받았을 때의 가드(작업 트리·민감 파일·`--no-verify` 금지) 추가
    - 구현 완료 후 task 문서의 `## 4-1. 변경 예정 파일/경로` 섹션을 갱신하라는 규칙 추가.
- (권장) 신규 `docs/90-decisions/ADR-00x-commit-convention.md` — Conventional Commits 기본 채택.

### 4.6 영향 범위
- 신규 `.claude/skills/finalize-workitem/`, ADR-00x-commit-convention.md(권장)
- 수정 `.claude/agents/builder-sonnet.md`, `docs/30-workitems/_templates/TASK_TEMPLATE.md`
- 문서: `WORKFLOW.md`, `AGENT_EXECUTION_STRATEGY.md`, `README.md`, `README_ko.md`

### 4.7 열린 질문
- 커밋 메시지 컨벤션을 Conventional Commits 고정으로 강제할지, charter에서 선택하게 할지(잠정: 기본 Conventional Commits, charter에서 override 가능).
- finalize 안에서 통합 명령을 한 번 더 돌리는 비용 vs `/validate-workitem` 결과를 신뢰하는 안전성 — 보수적으로 한 번 더 돌리는 쪽 추천.
- 멀티 task 묶음 커밋은 어떻게 다룰지(잠정: `/finalize-workitem T-001 T-002` 형태로 다중 ID 허용, 메시지에 모두 명시).

---

## 5. `/stack-guard` 1단계 — 검증 스크립트 + 통합 명령 생성

### 5.1 배경 / 동기
`docs/00-meta/GUARDRAILS_STRATEGY.md`는 "shared 기본값에는 OS, 셸, 런타임에 강하게 의존하는 자동화를 넣지 않는다"를 명시한다. 그러나 스택이 정해진 **다음 단계**, 즉 어떤 자동화를 어떻게 만들지에 대한 실제 산출물은 비어 있다. 그 결과 검증이 사람의 기억에 의존한다.

### 5.2 As-is
- `.claude/skills/bootstrap-stack/SKILL.md`는 산출물로 `ARCHITECTURE_OVERVIEW.md`, `ADR-003-stack-selection.md`, 선택적으로 `STACK_SETUP_PLAN.md`만 약속한다.
- `scripts/verify.*`, `.claude/settings.json` hook 등록은 사람이 수동으로 한다.
- `/validate-workitem` → validator-sonnet은 통합 검증 명령이 없으므로 실제로는 정적 판단만 하고 있다.

### 5.3 문제점
- 스택이 정해진 직후 자동화 공백이 길고, 사람이 까먹으면 검증이 누락된다.
- "동작 검증" 책임이 어디에도 명시 위임되어 있지 않다.
- finalize / repair 자동화의 전제(통합 명령)가 없다.

### 5.4 To-be

**1단계 (즉시 적용, 본 항목의 범위)** — `/stack-guard`가 만드는 산출물:

- 통합 진입점 — 이름은 **`validate`로 고정**한다(`pnpm validate` / `npm run validate` / `make validate` / `task validate` 중 스택에 자연스러운 단일 명령). 본 문서 흐름의 `/validate-workitem` 명칭과 통일해 사용자가 한 단어만 기억하면 되도록 한다.
- `scripts/verify.{sh,ps1,mjs,py}` (스택에 가장 자연스러운 런타임 1종, 추가 셸은 사람이 원할 때만)
- cross-platform 차이가 큰 팀이면 `.claude/settings.local.json` 예시 동봉
- `docs/00-meta/STACK_SETUP_PLAN.md`에 PostToolUse hook 등록 안내 문구(설정 예시 + 매뉴얼 등록 절차)만 둔다 — 본 1단계에서는 hook을 사용자 환경에 자동 생성하지 않는다.

`/stack-guard`는 R0에서 운영 환경 가정을 명시적으로 받는다 — 단일 OS/셸인지, mixed env인지. 이 응답은 추후 hook 자동 등록 항목(2단계)이 분리됐을 때의 입력이 된다. `/bootstrap-stack` 완료 후 사용자가 이어서 `/stack-guard`를 발화하는 흐름(skill 자동 호출 아님)으로 운영한다.

`/validate-workitem` 흐름을 다음으로 바꾼다:
1. (통합 `validate` 명령이 있으면) 자동 실행 — 결과를 stdout/stderr로 수집
2. 결과를 validator-sonnet에 컨텍스트로 전달
3. validator-sonnet은 정적 + 동적 결과를 함께 보고 `Pass / Needs Fix` 판정 + report 기록(항목 3)

통합 명령이 없으면 1번을 건너뛰고 정적 판정만 한다 — 기존과 호환.

**2단계 (prototyping 후, 별도 항목으로 분리)** — PostToolUse hook 자동 등록은 본 항목에서 처방하지 않는다. 이유는 보수적으로 정리하면:

- hook 입출력 JSON 스키마와 settings.json patch 양식이 본 문서 시점에 직접 검증되지 않았다.
- `settings.json:4`의 `defaultMode: "acceptEdits"`와 `PostToolUse=lint`가 결합되면 매 Edit/Write마다 lint가 자동 실행되어 비용이 폭증할 수 있다(사용자가 수락 프롬프트로 차단할 기회조차 없음).
- 검증되지 않은 메커니즘으로 사용자 환경을 자동 변형하는 것은 0.4 횡단 원칙 1과 동일한 유형의 리스크다.

prototyping 단계에서 다음을 직접 확인한 뒤 별도 항목(예: `5b. /stack-guard hook 자동 등록`)으로 분리해 본 IMPROVE-LIST에 추가한다:
- hook 입출력 스키마와 settings.json patch 양식
- `acceptEdits` 모드에서의 실측 비용(매 Edit당 lint 시간 × 빈도)
- 단일 OS/셸 가정의 자동 감지 신뢰도(R0 응답이 충분한가)

prototyping이 끝나기 전까지는 STACK_SETUP_PLAN.md의 안내 문구 + 사용자 매뉴얼 등록만으로 운영한다.

### 5.5 구현 스케치
- 신규 `.claude/skills/stack-guard/SKILL.md`
    - frontmatter: `disable-model-invocation: true`, `agent: builder-sonnet`, `context: fork`, `argument-hint: "[stack summary | empty to read existing docs]"`, `allowed-tools: Read Glob Grep Write Edit Bash`
    - 산출물: `scripts/verify.*`, `package.json`/`pyproject.toml` script 항목, `docs/00-meta/STACK_SETUP_PLAN.md`의 hook 등록 안내 문구(자동 등록 아님 — 위 2단계 참조)
- 수정 `.claude/skills/bootstrap-stack/SKILL.md`
    - 마지막 출력에 "다음 액션으로 `/stack-guard` 실행을 권장"을 텍스트로 추가(자동 호출 아님).
    - `output-checklist.md`에 검증 스크립트 / 통합 명령 / hook 등록 안내 문구 항목 추가.
- 수정 `.claude/skills/validate-workitem/SKILL.md`
    - "반드시 먼저 할 일"에 "통합 검증 명령이 있으면 먼저 실행한다" 추가.
- 수정 `.claude/agents/validator-sonnet.md`
    - 규칙에 "동적 검증 실행 결과가 있으면 정적 판단보다 우선 본다" 추가.
- 수정 `docs/00-meta/GUARDRAILS_STRATEGY.md`
    - `/stack-guard`의 1단계 산출물 범위 명시 (verify 스크립트 + 통합 `validate` 명령 + STACK_SETUP_PLAN.md의 hook 안내 문구). PostToolUse hook 자동 등록은 prototyping 후 별도 항목으로 분리한다는 정책도 함께 명시.

### 5.6 영향 범위
- 신규: `.claude/skills/stack-guard/`, `scripts/verify.{sh,ps1,mjs,py}`, `docs/00-meta/STACK_SETUP_PLAN.md`(`/bootstrap-stack`이 선택 산출물로 약속하지만 빈 보일러플레이트엔 미존재)
- 수정: `.claude/skills/{bootstrap-stack,validate-workitem}/SKILL.md`, `.claude/skills/bootstrap-stack/output-checklist.md`, `.claude/agents/validator-sonnet.md`, `docs/00-meta/{GUARDRAILS_STRATEGY,WORKFLOW,NEW_PROJECT_CHECKLIST}.md`, `README.md`, `README_ko.md`

### 5.7 열린 질문
2단계 prototyping에서 측정할 항목은 5.4 본문에 정리. 본 1단계의 추가 열린 질문:
- mixed env에서 hook을 강제하지 않더라도 line ending 차이는 남는다 — `.gitattributes`로 line ending 통일은 `/stack-guard` 산출물(1단계)에 포함한다.

---

## 6. `/stabilize-milestone` — 마일스톤 안정화

### 6.1 배경 / 동기
마일스톤은 여러 task의 합이다. 각 task가 finalize되었더라도, 마일스톤 단위에서 다음을 한 번 더 회수하지 않으면 누적 회귀와 부채가 쌓인다:
- end-to-end 흐름 검증
- 클린 코드 / 클린 아키텍처 관점 리팩토링 후보 도출
- 누락된 ADR / 문서 갱신
- QA 회귀 점검

이 단계의 명칭은 "마일스톤 마감"보다 "**안정화**"가 동작에 더 가깝다 — 코드 수정·커밋을 하지 않고 통합 점검 결과만 누적 기록하기 때문.

### 6.2 As-is
이 단계는 어디에도 명시되어 있지 않다. 사람이 기억하면 한다.

### 6.3 문제점
- 마일스톤 종결 시점이 모호하다.
- 부채(리팩토링 후보, 문서 갱신, ADR 누락)가 상시 누적된다.
- task 단위 검증만으로는 통합 시점에 발견되는 회귀를 잡지 못한다.

### 6.4 To-be
신규 `/stabilize-milestone [milestone-id]`.

동작:
1. milestone 문서를 읽고 포함된 feature/task 목록을 회수
2. 각 task의 status를 점검 — `done`이 아닌 항목이 있으면 명단을 출력하고 종료(완료를 강제하지 않음)
3. 통합 `validate` 명령 실행 + (있으면) E2E 명령 실행
4. `qa` agent에 회귀·엣지케이스 점검 위임 — qa는 보고만 한다(현재 `qa.md`의 tools에 Write가 없음). 반환된 보고를 본 skill이 받아 `docs/40-validation/QA_FINDINGS.md`에 누적 기록.
5. `reviewer` agent에 리팩토링 후보·아키텍처 부채 점검 위임 — reviewer도 보고만 하고, 본 skill이 받아 `docs/40-validation/IMPROVEMENT_GUIDE.md`에 정리. 구조 변경이 필요해 보이면 메인 세션 가이드에 따라 architect-opus 추가 호출 권장.
6. 미흡한 ADR 후보 제안(예: 마일스톤 중에 내려진 결정인데 ADR이 없는 것)
7. 최종 출력: 통합 결과 / P0·P1·P2 후속 작업 / 다음 마일스톤으로 넘기는 항목

**책임 경계** — 본 skill은 **코드 수정·커밋·workitem status 변경을 하지 않는다**. 검증 결과를 누적 기록하는 문서 갱신(`QA_FINDINGS.md`, `IMPROVEMENT_GUIDE.md`)은 정상 책임이며 그 외 변경은 금지한다. 후속 작업이 필요하면 `/repair-workitem` 또는 새 task로 연결한다(텍스트 제안 출력만).

### 6.5 구현 스케치
- 신규 `.claude/skills/stabilize-milestone/SKILL.md`
    - frontmatter: `disable-model-invocation: true`, `argument-hint: "[milestone id]"`, `allowed-tools: Read Glob Grep Write Edit Bash` — qa/reviewer가 Write 권한이 없으므로 누적 기록은 본 skill이 책임진다(Write/Edit 필요).
    - 본문: 위 7단계. 각 단계별 호출 대상(qa, reviewer)과 회수 방식 명시. 책임 경계(코드 수정·커밋·status 변경 금지)를 본문 첫 줄에.
- 수정 `docs/00-meta/WORKFLOW.md`
    - "5. 마일스톤 안정화" 섹션 신설.
- 수정 `docs/40-validation/QA_FINDINGS.md`
    - 마일스톤 단위 누적을 위해 최상위 헤더 구조를 변경한다 — 기존 평면 묶음을 `## M1 → ### P0/P1/P2/관찰 메모`, `## M2 → ###...` 식으로 마일스톤별로 중첩한다. 마일스톤이 정해지지 않은 초기 프로젝트는 `## 일반` 한 묶음만 둔다.
- 수정 `docs/40-validation/IMPROVEMENT_GUIDE.md`
    - 현재 양식을 유지하되, 각 섹션 안에서 `### M1`, `### M2` 식의 마일스톤 단위 그룹핑을 권장하는 한 단락을 문서 상단에 추가.
- **다운스트림 마이그레이션 가이드** — 본 보일러플레이트는 빈 템플릿이라 적용 시 즉시 변경 가능. 그러나 본 보일러플레이트로 시작한 다운스트림 프로젝트는 이미 누적된 데이터를 가질 수 있다. 마이그레이션은 두 단계: (1) 기존 평면 항목들을 `## M1` 또는 `## 일반` 한 묶음으로 감싸기(편집 1회). (2) 다음 회차부터 새 마일스톤 헤더로 누적. WORKFLOW.md "5. 마일스톤 안정화" 섹션 마지막에 이 마이그레이션 한 단락 추가.

### 6.6 영향 범위
- 신규 1 skill, 문서 3종 갱신.

### 6.7 열린 질문
- 5단계의 리팩토링 후보 도출은 architect-opus(비용↑) vs reviewer(비용↓) 어느 쪽이 적절한가? (잠정: reviewer 기본, "구조 변경이 필요해 보이면 architect-opus 추가 호출"이라는 메인 세션 가이드)
- E2E 명령이 없는 스택은 3번에서 어떻게? (잠정: 통합 명령만 돌리고 E2E는 skip, 출력에 명시)
- 본 skill이 너무 많은 일을 한다 — sub-skill로 쪼갤 가치는? (잠정: 마일스톤은 자주 일어나지 않으므로 단일 skill이 OK. 비용 폭주 시 분해 검토)

---

## 7. `/discover-product` — 발굴 단계 분리

### 7.1 배경 / 동기
현재 `/bootstrap-project`는 한 번의 자연어 입력으로 charter/architecture/M1/F-001을 한꺼번에 만든다. 입력이 짧으면 결과가 얕아지고, 페르소나·pain point·시나리오가 추측으로 채워진다. 사용자가 표현한 pain은 "기획 단계가 부자연스럽다 / 구체성이 부족하다".

기존 항목 6의 해결책은 `/bootstrap-project` 안에 R0~R4 라운드를 모두 담는 것이었다. 그러나 이는 단일 skill을 비대하게 만들고, 이후 charter 갱신·재발굴이 일어날 때 재사용성이 떨어진다. **discovery 단계를 별도 skill로 분리**하면:
- discovery 산출물이 charter와 별도 파일로 보존되어 스택 변경·재발굴 시 재활용 가능
- `/bootstrap-project`는 입력이 분명한 단순 변환 단계로 좁아진다
- 사용자가 발굴 단계를 직접 운전하고 싶을 때(처음부터 보일러플레이트 없이 발굴하고 싶을 때)와 자동 변환만 원할 때를 분리할 수 있다

### 7.2 As-is
- `bootstrap-project/SKILL.md`는 `context: fork`, `agent: architect-opus`, `effort: max`로 architect-opus에 위임된다.
- 본문 절차: 입력 구조화 → 문서 갱신 → 초기 workitem 생성.
- 사용자에게 되묻는 단계가 없음.
- `brief-template.md`가 있지만 사람이 자발적으로 채워와야 효과가 난다.
- 발굴(persona/pain/JTBD/시나리오)과 산출물 생성(charter/architecture/M1/F-001)이 하나의 skill에 묶여 있어 단계별 출구가 없다.

### 7.3 문제점
- 한 번 입력으로 끝내면 페르소나·문제·시나리오가 얕다.
- 가정과 추측의 경계가 흐려져 charter의 신뢰도가 낮아진다.
- 가장 큰 pain point를 찾는 과정 없이 바로 범위가 정해진다.
- 발굴이 charter 생성에 종속되어, 발굴만 하고 멈추거나 발굴 결과를 charter 외 곳에서 재활용하기 어렵다.

### 7.4 To-be

**메커니즘 — 메인 세션이 라운드를 직접 운전 + 산출물은 단일 파일로 보존**

- `/discover-product` skill의 frontmatter에서 `context: fork`/`agent`/`model`/`effort` 라인을 두지 않는다(있으면 제거). 이 skill은 메인 세션에 R0~R4 절차를 기재한 절차서로 동작한다.
- 메인 세션이 각 라운드의 사용자 응답을 직접 받는다. 무거운 추론(R0의 페르소나 후보, R1의 pain inventory)은 architect-opus 단발 sub-call로 위임한다 — sub-call은 task 입력만 받아 결과만 메인으로 돌려준다.
- R0~R3 산출물은 메인 컨텍스트에 누적시키지 않고 단일 파일 `docs/10-charter/DISCOVERY.md`에 누적 적재한다(파일 위치는 charter와 같은 디렉터리, 이름은 명시적). 메인 세션은 라운드 진행에 필요한 최소 요약만 유지한다.
- discovery 산출물(`DISCOVERY.md`)은 git 트래킹 대상으로 둔다 — charter의 근거이자 mid-project 재발굴 시 입력이므로 보존 가치가 있다(`.bootstrap-session-*.md` 같은 ephemeral과 다름).
- 본문에 트레이드오프 단락 명시 — "이 skill은 의도적으로 fork 격리를 풀고 메인 세션이 사용자 인터랙션을 직접 운전한다. R0~R3 산출물은 `DISCOVERY.md`에 적재해 메인 컨텍스트 부담을 최소화하지만, 라운드 요약 일부는 메인에 남는다. `AGENT_EXECUTION_STRATEGY.md`의 '메인 컨텍스트에 오래 보존하지 않는다' 원칙과의 트레이드오프이며, 종료 후 사용자가 `/clear` 또는 새 세션으로 컨텍스트를 정리할 것을 권장한다."

**라운드 구성** — 사용자가 매 라운드 끝에 `skip` / `good` / `refine: ...`로 응답할 수 있다. 각 라운드 종료 후 결과는 `DISCOVERY.md`에 append된다. **단계별 출구 보장**: 어느 라운드에서 멈춰도 그때까지의 산출물(`DISCOVERY.md`)이 `/bootstrap-project`의 입력으로 의미가 있도록 한다.

1. **R0 — 문제 한 줄 + 페르소나 확인**
   - 입력 한 줄을 그대로 되돌려 "이 한 문장이 핵심 맞나" 확인.
   - 동시에 페르소나 후보 2~3개를 한 단락씩 제시(직무·맥락·일과·압력 요소). 사용자가 1개를 고르거나 합쳐달라고 한다.
   - 페르소나 후보 생성은 architect-opus 단발 sub-call로 위임.
2. **R1 — 핵심 pain + JTBD + 시나리오**
   - 선택된 페르소나의 pain 6~10개를 brainstorm 후 빈도×고통으로 정렬, 상위 1~3개를 사용자가 고른다.
   - 가장 큰 pain의 JTBD(Jobs To Be Done) 한 줄, happy path/alternate path/fail path를 5~7단계로 같이 적는다. 사용자가 끊을 지점·수용 가능 fail을 정한다.
   - architect-opus 단발 sub-call로 위임.
3. **R2 — MVP 범위 vs 비범위 / 성공 기준**
   - R1 시나리오를 충족시키는 최소 기능 묶음과 의도적으로 미루는 것을 분리.
   - 측정 가능한 성공 기준 1~3개.
4. **R3 — 핵심 가정 + 열린 질문**
   - R0~R2에서 사용자가 추측으로 답한 모든 항목을 가정으로 표시.
   - 가장 위험한 가정 1~3개에 검증 방법 1줄.
5. **R4 — DISCOVERY.md 정리(저장 단계)**
   - 위 결과를 정해진 양식으로 `DISCOVERY.md`에 정리. 이 skill은 charter/architecture/workitem을 만들지 않는다 — `/bootstrap-project`가 이어 수행한다(자동 호출 아님, 텍스트 제안).

**fast 모드** — `/discover-product --fast "<설명>"`은 R0과 R3을 건너뛰고 R1+R2+R4만 도는 단축 흐름. 시간 없는 prototype 단계에서 사용한다.

**사용자 응답 수단** — 라운드별 응답은 자연어로만 받는다. [`AskUserQuestion`](https://code.claude.com/docs/en/agent-sdk/user-input)은 공식 도구지만(섹션 15.5) 본 안에선 채택하지 않는다 — 메인 세션이 직접 라운드를 운전하므로 사용자와 자연어로 직접 소통할 수 있고, 다중 선택지 UX의 추가 가치가 비용 대비 불명확. 향후 인터랙티브 UX 개선 시 별도 항목으로 채택 검토.

### 7.5 구현 스케치
- 신규 `.claude/skills/discover-product/SKILL.md`
    - frontmatter: `disable-model-invocation: true`, `argument-hint: "[product description | --fast]"`, `allowed-tools: Read Glob Grep Write Edit`. `context: fork`/`agent`/`model`/`effort` 라인은 두지 않는다.
    - 본문: R0~R4 절차. 각 라운드의 입력·산출물·종료 조건 명시. 단계별 출구 보장(어느 라운드에서 멈춰도 `DISCOVERY.md`가 의미 있는 산출물로 남도록). architect-opus 단발 sub-call의 입출력 양식.
    - 트레이드오프 단락 명시.
    - 사용자 응답은 자연어(`AskUserQuestion` 미채택 결정 명시).
- 신규 `.claude/skills/discover-product/persona-template.md`
    - 페르소나 카드 양식(역할/맥락/하루 흐름/현재 해결 방법/실패 패턴/측정 가능한 성공 신호)
- 신규 `.claude/skills/discover-product/pain-template.md`
    - pain 표(빈도/고통/현재 우회 방법/해결 시 효과)
- 신규 `docs/10-charter/_templates/DISCOVERY_TEMPLATE.md`
    - DISCOVERY.md의 정해진 양식(페르소나, pain, JTBD, 시나리오, MVP 범위/비범위, 성공 기준, 가정, 열린 질문)
- 수정 `docs/00-meta/BOOTSTRAP_PROMPT_EXAMPLES.md`
    - `/discover-product` 라운드 인터랙션 예시 1개 추가.

### 7.6 영향 범위
- 신규 1 skill (templates 2개 포함), 신규 discovery 템플릿 1개, 수정 BOOTSTRAP_PROMPT_EXAMPLES, 수정 README 입력 팁 섹션
- 항목 8과 PR로 묶어 적용 권장(charter 생성 경로의 일관성 유지)

### 7.7 열린 질문
- 페르소나가 2명 이상으로 늘어날 때 DISCOVERY 양식을 어떻게 확장할지(잠정: 기본 1명 우선, 2명 이상이면 페르소나별 시나리오 섹션을 분리)
- 다국어 — 한국어 사용자가 많지만 영문 입력도 들어오면 산출물 언어를 어떻게 결정할지(잠정: 입력 언어를 따른다는 규칙을 본문에 명시)
- 단발 sub-call 비용 — R0/R1 두 번 architect-opus를 호출하는 비용. 첫 사용 후 측정해 비용이 크면 R0/R1을 reviewer/planner로 강등하는 옵션 검토.
- DISCOVERY.md를 git 트래킹할지 여부의 후속 결정 — 잠정: 트래킹. ephemeral 메모가 아니라 charter의 근거 문서이기 때문.

---

## 8. `/bootstrap-project`가 discovery 산출물을 읽도록 조정

### 8.1 배경 / 동기
항목 7로 발굴 단계가 분리되면, `/bootstrap-project`의 책임은 "discovery 산출물을 입력으로 charter / architecture / M1 / F-001을 생성 또는 갱신"으로 좁아진다. 책임 단순화 + 재실행 안전성(기존 산출물을 갱신하는 시나리오)이 이 항목의 목표.

### 8.2 As-is
- `/bootstrap-project`는 자연어 입력 한 번을 받아 charter/architecture/M1/F-001을 한꺼번에 만든다.
- 이미 charter가 있는 상태에서 재실행하면 어떻게 동작할지 명시되어 있지 않다.
- frontmatter에 `model: claude-opus-4-6`이 박혀 있어 staleness 위험(항목 1과 결합).

### 8.3 문제점
- discovery 단계가 분리되면 입력 형식이 자연어가 아니라 `DISCOVERY.md`로 바뀐다. 이 변경을 명시하지 않으면 사용자는 어느 입력을 어떻게 줘야 할지 모른다.
- charter가 이미 있는 상태에서 재실행 시 동작이 명확하지 않다(덮어쓰기? 부분 갱신? 사용자 확인?).

### 8.4 To-be
- `/bootstrap-project`의 입력은 다음 우선순위로 결정한다:
    1. `docs/10-charter/DISCOVERY.md`가 있으면 그것을 입력으로 사용
    2. 없으면 사용자 자연어 입력 사용 — 이 경우 본문에서 `/discover-product` 선행을 권장하는 안내를 출력하되 강제 종료하지 않는다(빠른 prototype 사용을 막지 않기 위해)
- 동작:
    1. 입력 회수 — DISCOVERY.md 또는 자연어 입력
    2. 기존 산출물(charter/architecture/M1/F-001) 존재 여부 점검
        - 없으면 새로 생성
        - 있으면 "갱신 모드" — 기존 내용을 읽고 변경분을 식별해 사용자에게 diff 요약을 보여주며 확인 후 반영. 덮어쓰기는 사용자 명시 동의 시에만.
    3. architect-opus 단발 sub-call로 산출물 생성/갱신 — 입력은 DISCOVERY.md 또는 자연어 입력 + 기존 산출물(있으면)
    4. 산출물 위치는 기존과 동일(`docs/10-charter/PROJECT_CHARTER.md`, `docs/20-system/ARCHITECTURE_OVERVIEW.md`, `docs/30-workitems/milestones/M1-foundation.md`, `docs/30-workitems/features/F-001-core-value.md`)
- charter 템플릿에 페르소나/시나리오/가정 섹션은 그대로 유지 — `DISCOVERY.md`의 내용을 그대로 박는다.

### 8.5 구현 스케치
- 수정 `.claude/skills/bootstrap-project/SKILL.md`
    - frontmatter에서 `model` 라인 제거(별칭 정책 — 항목 1). `context: fork` / `agent: architect-opus`는 유지(이 skill은 단발 변환 작업이므로 fork가 적절).
    - 본문: 입력 우선순위(`DISCOVERY.md` → 자연어 입력) + 갱신 모드 흐름 + sub-call 입출력 양식.
    - "이 skill은 발굴이 아니라 변환을 한다 — 발굴은 `/discover-product`에서" 한 줄 명시.
- 수정 `.claude/skills/bootstrap-project/output-checklist.md`
    - 갱신 모드 흐름(기존 산출물 점검 → diff 요약 → 사용자 확인) 항목 추가.
- 수정 `docs/10-charter/PROJECT_CHARTER.md` (template)
    - 섹션 추가: "2.1 페르소나", "3.1 핵심 시나리오 (happy/alt/fail)", 기존 "9. 열린 질문" 위에 "8.1 핵심 가정"
- 수정 `.claude/skills/bootstrap-stack/SKILL.md`
    - frontmatter `model` 라인 처리(항목 1과 결합) — 별칭 또는 제거.
- 수정 `README.md`, `README_ko.md`
    - 권장 흐름을 `/discover-product → /bootstrap-project → /bootstrap-stack`으로 갱신.

### 8.6 영향 범위
- 수정 2 skill 본문 (`bootstrap-project`, `bootstrap-stack`), 수정 charter 템플릿, 수정 README
- 항목 7과 PR로 묶어 적용 권장.

### 8.7 열린 질문
- 갱신 모드의 "diff 요약 후 사용자 확인" 절차가 너무 무거우면 사용자가 빠른 변경을 못한다 — 잠정: 사용자가 인자로 `--apply` 같은 force 모드를 줄 수 있도록.
- 자연어 입력만 주고 `/discover-product`를 건너뛰는 사용자가 일정 비율 있을 것 — 그 흐름의 산출물 품질이 어느 수준인지 측정 후, 너무 떨어지면 자연어 단발 입력 시 결과에 "discovery를 통한 결과보다 얕을 수 있음" 표시를 박는다.

---

## 9. `/batch` + worktree 가이드 정리

### 9.1 배경 / 동기
이 저장소는 메인 세션이 오케스트레이션을 맡고 실작업을 위임하는 운영 모델을 약속한다. "독립적인 여러 task 동시 처리"는 그 모델의 진입점으로 여러 곳에서 언급된다. 그러나 진입점의 정체와 사용 시점이 사용자에게 명확하지 않다. 단, 즉시 실패는 아니고 핵심 워크플로 병목도 아니므로 우선순위는 낮다.

### 9.2 As-is
- `CLAUDE.md`의 위임 트리거: `독립적인 여러 task 동시 처리 → /batch 또는 worktree`.
- `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`도 동일 표기.
- `.claude/skills/` 아래에 `batch` skill은 없다.
- **`/batch`는 Claude Code 공식 bundled skill이다** — `/simplify`, `/batch`, `/debug`, `/loop`, `/claude-api`와 함께 번들 제공. `/batch`를 발화하면 작업 단위당 격리된 git worktree에서 background agent를 띄운다([skills 문서](https://code.claude.com/docs/en/skills)·[commands 문서](https://code.claude.com/docs/en/commands) 참조).
- worktree 사용도 같은 줄에 묶여 안내되지만, 언제·어떻게(Agent 도구의 `isolation: "worktree"` 인자) 가이드가 별도로 없다.

### 9.3 문제점
- 보일러플레이트 문서가 `/batch`를 명사처럼 쓴다 — custom skill인지 bundled인지 본문만으로 알기 어렵다.
- bundled `/batch`는 worktree에서 background agent를 띄우는 비교적 무거운 패턴이다. "한 메시지에서 Agent 도구를 여러 번 병렬 호출"하는 가벼운 패턴과 구분 없이 권장되어, 사용자가 어느 쪽을 골라야 할지 판단 근거가 없다.

### 9.4 To-be
**기본 워크아이템 흐름을 대체하는 것이 아니라, 독립적인 큰 작업에서 선택적으로 쓰는 고급 옵션**으로 위치를 정한다.

위임 트리거 표의 `/batch 또는 worktree` 항을 다음 3개 패턴으로 분리·명시한다(가벼움 순):

1. **한 메시지에서 Agent 호출 병렬** — 가장 가벼움. 독립적인 짧은 sub-agent 작업 여러 개를 한 turn에 띄운다. 메인이 결과만 통합. 일상 위임의 기본.
2. **`isolation: "worktree"` 인자로 단일 Agent 호출** — Agent 도구 호출 시 인자로 지정. sub-agent가 작업 디렉터리를 격리 git worktree에 받아 진행. 변경이 없으면 자동 cleanup, 변경이 있으면 worktree 경로와 브랜치 이름을 반환([공식 문서](https://code.claude.com/docs/en/sub-agents) — sub-agent frontmatter `isolation` 필드 설명). 같은 파일 충돌 가능성 있는 단일 작업에 적합.
3. **bundled `/batch` 호출** — 사용자가 직접 발화. Claude Code가 작업 단위당 background agent + worktree를 자동 생성. 큰 마이그레이션·codebase-wide 변경 같은 코드 단위 분리가 분명한 큰 작업에 적합. **본 보일러플레이트의 custom skill이 아니라 Claude Code bundled skill임을 문서에 명시한다.**

선택 기준: 가벼운 병렬 → 1, 같은 파일 충돌 가능성 있는 단일 작업 → 2, 작업 단위가 분명한 codebase-wide 분산 작업 → 3.

### 9.5 구현 스케치
- 수정 `CLAUDE.md`
    - 위임 트리거 표의 `독립적인 여러 task 동시 처리` 행을 "병렬 패턴 3종 — 상세는 AGENT_EXECUTION_STRATEGY.md의 '병렬 패턴 3종' 단락"으로 축약.
- 수정 `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`
    - 위임 트리거 표 동일하게 수정.
    - "병렬 작업 원칙"의 `/batch` 단독 언급을 위 3종으로 확장.
    - 신규 단락 "병렬 패턴 3종" 추가 — 위 9.4의 1·2·3을 본문 형태로. `/batch`가 Claude Code bundled skill이라는 한 줄 명시.

### 9.6 영향 범위
- 수정 2 문서 (`CLAUDE.md`, `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`)
- 신규 skill 없음(bundled `/batch` 그대로 활용)

### 9.7 열린 질문
- `/batch`의 trigger 조건을 보일러플레이트가 별도로 wrapping할 가치가 있는가? — 잠정: 없음. bundled로 충분하고, 본 보일러플레이트의 차별점(워크아이템 라이프사이클)은 항목 3·4·6의 skill들이 담당.

---

## 10. 에이전트 TDD 기본 흐름

### 10.1 배경 / 동기
이 보일러플레이트의 가치는 fork된 미래 프로젝트에서 에이전트가 일관된 규율로 작업하는 것이다. TDD는 그 규율 중 가장 잘 작동하는 신호로, builder-sonnet이 "구현 → 사후 테스트"가 아니라 "AC 정의 → 실패 테스트 → 최소 구현 → 정리" 사이클을 따르게 만든다. 동시에 AC를 코드 작성 전 명문화하기 때문에 항목 11의 단순성·YAGNI 원칙도 함께 강화한다 — 범위가 명문화되어야 "딱 그만큼"이 가능하다.

### 10.2 As-is
- `docs/30-workitems/_templates/TASK_TEMPLATE.md`의 `## 6. 테스트 포인트` — 자유 텍스트 메모. AC인지 단순 메모인지 구분 없음.
- `docs/30-workitems/_templates/FEATURE_TEMPLATE.md`의 `## 8. 검증 방법` — 시나리오 수준. task 단위 AC와의 매핑 규칙 없음.
- `.claude/agents/builder-sonnet.md` 규칙: "관련 테스트를 추가하거나 보강한다" — 사후 테스트 가능, TDD 강제 없음.
- `.claude/skills/implement-workitem/SKILL.md`: "필요한 테스트가 있으면 함께 보강한다" — 임의적.
- `.claude/skills/validate-workitem/SKILL.md`: "빠진 검증 포인트가 있는가" — 검증되지 않은 코드를 통과시킬 가능성.
- 통합 `validate` 명령(항목 5)이 있어도 테스트가 늦게 작성됐는지는 보지 않는다.

### 10.3 문제점
- 사후 테스트는 구현을 그대로 따라가서 발견 못한 케이스가 그대로 통과한다 — 회귀 위험.
- AC가 task 시작 전 명문화되지 않으면 finalize 시점의 "완료" 기준이 흐려진다.
- `/repair-workitem`(항목 3) 도입 시 "무엇을 fix"인지 모호해 무한 루프 가드(연속 3회)에 도달하기 쉽다.
- 시간 압박 시 builder-sonnet이 테스트를 건너뛰고 "구현만 했다"는 결과를 반환할 수 있다.

### 10.4 To-be

**핵심 규칙** — `/implement-workitem`의 디폴트 흐름을 **Red → Green → Refactor 3 phase**로 명시한다. 각 phase는 종료 조건이 분명하다.

1. **Red** — task의 `## 6. Acceptance Criteria` 항목 1개를 골라 그것을 위반하는 실패 테스트를 작성. 테스트가 "원하는 이유로" 실패하는지 확인 후 phase 종료.
2. **Green** — 그 테스트를 통과시키는 **최소 코드**만 작성. 다른 AC를 미리 만족시키지 않는다(항목 11의 YAGNI 강화점).
3. **Refactor** — 항목 11의 Clean Code 6항목 체크리스트(naming/size/duplication/premature abstraction/comment policy/layer leak)에 따라 정리. 외부 행동을 바꾸지 않는다.

위 사이클을 task의 모든 AC가 소진될 때까지 반복.

**TASK_TEMPLATE 분리** — 기존 `## 6. 테스트 포인트`를 다음 두 섹션으로 교체:
- `## 6. Acceptance Criteria` — Given-When-Then 또는 명세 형태. 항목별 식별자(`AC-1`, `AC-2` ...) 부여. 측정 가능해야 한다.
- `## 6-1. 테스트 시나리오 (TDD Red)` — 각 AC에 대응하는 테스트 파일·테스트 이름. 사람이 미리 채우거나 builder-sonnet이 Red phase 시작 전 채운다.

**검증 흐름**:
- `/validate-workitem`(validator-sonnet)에 다음 점검을 추가한다:
    - **AC ↔ 테스트 매핑** — task 문서의 AC-N마다 대응하는 테스트가 존재하는가(자연어 매칭 휴리스틱).
    - **테스트 선행 휴리스틱** — git log에서 동일 task 범위의 테스트 파일 추가/수정이 구현 파일보다 먼저(또는 동일 커밋) 들어왔는지. 단순 경고로만 보고하고 강제 종료하지 않는다(소규모 작업이 한 커밋에 묶이는 경우 정상).
    - 결과는 항목 3의 validation report에 `AC-1 ✅ / AC-2 ❌(테스트 없음)` 형태로 기록.
- `/finalize-workitem`(항목 4)은 통합 `validate` 명령 통과 외에 **AC 미충족 항목이 있으면 Needs Fix**로 종료한다.

**opt-out — `tdd: skip`** — 모든 task가 TDD에 적합하지 않다(spike, prototype, 외부 의존 탐색). TASK_TEMPLATE에 선택 섹션을 둔다:
- `## 6-2. TDD opt-out` — 비어 있으면 TDD 적용. 채울 때만 사유와 follow-up task 링크가 모두 있어야 한다(예: "spike 종료 후 T-014에서 TDD로 재구현"). 둘 중 하나라도 비면 형식 위반으로 표시.
- finalize 시점에 opt-out 사유를 사용자에게 명시적으로 보여주고 확인. 남용 방지.

**fast 모드** — `/implement-workitem --fast [task-id]`는 RGR 사이클을 1회만 돌려 첫 AC만 완료하고 종료한다. prototype에서 빠르게 흐름을 검증할 때 사용. 나머지 AC는 후속 호출.

### 10.5 구현 스케치

- 수정 `docs/30-workitems/_templates/TASK_TEMPLATE.md`
    - `## 6. 테스트 포인트` → `## 6. Acceptance Criteria` (`AC-1`, `AC-2` ... 형식)
    - 신규 `## 6-1. 테스트 시나리오 (TDD Red)` — AC별 테스트 파일·테스트 이름.
    - 신규 `## 6-2. TDD opt-out` — 비어 있는 게 디폴트. 채울 때만 사유와 follow-up task 링크 둘 다 필수.
- 수정 `docs/30-workitems/_templates/FEATURE_TEMPLATE.md`
    - `## 8. 검증 방법`에 "task 단위 AC로 분해된다" 한 줄 추가(feature는 시나리오, task는 AC라는 역할 분리).
- 수정 `.claude/skills/implement-workitem/SKILL.md`
    - 본문을 Red → Green → Refactor 3 phase로 분리. 각 phase의 입력·출력·종료 조건 명시.
    - opt-out 사유가 있을 때의 흐름(테스트 작성 건너뛰지만 사유는 출력에 명시).
    - `--fast` 인자 처리.
- 수정 `.claude/agents/builder-sonnet.md`
    - 규칙 추가: "AC가 정의된 task는 Red → Green → Refactor 사이클로 진행한다. opt-out 사유가 없으면 테스트가 구현보다 먼저 작성되어야 한다."
    - 출력 항목에 "AC별 진행 상태(완료/미완료)" 추가.
- 수정 `.claude/skills/validate-workitem/SKILL.md`
    - "AC ↔ 테스트 매핑 점검"·"테스트 선행 휴리스틱" 단계 추가.
- 수정 `.claude/agents/validator-sonnet.md`
    - 규칙: "AC 항목과 실제 테스트가 1:1 또는 다대일로 매핑되는지 점검. 미매핑 항목은 report에 명시."
- 수정 `.claude/skills/finalize-workitem/SKILL.md`(항목 4 신설본)
    - 통과 조건에 "AC 미충족 항목 0개" 추가.
- 수정 `CLAUDE.md`
    - "코드 및 변경 원칙" 섹션에 "구현 시 TDD 사이클(Red → Green → Refactor)을 디폴트로 따른다. opt-out은 task 문서에 사유와 follow-up이 모두 있을 때만." 한 단락 추가.
- (권장) 신규 `docs/90-decisions/ADR-00x-tdd-default.md` — TDD를 디폴트로 채택하고 opt-out 절차를 정의한 결정.

### 10.6 영향 범위
- 수정: 2 template (TASK, FEATURE), 2 skill (implement-workitem, validate-workitem), 2 agent (builder-sonnet, validator-sonnet), 1 신설 skill 본문 (finalize-workitem, 항목 4), 1 CLAUDE.md
- 신규: 1 ADR (권장)
- 의존: 항목 3·4(report에 AC 기록), 항목 5(통합 validate 명령에 test runner 포함). 항목 5 적용 후 도입이 가장 자연스럽다.

### 10.7 열린 질문
- AC 자연어 매핑이 안정적인가? 짧은 AC는 매칭이 헐거울 수 있다 — 잠정: AC-N 식별자를 테스트 이름에 포함하는 컨벤션(`test_AC_1_user_can_login`) 권장. 강제 아님.
- legacy 코드(테스트 없는 기존 모듈) 수정은 TDD 강제? — 잠정: characterization test를 먼저 작성한 뒤 RGR. task 단위로 사용자 결정.
- prototype 단계에서 opt-out 비율이 높아지면 정책 가치가 희석된다 — 한 마일스톤의 opt-out 비율을 `/stabilize-milestone`(항목 6)이 보고하는 항목으로 추가할지 검토.

---

## 11. 에이전트 단순성·Clean Code·Clean Architecture 가드레일

### 11.1 배경 / 동기
사용자의 핵심 요구는 "fork된 미래 프로젝트에서 AI 에이전트가 요구사항을 오버하지 않으면서 가장 단순하고 가독성 좋은 코드로 구현"하게 만드는 것이다. 이 요구는 세 층으로 분해된다(우선순위 순):

1. **단순성·YAGNI** — 요구한 범위만 구현, 추측성 추상화 금지, 미래 대비 코드 금지.
2. **Clean Code** — 명확한 네이밍, 한 함수 한 가지 일, 중복 회피, WHY가 비자명할 때만 주석.
3. **Clean Architecture** — 의존성 규칙, 레이어 경계 — 단 프로젝트 규모가 정당화할 때만.

순서가 중요하다. Clean Architecture를 단순성 위에 두면 작은 prototype에 4-layer를 강제해 사용자의 "오버하지 않으면서"와 직접 충돌한다.

### 11.2 As-is
- `CLAUDE.md`의 "코드 및 변경 원칙":
    - "읽기 쉽고 유지보수하기 쉬운 코드를 우선한다" — 추상적, 강제력 약함.
    - "계획되지 않은 광범위한 리팩토링은 피한다" — 리팩토링 범위만 다룬다. YAGNI·추측성 추상화는 명시 없음.
- `.claude/agents/builder-sonnet.md` 규칙: "범위 밖 변경은 하지 않는다", "국소 리팩토링" — 범위는 다루나 단순성·추상화 정책 없음.
- `.claude/agents/reviewer.md` 규칙: "누락된 요구사항, 모순, 숨은 복잡도, 엣지 케이스 누락" — Clean Code 6항목 같은 명시적 체크리스트 없음.
- `docs/20-system/ARCHITECTURE_OVERVIEW.md` 템플릿: `## 3. 상위 아키텍처(Container 레벨)` 섹션은 있으나 layer 경계·의존성 규칙 섹션은 없다.
- `/stabilize-milestone`(항목 6 신설본) 5단계 reviewer 위임의 "리팩토링 후보·아키텍처 부채 점검" — 기준 미정의.

### 11.3 문제점
- 단순성 정책이 약해서 fork된 프로젝트에서 에이전트가 추측성 try/except, 사용 안 되는 옵션 인자, premature factory 같은 것을 자연스럽게 만든다.
- Clean Code 체크리스트가 reviewer에 명시되지 않아 회차마다 지적 일관성이 떨어진다.
- ARCHITECTURE_OVERVIEW에 의존성 규칙 자리가 없어 마일스톤 후반부에 layer leak이 발견되어 비싼 리팩토링이 강제된다 — 또는 그 반대로, 레이어 규칙이 너무 일찍 박혀 작은 prototype에 4-layer가 강제된다.
- `/stabilize-milestone`의 "리팩토링 후보"가 어떤 기준인지 불명 → 결과가 세션마다 다르다.

### 11.4 To-be

**11a (의존 없음, 즉시 적용 가능)**

**1. CLAUDE.md에 단순성 단락 추가** — fork된 프로젝트의 모든 세션이 보는 위치이므로 정책 박는 비용이 가장 낮다. "코드 및 변경 원칙" 섹션에 다음 항목 추가:
- 요구한 범위만 구현한다. 추측성 추상화·미래 대비 코드·계획에 없는 헬퍼는 만들지 않는다.
- 동일 패턴이 3회 이상 반복될 때까지는 추출하지 않는다. 비슷한 코드 3줄이 premature abstraction보다 낫다.
- 시스템 경계(외부 입력, 외부 API)에서만 입력 검증·에러 핸들링을 둔다. 내부 호출에는 두지 않는다.
- WHY가 비자명할 때만 주석을 단다(숨은 제약, 미묘한 invariant, 특정 버그 우회). WHAT 주석은 좋은 식별자 이름으로 대체한다.
- backwards-compat shim, feature flag, dead 변수의 `_` rename 같은 호환 hack을 만들지 않는다. 정말 안 쓰면 삭제한다.

**2. builder-sonnet에 단순성 self-check 단계 추가** — 구현 직후 출력하기 전, 다음 4가지를 자기 점검:
- 추가한 추상화·팩토리·헬퍼가 정말 2회 이상 사용되는가?
- 추가한 try/except·null check가 시스템 경계에서 발생하는가, 아니면 내부 호출인가?
- 새 주석이 WHY를 설명하는가, WHAT을 설명하는가?
- 삭제 가능한 dead code(쓰이지 않는 import·변수·branch)가 남았는가?

self-check를 통과 못하면 출력에 명시(완벽 종료가 아니라 "남은 정리 항목 N건"으로 보고).

**3. reviewer에 Clean Code 6항목 체크리스트 명시** — agent 규칙에 다음 항목 추가:
1. **Naming** — 함수·변수·파일 이름이 의도를 표현하는가. 약자·번호·`util`/`helper`/`manager` 같은 무의미한 이름 회피.
2. **Function size + single responsibility** — 한 함수가 한 가지를 하는가. 여러 추상화 수준을 섞지 않는가.
3. **Duplication** — 3회 이상 반복되는 패턴인가. 1~2회는 인라인 유지.
4. **Premature abstraction** — 1~2회 사용에 그치는 abstraction이 있는가(인라인 후보).
5. **Comment policy** — WHAT 주석이 있는가(이름으로 대체할 후보). WHY 주석이 누락된 invariant가 있는가.
6. **Layer leak** — 의존성 규칙(있으면) 위반이 있는가. 상위 레이어가 하위 모듈을 직접 의존하지 않는가.

reviewer는 P0/P1/P2 분류와 함께 위 6항목 중 어디에 해당하는지 라벨링한다.

**4. architect-opus 규칙 보강** — 큰 설계 결정 시 "프로젝트 규모가 4-layer를 정당화하는가" self-check를 명시. 작은 프로젝트에 4-layer를 박는 것을 방지한다.

**11b (항목 6 적용 후)**

**5. ARCHITECTURE_OVERVIEW에 의존성 규칙 섹션 추가, YAGNI 보호 단서 동봉** — 신규 `## 3-1. 레이어 경계 + 의존성 규칙`:
- 본문 첫 줄: **"프로젝트 규모가 정당화될 때만 채운다. 단일 모듈 prototype은 비워둬도 된다. 단, 모듈이 3개 이상으로 분리되면 채운다."**
- 가이드: 의존성 방향 예시(`Domain ← Use Case ← Interface Adapter ← Framework`), ASCII 다이어그램, 위반 사례 1~2개.
- 비어 있어도 `/stabilize-milestone`이 모듈 수가 3개 이상이면 채울 것을 권장 출력한다.

**6. /stabilize-milestone 점검 기준 정교화**(항목 6 본문에 통합):
- 5단계 reviewer 위임 시 위 Clean Code 6항목 체크리스트를 명시 입력으로 전달.
- 6단계 ADR 후보 산출 기준에 "layer 경계·의존성 규칙 변경" 추가 — Clean Architecture 변경은 ADR로 보존.
- 새 단계 추가: ARCHITECTURE_OVERVIEW의 `## 3-1`이 비어 있고 모듈 수가 3개 이상이면 채울 것을 권장 출력.

### 11.5 구현 스케치

**11a 변경**:
- 수정 `CLAUDE.md`
    - "코드 및 변경 원칙" 섹션에 11.4-1의 5개 항목 추가.
- 수정 `.claude/agents/builder-sonnet.md`
    - 규칙: "구현 출력 전 단순성 self-check 4항목을 점검한다" 한 단락 추가.
    - 출력 항목에 "남은 정리 항목" 추가.
- 수정 `.claude/agents/reviewer.md`
    - 규칙 섹션에 Clean Code 6항목 체크리스트 추가. P0/P1/P2 분류와 항목 라벨링 명시.
- 수정 `.claude/agents/validator-sonnet.md`
    - 추가 점검 한 줄: "범위 밖 추상화·premature factory가 보이면 report에 표시."
- 수정 `.claude/agents/architect-opus.md`
    - 규칙: "프로젝트 규모가 4-layer를 정당화하는가 self-check. 정당화되지 않으면 단일 layer + 모듈 단위 의존성 규칙 권장."
- 수정 `.claude/skills/implement-workitem/SKILL.md`
    - Refactor phase 시 단순성 self-check 4항목 + Clean Code 6항목을 참조하도록 짧게 안내.

**11b 변경 (항목 6 동시 또는 이후)**:
- 수정 `docs/20-system/ARCHITECTURE_OVERVIEW.md` (template)
    - 신규 `## 3-1. 레이어 경계 + 의존성 규칙` 섹션. 본문 첫 줄에 YAGNI 보호 단서.
- 수정 `.claude/skills/stabilize-milestone/SKILL.md`(항목 6에서 신설)
    - 5단계 reviewer 입력에 Clean Code 6항목 체크리스트 명시.
    - 6단계 ADR 후보 기준에 "layer 경계·의존성 규칙 변경" 추가.
    - 새 단계: ARCHITECTURE_OVERVIEW `## 3-1` 자동 채움 권장.
- (권장) 신규 `docs/90-decisions/ADR-00x-simplicity-and-architecture.md` — 단순성 1순위, Clean Code 2순위, Clean Architecture 3순위 정책 결정.

### 11.6 영향 범위
- 수정: 1 CLAUDE.md, 1 architecture template, 4 agent (builder-sonnet, validator-sonnet, reviewer, architect-opus), 2 skill (implement-workitem, stabilize-milestone)
- 신규: 1 ADR (권장)
- 의존: 11a는 의존 없음(즉시 적용 가능). 11b는 항목 6(`/stabilize-milestone`)이 통합 진입점.

### 11.7 열린 질문
- 단순성 self-check 4항목이 builder-sonnet 출력을 늘려 비용이 증가하는가? — 잠정: 4항목은 매우 짧으므로 비용 영향 제한적. 측정 후 필요 시 축약.
- Clean Code 6항목 체크리스트가 reviewer 비용을 늘리는가? — 잠정: 라벨링은 어차피 reviewer가 P0/P1/P2 분류를 하므로 추가 비용 미미.
- "프로젝트 규모가 정당화될 때만"의 기준이 모호 — 잠정: "모듈 3개 이상" 휴리스틱을 ARCHITECTURE_OVERVIEW 섹션에 명시. 사용자 판단 우선.
- legacy 프로젝트로 이 보일러플레이트를 적용할 때 기존 코드의 premature abstraction은? — 잠정: legacy는 즉시 제거하지 않고 `/stabilize-milestone`이 후보로만 보고, 사용자 결정.
- 단순성과 TDD(항목 10)의 충돌 — TDD의 Red phase 자체가 ceremony로 보일 수 있다. 잠정: AC가 먼저 명문화되므로 오히려 YAGNI를 강화한다고 본다(범위 fix). 측정 후 충돌이 보이면 항목 10에 fast 모드를 더 자주 사용하도록 가이드.

---

## 12. 단일 출처(SSOT) 구조 — 중복 제거 + cross-ref 표준화

### 12.1 배경 / 동기

이 보일러플레이트의 모든 가치(fork 후 안정성, 변경 비용 최소화, 사용자 혼선 최소화)는 **같은 사실이 한 곳에서 정의되고 다른 곳은 그것을 참조한다**는 단일 출처(Single Source of Truth, 이하 SSOT) 원칙에 달려 있다. 그러나 문서·agent·skill 정의가 진화하면서 동일 사실이 여러 곳에 박히게 됐다. 본 IMPROVE-LIST 자체가 그 증거다 — 항목 1은 모델 별칭을 4 surface에서 동시 변경하라고 한다. 항목 9는 위임 트리거 표를 2곳에서 동시 변경하라고 한다. 사용자가 어느 한 곳을 빠뜨리면 fork된 새 프로젝트에서 stale·모순으로 노출된다.

본 항목은 (1) 어떤 사실이 어디에 한 번만 정의되어야 하는지(canonical owner)를 정하고, (2) 다른 surface는 짧은 요약 + 링크로 줄인다. 이렇게 하면 향후 정책 변경 시 한 곳만 고치면 모든 곳이 자동으로 따라간다. 본 항목은 항목 3~11이 만드는 신규 ADR·산출물·정책의 **전제 인프라**이므로 항목 1·2 직후 적용한다.

### 12.2 As-is — 발견된 중복·모순·끊김

**A. 동일 사실의 다중 정의 (literal duplication)**

| 사실 | 등장 위치 | 비고 |
|------|----------|------|
| 문서 계층 정의 (`docs/00-meta`, `docs/10-charter`, ...) | `CLAUDE.md`, `docs/00-meta/TEMPLATE_GUIDE.md`, `.claude/skills/boilerplate-context/SKILL.md`, `README.md`, `README_ko.md` | 5곳, 표현 미세 차이 (예: "템플릿 사용법과 공통 운영 규칙" vs "템플릿 사용법, 워크플로우, guardrail 철학" vs "templates and operational rules") |
| 위임 트리거 표 | `CLAUDE.md`, `docs/00-meta/AGENT_EXECUTION_STRATEGY.md` | 2곳, planner agent 행이 CLAUDE.md에서 누락(모순) |
| 상태값 + 전이 규칙 | `docs/00-meta/WORKFLOW.md`(전이 다이어그램), `docs/00-meta/TEMPLATE_GUIDE.md`(단순 리스트), `docs/30-workitems/README.md`(단순 리스트), `docs/90-decisions/_ADR_GUIDE.md`(ADR 전용) | 4곳, WORKFLOW.md만 전이가 있음 |
| 스킬 실행 순서 (한 줄 흐름) | `README.md`, `README_ko.md`, `docs/00-meta/AGENT_EXECUTION_STRATEGY.md` | 3곳 |
| Guardrail 핵심 원칙 (cross-platform 우선) | `README.md`, `README_ko.md`, `docs/00-meta/GUARDRAILS_STRATEGY.md`(출처), `docs/00-meta/TEMPLATE_GUIDE.md`(요약), `.claude/skills/boilerplate-context/SKILL.md`(요약) | 5곳 |
| Bootstrap 사용 흐름 (`/bootstrap-project → /bootstrap-stack`) | `README.md`, `README_ko.md`, `docs/00-meta/TEMPLATE_GUIDE.md`, `docs/00-meta/NEW_PROJECT_CHECKLIST.md`, `docs/00-meta/BOOTSTRAP_PROMPT_EXAMPLES.md` | 5곳 |
| 모델 ID/별칭 (항목 1과 결합) | `.claude/settings.json`, `.claude/agents/architect-opus.md`, `.claude/skills/bootstrap-project/SKILL.md`, `.claude/skills/bootstrap-stack/SKILL.md` | 4곳 |
| 보일러플레이트 핵심 원칙 (작업·문서·코드 변경) | `CLAUDE.md`(3개 섹션), `TEMPLATE_GUIDE.md`, `WORKFLOW.md`, `boilerplate-context` skill, `AGENT_EXECUTION_STRATEGY.md` | 5곳에 부분 중복 |

**B. 모순 (실제로 다른 내용)**

- **B-1** — `CLAUDE.md`의 위임 트리거에 `planner` agent 행이 없으나 `AGENT_EXECUTION_STRATEGY.md`에는 있다. 항목 2가 부분 처리하지만 SSOT 자체로는 미해결.
- **B-2** — `docs/00-meta/TEMPLATE_GUIDE.md`의 "새 프로젝트 시작 순서"는 수동 흐름(charter 직접 수정 → ARCHITECTURE_OVERVIEW 작성 → ...)을 1번 항목으로 두고 "권장 시작 방식"(/bootstrap-project)을 끝에 짧게 둔다. 반면 `README.md`와 `NEW_PROJECT_CHECKLIST.md`는 `/bootstrap-project` 우선. 우선순위가 문서마다 다르다.

**C. 끊긴 cross-ref / orphan**

- **C-1** — `.claude/skills/boilerplate-context/SKILL.md`가 `WORKFLOW.md`/`AGENT_EXECUTION_STRATEGY.md` 같은 깊은 운영 원칙으로의 링크 없음. fork 후 새 세션이 자동 로드되지만 어디로 더 가야 알 수 있는지 모른다.
- **C-2** — `docs/00-meta/GLOSSARY.md`가 거의 비어 있고("예시 항목" 8개) 어디서도 참조되지 않음. fork된 사용자가 이걸 어떻게 채울지 가이드 없음.
- **C-3** — `.claude/skills/write-charter/SKILL.md`, `.claude/skills/write-architecture/SKILL.md`가 존재하지만 `README`/`WORKFLOW`/`AGENT_EXECUTION_STRATEGY`/`NEW_PROJECT_CHECKLIST` 어디에도 사용 안내 없음 → orphan skill (항목 2가 삭제 결정, SSOT 관점에서 합당).
- **C-4** — `docs/90-decisions/`에 ADR 인덱스가 없다. 현재 ADR-001 한 개뿐이지만 IMPROVE-LIST 적용 후 ADR-002~ADR-00x가 다수 추가될 예정이며 인덱스 부재 시 검색 비용↑.
- **C-5** — IMPROVE-LIST 적용 시 신설되는 산출물(`DISCOVERY.md`, `STACK_SETUP_PLAN.md`, `reports/<task-id>.md`)이 어디 있는지 한 곳에서 안내하지 않음 → 사용자가 디렉터리를 일일이 ls해야 함.

**D. 형식 / 표현 불일치**

- **D-1** — agent 정의 frontmatter `model` 필드 표기 혼재 (이미 항목 1).
- **D-2** — skill 정의가 agent 바인딩 있는 군과 없는 군으로 나뉨 (이미 항목 2).
- **D-3** — 한국어/영어 README 동기화는 사람 책임 → drift 위험. 본 항목 12.4의 SSOT 패턴으로 README 본문이 짧아지면 drift 표면 자체가 줄어 영향이 작아진다.

**E. 절차 / 체크리스트의 다중 정의**

- **E-1** — "새 프로젝트 시작 절차"가 4곳(`README`, `TEMPLATE_GUIDE`, `NEW_PROJECT_CHECKLIST`, `BOOTSTRAP_PROMPT_EXAMPLES`)에 다른 형식으로 등장. 변경 시 4곳 갱신.
- **E-2** — 본 IMPROVE-LIST 자체가 SSOT 위반의 사례다. 섹션 14(체크리스트)가 항목 1·2·3·4·5·6·7·8·9·10·11의 변경을 평면적으로 나열 — 항목 본문에서 이미 명시한 변경을 다시 적는다. 적용 PR이 체크리스트 갱신을 잊으면 drift 발생.

### 12.3 문제점

- 변경 비용이 비대칭으로 커진다 — 모델 별칭 1개 바꾸려고 4 surface 동시 변경, 위임 트리거 변경하려고 2곳 동시 변경. 사람이 한 곳을 빠뜨리면 stale로 노출.
- 표현 차이가 사용자 혼선을 만든다 — 같은 디렉터리를 어떤 곳은 "운영 원칙"이라 하고 다른 곳은 "guardrail 철학"이라 부르면 fork된 사용자는 어느 정의가 권위인지 판단해야 한다.
- ADR 부재로 정책의 권위가 약해진다 — 단순성·TDD·모델 별칭은 정책이지만 현재 어느 ADR에도 박혀 있지 않다(IMPROVE-LIST에서 신설 예정). 정책이 코드/skill 본문에만 박히면 6개월 뒤 fork한 사용자는 "왜 이 정책이 박혔는가"를 추적할 길이 없다.
- 검증 결과·DISCOVERY 같은 ephemeral 산출물의 위치가 흩어지면 인덱싱·정리·아카이브가 어렵다.
- README 동기화는 cross-language drift의 첫 표면 — 본문이 길수록 drift 위험↑.

### 12.4 To-be — SSOT 설계

**대원칙 — "정의 1곳, 다른 곳은 링크"**

각 사실은 단 하나의 canonical 문서에 정의된다. 다른 문서는:
- 한 줄 요약 + canonical 문서로의 링크만 둔다.
- 정의 본문을 복제하지 않는다.
- 정의를 바꿀 일이 생기면 canonical만 바꾼다 — 다른 곳은 링크가 따라간다.

**Canonical Owner 매핑**

| 사실 | Canonical Owner | 다른 곳의 처리 |
|------|-----------------|----------------|
| 문서 계층 정의 (`docs/00-meta`, ...) | `docs/00-meta/TEMPLATE_GUIDE.md`의 "문서 계층" 섹션 | 다른 곳은 한 줄 + 링크. CLAUDE.md는 "문서 계층은 [TEMPLATE_GUIDE.md#문서-계층] 참조" 한 줄. boilerplate-context skill·README도 동일. |
| 위임 트리거 + 메인 세션 역할 | `docs/00-meta/AGENT_EXECUTION_STRATEGY.md` | CLAUDE.md는 "에이전트 실행 전략은 [AGENT_EXECUTION_STRATEGY.md] 참조" 한 줄. |
| 상태값 + 전이 규칙 (workitem 일반) | `docs/00-meta/WORKFLOW.md`의 "문서 상태 전이" | TEMPLATE_GUIDE·30-workitems/README는 정의 제거 + 링크. `_ADR_GUIDE.md`는 ADR 전용 상태값(`proposed`/`accepted`/`superseded`/`deprecated`)을 유지 — 의미가 다르므로 분리. |
| 워크플로우 단계 흐름 (한 줄 그림) | `docs/00-meta/WORKFLOW.md` (IMPROVE-LIST 항목 적용 후의 풀 흐름을 담는다) | README는 짧은 그림 1개만, 상세는 링크. AGENT_EXECUTION_STRATEGY는 위임 관점만, 흐름 전문은 링크. |
| Guardrail 원칙 | `docs/00-meta/GUARDRAILS_STRATEGY.md` | 다른 곳은 한 줄 + 링크. boilerplate-context skill도 한 줄로 줄임. |
| 새 프로젝트 시작 절차 (체크리스트) | `docs/00-meta/NEW_PROJECT_CHECKLIST.md` | TEMPLATE_GUIDE의 "새 프로젝트 시작 순서" 수동 흐름 제거 + "체크리스트는 [NEW_PROJECT_CHECKLIST.md]" 한 줄로. README는 Quick Start만. |
| Bootstrap 입력 예시 | `docs/00-meta/BOOTSTRAP_PROMPT_EXAMPLES.md` | README는 예시 1개만 + "더 많은 예시는 [BOOTSTRAP_PROMPT_EXAMPLES.md]" 링크. |
| 모델 별칭 정책 | (신규) `docs/90-decisions/ADR-00x-model-alias-policy.md` (항목 1) | 모든 frontmatter는 별칭만 박고 정책 설명 없음. 정책 설명은 ADR. |
| 단순성·YAGNI·Clean Code/Architecture 정책 | (신규) `docs/90-decisions/ADR-00x-simplicity-and-architecture.md` (항목 11) + CLAUDE.md(5줄 요약, fork된 세션 자동 로드 surface) | builder/reviewer/architect agent 본문은 ADR 링크 + 자기 영역 행동 규율(self-check 4항목 등)만. |
| TDD 정책 | (신규) `docs/90-decisions/ADR-00x-tdd-default.md` (항목 10) + CLAUDE.md(1줄) | implement-workitem skill, builder/validator agent는 ADR 링크. |
| Conventional Commits | (신규) `docs/90-decisions/ADR-00x-commit-convention.md` (항목 4) | finalize-workitem skill은 ADR 링크. |
| 워크아이템 라이프사이클 (discover→…→stabilize) | (신규) `docs/90-decisions/ADR-00x-workitem-lifecycle.md` (항목 4 권장) | WORKFLOW.md는 단계별 사용법, ADR은 "왜 이 단계 분리"의 근거. |
| 산출물 위치 인벤토리 | (신규) `docs/00-meta/STRUCTURE.md` (또는 TEMPLATE_GUIDE.md 부록 섹션) | 모든 산출물(charter, architecture, workitem, plan, report, qa, ADR, stack-setup-plan, discovery)을 한 표로. |
| ADR 인덱스 | (신규) `docs/90-decisions/README.md` | 모든 ADR을 번호·제목·상태·한 줄 요약으로. |

**도입 패턴 5종**

1. **"정의 → 링크" 패턴**
   canonical 문서가 정의를 담는다. 다른 문서는 "X는 [link]를 따른다" 한 줄만 둔다. 정의를 거기서 다시 풀지 않는다. 길어진다 싶으면 canonical을 갱신한다.

2. **"인덱스 README" 패턴**
   `docs/90-decisions/README.md`를 신설해 ADR 인덱스를 둔다. fork된 사용자가 디렉터리 ls 없이 ADR 전체를 한눈에 본다. 새 ADR 추가 시 이 README에 한 줄 추가하는 게 기본 절차로 `_ADR_GUIDE.md`에 명시한다.

3. **"산출물 인벤토리" 패턴**
   `docs/00-meta/STRUCTURE.md`(또는 TEMPLATE_GUIDE.md 섹션) 신설 — 모든 산출물의 위치, 생성 주체, 라이프사이클(Living/Reference/Record/ephemeral)을 단일 표로 관리. 신규 산출물(항목 5의 `STACK_SETUP_PLAN`, 항목 7의 `DISCOVERY`, 항목 3의 `reports/`) 추가 시 이 표 등록이 기본 절차.

4. **"정책 = ADR" 패턴**
   IMPROVE-LIST가 도입하는 모든 정책(모델 별칭, 단순성, TDD, commit convention, lifecycle)은 ADR로 박고, agent/skill 정의에는 정책 설명을 길게 박지 않는다. 정책 변경 시 ADR 한 곳만 갱신하면 모든 surface가 자동으로 따라온다 — 0.4 횡단 원칙 2(정책성 결정은 ADR 동반)와 직접 호환.

5. **"CLAUDE.md = 진입 페이지" 패턴**
   `CLAUDE.md`는 fork된 새 세션이 자동 로드하는 진입점이다. 모든 운영 원칙을 다 박지 않고 **목적·권위 있는 문서로의 링크 인덱스·핵심 행동 규율(단순성 5줄·TDD 1줄·민감 파일·범위 준수)**만 둔다. 깊은 정의는 `docs/00-meta`로 분리. 진입 페이지는 짧을수록 가치가 크다(현재 약 60줄 → 25~30줄 목표).

### 12.5 구현 스케치

**A. 신규 파일 (2개 + 선택 1)**

- 신규 `docs/90-decisions/README.md` — ADR 인덱스. 양식:
    ```markdown
    # ADR Index
    | # | 제목 | 상태 | 한 줄 요약 |
    |---|------|------|-----------|
    | 001 | Doc hierarchy | accepted | docs/ 디렉터리 6분할 결정 |
    | 002 | Initial project decisions | (placeholder) | bootstrap-project 단계의 초기 결정 모음 |
    | 003 | Stack selection | (placeholder, /bootstrap-stack 후 채워짐) | 스택 선택과 근거 |
    | ... | ... | ... | ... |
    ```
  새 ADR 추가 시 이 README에 행 추가가 기본 절차임을 `_ADR_GUIDE.md`에 명시.

- 신규 `docs/00-meta/STRUCTURE.md` — 산출물 인벤토리. 양식:
    ```markdown
    # 산출물 인벤토리
    | 산출물 | 위치 | 생성 주체 | 라이프사이클 |
    |--------|------|-----------|--------------|
    | charter | docs/10-charter/PROJECT_CHARTER.md | /bootstrap-project | Living |
    | architecture | docs/20-system/ARCHITECTURE_OVERVIEW.md | /bootstrap-project, /bootstrap-stack | Living |
    | discovery | docs/10-charter/DISCOVERY.md | /discover-product | Living |
    | milestone | docs/30-workitems/milestones/M*-*.md | /plan-workitem | Living |
    | feature | docs/30-workitems/features/F-*-*.md | /plan-workitem | Living |
    | task | docs/30-workitems/tasks/T-*-*.md | /plan-workitem, /implement-workitem | Living |
    | plan | docs/30-workitems/plans/ | Claude Code Plan mode | ephemeral |
    | validation report | docs/40-validation/reports/<task-id>.md | /validate-workitem | ephemeral |
    | qa findings | docs/40-validation/QA_FINDINGS.md | /stabilize-milestone | Record (마일스톤별 누적) |
    | improvement guide | docs/40-validation/IMPROVEMENT_GUIDE.md | /stabilize-milestone | Living |
    | ADR | docs/90-decisions/ADR-*.md (인덱스: README.md) | architect-opus, /bootstrap-project | Record |
    | stack setup plan | docs/00-meta/STACK_SETUP_PLAN.md | /bootstrap-stack, /stack-guard | Reference |
    ```
  TEMPLATE_GUIDE.md의 "문서 계층" 섹션에서 이 표로 링크.

- (선택) 신규 `docs/00-meta/_DOCUMENT_OWNERSHIP.md` — Canonical Owner 매핑 표(본 항목 12.4의 표를 보일러플레이트에 박는다). 새 정책 도입 시 이 표에 owner 등록이 절차. STRUCTURE.md 부록으로 합쳐도 된다.

**B. 슬림화 (정의 제거 + 링크)**

- 수정 `CLAUDE.md` — 다음 정의를 슬림화:
    - "문서 계층" → 한 줄 + `docs/00-meta/TEMPLATE_GUIDE.md#문서-계층` 링크.
    - "기본 작업 원칙"·"문서 작성 원칙" → 핵심 행동 규율(단순성 5줄·TDD 1줄·민감 파일·범위 준수)만 두고 깊은 절차는 `docs/00-meta/WORKFLOW.md` 링크.
    - "에이전트 실행 원칙"·"위임 트리거" → 한 줄 + `docs/00-meta/AGENT_EXECUTION_STRATEGY.md` 링크.
    - 단순성·YAGNI 5개 항목(항목 11a)·TDD 1줄(항목 10)은 진입 surface로서의 가치를 위해 CLAUDE.md에 그대로 둔다(예외 — 자동 로드 surface).
    - 결과: 약 60줄 → 25~30줄.

- 수정 `README.md`, `README_ko.md`:
    - "Quick Start"는 외부 사용자 진입점이므로 본문 유지.
    - "Overall Flow" 한 줄 그림 → `docs/00-meta/WORKFLOW.md` 링크 (그림은 1줄만).
    - "Structure" 디렉터리 트리 → `docs/00-meta/STRUCTURE.md` 링크.
    - "Guardrail Principles" → 한 줄 + 링크.
    - "Where to Start" 인덱스는 외부 진입 가치를 위해 유지.
    - 본문이 짧아지면 ko/en drift 표면도 줄어든다.
    - README 상단 코멘트로 "구조 변경은 두 README 동시 갱신 필수" 명시.

- 수정 `docs/00-meta/TEMPLATE_GUIDE.md`:
    - "새 프로젝트 시작 순서"의 수동 흐름(현재 1~6번)을 제거 + "체크리스트는 [NEW_PROJECT_CHECKLIST.md]" 한 줄로 대체 (B-2 모순 해소).
    - "상태값 예시"를 `WORKFLOW.md#문서-상태-전이` 링크로 대체.
    - "Guardrail 운영 원칙"을 한 줄 + 링크.
    - "산출물 인벤토리" 섹션 신설 (또는 STRUCTURE.md로 분리하고 여기는 링크만).

- 수정 `docs/30-workitems/README.md`:
    - "상태 관리 예시" 제거 + `WORKFLOW.md#문서-상태-전이` 링크.
    - "디렉터리 구성"의 "plans" 설명은 STRUCTURE.md 링크 + `.claude/settings.json`의 `plansDirectory`가 canonical임을 한 줄 명시.

- 수정 `.claude/skills/boilerplate-context/SKILL.md`:
    - "핵심 구조" 디렉터리 나열 → 한 줄 + `docs/00-meta/TEMPLATE_GUIDE.md` 링크.
    - "기본 원칙" → 한 줄 + `docs/00-meta/WORKFLOW.md` 링크.
    - "프로젝트 초기 세팅이 필요하면 ..." 안내는 actionable이므로 유지.
    - 끝에 "더 깊은 운영 원칙: [AGENT_EXECUTION_STRATEGY.md], [WORKFLOW.md]" 한 줄 추가 (C-1 보강).
    - 결과: 26줄 → 12~15줄.

- 수정 `docs/00-meta/WORKFLOW.md`:
    - 워크플로우 단계 흐름(한 줄 그림)을 IMPROVE-LIST 적용 후의 풀 흐름으로 갱신 (항목 5·6·7·10 등 적용 시).
    - "단계별 에이전트 위임"에 위임 트리거 표 정의가 다시 들어가지 않도록, AGENT_EXECUTION_STRATEGY.md 링크만 둔다.

- 수정 `.claude/agents/*.md` (5개):
    - 각 agent 본문의 "역할/규칙"은 행동 규율로 유지.
    - 정책성 내용(예: 단순성, TDD)은 ADR 링크로 줄이고 본인이 수행할 self-check만 본문에 명시.
    - 예: builder-sonnet — "단순성 self-check 4항목" 본문 + `ADR-00x-simplicity-and-architecture.md` 링크. TDD 행동 규율(`AC가 정의된 task는 RGR 사이클로 진행`) 본문 + `ADR-00x-tdd-default.md` 링크.

**C. 모순 해소**

- B-1: `CLAUDE.md`의 위임 트리거 표 자체를 제거하고 `AGENT_EXECUTION_STRATEGY.md` 링크로 대체. planner 누락은 자동 해소.
- B-2: `TEMPLATE_GUIDE.md`의 "새 프로젝트 시작 순서" 수동 흐름 제거 — `/bootstrap-project` 우선이 README/NEW_PROJECT_CHECKLIST와 일관됨.

**D. 끊김 보강**

- C-1: `boilerplate-context/SKILL.md` 끝에 깊은 운영 원칙 링크 한 줄.
- C-2: `docs/00-meta/GLOSSARY.md` 본문을 짧게 줄이고 "fork된 프로젝트에서 도메인 용어가 생기면 여기에 정의한다. 보일러플레이트 단계에선 의도적으로 비워둠" 한 단락으로 명시. orphan 상태를 의도적 비우기로 표시.
- C-3: 항목 2가 처리 (write-charter/write-architecture 삭제). SSOT 관점에서 합당.
- C-4: `docs/90-decisions/README.md` 신설 (위 A 신규 파일).
- C-5: `docs/00-meta/STRUCTURE.md` 신설 (위 A 신규 파일).
- 추가: `docs/30-workitems/_templates/TASK_TEMPLATE.md`·`FEATURE_TEMPLATE.md`의 "관련 문서" 섹션에 placeholder 양식 추가 — `[Milestone](../../milestones/M1-foundation.md)` 같은 상대 경로 예시 한 줄. 사용자가 어떻게 채울지 보이게.

**E. 본 IMPROVE-LIST 자체의 SSOT 정리**

본 IMPROVE-LIST의 섹션 14(체크리스트)는 항목별 본문에서 이미 적은 변경을 평면 나열한다 — 적용 PR이 체크리스트 갱신을 잊으면 drift. 본 IMPROVE-LIST는 적용 후 폐기되는 임시 문서이므로 자동화는 불필요. 다만 다음 운영 원칙을 0.4 횡단 원칙에 추가한다:
- "각 항목 본문(섹션 1~12)이 SSOT다. 섹션 14 체크리스트는 인덱스 역할이며, 본문과 충돌하면 본문이 권위다."

### 12.6 영향 범위

- 신규: 2 문서 (`docs/90-decisions/README.md`, `docs/00-meta/STRUCTURE.md`), 선택 1 (`_DOCUMENT_OWNERSHIP.md`)
- 수정: `CLAUDE.md`, `README.md`, `README_ko.md`, `docs/00-meta/{WORKFLOW,TEMPLATE_GUIDE,GUARDRAILS_STRATEGY,NEW_PROJECT_CHECKLIST,GLOSSARY}.md`, `docs/30-workitems/README.md`, `docs/30-workitems/_templates/{TASK_TEMPLATE,FEATURE_TEMPLATE}.md`, `.claude/skills/boilerplate-context/SKILL.md`, 모든 `.claude/agents/*.md`(5개, 정책 설명 → ADR 링크)
- **본 항목은 다른 항목 적용의 전제 인프라다.** 항목 1(모델 별칭)·항목 2(약한 skill 정리) 적용 후 즉시 본 항목을 적용한다. 그 위에 항목 3~11의 신규 ADR·산출물·정책이 SSOT 패턴으로 박혀야 향후 drift가 생기지 않는다.
- 의존: 항목 1·항목 2 (선행). 항목 12 적용 후 항목 3~11이 신규로 만드는 ADR·산출물·정책이 모두 SSOT 패턴을 따라간다.

### 12.7 열린 질문

- **TEMPLATE_GUIDE.md vs STRUCTURE.md 분리 여부** — 두 문서로 쪼개면 산출물 인벤토리가 깔끔하지만 cross-ref가 늘어난다. 잠정: 인벤토리 표가 길어지면 분리, 짧으면 TEMPLATE_GUIDE.md 안에. 적용 시점에 결정.
- **CLAUDE.md를 어디까지 슬림화할지** — 너무 짧으면 fork된 새 세션이 핵심 컨텍스트를 못 받는다. 잠정: 단순성 5개 항목·TDD 1줄·민감 파일·범위 준수는 본문에 유지. 그 외 정의는 모두 링크. 60줄 → 25~30줄 목표.
- **README의 본문 vs 링크 비율** — README는 외부 진입점이므로 너무 짧아지면 매력도가 떨어진다. 잠정: Quick Start + 핵심 가치는 유지, 깊은 운영 정의는 링크.
- **agent 본문에 행동 규율을 박는 것 vs ADR 링크만 두는 것** — SSOT 원칙으론 ADR 링크만이 맞지만 agent는 매 호출마다 시스템 프롬프트가 로드되므로 self-check 4항목 같은 **행동 규율은 본문에 두는 편이 안정적**. 잠정: 행동 규율(self-check, 출력 형식)은 본문, 정책 근거(왜 단순성이 1순위인가)는 ADR 링크.
- **ADR 인덱스 자동 생성 vs 수동** — 자동: stack-agnostic 원칙 위반(스크립트 런타임 결정 필요). 수동: 새 ADR 추가 시 README 갱신 잊힘. 잠정: 수동 + `_ADR_GUIDE.md`에 "새 ADR 추가 시 README.md 행 추가" 절차 명시. drift 발생 시 `/stabilize-milestone`(항목 6)이 점검 항목으로 추가하는 후속 항목 검토.
- **"정의 1곳" 원칙이 정말 항상 옳은가?** — 일부 사실(예: 워크플로우 한 줄 그림)은 README와 WORKFLOW 두 곳에 있는 게 사용자 경험 측면에서 자연스러울 수 있다. 잠정: 정의는 1곳, **요약된 그림은 다중 위치 허용** — 다만 요약끼리 모순되지 않도록 변경 시점에 함께 갱신.

---

## 13. 통합 흐름 (적용 후 그림)

> 아래 화살표는 **자동 호출이 아니라 제안 → 발화** 흐름이다(0.4 횡단 원칙 3 참조). 각 skill은 다음 액션을 텍스트로 제안하고, 사용자 또는 메인 세션이 받아 다음 skill을 발화한다.

```
/discover-product (메인 세션이 R0~R4 라운드 운전, 산출물은 docs/10-charter/DISCOVERY.md)
    ↓ 다음 액션 제안: /bootstrap-project
/bootstrap-project (DISCOVERY.md를 입력으로 charter/architecture/M1/F-001 생성·갱신)
    ↓ 다음 액션 제안: /bootstrap-stack
/bootstrap-stack (스택 확정 + 문서 갱신)
    ↓ 다음 액션 제안: /stack-guard
/stack-guard (R0=운영 환경 가정 확인 → 통합 `validate` 명령·검증 스크립트 생성. PostToolUse hook 자동 등록은 본 1단계 제외)
    ↓
/plan-workitem (planner agent + fork) — milestone/feature/task 분해
    │      task 문서에 AC-1, AC-2 ... 형식으로 Acceptance Criteria 명시 (항목 10)
    ↓
/implement-workitem (builder-sonnet)
    │      Red(실패 테스트) → Green(최소 구현) → Refactor(Clean Code 6항목 정리) 사이클
    │      각 phase 종료 후 단순성 self-check 4항목 (항목 11)
    │      모든 AC가 소진될 때까지 반복. 또는 --fast로 첫 AC만.
    ↓
/validate-workitem (validator-sonnet, 판정 + report 기록; 통합 `validate` 명령 자동 실행)
    │      추가 점검: AC ↔ 테스트 매핑, 테스트 선행 휴리스틱 (항목 10)
    │      → docs/40-validation/reports/<task-id>.md  (AC-N별 ✅/❌ 기록)
    ├─ Pass (AC 미충족 0개) → /finalize-workitem 안내 (status=done + 명시적 파일 add + 커밋)
    └─ Needs Fix → /repair-workitem 안내 (report의 실패 항목만 수정, 자동 커밋 없음)
                      → /validate-workitem 재실행 안내
                                 ↑
                                 └ 무한 루프 가드(연속 3회 시 사용자 확인)

마일스톤의 모든 task가 done이 되면:
/stabilize-milestone (E2E + 회귀 + 리팩토링 후보 + ADR 점검; 코드 수정·커밋 없음)
    │      reviewer에 Clean Code 6항목 체크리스트 입력 (항목 11)
    │      ARCHITECTURE_OVERVIEW의 ## 3-1이 비어있고 모듈 ≥3개면 채울 것을 권장 출력
    ↓
다음 마일스톤으로 진행 또는 후속 task로 큐잉
```

병렬 패턴(고급, 선택)은 위 흐름과 직교한다 — 항목 9 참조.

횡단 가드레일(상시 적용, 흐름 직교):
- (항목 11a) `CLAUDE.md`의 단순성·YAGNI 정책은 모든 세션이 본다.
- (항목 11a) builder-sonnet은 출력 직전 단순성 self-check 4항목.
- (항목 11a) reviewer는 호출될 때마다 Clean Code 6항목 체크리스트로 라벨링.
- (항목 12) 모든 정책은 ADR 1곳에 정의되고 agent/skill 본문은 행동 규율 + ADR 링크만. 신규 산출물은 `docs/00-meta/STRUCTURE.md`에 등록, 신규 ADR은 `docs/90-decisions/README.md` 인덱스에 행 추가.

---

## 14. 문서 반영 체크리스트 (적용 시)

> 항목 12 적용 후의 운영 원칙 — 본 체크리스트는 **인덱스 역할**이다. 정의는 각 항목 본문(섹션 1~12)이 SSOT다. 본문과 충돌 시 본문이 권위. 새 변경을 추가할 때는 항목 본문을 먼저 갱신하고 이 체크리스트에 한 줄을 추가한다.

**문서 / 흐름 갱신**
- [ ] `README.md`, `README_ko.md` — 권장 흐름을 `/discover-product → /bootstrap-project → /bootstrap-stack → /stack-guard → ...`으로 갱신, `/batch` 사용 가이드(bundled 명시) 추가, mid-project 갱신 동선 한 단락(항목 2). **본문 슬림화** — 흐름·구조·guardrail 정의를 canonical 문서 링크로 줄이고, 두 README 동시 갱신 정책 코멘트를 상단에 명시 (항목 12)
- [ ] `CLAUDE.md` — 위임 트리거 표에서 병렬 패턴 3종으로 분리 (항목 9), "코드 및 변경 원칙"에 단순성·YAGNI 5개 항목 추가 (항목 11a), TDD 사이클을 디폴트로 따른다 단락 추가 (항목 10). **슬림화** — 문서 계층·위임 트리거·운영 원칙 정의를 모두 `docs/00-meta/*` 링크로 줄이고 진입 surface로서의 행동 규율만 유지 (항목 12, 60줄 → 25~30줄 목표)
- [ ] `docs/00-meta/WORKFLOW.md` — `/discover-product`, `/finalize-workitem`, `/repair-workitem`, `/stabilize-milestone` 단계 추가, charter/architecture mid-project 갱신 동선 단락 추가. **canonical 역할 강화** — 워크플로우 단계 흐름 + 문서 상태 전이 규칙의 SSOT (항목 12)
- [ ] `docs/00-meta/AGENT_EXECUTION_STRATEGY.md` — 위임 트리거 표 갱신, 병렬 패턴 3종 단락 추가, 모델 표기 정책 단락 추가 (항목 1·9). **canonical 역할 강화** — 위임 트리거의 SSOT (항목 12)
- [ ] `docs/00-meta/TEMPLATE_GUIDE.md` — "새 프로젝트 시작 순서" 수동 흐름 제거 + `NEW_PROJECT_CHECKLIST.md` 링크, 상태값 정의 제거 + `WORKFLOW.md#문서-상태-전이` 링크, 산출물 인벤토리 섹션 신설 (또는 `STRUCTURE.md` 링크) (항목 12)
- [ ] `docs/00-meta/NEW_PROJECT_CHECKLIST.md` — discovery 단계 분리 반영. **canonical 역할 강화** — 새 프로젝트 시작 절차의 SSOT (항목 12)
- [ ] `docs/00-meta/GUARDRAILS_STRATEGY.md` — `/stack-guard` 1단계 산출물 범위 명시, PostToolUse hook 자동 등록은 prototyping 후 별도 항목으로 분리 명시. **canonical 역할 강화** — guardrail 원칙의 SSOT (항목 12)
- [ ] `docs/00-meta/STRUCTURE.md` 신규 — 모든 산출물의 위치·생성 주체·라이프사이클(Living/Reference/Record/ephemeral) 단일 인벤토리 표 (항목 12)
- [ ] `docs/00-meta/GLOSSARY.md` — 본문 짧게 재구성, "보일러플레이트 단계엔 의도적으로 비워둠. fork된 프로젝트의 도메인 용어가 생기면 여기에 정의" 한 단락 (항목 12, C-2 보강)
- [ ] `docs/30-workitems/README.md` — "상태 관리 예시" 제거 + `WORKFLOW.md#문서-상태-전이` 링크, plans 설명에 `.claude/settings.json`의 `plansDirectory`가 canonical임을 한 줄 명시 (항목 12)
- [ ] `docs/30-workitems/_templates/TASK_TEMPLATE.md`·`FEATURE_TEMPLATE.md` "관련 문서" 섹션에 상대 경로 placeholder 양식 추가 (항목 12, 끊김 보강)
- [ ] `docs/90-decisions/README.md` 신규 — ADR 인덱스(번호·제목·상태·한 줄 요약) (항목 12)
- [ ] `docs/90-decisions/_ADR_GUIDE.md` — "새 ADR 추가 시 README.md 행 추가" 절차 한 줄 추가 (항목 12)
- [ ] `docs/10-charter/PROJECT_CHARTER.md` (template) — 페르소나/시나리오/가정 섹션 추가
- [ ] `docs/10-charter/_templates/DISCOVERY_TEMPLATE.md` 신규 (항목 7)
- [ ] `docs/30-workitems/_templates/TASK_TEMPLATE.md` — `## 4-1. 변경 예정 파일/경로` 섹션 추가 (항목 4), `## 6. 테스트 포인트` → `## 6. Acceptance Criteria` 교체 + `## 6-1. 테스트 시나리오 (TDD Red)` + `## 6-2. TDD opt-out` 신설 (항목 10)
- [ ] `docs/30-workitems/_templates/FEATURE_TEMPLATE.md` — `## 8. 검증 방법`에 "task 단위 AC로 분해된다" 한 줄 추가 (항목 10)
- [ ] `docs/20-system/ARCHITECTURE_OVERVIEW.md` (template) — `## 3-1. 레이어 경계 + 의존성 규칙` 섹션 신설, "프로젝트 규모가 정당화될 때만 채운다" YAGNI 보호 단서 동봉 (항목 11b)
- [ ] `docs/40-validation/{QA_FINDINGS,IMPROVEMENT_GUIDE}.md` — 마일스톤 단위 누적 양식 (항목 6)
- [ ] `docs/40-validation/reports/.gitkeep` 신규 (항목 3)

**agents / skills**
- [ ] `.claude/agents/validator-sonnet.md` — "판정 + report 기록 전용" 명시, tools에 `Write` 추가, report 파일 저장 규칙 (항목 3·4), AC↔테스트 매핑 점검 규칙 (항목 10), 범위 밖 추상화·premature factory 표시 (항목 11a). **정책 설명 → ADR 링크** (항목 12)
- [ ] `.claude/agents/builder-sonnet.md` — finalize 위임 시 가드, 구현 후 task 문서 `## 4-1` 갱신 규칙 (항목 4), Red→Green→Refactor 사이클 + AC별 진행 상태 출력 (항목 10), 단순성 self-check 4항목 (항목 11a). **정책 설명 → ADR 링크** (항목 12)
- [ ] `.claude/agents/architect-opus.md` — `model: opus` 별칭 (항목 1), "프로젝트 규모가 4-layer를 정당화하는가" self-check (항목 11a). **정책 설명 → ADR 링크** (항목 12)
- [ ] `.claude/agents/reviewer.md` — Clean Code 6항목 체크리스트(naming/size/duplication/premature abstraction/comment policy/layer leak), P0/P1/P2 + 항목 라벨링 (항목 11a). **정책 설명 → ADR 링크** (항목 12)
- [ ] `.claude/agents/planner.md`, `.claude/agents/qa.md` — **정책 설명 → ADR 링크** 일관 적용 (항목 12)
- [ ] `.claude/skills/boilerplate-context/SKILL.md` — 디렉터리 나열·기본 원칙을 한 줄 + 링크로 슬림화, 깊은 운영 원칙 링크 한 줄 추가 (항목 12)
- [ ] `.claude/skills/bootstrap-project/SKILL.md` — `model` 라인 처리(항목 1·8 결합), 입력 우선순위 갱신(항목 8)
- [ ] `.claude/skills/bootstrap-stack/SKILL.md` — `model` 라인 처리(항목 1), 다음 액션으로 `/stack-guard` 안내 (항목 5)
- [ ] `.claude/skills/implement-workitem/SKILL.md` — Red→Green→Refactor 3 phase로 본문 분리, opt-out 흐름, `--fast` 인자 (항목 10), Refactor phase에 단순성 self-check + Clean Code 6항목 참조 안내 (항목 11a)
- [ ] `.claude/skills/validate-workitem/SKILL.md` — AC↔테스트 매핑 점검·테스트 선행 휴리스틱 추가 (항목 10)
- [ ] `.claude/skills/finalize-workitem/SKILL.md` (신설, 항목 4) — 통과 조건에 "AC 미충족 항목 0개" 추가 (항목 10)
- [ ] `.claude/skills/stabilize-milestone/SKILL.md` (신설, 항목 6) — reviewer 입력에 Clean Code 6항목 명시, ADR 후보에 layer/의존성 규칙 변경 추가, ARCHITECTURE_OVERVIEW `## 3-1` 자동 채움 권장 단계 (항목 11b)
- [ ] `.claude/settings.json` — `"model": "sonnet"` 별칭 (항목 1). PostToolUse hook 자동 등록은 prototyping 후.

**ADR (정책성 결정)**
- [ ] `docs/90-decisions/ADR-00x-model-alias-policy.md` (항목 1) — 모델 별칭 우선 정책
- [ ] `docs/90-decisions/ADR-00x-commit-convention.md` (항목 4) — Conventional Commits 기본 채택
- [ ] (선택) `docs/90-decisions/ADR-00x-workitem-lifecycle.md` — discover→bootstrap→plan→implement→validate→repair→finalize→stabilize 라이프사이클 정의
- [ ] `docs/90-decisions/ADR-00x-tdd-default.md` (항목 10) — TDD를 디폴트로 채택, opt-out 절차 정의
- [ ] `docs/90-decisions/ADR-00x-simplicity-and-architecture.md` (항목 11) — 단순성 1순위, Clean Code 2순위, Clean Architecture 3순위 정책
- [ ] (선택) `docs/90-decisions/ADR-00x-ssot.md` (항목 12) — 단일 출처 원칙·canonical owner 매핑·"정책=ADR" 패턴 결정

**.gitignore**
- [ ] `docs/40-validation/reports/*.md` 추가 (항목 3, ephemeral 검증 산출물). `.gitkeep`은 트래킹.

---

## 15. 참고 — 공식 문서 조사 결과 (요약)

> 2026-04-28 시점 [code.claude.com/docs](https://code.claude.com/docs/en) 1차 출처 기반. 실제 구현 직전에 최신 문서로 한 번 더 확인 권장.

### 15.1 Skill frontmatter (공식 확인)
출처: [Skills 문서](https://code.claude.com/docs/en/skills) / [Permissions 문서](https://code.claude.com/docs/en/permissions) / [Commands 문서](https://code.claude.com/docs/en/commands)
- 지원 필드: `name`, `description`, `when_to_use`, `argument-hint`, `arguments`, `disable-model-invocation`, `user-invocable`, `allowed-tools`, `model`, `effort`, `context`, `agent`, `hooks`, `paths`, `shell`
- `context: fork` — fork된 sub-agent 컨텍스트로 실행. `agent` 필드로 사용할 sub-agent 지정. **공식 경고**: skill 본문이 단순 가이드라인이면 sub-agent가 actionable prompt 없이 받아 무의미한 결과 반환. 본 문서가 `context: fork`로 올리는 모든 skill(`plan-workitem`, `review-doc` 등 항목 2)은 본문이 명령형 절차여야 한다.
- `allowed-tools`의 Bash 패턴(공식 [permissions 문서](https://code.claude.com/docs/en/permissions)):
    - 정확 일치 — `Bash(npm run build)` 같은 와일드카드 없는 형식이 가장 기본. 공식 예시로 등재.
    - 공백 + 별표 — `Bash(git add *)`, `Bash(git commit *)` 등. wildcard는 끝 외에도 어느 위치든 가능(`Bash(* install)`, `Bash(git * main)`).
    - 콜론 + 별표(trailing 한정) — `Bash(ls:*)`는 `Bash(ls *)`와 **동등**(공식: "`Bash(ls:*)` matches the same commands as `Bash(ls *)`"). 단, trailing 위치에서만 인식 — `Bash(git:* push)`처럼 중간에 쓰면 콜론이 리터럴로 처리되어 매칭 실패.
    - 권한 다이얼로그가 "Yes, don't ask again"으로 저장하는 형식은 공백 형식.
- bundled skill: `/simplify`, `/batch`, `/debug`, `/loop`, `/claude-api`, `/init`, `/review`, `/security-review` 등이 기본 제공(commands 문서에 `[Skill]` 표시). `/batch`가 bundled라는 사실이 항목 9와 직접 관련.

### 15.2 Sub-agent frontmatter (공식 확인)
출처: [Sub-agents 문서](https://code.claude.com/docs/en/sub-agents)
- 지원 필드(file-based markdown 정의): `name`, `description`, `tools`, `disallowedTools`, `model`, `permissionMode`, `maxTurns`, `skills`, `mcpServers`, `hooks`, `memory`, `background`, `effort`, `isolation`, `color`, `initialPrompt`
- **두 가지 정의 형식** — sub-agent는 두 인터페이스로 정의 가능하며 같은 sub-agent를 표현하는 등가 형식이다:
    - **file-based markdown** (`.claude/agents/<name>.md`) — frontmatter + markdown body. body가 system prompt로 사용된다. 본 보일러플레이트의 모든 sub-agent가 이 형식.
    - **CLI / SDK JSON 정의** — JSON 객체로 정의하며 system prompt를 별도의 `prompt` 필드로 넘긴다. file-based의 markdown body와 동일 역할.
- `isolation: worktree` — sub-agent를 임시 git worktree에서 실행. 변경이 없으면 자동 cleanup, 변경이 있으면 worktree 경로와 브랜치를 결과에 포함(공식 동작).
- `model` 필드는 `sonnet`/`opus`/`haiku` 별칭, 전체 ID(`claude-opus-4-7` 같은), 또는 `inherit` 허용. 별칭 매핑은 15.4 참조.
- `description`의 "proactively"는 관례일 뿐 공식 키워드 효과는 별도 명시되어 있지 않다(공식은 "trigger 조건 + Use when ..." 형식의 구체성 권장). `architect-opus.md` 등이 사용하는 "Use proactively for ..." 형식은 트리거 조건이 명확하므로 그대로 유지해도 무방.

### 15.3 Hooks (공식 확인)
출처: [Hooks 문서](https://code.claude.com/docs/en/hooks)
- 이벤트: `SessionStart`/`SessionEnd`, `UserPromptSubmit`/`UserPromptExpansion`, `PreToolUse`/`PostToolUse`/`PostToolUseFailure`/`PostToolBatch`, `Stop`/`StopFailure`/`SubagentStart`/`SubagentStop`, `PermissionRequest`/`PermissionDenied` 등
- `.claude/settings.json`의 `hooks` 필드에 matcher + 명령으로 등록. exit code 2로 즉시 차단. skill·sub-agent frontmatter의 `hooks` 필드도 공식 지원(스코프 격리).
- 입출력 JSON 스키마는 공통 필드와 출력(`continue`, `stopReason`, `suppressOutput`, `systemMessage`)이 공식 문서에 정의. 이벤트별 추가 필드는 [hooks-reference](https://code.claude.com/docs/en/hooks-reference) 참조.

### 15.4 모델 별칭 (공식 확인)
출처: [Model Config 문서](https://code.claude.com/docs/en/model-config)
- `sonnet` / `opus` / `haiku` 별칭은 자동 최신 매핑. settings.json의 `"model": "opus"` 같은 표기 권장. 특정 버전 핀이 필요할 때만 전체 ID(`claude-opus-4-7`).

### 15.5 AskUserQuestion (공식 확인)
출처: [Agent SDK user-input 문서](https://code.claude.com/docs/en/agent-sdk/user-input)
- Claude가 다중 선택지(2~4개 옵션, 1~4개 질문)로 사용자에게 묻는 공식 도구. 메인 세션에서는 기본 가용. 자유 입력은 호출자 측 UI에서 "Other" 보조 옵션으로 구현하는 패턴이며 도구 자체가 자유 입력을 받는 것은 아니다.
- **컨텍스트 주의** — 이 도구는 Claude **Agent SDK** 컨텍스트에서 정의되어 있다. SDK 호출자가 `canUseTool` 콜백으로 질문을 가로채 사용자에게 표시하고 응답을 다시 SDK로 돌려주는 메커니즘. Claude Code CLI를 직접 사용하는 보일러플레이트 사용자에게 이 메커니즘이 그대로 노출되는 형태가 아닐 수 있어, 본 문서 항목 7은 자연어 응답을 채택한다.
- **공식 제약 (Limitations 섹션)** — `AskUserQuestion is not currently available in subagents spawned via the Agent tool`. 즉 sub-agent의 `tools` 필드를 어떻게 두든 무관하게 모든 sub-agent에서 사용 불가. 본 보일러플레이트가 fork된 skill에서 사용자에게 직접 묻는 흐름을 만들 수 없다는 뜻이며, 항목 7이 `/discover-product`를 메인 세션 운전으로 두는 결정의 보강 근거.
- 본 문서 항목 7은 보일러플레이트 단계에서는 자연어 응답으로 충분하다는 design 결정. 향후 인터랙티브 UX 개선 시 별도 항목으로 채택 검토.

### 15.6 보일러플레이트 표준 패턴 (공식 미정)
"persona → pain → JTBD → 시나리오" 라운드(항목 7), 워크아이템 라이프사이클(discover→bootstrap→plan→implement→validate→repair→finalize→stabilize, 항목 3~6), 검증 결과 파일 매개 전달(항목 3) 등은 공식 표준이 없으며 본 보일러플레이트가 제안하는 권장 패턴이다. 공식 표준이 등장하면 그 시점에 본 문서를 갱신한다.

---

## 16. 재검토 메모

### 16.1 분류 기준
본 문서 항목들은 다섯 층으로 나뉜다.

- **staleness 정리 (P0)** — 지금 동작은 하지만 모델 버전 갱신 등으로 가까운 시점에 fork된 새 프로젝트가 noise를 만들 위험. 항목 1(모델 별칭 — `claude-opus-4-6`이 1버전 뒤)이 이에 해당하며 적용 순서 최우선. 즉시 실패는 아니므로 표현은 보수적으로.
- **표현·가이드 정리 (P1·P2)** — 즉시 동작 실패는 아니지만 사용자 혼선을 키우는 항목. 항목 2(약한 skill), 항목 9(`/batch`).
- **품질 향상 (P0~P2)** — 보일러플레이트 가치(워크아이템 라이프사이클 일관 마감, 인터랙티브 발굴)를 끌어올리는 항목. 항목 3·4·5·6·7·8.
- **에이전트 작업 규율 (P1)** — fork된 미래 프로젝트의 에이전트가 "요구한 것만 단순하게" 작업하도록 만드는 항목. 항목 10(TDD), 항목 11(단순성·Clean Code·Clean Architecture). 사용자가 핵심 가치로 강조한 "오버엔지니어링 회피·가독성 우선"을 fork-time surfaces(`CLAUDE.md`, `.claude/agents/*`, 템플릿)에 박는다.
- **인프라 / 메타 구조 (P0)** — 다른 항목들이 위에서 안전하게 동작하기 위한 기반. 항목 12(SSOT). 항목 3~11이 만드는 신규 ADR·산출물·정책의 전제 인프라. 본 항목 적용 후 모든 신규 작성은 SSOT 패턴(canonical owner 매핑, 정책=ADR, 산출물 인벤토리 등록, ADR 인덱스 갱신)을 따라간다.

### 16.2 이번 재검토 사이클에서 제외/삭제한 후보
- **`.claude/settings.local.json` 저장소 모순(원 항목 11)** — `.gitignore`가 제외하고 있어 git 트래킹되지 않음. "저장소에 들어 있다"는 전제가 사실이 아니므로 삭제.
- **템플릿 자기검증 스크립트(원 항목 13)** — 유일 근거가 위 settings.local 모순. 자기검증 스크립트의 런타임 결정이 GUARDRAILS_STRATEGY의 stack-agnostic 원칙과 충돌. 별도 항목으로 두지 않고 `/stack-guard` 산출물에 흡수 여지 남김.
- **slash command 실행 방식 불일치(원 항목 12)** — 항목 2와 동일. 흡수.
- **README 이중 언어 운영 개선** — 드리프트 위험은 있으나 즉시 기능 오작동은 아니라 후순위.
- **문서 중복 축소** — 유지보수성 이슈는 맞지만 핵심 결함보다 후순위.
