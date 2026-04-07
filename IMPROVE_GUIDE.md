# 보일러플레이트 개선 실행 가이드

> 기준일: 2026-04-07
> IMPROVE_GUIDE.md의 항목들을 실제 코드와 대조 검증한 뒤, 실행 가능한 수정 흐름으로 재구성한 문서다.
> 이 문서를 위에서 아래로 따라가면 모든 개선이 완료된다.

---

## 전체 흐름

이 가이드는 5단계로 구성되어 있다.

| 단계 | 주제 | 핵심 이유 |
|------|------|-----------|
| 1단계 | 원칙 분산 정리와 문서 역할 분리 | 같은 내용이 4곳에 흩어져 있어서, 이후 수정이 전부 연쇄됨. 가장 먼저 해야 함 |
| 2단계 | 설정과 문서의 불일치 해소 | `plansDirectory` 등 실제 구조와 문서가 어긋남 |
| 3단계 | 문서 운영 규칙 정비 | Living Docs/Records 구분, 상태 전이, 충돌 해결, ADR 가이드, DoD |
| 4단계 | 템플릿 보강 | NFR 섹션, 비UI 대응, 스켈레톤 힌트 |
| 5단계 | 에이전트 설정 개선 | description 통일, 턴 예산 종료 규칙, 스킬 체이닝 안내 |

---

## 1단계: 원칙 분산 정리와 문서 역할 분리

> 관련 항목: IG-017, IG-018, IG-021

### 왜 가장 먼저 해야 하는가

이 보일러플레이트의 핵심 원칙이 "흩어진 임시 메모보다 정해진 위치의 문서를 갱신한다"인데, subagent-first 원칙 자체가 이 원칙을 위반하고 있다. 현재 동일한 에이전트 실행 원칙이 **4곳에 분산**되어 있다:

1. `CLAUDE.md` — 에이전트 실행 원칙 (6줄)
2. `docs/00-meta/AGENT_EXECUTION_STRATEGY.md` — 전체 전략 (58줄)
3. `docs/00-meta/NEW_PROJECT_CHECKLIST.md` — 실행 원칙 (3줄)
4. `README.md` — Subagent-first execution examples (20줄 이상)

또한 `WORKFLOW.md`에 "단계별 추천 에이전트" 표가 있고, `AGENT_EXECUTION_STRATEGY.md`에도 "권장 역할 분담" 표가 있어서 에이전트 추천 정보가 중복된다.

이걸 먼저 정리하지 않으면, 이후 모든 수정이 4곳에서 연쇄적으로 불일치를 만든다.

---

### 수정 1-1. CLAUDE.md 에이전트 실행 원칙 축소

**현재 내용** (`CLAUDE.md` 46~51행):

```markdown
## 에이전트 실행 원칙
- 메인 세션은 가능한 한 오케스트레이션에 집중한다.
- 탐색, 구현, 검증, QA는 관련 서브에이전트에 우선 위임한다.
- 메인 세션에는 현재 목표, 최근 결정, 다음 액션, 핵심 리스크만 유지한다.
- 긴 탐색 결과나 로그는 메인에 오래 보존하지 않는다.
- 중요한 기획/설계는 `architect-opus`를 우선 사용한다.
- 큰 독립 작업은 `/batch` 또는 worktree 기반 병렬 실행을 우선 고려한다.
```

**수정할 내용**:

핵심 원칙만 남기고, 위임 트리거 조건을 추가하고, 상세는 AGENT_EXECUTION_STRATEGY.md로 링크한다.

```markdown
## 에이전트 실행 원칙
- 메인 세션은 오케스트레이션에 집중하고, 실작업은 서브에이전트에 우선 위임한다.
- 메인 세션에는 현재 목표, 최근 결정, 다음 액션, 핵심 리스크만 유지한다.

위임 트리거:
- task 문서가 존재하는 구현 작업 → `builder-sonnet`
- 구현 완료 후 범위 검증 → `validator-sonnet`
- 중요한 설계 변경, 큰 tradeoff → `architect-opus`
- 문서/코드의 모순·누락 검토 → `reviewer`
- 구현 후 회귀 위험·엣지 케이스 점검 → `qa`
- 독립적인 여러 task 동시 처리 → `/batch` 또는 worktree

상세 전략은 [AGENT_EXECUTION_STRATEGY.md](docs/00-meta/AGENT_EXECUTION_STRATEGY.md)를 참조한다.
```

**변경 이유**: 현재 6줄은 "우선 위임한다" 수준의 선언인데, "어떤 상황에서 누구에게"가 없어서 실효성이 약하다. 위임 트리거를 명시하면 에이전트의 위임 판단 기준이 명확해진다 (단, description과 context를 종합적으로 보고 판단하므로 트리거가 자동 위임을 보장하지는 않는다). 동시에 장문의 전략 설명을 CLAUDE.md에 두지 않고 전용 문서로 분리한다.

---

### 수정 1-2. AGENT_EXECUTION_STRATEGY.md를 단일 source of truth로 강화

**현재 내용** (`docs/00-meta/AGENT_EXECUTION_STRATEGY.md`):

에이전트 전략 문서인데, 위임 조건이 구체적이지 않고 역할 분담 표도 에이전트명 나열 수준이다.

**수정할 내용**:

기존 구조를 유지하면서 아래 내용을 보강한다.

1. "권장 역할 분담" 섹션을 **위임 트리거 표**로 교체한다:

