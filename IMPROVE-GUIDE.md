# Boilerplate 개선 적용 가이드 (IMPROVE-GUIDE)

> 작성일: 2026-05-03
> 목적: 보일러플레이트의 문서·skill·agent 구조를 개선하기 위한 단계별 행동 지침. 모델 별칭 통일, SSOT 인프라, 워크아이템 라이프사이클(/repair-workitem · /finalize-workitem · /stabilize-milestone), TDD 디폴트, 단순성·Clean Code 가드레일, 발굴 단계 분리(/discover-product) 등 13개 Step.
> 사용법: 이 문서를 위에서부터 아래로 순서대로 따라간다. 각 Step은 자체 완결적이며, 직전 Step이 끝나야 다음 Step을 시작할 수 있다.

---

## 0. 이 가이드 사용법

### 0.1 적용 단계 한눈에

| Step | 주제 | 핵심 산출물 | 의존 |
|------|------|-------------|------|
| 1 | 모델 별칭 정책 | settings/agents의 model을 별칭으로 통일 + ADR-004 | — |
| 2 | 약한 skill 정리 | plan-workitem/review-doc agent 바인딩, write-charter/architecture 삭제 | — |
| 3 | SSOT 인프라 도입 | STRUCTURE.md, 90-decisions/README.md 신설 + 슬림화 + ADR-005 | 1, 2 |
| 4 | 단순성·Clean Code 가드레일 1단계 | CLAUDE.md 단순성 단락, agent self-check, ADR-006 | 3 |
| 5 | validation report + /repair-workitem | reports/ 디렉터리, /repair-workitem skill, validate-workitem 강화 | 3 |
| 6 | /finalize-workitem 신설 | finalize-workitem skill, TASK_TEMPLATE 갱신, ADR-007, ADR-008 | 5 |
| 7 | /stack-guard 1단계 | stack-guard skill, validate 통합 명령 안내(verify 스크립트는 보일러플레이트엔 만들지 않고 `/stack-guard` 발화 시 생성) | 3 |
| 8 | 에이전트 TDD 기본 흐름 | TASK_TEMPLATE의 AC 섹션, implement-workitem RGR phase, ADR-009 | 5, 6, 7 |
| 9 | /stabilize-milestone | stabilize-milestone skill, QA_FINDINGS·IMPROVEMENT_GUIDE 양식 | 5, 6, 7 |
| 10 | Architecture 의존성 규칙 + stabilize-milestone 통합 | ARCHITECTURE_OVERVIEW의 ## 3-1, stabilize-milestone에 Clean Code 6항목 통합 | 4, 9 |
| 11 | /discover-product 신설 | discover-product skill, DISCOVERY 템플릿 | 3 |
| 12 | /bootstrap-project 입력 조정 | bootstrap-project 입력 우선순위 조정 (`$ARGUMENTS` brief 우선 → 비었을 때 DISCOVERY.md), charter 템플릿 확장 | 11 |
| 13 | /batch + worktree 가이드 | AGENT_EXECUTION_STRATEGY.md 위임 트리거 갱신 (CLAUDE.md는 조건부 정리) | 3 |

### 0.2 공통 운영 원칙 (모든 Step에 적용)