현재:
```markdown
## 권장 역할 분담
- architect-opus: 중요한 기획, 아키텍처, 기술 선택, milestone 분해
- planner: 일상적인 요구사항 정리, workitem 분해
- builder-sonnet: task 단위 구현, 테스트 추가, 국소 리팩토링
- validator-sonnet: 구현 결과가 문서 범위와 일치하는지 검증
- reviewer: 문서/코드 비판 리뷰
- qa: 사용자 영향, 엣지 케이스, 회귀 위험 점검
```

수정 후:
```markdown
## 위임 트리거

| 상황 | 우선 위임 대상 | 비고 |
|------|---------------|------|
| task 문서가 존재하는 구현 작업 | builder-sonnet | 범위 밖 변경 금지 |
| 구현 완료 후 범위 검증 | validator-sonnet | diff 기반 검증 |
| 중요한 설계 변경, 큰 tradeoff, 상위 아키텍처 수정 | architect-opus | 비용이 크므로 일상 작업에는 사용하지 않음 |
| 요구사항 정리, workitem 분해 | planner | 아키텍처 결정은 architect-opus로 |
| 문서/코드의 모순·누락·숨은 복잡도 검토 | reviewer | |
| 구현 후 회귀 위험·엣지 케이스 점검 | qa | |
| 독립적인 여러 task 동시 처리 | /batch + worktree | 같은 파일 변경은 병렬화하지 않음 |
| 장문 코드/문서 탐색 | Explore 등 built-in subagent | 선택적 사용. 메인 컨텍스트 오염 방지 |
```

2. "중요 원칙" 섹션 끝에 **스킬 체이닝 흐름**을 추가한다:

```markdown
## 스킬 실행 순서 가이드

일반적인 프로젝트 진행에서의 추천 스킬 순서:

1. `/bootstrap-project` → charter + architecture + 초기 workitem 생성
2. `/bootstrap-stack` → 스택 확정 후 자동화 설계
3. `/plan-workitem` 또는 planner → workitem 분해
4. `/implement-workitem` → task 구현
5. `/validate-workitem` → 구현 검증
6. qa agent → 회귀 위험 점검

각 단계에서 중요한 설계 판단이 필요하면 architect-opus를 먼저 사용한다.
문서 품질이 걱정되면 `/review-doc` 또는 reviewer를 사이에 끼운다.
```

---

### 수정 1-3. WORKFLOW.md에서 에이전트 표 제거

**현재 내용** (`docs/00-meta/WORKFLOW.md` 33~44행):

```markdown
## 단계별 추천 에이전트

| 단계 | 주요 작업 | 추천 에이전트/명령 |
|------|-----------|-------------------|
| 프로젝트 초기화 | charter + architecture 초안 생성 | `/bootstrap-project` |
| 중요한 기획/설계 | 제품 방향, 아키텍처, 큰 tradeoff | `architect-opus` |
| 일반 문서 정리 | 요구사항 정제, workitem 분해 | `planner` |
| 구현 | task 단위 코드 변경, 테스트 추가 | `builder-sonnet` |
| 구현 검증 | 문서-구현 일치, 범위 점검 | `validator-sonnet` |
| 비판 리뷰 | 문서/코드의 누락, 모순 검토 | `reviewer` |
| QA | 엣지 케이스, 회귀 위험, 사용자 영향 점검 | `qa` |
| 큰 병렬 변경 | 독립 task 병렬 처리 | `/batch` + worktree |
```

**수정할 내용**:

이 표 전체를 제거하고 한 줄 링크로 대체한다:

```markdown
## 단계별 에이전트 위임

각 단계에서의 에이전트 선택과 위임 조건은 [AGENT_EXECUTION_STRATEGY.md](AGENT_EXECUTION_STRATEGY.md)를 참조한다.
```

**변경 이유**: WORKFLOW.md의 역할은 "언제 무엇을 하는가"(프로세스 흐름)이고, "누가 어떻게 하는가"는 AGENT_EXECUTION_STRATEGY.md의 역할이다. 같은 표를 두 곳에 두면 한 곳만 갱신되고 나머지가 뒤처지는 drift가 생긴다.

---

### 수정 1-4. NEW_PROJECT_CHECKLIST.md 실행 원칙 축소

**현재 내용** (`docs/00-meta/NEW_PROJECT_CHECKLIST.md` 41~44행):

```markdown
## 실행 원칙
- 메인 세션이 모든 구현을 직접 처리하지 않는다.
- 문서 작성, 구현, 검증, QA는 가능한 한 관련 서브에이전트/skill에 먼저 위임한다.
- 메인 세션은 목표 정리, 결과 통합, 다음 결정에 집중한다.
```

**수정할 내용**:

```markdown
## 실행 원칙
- 에이전트 위임 전략은 [AGENT_EXECUTION_STRATEGY.md](AGENT_EXECUTION_STRATEGY.md)를 따른다.
```

**변경 이유**: 체크리스트 문서에 실행 원칙을 3줄로 요약해두면, 원본 문서와 미묘하게 달라질 때 어느 쪽이 맞는지 혼란이 생긴다. 한 줄 링크면 충분하다.

---

### 수정 1-5. README.md subagent 섹션 경량화

**현재 내용** (`README.md` 86~107행):

```markdown
## Subagent-first execution examples

### 문서 작성/분해
- 중요한 설계: architect-opus
- 일반 workitem 분해: planner

### 구현
\`\`\`text
/implement-workitem T-001-auth-session
\`\`\`

### 구현 검증
\`\`\`text
/validate-workitem T-001-auth-session
\`\`\`

### QA
자연어로 qa 에이전트를 사용해 방금 변경에 대한 회귀 위험과 엣지 케이스를 검토하도록 지시한다.

### 큰 병렬 변경
독립적인 여러 task를 동시에 처리해야 하면 /batch와 worktree를 우선 고려한다.
```

**수정할 내용**:

````markdown
## 에이전트 실행

이 보일러플레이트는 메인 세션이 오케스트레이션을 담당하고, 서브에이전트가 실작업을 수행하는 방식을 우선한다.

```text
/implement-workitem T-001-auth-session   # 구현
/validate-workitem T-001-auth-session    # 검증
```

상세 전략과 위임 조건은 [docs/00-meta/AGENT_EXECUTION_STRATEGY.md](docs/00-meta/AGENT_EXECUTION_STRATEGY.md)를 참조한다.
````

**변경 이유**: README는 첫 진입 문서이므로 핵심만 보여주고 상세는 링크한다. 모든 에이전트 예시를 README에 나열하면 README가 비대해지고, AGENT_EXECUTION_STRATEGY.md와 내용이 중복된다.

---

### Commit

> 수정 1-1 ~ 1-5 완료 후:
> `refactor: consolidate subagent-first principle into single source of truth`
> 대상 파일: `CLAUDE.md`, `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`, `docs/00-meta/WORKFLOW.md`, `docs/00-meta/NEW_PROJECT_CHECKLIST.md`, `README.md`

---

## 2단계: 설정과 문서의 불일치 해소

> 관련 항목: IG-013

### 왜 필요한가

`.claude/settings.json`에 `"plansDirectory": "./docs/30-workitems/plans"`가 설정되어 있고, `docs/30-workitems/plans/.gitkeep`도 존재한다. 하지만 이 `plans/` 디렉터리의 역할과 용도가 **어떤 문서에도 설명되어 있지 않다**.

Claude Code에서 Plan 모드를 사용하면 이 디렉터리에 계획 문서가 자동 생성되는데, 사용자가 이 디렉터리를 처음 보면 milestones/features/tasks와 어떻게 다른지 알 수 없다.

---

### 수정 2-1. docs/30-workitems/README.md에 plans 디렉터리 설명 추가

**현재 내용** (`docs/30-workitems/README.md` 6~10행):

```markdown
## 디렉터리 구성
- `milestones`: 큰 단계 목표
- `features`: 사용자 가치 단위 기능 문서
- `tasks`: 실제 구현 단위 작업 문서
- `_templates`: 새 문서를 만들 때 복사할 템플릿
```

**수정할 내용**:

```markdown
## 디렉터리 구성
- `milestones`: 큰 단계 목표
- `features`: 사용자 가치 단위 기능 문서
- `tasks`: 실제 구현 단위 작업 문서
- `plans`: Claude Code Plan 모드가 자동 생성하는 실행 계획 (`.claude/settings.json`의 `plansDirectory` 설정)
- `_templates`: 새 문서를 만들 때 복사할 템플릿
```

---

### 수정 2-2. docs/00-meta/TEMPLATE_GUIDE.md에 plansDirectory 매핑 명시

**현재 내용** (`docs/00-meta/TEMPLATE_GUIDE.md`):

settings.json과 실제 디렉터리 경로 매핑에 대한 언급이 전혀 없다.

**수정할 내용**:

`## Guardrail 운영 원칙` 섹션 바로 위에 아래 섹션을 추가한다:

```markdown
## 설정과 경로 매핑
- `.claude/settings.json`의 `plansDirectory`는 `./docs/30-workitems/plans`를 가리킨다.
- Claude Code Plan 모드에서 생성되는 실행 계획은 이 경로에 저장된다.
- plans는 workitem 문서(milestone/feature/task)와 다른 성격이다. workitem은 사람이 정의한 작업 단위이고, plan은 에이전트가 생성한 실행 계획이다.
```

---

### 수정 2-3. docs/00-meta/WORKFLOW.md에 plans 언급 추가

**현재 내용** (`docs/00-meta/WORKFLOW.md`의 `## 3. 작업 단위 분해` 섹션):

```markdown
## 3. 작업 단위 분해
- 마일스톤 단위 목표를 `docs/30-workitems/milestones`에 만든다.
- 기능 단위 문서를 `docs/30-workitems/features`에 만든다.
- 실제 구현 단위 문서를 `docs/30-workitems/tasks`에 만든다.
```

**수정할 내용**:

```markdown
## 3. 작업 단위 분해
- 마일스톤 단위 목표를 `docs/30-workitems/milestones`에 만든다.
- 기능 단위 문서를 `docs/30-workitems/features`에 만든다.
- 실제 구현 단위 문서를 `docs/30-workitems/tasks`에 만든다.
- 에이전트의 실행 계획은 `docs/30-workitems/plans`에 자동 저장된다.
```

---

### Commit

> 수정 2-1 ~ 2-3 완료 후:
> `docs: document plansDirectory mapping and plans/ directory purpose`
> 대상 파일: `docs/30-workitems/README.md`, `docs/00-meta/TEMPLATE_GUIDE.md`, `docs/00-meta/WORKFLOW.md`

---

## 3단계: 문서 운영 규칙 정비

> 관련 항목: IG-010, IG-015, IG-016, IG-006, IG-002

이 단계에서는 지금까지 비어 있던 "문서를 어떻게 운영하는가"에 대한 규칙을 정의한다. 모든 내용은 `docs/00-meta/WORKFLOW.md`에 추가한다. WORKFLOW.md가 프로세스 흐름의 중심 문서이고, 문서 운영 규칙도 프로세스의 일부이기 때문이다.

---