- **한 Step = 한 PR(또는 한 commit 묶음) 권장**. 큰 Step은 내부에서 논리적 sub-commit으로 나눠도 무방.
- **사전에 새 브랜치를 만든다**. 예: `git checkout -b chore/improve-step-01-model-alias`.
- **각 Step 끝의 "검증" 체크리스트가 모두 통과해야 다음 Step으로 넘어간다.**
- **prototyping 대상이라고 표시된 항목(예: PostToolUse hook 자동 등록)은 본 가이드 범위 밖**. 공식 문서로 동작이 검증되지 않은 메커니즘은 사용자 환경에 자동 적용하지 않는다.
- **본 가이드의 모든 코드/문서 수정은 "현재 파일이 이러하다 → 이렇게 바꾼다"의 형태로 명시**한다. 모호한 부분은 보일러플레이트의 코드와 문서를 모두 파악한 뒤 본 가이드가 결정한 권장안을 그대로 따른다(0.4 결정 사항 표 참조).
- **다중 Step에서 같은 파일을 수정하는 경우 매 Step 시작 시 그 파일을 다시 읽어 라인 번호를 재확인한다.** 본 가이드의 라인 번호 클레임은 *해당 Step 시작 시점*을 기준으로 작성됐다. 예: WORKFLOW.md는 Step 5·6·9에서, TASK_TEMPLATE.md는 Step 3·6·8에서, CLAUDE.md는 Step 3·4·8에서 각각 수정된다. 이전 Step의 변경분 때문에 라인 번호는 매 단계마다 시프트한다.
- **코드 블록 안에 markdown 코드 블록을 보여줄 때**는 가이드가 백슬래시 이스케이프(`\`\`\``)를 사용한다. 실제 파일에 적용할 때는 백슬래시를 제거하고 진짜 backtick(`` ``` ``)으로 변환한다.

### 0.3 ADR 번호 체계 (사전 결정)

본 가이드 적용 시 새로 추가되는 ADR 번호:

| ADR # | 제목 | 추가 단계 |
|-------|------|-----------|
| 001 | Doc hierarchy | (기존, 변경 없음) |
| 002 | Initial project decisions | (bootstrap-project가 fork된 새 프로젝트에서 생성하는 placeholder, 보일러플레이트엔 미존재) |
| 003 | Stack selection | (bootstrap-stack가 fork된 새 프로젝트에서 생성하는 placeholder, 보일러플레이트엔 미존재) |
| 004 | Model alias policy | Step 1 |
| 005 | SSOT (Single Source of Truth) | Step 3 |
| 006 | Simplicity, Clean Code, and Clean Architecture priority | Step 4 |
| 007 | Workitem lifecycle | Step 6 |
| 008 | Commit convention (Conventional Commits) | Step 6 |
| 009 | TDD default + opt-out procedure | Step 8 |

ADR-002, ADR-003은 보일러플레이트 자체에 존재하지 않고(파일 없음) bootstrap 단계에서 새 프로젝트가 생성하는 placeholder다. 본 가이드는 ADR-002/003을 만들지 않는다 — 기존 skill의 출력 명세에서 그 위치만 그대로 유지한다.

### 0.4 모호점 결정 사항 (가이드 작성자 결정)

다음 항목들은 두 가지 처분안 또는 옵션이 가능하지만, 본 가이드는 일관성을 위해 다음과 같이 결정한다. 결정 근거는 보일러플레이트 코드·문서의 현재 상태와 fork 컨텍스트 제약을 종합한 것이다.

| 항목 | 옵션 | 결정 |
|------|------|------|
| Step 2 — write-charter/write-architecture 처분 | (A) 삭제 vs (B) refine으로 축소 | (A) 삭제 — mid-project 갱신은 자연어 + planner agent로 충분. 진입점 중복은 흐름을 흐린다. |
| Step 5 — validation report 양식 | markdown vs JSON | markdown — 사람과 sub-agent 모두 Read로 처리 가능. |
| Step 3 — _DOCUMENT_OWNERSHIP.md vs STRUCTURE.md 분리 | 별도 파일 vs STRUCTURE.md 부록 | STRUCTURE.md 부록 섹션 — 추가 cross-ref 비용 회피. |
| Step 3 — ADR-005-ssot.md 작성 여부 | 선택 | 작성 — SSOT는 정책이므로 ADR이 적합. |
| Step 6 — ADR-007-workitem-lifecycle.md 작성 여부 | 선택 | 작성 — discover→…→stabilize 라이프사이클은 정책이므로 ADR이 적합. |
| Step 1 — bootstrap-project/bootstrap-stack의 model 라인 처리 | (a) `opus`로 변경 vs (b) Step 12에서 라인 자체 제거 | Step 1에서 `opus`로 변경, Step 12에서 bootstrap-project의 model 라인은 그대로 유지(별칭이라 staleness 없음). bootstrap-stack도 동일. |
| Step 6 — finalize-workitem 차이 처리 (task 문서 변경 예정 파일과 git 실제 변경 어긋남) | (A) 사용자 확인 후 진행 vs (B) 차이 출력 후 즉시 종료(`Needs Review`) + `--apply` 인자로 force 진행 | (B) 채택. 이유: skill이 `context: fork`라 sub-agent가 사용자에게 실시간 확인을 받을 수 없다. |
| Step 12 — bootstrap-project 갱신 모드의 사용자 확인 | (A) 사용자 확인 후 반영 vs (B) `--apply` 없으면 diff 출력 후 즉시 종료 | (B) 채택. 이유: 동일하게 fork 컨텍스트 제약. |
| Step 12 — PROJECT_CHARTER 핵심 가정 섹션 위치 | (A) `## 8.1 핵심 가정` 서브섹션 vs (B) `## 9. 핵심 가정` 신설 + 기존 `## 9. 열린 질문`을 `## 10. 열린 질문`으로 시프트 | (B) 채택. 이유: `## 8`(핵심 사용자 흐름)과 핵심 가정은 의미상 sibling이 아니다(흐름 vs 추측). 서브섹션 번호가 잘못된 종속 관계를 시사한다. fork 다운스트림 charter가 `## 9. 열린 질문`을 직접 참조 중일 가능성을 고려해 본 결정은 다운스트림 마이그레이션 한 줄(헤더 번호만 시프트)을 동반한다. |
| Step 11 — `/discover-product --fast` R0 처리 | (A) R0과 R3을 모두 건너뛰고 R1+R2+R4만 도는 단축 흐름 vs (B) R0의 페르소나 *후보 제시·선택* 단계만 건너뛴다. 페르소나는 (1) `$ARGUMENTS` 명시 (2) architect-opus 자동 선정으로 확보 | (B) 채택. 이유: R1이 페르소나를 입력으로 요구하므로 R0 페르소나 확보 단계 자체는 어떤 형태로든 필요하다. fast 모드는 *사용자 인터랙션*을 생략하지 단계 자체를 생략하지는 않는다. |
| Step 11 — `/discover-product` 권한 | (A) `allowed-tools: Read Glob Grep Write Edit` vs (B) `+Agent` 추가 | (B) 채택. skill 본문이 architect-opus 단발 sub-call을 요구하므로 Agent 권한 필요. |
| Step 9 — `/stabilize-milestone` 권한 | (A) `allowed-tools: Read Glob Grep Write Edit Bash` vs (B) `+Agent` 추가 | (B) 채택. skill 본문이 qa·reviewer agent 위임을 요구하므로 Agent 권한 필요. |
| Step 2 — `/plan-workitem` 권한 + planner agent tools | (A) skill `allowed-tools: Read Glob Grep` + planner.md `tools: Read, Glob, Grep` vs (B) 양쪽 모두 `Write Edit` 추가 | (B) 채택. skill 본문이 새 workitem 문서 생성·갱신을 요구하므로 Write/Edit 필요. |

---

## 사전 준비

본 가이드 적용 전에 한 번만 수행한다.

1. **현재 작업 트리가 깨끗한지 확인**:
   ```
   git status
   ```
   결과가 `nothing to commit, working tree clean`이어야 한다.

   > WSL/cross-mount 환경에서 `fatal: detected dubious ownership in repository`가 나오면 다음 한 줄로 해결:
   > ```
   > git config --global --add safe.directory <레포지토리 절대경로>
   > ```

2. **메인 브랜치 최신화**:
   ```
   git checkout main
   git pull --ff-only
   ```

3. **본 가이드 적용 전용 통합 브랜치 생성** (선택. Step별 브랜치를 만들 거면 생략):
   ```
   git checkout -b chore/improve-list-apply
   ```

4. **`docs/40-validation/reports/` 디렉터리는 Step 5에서 생성한다.** 사전 준비 단계에서 만들지 않는다.

---

## Step 1 — 모델 별칭 정책

### 1.1 목표

shared 기본값에 박힌 모델 ID를 별칭(`sonnet`, `opus`, `haiku`)으로 통일하고, 정책을 ADR로 보존한다.

### 1.2 의존성

없음. 즉시 적용 가능.

### 1.3 영향 범위

- 수정 4개 (`.claude/settings.json`, `.claude/agents/architect-opus.md`, `.claude/skills/bootstrap-project/SKILL.md`, `.claude/skills/bootstrap-stack/SKILL.md`)
- 신규 1개 (`docs/90-decisions/ADR-004-model-alias-policy.md`)
- 수정 1개 (`docs/00-meta/AGENT_EXECUTION_STRATEGY.md`)

### 1.4 변경 작업

#### 1.4.1 `.claude/settings.json`

**현재 내용 (3번째 줄)**:
```json
"model": "claude-sonnet-4-6",
```

**변경 후**:
```json
"model": "sonnet",
```

#### 1.4.2 `.claude/agents/architect-opus.md`

**현재 내용 (5번째 줄, frontmatter)**:
```yaml
model: claude-opus-4-6
```

**변경 후**:
```yaml
model: opus
```

#### 1.4.3 `.claude/skills/bootstrap-project/SKILL.md`

**현재 내용 (9번째 줄, frontmatter)**:
```yaml
model: claude-opus-4-6
```

**변경 후**:
```yaml
model: opus
```

#### 1.4.4 `.claude/skills/bootstrap-stack/SKILL.md`

**현재 내용 (9번째 줄, frontmatter)**:
```yaml
model: claude-opus-4-6
```

**변경 후**:
```yaml
model: opus
```

#### 1.4.5 `docs/90-decisions/ADR-004-model-alias-policy.md` (신규 작성)

**파일 전체 내용**:
```markdown
# ADR-004 모델 별칭 우선 정책

## 상태
accepted

## 배경
이 보일러플레이트의 핵심 가치는 "여러 프로젝트에서 반복 재사용"이다.
모델 ID를 전체 버전 표기(`claude-opus-4-6` 등)로 고정하면, fork된 새 프로젝트에서
시간이 지남에 따라 staleness가 누적되어 모델 갱신을 사람이 매번 기억해야 한다.

`.claude/settings.json`, agent 정의, skill 정의에 모델 표기가 흩어져 있고,
일부는 별칭(`sonnet`)을, 일부는 전체 ID를 사용하고 있어 표기 일관성이 없다.

## 결정
shared 기본값에서는 모델 별칭(`sonnet`, `opus`, `haiku`)만 사용한다.
특정 버전을 강제해야 하는 이유가 있으면 별도 ADR로 남기고 그 자리에서만 전체 ID를 사용한다.

## 근거
- Claude Code의 별칭은 자동 최신 매핑을 제공한다([model-config 문서](https://code.claude.com/docs/en/model-config)).
- 보일러플레이트의 "재사용 가능" 약속과 자동 최신 매핑이 가장 잘 맞는다.
- 사람이 모델 갱신을 잊어 staleness가 누적되는 것을 저비용으로 막는다.

## 결과
- `.claude/settings.json`: `"model": "sonnet"`
- `.claude/agents/architect-opus.md`: `model: opus`
- `.claude/skills/bootstrap-project/SKILL.md`, `.claude/skills/bootstrap-stack/SKILL.md`: `model: opus`
- 다른 sub-agent의 `model: sonnet` 표기는 그대로 유지(이미 별칭).

## 후속 작업
- 별칭 정책을 깰 때(특정 버전 강제)의 절차: 새 ADR로 이유와 갱신 책임자를 기록하고, 그 자리에서만 전체 ID를 사용한다.
- 사용자가 fork 직후 자기 환경의 비용 정책을 강제해야 하면 `.claude/settings.local.json`에서 model을 override한다.
- **Provider별 별칭 해석 차이 주의**: Anthropic API와 Bedrock/Vertex/Foundry에서 별칭이 매핑되는 ID·시점이 다를 수 있다. 특정 provider에서 재현성이 중요한 시점(릴리스 직전, 회계 감사 등)에는 그 provider 환경 변수 또는 settings 단에서 전체 ID로 임시 pinning한다(별도 ADR로 기록).
```

#### 1.4.6 `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`

**문서 끝(현재 77번째 줄 이후)에 다음 단락을 추가**:

```markdown

## 모델 표기 정책

shared 기본값에서는 모델 별칭(`sonnet`, `opus`, `haiku`)만 사용한다.
특정 버전을 강제해야 하면 ADR로 남기고 그 자리에서만 전체 ID를 사용한다.
정책 근거는 [ADR-004-model-alias-policy.md](../90-decisions/ADR-004-model-alias-policy.md)를 참조한다.
```

### 1.5 검증

- [ ] `.claude/settings.json`의 model 필드가 `"sonnet"`이다.
- [ ] `.claude/agents/architect-opus.md`의 frontmatter `model`이 `opus`다.
- [ ] `.claude/skills/bootstrap-project/SKILL.md`, `.claude/skills/bootstrap-stack/SKILL.md`의 frontmatter `model`이 `opus`다.
- [ ] `docs/90-decisions/ADR-004-model-alias-policy.md`가 존재한다.
- [ ] `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`에 "모델 표기 정책" 단락이 추가됐다.
- [ ] `git grep "claude-opus-4-6" -- ':!IMPROVE-GUIDE.md'` 결과가 비어 있다(전체 ID 표기가 모두 별칭으로 변경됐다. 본 가이드 자체에는 변경 전 문자열이 설명 목적으로 남아 있으므로 가이드 파일은 제외한다).
- [ ] `git grep "claude-sonnet-4-6" -- ':!IMPROVE-GUIDE.md'` 결과가 비어 있다.

### 1.6 커밋 권장

```
chore: switch model fields to aliases and add ADR-004
```

---

## Step 2 — 약한 skill 정리 + agent 바인딩

### 2.1 목표

agent 바인딩이 없어 사실상 "지침 한 페이지"로 동작하던 skill을 정리한다.
- `plan-workitem`, `review-doc`은 agent 바인딩을 추가해 실제 위임 흐름으로 강화.
- `write-charter`, `write-architecture`는 삭제(중복 진입점 제거).

### 2.2 의존성

없음. 즉시 적용 가능.

### 2.3 영향 범위

- 수정 2개 skill (`.claude/skills/plan-workitem/SKILL.md`, `.claude/skills/review-doc/SKILL.md`)
- 수정 1개 agent (`.claude/agents/planner.md` — Write/Edit tool 추가)
- 삭제 2개 디렉터리 (`.claude/skills/write-charter/`, `.claude/skills/write-architecture/`)
- 수정 4개 (`README.md`, `README_ko.md`, `docs/00-meta/WORKFLOW.md`, `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`)

### 2.4 변경 작업

#### 2.4.1 `.claude/skills/plan-workitem/SKILL.md` (전면 재작성)

**현재 본문 전체를 다음으로 교체**:

```markdown
---
name: plan-workitem
description: 상위 설계 문서를 기반으로 milestone, feature, task 단위 문서를 생성하거나 정리할 때 사용한다.
argument-hint: "[milestone or feature id]"
disable-model-invocation: true
allowed-tools: Read Glob Grep Write Edit
context: fork
agent: planner
---

너의 역할은 입력으로 받은 milestone/feature/task ID에 대한 workitem 문서를 분해·생성·갱신하는 것이다.

입력:
- `$ARGUMENTS`에는 milestone ID(예: `M1`), feature ID(예: `F-001`), 또는 자연어 분해 요청이 들어온다.

반드시 먼저 읽을 파일:
- `docs/10-charter/PROJECT_CHARTER.md`
- `docs/20-system/ARCHITECTURE_OVERVIEW.md`
- 입력 ID에 해당하는 상위 workitem 문서(있으면)
- `docs/30-workitems/_templates/MILESTONE_TEMPLATE.md`, `FEATURE_TEMPLATE.md`, `TASK_TEMPLATE.md`

반드시 수행할 일:
1. 입력 ID에 해당하는 상위 문서를 읽어 범위와 비범위를 파악한다.
2. 작업을 milestone, feature, task 중 적절한 레벨로 나눈다.
3. 각 문서의 범위와 비범위를 명확히 적는다.
4. 관련 문서 링크를 함께 기록한다.
5. 검증 포인트와 완료 기준을 포함한다.
6. 새 문서를 만들 때는 해당 레벨의 템플릿을 복사해 채운다.

반드시 지킬 원칙:
- 코드를 구현하지 않는다.
- 서로 다른 추상화 레벨을 한 문서에 섞지 않는다(milestone은 큰 목표, feature는 사용자 가치, task는 구현 단위).
- 하위 문서는 상위 문서를 링크한다.
- 확실하지 않은 내용은 가정으로 표시한다.
- 열린 질문이 남으면 문서에 명시한다.

마지막 출력:
- 생성·갱신한 문서 목록(상대 경로)
- 핵심 가정
- 남은 미결정 사항
- 다음 추천 단계(보통 `/implement-workitem [task-id]`)
```

> 주: 메인 세션이 분해 과정을 직접 보고 싶을 때는 본 skill 대신 메인에서 자연어로 planner agent에게 직접 위임하면 된다(skill을 거치지 않으므로 fork되지 않는다).

#### 2.4.2 `.claude/skills/review-doc/SKILL.md` (전면 재작성)

**현재 본문 전체를 다음으로 교체**:

```markdown
---
name: review-doc
description: 문서의 모호함, 누락, 모순, 숨은 복잡도를 비판적으로 검토할 때 사용한다.
argument-hint: "[doc path]"
disable-model-invocation: true
allowed-tools: Read Glob Grep
context: fork
agent: reviewer
---

너의 역할은 입력 경로의 문서를 비판적으로 검토하는 것이다.

입력:
- `$ARGUMENTS`에는 검토할 문서의 경로가 들어온다(예: `docs/10-charter/PROJECT_CHARTER.md`).

반드시 먼저 할 일:
1. 입력 경로의 문서를 읽는다.
2. 필요하면 관련 상위/하위 문서를 함께 읽어 맥락을 파악한다.

검토 항목:
- 누락된 요구사항
- 모순된 진술
- 모호하거나 검증 불가능한 표현
- 숨은 복잡도
- 빠진 엣지 케이스

마지막 출력:
- 결과를 P0, P1, P2로 나눈다.
- 어떤 섹션을 어떻게 수정하면 좋을지 구체적으로 제안한다.
- 상위 설계 문제와 하위 구현 문제를 구분한다.
- 막연한 칭찬은 하지 않는다.
- 시간/턴이 부족하면 확인된 범위까지의 핵심 판단만 요약하고 종료한다.
```

#### 2.4.3 `.claude/skills/write-charter/` 디렉터리 삭제

```
rm -r .claude/skills/write-charter
```
(PowerShell: `Remove-Item -Recurse -Force .claude/skills/write-charter`)

#### 2.4.4 `.claude/skills/write-architecture/` 디렉터리 삭제

```
rm -r .claude/skills/write-architecture
```

#### 2.4.5 `.claude/agents/planner.md` 갱신

planner agent의 `tools:` 필드에 `Write, Edit`를 추가한다(plan-workitem skill이 `context: fork`로 planner에 위임할 때 새 workitem 문서를 생성·갱신할 권한 필요).

**현재 내용 (4번째 줄)**:
```yaml
tools: Read, Glob, Grep
```

**변경 후**:
```yaml
tools: Read, Glob, Grep, Write, Edit
```

> 주: planner의 본분은 분해(코드 구현 금지) 그대로 유지된다. agent 본문 "코드를 구현하지 않는다" 규칙은 그대로 둔다 — Write/Edit는 *workitem 문서 생성/갱신*용이다.

#### 2.4.6 `docs/00-meta/AGENT_EXECUTION_STRATEGY.md` 갱신

**"## 위임 트리거" 표 (현재 22~33번째 줄)**: 표는 그대로 유지한다(Step 13에서 한 번에 갱신).

**"## 스킬 실행 순서 가이드" 섹션 (현재 64~76번째 줄)** 다음 직후에 새 섹션을 추가한다:

```markdown

## Mid-project 문서 갱신 동선

charter/architecture는 Living Doc로 분류돼 진행 중 재진입이 필요하다. 별도 skill 없이 다음 경로를 따른다.

| 갱신 종류 | 경로 |
|----------|------|
| charter 부분 갱신 | 자연어로 메인 세션에 변경 요청 → `planner` agent에 fork 위임 |
| charter 전면 재정의 | `/discover-product` 재실행(또는 산출물만 갱신) → `/bootstrap-project`로 charter 재생성 |
| architecture 스택 변경 | `/bootstrap-stack` 재실행 후 `/stack-guard` 이어 실행 |
| architecture 시스템 경계만 갱신 | 자연어 + `architect-opus` 단발 호출 |
```

> 주: `/discover-product`, `/stack-guard`는 이후 Step에서 도입된다. 동선 표는 미리 정의해 둔다.

#### 2.4.7 `docs/00-meta/WORKFLOW.md` 갱신

**"## 단계별 에이전트 위임" 섹션(현재 76번째 줄에 헤더, 75번째 줄은 빈 줄, 78번째 줄에 본문 한 줄) 직전에** 다음 단락을 추가한다(즉 75번째 빈 줄 자리부터):

```markdown

## Mid-project 문서 갱신 동선

charter/architecture/스택 관련 mid-project 갱신 경로는 [AGENT_EXECUTION_STRATEGY.md#mid-project-문서-갱신-동선](AGENT_EXECUTION_STRATEGY.md#mid-project-문서-갱신-동선)을 참조한다.

```

#### 2.4.8 `README.md` 및 `README_ko.md`

본 Step에서는 두 README의 "Overall Flow" / "전체 흐름" 행을 그대로 유지한다(Step 3에서 본문 슬림화, Step 12에서 흐름 교체). 단, 두 skill이 삭제되었으므로 README가 그 skill을 명시적으로 참조하면 안 된다 — 현재 README는 두 skill을 직접 참조하지 않으므로 추가 수정이 없다.

### 2.5 검증

- [ ] `.claude/skills/plan-workitem/SKILL.md`의 frontmatter에 `agent: planner`, `context: fork`, `disable-model-invocation: true`가 들어가 있고, `allowed-tools`에 `Write Edit`가 포함된다.
- [ ] `.claude/agents/planner.md`의 `tools:`에 `Write, Edit`가 추가됐다.
- [ ] `.claude/skills/review-doc/SKILL.md`의 frontmatter에 `agent: reviewer`, `context: fork`, `disable-model-invocation: true`가 들어가 있다.
- [ ] `.claude/skills/write-charter/`, `.claude/skills/write-architecture/` 디렉터리가 존재하지 않는다.
- [ ] `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`에 "Mid-project 문서 갱신 동선" 섹션이 있다.
- [ ] `docs/00-meta/WORKFLOW.md`가 mid-project 동선을 AGENT_EXECUTION_STRATEGY.md로 링크한다.

### 2.6 커밋 권장

```
chore: bind plan-workitem/review-doc to agents and remove orphan skills
```

---

## Step 3 — SSOT 인프라 도입

### 3.1 목표

같은 사실이 5곳까지 흩어져 있는 현 상태를 정리해, 후속 Step(4·5·6 등)이 만들 신규 ADR·산출물·정책이 SSOT 패턴을 따르도록 전제 인프라를 깐다.

이 Step은 큰 사이즈이며 5개 sub-step으로 나누어 진행한다.

### 3.2 의존성

- Step 1 (모델 별칭) 완료 — Step 3에서 frontmatter의 `model` 라인이 별칭으로 정리되어 있어야 SSOT 정의가 모순되지 않는다.
- Step 2 (약한 skill 정리) 완료 — write-charter/write-architecture가 삭제되어 있어야 STRUCTURE.md 인벤토리가 정확해진다.

### 3.3 영향 범위

- 신규 3개 (`docs/00-meta/STRUCTURE.md`, `docs/90-decisions/README.md`, `docs/90-decisions/ADR-005-ssot.md`)
- 슬림화 8개 (`CLAUDE.md`, `README.md`, `README_ko.md`, `docs/00-meta/TEMPLATE_GUIDE.md`, `docs/30-workitems/README.md`, `.claude/skills/boilerplate-context/SKILL.md`, `docs/00-meta/GLOSSARY.md`, `docs/90-decisions/_ADR_GUIDE.md`)
- 끊김 보강 2개 (`docs/30-workitems/_templates/TASK_TEMPLATE.md`, `docs/30-workitems/_templates/FEATURE_TEMPLATE.md`)

### 3.4 sub-step A — 신규 인덱스 문서 작성

#### 3.4.1 `docs/00-meta/STRUCTURE.md` 신규

**파일 전체 내용**:

```markdown
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
| plan (Claude Code Plan 모드) | `docs/30-workitems/plans/` | Claude Code Plan 모드 | ephemeral |
| validation report | `docs/40-validation/reports/<task-id>.md` | `/validate-workitem` | ephemeral |
| qa findings | `docs/40-validation/QA_FINDINGS.md` | `/stabilize-milestone` (mile별 누적) | Record |
| improvement guide | `docs/40-validation/IMPROVEMENT_GUIDE.md` | `/stabilize-milestone` | Living |
| ADR | `docs/90-decisions/ADR-*.md` (인덱스: `docs/90-decisions/README.md`) | architect-opus, `/bootstrap-project` 등 | Record |
| stack setup plan | `docs/00-meta/STACK_SETUP_PLAN.md` | `/bootstrap-stack`, `/stack-guard` | Reference |
| verify scripts | `scripts/verify.{sh,ps1,mjs,py}` | `/stack-guard` | Reference |

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
| 단순성·YAGNI·Clean Code/Architecture 정책 | `docs/90-decisions/ADR-006-simplicity-and-architecture.md` + `CLAUDE.md`(요약) |
| TDD 정책 | `docs/90-decisions/ADR-009-tdd-default.md` + `CLAUDE.md`(1줄) |
| 워크아이템 라이프사이클 | `docs/90-decisions/ADR-007-workitem-lifecycle.md` |
| Conventional Commits | `docs/90-decisions/ADR-008-commit-convention.md` |
| 산출물 위치 인벤토리 | 본 문서(`docs/00-meta/STRUCTURE.md`) |
| ADR 인덱스 | `docs/90-decisions/README.md` |

## 절차

### 새 산출물 도입 시
1. 산출물 표에 한 줄 추가(위치, 생성 주체, 라이프사이클).
2. 라이프사이클이 Record/ephemeral이면 `.gitignore` 처리 여부도 함께 판단.

### 새 정책 도입 시
1. ADR을 만든다 — 정책 본문은 ADR이 SSOT.
2. `docs/90-decisions/README.md`에 한 줄 추가.
3. 관련 agent/skill 본문에는 정책 설명 대신 ADR 링크 + 자기 영역 행동 규율(self-check 등)만 둔다.
4. canonical owner 매핑이 변하면 본 문서의 Canonical Owner 표 갱신.
```

> 가이드 적용 시 주의: 위 STRUCTURE.md 본문의 Canonical Owner 매핑은 ADR-006/007/008/009 경로를 미리 가리킨다. 본 Step 3 완료 시점에는 ADR-005까지만 존재하므로 ADR-006~009 링크는 일시적으로 dangling이다. Step 4(ADR-006), Step 6(ADR-007/008), Step 8(ADR-009) 완료 후 모두 해소된다.

#### 3.4.2 `docs/90-decisions/README.md` 신규

**파일 전체 내용**:

```markdown
# ADR Index

> 이 디렉터리의 모든 ADR을 한눈에 본다. 새 ADR 추가 시 이 표에 한 줄을 추가하는 것이 기본 절차다(상세는 [_ADR_GUIDE.md](_ADR_GUIDE.md) 참조).

| # | 제목 | 상태 | 한 줄 요약 |
|---|------|------|-----------|
| 001 | Doc hierarchy | accepted | docs/ 디렉터리 6분할 결정 |
| 002 | Initial project decisions | (placeholder, `/bootstrap-project`가 새 프로젝트에서 생성) | bootstrap 단계의 초기 결정 모음 |
| 003 | Stack selection | (placeholder, `/bootstrap-stack`가 새 프로젝트에서 생성) | 스택 선택과 근거 |
| 004 | Model alias policy | accepted | shared 기본값에서 모델 별칭(`sonnet`, `opus`, `haiku`)만 사용 |
| 005 | Single Source of Truth (SSOT) | accepted | 같은 사실은 1곳에서 정의, 다른 곳은 한 줄 + 링크. 정책=ADR 패턴. |

## 신규 ADR 추가 절차
1. `_ADR_GUIDE.md`의 "권장 섹션"을 따라 ADR 본문 작성.
2. ADR 번호는 가장 큰 기존 번호 + 1.
3. 본 README의 표에 한 줄 추가(번호, 제목, 상태, 한 줄 요약).
4. 관련 agent/skill 본문에 ADR 링크를 박는다.
```

> 주: ADR-005는 본 Step의 sub-step C에서 작성한다. 위 표는 본 Step 완료 시점 기준 — Step 4·6·8 완료 후에는 ADR-006/007/008/009 행이 더 추가된다(각 Step에서 추가 작업 명시).

#### 3.4.3 `docs/90-decisions/ADR-005-ssot.md` 신규

**파일 전체 내용**:

```markdown
# ADR-005 단일 출처(SSOT) 원칙

## 상태
accepted

## 배경
이 보일러플레이트의 모든 가치(fork 후 안정성, 변경 비용 최소화, 사용자 혼선 최소화)는 같은 사실이 한 곳에서 정의되고 다른 곳은 그것을 참조한다는 단일 출처(Single Source of Truth, 이하 SSOT) 원칙에 달려 있다.

본 ADR 이전 시점의 정적 분석에서 다음과 같은 다중 정의가 발견됐다:
- 문서 계층 정의: 5곳 (CLAUDE.md, TEMPLATE_GUIDE.md, boilerplate-context skill, README.md, README_ko.md)
- 위임 트리거 표: 2곳 (CLAUDE.md, AGENT_EXECUTION_STRATEGY.md, planner 행 누락 모순 포함)
- 상태값 + 전이 규칙: 4곳
- 모델 ID/별칭 표기: 4곳
- Bootstrap 사용 흐름: 5곳

표현 차이가 누적되면 사용자 혼선을 만들고, 변경 시 어느 한 곳이라도 빠뜨리면 stale로 노출된다.

## 결정
다음 다섯 패턴을 보일러플레이트의 SSOT 운영 방식으로 채택한다.

1. **"정의 1곳, 다른 곳은 링크" 패턴** — 각 사실은 단 하나의 canonical 문서에 정의된다. 다른 문서는 한 줄 요약 + canonical 링크만 둔다. 정의 본문을 복제하지 않는다.
2. **"인덱스 README" 패턴** — `docs/90-decisions/README.md`가 ADR 인덱스를 담는다. 새 ADR 추가 시 README 갱신이 기본 절차.
3. **"산출물 인벤토리" 패턴** — `docs/00-meta/STRUCTURE.md`가 모든 산출물의 위치·생성 주체·라이프사이클을 단일 표로 관리. 신규 산출물은 이 표에 등록.
4. **"정책 = ADR" 패턴** — 보일러플레이트가 도입하는 모든 정책(모델 별칭, 단순성, TDD, commit convention, lifecycle, SSOT 등)은 ADR로 박고, agent/skill 본문에는 정책 설명을 길게 박지 않는다.
5. **"CLAUDE.md = 진입 페이지" 패턴** — `CLAUDE.md`는 fork된 새 세션이 자동 로드하는 진입점이다. 모든 운영 원칙을 다 박지 않고 목적·권위 있는 문서로의 링크 인덱스·핵심 행동 규율만 둔다.

Canonical Owner 매핑 표는 `docs/00-meta/STRUCTURE.md`의 "Canonical Owner 매핑(SSOT 부록)" 섹션이 SSOT다.

## 근거
- 변경 비용 비대칭 해소 — 모델 별칭 1개 바꿀 때 4 surface 동시 변경하던 것이 1곳 변경으로 끝난다.
- 표현 drift 방지 — 같은 디렉터리를 어떤 곳은 "운영 원칙"이라 부르고 다른 곳은 "guardrail 철학"이라 부르던 혼선이 사라진다.
- 정책 권위 강화 — 정책이 ADR에 박히면 6개월 뒤 fork한 사용자가 "왜 이 정책인가"를 ADR로 추적할 수 있다.
- README cross-language drift 표면 축소 — README 본문이 짧아지면 ko/en 동기화 부담이 줄어든다.

## 결과
- `docs/00-meta/STRUCTURE.md` 신설.
- `docs/90-decisions/README.md` 신설.
- `CLAUDE.md`, `README.md`, `README_ko.md`, `docs/00-meta/TEMPLATE_GUIDE.md` 등이 슬림화되어 정의 대신 링크를 사용한다.
- agent 본문은 행동 규율 + ADR 링크 형태로 정리된다.

## 후속 작업
- 새 정책이 도입될 때마다 ADR로 박고 README 인덱스에 한 줄 추가.
- 새 산출물이 도입될 때마다 STRUCTURE.md 인벤토리에 등록.
- agent/skill 본문에 정책 설명이 길게 들어 있으면 ADR 링크로 줄이는 후속 정리.
- `/stabilize-milestone`이 SSOT drift 점검을 수행하도록 후속 항목 검토.
```

### 3.5 sub-step B — 슬림화

> 핵심 원칙: 정의 본문을 복제하지 않고, 한 줄 요약 + canonical 링크로 줄인다.

#### 3.5.1 `CLAUDE.md` 슬림화

**현재 파일 전체를 다음으로 교체** (약 57줄 → 약 22줄, Step 4·8에서 단순성·TDD 단락 추가로 최종 약 30~40줄):

```markdown
# 프로젝트 지침

## 목표
- 이 저장소는 Claude Code 기반의 재사용 가능한 문서 중심 개발 보일러플레이트다.
- 새 프로젝트를 시작할 때 이 구조를 복제해 빠르게 적용할 수 있어야 한다.
- 중요한 단계마다 해당 계층의 문서를 먼저 갱신한 뒤 다음 단계로 진행한다.

## 핵심 행동 규율
- 상위 문서 없이 하위 문서를 먼저 만들지 않는다.
- `.env`, `secrets/` 같은 민감 파일은 건드리지 않는다.
- 작업 범위와 비범위를 명확히 적고, 범위 밖 변경은 하지 않는다.
- 흩어진 임시 메모보다 정해진 위치의 문서를 갱신한다.
- 사실, 가정, 열린 질문을 구분해서 적는다. 검증 가능한 표현을 우선한다.
- 커밋은 작고 논리적인 단위로 나눈다. 커밋 전에 관련 workitem 문서와 구현 범위가 일치하는지 확인한다.

## 깊은 운영 원칙은 다음 문서를 따른다
- [문서 계층과 산출물 인벤토리](docs/00-meta/STRUCTURE.md)
- [문서 계층 정의 + 네이밍](docs/00-meta/TEMPLATE_GUIDE.md)
- [워크플로우 + 문서 상태 전이](docs/00-meta/WORKFLOW.md)
- [에이전트 실행 전략 + 위임 트리거](docs/00-meta/AGENT_EXECUTION_STRATEGY.md)
- [Guardrail 운영 원칙](docs/00-meta/GUARDRAILS_STRATEGY.md)
- [ADR 인덱스](docs/90-decisions/README.md)
```

> 주: 본 sub-step 적용 직후 CLAUDE.md는 약 22줄이다. Step 4(단순성 5개 항목 + 도입부 + ADR 링크 ≈ 8줄)와 Step 8(TDD 1줄 + 도입부 + ADR 링크 ≈ 4줄)에서 추가되어 최종 약 30~40줄. Step 13의 CLAUDE.md 변경은 조건부이며 보통 추가 라인이 발생하지 않는다(13.4.2 참조).

#### 3.5.2 `README.md` 슬림화

**"## Overall Flow" 섹션 (60~64번째 줄) 교체**:

기존:
```
## Overall Flow

\`\`\`
/bootstrap-project → /bootstrap-stack → /plan-workitem → /implement-workitem → /validate-workitem → qa/reviewer
\`\`\`

For details on each step, see:

- [Workflow](docs/00-meta/WORKFLOW.md)
- [Agent Execution Strategy](docs/00-meta/AGENT_EXECUTION_STRATEGY.md)
```

변경 후 (한 줄 그림은 유지하되 정의는 링크):
```
## Overall Flow

\`\`\`
/bootstrap-project → /bootstrap-stack → /plan-workitem → /implement-workitem → /validate-workitem → qa/reviewer
\`\`\`

For step-by-step details, see [WORKFLOW.md](docs/00-meta/WORKFLOW.md).
For sub-agent delegation, see [AGENT_EXECUTION_STRATEGY.md](docs/00-meta/AGENT_EXECUTION_STRATEGY.md).
```

> 주: 한 줄 그림은 README의 진입 가치를 위해 유지(STRUCTURE.md 12.7 결정). Step 6·11·13 적용 후 풀 흐름으로 갱신.

**"## Structure" 섹션 (82~99번째 줄) 교체**:

기존 디렉터리 트리 전체를 다음 한 단락으로 줄인다:
```
## Structure

For a full inventory of all artifacts (location, owner, lifecycle), see [STRUCTURE.md](docs/00-meta/STRUCTURE.md).

\`\`\`
.
├── CLAUDE.md          # Shared project instructions
├── .claude/           # Sub-agents, skills, settings
├── docs/              # Documentation (charter, system, workitems, validation, decisions)
└── scripts/           # Project-specific automation (after stack is chosen)
\`\`\`
```

**"## Guardrail Principles" 섹션 (101~107번째 줄) 교체**:

```
## Guardrail Principles

This template prioritizes cross-platform reusability — shared base settings do not include OS/shell/runtime-dependent hooks. See [GUARDRAILS_STRATEGY.md](docs/00-meta/GUARDRAILS_STRATEGY.md) for details.
```

**README 최상단(1번째 줄 직전)에** 다음 코멘트를 추가한다:
```
<!-- 구조 변경 시 README.md와 README_ko.md를 동시에 갱신한다. drift를 막기 위해 두 README 본문은 짧게 유지하고 깊은 정의는 docs/ 링크로 둔다. -->
```

#### 3.5.3 `README_ko.md` 슬림화

**"## 전체 흐름" 섹션 (60~69번째 줄)을** README.md와 동일한 구조의 한국어 버전으로 교체:
```
## 전체 흐름

\`\`\`
/bootstrap-project → /bootstrap-stack → /plan-workitem → /implement-workitem → /validate-workitem → qa/reviewer
\`\`\`

각 단계 상세는 [WORKFLOW.md](docs/00-meta/WORKFLOW.md), 서브에이전트 위임은 [AGENT_EXECUTION_STRATEGY.md](docs/00-meta/AGENT_EXECUTION_STRATEGY.md)를 참조한다.
```

**"## 구조" 섹션 (82~99번째 줄)**을 영문판과 동일한 구조로 교체:
```
## 구조

산출물 전체 인벤토리(위치·생성 주체·라이프사이클)는 [STRUCTURE.md](docs/00-meta/STRUCTURE.md)를 참조한다.

\`\`\`
.
├── CLAUDE.md          # 프로젝트 공통 지침
├── .claude/           # 서브에이전트, 스킬, 설정
├── docs/              # 문서 (charter, system, workitems, validation, decisions)
└── scripts/           # 프로젝트별 자동화 (스택 확정 후 추가)
\`\`\`
```

**"## Guardrail 원칙" 섹션 (101~107번째 줄) 교체**:
```
## Guardrail 원칙

이 템플릿은 cross-platform 재사용성을 우선한다 — shared 기본값에 OS/셸/런타임 종속적인 hook를 포함하지 않는다. 자세한 내용은 [GUARDRAILS_STRATEGY.md](docs/00-meta/GUARDRAILS_STRATEGY.md)를 참고한다.
```

**README_ko.md 최상단(1번째 줄 직전)에** 다음 코멘트를 추가한다:
```
<!-- 구조 변경 시 README.md와 README_ko.md를 동시에 갱신한다. drift를 막기 위해 두 README 본문은 짧게 유지하고 깊은 정의는 docs/ 링크로 둔다. -->
```

#### 3.5.4 `docs/00-meta/TEMPLATE_GUIDE.md` 슬림화

**"## 새 프로젝트 시작 순서" 섹션 (15~21번째 줄) 전체 제거** 후 다음 한 줄로 대체:
```markdown
## 새 프로젝트 시작 순서
새 프로젝트 시작 체크리스트는 [NEW_PROJECT_CHECKLIST.md](NEW_PROJECT_CHECKLIST.md)가 SSOT다. 본 문서에서는 정의를 복제하지 않는다.
```

**"## 상태값 예시" 섹션 (34~40번째 줄) 전체 제거** 후 다음 한 줄로 대체:
```markdown
## 문서 상태값
workitem 상태값과 전이 규칙은 [WORKFLOW.md#문서-상태-전이](WORKFLOW.md#문서-상태-전이)가 SSOT다.
ADR 전용 상태값(`proposed`/`accepted`/`superseded`/`deprecated`)은 [_ADR_GUIDE.md](../90-decisions/_ADR_GUIDE.md)가 SSOT다.
```

**"## Guardrail 운영 원칙" 섹션 (47~52번째 줄) 전체 제거** 후 다음 한 줄로 대체:
```markdown
## Guardrail 운영 원칙
shared 기본값과 stack-specific 자동화의 분리 원칙은 [GUARDRAILS_STRATEGY.md](GUARDRAILS_STRATEGY.md)가 SSOT다.
```

**"## 권장 시작 방식" 섹션 (57~61번째 줄) 직전에** 새 섹션 추가:
```markdown
## 산출물 인벤토리
모든 산출물의 위치·생성 주체·라이프사이클은 [STRUCTURE.md](STRUCTURE.md)가 SSOT다.

```

#### 3.5.5 `docs/30-workitems/README.md` 슬림화

**"## 상태 관리 예시" 섹션 (23~29번째 줄) 전체 제거** 후 다음 한 줄로 대체:
```markdown
## 상태 관리
workitem 상태값과 전이 규칙은 [../00-meta/WORKFLOW.md#문서-상태-전이](../00-meta/WORKFLOW.md#문서-상태-전이)가 SSOT다.
```

**"## 디렉터리 구성" 섹션 중 plans 항목**(현재 10번째 줄)을 다음으로 교체:
```markdown
- `plans`: Claude Code Plan 모드가 자동 생성하는 실행 계획. 경로는 `.claude/settings.json`의 `plansDirectory`가 canonical이다(현재 `./docs/30-workitems/plans`).
```

#### 3.5.6 `.claude/skills/boilerplate-context/SKILL.md` 슬림화

**현재 파일 전체를 다음으로 교체** (26줄 → 약 14줄):

```markdown
---
name: boilerplate-context
description: Use when working in this repository to understand the boilerplate's layered documentation system, workitem flow, and guardrail philosophy.
user-invocable: false
---

이 저장소는 Claude Code용 문서 중심 보일러플레이트다.

핵심 구조와 산출물 인벤토리: [docs/00-meta/STRUCTURE.md](../../../docs/00-meta/STRUCTURE.md)
워크플로우와 문서 상태 전이: [docs/00-meta/WORKFLOW.md](../../../docs/00-meta/WORKFLOW.md)
에이전트 위임 전략: [docs/00-meta/AGENT_EXECUTION_STRATEGY.md](../../../docs/00-meta/AGENT_EXECUTION_STRATEGY.md)
Guardrail 원칙: [docs/00-meta/GUARDRAILS_STRATEGY.md](../../../docs/00-meta/GUARDRAILS_STRATEGY.md)

기본 진입점:
- 프로젝트 초기 세팅이 필요하면 `/bootstrap-project`를 우선 사용한다.
- 스택 자동화 세팅이 필요하면 `/bootstrap-stack`을 사용한다.
```

#### 3.5.7 `docs/00-meta/GLOSSARY.md` 슬림화

**현재 파일 전체를 다음으로 교체**:

```markdown
# 용어집

## 목적
프로젝트에서 반복적으로 쓰는 핵심 용어를 일관되게 정의한다.

## 보일러플레이트 단계의 운영 원칙
보일러플레이트 단계에서는 의도적으로 비워둔다. fork된 새 프로젝트의 도메인 용어가 생기면 여기에 정의한다.

## 작성 원칙
- 한 용어에는 하나의 대표 의미만 둔다.
- 서로 비슷한 용어는 차이를 함께 적는다.
- 문서 전반에서 같은 표현을 유지한다.
```

#### 3.5.8 `docs/90-decisions/_ADR_GUIDE.md` 갱신

**"## 권장 섹션" 섹션 (22~28번째 줄) 직후, "## 참고" 직전에** 새 섹션 추가:

```markdown
## 새 ADR 추가 절차
1. ADR 본문을 작성한다(번호는 가장 큰 기존 번호 + 1).
2. [README.md](README.md) 인덱스 표에 한 줄 추가(번호, 제목, 상태, 한 줄 요약).
3. 관련 agent/skill 본문에 ADR 링크를 박는다(정책 설명을 길게 박지 않는다).

```

### 3.6 sub-step C — 끊김 보강 (TASK / FEATURE 템플릿)

#### 3.6.1 `docs/30-workitems/_templates/TASK_TEMPLATE.md`

**"## 7. 관련 문서" 섹션 (18~22번째 줄)을** 다음으로 교체:

```markdown
## 7. 관련 문서
- Milestone: <!-- 예: [M1-foundation](../milestones/M1-foundation.md) -->
- Feature: <!-- 예: [F-001-core-value](../features/F-001-core-value.md) -->
- Architecture: <!-- 예: [ARCHITECTURE_OVERVIEW](../../20-system/ARCHITECTURE_OVERVIEW.md) -->
- ADR: <!-- 예: [ADR-007-workitem-lifecycle](../../90-decisions/ADR-007-workitem-lifecycle.md) -->
```

#### 3.6.2 `docs/30-workitems/_templates/FEATURE_TEMPLATE.md`

**"## 9. 관련 문서" 섹션 (22~26번째 줄)을** 다음으로 교체:

```markdown
## 9. 관련 문서
- Milestone: <!-- 예: [M1-foundation](../milestones/M1-foundation.md) -->
- Charter: <!-- 예: [PROJECT_CHARTER](../../10-charter/PROJECT_CHARTER.md) -->
- Architecture: <!-- 예: [ARCHITECTURE_OVERVIEW](../../20-system/ARCHITECTURE_OVERVIEW.md) -->
- ADR: <!-- 예: [ADR-007-workitem-lifecycle](../../90-decisions/ADR-007-workitem-lifecycle.md) -->
```

### 3.7 sub-step D — `docs/00-meta/AGENT_EXECUTION_STRATEGY.md` SSOT 강화

본 문서는 위임 트리거의 SSOT다. 다른 문서에서 위임 트리거를 정의하면 안 된다(CLAUDE.md는 Step 13에서 정리).

본 Step에서 추가 변경은 없다(Step 13에서 위임 트리거 표 갱신).

### 3.8 검증

- [ ] `docs/00-meta/STRUCTURE.md`가 존재하고 산출물 인벤토리 표 + Canonical Owner 매핑이 들어 있다.
- [ ] `docs/90-decisions/README.md`가 존재하고 ADR-001~005 표가 들어 있다.
- [ ] `docs/90-decisions/ADR-005-ssot.md`가 존재한다.
- [ ] `CLAUDE.md`가 약 22줄 안팎으로 슬림화됐고 STRUCTURE/WORKFLOW/AGENT_EXECUTION_STRATEGY 등 링크를 가리킨다(Step 4·8에서 단순성·TDD 단락이 추가되어 최종은 약 30~40줄이 된다).
- [ ] `README.md`, `README_ko.md`의 Structure 섹션이 STRUCTURE.md 링크 포함으로 줄어들었다.
- [ ] `docs/00-meta/TEMPLATE_GUIDE.md`가 "새 프로젝트 시작 순서"를 NEW_PROJECT_CHECKLIST 링크로 대체했다.
- [ ] `docs/30-workitems/README.md`의 "상태 관리 예시"가 WORKFLOW 링크로 대체됐다.
- [ ] `.claude/skills/boilerplate-context/SKILL.md`가 14줄 안팎으로 슬림화됐다.
- [ ] `docs/00-meta/GLOSSARY.md`가 의도적 비우기 단락을 포함한다.
- [ ] `docs/90-decisions/_ADR_GUIDE.md`에 "새 ADR 추가 절차" 섹션이 추가됐다.
- [ ] TASK_TEMPLATE, FEATURE_TEMPLATE의 "관련 문서" 섹션에 상대 경로 placeholder 예시가 들어 있다.
- [ ] `git grep -l "draft.*ready.*in-progress"` 결과가 `docs/00-meta/WORKFLOW.md`만 남는다(상태값 정의 SSOT가 한 곳).

### 3.9 커밋 권장

sub-step별로 분리해도 좋고, 한 번에 커밋해도 무방.

```
docs: introduce SSOT infrastructure (STRUCTURE/ADR index/ADR-005 + slim CLAUDE/README)
```

---

## Step 4 — 단순성·Clean Code 가드레일 1단계

### 4.1 목표

사용자의 핵심 요구("요구한 것만 단순하게, 오버하지 않으면서 가독성 우선")를 fork-time surfaces(`CLAUDE.md`, `.claude/agents/*`)에 박는다.

이 Step의 1단계(11a)는 의존성 없이 즉시 적용 가능하다. 2단계(11b — Architecture 의존성 규칙)는 Step 10에서 처리.

### 4.2 의존성

- Step 3 (SSOT 인프라) 완료 — CLAUDE.md가 슬림화된 상태에서 단순성 단락을 추가한다. ADR-006이 SSOT 패턴을 따라야 한다.

### 4.3 영향 범위

- 수정 1개 (`CLAUDE.md`)
- 수정 4개 agent (`.claude/agents/builder-sonnet.md`, `validator-sonnet.md`, `reviewer.md`, `architect-opus.md`)
- 수정 1개 skill (`.claude/skills/implement-workitem/SKILL.md`)
- 신규 1개 (`docs/90-decisions/ADR-006-simplicity-and-architecture.md`)
- 갱신 1개 (`docs/90-decisions/README.md` 인덱스에 ADR-006 추가)

### 4.4 변경 작업

#### 4.4.1 `CLAUDE.md`에 단순성 5개 항목 추가

**Step 3에서 슬림화된 CLAUDE.md의 "## 핵심 행동 규율" 섹션 직후에** 새 섹션 추가:

```markdown

## 단순성·YAGNI (구현 시 항상 적용)
- 요구한 범위만 구현한다. 추측성 추상화·미래 대비 코드·계획에 없는 헬퍼는 만들지 않는다.
- 동일 패턴이 3회 이상 반복될 때까지는 추출하지 않는다. 비슷한 코드 3줄이 premature abstraction보다 낫다.
- 시스템 경계(외부 입력, 외부 API)에서만 입력 검증·에러 핸들링을 둔다. 내부 호출에는 두지 않는다.
- WHY가 비자명할 때만 주석을 단다(숨은 제약, 미묘한 invariant, 특정 버그 우회). WHAT 주석은 좋은 식별자 이름으로 대체한다.
- backwards-compat shim, feature flag, 사용 안 되는 변수의 `_` rename 같은 호환 hack을 만들지 않는다. 정말 안 쓰면 삭제한다.

정책 근거: [ADR-006-simplicity-and-architecture.md](docs/90-decisions/ADR-006-simplicity-and-architecture.md).
```

#### 4.4.2 `.claude/agents/builder-sonnet.md` 갱신

**"규칙:" 섹션 (현재 23~32번째 줄) 다음에** 새 단락을 추가한다:

```markdown

단순성 self-check (구현 출력 직전 점검):
- 추가한 추상화·팩토리·헬퍼가 정말 2회 이상 사용되는가?
- 추가한 try/except·null check가 시스템 경계에서 발생하는가, 아니면 내부 호출인가?
- 새 주석이 WHY를 설명하는가, WHAT을 설명하는가?
- 삭제 가능한 dead code(쓰이지 않는 import·변수·branch)가 남았는가?

self-check를 통과하지 못한 항목은 출력의 "남은 정리 항목"에 명시한다.
정책 근거: [ADR-006](../../docs/90-decisions/ADR-006-simplicity-and-architecture.md).
```

**기존 출력 형식("구현 후 아래를 짧게 요약한다")의 항목 목록**(현재 26번째 줄의 도입부 + 27~30번째 줄의 4개 sub-bullet)에 한 줄 추가:
- 기존: `- 수정 파일`, `- 핵심 변경 사항`, `- 테스트/검증 여부`, `- 남은 리스크 또는 미결정 사항`
- 추가: `- 남은 정리 항목 (단순성 self-check 미통과)`

#### 4.4.3 `.claude/agents/validator-sonnet.md` 갱신

**"규칙:" 섹션 (현재 30~34번째 줄)에** 다음 한 줄 추가:

```markdown
- 범위 밖 추상화·premature factory·미사용 dead code가 보이면 출력에 명시한다(Clean Code 정책: ADR-006).
```

#### 4.4.4 `.claude/agents/reviewer.md` 갱신

**"규칙:" 섹션 (현재 17~22번째 줄) 다음에** 새 섹션 추가:

```markdown

Clean Code 6항목 체크리스트 (호출될 때마다 적용):
1. **Naming** — 함수·변수·파일 이름이 의도를 표현하는가. 약자·번호·`util`/`helper`/`manager` 같은 무의미한 이름 회피.
2. **Function size + single responsibility** — 한 함수가 한 가지를 하는가. 여러 추상화 수준을 섞지 않는가.
3. **Duplication** — 3회 이상 반복되는 패턴인가. 1~2회는 인라인 유지.
4. **Premature abstraction** — 1~2회 사용에 그치는 abstraction이 있는가(인라인 후보).
5. **Comment policy** — WHAT 주석이 있는가(이름으로 대체할 후보). WHY 주석이 누락된 invariant가 있는가.
6. **Layer leak** — 의존성 규칙(있으면) 위반이 있는가. 상위 레이어가 하위 모듈을 직접 의존하지 않는가.

P0/P1/P2 분류와 함께 위 6항목 중 어디에 해당하는지 라벨링한다(예: `P1 [Duplication] auth.ts:42 — 같은 정규화 로직이 3곳에 반복`).

정책 근거: [ADR-006](../../docs/90-decisions/ADR-006-simplicity-and-architecture.md).
```

#### 4.4.5 `.claude/agents/architect-opus.md` 갱신

**"규칙:" 섹션 (현재 23~28번째 줄)에** 다음 한 줄 추가:

```markdown
- 큰 설계 결정 시 "프로젝트 규모가 4-layer 등 다층 아키텍처를 정당화하는가" self-check를 한다. 정당화되지 않으면 단일 layer + 모듈 단위 의존성 규칙을 권장한다(정책: ADR-006, 단순성 1순위 → Clean Code 2순위 → Clean Architecture 3순위).
```

#### 4.4.6 `.claude/skills/implement-workitem/SKILL.md` 갱신

**"구현 원칙:" 섹션 (현재 21~25번째 줄) 마지막에** 한 줄 추가:

```markdown
- 구현 직전·직후 단순성 self-check 4항목과 Clean Code 6항목을 참조한다(정책: ADR-006).
```

> 주: Red→Green→Refactor 3 phase로 본문을 분리하는 변경은 Step 8에서 수행한다. Step 8.4.3은 본 파일을 전면 재작성하며, 그 본문에 단순성 self-check + Clean Code 6항목 참조 라인이 다시 포함된다(따라서 본 줄 추가는 Step 8 적용 시점까지의 잠정 patch다).

#### 4.4.7 `docs/90-decisions/ADR-006-simplicity-and-architecture.md` 신규

**파일 전체 내용**:

```markdown
# ADR-006 단순성·Clean Code·Clean Architecture 우선순위

## 상태
accepted

## 배경
이 보일러플레이트의 핵심 가치는 fork된 미래 프로젝트에서 AI 에이전트가 "요구한 범위만, 오버엔지니어링 없이, 가독성 좋게" 코드를 작성하게 만드는 것이다.

이 요구는 다음 세 층으로 분해된다.
1. **단순성·YAGNI** — 요구한 범위만 구현, 추측성 추상화 금지, 미래 대비 코드 금지.
2. **Clean Code** — 명확한 네이밍, 한 함수 한 가지 일, 중복 회피, WHY 주석.
3. **Clean Architecture** — 의존성 규칙, 레이어 경계 — 단 프로젝트 규모가 정당화할 때만.

순서가 중요하다. Clean Architecture를 단순성 위에 두면 작은 prototype에 4-layer를 강제해 사용자의 "오버하지 않으면서"와 직접 충돌한다.

## 결정
보일러플레이트는 위 세 층을 다음 우선순위로 적용한다(우선순위가 높을수록 강하게 강제한다).

1. **단순성·YAGNI (1순위)** — 모든 코드 변경에 강제 적용.
2. **Clean Code 6항목 (2순위)** — reviewer agent가 호출될 때마다 라벨링.
3. **Clean Architecture (3순위)** — 모듈이 3개 이상이거나 프로젝트 규모가 정당화할 때만 적용.

각 층의 surface 매핑:

| 층 | Surface | 형태 |
|----|---------|------|
| 단순성·YAGNI | `CLAUDE.md` | 5개 항목, fork된 새 세션이 자동 로드. |
| 단순성·YAGNI | `.claude/agents/builder-sonnet.md` | 구현 출력 직전 self-check 4항목. 미통과 항목은 "남은 정리 항목"에 명시. |
| Clean Code 6항목 | `.claude/agents/reviewer.md` | 6항목 체크리스트, P0/P1/P2 + 항목 라벨링. |
| Clean Architecture | `.claude/agents/architect-opus.md` | "프로젝트 규모가 정당화하는가" self-check. |
| Clean Architecture | `docs/20-system/ARCHITECTURE_OVERVIEW.md`의 `## 3-1. 레이어 경계 + 의존성 규칙` | 모듈 ≥3 시 채울 것을 권장. |

## 근거
- 사용자가 명시적으로 "오버엔지니어링 회피·가독성 우선"을 핵심 가치로 강조했다.
- prototype 단계에서 4-layer를 강제하면 ceremony가 가치를 압도한다 — 단순성을 1순위로 두는 이유.
- Clean Code는 라벨링 비용이 낮으므로 reviewer가 매번 적용해도 비용 부담이 작다.
- Clean Architecture는 적용 비용이 크므로 규모가 정당화할 때만 강제한다.

## 결과
- `CLAUDE.md`에 단순성 5개 항목 단락 추가.
- builder-sonnet, validator-sonnet, reviewer, architect-opus의 규칙에 self-check / 체크리스트 / 규모 점검 추가.
- `/implement-workitem` skill이 구현 시 단순성 self-check + Clean Code 6항목을 참조.
- ARCHITECTURE_OVERVIEW의 `## 3-1. 레이어 경계 + 의존성 규칙` 섹션이 "프로젝트 규모가 정당화될 때만 채운다" YAGNI 보호 단서와 함께 도입된다(이 ADR과 같은 적용 사이클의 후속 변경).
- `/stabilize-milestone`이 reviewer 입력에 Clean Code 6항목 체크리스트를 명시한다.

## 후속 작업
- 단순성 self-check 4항목이 builder-sonnet 출력 비용을 늘리는지 측정. 비용이 크면 축약 검토.
- `legacy 코드`의 premature abstraction은 즉시 제거하지 않고 `/stabilize-milestone`이 후보로만 보고하는 정책을 유지한다(사용자 결정 우선).
```

#### 4.4.8 `docs/90-decisions/README.md` 인덱스 갱신

**ADR 인덱스 표(Step 3에서 만든)에** 다음 행을 추가한다:

```
| 006 | Simplicity, Clean Code, and Clean Architecture priority | accepted | 단순성 1순위, Clean Code 2순위, Clean Architecture 3순위 (정당화 시) |
```

`docs/00-meta/STRUCTURE.md`의 "Canonical Owner 매핑" 표는 Step 3에서 이미 ADR-006을 가리키도록 작성됐으므로 추가 변경 없다.

### 4.5 검증

- [ ] `CLAUDE.md`에 "단순성·YAGNI" 섹션과 5개 항목, ADR-006 링크가 있다.
- [ ] `.claude/agents/builder-sonnet.md`에 "단순성 self-check" 단락과 "남은 정리 항목" 출력 항목이 있다.
- [ ] `.claude/agents/reviewer.md`에 "Clean Code 6항목 체크리스트" 섹션이 있다.
- [ ] `.claude/agents/validator-sonnet.md`에 "범위 밖 추상화·premature factory" 한 줄이 있다.
- [ ] `.claude/agents/architect-opus.md`에 "프로젝트 규모가 다층 아키텍처를 정당화하는가" 한 줄이 있다.
- [ ] `.claude/skills/implement-workitem/SKILL.md`에 단순성 self-check 참조 한 줄이 있다.
- [ ] `docs/90-decisions/ADR-006-simplicity-and-architecture.md`가 존재한다.
- [ ] `docs/90-decisions/README.md` 인덱스 표에 ADR-006 행이 있다.

### 4.6 커밋 권장

```
docs(claude): add simplicity/YAGNI guardrails (CLAUDE.md, agents, ADR-006)
```

---

## Step 5 — validation report + /repair-workitem

### 5.1 목표

검증 결과를 sub-agent fork 환경에서 회수 가능하도록 파일로 표준화하고, 검증 실패 후 수정만 담당하는 `/repair-workitem`을 신설한다.

### 5.2 의존성

- Step 3 (SSOT 인프라) 완료 — STRUCTURE.md에 validation report 산출물이 등록되어 있어야 한다(Step 3에서 사전 등록 완료).

### 5.3 영향 범위

- 신규 1개 디렉터리 (`docs/40-validation/reports/` + `.gitkeep`)
- 신규 1개 skill (`.claude/skills/repair-workitem/SKILL.md`)
- 수정 1개 skill (`.claude/skills/validate-workitem/SKILL.md`)
- 수정 1개 agent (`.claude/agents/validator-sonnet.md`)
- 수정 1개 (`.gitignore`)
- 수정 2개 문서 (`docs/00-meta/WORKFLOW.md`, `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`. `README.md`/`README_ko.md`는 Step 12까지 변경 보류)

### 5.4 변경 작업

#### 5.4.1 신규 디렉터리 `docs/40-validation/reports/.gitkeep`

빈 파일을 생성한다(`.gitkeep`만, 디렉터리가 git에 잡히도록).

#### 5.4.2 `.gitignore` 갱신

**파일 끝에** 다음 두 줄을 추가한다:

```
docs/40-validation/reports/*.md
!docs/40-validation/reports/.gitkeep
```

#### 5.4.3 `.claude/agents/validator-sonnet.md` 갱신

**frontmatter의 `tools:` 필드 (현재 4번째 줄)**:

기존:
```yaml
tools: Read, Glob, Grep, Bash
```

변경 후:
```yaml
tools: Read, Glob, Grep, Bash, Write
```

**역할 섹션 (현재 12~16번째 줄) 직전(즉, "너는 구현 검증 전담 에이전트다." 직후)에** 다음 단락 추가:

```markdown

이 에이전트는 **판정 + report 기록 전용**이다. 코드 수정, status 변경, 커밋은 직접 수행하지 않는다.
```

**"규칙:" 섹션 (Step 4에서 갱신된 항목 다음)에** 다음 두 줄 추가:

```markdown
- 판정 결과를 표준 양식으로 `docs/40-validation/reports/<task-id>.md`에 기록한다(파일은 task-id 단위로 덮어쓴다 — 가장 최근 1회만 남긴다).
- 구현이나 status 갱신, 커밋을 직접 수행하지 않는다.
```

**"출력 형식:" 섹션 (현재 23~28번째 줄) 끝에** 한 줄 추가:

```markdown
- report 파일 경로 (`docs/40-validation/reports/<task-id>.md`)
- 다음 권장 액션 (Pass면 `/finalize-workitem`, Needs Fix면 `/repair-workitem` — 텍스트 제안임을 명시)
```

#### 5.4.4 `.claude/skills/validate-workitem/SKILL.md` 갱신

**현재 파일 전체를 다음으로 교체**:

```markdown
---
name: validate-workitem
description: Validate whether a completed workitem implementation matches its documented scope and is ready for the next step.
argument-hint: "[task or feature identifier]"
disable-model-invocation: true
allowed-tools: Read Glob Grep Write Bash(pnpm validate) Bash(npm run validate) Bash(make validate) Bash(task validate) Bash(git diff *) Bash(git log *) Bash(git status *)
context: fork
agent: validator-sonnet
---

이 skill은 **판정 + report 기록 전용**이다. status 변경, 코드 수정, 커밋은 하지 않는다.

너의 역할은 지정된 workitem 구현 결과를 검증하고 표준 양식의 report를 기록하는 것이다.

입력:
- `$ARGUMENTS`에는 task ID 또는 feature ID가 들어온다.

반드시 먼저 할 일:
1. 통합 검증 명령(`pnpm validate` / `npm run validate` / `make validate` / `task validate` 중 하나)이 있으면 실행하고 stdout/stderr를 수집한다. 없으면 이 단계는 건너뛴다.
   - 다른 빌더(`bun validate`, `mise run validate`, `just validate` 등)를 쓰는 스택은 본 skill의 `allowed-tools`에 해당 패턴(`Bash(bun validate)` 등)을 추가해야 자동 실행된다. 추가하지 않으면 이 단계는 건너뛰고 정적 판정만 한다.
2. 관련 workitem 문서를 읽는다.
3. 필요한 상위 문서를 읽는다.
4. 최근 변경 파일 또는 diff를 기준으로 구현 결과를 본다.

검증 기준:
- 문서 범위와 구현이 일치하는가
- 범위 밖 변경이 있는가
- 빠진 검증 포인트가 있는가
- obvious regression risk가 있는가
- 통합 검증 명령(있으면) 결과는 통과인가

마지막 단계 — report 파일 작성:
판정 결과를 다음 양식으로 `docs/40-validation/reports/<task-id>.md`에 기록한다(이미 있으면 덮어쓴다).

\`\`\`markdown
# Validation Report: <task-id>

- 검증 시각: <ISO 8601 타임스탬프>
- task-id: <task-id>
- 판정: Pass | Needs Fix

## 통합 명령 실행 결과
<있으면 명령어와 stdout/stderr 요약, 없으면 "통합 명령 미설정 — 정적 판정만 수행">

## 실패 항목 (Needs Fix일 때만)
- [P0] <짧은 설명> — <관련 파일:라인>
- [P1] <...>
- [P2] <...>

## 다음 권장 액션
- Pass: `/finalize-workitem <task-id>` (자동 호출 아님 — 사용자 또는 메인 세션이 발화한다)
- Needs Fix: `/repair-workitem <task-id>` (자동 호출 아님)
\`\`\`

마지막 출력 (메인 세션에 텍스트로):
- Pass / Needs Fix
- 핵심 문제 최대 5개
- report 파일 경로
- 다음 추천 단계 (텍스트 제안임을 명시)
```

#### 5.4.5 `.claude/skills/repair-workitem/SKILL.md` 신규

**파일 전체 내용**:

```markdown
---
name: repair-workitem
description: Apply fixes for failed validation report items, scoped to the documented workitem.
argument-hint: "[task id] [optional notes]"
disable-model-invocation: true
allowed-tools: Read Glob Grep Write Edit Bash
context: fork
agent: builder-sonnet
---

이 skill은 직전 `/validate-workitem`이 남긴 report의 실패 항목만 수정한다.
**새 기능 추가, 범위 밖 변경, 자동 커밋은 금지한다.**

입력:
- `$ARGUMENTS`에는 task ID와 (선택) 부분 지정 메모가 들어온다.
  - 예: `T-001`
  - 예: `T-001 "P0 #1, P1 #3만"` — report의 일부 항목만 수정

반드시 먼저 할 일:
1. 관련 task 문서를 읽는다.
2. `docs/40-validation/reports/<task-id>.md`를 읽는다.
   - 파일이 없거나 stale(파일 mtime이 task 문서의 변경보다 오래됨)하면, 사용자에게 `/validate-workitem` 선행을 안내하고 종료한다.
   - 파일이 `Pass`이면 `/finalize-workitem`을 안내하고 종료한다(repair 대상 없음).
3. 사용자가 인자로 부분 지정을 줬으면 그 부분만 대상으로 한다.
4. 실패 항목을 우선순위(P0 > P1 > P2)로 정렬한다.

수행:
1. 우선순위 순으로 실패 항목을 수정한다.
2. 한 라운드에는 P0/P1만 처리하고 P2 이하는 다음 라운드로 추천한다(가능하면).

책임 경계:
- 새 기능을 추가하지 않는다.
- task 범위 밖 파일을 수정하지 않는다.
- 자동 커밋하지 않는다 — 결과만 반환하고 커밋은 `/finalize-workitem` 또는 사용자가 별도로.
- 무한 루프 가드: 한 task당 연속 3회 이상 repair → validate 사이클이 돌면 사용자 확인을 요구한다.

마지막 출력:
- 수정 파일 목록
- 어떤 실패 항목을 어떻게 해소했는지
- 미해결 항목 (있으면)
- 다음 권장 액션 (보통 `/validate-workitem <task-id>` 재실행)
```

#### 5.4.6 `docs/00-meta/WORKFLOW.md` 갱신

**"## 4. 구현 및 검증" 섹션 (현재 16~18번째 줄)을** 다음으로 교체:

```markdown
## 4. 구현 및 검증
- 구현은 `/implement-workitem`으로 시작한다.
- 검증은 `/validate-workitem`으로 수행한다 — 판정 + `docs/40-validation/reports/<task-id>.md` 기록.
- 검증 실패 시 `/repair-workitem`으로 report의 실패 항목을 수정한다.
- 검증 통과 시 `/finalize-workitem`으로 status `done` 갱신 + 커밋.
- 마일스톤 단위 종합 점검은 `/stabilize-milestone`에서 수행한다.
- 누적 QA 결과는 `docs/40-validation/QA_FINDINGS.md`에 기록한다.
- 개선 제안은 `docs/40-validation/IMPROVEMENT_GUIDE.md`에 정리한다.
```

> 주: `/finalize-workitem`은 Step 6에서, `/stabilize-milestone`은 Step 9에서 신설된다. 본 Step 5 적용 직후에는 두 명령이 아직 존재하지 않지만, 본 텍스트는 영구 산출물이므로 미래 Step의 도입 시점을 본문에 박지 않는다("(이후 Step에서 도입)" 같은 가이드 한정 문구 사용 금지). 사용자 관점에서는 본 줄이 "장차 흐름이 이렇다"로 읽혀 무관하다.

#### 5.4.7 `docs/00-meta/AGENT_EXECUTION_STRATEGY.md` 갱신

**"## 스킬 실행 순서 가이드" 섹션 (현재 64~76번째 줄)을** 다음으로 교체:

```markdown
## 스킬 실행 순서 가이드

일반적인 프로젝트 진행에서의 추천 스킬 순서:

1. `/bootstrap-project` → charter + architecture + 초기 workitem 생성
2. `/bootstrap-stack` → 스택 확정 후 자동화 설계
3. `/plan-workitem` → milestone/feature/task 분해
4. `/implement-workitem` → task 구현
5. `/validate-workitem` → 판정 + report 기록
6. `/repair-workitem` (Needs Fix일 때만) → report의 실패 항목 수정
7. `/finalize-workitem` (Pass일 때) → status `done` + 커밋
8. 마일스톤의 모든 task가 `done`이 되면 `/stabilize-milestone`

각 단계에서 중요한 설계 판단이 필요하면 architect-opus를 먼저 사용한다.
문서 품질이 걱정되면 `/review-doc` 또는 reviewer를 사이에 끼운다.

**스킬 자동 호출 아님** — `/validate-workitem`이 출력하는 "다음 액션 추천"은 텍스트 제안일 뿐이다. 사용자 또는 메인 세션이 그 제안을 받아 실제 다음 skill을 발화한다.
```

`docs/90-decisions/README.md`는 본 Step에서 변경 없음(ADR-007/008은 Step 6에서 추가).

### 5.5 검증

- [ ] `docs/40-validation/reports/.gitkeep`이 존재한다.
- [ ] `.gitignore`에 `docs/40-validation/reports/*.md`와 `!docs/40-validation/reports/.gitkeep`이 추가됐다.
- [ ] `.claude/agents/validator-sonnet.md`의 `tools:`에 `Write`가 포함됐고, "판정 + report 기록 전용" 명시 단락이 있다.
- [ ] `.claude/skills/validate-workitem/SKILL.md`이 통합 명령 실행 → report 작성 절차를 본문에 담고 있다.
- [ ] `.claude/skills/repair-workitem/SKILL.md`가 신규 생성되어 책임 경계와 stale/Pass 종료 흐름을 담고 있다.
- [ ] `docs/00-meta/WORKFLOW.md`에 `/validate-workitem`/`/repair-workitem` 흐름이 반영됐다.
- [ ] `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`의 "스킬 실행 순서"에 새 흐름이 반영됐다.

### 5.6 커밋 권장

```
feat(skill): add /repair-workitem and validation report standardization
```

---

## Step 6 — /finalize-workitem 신설 + 워크아이템 라이프사이클 + commit convention

### 6.1 목표

검증 통과 후의 마감 절차(status `done` 갱신, 명시적 파일 add, 커밋)를 단일 skill로 표준화한다. Conventional Commits 기본 채택을 ADR로 고정하고, 워크아이템 라이프사이클(discover→…→stabilize)도 ADR로 보존한다.

### 6.2 의존성

- Step 5 (validation report) 완료 — `/validate-workitem`이 report를 기록하고 있어야 finalize 흐름이 자연스럽다.

### 6.3 영향 범위

- 신규 1개 skill (`.claude/skills/finalize-workitem/SKILL.md`)
- 수정 1개 agent (`.claude/agents/builder-sonnet.md`)
- 수정 1개 템플릿 (`docs/30-workitems/_templates/TASK_TEMPLATE.md`)
- 신규 2개 ADR (`docs/90-decisions/ADR-007-workitem-lifecycle.md`, `docs/90-decisions/ADR-008-commit-convention.md`)
- 갱신 1개 (`docs/90-decisions/README.md` 인덱스)
- 갱신 2개 문서 (`docs/00-meta/WORKFLOW.md`, `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`)

### 6.4 변경 작업

#### 6.4.1 `docs/30-workitems/_templates/TASK_TEMPLATE.md` 갱신

**현재 "## 4. 제외 항목" (12번째 줄) 다음에** 새 섹션 삽입:

```markdown
## 4-1. 변경 예정 파일/경로
<!-- 구현 시점에 채운다. /finalize-workitem이 명시적 파일 add 시 우선 참조한다.
     엄격한 화이트리스트가 아니라 참조 목록이다. 비어 있거나 git 실제 변경과 어긋나면 finalize는 차이를 출력에 명시하고 Needs Review로 즉시 종료한다 — 본 섹션을 갱신해 재실행하거나 `--apply` force 모드로 진행한다.
     task 문서 자체는 finalize가 자동 포함하므로 본 섹션에 적지 않는다. -->
- 

```

> 주: TASK_TEMPLATE의 `## 6. 테스트 포인트` → `## 6. Acceptance Criteria` 교체와 `## 6-1`/`## 6-2` 신설은 Step 8에서 수행한다.

#### 6.4.2 `.claude/skills/finalize-workitem/SKILL.md` 신규

**파일 전체 내용**:

```markdown
---
name: finalize-workitem
description: Finalize a passed workitem — set status done, stage explicit files, and commit.
argument-hint: "[task or feature identifier(s)] [--apply]"
disable-model-invocation: true
allowed-tools: Read Glob Grep Write Edit Bash(git add *) Bash(git status *) Bash(git diff *) Bash(git commit *) Bash(pnpm validate) Bash(npm run validate) Bash(make validate) Bash(task validate)
context: fork
agent: builder-sonnet
---

이 skill은 검증을 통과한 workitem을 마감한다 — status 갱신 + 명시적 파일 add + 커밋.

입력:
- `$ARGUMENTS`에는 task ID(또는 다중 ID, 예: `T-001 T-002`)가 들어온다.
- 선택 플래그 `--apply` — task 문서 `## 4-1. 변경 예정 파일/경로`와 git 실제 변경이 어긋나도 git 실제 변경을 신뢰하고 진행(아래 5-(4) 차이 처리에서 종료하지 않는다). 단 민감 경로 가드는 그대로 적용된다.

반드시 먼저 할 일:
1. 관련 task 문서를 읽는다.
2. 통합 검증 명령(`pnpm validate` / `npm run validate` / `make validate` / `task validate`)이 있으면 실행한다.
   - 실패 → `Needs Fix`로 종료. 커밋하지 않음. `/repair-workitem <task-id>`를 텍스트로 제안.
   - 통합 명령이 없으면(스택 미정) 이 단계는 건너뛴다.

수행:
3. task 문서의 `## 0. Status`를 `done`으로 갱신한다.
4. `git status --porcelain` / `git diff --name-only`로 실제 변경 파일을 회수한다.
5. 명시적 파일 add — **`git add -A` / `git add .`는 사용하지 않는다**.
   파일 목록 산출 우선순위:
   - **(0) 자동 포함**: 본 skill이 step 3에서 갱신한 task 문서 자체는 항상 add 대상에 포함하고, 아래 (1)·(2) 비교에서는 제외한다.
   - **(1) task 문서의 `## 4-1. 변경 예정 파일/경로`** — 있으면 우선 참조. 본 섹션은 task 문서 자체를 다시 적지 않는다(자동 포함됨).
   - **(2) git 실제 변경 파일** — task 문서를 제외한 나머지.
   - **(3) 제외 규칙** — 다음을 add 대상에서 제외:
     - 민감 경로(`.env*`, `secrets/**`)
     - 빌드 산출물(`node_modules/`, `dist/`, `build/`, `.next/`, `coverage/`)
     - task 범위와 명백히 무관한 파일
   - **(4) 차이 처리** — 본 skill은 `context: fork` 환경에서 실행되므로 사용자에게 실시간 확인을 받을 수 없다. (1)과 (2)(둘 다 task 문서 제외 기준)가 어긋나면(또는 (1)이 비어 있고 (2)에 add 대상으로 의심되는 파일이 섞여 있으면) **차이를 출력에 명시하고 즉시 종료**한다(`Needs Review` 종료). 사용자가 task 문서의 `## 4-1`을 갱신하거나 `--apply` force 모드로 재실행하도록 안내한다.
   민감 경로가 staged 영역에 들어오면 즉시 종료한다.
6. 커밋 메시지 초안을 Conventional Commits 스타일로 생성한다(정책: ADR-008).
   - 형식: `<type>(<scope>): <summary>` — `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `perf` 등.
   - 본문에 변경 요약 한 단락 + task ID 참조.
7. `git commit -m "..."` 실행.
   - **금지**: `--no-verify`, `--amend`, `git push`.

마지막 출력:
- 커밋 해시
- 커밋 메시지
- 갱신된 task status
- 다음 권장 단계 (다음 task로 진행 또는 마일스톤이면 `/stabilize-milestone`)

가드:
- 작업 트리에 변경이 없으면 "변경 없음" 종료.
- `git add -A` / `git add .` 금지.
- 민감 경로 staged 시 즉시 종료.
- `--amend` 금지(--amend는 직전 커밋 변경 — 작업 단위가 흐려진다).
- `--no-verify` 금지(pre-commit hook은 우회하지 않는다).
- `git push`는 사용자 명시 요청 없이 실행하지 않는다.

다중 ID 처리:
- `$ARGUMENTS`에 여러 task ID가 있으면 모든 ID의 status를 갱신하고 한 커밋에 묶는다.
- 커밋 메시지에 모든 ID를 명시한다.
```

#### 6.4.3 `.claude/agents/builder-sonnet.md` 갱신

**"규칙:" 섹션의 단순성 self-check 단락 다음에** finalize 위임 가드 섹션을 추가:

```markdown

finalize 위임을 받았을 때의 가드 (`/finalize-workitem`이 본 에이전트를 fork할 때 적용):
- `git add -A` / `git add .` 금지 — 명시적 파일 목록만 add.
- 민감 경로(`.env*`, `secrets/**`)가 staged 영역에 들어오면 즉시 종료.
- `git commit --no-verify`, `git commit --amend`, `git push` 금지.
- 커밋 메시지는 Conventional Commits 스타일(정책: [ADR-008](../../docs/90-decisions/ADR-008-commit-convention.md)).

구현 완료 후 task 문서의 `## 4-1. 변경 예정 파일/경로` 섹션을 갱신한다 — finalize의 add 참조 목록으로 사용된다.
```

#### 6.4.4 `docs/90-decisions/ADR-007-workitem-lifecycle.md` 신규

**파일 전체 내용**:

```markdown
# ADR-007 워크아이템 라이프사이클

## 상태
accepted

## 배경
이 보일러플레이트의 워크아이템 흐름은 단순한 "구현 → 검증" 두 단계가 아니라 다음 8단계로 정의되어야 한다. 각 단계의 책임이 분리되어 있어야 sub-agent fork 환경에서 결과 회수와 책임 경계가 명확해진다.

본 ADR 이전의 워크플로우에서는 검증과 마감이 한 명령에 묶이지 않아 사람이 매번 status 갱신과 커밋을 수동으로 했고, `/validate-workitem`이 검증 외 작업까지 떠안는 경향이 생겼다.

## 결정
워크아이템 라이프사이클을 다음 8단계로 정의한다.

| # | 단계 | skill | 주체 agent | 책임 경계 |
|---|------|-------|-----------|----------|
| 1 | discover | `/discover-product` | (메인 세션 운전) | persona/pain/JTBD/시나리오 발굴 → DISCOVERY.md |
| 2 | bootstrap | `/bootstrap-project` | architect-opus | DISCOVERY.md → charter/architecture/M1/F-001 |
| 3 | plan | `/plan-workitem` | planner | milestone/feature/task 분해 |
| 4 | implement | `/implement-workitem` | builder-sonnet | task 구현 (Red→Green→Refactor 사이클, ADR-009) |
| 5 | validate | `/validate-workitem` | validator-sonnet | 판정 + report 기록. **status 변경·코드 수정·커밋 금지.** |
| 6 | repair (Needs Fix일 때만) | `/repair-workitem` | builder-sonnet | report의 실패 항목만 수정. **자동 커밋 금지, 새 기능 금지, 범위 밖 변경 금지.** |
| 7 | finalize (Pass일 때) | `/finalize-workitem` | builder-sonnet | status `done` 갱신 + 명시적 파일 add + Conventional Commits 커밋 |
| 8 | stabilize | `/stabilize-milestone` | (qa, reviewer를 위임) | 마일스톤 단위 종합 점검. **코드 수정·커밋·status 변경 금지.** |

skill 간 흐름은 **자동 호출이 아니라 텍스트 제안 → 사용자/메인이 발화**한다. 예: validate Pass 출력은 "다음 액션: `/finalize-workitem T-001`"을 텍스트로 제안한다. 자동 호출이 아니다.

## 근거
- 책임 분리로 sub-agent fork 환경에서 각 단계의 입출력이 명확해진다.
- "검증" 단계와 "마감" 단계 분리로 "검증은 통과인데 status가 in-progress"인 모순이 사라진다.
- repair 단계가 명시적이라 무한 루프(repair → validate → repair)에 가드를 둘 수 있다(연속 3회 시 사용자 확인).
- 라이프사이클 정의가 ADR로 박혀 있으면 fork된 미래 프로젝트에서 6개월 뒤 사용자가 "왜 이런 단계 분리인가"를 추적할 수 있다.

## 결과
- 8개 skill이 각 단계에 1:1로 대응한다.
- `docs/00-meta/WORKFLOW.md`가 이 라이프사이클을 단계별 사용법으로 풀어 적는다.
- `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`가 단계별 위임 대상을 정의한다.
- `/validate-workitem`은 **판정 + report 기록 전용**. 자동 수정·자동 마감 금지.
- 무한 루프 가드: repair 한 라운드는 P0/P1만 처리하고 P2 이하는 다음 라운드 추천. 같은 task에서 repair → validate 사이클이 연속 3회 이상이면 사용자 확인 요구.

## 후속 작업
- `/finalize-workitem`이 통합 검증 명령(`validate`)을 한 번 더 돌리는 정책 — 직전 `/validate-workitem` 통과 후에도 안전성을 위해 한 번 더(상태가 변했을 수 있음).
- `/stabilize-milestone`은 코드 수정·커밋을 하지 않고 점검 결과를 누적 기록한다 — 후속 작업이 필요하면 `/repair-workitem` 또는 새 task로 연결.
```

#### 6.4.5 `docs/90-decisions/ADR-008-commit-convention.md` 신규

**파일 전체 내용**:

```markdown
# ADR-008 Conventional Commits 기본 채택

## 상태
accepted

## 배경
`/finalize-workitem`이 도입되면서 커밋 메시지 양식의 표준이 필요해졌다. 일관된 양식은 다음을 가능하게 한다.
- changelog 자동화 가능성 (스택 확정 후)
- 커밋 단위 작업 추적
- 변경 종류(feat/fix/chore/...)에 따른 리뷰 우선순위 판단

## 결정
보일러플레이트의 기본 커밋 컨벤션은 [Conventional Commits](https://www.conventionalcommits.org)다.

기본 형식:
\`\`\`
<type>(<scope>): <summary>

<optional body>
<optional footer (e.g., task ID 참조)>
\`\`\`

기본 type 어휘: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `perf`, `build`, `ci`, `style`, `revert`.

## 근거
- 커뮤니티 표준이라 진입 비용이 낮다.
- 단순한 형식으로 메시지의 의도가 한눈에 보인다.
- changelog 도구 생태계가 잘 정비되어 있다.
- 보일러플레이트 자체가 다언어/다스택을 지원하므로 stack-agnostic한 표준이 필요하다.

## 결과
- `/finalize-workitem`이 커밋 메시지 초안을 Conventional Commits 형식으로 생성한다.
- charter에서 다른 컨벤션을 명시적으로 override하지 않는 한 기본은 Conventional Commits.

## 후속 작업
- 사용자가 다른 컨벤션을 원하면 charter에서 명시적으로 override + 새 ADR로 결정 보존.
- changelog 자동화는 스택 확정 후 별도 task로 도입.
```

#### 6.4.6 `docs/90-decisions/README.md` 인덱스 갱신

**ADR 인덱스 표에** 두 행 추가:

```
| 007 | Workitem lifecycle | accepted | discover→bootstrap→plan→implement→validate→repair→finalize→stabilize 8단계 |
| 008 | Commit convention | accepted | Conventional Commits 기본 채택 |
```

#### 6.4.7 `docs/00-meta/WORKFLOW.md` 갱신

> 주: 본 Step 시작 시점에는 Step 5의 변경(`## 4. 구현 및 검증` 본문 교체)이 이미 반영되어 있다. 라인 번호는 직접 파일을 다시 읽어 확인한다.

**Step 5에서 갱신한 "## 4. 구현 및 검증" 섹션 직후(즉 다음 헤더인 `## 5. 의사결정 기록` 직전)에** 새 섹션 추가:

```markdown

## 4-1. 마감 (finalize)
- `/finalize-workitem`이 task 문서 status를 `done`으로 갱신한다.
- 명시적 파일 add — `git add -A` / `git add .` 금지.
- 커밋 메시지는 Conventional Commits 스타일(ADR-008).
- 다중 task 묶음 커밋: `/finalize-workitem T-001 T-002` 형태로 다중 ID 허용.

```

**"## 문서 상태 전이" 섹션 직전에** 워크아이템 라이프사이클 한 줄 그림 추가(SSOT는 ADR-007이지만 사용자 동선을 위해 한 줄 그림 1개는 본 문서에도 둔다 — 정의는 ADR이 권위):

```markdown

## 워크아이템 라이프사이클

\`\`\`
discover → bootstrap → plan → implement → validate ─┬─Pass─→ finalize → stabilize
                                                     └─Needs Fix─→ repair → (validate 재실행)
\`\`\`

각 단계의 정의와 책임 경계는 [ADR-007-workitem-lifecycle.md](../90-decisions/ADR-007-workitem-lifecycle.md)가 SSOT다.
스킬 간 흐름은 **자동 호출이 아니라 텍스트 제안 → 사용자/메인이 발화**한다.
```

#### 6.4.8 `docs/00-meta/AGENT_EXECUTION_STRATEGY.md` 갱신

**Step 5에서 갱신한 "## 스킬 실행 순서 가이드" 섹션의 항목 7을** 다음으로 교체(ADR 링크 보강):

기존:
```
7. `/finalize-workitem` (Pass일 때) → status `done` + 커밋
```

변경 후:
```
7. `/finalize-workitem` (Pass일 때) → status `done` 갱신 + 명시적 파일 add + Conventional Commits 커밋 (정책: [ADR-007](../90-decisions/ADR-007-workitem-lifecycle.md), [ADR-008](../90-decisions/ADR-008-commit-convention.md))
```

### 6.5 검증

- [ ] `.claude/skills/finalize-workitem/SKILL.md`가 존재하고 8단계 절차 + 가드 + 다중 ID 처리를 담는다.
- [ ] `.claude/agents/builder-sonnet.md`에 finalize 가드 단락이 있다.
- [ ] `docs/30-workitems/_templates/TASK_TEMPLATE.md`에 `## 4-1. 변경 예정 파일/경로` 섹션이 있다.
- [ ] `docs/90-decisions/ADR-007-workitem-lifecycle.md`, `docs/90-decisions/ADR-008-commit-convention.md`가 존재한다.
- [ ] `docs/90-decisions/README.md` 인덱스 표에 ADR-007, ADR-008 행이 있다.
- [ ] `docs/00-meta/WORKFLOW.md`에 "## 4-1. 마감" 섹션과 워크아이템 라이프사이클 한 줄 그림이 있다.
- [ ] `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`의 항목 7에 ADR-007/008 링크가 있다.

### 6.6 커밋 권장

```
feat(skill): add /finalize-workitem with lifecycle and commit-convention ADRs
```

---

## Step 7 — /stack-guard 1단계

### 7.1 목표

`/bootstrap-stack` 이후 사용자가 이어서 발화하는 진입점 `/stack-guard`를 신설하여, 통합 검증 명령(`validate`)과 검증 스크립트를 생성한다.

**범위 명시**: PostToolUse hook 자동 등록은 본 1단계에서 처방하지 않는다(0.2 운영 원칙 — prototyping 미완료 메커니즘은 사용자 환경에 자동 적용하지 않음). prototyping 후 별도 항목으로 분리. 본 Step은 검증 스크립트 placeholder + 통합 명령 안내 + STACK_SETUP_PLAN의 hook 등록 안내 문구만 만든다.

### 7.2 의존성

- Step 3 (SSOT) 완료 — STRUCTURE.md에 `verify scripts`와 `stack setup plan` 산출물이 등록되어 있어야 한다.

### 7.3 영향 범위

- 신규 1개 skill (`.claude/skills/stack-guard/SKILL.md`)
- 수정 1개 skill (`.claude/skills/bootstrap-stack/SKILL.md`)
- 수정 1개 (`.claude/skills/bootstrap-stack/output-checklist.md`)
- 수정 1개 (`docs/00-meta/GUARDRAILS_STRATEGY.md`)
- 수정 0개 — `scripts/verify.*`은 본 Step에서는 placeholder를 만들지 않는다(스택 미정 상태에서 구체 스크립트는 의미 없음). `/stack-guard` 발화 시 생성하도록 skill 본문에 절차만 명시.

### 7.4 변경 작업

#### 7.4.1 `.claude/skills/stack-guard/SKILL.md` 신규

> frontmatter 결정: `model`/`effort` 라인은 두지 않는다. agent 바인딩(`builder-sonnet`)이 모델을 결정한다(별칭 정책 — ADR-004). bootstrap-stack과 달리 stack-guard는 정해진 스택을 받아 스크립트·명령을 *변환*하는 작업이라 architect-opus 수준의 reasoning이 필요하지 않다. 큰 설계 판단이 필요해지면 사용자가 명시적으로 architect-opus 단발 호출로 보강한다.

**파일 전체 내용**:

```markdown
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
```

#### 7.4.2 `.claude/skills/bootstrap-stack/SKILL.md` 갱신

**"마지막 출력:" 섹션 (현재 41~46번째 줄) 끝에** 한 줄 추가:

```markdown
- 다음 권장 단계로 `/stack-guard`를 안내한다(자동 호출 아님 — 사용자가 발화한다).
```

#### 7.4.3 `.claude/skills/bootstrap-stack/output-checklist.md` 갱신

**"## 출력 원칙" 섹션 끝에** 다음 항목 추가:

```markdown
- 통합 검증 명령(`validate`)·검증 스크립트·hook 등록 안내는 `/stack-guard`가 별도로 생성한다 — bootstrap-stack 이후 다음 단계 안내에 포함한다.
```

#### 7.4.4 `docs/00-meta/GUARDRAILS_STRATEGY.md` 갱신

**"## stack-specific 생성 시점" 섹션 (현재 34~42번째 줄) 직후에** 새 섹션 추가:

```markdown

## /stack-guard 1단계 산출물 범위
스택이 확정된 후 사용자가 `/stack-guard`를 발화하면 다음을 생성한다.

**1단계 산출물 (자동 생성)**:
- 통합 진입점 — 이름은 `validate`로 고정 (`pnpm validate` / `npm run validate` / `make validate` / `task validate` 중 스택에 자연스러운 1종).
- `scripts/verify.{sh,ps1,mjs,py}` 중 스택에 자연스러운 런타임 1종.
- `.gitattributes` (line ending 통일).
- `docs/00-meta/STACK_SETUP_PLAN.md`에 PostToolUse hook **매뉴얼** 등록 안내 문구.

**1단계 비범위 (prototyping 후 분리)**:
- PostToolUse hook 자동 등록 — `acceptEdits` 모드에서 매 Edit/Write마다 lint를 자동 실행하면 비용이 폭증할 수 있다(사용자가 수락 프롬프트로 차단할 기회조차 없음). hook 입출력 JSON 스키마와 settings.json patch 양식이 본 시점에 직접 검증되지 않았다. prototyping 단계에서 (1) hook 입출력 스키마와 settings.json patch 양식 (2) `acceptEdits` 모드의 실측 비용 (3) 단일 OS/셸 가정의 자동 감지 신뢰도를 측정한 뒤 별도 항목(`5b. /stack-guard hook 자동 등록`)으로 분리한다.

```

### 7.5 검증

- [ ] `.claude/skills/stack-guard/SKILL.md`가 존재하고 1단계 범위·비범위가 명시돼 있다.
- [ ] `.claude/skills/bootstrap-stack/SKILL.md`의 마지막 출력에 "다음 권장 단계로 `/stack-guard` 안내" 한 줄이 있다.
- [ ] `.claude/skills/bootstrap-stack/output-checklist.md`에 "통합 검증 명령은 `/stack-guard`가 생성" 안내가 있다.
- [ ] `docs/00-meta/GUARDRAILS_STRATEGY.md`에 "/stack-guard 1단계 산출물 범위" 섹션이 있다.

### 7.6 커밋 권장

```
feat(skill): add /stack-guard for unified validate command and verify scripts (1단계)
```

---

## Step 8 — 에이전트 TDD 기본 흐름

### 8.1 목표

`/implement-workitem`의 디폴트 흐름을 **Red → Green → Refactor 3 phase**로 명시한다. AC를 task 단위로 명문화하고, validation 단계가 AC ↔ 테스트 매핑을 점검하도록 한다.

### 8.2 의존성

- Step 5 (validation report) 완료 — report에 `AC-N별 ✅/❌` 기록이 가능해야 한다.
- Step 6 (finalize) 완료 — finalize의 통과 조건에 "AC 미충족 0개"가 추가될 자리.
- Step 7 (stack-guard) 완료 — 통합 `validate` 명령에 test runner가 포함되어 있어야 검증 단계의 AC↔테스트 점검이 의미를 갖는다.

### 8.3 영향 범위

- 수정 2개 템플릿 (`docs/30-workitems/_templates/TASK_TEMPLATE.md`, `docs/30-workitems/_templates/FEATURE_TEMPLATE.md`)
- 수정 2개 skill (`.claude/skills/implement-workitem/SKILL.md`, `.claude/skills/validate-workitem/SKILL.md`)
- 수정 1개 skill (`.claude/skills/finalize-workitem/SKILL.md`)
- 수정 2개 agent (`.claude/agents/builder-sonnet.md`, `.claude/agents/validator-sonnet.md`)
- 수정 1개 (`CLAUDE.md`)
- 신규 1개 (`docs/90-decisions/ADR-009-tdd-default.md`)
- 갱신 1개 (`docs/90-decisions/README.md` 인덱스)

### 8.4 변경 작업

#### 8.4.1 `docs/30-workitems/_templates/TASK_TEMPLATE.md` 갱신

**"## 6. 테스트 포인트" (Step 6에서 `## 4-1. 변경 예정 파일/경로` 5~6줄을 삽입한 뒤이므로 21~22번째 줄 안팎)을 다음 세 섹션으로 교체**:

```markdown
## 6. Acceptance Criteria
<!-- AC-1, AC-2 ... 형식. Given-When-Then 또는 명세 형태. 측정 가능해야 한다.
     예: AC-1 [Given] 로그인되지 않은 사용자가 [When] /me 를 호출하면 [Then] 401을 반환한다. -->
- AC-1:
- AC-2:

## 6-1. 테스트 시나리오 (TDD Red)
<!-- 각 AC에 대응하는 테스트 파일·테스트 이름. 사람이 미리 채우거나 builder-sonnet이 Red phase 시작 전에 채운다.
     예:
     - AC-1 → tests/auth/me.spec.ts > test_AC_1_unauthenticated_returns_401
     - AC-2 → tests/auth/me.spec.ts > test_AC_2_authenticated_returns_user -->
- 

## 6-2. TDD opt-out
<!-- 비어 있으면 TDD 적용. 채울 때만 사유와 follow-up task 링크가 모두 있어야 한다.
     예: spike 종료 후 T-014에서 TDD로 재구현 (사유: 외부 의존 탐색).
     사유와 follow-up 링크 둘 중 하나라도 비면 형식 위반으로 표시된다. -->
- 사유:
- Follow-up task:
```

#### 8.4.2 `docs/30-workitems/_templates/FEATURE_TEMPLATE.md` 갱신

**"## 8. 검증 방법" 섹션 (현재 20번째 줄)을** 다음으로 교체:

```markdown
## 8. 검증 방법
<!-- 시나리오 수준 검증. task 단위로는 Acceptance Criteria(AC-1, AC-2 ...)로 분해된다(TASK_TEMPLATE의 ## 6 참조). -->

```

#### 8.4.3 `.claude/skills/implement-workitem/SKILL.md` 전면 재작성

**파일 전체 내용**:

```markdown
---
name: implement-workitem
description: Implement one scoped workitem using builder-sonnet, following Red→Green→Refactor TDD cycle.
argument-hint: "[task or feature identifier] [--fast]"
disable-model-invocation: true
allowed-tools: Read Glob Grep Write Edit Bash
context: fork
agent: builder-sonnet
---

너의 역할은 지정된 workitem을 Red → Green → Refactor 3 phase 사이클로 구현하는 것이다.

입력:
- `$ARGUMENTS`에는 task ID 또는 feature ID가 들어온다.
- `--fast` 플래그가 있으면 RGR 사이클을 1회만 돌려 첫 AC만 완료하고 종료한다(prototype용).

반드시 먼저 할 일:
1. 관련 task 문서를 읽는다.
2. 필요하면 상위 feature/milestone/architecture 문서를 읽는다.
3. task 문서의 `## 6. Acceptance Criteria`(AC-1, AC-2 ...)를 회수한다.
4. `## 6-2. TDD opt-out`을 점검한다 — 사유와 follow-up이 모두 있으면 opt-out 모드로 진행, 둘 중 하나만 비어 있으면 형식 위반으로 표시하고 종료(사용자에게 보강 요청).

opt-out 흐름 (사유와 follow-up 모두 채워졌을 때만):
- 테스트 작성을 건너뛴다.
- 출력에 "TDD opt-out 사유: <사유> / Follow-up: <task ID>"를 명시.
- 다른 흐름은 동일.

기본 흐름 — Red → Green → Refactor (각 AC마다 반복):

**1. Red**
- task의 `## 6. Acceptance Criteria` 항목을 1개 골라 그것을 위반하는 실패 테스트를 작성한다.
- 테스트 이름에 `AC_N` 식별자를 포함하는 것을 권장(예: `test_AC_1_user_can_login`). 강제 아님.
- 테스트 실행 → "원하는 이유로" 실패하는지 확인 후 phase 종료.

**2. Green**
- 그 테스트를 통과시키는 **최소 코드**만 작성한다.
- 다른 AC를 미리 만족시키지 않는다(YAGNI 강화 — ADR-006).
- 테스트 통과 확인 후 phase 종료.

**3. Refactor**
- 단순성 self-check 4항목 + Clean Code 6항목(ADR-006)에 따라 정리한다.
- 외부 행동을 바꾸지 않는다.
- 테스트 통과 유지 확인 후 phase 종료.

위 사이클을 task의 모든 AC가 소진될 때까지 반복.
`--fast`면 첫 AC만 완료하고 종료, 나머지 AC는 후속 호출 권장.

마지막에 task 문서의 `## 4-1. 변경 예정 파일/경로`를 갱신한다(finalize의 add 참조 목록).

마지막 출력:
- 수정 파일 목록
- AC별 진행 상태 (완료/미완료, 예: `AC-1 ✅, AC-2 ✅, AC-3 ❌(다음 호출)`)
- 핵심 변경 사항
- 단순성 self-check 결과 (남은 정리 항목 N건, 있으면 명시)
- 남은 리스크
- 다음 추천 단계 (보통 `/validate-workitem <task-id>`)

정책 근거:
- TDD: [ADR-009-tdd-default.md](../../../docs/90-decisions/ADR-009-tdd-default.md)
- 단순성·Clean Code: [ADR-006-simplicity-and-architecture.md](../../../docs/90-decisions/ADR-006-simplicity-and-architecture.md)
```

#### 8.4.4 `.claude/skills/validate-workitem/SKILL.md` 갱신

**"검증 기준:" 섹션에** 다음 두 줄 추가(통합 명령 결과 직후):

```markdown
- AC ↔ 테스트 매핑 — task 문서의 AC-N마다 대응하는 테스트가 존재하는가(자연어 매칭 휴리스틱 또는 테스트 이름의 `AC_N` 식별자 매칭).
- 테스트 선행 휴리스틱 — git log에서 동일 task 범위의 테스트 파일 추가/수정이 구현 파일보다 먼저(또는 동일 커밋) 들어왔는지. 단순 경고로만 보고하고 강제 종료하지 않는다(소규모 작업이 한 커밋에 묶이는 경우 정상).
```

**report 양식 (Step 5에서 작성한 markdown 양식)에** 다음 섹션을 추가:

```markdown
## AC ↔ 테스트 매핑
- AC-1: ✅ tests/foo.spec.ts > test_AC_1_xxx
- AC-2: ❌ (테스트 없음)
- AC-3: ✅ tests/bar.spec.ts > test_AC_3_xxx
```

> 변경 위치: `.claude/skills/validate-workitem/SKILL.md`의 "마지막 단계 — report 파일 작성:" 안의 markdown 양식. `## 실패 항목` 섹션 직후에 `## AC ↔ 테스트 매핑` 섹션을 추가한다.

#### 8.4.5 `.claude/skills/finalize-workitem/SKILL.md` 갱신

**"반드시 먼저 할 일:" 섹션의 마지막 단계(즉 통합 검증 명령 실행 단계, 현재 step 2) 다음에** 새 step 3을 추가하고 기존 "수행:" 섹션의 step 번호를 한 단계씩 미는 식으로 갱신한다(`3 → 4`, `4 → 5`, ... `7 → 8`).

추가할 새 step 3:

```markdown
3. AC 미충족 점검 — 직전 `/validate-workitem`의 report(`docs/40-validation/reports/<task-id>.md`)에서 AC 매핑이 모두 ✅인지 확인한다.
   - report 파일이 없거나 stale(파일 mtime이 task 문서 또는 변경된 구현 파일보다 오래됨)하면 `/validate-workitem <task-id>` 선행을 안내하고 `Needs Validation`으로 종료한다(커밋하지 않음).
   - ❌가 하나라도 있으면 `Needs Fix`로 종료하고 `/repair-workitem <task-id>`를 안내한다.
   - opt-out 사유가 있는 task(task 문서 `## 6-2. TDD opt-out`이 채워진 경우)는 예외 — 출력에 opt-out 사유와 follow-up task ID를 명시한다.
```

> 주: 본 가드는 "반드시 먼저 할 일"의 일부다(검증 단계지 수행 단계가 아님). 따라서 "수행:" 섹션의 step 번호는 본 추가 후 `3 → 4`, `4 → 5`, `5 → 6`, `6 → 7`, `7 → 8`로 시프트한다.

#### 8.4.6 `.claude/agents/builder-sonnet.md` 갱신

**"규칙:" 섹션에** 다음 한 줄 추가(단순성 self-check 단락 직후):

```markdown
- AC가 정의된 task는 Red → Green → Refactor 사이클로 진행한다. opt-out 사유가 task 문서에 있고 follow-up이 같이 적혀 있을 때만 테스트 작성을 건너뛴다(정책: [ADR-009](../../docs/90-decisions/ADR-009-tdd-default.md)).
```

**출력 형식의 항목 목록에** 한 줄 추가(이미 Step 4에서 "남은 정리 항목"을 추가한 직후):

```markdown
- AC별 진행 상태 (예: AC-1 ✅, AC-2 ❌)
```

#### 8.4.7 `.claude/agents/validator-sonnet.md` 갱신

**"규칙:" 섹션에** 다음 한 줄 추가:

```markdown
- AC 항목과 실제 테스트가 1:1 또는 다대일로 매핑되는지 점검한다. 미매핑 항목은 report에 명시한다(정책: ADR-009).
```

#### 8.4.8 `CLAUDE.md` 갱신

**Step 4에서 추가한 "## 단순성·YAGNI" 섹션 직후에** 한 줄 단락 추가:

```markdown

## TDD 기본 (구현 시 디폴트)
구현은 Red → Green → Refactor 3 phase 사이클을 디폴트로 따른다. opt-out은 task 문서의 `## 6-2. TDD opt-out`에 사유와 follow-up이 모두 있을 때만. 정책 근거: [ADR-009-tdd-default.md](docs/90-decisions/ADR-009-tdd-default.md).
```

#### 8.4.9 `docs/90-decisions/ADR-009-tdd-default.md` 신규

**파일 전체 내용**:

```markdown
# ADR-009 TDD를 디폴트로 채택, opt-out 절차 정의

## 상태
accepted

## 배경
이 보일러플레이트의 가치는 fork된 미래 프로젝트에서 에이전트가 일관된 규율로 작업하는 것이다. TDD는 그 규율 중 가장 잘 작동하는 신호로, builder-sonnet이 "구현 → 사후 테스트"가 아니라 "AC 정의 → 실패 테스트 → 최소 구현 → 정리" 사이클을 따르게 만든다.

본 ADR 이전의 상태:
- TASK_TEMPLATE의 `## 6. 테스트 포인트` — 자유 텍스트 메모. AC인지 단순 메모인지 구분 없음.
- builder-sonnet 규칙: "관련 테스트를 추가하거나 보강한다" — 사후 테스트 가능, TDD 강제 없음.
- `/implement-workitem`: "필요한 테스트가 있으면 함께 보강한다" — 임의적.

## 결정
`/implement-workitem`의 디폴트 흐름을 Red → Green → Refactor 3 phase로 명시한다.

| Phase | 종료 조건 |
|-------|-----------|
| Red | task의 AC-N에 대응하는 실패 테스트가 작성되고, "원하는 이유로" 실패함을 확인 |
| Green | Red phase의 테스트가 통과하는 최소 코드가 작성됨. 다른 AC는 미리 만족시키지 않음 (YAGNI). |
| Refactor | 단순성 self-check 4항목 + Clean Code 6항목 적용. 외부 행동 미변경. 테스트 통과 유지. |

위 사이클을 task의 모든 AC가 소진될 때까지 반복.

opt-out 절차:
- task 문서의 `## 6-2. TDD opt-out` 섹션에 사유와 follow-up task 링크가 **모두** 있을 때만 적용된다.
- 둘 중 하나만 비어 있으면 형식 위반으로 표시되어 진행 불가.
- finalize 시점에 opt-out 사유를 사용자에게 명시적으로 보여주고 확인.

검증 흐름:
- `/validate-workitem`(validator-sonnet)이 AC ↔ 테스트 매핑과 테스트 선행 휴리스틱을 점검.
- 결과는 validation report에 `AC-1 ✅ / AC-2 ❌(테스트 없음)` 형태로 기록.
- `/finalize-workitem`은 통합 `validate` 명령 통과 외에 AC 미충족 항목이 있으면 `Needs Fix`로 종료.

fast 모드:
- `/implement-workitem --fast [task-id]`는 RGR 사이클을 1회만 돌려 첫 AC만 완료하고 종료. prototype에서 빠르게 흐름을 검증할 때 사용. 나머지 AC는 후속 호출.

## 근거
- 사후 테스트는 구현을 그대로 따라가서 발견 못한 케이스가 그대로 통과한다 — 회귀 위험.
- AC가 task 시작 전 명문화되지 않으면 finalize 시점의 "완료" 기준이 흐려진다.
- TDD는 단순성·YAGNI(ADR-006)를 강화한다 — 범위가 명문화되어야 "딱 그만큼"이 가능하다.
- opt-out에 follow-up을 강제하면 spike/prototype 후 정식 구현 부채가 누적되지 않는다.

## 결과
- TASK_TEMPLATE의 `## 6. 테스트 포인트` → `## 6. Acceptance Criteria` + `## 6-1. 테스트 시나리오 (TDD Red)` + `## 6-2. TDD opt-out`로 분리.
- `/implement-workitem`이 RGR 3 phase 흐름.
- builder-sonnet 규칙에 RGR 사이클 강제.
- validator-sonnet 규칙에 AC ↔ 테스트 매핑 점검.
- `/finalize-workitem` 통과 조건에 AC 미충족 0개 추가.
- CLAUDE.md에 "TDD 기본" 1단락(fork된 새 세션 자동 로드 surface).

## 후속 작업
- AC 자연어 매핑이 헐거우면 테스트 이름에 `AC_N` 식별자 컨벤션 권장 강화.
- legacy 코드 수정은 characterization test 선행 후 RGR(task 단위 사용자 결정).
- prototype 단계의 opt-out 비율을 `/stabilize-milestone`이 보고 항목으로 추가할지 후속 검토.
```

#### 8.4.10 `docs/90-decisions/README.md` 인덱스 갱신

**ADR 인덱스 표에** 한 행 추가:

```
| 009 | TDD default + opt-out | accepted | /implement-workitem 디폴트는 Red→Green→Refactor 사이클, opt-out은 사유+follow-up 모두 필요 |
```

### 8.5 검증

- [ ] `docs/30-workitems/_templates/TASK_TEMPLATE.md`에 `## 6. Acceptance Criteria`, `## 6-1. 테스트 시나리오 (TDD Red)`, `## 6-2. TDD opt-out` 세 섹션이 있다.
- [ ] `docs/30-workitems/_templates/FEATURE_TEMPLATE.md`의 `## 8. 검증 방법`에 "task 단위로는 AC로 분해된다" 안내가 있다.
- [ ] `.claude/skills/implement-workitem/SKILL.md`이 RGR 3 phase 흐름을 본문에 담고 `--fast` 인자를 처리한다.
- [ ] `.claude/skills/validate-workitem/SKILL.md`의 검증 기준에 AC↔테스트 매핑·테스트 선행 휴리스틱이 있다.
- [ ] `.claude/skills/validate-workitem/SKILL.md`의 report 양식에 `## AC ↔ 테스트 매핑` 섹션이 있다.
- [ ] `.claude/skills/finalize-workitem/SKILL.md`에 AC 미충족 점검 단계가 있다.
- [ ] `.claude/agents/builder-sonnet.md`에 RGR 사이클 규칙과 AC별 진행 상태 출력 항목이 있다.
- [ ] `.claude/agents/validator-sonnet.md`에 AC ↔ 테스트 매핑 점검 규칙이 있다.
- [ ] `CLAUDE.md`에 "TDD 기본" 단락이 있다.
- [ ] `docs/90-decisions/ADR-009-tdd-default.md`가 존재한다.
- [ ] `docs/90-decisions/README.md` 인덱스에 ADR-009 행이 있다.

### 8.6 커밋 권장

```
feat(workflow): introduce TDD default (RGR cycle, AC↔test mapping, ADR-009)
```

---

## Step 9 — /stabilize-milestone

### 9.1 목표

마일스톤 단위 종합 점검(E2E + 회귀 + 리팩토링 후보 + ADR 누락 점검)을 단일 skill로 캡슐화한다. **코드 수정·커밋·status 변경은 하지 않는다.**

### 9.2 의존성

- Step 5 (validation report) 완료 — task 단위 검증이 표준화되어 있어야 마일스톤 단위 누적이 의미를 갖는다.
- Step 6 (finalize) 완료 — task가 `done` 상태인지 점검할 수 있어야 한다.
- Step 7 (stack-guard) 완료 — 통합 `validate` 명령이 마일스톤 단위로도 호출 가능해야 한다.

### 9.3 영향 범위

- 신규 1개 skill (`.claude/skills/stabilize-milestone/SKILL.md`)
- 수정 1개 (`docs/00-meta/WORKFLOW.md`)
- 수정 1개 (`docs/00-meta/AGENT_EXECUTION_STRATEGY.md` — "스킬 실행 순서" 8번 항목 ADR 링크 보강)
- 수정 2개 (`docs/40-validation/QA_FINDINGS.md`, `docs/40-validation/IMPROVEMENT_GUIDE.md`)

### 9.4 변경 작업

#### 9.4.1 `.claude/skills/stabilize-milestone/SKILL.md` 신규

**파일 전체 내용**:

```markdown
---
name: stabilize-milestone
description: Stabilize a milestone — run E2E + regression + refactoring/ADR review. No code changes, no commits.
argument-hint: "[milestone id]"
disable-model-invocation: true
allowed-tools: Read Glob Grep Write Edit Bash Agent
---

이 skill은 **코드 수정·커밋·workitem status 변경을 하지 않는다.**
검증 결과를 `docs/40-validation/QA_FINDINGS.md`와 `docs/40-validation/IMPROVEMENT_GUIDE.md`에 누적 기록하는 문서 갱신은 정상 책임이다 — 그 외 변경은 금지한다.
후속 작업이 필요하면 `/repair-workitem` 또는 새 task로 텍스트 제안만 출력한다.

입력:
- `$ARGUMENTS`에는 milestone ID(예: `M1`)가 들어온다.

수행:
1. milestone 문서를 읽고 포함된 feature/task 목록을 회수한다.
2. 각 task의 status를 점검 — `done`이 아닌 항목이 있으면 명단을 출력하고 종료(완료를 강제하지 않음).
3. 통합 `validate` 명령을 실행한다 + (있으면) E2E 명령을 실행한다.
4. **qa agent에 회귀·엣지케이스 점검 위임** — qa는 보고만 한다(qa.md의 tools에 Write 없음). 반환된 보고를 본 skill이 받아 `docs/40-validation/QA_FINDINGS.md`에 누적 기록한다.
5. **reviewer agent에 리팩토링 후보·아키텍처 부채 점검 위임** — reviewer 입력에 Clean Code 6항목 체크리스트(ADR-006)를 명시 전달한다. reviewer도 보고만 한다. 반환된 보고를 본 skill이 받아 `docs/40-validation/IMPROVEMENT_GUIDE.md`에 정리.
   - reviewer 결과에 구조 변경이 필요해 보이면 메인 세션에 architect-opus 추가 호출을 텍스트로 제안.
6. 미흡한 ADR 후보 제안 — 마일스톤 중에 내려진 결정인데 ADR이 없는 것을 식별. ADR 후보 기준에 "layer 경계·의존성 규칙 변경"도 포함(ADR-006 정책).
7. ARCHITECTURE_OVERVIEW의 `## 3-1. 레이어 경계 + 의존성 규칙` 섹션이 비어 있고 모듈 수가 3개 이상이면 채울 것을 권장 출력한다(정책: ADR-006).
8. 최종 출력:
   - 통합 `validate` 결과 + E2E 결과 (있으면)
   - P0 / P1 / P2 후속 작업
   - QA_FINDINGS / IMPROVEMENT_GUIDE 갱신 위치
   - 다음 마일스톤으로 넘기는 항목
   - architect-opus 호출 권장 (있으면)

책임 경계:
- 코드 수정·커밋·workitem status 변경 금지.
- 누적 문서 갱신만 허용 (`QA_FINDINGS.md`, `IMPROVEMENT_GUIDE.md`).

E2E 명령이 없는 스택은 3단계에서 통합 `validate`만 돌리고 E2E는 skip한다(출력에 명시).
```

#### 9.4.2 `docs/40-validation/QA_FINDINGS.md` 양식 변경

**현재 파일 전체를 다음으로 교체**:

```markdown
# QA 결과

> 본 문서는 마일스톤 단위로 누적된다. 각 마일스톤별 P0/P1/P2/관찰 메모를 중첩 헤더로 분리한다.
> 마일스톤이 정해지지 않은 초기 프로젝트는 `## 일반` 한 묶음만 둔다.

## 다운스트림 마이그레이션 가이드
이 보일러플레이트는 빈 템플릿이라 적용 즉시 변경 가능하다. 그러나 본 보일러플레이트로 시작한 다운스트림 프로젝트가 이미 평면 양식의 누적 데이터를 가질 수 있다.
- (1) 기존 평면 항목들을 `## M1` 또는 `## 일반` 한 묶음으로 감싼다(편집 1회).
- (2) 다음 회차부터 새 마일스톤 헤더로 누적한다.

---

## 일반

### P0

### P1

### P2

### 관찰 메모
```

#### 9.4.3 `docs/40-validation/IMPROVEMENT_GUIDE.md` 양식 변경

**현재 파일 전체를 다음으로 교체**:

```markdown
# 개선 가이드

> 본 문서는 Living Doc이다. 각 섹션 안에서 `### M1`, `### M2` 식의 마일스톤 단위 그룹핑을 권장한다.
> `/stabilize-milestone`이 reviewer 결과를 누적 기록할 때 마일스톤 헤더를 사용한다.

## 0. 요약

## 1. 우선순위

## 2. 즉시 수정할 항목

## 3. 권장 리팩토링

## 4. 보류 항목
```

#### 9.4.4 `docs/00-meta/WORKFLOW.md` 갱신

**Step 6에서 추가한 "## 4-1. 마감" 섹션 직후에** 새 섹션 추가:

```markdown

## 5. 마일스톤 안정화
- `/stabilize-milestone [milestone-id]`으로 통합 점검을 수행한다.
- E2E + 회귀 + 리팩토링 후보 + ADR 후보 점검.
- **코드 수정·커밋·status 변경 금지** — 결과는 `QA_FINDINGS.md`와 `IMPROVEMENT_GUIDE.md`에 누적 기록.
- 후속 작업이 필요하면 `/repair-workitem` 또는 새 task로 연결.

다운스트림 마이그레이션: 이미 평면 양식의 QA 데이터를 가진 프로젝트는 (1) 기존 항목을 `## M1` 또는 `## 일반` 묶음으로 감싸고 (2) 다음 회차부터 새 마일스톤 헤더로 누적한다.
```

**WORKFLOW.md 헤더 재구조화**:

현재 WORKFLOW.md는 번호 매긴 섹션과 번호 없는 섹션이 섞여 있다(`## 1`, `## 2`, `## 3`, `## 4`, `## 5. 의사결정 기록`, **`## 기본 원칙`**, **`## 문서 운영 방식`**, **`## 문서 상태 전이`**, **`## 문서 충돌 해결`**, `## 6. 프로젝트별 자동화 추가`, `## 단계별 에이전트 위임`).

번호 없는 섹션은 **위치를 그대로 유지**하고 번호만 다음과 같이 시프트한다(Step 6.4.7에서 `## 워크아이템 라이프사이클`은 `## 문서 상태 전이` 직전에 삽입되었음에 주의 — 즉 unnumbered 섹션 cluster 한가운데에 위치한다):

```
## 1. 범위 정의                              (그대로)
## 2. 시스템 설계                            (그대로)
## 3. 작업 단위 분해                          (그대로)
## 4. 구현 및 검증                            (그대로, Step 5에서 본문 갱신)
## 4-1. 마감                                  ← Step 6 신규
## 5. 마일스톤 안정화                          ← 본 Step 신규 (기존 "## 5. 의사결정 기록" 자리에 삽입)
## 6. 의사결정 기록                            ← 기존 "## 5"에서 번호 시프트
## 기본 원칙                                  (그대로, 번호 없음)
## 문서 운영 방식                              (그대로, 번호 없음)
## 워크아이템 라이프사이클                     (Step 6에서 ## 문서 상태 전이 직전에 삽입, 번호 없음)
## 문서 상태 전이                              (그대로, 번호 없음)
## 문서 충돌 해결                              (그대로, 번호 없음)
## 7. 프로젝트별 자동화 추가                  ← 기존 "## 6"에서 번호 시프트
## 단계별 에이전트 위임                        (그대로, 번호 없음)
```

WORKFLOW.md 본문에서 다음 두 헤더만 텍스트 치환한다:
- `## 5. 의사결정 기록` → `## 6. 의사결정 기록`
- `## 6. 프로젝트별 자동화 추가` → `## 7. 프로젝트별 자동화 추가`

신규 `## 5. 마일스톤 안정화` 섹션은 기존 `## 6. 의사결정 기록`(번호 시프트 후) 직전, 즉 `## 4-1. 마감` 직후에 삽입한다.

**WORKFLOW.md 헤더 시프트 후 cross-reference 점검**: 다른 문서가 "WORKFLOW.md ## 5"·"WORKFLOW.md ## 6" 식으로 명시 참조하는 곳이 있는지 다음 명령으로 확인하고, 있으면 새 번호로 갱신한다.

```
git grep -n "WORKFLOW.md#5\|WORKFLOW.md ## 5\|WORKFLOW.md#6\|WORKFLOW.md ## 6" -- 'docs/' '.claude/' 'CLAUDE.md' 'README*.md'
```

(보일러플레이트 시점에서는 본 가이드 외 직접 참조가 없을 가능성이 크지만, fork된 다운스트림 프로젝트에서는 cross-ref가 생겼을 수 있다.)

#### 9.4.5 `docs/00-meta/AGENT_EXECUTION_STRATEGY.md` 갱신

**Step 5에서 갱신한 "## 스킬 실행 순서 가이드" 섹션의 항목 8을** 다음으로 교체(ADR 링크 보강):

기존:
```
8. 마일스톤의 모든 task가 `done`이 되면 `/stabilize-milestone`
```

변경 후:
```
8. 마일스톤의 모든 task가 `done`이 되면 `/stabilize-milestone` — 통합 점검(코드 수정·커밋·status 변경 금지). 정책: [ADR-007](../90-decisions/ADR-007-workitem-lifecycle.md).
```

### 9.5 검증

- [ ] `.claude/skills/stabilize-milestone/SKILL.md`가 존재하고 책임 경계(코드 수정·커밋·status 변경 금지)가 명시돼 있다.
- [ ] `docs/40-validation/QA_FINDINGS.md`가 마일스톤 단위 중첩 헤더 양식으로 변경됐고 마이그레이션 가이드 단락이 있다.
- [ ] `docs/40-validation/IMPROVEMENT_GUIDE.md`가 마일스톤 단위 그룹핑 안내 단락을 포함한다.
- [ ] `docs/00-meta/WORKFLOW.md`에 "## 5. 마일스톤 안정화" 섹션이 있고 후속 헤더 번호가 한 단계 밀려 있다.
- [ ] `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`의 항목 8에 ADR-007 링크가 있다.
- [ ] WORKFLOW.md 헤더 시프트 후 다른 문서의 cross-reference 점검 명령(9.4.4 후반)이 수행됐고, 발견된 참조가 새 번호로 갱신됐다(또는 참조가 없음을 확인).

### 9.6 커밋 권장

```
feat(skill): add /stabilize-milestone for milestone-level QA/refactor review
```

---

## Step 10 — Architecture 의존성 규칙 + stabilize-milestone 통합

### 10.1 목표

ARCHITECTURE_OVERVIEW에 의존성 규칙 섹션을 추가한다. YAGNI 보호 단서로 "프로젝트 규모가 정당화될 때만 채운다" 원칙을 동봉. `/stabilize-milestone`이 reviewer 입력에 Clean Code 6항목 체크리스트를 명시 전달하도록 통합한다.

### 10.2 의존성

- Step 4 (단순성·Clean Code 1단계) 완료 — ADR-006이 있어야 의존성 규칙 정책의 SSOT가 된다.
- Step 9 (/stabilize-milestone) 완료 — 통합할 진입점이 있어야 한다.

### 10.3 영향 범위

- 수정 1개 (`docs/20-system/ARCHITECTURE_OVERVIEW.md`)
- 수정 1개 skill (`.claude/skills/stabilize-milestone/SKILL.md`)

### 10.4 변경 작업

#### 10.4.1 `docs/20-system/ARCHITECTURE_OVERVIEW.md` 갱신

**"## 3. 상위 아키텍처" 섹션 (현재 13~16번째 줄) 직후에** 새 섹션 추가:

```markdown

## 3-1. 레이어 경계 + 의존성 규칙
<!-- 프로젝트 규모가 정당화될 때만 채운다. 단일 모듈 prototype은 비워둬도 된다.
     단, 모듈이 3개 이상으로 분리되면 채운다. (정책: ADR-006)

     기재 항목:
     - 레이어 정의 (예: Domain / Use Case / Interface Adapter / Framework)
     - 의존성 방향 (예: Domain ← Use Case ← Interface Adapter ← Framework)
     - ASCII 다이어그램 또는 화살표 그림
     - 위반 사례 1~2개 예시 (예: Framework 코드가 Domain을 직접 import하면 OK / 반대 방향은 violation)

     /stabilize-milestone이 모듈 수 ≥3 시 이 섹션 채움을 권장 출력한다. -->

```

#### 10.4.2 `.claude/skills/stabilize-milestone/SKILL.md` 갱신

**Step 9에서 신규 작성한 본문의 "5. reviewer agent에 ... 위임" 항목**을 다음으로 강화 — Step 9에서 이미 Clean Code 6항목 명시 입력 전달이 들어 있다면 그대로 유지. 추가로 "6. 미흡한 ADR 후보 제안" 항목에 다음 한 줄 추가:

```markdown
   - layer 경계·의존성 규칙 변경(ARCHITECTURE_OVERVIEW의 ## 3-1)이 마일스톤 중에 발생했으면 ADR 후보로 표시한다(정책: ADR-006).
```

**"7. ARCHITECTURE_OVERVIEW의 ## 3-1 ..." 항목**(Step 9 본문에 이미 들어 있음)은 그대로 유지.

### 10.5 검증

- [ ] `docs/20-system/ARCHITECTURE_OVERVIEW.md`에 `## 3-1. 레이어 경계 + 의존성 규칙` 섹션이 있고 YAGNI 보호 단서("프로젝트 규모가 정당화될 때만 채운다")가 첫 줄에 있다.
- [ ] `.claude/skills/stabilize-milestone/SKILL.md`의 ADR 후보 기준에 "layer 경계·의존성 규칙 변경"이 들어 있다.
- [ ] `.claude/skills/stabilize-milestone/SKILL.md`가 ARCHITECTURE의 `## 3-1` 자동 채움 권장을 포함한다.

### 10.6 커밋 권장

```
docs(arch): add layer dependency section (## 3-1) and integrate Clean Code review in stabilize
```

---

## Step 11 — /discover-product 신설

### 11.1 목표

발굴 단계(persona / pain / JTBD / 시나리오)를 별도 skill로 분리해, charter 생성과 발굴 책임을 분리한다. discovery 산출물은 `docs/10-charter/DISCOVERY.md`에 보존되어 mid-project 재발굴·재활용이 가능하다.

### 11.2 의존성

- Step 3 (SSOT) 완료 — STRUCTURE.md에 discovery 산출물이 등록되어 있어야 한다(이미 등록됨).

### 11.3 영향 범위

- 신규 1개 skill (`.claude/skills/discover-product/SKILL.md` + 보조 templates)
- 신규 2개 (`.claude/skills/discover-product/persona-template.md`, `.claude/skills/discover-product/pain-template.md`)
- 신규 1개 (`docs/10-charter/_templates/DISCOVERY_TEMPLATE.md`)
- 수정 1개 (`docs/00-meta/BOOTSTRAP_PROMPT_EXAMPLES.md`)
- 수정 1개 (`docs/00-meta/NEW_PROJECT_CHECKLIST.md` — discovery 단계 분리 반영)

### 11.4 변경 작업

#### 11.4.1 `.claude/skills/discover-product/SKILL.md` 신규

> frontmatter 결정: `Agent` 권한이 필요한 이유는 본문이 architect-opus 단발 sub-call(R0의 페르소나 후보 생성, R1의 pain inventory 정렬)을 띄우기 때문이다. [공식 permissions 문서](https://code.claude.com/docs/en/permissions)는 `Agent(AgentName)` 패턴을 deny 컨텍스트 예시(`Agent(Explore)`, `Agent(my-custom-agent)` 등)로 명시하지만, skill frontmatter `allowed-tools`처럼 allow 컨텍스트에서의 동작은 직접 demonstrate되지 않았다. 안전하게 plain `Agent`로 두고 호출 대상 제한은 본문(actionable prompt)에 둔다 — 적용 후 `Agent(architect-opus)`로 좁히는 것은 사용자가 실측 검증 후 별도로 결정한다. `context: fork`/`agent` 라인은 두지 않는다 — 메인 세션이 R0~R4 라운드를 직접 운전한다.

**파일 전체 내용**:

```markdown
---
name: discover-product
description: Run a multi-round discovery (persona / pain / JTBD / scenario / MVP / assumptions) and write DISCOVERY.md.
argument-hint: "[product description | --fast]"
disable-model-invocation: true
allowed-tools: Read Glob Grep Write Edit Agent
---

이 skill은 메인 세션이 R0~R4 라운드를 직접 운전하는 절차서다.
무거운 추론(R0의 페르소나 후보, R1의 pain inventory)은 architect-opus 단발 sub-call로 위임한다.
R0~R3 산출물은 메인 컨텍스트에 누적시키지 않고 `docs/10-charter/DISCOVERY.md`에 누적 적재한다.

**트레이드오프 명시** — 이 skill은 의도적으로 fork 격리를 풀고 메인 세션이 사용자 인터랙션을 직접 운전한다. R0~R3 산출물은 `DISCOVERY.md`에 적재해 메인 컨텍스트 부담을 최소화하지만, 라운드 요약 일부는 메인에 남는다. 종료 후 사용자가 `/clear` 또는 새 세션으로 컨텍스트를 정리할 것을 권장한다.

입력:
- `$ARGUMENTS`에 자연어 설명이 들어온다.
- `--fast` 플래그가 있으면 R0의 페르소나 후보 제시·선택을 건너뛰고 R3(가정 정리)도 생략한 채 R1+R2+R4만 도는 단축 흐름. 단, R1은 페르소나 입력이 필요하므로 다음 절차로 페르소나를 확보한다:
  1. `$ARGUMENTS`에 페르소나가 한 줄이라도 명시되어 있으면 그것을 그대로 사용한다(예: `--fast 회고 SaaS — 페르소나: 직장인 개인 사용자`).
  2. 명시되지 않았으면 architect-opus 단발 sub-call로 페르소나 후보 1개를 *자동 선정*하고, 해당 페르소나를 가정으로 표시해 R1을 진행한다(출력에 "이 페르소나는 fast 모드의 R0 사용자 확인 단계를 생략한 자동 선정 결과이며, 가정으로 취급한다"를 명시).
  3. 사용자가 마음에 들지 않으면 `/discover-product` (fast 없이)로 재실행하면 된다는 안내를 출력에 포함.

사용자 응답 수단:
- 라운드별 응답은 자연어로만 받는다(`AskUserQuestion`은 [공식 문서](https://code.claude.com/docs/en/agent-sdk/user-input)의 Limitations 섹션 기준 sub-agent에서 사용 불가).
- 매 라운드 끝에 `skip` / `good` / `refine: ...`로 응답할 수 있다.

라운드 구성:

**R0 — 문제 한 줄 + 페르소나 확인**
- 입력 한 줄을 그대로 되돌려 "이 한 문장이 핵심 맞나" 확인.
- 동시에 페르소나 후보 2~3개를 한 단락씩 제시(직무·맥락·일과·압력 요소). architect-opus 단발 sub-call로 위임해 후보 생성.
- 사용자가 1개를 고르거나 합쳐달라고 한다.

**R1 — 핵심 pain + JTBD + 시나리오**
- 선택된 페르소나의 pain 6~10개를 brainstorm 후 빈도×고통으로 정렬. architect-opus 단발 sub-call.
- 상위 1~3개를 사용자가 고른다.
- 가장 큰 pain의 JTBD(Jobs To Be Done) 한 줄, happy path/alternate path/fail path를 5~7단계로 같이 적는다.
- 사용자가 끊을 지점·수용 가능 fail을 정한다.

**R2 — MVP 범위 vs 비범위 / 성공 기준**
- R1 시나리오를 충족시키는 최소 기능 묶음과 의도적으로 미루는 것을 분리.
- 측정 가능한 성공 기준 1~3개.

**R3 — 핵심 가정 + 열린 질문**
- R0~R2에서 사용자가 추측으로 답한 모든 항목을 가정으로 표시.
- 가장 위험한 가정 1~3개에 검증 방법 1줄.

**R4 — DISCOVERY.md 정리(저장 단계)**
- 위 결과를 `docs/10-charter/_templates/DISCOVERY_TEMPLATE.md` 양식에 맞춰 `docs/10-charter/DISCOVERY.md`에 저장.
- 이 skill은 charter/architecture/workitem을 만들지 않는다 — `/bootstrap-project`가 이어 수행한다(자동 호출 아님).

**단계별 출구 보장**: 어느 라운드에서 멈춰도 그때까지의 산출물이 `DISCOVERY.md`에 들어가 `/bootstrap-project`의 입력으로 의미가 있다.

**다국어**: 입력 언어를 따른다. 한국어 입력이면 산출물도 한국어, 영문 입력이면 영문.

마지막 출력:
- DISCOVERY.md 경로
- 핵심 가정과 열린 질문 요약
- 다음 권장 단계 (`/bootstrap-project` — DISCOVERY.md를 입력으로 사용)
```

#### 11.4.2 `.claude/skills/discover-product/persona-template.md` 신규

**파일 전체 내용**:

```markdown
# Persona Card Template

## 역할
[직무 / 직책]

## 맥락
[일하는 환경, 사용 도구, 외부 압력]

## 하루 흐름
[일과 중 어떤 흐름으로 시간을 보내는지 — 시간대별 활동 1~2줄씩]

## 현재 해결 방법
[지금 같은 문제를 어떻게 우회·해결하는지 — 도구, 절차, 사람]

## 실패 패턴
[현재 방법이 깨지는 지점 — 누락, 시간 부족, 정보 분산 등]

## 측정 가능한 성공 신호
[페르소나 입장에서 "이 제품이 도움이 됐다"고 판단할 수 있는 측정 가능 신호]
```

#### 11.4.3 `.claude/skills/discover-product/pain-template.md` 신규

**파일 전체 내용**:

```markdown
# Pain Inventory Table

| Pain | 빈도 | 고통 | 현재 우회 방법 | 해결 시 효과 |
|------|------|------|----------------|--------------|
| 예: 매일 이력서 버전 관리 혼선 | 매일 | 중 | 파일명에 날짜 접미사 | 시간 절감 + 실수 감소 |
|  |  |  |  |  |
|  |  |  |  |  |

## 사용 방법
- 빈도: 매일 / 매주 / 가끔
- 고통: 상 / 중 / 하 (사용자가 표현한 표현 그대로 매핑)
- 빈도 × 고통이 큰 항목 상위 1~3개를 R1에서 핵심 pain으로 선정한다.
```

#### 11.4.4 `docs/10-charter/_templates/DISCOVERY_TEMPLATE.md` 신규

(디렉터리 `docs/10-charter/_templates/`가 없으면 함께 생성)

**파일 전체 내용**:

```markdown
# Discovery: <프로젝트 이름>

## 0. Status
draft

## 1. 문제 한 줄
<!-- R0: 무엇이 핵심 문제인가 — 한 문장 -->

## 2. 페르소나
<!-- R0: 선택된 페르소나(persona-template.md 양식). 페르소나가 2명 이상이면 ### Persona A, ### Persona B로 분리. -->

## 3. Pain Inventory
<!-- R1: pain-template.md 양식의 표. 빈도 × 고통으로 정렬. -->

## 4. 핵심 Pain (상위 1~3개)
<!-- R1: 사용자가 선택한 핵심 pain. -->

## 5. JTBD (Jobs To Be Done)
<!-- R1: 가장 큰 pain의 JTBD 한 줄. -->

## 6. 시나리오
<!-- R1: happy path / alternate path / fail path 각 5~7단계. -->

### Happy path

### Alternate path

### Fail path
<!-- 사용자가 끊을 지점과 수용 가능한 fail을 명시 -->

## 7. MVP 범위
<!-- R2: 최소 기능 묶음 -->

## 8. MVP 비범위
<!-- R2: 의도적으로 미루는 것 -->

## 9. 성공 기준
<!-- R2: 측정 가능한 1~3개 -->

## 10. 핵심 가정
<!-- R3: 추측으로 답한 항목들. 가장 위험한 1~3개에 검증 방법 1줄 동봉. -->

## 11. 열린 질문
<!-- R3: 아직 답이 없는 중요한 질문 -->
```

#### 11.4.5 `docs/00-meta/BOOTSTRAP_PROMPT_EXAMPLES.md` 갱신

**파일 끝에** 다음 섹션 추가:

```markdown

## /discover-product 라운드 인터랙션 예시

\`\`\`
사용자: /discover-product 개인 회고 SaaS. 사용자는 하루/주간 회고를 기록하고, 원인 분석과 개선 추적을 한다.

[R0] 메인이 한 줄을 되돌리고 페르소나 후보 3개 제시
사용자: "1번 + 직장인 강조로 합쳐줘"

[R1] 메인이 pain 8개를 빈도×고통으로 정렬해 제시
사용자: "1번, 3번이 핵심"
[R1 계속] JTBD 한 줄 + happy/alternate/fail 5단계 작성

[R2] MVP 범위 vs 비범위, 성공 기준 1~3개
사용자: "good"

[R3] 가정 5개, 위험한 가정 2개에 검증 방법 1줄
사용자: "good"

[R4] DISCOVERY.md 작성
출력: docs/10-charter/DISCOVERY.md 저장됨. 다음 액션: /bootstrap-project (DISCOVERY.md 사용)
\`\`\`
```

#### 11.4.6 `docs/00-meta/NEW_PROJECT_CHECKLIST.md` 갱신

`/discover-product` 도입에 따라 새 프로젝트 시작 절차에 발굴 단계를 명시한다.

**"## 1. 저장소 복제 직후" 섹션 (현재 3~8번째 줄)을** 다음으로 교체:

```markdown
## 1. 저장소 복제 직후
- [ ] (선택) `/discover-product [프로젝트 설명]`을 실행해 페르소나·pain·JTBD·시나리오를 발굴하고 `docs/10-charter/DISCOVERY.md`를 생성했다 — charter 신뢰도가 중요한 새 프로젝트에 권장. 빠른 prototype에서는 건너뛸 수 있다.
- [ ] Claude Code에서 `/bootstrap-project [프로젝트 설명 또는 DISCOVERY.md 사용]`을 실행했다
- [ ] `README.md`가 새 프로젝트 기준으로 갱신되었다
- [ ] `docs/10-charter/PROJECT_CHARTER.md`가 새 프로젝트 내용으로 채워졌다(DISCOVERY.md를 사용한 경우 페르소나·시나리오·핵심 가정 섹션이 함께 채워졌다)
- [ ] `docs/20-system/ARCHITECTURE_OVERVIEW.md`가 초기 구조를 반영한다
- [ ] 첫 milestone/feature 문서가 생성되었다
```

**"## 권장 원칙" 섹션 (현재 36~39번째 줄)의 첫 줄**:

기존:
```
- 먼저 수동으로 여러 문서를 고치기보다 /bootstrap-project를 먼저 실행한다.
```

변경 후:
```
- charter 신뢰도가 중요한 프로젝트는 `/discover-product`로 발굴 단계를 먼저 거친다. 그 외에는 `/bootstrap-project`로 바로 시작해도 된다.
- 먼저 수동으로 여러 문서를 고치기보다 위 두 skill 중 하나로 시작한다.
```

### 11.5 검증

- [ ] `.claude/skills/discover-product/SKILL.md`가 존재하고 R0~R4 라운드 + `--fast` 모드 + 트레이드오프 단락이 있다.
- [ ] `.claude/skills/discover-product/persona-template.md`, `pain-template.md`가 존재한다.
- [ ] `docs/10-charter/_templates/DISCOVERY_TEMPLATE.md`가 존재하고 11개 섹션을 담는다.
- [ ] `docs/00-meta/BOOTSTRAP_PROMPT_EXAMPLES.md`에 `/discover-product` 라운드 인터랙션 예시가 있다.
- [ ] `docs/00-meta/NEW_PROJECT_CHECKLIST.md` "1. 저장소 복제 직후" 섹션에 `/discover-product` (선택) 항목과 "DISCOVERY.md 사용" 분기가 들어 있다.

### 11.6 커밋 권장

```
feat(skill): add /discover-product with multi-round persona/pain/JTBD discovery
```

---

## Step 12 — /bootstrap-project 입력 조정

### 12.1 목표

`/bootstrap-project`의 책임을 "discovery 산출물 → charter/architecture/M1/F-001 변환"으로 좁힌다. 재실행 시 갱신 모드(diff 요약 + 사용자 확인)를 명시한다.

### 12.2 의존성

- Step 11 (/discover-product) 완료 — DISCOVERY.md를 입력으로 받을 수 있어야 한다.

### 12.3 영향 범위

- 수정 1개 skill (`.claude/skills/bootstrap-project/SKILL.md`)
- 수정 1개 (`.claude/skills/bootstrap-project/output-checklist.md`)
- 수정 1개 (`docs/10-charter/PROJECT_CHARTER.md` 템플릿)
- 수정 2개 (`README.md`, `README_ko.md`)

### 12.4 변경 작업

#### 12.4.1 `.claude/skills/bootstrap-project/SKILL.md` 갱신

**frontmatter (현재 1~11번째 줄)에서**: Step 1에서 model이 `opus`로 변경되어 있다. 본 Step에서는 frontmatter를 그대로 유지(0.4 결정 — model 라인은 별칭이라 staleness 없음).

**`description:` 필드를** 다음으로 교체:

기존:
```yaml
description: Initialize a new project from this boilerplate using one natural-language project brief.
```

변경 후:
```yaml
description: Convert discovery output (DISCOVERY.md) or a natural-language brief into charter/architecture/M1/F-001. Re-run safe with update mode.
```

**`argument-hint:` 필드도 함께 갱신**해 `--apply` 인자 노출:

기존:
```yaml
argument-hint: "[project brief]"
```

변경 후:
```yaml
argument-hint: "[project brief or empty (uses DISCOVERY.md)] [--apply]"
```

**본문 "입력:" 섹션 (현재 15~18번째 줄, line 15가 `입력:` 헤더)을** 다음으로 교체:

```markdown
입력 우선순위:
1. `$ARGUMENTS`에 brief 내용이 있으면(비어 있지 않으면) 그것을 우선 입력으로 사용한다. `docs/10-charter/DISCOVERY.md`가 함께 있으면 보조 컨텍스트로만 참조하고, 둘이 어긋나면 출력에 명시한다(silent override 금지).
2. `$ARGUMENTS`가 비어 있고 `docs/10-charter/DISCOVERY.md`가 있으면 그것을 입력으로 사용한다.
3. 둘 다 없으면 `/discover-product` 선행 또는 brief 입력을 안내하고 종료한다(강제 진행하지 않는다).

이 skill은 발굴이 아니라 변환을 한다 — 발굴은 `/discover-product`에서.
```

**본문 "반드시 수행할 일:" 섹션 (현재 30~41번째 줄)을** 다음으로 교체:

```markdown
반드시 수행할 일:
1. 입력 회수 — DISCOVERY.md 또는 자연어 입력.
2. 기존 산출물(charter/architecture/M1/F-001) 존재 여부 점검.
   - 없으면 새로 생성.
   - 있으면 **갱신 모드** — 본 skill은 `context: fork`에서 실행되므로 사용자에게 실시간 확인을 받을 수 없다.
     - `--apply` 인자가 있으면: 기존 산출물을 읽고 architect-opus로 갱신본을 생성해 즉시 반영한다.
     - `--apply` 인자가 없으면: 기존 산출물을 읽고 갱신 제안 diff를 출력에만 표시하고 **종료**한다(파일 수정 없음). 사용자가 검토 후 `/bootstrap-project --apply ...`로 재실행하거나, 메인 세션에서 architect-opus를 직접 호출해 부분 반영한다.
3. 현재 architect-opus agent가 산출물을 직접 생성/갱신한다 — 입력은 DISCOVERY.md 또는 자연어 입력 + 기존 산출물(있으면). 본 skill은 frontmatter `agent: architect-opus` + `context: fork`로 이미 architect-opus 컨텍스트에서 fork되어 실행되므로 별도 sub-call이 필요 없고 `Agent` 권한도 보유하지 않는다.
4. 다음 산출물을 갱신한다.
   - `README.md`
   - `docs/10-charter/PROJECT_CHARTER.md`
   - `docs/20-system/ARCHITECTURE_OVERVIEW.md`
5. 필요하면 다음도 함께 갱신.
   - `docs/20-system/DESIGN_SYSTEM.md`
   - `docs/90-decisions/ADR-002-initial-project-decisions.md`
6. 최초 workitem 문서를 만든다.
   - `docs/30-workitems/milestones/M1-foundation.md`
   - `docs/30-workitems/features/F-001-core-value.md`
```

#### 12.4.2 `.claude/skills/bootstrap-project/output-checklist.md` 갱신

**"## 출력 원칙" 섹션 끝에** 다음 항목 추가:

```markdown
- 갱신 모드 흐름 — 기존 산출물이 있으면 diff 요약을 사용자에게 보여주고 확인 후 반영. `--apply` force 모드는 명시 인자가 있을 때만.
- 발굴은 `/discover-product`가 책임이다. bootstrap-project는 변환을 한다.
```

#### 12.4.3 `docs/10-charter/PROJECT_CHARTER.md` 템플릿 갱신

**"## 2. 대상 사용자" 섹션 (현재 9~10번째 줄) 직후에** 새 섹션 추가:

```markdown

## 2.1 페르소나
<!-- DISCOVERY.md의 페르소나를 그대로 박는다. 페르소나가 2명 이상이면 ### Persona A, ### Persona B로 분리. -->

```

**"## 3. 해결하려는 문제" 섹션 (이제 16번째 줄 안팎 — 앞 삽입 4줄을 반영)** 직후에 새 섹션 추가:

```markdown

## 3.1 핵심 시나리오 (happy / alternate / fail)
<!-- DISCOVERY.md의 시나리오를 그대로 박는다. -->

```

**"## 9. 열린 질문" 섹션 직전(앞선 두 삽입을 반영하면 약 38번째 줄)에** `## 9. 핵심 가정` 섹션을 새로 추가한다. 기존 `## 9. 열린 질문`은 본문을 그대로 두고 헤더 번호만 `## 10. 열린 질문`으로 시프트한다(상위 번호 시프트 — 8과 9는 의미상 sibling이므로 서브섹션 8.1보다 새 ## 9이 자연스럽다). 결과는 다음과 같다:

```markdown

## 9. 핵심 가정
<!-- DISCOVERY.md의 핵심 가정을 그대로 박는다. 가장 위험한 가정에는 검증 방법을 함께 적는다. -->

## 10. 열린 질문
<!-- 아직 답이 없는 중요한 질문들. -->
```

> 주: 위 코드블록의 `## 10. 열린 질문` 본문은 현재 PROJECT_CHARTER.md `## 9. 열린 질문`의 코멘트(`<!-- 아직 답이 없는 중요한 질문들. -->`)를 그대로 옮긴 것이다 — 헤더 번호만 9 → 10으로 시프트, 본문 변경 없음. 본 가이드의 다른 ADR/skill 본문에서 PROJECT_CHARTER 섹션 번호를 직접 참조하는 곳은 없으므로(2026-05-03 기준) 시프트로 인한 dangling 참조는 없다.

#### 12.4.4 `README.md` 갱신

**"## Overall Flow" 섹션의 한 줄 그림(Step 3에서 보존된 `/bootstrap-project → /bootstrap-stack → /plan-workitem → /implement-workitem → /validate-workitem → qa/reviewer`)을** 다음 새 흐름으로 교체한다(README에 두 다른 흐름이 공존하지 않도록 단일 SSOT 유지):

```
/discover-product → /bootstrap-project → /bootstrap-stack → /stack-guard → /plan-workitem → /implement-workitem → /validate-workitem → /finalize-workitem (or /repair-workitem) → /stabilize-milestone
```

**그리고 같은 "## Overall Flow" 섹션 본문 끝(링크 직후)에** 다음 단락을 추가한다:

```markdown

`/discover-product` is recommended for new projects to ground charter in concrete persona/pain/scenarios. It writes `docs/10-charter/DISCOVERY.md`, which `/bootstrap-project` then converts into charter/architecture/initial workitems. For a quick prototype, you can skip `/discover-product` and pass a natural-language brief directly to `/bootstrap-project`.
```

> 주: 기존 IMPROVE-GUIDE 초안의 "Recommended Flow" 신설 안은 폐기. Overall Flow 한 곳에서 단일 흐름을 유지한다.

#### 12.4.5 `README_ko.md` 갱신

**"## 전체 흐름" 섹션의 한 줄 그림을** README.md와 동일한 새 흐름으로 교체하고, 본문 끝에 다음 단락을 추가한다:

```markdown

새 프로젝트는 `/discover-product`로 페르소나·pain·시나리오를 먼저 발굴해 charter의 신뢰도를 높이는 것을 권장한다. 발굴 결과는 `docs/10-charter/DISCOVERY.md`에 저장되고, `/bootstrap-project`가 이를 charter/architecture/초기 workitem으로 변환한다. 빠른 prototype에서는 `/discover-product`를 건너뛰고 `/bootstrap-project`에 자연어 설명을 바로 줘도 된다.
```

### 12.5 검증

- [ ] `.claude/skills/bootstrap-project/SKILL.md`이 입력 우선순위(`$ARGUMENTS` brief 우선 → 비었을 때 DISCOVERY.md), 갱신 모드 흐름을 본문에 담는다(`--apply` 인자 처리 포함).
- [ ] `.claude/skills/bootstrap-project/output-checklist.md`에 갱신 모드 항목이 있다.
- [ ] `docs/10-charter/PROJECT_CHARTER.md`에 `## 2.1 페르소나`, `## 3.1 핵심 시나리오`, `## 9. 핵심 가정` 섹션이 있고, 기존 `## 9. 열린 질문`이 `## 10. 열린 질문`으로 시프트됐다.
- [ ] `README.md`의 `## Overall Flow`와 `README_ko.md`의 `## 전체 흐름`이 모두 새 흐름(`/discover-product → ... → /stabilize-milestone`)으로 교체됐고, "Recommended Flow"/"권장 흐름" 같은 별도 섹션이 신설되지 않았다(단일 SSOT).

### 12.6 커밋 권장

```
feat(skill): bootstrap-project reads DISCOVERY.md, supports update mode
```

---

## Step 13 — /batch + worktree 가이드

### 13.1 목표

위임 트리거의 `/batch 또는 worktree` 한 줄을 병렬 패턴 3종으로 분리해, bundled `/batch`가 Claude Code 공식 skill임을 명시하고 사용 시점을 정의한다.

### 13.2 의존성

- Step 3 (SSOT) 완료 — AGENT_EXECUTION_STRATEGY.md가 위임 트리거의 SSOT다. CLAUDE.md는 이미 슬림화되어 정의 대신 링크를 두고 있어야 한다.
- (권장) Step별 PR을 독립 운영하는 경우 — Step 4·8 적용 후 CLAUDE.md 최종 상태에서 13.5 검증 항목("CLAUDE.md가 위임 트리거를 정의하지 않음")이 성립하는지 함께 확인한다. 순차 적용 시에는 자동 충족.

### 13.3 영향 범위

- 수정 1개 (`docs/00-meta/AGENT_EXECUTION_STRATEGY.md`) — 위임 트리거 표 + 병렬 패턴 3종 섹션 추가가 핵심 변경.
- 조건부 수정 (`CLAUDE.md`) — Step 3 슬림화로 위임 트리거 정의가 이미 빠졌으면 변경 없음. 어떤 형태로든 남아 있으면 13.4.2의 한 줄 정리만 적용.

### 13.4 변경 작업

#### 13.4.1 `docs/00-meta/AGENT_EXECUTION_STRATEGY.md` 갱신

**"## 위임 트리거" 표 (현재 22~33번째 줄)의 마지막에서 두 번째 행** "독립적인 여러 task 동시 처리"를 다음으로 교체:

기존:
```
| 독립적인 여러 task 동시 처리 | /batch + worktree | 같은 파일 변경은 병렬화하지 않음 |
```

변경 후:
```
| 독립적인 여러 task 동시 처리 | 병렬 패턴 3종 (아래 단락 참조) | 가벼운 → 무거운 순으로 선택 |
```

**"## 병렬 작업 원칙" 섹션 (현재 54~57번째 줄)의 첫 항목** "서로 독립적인 작업은 ..."을 다음으로 교체:

기존:
```
- 서로 독립적인 작업은 `/batch` 또는 background subagent + worktree를 우선 고려한다
```

변경 후:
```
- 서로 독립적인 작업은 아래 "병렬 패턴 3종" 중 작업의 독립성·격리 필요성에 맞는 패턴을 선택한다
```

**"## 병렬 작업 원칙" 섹션 직후에** 새 섹션 추가:

```markdown

## 병렬 패턴 3종

가벼운 순으로 정리한다. 작업의 독립성과 격리 필요성에 맞춰 선택한다.

| # | 패턴 | 설명 | 적합한 작업 |
|---|------|------|-------------|
| 1 | 한 메시지에서 Agent 호출 병렬 | 메인이 한 turn에 Agent 도구를 여러 번 호출. 결과만 통합. | 일상 위임의 기본. 독립적인 짧은 sub-agent 작업 여러 개. |
| 2 | `isolation: "worktree"` 인자로 단일 Agent 호출 | Agent 도구 호출 시 인자로 격리 git worktree 지정. 변경 없으면 자동 cleanup, 있으면 worktree 경로·브랜치를 결과에 포함. | 같은 파일 충돌 가능성 있는 단일 작업. |
| 3 | bundled `/batch` 호출 | Claude Code의 **공식 bundled skill**. 사용자가 직접 발화. 작업 단위당 background agent + worktree 자동 생성. | 큰 마이그레이션·codebase-wide 변경 같은 코드 단위 분리가 분명한 큰 작업. |

선택 기준 — 가벼운 병렬: 1, 같은 파일 충돌 가능성 있는 단일 작업: 2, 작업 단위가 분명한 codebase-wide 분산 작업: 3.

`/batch`가 본 보일러플레이트의 custom skill이 아니라 Claude Code bundled skill임을 명시한다([commands 문서](https://code.claude.com/docs/en/commands)에 `[Skill]` 표시).
```

#### 13.4.2 `CLAUDE.md` 갱신

Step 3에서 슬림화된 CLAUDE.md는 위임 트리거 표를 갖고 있지 않다(링크만 둔다). 본 Step에서는 CLAUDE.md에 별도 변경이 없다 — AGENT_EXECUTION_STRATEGY.md가 SSOT.

만약 Step 3에서 CLAUDE.md에 위임 트리거가 어떤 형태로든 남아 있다면, 본 Step에서 다음 한 줄로 추가 정리한다:

```markdown
## 에이전트 실행 원칙
에이전트 실행 전략과 위임 트리거(병렬 패턴 3종 포함)는 [AGENT_EXECUTION_STRATEGY.md](docs/00-meta/AGENT_EXECUTION_STRATEGY.md)가 SSOT다.
```

### 13.5 검증

- [ ] `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`의 위임 트리거 표에서 `/batch + worktree` 행이 "병렬 패턴 3종" 참조로 교체됐다.
- [ ] `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`에 "## 병렬 패턴 3종" 섹션이 있고 `/batch`가 bundled임을 명시한다.
- [ ] `CLAUDE.md`가 위임 트리거를 정의하지 않고 AGENT_EXECUTION_STRATEGY.md 링크만 가리킨다.

### 13.6 커밋 권장

```
docs: split /batch+worktree into 3 parallel patterns (bundled skill annotation)
```

---

## 부록 A — 적용 후 통합 체크리스트

본 가이드의 모든 Step을 적용한 후, 다음을 모두 만족하는지 최종 점검한다.

### A.1 신규 파일

- [ ] `docs/00-meta/STRUCTURE.md`
- [ ] `docs/90-decisions/README.md` (인덱스)
- [ ] `docs/90-decisions/ADR-004-model-alias-policy.md`
- [ ] `docs/90-decisions/ADR-005-ssot.md`
- [ ] `docs/90-decisions/ADR-006-simplicity-and-architecture.md`
- [ ] `docs/90-decisions/ADR-007-workitem-lifecycle.md`
- [ ] `docs/90-decisions/ADR-008-commit-convention.md`
- [ ] `docs/90-decisions/ADR-009-tdd-default.md`
- [ ] `docs/40-validation/reports/.gitkeep`
- [ ] `docs/10-charter/_templates/DISCOVERY_TEMPLATE.md`
- [ ] `.claude/skills/discover-product/SKILL.md` + `persona-template.md` + `pain-template.md`
- [ ] `.claude/skills/repair-workitem/SKILL.md`
- [ ] `.claude/skills/finalize-workitem/SKILL.md`
- [ ] `.claude/skills/stack-guard/SKILL.md`
- [ ] `.claude/skills/stabilize-milestone/SKILL.md`

### A.2 삭제된 파일

- [ ] `.claude/skills/write-charter/` (디렉터리 전체)
- [ ] `.claude/skills/write-architecture/` (디렉터리 전체)

### A.3 수정 — frontmatter 모델 라인

- [ ] `.claude/settings.json`: `"model": "sonnet"`
- [ ] `.claude/agents/architect-opus.md`: `model: opus`
- [ ] `.claude/skills/bootstrap-project/SKILL.md`: `model: opus`
- [ ] `.claude/skills/bootstrap-stack/SKILL.md`: `model: opus`
- [ ] `git grep "claude-opus-4-6" -- ':!IMPROVE-GUIDE.md'` 결과 없음
- [ ] `git grep "claude-sonnet-4-6" -- ':!IMPROVE-GUIDE.md'` 결과 없음

### A.4 수정 — agent 본문

- [ ] `.claude/agents/builder-sonnet.md`: 단순성 self-check 4항목, RGR 사이클 규칙, finalize 가드, AC별 진행 상태 출력
- [ ] `.claude/agents/validator-sonnet.md`: tools에 `Write` 추가, "판정 + report 기록 전용" 명시, AC↔테스트 매핑 점검 규칙, premature factory 점검
- [ ] `.claude/agents/reviewer.md`: Clean Code 6항목 체크리스트
- [ ] `.claude/agents/architect-opus.md`: 모델 별칭, "프로젝트 규모 정당화" self-check
- [ ] `.claude/agents/qa.md`, `.claude/agents/planner.md`: 본 가이드에서는 본문 변경 없음(SSOT 패턴 기준 ADR 링크만 박는 후속 정리는 별도 항목으로)

### A.5 수정 — skill 본문

- [ ] `plan-workitem`: agent 바인딩 (planner)
- [ ] `review-doc`: agent 바인딩 (reviewer)
- [ ] `bootstrap-project`: 입력 우선순위(`$ARGUMENTS` brief 우선 → 비었을 때 DISCOVERY.md), 갱신 모드
- [ ] `bootstrap-stack`: 마지막 출력에 `/stack-guard` 안내
- [ ] `implement-workitem`: RGR 3 phase, `--fast` 인자, opt-out 흐름, 단순성 self-check 참조
- [ ] `validate-workitem`: 통합 명령 실행, report 기록(AC↔테스트 매핑 포함), 다음 액션 텍스트 제안
- [ ] `boilerplate-context`: 슬림화

### A.6 수정 — 운영 문서

- [ ] `CLAUDE.md`: 슬림화(약 30~40줄), 단순성 5개 항목, TDD 1줄, ADR 링크
- [ ] `README.md`, `README_ko.md`: Structure/Overall Flow/Guardrail 슬림화, Overall Flow를 새 풀 흐름(`/discover-product → ... → /stabilize-milestone`)으로 교체, "두 README 동시 갱신" 코멘트
- [ ] `docs/00-meta/TEMPLATE_GUIDE.md`: "새 프로젝트 시작 순서" → NEW_PROJECT_CHECKLIST 링크, 상태값 → WORKFLOW 링크, Guardrail → GUARDRAILS_STRATEGY 링크, "산출물 인벤토리" → STRUCTURE.md 링크
- [ ] `docs/00-meta/WORKFLOW.md`: `/validate-workitem` / `/repair-workitem` / `/finalize-workitem` / `/stabilize-milestone` 단계 반영, 워크아이템 라이프사이클 한 줄 그림
- [ ] `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`: 스킬 실행 순서 갱신, 병렬 패턴 3종, mid-project 동선, 모델 표기 정책
- [ ] `docs/00-meta/GUARDRAILS_STRATEGY.md`: `/stack-guard` 1단계 산출물 범위 + 2단계 prototyping 후 분리 명시
- [ ] `docs/00-meta/GLOSSARY.md`: 의도적 비우기 단락
- [ ] `docs/00-meta/BOOTSTRAP_PROMPT_EXAMPLES.md`: `/discover-product` 라운드 인터랙션 예시
- [ ] `docs/00-meta/NEW_PROJECT_CHECKLIST.md`: discovery 단계 분리 반영 — Step 11.4.6에서 "1. 저장소 복제 직후" + "권장 원칙" 갱신
- [ ] `docs/30-workitems/README.md`: "상태 관리 예시" → WORKFLOW 링크
- [ ] `docs/30-workitems/_templates/TASK_TEMPLATE.md`: `## 4-1. 변경 예정 파일/경로`, `## 6. Acceptance Criteria`, `## 6-1. 테스트 시나리오 (TDD Red)`, `## 6-2. TDD opt-out`, "관련 문서" placeholder
- [ ] `docs/30-workitems/_templates/FEATURE_TEMPLATE.md`: 검증 방법 안내 추가, "관련 문서" placeholder
- [ ] `docs/20-system/ARCHITECTURE_OVERVIEW.md`: `## 3-1. 레이어 경계 + 의존성 규칙`
- [ ] `docs/10-charter/PROJECT_CHARTER.md`: `## 2.1 페르소나`, `## 3.1 핵심 시나리오`, `## 9. 핵심 가정` 신설(기존 `## 9. 열린 질문`은 `## 10. 열린 질문`으로 시프트)
- [ ] `docs/40-validation/QA_FINDINGS.md`: 마일스톤 단위 양식 + 마이그레이션 가이드
- [ ] `docs/40-validation/IMPROVEMENT_GUIDE.md`: 마일스톤 단위 그룹핑 안내
- [ ] `docs/90-decisions/_ADR_GUIDE.md`: "새 ADR 추가 절차" 섹션
- [ ] `docs/90-decisions/README.md` 인덱스: ADR-001~009 모두 들어 있음

### A.7 .gitignore

- [ ] `docs/40-validation/reports/*.md`와 `!docs/40-validation/reports/.gitkeep` 추가

### A.8 본 가이드 처분

본 가이드는 적용 후 폐기하는 임시 문서다. 적용 이력은 git에 그대로 남으므로 영구 산출물에 별도 보존 절차를 두지 않는다(부록 C의 "영구 산출물에 IMPROVE-GUIDE 참조 0건" 검증과 일관).

> 마지막 commit으로 `git rm IMPROVE-GUIDE.md`.

---

## 부록 B — 적용 시 자주 빠뜨리는 항목

본 가이드 적용 시 SSOT drift를 막기 위해 다음을 자주 점검한다.

1. **새 ADR 만들면 `docs/90-decisions/README.md` 인덱스 표에 한 줄 추가** — Step 4·6·8에서 각각 ADR-006·007·008·009를 만들 때 잊기 쉽다.
2. **새 산출물 만들면 `docs/00-meta/STRUCTURE.md` 인벤토리에 한 줄 추가** — 본 가이드는 Step 3에서 미리 모든 신규 산출물을 등록해 둔다.
3. **agent/skill 본문에 정책 설명을 길게 박지 않는다** — 정책은 ADR로, agent/skill은 행동 규율 + ADR 링크로.
4. **CLAUDE.md를 다시 비대하게 만들지 않는다** — 진입 페이지는 짧을수록 가치가 크다(약 30~40줄 목표).
5. **README.md / README_ko.md 동시 갱신** — 한쪽만 고치면 cross-language drift 발생.
6. **`git add -A` 사용 금지** — `/finalize-workitem`이 명시적 파일 add를 강제. 가이드 적용 시에도 동일 원칙 권장.

---

## 부록 C — 가이드 적용 종료 기준

다음을 모두 만족하면 본 가이드 적용 완료다.

- [ ] 부록 A의 모든 체크박스 ✅
- [ ] `git status`가 clean (모든 변경이 commit됨)
- [ ] **SSOT 일관성 검증** — 다음 명령으로 dangling 참조와 누락이 없음을 확인. `pathspec`으로 임시 문서를 제외해야 가이드 자체가 거짓 양성을 일으키지 않는다:
  - `git grep -n "claude-opus-4-6\|claude-sonnet-4-6" -- ':!IMPROVE-GUIDE.md'` 결과 없음 (Step 1 모델 별칭. 본 가이드 자체에는 해당 문자열이 설명 목적으로 남아 있을 수 있으므로 가이드 파일을 제외한다.)
  - 다음 7개 ADR 파일이 보일러플레이트에 존재한다: `ADR-001, 004, 005, 006, 007, 008, 009`. **ADR-002·003은 보일러플레이트에 존재하지 않는다 — `/bootstrap-project`·`/bootstrap-stack`이 fork된 새 프로젝트에서 생성하는 placeholder다(0.3 결정).** `docs/90-decisions/README.md` 인덱스에는 9개 행이 모두 등재되되 ADR-002·003 행은 placeholder임을 표시한다.
  - `git grep -nE "ADR-[0-9]{3}" -- 'docs/' '.claude/' 'CLAUDE.md' 'README*.md'`로 ADR 참조가 모두 위 정책에 부합하는지(또는 placeholder임을 명시하는지) 확인. (3자리 번호 매칭이라 ADR-010+ 도입 후에도 유효.)
  - `git grep -n "docs/40-validation/reports"` 결과가 STRUCTURE.md, .gitignore, validate-workitem 본문, repair-workitem 본문 등에서 일관되게 등장 (산출물 인벤토리)
  - `git grep -n "Step [0-9]" -- 'docs/' '.claude/' 'CLAUDE.md' 'README*.md'` 결과 0건이어야 한다 — Step 번호는 가이드에만 한정. ADR이나 skill 본문에 "Step N" 표현이 남아 있으면 수정 필요. (가이드 본문의 코드블록 안에는 산출물의 *내용*만 들어 있고, "Step N" 같은 가이드 메타 코멘트는 모두 코드블록 *밖*의 `> 주:` 형태로 둔다 — 적용 시 코드블록 안의 텍스트만 복사하고 메타 코멘트는 복사하지 않는다.)
  - `git grep -n "IMPROVE-GUIDE" -- 'docs/' '.claude/' 'CLAUDE.md' 'README*.md'` 결과 0건. 본 가이드는 임시 문서이므로 영구 산출물에 참조가 새지 않아야 한다.
- [ ] **Markdown anchor 링크 검증** — `git grep -nE '\[[^]]*#[^]]*\]\([^)]*\.md\)' -- ':!IMPROVE-GUIDE.md'`로 표시 텍스트에 anchor가 있고 URL에는 없는 broken link가 없음을 확인.
- [ ] (선택) 새 프로젝트 한 곳에 fork 시뮬레이션 — 새 디렉터리에 본 보일러플레이트를 복제하고 `/discover-product` 한 라운드를 돌려본다. 흐름이 자연스러우면 가이드 적용이 성공적이다.

---

> 본 가이드는 적용 후 폐기되는 임시 문서다. 적용 이력은 git에 남으므로 별도 보존 절차를 두지 않는다.