### 수정 3-1. Living Docs vs Immutable Records 구분 추가

**추가 위치**: `docs/00-meta/WORKFLOW.md`의 `## 기본 원칙` 섹션 바로 뒤

**추가할 내용**:

```markdown
## 문서 운영 방식

| 유형 | 예시 | 운영 방식 |
|------|------|-----------|
| Living Doc | Charter, Architecture, Design System, Workflow | 현재 기준으로 계속 갱신한다. 과거 버전은 Git 이력으로 확인한다. |
| Record | ADR, QA Findings | 기록 보존 우선. 기존 항목을 덮어쓰지 않고 추가 또는 대체한다. |

- ADR은 기존 결정을 뒤집을 때 새 ADR로 대체하는 것을 기본으로 한다.
- QA Findings는 회차 또는 날짜 기준으로 누적 기록한다.
- Improvement Guide는 Living Doc이지만, 완료된 항목은 삭제하지 않고 상태를 갱신한다.
```

**왜 필요한가**: 지금은 charter 같은 "계속 갱신하는 문서"와 ADR 같은 "기록으로 남기는 문서"의 운영 방식이 구분되어 있지 않다. 이 구분이 없으면 ADR을 일반 문서처럼 덮어쓰거나, charter를 함부로 과거 버전 위에 쌓는 식의 혼란이 생길 수 있다.

---

### 수정 3-2. 문서 상태 전이 규칙 추가

**추가 위치**: `docs/00-meta/WORKFLOW.md`의 수정 3-1에서 추가한 "문서 운영 방식" 바로 뒤

**추가할 내용**:

````markdown
## 문서 상태 전이

    draft → ready → in-progress → done
                         ↓↑
                      blocked
    done → deprecated (필요 시)

| 전이 | 최소 조건 |
|------|-----------|
| draft → ready | 필수 섹션이 채워졌고, 자기 검증 또는 리뷰를 거쳤다 |
| ready → in-progress | 실제 구현/작업이 시작됐다 |
| in-progress → blocked | 외부 의존성이나 미결 질문으로 진행 불가 |
| blocked → in-progress | 블로킹 원인이 해소됐다 |
| in-progress → done | 완료 기준을 충족했다 |
| done → deprecated | 대체되었거나 더 이상 유효하지 않다 |

이 규칙은 가이드 수준이며, 훅이나 스크립트로 강제하지 않는다.
````

**왜 필요한가**: 현재 모든 템플릿에 `Status` 필드가 있고 `draft/ready/in-progress/blocked/done/deprecated` 값을 사용하지만, 유효한 전이 경로와 전이 조건이 없다. `draft`에서 바로 `done`으로 점프하거나, `blocked` 상태가 해소 조건 없이 방치될 수 있다.

---

### 수정 3-3. 계층 간 문서 충돌 해결 원칙 추가

**추가 위치**: `docs/00-meta/WORKFLOW.md`의 수정 3-2 뒤

**추가할 내용**:

```markdown
## 문서 충돌 해결

문서 간 내용이 모순될 때:
1. 상위 문서가 하위 문서보다 권위가 높다 (charter > architecture > workitems).
2. 상위가 맞다면 하위를 수정하고, 상위가 outdated라면 상위를 먼저 갱신한 뒤 하위를 맞춘다.
3. 의도적 범위/기술 변경이라면 ADR을 남긴다.
```

**왜 필요한가**: "상위 문서 없이 하위 문서를 먼저 만들지 않는다"는 원칙은 있지만, 이미 만들어진 문서 사이에서 모순이 발견됐을 때의 대응 규칙이 없다. 예를 들어 charter에서 "비범위"로 선언한 기능이 architecture에 포함된 경우, 어느 문서를 기준으로 수정해야 하는지 불명확하다.

---

### 수정 3-4. ADR 작성 가이드 신규 파일 추가

**새 파일**: `docs/90-decisions/_ADR_GUIDE.md`

**내용**:

```markdown
# ADR 작성 가이드

## ADR로 남길 결정의 기준
- 되돌리기 어려운 기술 선택 (프레임워크, DB, 인증 방식 등)
- 두 가지 이상 합리적인 대안이 있었던 결정
- 프로젝트 범위나 아키텍처를 크게 바꾸는 결정

일상적인 구현 결정(변수명, 함수 분리 등)은 ADR 대상이 아니다.

## 상태값
- `proposed`: 제안됨, 아직 확정 아님
- `accepted`: 팀/개인이 수용함
- `superseded`: 새 ADR로 대체됨
- `deprecated`: 더 이상 유효하지 않음

## 대체 절차
새 ADR이 기존 ADR을 대체할 때:
1. 기존 ADR 상태를 `superseded`로 변경한다.
2. 기존 ADR 상단에 "대체: ADR-xxx"를 기록한다.
3. 새 ADR에서 기존 ADR을 참조한다.

## 권장 섹션
- 상태
- 배경 (왜 이 결정이 필요했는가)
- 결정 (무엇을 선택했는가)
- 근거 (왜 이 선택인가, 대안은 무엇이었는가)
- 결과 (이 결정으로 무엇이 달라지는가)
- 후속 작업

## 참고
- 짧아도 된다. 핵심은 "왜 이 선택을 했는가"를 기록하는 것이다.
- ADR-001-doc-hierarchy.md를 예시로 참고한다.
```

**왜 필요한가**: 현재 `CLAUDE.md`에 "중요한 기술적 선택은 ADR로 남긴다"고만 적혀 있는데, "중요한"의 기준이 없다. ADR-001이 예시로 존재하지만, 상태 수명주기와 대체 절차가 정의되어 있지 않다.

---

### 수정 3-5. 완료 정의(DoD) 가이드 추가

**추가 위치**: `docs/00-meta/WORKFLOW.md`의 `## 기본 원칙` 섹션에 항목 추가

**현재 내용**:

```markdown
## 기본 원칙
- 상위 문서 없이 하위 문서만 먼저 만들지 않는다.
- 흩어진 메모보다 정해진 위치의 문서를 갱신한다.
- 애매한 사항은 문서에 가정과 열린 질문으로 남긴다.
```

**수정할 내용**:

```markdown
## 기본 원칙
- 상위 문서 없이 하위 문서만 먼저 만들지 않는다.
- 흩어진 메모보다 정해진 위치의 문서를 갱신한다.
- 애매한 사항은 문서에 가정과 열린 질문으로 남긴다.
- 작업을 완료(done)로 전환하기 전에 최소한 다음을 확인한다:
    - 구현 범위가 관련 workitem 문서와 일치한다
    - 관련 검증 항목(테스트 포인트, 검증 방법)이 통과했다
    - 필요한 경우 관련 문서가 함께 갱신되었다
```

**왜 필요한가**: 현재 템플릿에 `완료 기준`, `검증 방법`, `완료 조건` 등이 개별적으로 존재하지만, 통합된 완료 정의 원칙이 없다. 단, 이 보일러플레이트는 solo 개발자부터 팀까지 대상이므로 "코드 리뷰 통과" 같은 팀 전용 조건은 넣지 않는다.

---

### Commit (2개)

> 수정 3-1, 3-2, 3-3, 3-5 완료 후:
> `docs: add document operation rules, status transitions, conflict resolution, and DoD to WORKFLOW.md`
> 대상 파일: `docs/00-meta/WORKFLOW.md`

> 수정 3-4 완료 후:
> `docs: add ADR writing guide with lifecycle and supersede rules`
> 대상 파일: `docs/90-decisions/_ADR_GUIDE.md`

---

## 4단계: 템플릿 보강

> 관련 항목: IG-009, IG-014, IG-001

---

### 수정 4-1. ARCHITECTURE_OVERVIEW.md에 품질 속성(NFR) 섹션 추가

**현재 내용** (`docs/20-system/ARCHITECTURE_OVERVIEW.md`):

```markdown
# 아키텍처 개요

## 0. Status
draft

## 1. 기술 요약
## 2. 시스템 경계
## 3. 상위 아키텍처
## 4. 주요 도메인 모델
## 5. 데이터 흐름
## 6. 외부 연동 지점
## 7. 기술 선택
## 8. 리스크
## 9. 열린 질문
```

**수정할 내용**:

`## 7. 기술 선택`과 `## 8. 리스크` 사이에 새 섹션을 추가한다. 기존 8, 9번의 번호도 밀어준다:

```markdown
## 7. 기술 선택

## 8. 품질 속성
<!-- 프로젝트에서 중요한 품질 요구사항을 시나리오 기반으로 정리한다.
     모든 항목을 채울 필요는 없다. 프로젝트에 해당하는 것만 작성한다. -->

### 성능
<!-- 예: 핵심 사용자 흐름의 응답시간 목표, 동시 사용자 기대치 -->

### 보안
<!-- 예: 보호해야 할 데이터, 인증/인가 최소 기준, 민감 데이터 처리 방침 -->

### 신뢰성
<!-- 예: 허용 가능한 장애 범위, 데이터 유실 허용 수준 -->

### 운영성
<!-- 예: 로그/모니터링/추적성 기대 수준, 배포 빈도 -->

## 9. 리스크

## 10. 열린 질문
```

**왜 필요한가**: 현재 구조는 기능 요구(what to build)에는 강하지만, 품질 요구(how well)를 다루지 않는다. 아키텍처 결정을 가장 크게 좌우하는 것은 기능이 아니라 NFR(성능, 보안, 신뢰성 등)이다. arc42, AWS Well-Architected Framework 모두 품질 속성을 별도 축으로 다룬다. 이 섹션이 없으면 ADR의 근거도 약해지고, 구현 단계에서 품질 요구가 뒤늦게 등장한다.

**주의**: 이 섹션은 HTML 주석으로 가이드를 제공하되, 선언문("보안을 중시한다")보다 시나리오 기반 표현("사용자 비밀번호는 bcrypt로 해싱하고, 세션 토큰은 24시간 후 만료한다")을 유도한다.

---

### 수정 4-2. DESIGN_SYSTEM.md를 비UI 프로젝트에서도 사용 가능하게 수정

**현재 내용** (`docs/20-system/DESIGN_SYSTEM.md`):

```markdown
# 디자인 시스템

## 0. Status
draft

## 1. 시각 방향
## 2. 색상 체계
## 3. 타이포그래피
## 4. 간격 규칙
## 5. 레이아웃 규칙
## 6. 재사용 컴포넌트
## 7. 상태와 변형
## 8. 접근성
## 9. 열린 질문
```

**수정할 내용**:

```markdown
# 디자인 시스템

## 0. Status
draft

<!-- 이 문서는 프로젝트의 인터페이스 설계 규칙을 정의한다.
     프로젝트 유형에 따라 해당하는 그룹의 섹션만 채운다.
     사용하지 않는 그룹은 통째로 삭제해도 된다.

     UI 프로젝트 (웹, 모바일): "UI 프로젝트" 그룹을 사용한다.
     API/백엔드 프로젝트: "API/백엔드 프로젝트" 그룹을 사용한다.
     CLI 프로젝트: "CLI 프로젝트" 그룹을 사용한다.
     혼합 프로젝트: 해당하는 그룹을 조합한다. -->

---

## UI 프로젝트

### 1. 시각 방향
### 2. 색상 체계
### 3. 타이포그래피
### 4. 간격 규칙
### 5. 레이아웃 규칙
### 6. 재사용 컴포넌트
### 7. 상태와 변형
### 8. 접근성
### 9. 열린 질문

---

## API/백엔드 프로젝트

### 1. API 설계 규칙
<!-- 응답 포맷 (JSON 구조, envelope 패턴 등), 버전 관리, 페이지네이션 규칙 -->

### 2. 에러 코드 체계
<!-- HTTP 상태 코드 매핑, 에러 응답 포맷, 비즈니스 에러 코드 규칙 -->

### 3. 네이밍 컨벤션
<!-- 엔드포인트, 필드명, DB 스키마 네이밍 규칙 -->

### 4. 로깅/관측성 규칙
<!-- 로그 레벨 기준, 구조화 로그 포맷, 요청 추적 ID 규칙 -->

### 5. 열린 질문

---

## CLI 프로젝트

### 1. 출력 포맷 규칙
<!-- 터미널 출력 스타일, 색상 사용 규칙, 진행 표시 방식 -->

### 2. 플래그/명령어 네이밍
<!-- 명령어 구조, 플래그 네이밍 컨벤션, 도움말 포맷 -->

### 3. 열린 질문
```

**왜 필요한가**: 현재 템플릿은 UI 프로젝트 전용이다. 백엔드 API, CLI 도구, 데이터 파이프라인 프로젝트에서는 80% 이상이 "해당 없음"이 되어서, 사용자가 이 문서를 아예 건너뛴다. 하지만 비UI 프로젝트에도 설계 규칙은 존재한다 (API 응답 포맷, 에러 코드, 네이밍 등). 파일을 분리하지 않고 하나의 파일 안에서 프로젝트 유형별 그룹을 두되, 사용하지 않는 그룹은 통째로 삭제하도록 안내한다. 모든 유형의 섹션이 동시에 보이는 것이 아니라 프로젝트 초기에 해당 그룹만 남기는 방식이므로, "모든 유형을 한 파일에 전부 나열하는" 문제를 회피한다. 추후 이 구조가 어색하다면 파일명을 `INTERFACE_CONVENTIONS.md`로 변경하는 것도 검토할 수 있다.

---

### 수정 4-3. 상위 템플릿 문서에 작성 가이드 힌트 추가

각 템플릿의 빈 섹션에 HTML 주석으로 짧은 작성 안내를 추가한다.

#### 4-3a. PROJECT_CHARTER.md

**현재 내용**:

```markdown
# 프로젝트 헌장

## 0. Status
draft

## 1. 프로젝트 요약
## 2. 대상 사용자
## 3. 해결하려는 문제
## 4. 목표
## 5. 비목표
## 6. 성공 기준
## 7. 제약 조건
## 8. 핵심 사용자 흐름
## 9. 열린 질문
```

**수정할 내용**:

```markdown
# 프로젝트 헌장

## 0. Status
draft

## 1. 프로젝트 요약
<!-- 한두 문장으로 무엇을 만드는지 적는다. -->

## 2. 대상 사용자
<!-- 누가 이 제품을 쓰는지. 가능하면 구체적인 페르소나를 적는다. -->

## 3. 해결하려는 문제
<!-- 대상 사용자가 현재 겪고 있는 문제. "왜 이것을 만드는가"에 대한 답. -->

## 4. 목표
<!-- 이 프로젝트가 달성하려는 것. 검증 가능한 표현을 우선한다. -->

## 5. 비목표
<!-- 의도적으로 하지 않기로 한 것. 범위를 제한하는 데 중요하다. -->

## 6. 성공 기준
<!-- 프로젝트가 성공했다고 판단할 수 있는 측정 가능한 기준. -->

## 7. 제약 조건
<!-- 기술적, 시간적, 인력적 제약. 스택이 미정이면 그것도 제약이다. -->

## 8. 핵심 사용자 흐름
<!-- 사용자가 가장 자주 할 1~3가지 핵심 동작의 흐름. -->

## 9. 열린 질문
<!-- 아직 답이 없는 중요한 질문들. -->
```

#### 4-3b. ARCHITECTURE_OVERVIEW.md

4-1에서 NFR 섹션을 추가하면서 이미 HTML 주석 가이드를 포함했다. 나머지 섹션에도 동일한 패턴으로 추가한다:

```markdown
## 1. 기술 요약
<!-- 한두 문장으로 이 시스템이 무엇인지, 어떤 기술로 구성되는지 적는다. -->

## 2. 시스템 경계
<!-- 이 시스템이 다루는 것과 다루지 않는 것. 외부 시스템과의 경계선. -->
<!-- C4 관점: 이 섹션은 System Context 레벨에 해당한다. -->

## 3. 상위 아키텍처
<!-- 주요 실행 단위(서비스, 모듈, 패키지)와 그 관계. -->
<!-- C4 관점: 이 섹션은 Container 레벨에 해당한다. -->

## 4. 주요 도메인 모델
<!-- 핵심 엔티티와 관계. 과도한 세부 필드보다 개념 수준. -->

## 5. 데이터 흐름
<!-- 주요 데이터가 시스템을 통해 어떻게 흘러가는지. -->

## 6. 외부 연동 지점
<!-- 외부 API, 서드파티 서비스, 외부 데이터 소스. -->

## 7. 기술 선택
<!-- 언어, 프레임워크, DB, 인프라 등 주요 기술 선택과 이유. 스택이 미정이면 미정으로 적는다. -->
```

**왜 필요한가**: `/bootstrap-project` 스킬을 사용하면 이 템플릿을 자동으로 채워주지만, 수동으로 시작하는 사용자는 각 섹션에 무엇을 어느 수준까지 써야 하는지 알 수 없다. HTML 주석이므로 렌더링 시 보이지 않고, 작성 후 지워도 되고 남겨도 된다.

---

### Commit (2개)

> 수정 4-1, 4-3b 완료 후:
> `docs: add NFR section and inline guidance hints to ARCHITECTURE_OVERVIEW template`
> 대상 파일: `docs/20-system/ARCHITECTURE_OVERVIEW.md`

> 수정 4-2, 4-3a 완료 후:
> `docs: add non-UI project sections to DESIGN_SYSTEM and inline hints to PROJECT_CHARTER`
> 대상 파일: `docs/20-system/DESIGN_SYSTEM.md`, `docs/10-charter/PROJECT_CHARTER.md`

---

## 5단계: 에이전트 설정 개선

> 관련 항목: IG-019, IG-020

---

### 수정 5-1. architect-opus description에 proactive 힌트와 비용 가드 추가

**현재 내용** (`.claude/agents/architect-opus.md` 3행):

```yaml
description: Use for high-impact product planning, major architecture decisions, milestone decomposition, and important design tradeoff analysis.
```

**수정할 내용**:

```yaml
description: Use proactively for major product or architecture decisions, milestone decomposition, and important design tradeoff analysis. Avoid routine low-impact tasks.
```

**왜 필요한가**: 다른 에이전트(planner, qa, reviewer, builder-sonnet, validator-sonnet)는 모두 "Use proactively for..."로 통일되어 있는데, architect-opus만 "Use for..."다. Claude Code의 description은 위임 시점 판단에 사용되며, proactive 힌트가 없으면 다른 에이전트보다 위임 우선순위가 낮아질 수 있다. 동시에 Opus는 비용이 크므로 "Avoid routine low-impact tasks"로 비용 가드를 건다.

---

### 수정 5-2. 각 에이전트에 턴 예산 소진 시 종료 규칙 추가

현재 어떤 에이전트도 maxTurns에 도달했을 때의 행동이 정의되어 있지 않다. 큰 구현 작업이 20턴에서 중단되면 중간 상태의 코드가 남고, 메인 세션이 어디까지 완료됐는지 파악할 수 없다.

#### 5-2a. builder-sonnet.md

**현재 내용** (`.claude/agents/builder-sonnet.md` 규칙 섹션 끝):

```markdown
규칙:
- 범위 밖 변경은 하지 않는다.
- 작업 전 관련 문서의 범위와 비범위를 먼저 확인한다.
- 구현 후 아래를 짧게 요약한다.
    - 수정 파일
    - 핵심 변경 사항
    - 테스트/검증 여부
    - 남은 리스크 또는 미결정 사항
- 장문의 탐색 결과를 메인 세션에 그대로 넘기지 않는다.
```

**수정할 내용** (마지막 항목 뒤에 추가):

```markdown
- 턴이 부족하거나 범위가 예상보다 크면, 현재까지의 진행 상황·수정 파일·남은 작업·추천 다음 액션을 요약하고 종료한다.
```

#### 5-2b. validator-sonnet.md

**현재 내용** (`.claude/agents/validator-sonnet.md` 규칙 섹션):

```markdown
규칙:
- 구현 자체를 다시 크게 고치지 않는다.
- 검증과 판정에 집중한다.
- 장문의 로그 대신 핵심 판단만 요약한다.
```

**수정할 내용** (마지막 항목 뒤에 추가):

```markdown
- 시간/턴이 부족하면 확인된 범위까지의 핵심 판단만 요약하고 종료한다.
```

#### 5-2c. qa.md

**현재 내용** (`.claude/agents/qa.md` 규칙 섹션):

```markdown
규칙:
- 사용자에게 보이는 오류, 데이터 무결성 문제, 상태 관리 버그, 검증 누락을 우선 본다.
- 결과는 P0, P1, P2로 나눈다.
- 중복 지적은 피한다.
- 가능하면 재현 절차와 영향 범위를 함께 적는다.
```

**수정할 내용** (마지막 항목 뒤에 추가):

```markdown
- 시간/턴이 부족하면 확인된 범위까지의 핵심 판단만 요약하고 종료한다.
```

#### 5-2d. planner.md

**현재 내용** (`.claude/agents/planner.md` 규칙 섹션):

```markdown
규칙:
- 코드를 구현하지 않는다.
- 중요한 아키텍처 결정이나 큰 제품 방향 설계는 `architect-opus`를 우선 고려한다.
- 확실하지 않은 내용은 가정으로 표시한다.
- 상위 문서와 하위 문서의 역할을 섞지 않는다.
- 열린 질문이 남으면 문서에 명시한다.
```

**수정할 내용** (마지막 항목 뒤에 추가):

```markdown
- 시간/턴이 부족하면 정리된 범위까지의 결과와 남은 분해 작업을 요약하고 종료한다.
```

#### 5-2e. reviewer.md

**현재 내용** (`.claude/agents/reviewer.md` 규칙 섹션):

```markdown
규칙:
- 막연한 칭찬은 하지 않는다.
- 결과는 P0, P1, P2로 나눈다.
- 어떤 문서를 어떻게 고치면 좋을지 구체적으로 제안한다.
- 상위 설계 문제와 하위 구현 문제를 구분해서 지적한다.
```

**수정할 내용** (마지막 항목 뒤에 추가):

```markdown
- 시간/턴이 부족하면 확인된 범위까지의 핵심 판단만 요약하고 종료한다.
```

**왜 필요한가**: maxTurns 설정(planner: 12, reviewer: 12, qa: 16, validator: 16, builder: 20)이 있지만, 턴 수 소진 시의 행동이 정의되어 있지 않다. 이 한 줄이 없으면 에이전트가 중간에 끊기면서 불완전한 상태로 종료되고, 메인 세션이 상황을 파악하기 어려워진다.

---

### Commit (2개)

> 수정 5-1 완료 후:
> `fix: add proactive hint and cost guard to architect-opus description`
> 대상 파일: `.claude/agents/architect-opus.md`

> 수정 5-2a ~ 5-2e 완료 후:
> `fix: add graceful turn-budget exhaustion rule to all subagent prompts`
> 대상 파일: `.claude/agents/builder-sonnet.md`, `.claude/agents/validator-sonnet.md`, `.claude/agents/qa.md`, `.claude/agents/planner.md`, `.claude/agents/reviewer.md`

---

## 채택하지 않는 항목과 그 이유

아래 항목들은 IMPROVE_GUIDE.md에서 제안되었지만, 현재 시점에서 수정하지 않는다.

### IG-003. 테스트 전략 문서화 (P2)

GUARDRAILS_STRATEGY.md가 명시한 대로, 테스트 도구는 스택 확정 후에 정해진다. 스택 미확정 상태에서 ARCHITECTURE_OVERVIEW에 테스트 전략 섹션을 추가하면 cross-platform 철학과 충돌한다. 대신 `bootstrap-stack` 결과물에서 다루는 것이 올바른 위치다.

### IG-004. CI/CD 가이드라인 (P2)

IG-003과 같은 이유. CI/CD는 GitHub Actions인지 GitLab CI인지, Node인지 Python인지에 따라 완전히 달라진다. shared 기본값에 넣을 수 없다.

### IG-008. C4 아키텍처 구조 (P2)

수정 4-3b에서 ARCHITECTURE_OVERVIEW의 시스템 경계와 상위 아키텍처 섹션에 C4 관점 힌트를 HTML 주석으로 이미 추가했다. 이것으로 충분하다. C4 전면 도입은 다이어그램 도구 없이 의미가 반감된다.

### IG-011. GitHub Issues/Projects 연동 (P2)

이 보일러플레이트는 "문서 중심"이지 "GitHub 중심"이 아니다. GitLab, Linear 등을 쓰는 팀에게는 해당 없는 가이드가 된다. 현재 `.github/ISSUE_TEMPLATE`는 유지하되, workitem과의 연동 규칙을 필수로 강제하지 않는다.

### IG-012. 문서 메타데이터 표준화 (P3)

YAML frontmatter를 파싱하는 도구가 없다. 수동으로만 유지해야 하는 메타데이터는 비용만 늘린다. 자동 검증이 필요한 시점이 되면 그때 도입한다.

### IG-005. 문서 버전 관리 체계 (제거)

Git commit history + ADR이 이미 "언제, 무엇을, 왜"를 담는다. 수동 변경 이력 표는 Git과 이중 관리가 되어 신뢰를 잃는다.

---

## 수정 파일 요약

| 파일 | 수정 내용 | 관련 단계 |
|------|-----------|-----------|
| `CLAUDE.md` | 에이전트 실행 원칙 축소 + 위임 트리거 추가 + 상세 링크 | 1단계 |
| `docs/00-meta/AGENT_EXECUTION_STRATEGY.md` | 위임 트리거 표 교체 + 스킬 체이닝 흐름 추가 | 1단계 |
| `docs/00-meta/WORKFLOW.md` | 에이전트 표 → 링크로 대체 + plans 언급 + 운영 규칙 5개 추가 | 1, 2, 3단계 |
| `docs/00-meta/NEW_PROJECT_CHECKLIST.md` | 실행 원칙 → 1줄 링크로 축소 | 1단계 |
| `README.md` | subagent 섹션 경량화 | 1단계 |
| `docs/30-workitems/README.md` | plans 디렉터리 설명 추가 | 2단계 |
| `docs/00-meta/TEMPLATE_GUIDE.md` | plansDirectory 매핑 섹션 추가 | 2단계 |
| `docs/90-decisions/_ADR_GUIDE.md` | 신규 파일 생성 | 3단계 |
| `docs/20-system/ARCHITECTURE_OVERVIEW.md` | NFR 섹션 추가 + HTML 주석 가이드 | 4단계 |
| `docs/20-system/DESIGN_SYSTEM.md` | 비UI 프로젝트 섹션 추가 | 4단계 |
| `docs/10-charter/PROJECT_CHARTER.md` | HTML 주석 가이드 추가 | 4단계 |
| `.claude/agents/architect-opus.md` | description 수정 | 5단계 |
| `.claude/agents/builder-sonnet.md` | 턴 소진 종료 규칙 추가 | 5단계 |
| `.claude/agents/validator-sonnet.md` | 턴 소진 종료 규칙 추가 | 5단계 |
| `.claude/agents/qa.md` | 턴 소진 종료 규칙 추가 | 5단계 |
| `.claude/agents/planner.md` | 턴 소진 종료 규칙 추가 | 5단계 |
| `.claude/agents/reviewer.md` | 턴 소진 종료 규칙 추가 | 5단계 |