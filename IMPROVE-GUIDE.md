# IMPROVE-GUIDE

> 이 문서를 위에서부터 따라가면 [IMPROVE-LIST.md](IMPROVE-LIST.md) 의 24 항목 적용이 완료된다. 단 *왜 이 분류·강도·evidence label* 인지의 추적성은 IMPROVE-LIST 본문 참조 (Ratchet 강/약, P0/P1/P2, `[관측됨]`/`[가설]`/`[가설→실증]` 라벨).
>
> - 각 Step은 **파일 경로 → 현재 상태 → 변경 액션 → Before/After → 검증** 의 5필드 구조.
> - Before/After 코드 블록은 그대로 복사 가능한 verbatim 형식이다. 단 **hook JSON 예시 (Step 17 / Step 19)** 는 Anthropic hooks docs schema 가 변형 가능성이 있어 *검증 필요 예시* 로 다룬다 — 첫 fork 적용 시 [hooks docs](https://code.claude.com/docs/en/hooks-guide) 본문을 직접 확인 (특히 `asyncRewake` 가 어떤 exit code 에서 깨우는지, `if` 의 brace glob 지원 여부, Windows `command`/`args` 조합) 한 뒤 박는다.
> - **Line citation 은 *원본 파일 기준*** — 같은 파일에 앞선 Step 이 라인 추가 후 다음 Step 의 `line N` 인용은 *원본 기준 위치 식별자* 일 뿐이다. 실제 적용 시 *Before 블록의 content matching* 이 권위. 특히 *같은 파일 다중 변경* 인 `stabilize-milestone/SKILL.md` (Step 21 + Step 22(3) + Step 23) / `stack-guard/SKILL.md` (Step 16 + 17 + 18 + 19) / `review-doc/SKILL.md` (Step 1 + Step 11) 는 content matching 으로 위치 재확인.
> - 커밋 메시지는 그룹별로 1줄 영어로 박혀 있다 (Conventional Commits — ADR-008 정합). 그룹 B 는 의미 단위가 둘이라 **2 커밋 (B1 / B2)** 으로 분할.
> - **순서 의존성**: 다음 의존성을 어기면 후속 Step 이 깨진다.
>   - Step 5 (ADR-006 Amendment) → Step 15 (AGENTS.md SSOT 1줄) 이전에 박는다.
>   - Step 3 (FEATURE_TEMPLATE `## 7-1`) → Step 4 (validate-workitem audit) / Step 21 (stabilize preflight) 이전에 박는다.
>   - Step 20 (stack-guard Codex wrapper) → Step 16~19 후에 박는다 (wrapper 가 본문을 가리키므로).
> - **Step 1 ↔ Step 11** (`review-doc/SKILL.md` 동시 변경) 은 *각 anchor 영역이 분리* 되어 있어 어느 순서로 적용해도 최종 상태가 동일 (Step 1 노트의 *최종 순서* 도식 참조).

## 진행 그룹 요약

| 그룹 | 범위 | Step 번호 | 커밋 메시지 |
|------|------|-----------|-------------|
| A | P0 drift fix (자기 모순 / cross-tool drift 직접 교정) | Step 1 ~ Step 4 | `fix(drift): resolve cross-tool drift and self-contradictions in core skills` |
| B1 | ADR-006 Amendment 1 + Surgical Changes + LOC sanity 확산 (builder / validator / reviewer) | Step 5 ~ Step 11 | `feat(policy): adopt Surgical Changes + LOC sanity heuristic across builder/validator/reviewer (ADR-006 amend1)` |
| B2 | Ambiguity surfacing + Step→verify + AGENTS.md 진입 SSOT | Step 12 ~ Step 15 | `feat(policy): add ambiguity surfacing protocol and AGENTS.md Surgical Changes SSOT` |
| C | stack-guard 핵심 강화 (smoke test / exec form / verify 풀세트 / hook adapter / Codex wrapper) | Step 16 ~ Step 20 | `feat(stack-guard): smoke test, exec-form hooks, stack verify defaults, Codex wrapper` |
| D | stabilize-milestone 강화 + bootstrap-stack 운영 사실 | Step 21 ~ Step 24 | `feat(stabilize-milestone): deterministic preflight, evaluator-optimizer naming; bootstrap-stack ops facts` |

---

# 그룹 A — P0 drift fix

자기 모순 / cross-tool drift / 영속화 누락의 직접 교정. 후속 그룹의 토대.

---

## Step 1. `review-doc` skill의 권한 ↔ 책무 내부 모순 해소 (IMPROVE-LIST #13)

### 대상 파일
- `.claude/skills/review-doc/SKILL.md`

### 현재 상태
- frontmatter `allowed-tools: Read Glob Grep` (line 6) — Write/Edit 권한 없음.
- 본문 line 28~32 5곳에서 *"IMPROVEMENT_GUIDE.md에 보고"* 명령 — 권한 없이 쓰기 호출 → 물리적 실행 불가.

### 변경 액션
(1) `.claude/skills/review-doc/SKILL.md` frontmatter `allowed-tools` 에 `Write Edit` 를 unscoped 로 추가 — *도구 호출 가능성* 을 정한다.
(2) **`.claude/agents/reviewer.md` `tools:` 에도 `Write, Edit` 추가** — review-doc 은 `agent: reviewer` 로 subagent 위임이라 *agent 의 tools 가 실 권한*. skill frontmatter 만 갱신하면 subagent 가 Write 호출 불가 (실 권한 부재). validator/validate-workitem 패턴 (둘 다 Write 보유) 정합.
(3) reviewer.md 본문에 *scoped Write 사용 룰* 1줄 추가 — Write/Edit 는 review-doc 호출 시 IMPROVEMENT_GUIDE.md 단일 파일만 허용. 다른 surface (stabilize-milestone / manual fork) 에서는 본 reviewer 가 report-only.
(4) review-doc/SKILL.md 본문 **`검토 항목:` 목록의 마지막 bullet 뒤 — `마지막 출력:` plain-text label 의 직전 빈 줄 위치** 에 *Write 범위 제한* 단락 1개를 plain-text label 형식으로 추가 — 본문 텍스트가 *수정 대상 파일* 을 제한한다.

> **표현 주의**: frontmatter `allowed-tools` 는 도구 호출 가능성, 본문은 *수정 대상 파일 제한 규율*. **둘은 직교** — 한쪽이 다른 쪽보다 *강하다* / *SSOT 다* 같은 표현은 쓰지 않는다.

> **Step 1 ↔ Step 11 의 최종 순서** (둘 다 같은 파일을 건드림): `검토 항목:` label → 기존 bullet 6개 → Step 11 새 bullet 1개 → 빈 줄 → **Step 1 의 Write 범위 제한 단락** → 빈 줄 → `마지막 출력:` label.
> 두 Step 은 *어느 순서로 적용해도* 동일 최종 상태가 나오게 *content-anchored 위치* 로 박혀 있다 — Step 1 은 "마지막 출력: 직전 빈 줄", Step 11 은 "검토 항목 마지막 bullet". line 번호 인용은 *참고용* 일 뿐 content matching 이 권위.

### Before/After

**(1) `review-doc/SKILL.md` frontmatter — line 6**

Before:
```yaml
allowed-tools: Read Glob Grep
```

After:
```yaml
allowed-tools: Read Glob Grep Write Edit
```

**(2) `reviewer.md` frontmatter `tools:` — line 4**

`reviewer.md` 현재 line 4:

Before:
```markdown
tools: Read, Glob, Grep
```

After:
```markdown
tools: Read, Glob, Grep, Write, Edit
```

**(3) `reviewer.md` 본문에 *scoped Write 사용 룰* 1줄 추가** — 원본 line 34 (`정책 근거: [ADR-006](...)`) *직전* 위치 anchor. 단 *Step 10 (Scope Discipline) + Step 11 (Document Consistency)* 도 동일 영역 (Clean Code 6 와 정책 근거 사이) 에 박힌다.

**reviewer.md 최종 순서** (Step 1(3) + Step 10 + Step 11 모두 적용 후):
1. Clean Code 6항목 체크리스트 (line 24~32, 원본 그대로).
2. 빈 줄.
3. `## Scope Discipline 체크` (Step 10).
4. 빈 줄.
5. `## Document Consistency 체크` + 호출 surface 단락 (Step 11).
6. 빈 줄.
7. **`Write/Edit 사용 범위: ...` 1줄 (Step 1 (3) — 운영 메모, 정책 근거 직전)**.
8. 빈 줄.
9. `정책 근거: [ADR-006](...)` (원본 line 34).

Step 1 (3) 의 anchor 는 *"정책 근거 직전"* 으로 박혀 있어 Step 10/11 의 dimension 섹션 들이 Clean Code 6 *직후* anchor 로 박히면 자연 정합. 적용 순서 (A → B1) 상 Step 1 이 먼저지만 *content-anchored* 라 어느 순서로 적용해도 최종 상태 동일.

After (삽입할 1줄):
```markdown
Write/Edit 사용 범위: `/review-doc` 호출 시 `docs/40-validation/IMPROVEMENT_GUIDE.md` 단일 파일만 허용 (review-doc body 의 *Write 범위 제한* 단락 정합). 다른 surface (`/stabilize-milestone` / manual fork) 호출 시 reviewer 는 *report-only* — 본 agent 가 직접 쓰지 않고 호출 측이 받아 적는다.
```

**(4) `review-doc/SKILL.md` 본문 — `검토 항목:` 목록 (Step 11 의 bullet 포함) 의 마지막 bullet 뒤 + `마지막 출력:` plain-text label 의 직전 빈 줄 위치 에 새 단락 1개 삽입**

원본 `review-doc/SKILL.md` 는 본문에서 H2 가 아닌 plain-text label (`검토 항목:` / `마지막 출력:`) 을 사용한다. 본 단락도 동일 스타일을 따른다 (H2 사용 금지).

After (`마지막 출력:` 직전 빈 줄 자리에 삽입):
```markdown
Write 범위 제한 (수정 대상 파일 제한 — frontmatter `allowed-tools` 와 직교):
- frontmatter `allowed-tools` 의 Write/Edit 는 *도구 호출 가능성* 만 정한다 (그래야 IMPROVEMENT_GUIDE 에 기록 가능).
- 본문은 *수정 대상 파일* 을 `docs/40-validation/IMPROVEMENT_GUIDE.md` **단일 파일** 로 제한한다 — 그 외 어떤 파일도 Write/Edit 금지.
- 본문 외 변경이 필요해 보이면 출력에 "후속 task 권장" 텍스트만 남긴다 — `/plan-workitem` 또는 사용자가 후속 발화.
- 위반 발견 시 IMPROVEMENT_GUIDE.md 에 *self-report* (예: `P1 [Self-violation] review-doc edited <file>`) + 다음 라운드 stabilize 가 회수.
```

### 검증
- `review-doc/SKILL.md` frontmatter `allowed-tools` 에 `Write Edit` 추가 확인.
- `reviewer.md` frontmatter `tools:` 에 `Write, Edit` 추가 확인.
- `reviewer.md` 본문에 *Write/Edit 사용 범위* 1줄 (review-doc 호출 시 IMPROVEMENT_GUIDE.md 단일 파일만 허용) 박혀 있는지 확인.
- review-doc 본문에 *Write 범위 제한* plain-text label 단락이 `마지막 출력:` *직전* 에 있는지 확인 (H2 가 아닌 일반 텍스트 라벨 형식).
- Step 11 의 bullet 이 *Write 범위 제한* 단락 **위** 의 `검토 항목:` 목록 마지막에 있는지 확인 (위 *최종 순서* 도식 정합).

---

## Step 2. `stabilize-milestone` 회고 책임 경계 명시 (IMPROVE-LIST #14)

### 대상 파일
- `.claude/skills/stabilize-milestone/SKILL.md`

### 현재 상태
- line 10~12 책임 경계: *"workitem status 변경 X / 그 외 변경 금지"*.
- line 57: *milestone 문서의 `## 8. 회고` 섹션 자동 채움* — 같은 skill 내부에서 정면 모순.

### 변경 액션
(1) line 10~12 책임 경계 단락을 3개 허용 변경(`QA_FINDINGS`, `IMPROVEMENT_GUIDE`, milestone `## 8. 회고`)을 명시한 형태로 교체.
(2) line 54~57 본문 끝의 "책임 경계" 단락을 도입부 SSOT를 참조하는 짧은 재확인 형태로 압축.

### Before/After

**(1) line 10~12 (책임 경계 도입부)**

Before (현재 정확한 3줄):
```markdown
이 skill은 **코드 수정·커밋·workitem status 변경을 하지 않는다.**
검증 결과를 `docs/40-validation/QA_FINDINGS.md`와 `docs/40-validation/IMPROVEMENT_GUIDE.md`에 누적 기록하는 문서 갱신은 정상 책임이다 — 그 외 변경은 금지한다.
후속 작업이 필요하면 `/repair-workitem` 또는 새 task로 텍스트 제안만 출력한다.
```

After:
```markdown
이 skill은 **코드 수정·커밋·workitem status 변경을 하지 않는다.**
다음 세 종류의 문서 갱신만 정상 책임이다:
1. `docs/40-validation/QA_FINDINGS.md` 누적 기록 (qa 위임 결과).
2. `docs/40-validation/IMPROVEMENT_GUIDE.md` 누적 기록 (reviewer 위임 결과 + deterministic preflight 결과).
3. milestone 문서의 `## 8. 회고` 섹션 자동 채움 ([ADR-014](../../../docs/90-decisions/boilerplate/ADR-014-milestone-graduation.md) graduation contract — status 변경 X, 본문 단락 갱신만).
   - 회고 본문 4 항목: 목표 달성도 / scope creep / 비목표 위반 / 핵심 학습 3개 이내.

그 외 변경은 금지한다 — milestone 문서의 `## 0. Status` / `## 1~7` 섹션 / 다른 workitem 문서 / 코드 일체.
후속 작업이 필요하면 `/repair-workitem` 또는 새 task로 텍스트 제안만 출력한다.
```

**(2) line 54~57 (본문 끝 "책임 경계" 단락)**

Before (현재 정확한 4줄):
```markdown
책임 경계:
- 코드 수정·커밋·workitem status 변경 금지.
- 누적 문서 갱신만 허용 (`QA_FINDINGS.md`, `IMPROVEMENT_GUIDE.md`).
- milestone 문서의 `## 8. 회고` 섹션을 stabilize 종료 시점에 자동 채운다 (ADR-014). 목표 달성도 / scope creep / 비목표 위반 / 핵심 학습 3개 이내.
```

After:
```markdown
책임 경계:
- 코드 수정·커밋·workitem status 변경 금지.
- 누적 문서 갱신 + milestone `## 8. 회고` 자동 채움.
- *상세 SSOT 는 본 skill 도입부 책임 경계 단락* — 본 단락은 단순 재확인.
```

> *unique 정보 통합*: 회고 본문 4 항목 (목표 달성도 / scope creep / 비목표 위반 / 핵심 학습 3개) 은 도입부 책임 경계 단락 3 번 항목 *직하* 의 하위 bullet 으로 통합 (위 (1) After 블록 참조). 본문 끝 단락은 unique 정보를 갖지 않는 진정한 재확인.

### 검증
- 도입부 책임 경계 단락에 *3개 허용 변경*이 번호 목록으로 명시.
- 본문 끝 "책임 경계" 단락이 도입부 단락을 참조하는 짧은 재확인 형태.

---

## Step 3. `FEATURE_TEMPLATE` 에 `## 7-1. FAC ↔ AC 매핑표` subsection 신설 + `plan-workitem` 영속화 (IMPROVE-LIST #15)

### 대상 파일
- `docs/30-workitems/_templates/FEATURE_TEMPLATE.md`
- `.claude/skills/plan-workitem/SKILL.md`

### 현재 상태
- `plan-workitem` 출력 텍스트 (line 66~72) 에만 매핑표가 박힘 — 세션 종료 시 사라짐.
- `FEATURE_TEMPLATE.md` 에 FAC↔AC 매핑 자리 부재.
- → cross-round 추적 불가. validator(Step 4) / stabilize(Step 21) 가 점검할 surface 없음.

### 변경 액션
(1) `FEATURE_TEMPLATE.md` `## 7. Feature-level Acceptance Criteria` 단락 (line 22~25) **직후** 에 `## 7-1. FAC ↔ AC 매핑표` subsection 추가. ADR-036 의 12-섹션 main 구조를 깨지 않기 위해 7-1 *subsection* 으로 박는다.
(2) `plan-workitem/SKILL.md` 본문 39~40 line "feature 분해 시" 단락 갱신.
(3) `plan-workitem/SKILL.md` 본문 66~72 line "마지막 출력" 의 매핑표 부분 갱신.
(4) **`ADR-037-spec-coverage-audit.md` 본문 끝에 `## Amendment 1` 추가** — `## 7-1` 영속 SSOT 위치 정착. ADR-037 이 spec coverage 정책의 canonical 이라 매핑 *저장 위치* 변경을 ADR-037 amendment 로 박는다 (ADR-005 패턴 4 "정책 = ADR" 정합).
(5) **ADR 인덱스 README 의 ADR-037 행 Amendments 칼럼 갱신**.

### Before/After

**(1) `FEATURE_TEMPLATE.md` — `## 7. Feature-level Acceptance Criteria` 단락 (line 22~25) 직후 (line 26 빈 줄 위치) 에 다음 단락 삽입**

After (삽입할 블록):
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

**(2) `plan-workitem/SKILL.md` line 39~40 (`## feature 분해 시 (ADR-036)` 헤딩 + 다음 한 줄)**

Before (현재 정확한 2줄):
```markdown
## feature 분해 시 (ADR-036)
feature 분해 시 12섹션 모두 채운다. `## 7 FAC`는 task `## 6 AC`로 분해되며 매핑 누락 시 plan 출력의 "남은 미결정 사항"에 명시.
```

After:
```markdown
## feature 분해 시 (ADR-036)
feature 분해 시 12 main sections + `## 7-1` mapping subsection 모두 채운다.
`## 7 FAC`는 task `## 6 AC`로 분해되며 매핑 결과는 **feature 문서의 `## 7-1. FAC ↔ AC 매핑표` subsection에 영속 저장** (출력만 X — drift 차단).
매핑 누락(unmapped FAC)은 plan 출력의 "남은 미결정 사항"에 *추가*로 명시.
다음 라운드의 [validate-workitem](../validate-workitem/SKILL.md) Spec coverage audit (ADR-037)
및 [stabilize-milestone deterministic preflight](../stabilize-milestone/SKILL.md)가
본 영속 표를 참조해 cross-round 추적.

**Legacy fallback** — 기존 feature 문서(템플릿 변경 *전*에 생성된 것)는 `## 7-1` subsection이 부재할 수 있다. validator / stabilize preflight는 다음 순서로 회수한다:

1. `## 7-1` subsection 존재 → 본문 매핑 표 직접 점검.
2. 부재 → `## 7 FAC` 본문에서 *inline 매핑 표기*(예: `- FAC-1 → T-001:AC-1`) 휴리스틱 검출.
3. 둘 다 부재 → `Spec Gap: <feature> 매핑표 부재 — legacy 문서 보강 권장` 라벨로 IMPROVEMENT_GUIDE에 P1 기록 + 다음 plan 라운드에서 `## 7-1` 보강.
```

**(3) `plan-workitem/SKILL.md` line 66~72 (마지막 출력의 매핑표 항목)**

Before (현재 정확한 7줄):
```markdown
- `## 8. FAC ↔ AC 매핑표` (feature 분해 시 — plan 출력에 섹션 헤딩으로 박힘, ADR-037):
  ```
  ## 8. FAC ↔ AC 매핑표
  FAC-1 → T-001:AC-1, T-002:AC-2
  FAC-2 → T-003:AC-1
  FAC-3 → unmapped  ← 미커버 task 필요
  ```
```

After:
```markdown
- feature 분해 시: 매핑표를 *feature 문서의 `## 7-1`에 직접 기록* + plan 출력 요약에 동일 표 echo
  (`## 7-1` 본문이 SSOT, plan 출력은 사람 확인용):
  ```
  ## 7-1. FAC ↔ AC 매핑표
  FAC-1 → T-001:AC-1, T-002:AC-2
  FAC-2 → T-003:AC-1
  FAC-3 → unmapped  ← 미커버 task 필요
  ```
```

**(4) `ADR-037-spec-coverage-audit.md` 파일 끝 (`## 참고` 단락 직전 또는 파일 마지막) 에 `## Amendment 1` 단락 추가**

`ADR-037` 현재 line 30~31 (`## 후속 작업` / `없음`) 다음, `## 참고` (line 33) *직전* 에 새 단락 삽입 (또는 파일 끝에 append 후 `## 참고` 단락 보존):

After (삽입할 단락):
```markdown

## Amendment 1 (2026-05-16) — FAC ↔ AC 매핑표 영속 SSOT 위치

### 결정

FAC ↔ AC 매핑은 *plan-workitem 출력 echo* 가 아니라 **feature 문서의 `## 7-1. FAC ↔ AC 매핑표` subsection** 에 영속 저장한다. plan 출력은 사람 확인용 echo.

- `## 7-1` 위치: ADR-036 의 12-섹션 main 구조를 보존하기 위해 `## 7 FAC` 의 *subsection* 으로 박는다 (추가 main section 신설 X).
- 영속 SSOT 가 있어야 다음 라운드의 validate-workitem (본 ADR 결정 1 의 Spec coverage audit) 과 stabilize-milestone deterministic preflight 가 cross-round 추적 가능.
- legacy feature 문서 (template 변경 전 생성) 는 *Legacy fallback* 3-단계로 회수 — (1) `## 7-1` 존재 / (2) `## 7 FAC` 본문 inline 매핑 휴리스틱 / (3) `Spec Gap` P1 라벨.

### 근거

- 기존 본 ADR 결정 2 ("plan-workitem 출력 형식에 매핑표 추가") 는 *출력 텍스트만* 명시 — 세션 종료 시 사라져 cross-round 추적 surface 부재.
- [관측됨] plan-workitem 출력 텍스트만 있고 영속 자리 부재 — feature 문서 본문 / task 본문 어느 곳에도 매핑이 저장되지 않아 다음 round 의 validator / stabilize 가 점검 surface 가 없음.

### 적용 surface

- [FEATURE_TEMPLATE.md](../../../docs/30-workitems/_templates/FEATURE_TEMPLATE.md) `## 7-1` subsection 신설.
- [plan-workitem/SKILL.md](../../../.claude/skills/plan-workitem/SKILL.md) "feature 분해 시" 단락 — 영속 저장 + plan 출력은 echo 정합.
- [validate-workitem/SKILL.md](../../../.claude/skills/validate-workitem/SKILL.md) Spec coverage audit (본 ADR 결정 1 의 surface 확장).
- [stabilize-milestone/SKILL.md](../../../.claude/skills/stabilize-milestone/SKILL.md) deterministic preflight FAC unmapped 점검.

### 후속 작업

- 기존 feature 문서 (template 변경 전 생성) 의 `## 7-1` 보강 — Legacy fallback 3-단계로 운영 차단 없이 회수되므로 fork 별로 일괄 migration / lazy migration / 신규 feature 부터만 적용 중 선택. plan-workitem 호출 자연 발생 시점에 보강 가능.
```

**(5) `docs/90-decisions/boilerplate/README.md` 의 ADR-037 행 (line 33) Amendments 칼럼 갱신**

Before:
```markdown
| 037 | Spec coverage self-audit | accepted | — | FAC→AC 매핑 추적, Spec Gap report, 자동 차단 X |
```

After:
```markdown
| 037 | Spec coverage self-audit | accepted | (+amend1: FAC↔AC 매핑표 영속 SSOT 위치 `## 7-1`) | FAC→AC 매핑 추적, Spec Gap report, 자동 차단 X |
```

### Step 3 보조: 기존 feature 문서 1회 migration (본 가이드 적용 직후)

> 본 절차는 IMPROVE-GUIDE 의 일회성 적용 단계이며 plan-workitem/SKILL.md 에는 박지 않는다 — plan-workitem 본문은 Legacy fallback 만 가진다.

본 Step 적용 후 *기존 feature 문서들* 에 `## 7-1` subsection 이 자동으로 박히지 않는다. fork repo 의 정책 선택:

- **선택지 A — 즉시 일괄 migration** (현재 feature 수가 5개 이하 권장):
  1. `docs/30-workitems/features/F-*-*.md` 글로브로 기존 feature 목록 회수.
  2. 각 feature 문서 `## 7. Feature-level Acceptance Criteria` 단락 직후에 `## 7-1` subsection 을 *수동* 으로 박고, 기존 `## 7 FAC` 와 task `## 6 AC` 의 매핑을 채운다.
  3. 단발 보강 task 1개로 `/plan-workitem` 호출 권장 — task body 에 *"기존 feature N개의 `## 7-1` 매핑표 보강"* 명시.
- **선택지 B — Lazy migration** (feature 가 많거나 안정적이면):
  - 기존 feature 는 그대로 두고 *다음 plan-workitem 호출 시* (feature 갱신 자연 발생) `## 7-1` 보강.
  - 그 전까지는 plan-workitem/SKILL.md 의 *Legacy fallback* (위 (2) inline 휴리스틱 / (3) P1 라벨) 으로 운영 차단 없이 회수.
- **선택지 C — 템플릿만 적용 (no migration)**:
  - 신규 feature 부터만 `## 7-1` 채움. 기존 feature 는 영영 inline 휴리스틱 또는 P1 라벨에 의존.
  - fork 사용자가 운영 부담 최소를 원할 때.

선택은 fork 사용자 결정. *어떤 선택을 했는지* 본 가이드 적용 PR 본문에 한 줄 기록 권장.

### 검증
- `FEATURE_TEMPLATE.md` 에 `## 7-1. FAC ↔ AC 매핑표 (subsection of ## 7)` 헤딩이 `## 7` 직후에 있다.
- `plan-workitem/SKILL.md` 의 두 단락이 갱신됐다.
- `ADR-037-spec-coverage-audit.md` 본문에 `## Amendment 1 (2026-05-16) — FAC ↔ AC 매핑표 영속 SSOT 위치` 단락 박혀 있다.
- ADR 인덱스 README 의 ADR-037 행 Amendments 가 `(+amend1: FAC↔AC 매핑표 영속 SSOT 위치 ## 7-1)`.

---

## Step 4. `validate-workitem` 에 FAC → AC spec coverage audit 추가 (IMPROVE-LIST #12)

### 대상 파일
- `.claude/skills/validate-workitem/SKILL.md`

### 현재 상태
- 검증 기준 7항목 (line 26~33) 에 FAC → AC audit 누락.
- 동일 책무가 `validator.md` line 45 에만 박혀 있어 Codex wrapper(`$validate-workitem`) 호출 시 skill body 만 따르는 흐름에서 누락.

### 변경 액션
(1) 검증 기준 (line 26~33) 의 마지막에 새 항목 1개 추가.
(2) report 양식 (line 38~61) 의 `## AC ↔ 테스트 매핑` 섹션 *직후* 에 `## Spec coverage` 섹션 추가.

### Before/After

**(1) 검증 기준 추가 — 현재 line 33 (`- 테스트 선행 휴리스틱 — ...`) 뒤에 1줄 추가**

현재 line 26~33 형태:
```markdown
검증 기준:
- 문서 범위와 구현이 일치하는가
- 범위 밖 변경이 있는가
- 빠진 검증 포인트가 있는가
- obvious regression risk가 있는가
- 통합 검증 명령(있으면) 결과는 통과인가
- AC ↔ 테스트 매핑 — task 문서의 AC-N마다 대응하는 테스트가 존재하는가(자연어 매칭 휴리스틱 또는 테스트 이름의 `AC_N` 식별자 매칭).
- 테스트 선행 휴리스틱 — git log에서 동일 task 범위의 테스트 파일 추가/수정이 구현 파일보다 먼저(또는 동일 커밋) 들어왔는지. 단순 경고로만 보고하고 강제 종료하지 않는다(소규모 작업이 한 커밋에 묶이는 경우 정상).
```

After — 마지막 줄 *뒤* 에 다음 1항목 추가:
```markdown
- FAC → AC spec coverage audit ([ADR-037](../../../docs/90-decisions/boilerplate/ADR-037-spec-coverage-audit.md)):
  feature `## 7 FAC`의 각 항목이 본 task의 `## 6 AC` 또는 *연관 task의 AC*에
  매핑되는가? 매핑 안 된 FAC가 있으면 report의 "Spec coverage" 섹션에
  `Spec Gap: FAC-N → unmapped` 명시 + 미커버 task 추가 권장.
  자동 차단 X — ADR-007 책임 경계 정합. legacy fallback은 plan-workitem SKILL.md "feature 분해 시" 단락 참조.
```

**(2) report 양식 — `## AC ↔ 테스트 매핑` 섹션 (line 53~56) 직후 에 새 섹션 1개 삽입**

현재 line 53~56:
```markdown
## AC ↔ 테스트 매핑
- AC-1: ✅ tests/foo.spec.ts > test_AC_1_xxx
- AC-2: ❌ (테스트 없음)
- AC-3: ✅ tests/bar.spec.ts > test_AC_3_xxx
```

After — 위 4줄 *뒤* 에 다음 섹션 1개 삽입 (현재 line 57의 빈 줄 자리에 박는다):
```markdown

## Spec coverage (FAC ↔ AC, ADR-037)
- FAC-1: ✅ T-001:AC-1
- FAC-2: ✅ T-001:AC-2
- FAC-3: ❌ unmapped — 미커버 task 추가 권장 (예: T-XXX [Given]...[When]...[Then]...)
```

### 검증
- 검증 기준 항목이 8개로 늘었고 마지막 항목이 `FAC → AC spec coverage audit` 다.
- report 양식의 `## Spec coverage` 섹션이 `## AC ↔ 테스트 매핑` 와 `## 다음 권장 액션` 사이에 박혀 있다.

---

## ✅ 그룹 A 완료 — 커밋

```
fix(drift): resolve cross-tool drift and self-contradictions in core skills
```

---

# 그룹 B — ADR-006 Amendment + Surgical Changes / Ambiguity surfacing 확산

정책 SSOT 1곳 (ADR-006 Amendment 1) 박은 뒤 builder / validator / reviewer / implement / plan / AGENTS.md 의 6개 surface 에 propagate.

그룹 B 는 의미 단위가 둘이라 **B1 (Surgical Changes 확산: Step 5~11) + B2 (Ambiguity surfacing + AGENTS.md: Step 12~15)** 두 커밋으로 분할한다. 핵심 의존성:
- **Step 5 (ADR-006 amend) → Step 15 (AGENTS.md 1줄)** 보다 *먼저* (Step 5 는 B1, Step 15 는 B2 — 자연 만족).
- B1 의 모든 Step (5~11) 을 완료한 뒤 **B1 커밋** 박고, 그 다음 B2 의 Step 12~15 진행 후 **B2 커밋**.

---

## Step 5. ADR-006 Amendment 1 신설 + 인덱스 갱신 (IMPROVE-LIST #11)

### 대상 파일
- `docs/90-decisions/boilerplate/ADR-006-simplicity-and-architecture.md`
- `docs/90-decisions/boilerplate/README.md`

### 현재 상태
- ADR-006 본문에 Amendment 섹션 없음 (line 1~50).
- ADR 인덱스 README 의 ADR-006 행: `| 006 | Simplicity, Clean Code, and Clean Architecture priority | accepted | — | 단순성 1순위, ... |` — Amendments 칼럼 `—`.

### 변경 액션
(1) ADR-006 본문 끝 (line 50 이후) 에 `## Amendment 1` 단락 추가.
(2) ADR 인덱스 README 의 ADR-006 행 Amendments 칼럼을 `(+amend1: Surgical Changes + ambiguity surfacing)` 로 갱신.

### Before/After

**(1) `ADR-006-simplicity-and-architecture.md` 끝 (`## 후속 작업` 단락 line 48~50 뒤) 에 다음 단락 추가**

After (파일 끝에 append):
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
- 자동 차단 X — 사용자가 출력 보고 차단/수정 결정 ([ADR-007](ADR-007-workitem-lifecycle.md) 정합).

### 강도 분류 (ADR-022 정합)
- **제약(강)** — pre-existing dead code 의도 외 *삭제*는 validator의 Pass 차단 트리거. 근거 라벨: `[관측됨+외부실증]` (builder.md 내 dead code 문구 drift 관측 + Karpathy testimony).
- **enabling(약)** — 나머지 sub-원칙(diff trace 라벨링 / 인접 포맷팅 / 스타일 무시 / ambiguity surfacing 권장 텍스트)은 *report only* / *권장 텍스트 출력*. 근거 라벨: `[가설]`. 향후 fork 프로젝트에서 [관측됨] 회수 시 [가설→실증] 승격 검토.

### 적용 surface
- [AGENTS.md](../../../AGENTS.md) 단순성 단락 1줄 (Surgical Changes 진입 SSOT).
- [.claude/agents/builder.md](../../../.claude/agents/builder.md) self-check 3 항목 (dead code wording 교정 + diff trace + LOC sanity heuristic — Simplicity First sub-원칙).
- [.claude/agents/reviewer.md](../../../.claude/agents/reviewer.md) *별도 Scope Discipline 섹션* (Clean Code 6은 그대로 유지).
- [.claude/skills/validate-workitem/SKILL.md](../../../.claude/skills/validate-workitem/SKILL.md) diff trace audit + report 섹션 (카테고리 (c) Needs Fix 트리거).
- [.claude/skills/implement-workitem/SKILL.md](../../../.claude/skills/implement-workitem/SKILL.md) Red phase plan + ambiguity gate.
- [.claude/skills/plan-workitem/SKILL.md](../../../.claude/skills/plan-workitem/SKILL.md) AC interpretation diversity self-check.

> Document Consistency 라벨링 (reviewer.md 의 Doc Consistency 섹션) 은 *문서 review surface 의 별도 차원* — 본 amendment 의 Surgical Changes / Ambiguity surfacing 범위 외다. reviewer.md 본문 자체가 Doc Consistency 라벨링의 SSOT 다.

본 amendment의 적용은 단일 task로 박지 않고 surface별로 분할 — 각 적용 task가 *amendment 본문의 적용 surface 완성*임을 amendment 자체에 명시.

### 근거
- Karpathy 직접 testimony (2026-01-26 X post): "silent assumptions", "collateral damage" 실패 모드 명명.
- Chang의 4-원칙 정리본의 커뮤니티 widespread adoption (단 star 수는 *adoption signal*이지 *outcome 검증 evidence*는 아님 — [ADR-022](ADR-022-ratchet-principle.md) 정합).
- 내부 [관측됨]: [.claude/agents/builder.md](../../../.claude/agents/builder.md) self-check 4번 "dead code 정리" 문구와 핵심 행동 규율 "범위 밖 변경 금지" 사이의 방향 모순.

### 후속 작업
- 본 Amendment 적용 후 builder self-check 비용 측정. 항목 추가가 builder 출력 토큰을 크게 늘리면 축약 검토.
- fork 프로젝트에서 [관측됨] 데이터 회수 후 enabling 부분을 [가설→실증]로 승격 검토.
```

**(2) `docs/90-decisions/boilerplate/README.md` line 13 (ADR-006 행)**

Before (현재 정확한 1줄):
```markdown
| 006 | Simplicity, Clean Code, and Clean Architecture priority | accepted | — | 단순성 1순위, Clean Code 2순위, Clean Architecture 3순위 (정당화 시) |
```

After:
```markdown
| 006 | Simplicity, Clean Code, and Clean Architecture priority | accepted | (+amend1: Surgical Changes + ambiguity surfacing) | 단순성 1순위, Clean Code 2순위, Clean Architecture 3순위 (정당화 시) |
```

### 검증
- ADR-006 본문에 `## Amendment 1 (2026-05-16) — Surgical Changes 명시 + ambiguity surfacing protocol` 헤딩이 있다.
- ADR 인덱스 README 의 ADR-006 행 Amendments 칼럼이 `(+amend1: ...)` 로 갱신.

---

## Step 6. `builder.md` dead code self-check 문구 방향 교정 (IMPROVE-LIST #1)

### 대상 파일
- `.claude/agents/builder.md`

### 현재 상태
- line 40 단순성 self-check 4번째 항목:
  `- 삭제 가능한 dead code(쓰이지 않는 import·변수·branch)가 남았는가?`

### 변경 액션
line 40 한 줄을 *이번 변경이 만든 orphan만 정리* 로 교체.

### Before/After

**`builder.md` line 40**

Before:
```markdown
- 삭제 가능한 dead code(쓰이지 않는 import·변수·branch)가 남았는가?
```

After:
```markdown
- 이번 변경이 만든 orphan(쓰이지 않게 된 import·변수·branch)만 정리했는가?
  pre-existing dead code는 출력에 *언급*만 하고 *삭제하지 않았는가*?
```

### 검증
- line 40 부근에 *pre-existing dead code 삭제 금지* 의미의 문구가 박혀 있다.

---

## Step 7. `builder.md` diff traceability self-check 추가 (IMPROVE-LIST #2)

### 대상 파일
- `.claude/agents/builder.md`

### 현재 상태
- self-check 6항목 (line 36~43). Surgical Changes 4 sub-failure (인접 개선 / 무관 리팩토링 / 스타일 무시 / trace 가능성) 가 항목화되지 않음.

### 변경 액션
self-check 마지막 항목 (line 42 `- 이번 task의 인터페이스 요소...`) **직후** 에 새 항목 1개 추가 (들여쓰기 동일).

### Before/After

**`builder.md` line 42 다음 (line 43 빈 줄 위치) 에 다음 항목 삽입**

After (삽입할 항목):
```markdown
- 이번 변경의 모든 줄이 task의 AC 또는 명시 요청으로 거꾸로 추적 가능한가?
  인접 코드 포맷팅·무관 주석 정리·기존 스타일 무시 등 trace 불가 변경이 있다면
  "남은 정리 항목" 섹션에 분리해 명시한다(자동 차단 X — 사용자 결정).
```

### 검증
- self-check 항목이 7개로 늘었고, *trace 가능성* 항목이 인터페이스 SSOT 항목 직후에 박혀 있다.

---

## Step 8. `builder.md` LOC sanity heuristic 추가 (IMPROVE-LIST #8)

### 대상 파일
- `.claude/agents/builder.md`

### 현재 상태
- self-check 7항목 (Step 7 완료 후). LOC 규모 sanity heuristic 부재.

### 변경 액션
Step 7 에서 추가한 diff trace 항목 직후, self-check 끝에 LOC heuristic 1 항목 추가 (자동 차단 X, 권장 텍스트만).

### Before/After

**`builder.md` Step 7 직후 — self-check 마지막에 다음 항목 추가**

After (삽입할 항목):
```markdown
- 이번 task의 총 변경 LOC가 task 범위에 비해 큰 편인가?
  체감 200줄 이상 + 단순화 여지 있으면 "단순화 후보" 1~3개를
  *권장 텍스트*로 출력(자동 차단 X, 사용자 결정).
  initial scaffolding·auth 등 자연스럽게 큰 task는 면제.
  *수치는 hard cap이 아니라 휴리스틱*임을 명시.
```

### 검증
- self-check 항목이 8개. 8번째가 LOC heuristic.

---

## Step 9. `validate-workitem` 에 diff trace audit + report 섹션 추가 (IMPROVE-LIST #3)

### 대상 파일
- `.claude/skills/validate-workitem/SKILL.md`

### 현재 상태
- 검증 기준 (Step 4 완료 후 8항목). *"범위 밖 변경이 있는가"* (line 28) 가 추상 명령.
- report 양식 에 diff trace audit 결과를 박는 표준 자리 없음.

### 변경 액션
(1) 검증 기준 line 28 (`- 범위 밖 변경이 있는가`) 을 4 카테고리 (a)(b)(c)(d) 로 확장.
(2) report 양식 의 `## AC ↔ 테스트 매핑` 섹션 *바로 앞* 에 `## Diff trace audit` 섹션 추가.

### Before/After

**(1) 검증 기준 — line 28 (현재 `- 범위 밖 변경이 있는가`) 한 줄을 다음 블록으로 교체**

Before:
```markdown
- 범위 밖 변경이 있는가
```

After:
```markdown
- 범위 밖 변경 + diff trace audit (ADR-006 amend1):
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

**(2) report 양식 — `## AC ↔ 테스트 매핑` 섹션 *직전* (현재 line 52 빈 줄) 에 새 섹션 1개 삽입**

After (삽입할 블록):
```markdown
## Diff trace audit (ADR-006 amend1)
- 추적 가능 변경 줄: N개 (AC-1: M개, AC-2: ...)
- 추적 불가 변경 줄: K개
  - (a) 인접 포맷팅/주석: <file:line> ... [P1]
  - (b) 무관 리팩토링: ... [P1]
  - (c) pre-existing dead code 삭제: ... [P0 — Needs Fix 트리거]
  - (d) 스타일 변경: ... [P2]
- 판정 영향: <Pass 유지 / Needs Fix 트리거 (오직 (c) 의도 외 발견 시)>

```

### 검증
- 검증 기준 *"범위 밖 변경"* 항목이 4 카테고리 (a)(b)(c)(d) 로 확장됐다.
- report 양식에 `## Diff trace audit` 섹션이 `## AC ↔ 테스트 매핑` 직전에 박혀 있다.

---

## Step 10. `reviewer.md` 에 Scope Discipline 별도 섹션 신설 (IMPROVE-LIST #4)

### 대상 파일
- `.claude/agents/reviewer.md`

### 현재 상태
- Clean Code 6항목 체크리스트 (line 24~30) 만 있음.
- 변경 diff scope 위반의 라벨 자리가 없어 SSOT drift 위험.

### 변경 액션
Clean Code 6항목 단락 (line 24~32) **그대로 유지**, 그 *아래* (Clean Code 6 마지막 줄 line 32 와 정책 근거 line 34 사이 의 빈 줄 line 33 위치) 에 별도 섹션 신설.

### Before/After

**`reviewer.md` 현재 line 32 (`P0/P1/P2 분류와 함께 위 6항목 ...`) 와 line 34 (`정책 근거: ...`) 사이 의 빈 줄 자리 (line 33) 에 다음 단락 삽입**

After (삽입할 블록):
```markdown

## Scope Discipline 체크 (별도 차원 — Clean Code와 독립, ADR-006 amend1)

변경 줄이 task의 AC 또는 명시 요청으로 거꾸로 추적 가능한가.
다음 4 카테고리의 *범위 정합 위반*을 발견 시 라벨링.

- (a) 인접 코드 포맷팅/주석 정리 — `[Scope-format]`
- (b) 무관 리팩토링 — `[Scope-refactor]`
- (c) pre-existing dead code 삭제 — `[Scope-purge]` (P0 권장)
- (d) 기존 스타일 무시·변경 — `[Scope-style]`

reviewer 출력 라벨링 예: `P0 [Scope-purge] auth.ts:120 — 무관 dead function 삭제`.
```

### 검증
- `reviewer.md` 에 Clean Code 6항목과 별개로 `## Scope Discipline 체크` 단락이 존재.

---

## Step 11. `reviewer.md` 에 Document Consistency 별도 섹션 신설 (IMPROVE-LIST #24)

### 대상 파일
- `.claude/agents/reviewer.md`

### 현재 상태
- Step 10 후 Scope Discipline 단락이 추가된 상태. 문서 review 시 적용할 라벨 셋이 부재.

### 변경 액션
Step 10 에서 추가한 `## Scope Discipline 체크` 단락 *바로 뒤* 에 별도 단락 신설 (3 차원 분리: Clean Code 6 / Scope Discipline 4 / Doc Consistency 4).

### Before/After

**`reviewer.md` — Step 10 의 Scope Discipline 단락 마지막 줄 (`reviewer 출력 라벨링 예: ...`) 직후 에 다음 블록 삽입**

After (삽입할 블록):
```markdown

## Document Consistency 체크 (별도 차원 — 문서 review 시 호출)

review-doc 또는 stabilize-milestone deterministic preflight 가
reviewer를 호출하면 다음 4 카테고리의 *문서 일관성 위반*을 발견 시 라벨링.

- (e) 모드 라벨(`> 모드: ...`)과 본문 정합 불일치 ([ADR-012](../../docs/90-decisions/boilerplate/ADR-012-doc-architecture-cleanup.md) Diátaxis) — `[Doc-mode]`
- (f) cross-reference link 유효성 (특히 `[ADR-NNN]` 참조와 실제 파일 매칭) — `[Doc-link]`
- (g) 인용된 ADR 본문과 *현재 ADR 본문* 정합 (citation drift — ADR이 amend된 후 인용자 미갱신) — `[Doc-adr-drift]`
- (h) Terminology consistency — 같은 개념이 다른 용어로 부르는 경우 (예: "Acceptance Criteria" vs "완료 조건") — `[Doc-term]`

reviewer 출력 라벨링 예: `P1 [Doc-link] AGENTS.md:38 — broken ADR link to ADR-XX`.

**호출 surface 명시**: 본 agent가 호출될 때 입력에 *"review surface: code | doc | mixed"*를 명시받는다. surface에 따라 적용 차원:
- `code`: Clean Code 6 + Scope Discipline 4.
- `doc`: Doc Consistency 4 + (해당 시) Scope Discipline 4 (변경 diff가 있을 때만).
- `mixed`: 3 차원 모두.
```

또한, **`stabilize-milestone/SKILL.md` line 30 (reviewer 위임 단계)** 의 reviewer 위임 호출 텍스트에 `review surface: code` 명시를 추가한다.

`.claude/skills/stabilize-milestone/SKILL.md` line 30:

Before:
```markdown
5. **reviewer agent에 리팩토링 후보·아키텍처 부채 점검 위임** — reviewer 입력에 Clean Code 6항목 체크리스트(ADR-006)를 명시 전달한다. reviewer도 보고만 한다. 반환된 보고를 본 skill이 받아 `docs/40-validation/IMPROVEMENT_GUIDE.md`에 정리.
```

After:
```markdown
5. **reviewer agent에 리팩토링 후보·아키텍처 부채 점검 위임** — reviewer 입력에 Clean Code 6항목 체크리스트(ADR-006) + `review surface: code` 를 명시 전달한다. reviewer도 보고만 한다. 반환된 보고를 본 skill이 받아 `docs/40-validation/IMPROVEMENT_GUIDE.md`에 정리.
```

**`review-doc/SKILL.md` 의 reviewer 위임 시점** — `agent: reviewer` frontmatter 로 reviewer 를 fork 하므로 *검토 항목 목록의 마지막 bullet* 으로 추가한다 (content-anchored — line 번호 인용 X, "`docs/00-meta/ 파일 수가 ADR-012의 *6개* 원칙과 일치하는지 ...`" bullet 뒤가 anchor).

After (`검토 항목:` 목록의 마지막 bullet 으로 추가):
```markdown
- reviewer 위임 시 입력에 `review surface: doc` 를 명시한다 (reviewer.md 의 Document Consistency 차원 정합 — Clean Code / Scope Discipline / Document Consistency 3 차원 중 doc surface 선택).
```

> **Step 1 과의 적용 순서 무관성**: Step 1 의 *Write 범위 제한* 단락은 *`마지막 출력:` plain-text label 의 직전 빈 줄* 에 박힌다 — 본 Step 의 bullet 은 *`검토 항목:` 목록 내부*. 두 anchor 가 서로 다른 영역이라 *어느 Step 을 먼저 적용해도* 최종 상태 동일 (Step 1 의 *최종 순서* 도식 참조).

### 검증
- `reviewer.md` 에 `## Document Consistency 체크` 단락이 Scope Discipline 단락 직후에 있다.
- `stabilize-milestone/SKILL.md` line 30 의 reviewer 위임 텍스트에 `review surface: code` 가 박혀 있다.
- `review-doc/SKILL.md` 에 `review surface: doc` 명시 1줄이 박혀 있다.

---

---

> **B1 커밋 시점** — Step 5~11 까지 Surgical Changes 확산 완료. 여기서 **B1 커밋 박은 뒤** B2 (Step 12~15) 로 진행.
>
> B1 커밋 메시지:
> ```
> feat(policy): adopt Surgical Changes across builder/validator/reviewer (ADR-006 amend1)
> ```

---

## Step 12. `implement-workitem` Red phase 직전 ambiguity surfacing protocol 추가 (IMPROVE-LIST #5)

> **B2 시작** — implement-workitem ambiguity / Step→verify + plan-workitem AC interpretation diversity + AGENTS.md 진입 SSOT 4 Step.

### 대상 파일
- `.claude/skills/implement-workitem/SKILL.md`

### 현재 상태
- line 31: Red phase 직전 plan 단락 *"1~3문장으로 명시한다(plan 모드 의존 없이 think-before-edit 규율 확보)."* 만 존재.

### 변경 액션
line 31 의 단락 *바로 뒤* (line 32 빈 줄 위치) 에 ambiguity surfacing 단락 1개 추가.

### Before/After

**`implement-workitem/SKILL.md` — line 31 직후 (line 32 빈 줄 위치) 에 다음 블록 삽입**

After (삽입할 블록):
```markdown

AC가 2+ 해석이 가능하다고 판단되면 해석안을 1~3개 나열하고
*자기가 선택한 해석*을 표시한다(예: "해석 A를 따른다 — 이유: ...").
자동 차단 X — 사용자가 출력 보고 차단/수정 결정 (ADR-006 amend1 ambiguity surfacing).
```

### 검증
- `implement-workitem/SKILL.md` 에 Red phase 직전 plan 단락 *직후* 에 ambiguity surfacing 문장이 박혀 있다.

---

## Step 13. `implement-workitem` Red phase plan 을 `Step → verify` 형식 권장 (IMPROVE-LIST #6)

### 대상 파일
- `.claude/skills/implement-workitem/SKILL.md`

### 현재 상태
- line 31 의 Red phase plan 단락이 *"1~3문장 자유 텍스트"*. Step → verify 형식 명시 없음.

### 변경 액션
line 31 의 plan 단락 텍스트를 자유 텍스트 + Step → verify 형식 권장으로 확장.

### Before/After

**`implement-workitem/SKILL.md` line 31 (현재 1줄)**

Before:
```markdown
Red phase 진입 직전, 출력의 첫 단락으로 "이 task에서 어떤 테스트를 어떤 순서로 작성할 것인가"를 1~3문장으로 명시한다(plan 모드 의존 없이 think-before-edit 규율 확보).
```

After:
```markdown
Red phase 진입 직전, 출력의 첫 단락으로 plan 을 다음 형식으로 명시할 것을 *권장* 한다 (plan 모드 의존 없이 think-before-edit 규율 확보):

  1. <Step> → verify: <어떤 테스트/조건으로 확인>
  2. <Step> → verify: <...>
  3. <Step> → verify: <...>

자유 텍스트 1~3 문장도 허용 — Step → verify 형식은 *권장이지 강제 X*. RGR 사이클이 이미 verify 를 강제하므로 형식 자체는 보조 규율.
*AC-N과 Step의 대응*은 plan 단계에서 명시.
```

> 주의: Step 12 에서 추가한 *ambiguity surfacing 블록* 은 *이 plan 단락 직후* 에 박힌다. 본 Step 적용 후에도 ambiguity 블록은 plan 단락 뒤에 그대로 유지돼야 한다.

### 검증
- Red phase plan 단락이 Step → verify 형식 권장 + 자유 텍스트 fallback 텍스트로 갱신.
- ambiguity surfacing 단락이 그 *뒤* 에 그대로 유지.

---

## Step 14. `plan-workitem` 에 AC interpretation diversity self-check 추가 (IMPROVE-LIST #23)

### 대상 파일
- `.claude/skills/plan-workitem/SKILL.md`

### 현재 상태
- line 36 "9. AC 형식 권장 + 금지 verb 점검" — *형식* 만 점검. plan 시점의 *해석 다양성* surface 없음.

### 변경 액션
line 36 (9번 항목) 의 *바로 뒤* (line 37 위치) 에 9-1 단락 1개 신설.

### Before/After

**`plan-workitem/SKILL.md` — 현재 line 36 ("9. AC 형식 권장 + 금지 verb 점검 — ...") 뒤 (현재 line 37: `10. **task 의존성 채움** ...`) *사이* 에 다음 블록 삽입**

After (삽입할 블록):
```markdown
9-1. **AC interpretation diversity self-check** (분해 직후 1회 실행, ADR-006 amend1):

각 AC를 *2+ 합리적 해석이 가능한지* self-check.
가능 시 plan 출력의 "남은 미결정 사항" 섹션에 다음 형식으로 박음:

- AC-N (T-NNN): 해석 A=<...>, 해석 B=<...>, 권장 선택=<...>
  (이유: charter ## 7. 제약 조건 또는 ## 5. 비목표 정합 / 비용 정합 등)

자동 차단 X — 사용자가 plan 검토 시 *해석 결정 협상*.

본 self-check가 plan 단계에서 발화하면 [implement-workitem ambiguity surfacing](../implement-workitem/SKILL.md)은
*재확인 surface*가 됨 — 2-layer defense (plan에서 잡으면 RGR 1회 절감).
```

### 검증
- `plan-workitem/SKILL.md` 에 9번 직후 9-1 단락이 신설됐고 *10. task 의존성 채움* 가 그 다음에 온다.

---

## Step 15. `AGENTS.md` 단순성 단락에 Surgical Changes 1줄 SSOT 추가 (IMPROVE-LIST #7)

### 대상 파일
- `AGENTS.md`

### 현재 상태
- 단순성·YAGNI 단락 (line 16~21) 끝에 backwards-compat hack 줄 (line 21) 까지 5줄.
- 진입 SSOT 1줄 부재.

### 변경 액션
**선행 조건**: Step 5 (ADR-006 Amendment 1) 이 *반드시* 먼저 박혀 있어야 한다.

단순성·YAGNI 단락 마지막 줄 (line 21) **바로 뒤** 에 1줄 추가.

### Before/After

**`AGENTS.md` line 21 (`- backwards-compat shim, feature flag, ...`) 뒤 에 다음 1항목 추가**

After (line 21 직후 line 22 위치):
```markdown
- 변경한 모든 줄은 task의 AC 또는 명시 요청으로 거꾸로 추적 가능해야 한다.
  인접 코드 개선·무관 포맷팅·기존 스타일 무시·pre-existing dead code 삭제는 금지 (ADR-006 amend1).
```

### 검증
- `AGENTS.md` 줄 수 47줄 (46 → 48) 부근 — 100줄 hard cap 여유 충분 (ADR-011).
- 단순성·YAGNI 단락 마지막에 *trace 가능성 + 4가지 금지* 1항목이 박혀 있다.

---

## ✅ B2 완료 — 커밋

```
feat(policy): add ambiguity surfacing protocol and AGENTS.md Surgical Changes SSOT
```

> 그룹 B 전체 = B1 + B2 두 커밋으로 분할됨 (위 *B1 커밋 시점* 분기 참조).

---

# 그룹 C — stack-guard 핵심 강화

`/stack-guard` 는 장기 운영의 *검증 계약 SSOT*. SIMULATION_RUN 에서 직접 관측된 마찰점 해소 + Codex parity + verify 풀세트 default + hook adapter 옵션.

> ⚠️ **그룹 C 진입 전 필독 — Anthropic hooks API schema 검증 의무**
>
> Step 17 / Step 19 의 hook JSON 예시는 본 가이드 *1 차 해석* 이다. 본 가이드는 다음 3 가지를 *공식 docs 본문에서 직접 확인* 한 후 박을 것을 강제한다:
>
> 1. `if` 필드의 *단일 permission rule* 제약 (`|`/`&&`/list 미지원) — 본 가이드는 이 제약을 가정해 `if` 를 *미사용* 하고 `matcher` + verify 스크립트 내부 필터로 처리.
> 2. `asyncRewake: true` 의 *깨우는 exit code* — 본 가이드는 exit code 2 로 가정 (Anthropic 공식 hooks docs 인용 — fork 적용 시 재확인 권장).
> 3. Windows 의 `command: "powershell" + args: ["-File", ...]` 조합 — `.cmd` shim 회피 패턴.
>
> 첫 fork 적용 시 [hooks docs](https://code.claude.com/docs/en/hooks) 본문 직접 확인. schema variant 발견 시 본 단락을 수정 PR 로 제출 — 본 가이드는 자기 자신을 SSOT 로 박지 않는다.

---

## Step 16. `/stack-guard` 생성 후 `validate` 1회 smoke test 필수 단계 추가 (IMPROVE-LIST #16)

### 대상 파일
- `.claude/skills/stack-guard/SKILL.md`

### 현재 상태
- 수행 단계 (line 37~47) 1~4 단계 — `validate` 진입점 / verify 스크립트 / STACK_SETUP_PLAN.md / `.gitattributes` 생성만. *생성 직후 1회 실행* 단계 없음.
- `.boilerplate/validation/SIMULATION_RUN.md` Round 1 에서 *생성 명령이 실 환경 실패* 직접 관측.

### 변경 액션
(1) 수행 단계 끝 (현재 4번 line 47 뒤) 에 새 단계 5 추가.
(2) "마지막 출력" 단락 (line 49~54) 에 *validate smoke test 결과* 1줄 추가.

### Before/After

**(1) `stack-guard/SKILL.md` — 현재 line 47 (`4. \`.gitattributes\`가 없으면 생성, 있으면 line ending 규칙 추가.`) 뒤 (line 48 빈 줄 자리) 에 다음 블록 삽입**

After (삽입할 블록):
```markdown
5. **Smoke test (필수)**: 생성된 `validate` 명령을 1회 실행한다 (`allowed-tools` 의 Bash 권한 활용 — 신규 권한 추가 불필요).
   본 smoke test 는 *wiring 검증* 이 목적 (명령이 올바르게 연결됐는지) — *프로젝트 자체의 lint/test 통과 여부* 와 분리해 보고한다.

   판정 표:
   - **wiring 성공 + 프로젝트 PASS** → `validate smoke test: PASS (wiring OK, project clean)`.
   - **wiring 성공 + 프로젝트 빈 케이스** (비어있는 lint 룰 / 테스트 0건) → `validate smoke test: PASS (wiring OK, empty rules/tests warning)`.
   - **wiring 성공 + 프로젝트 lint/test 실 위반** → `validate smoke test: WIRING OK, PROJECT FAIL` + stderr 요약. stack-guard 자체는 성공이라 종료 X, 사용자에게 *프로젝트 수정* 안내.
   - **wiring 실패** (명령 없음 / 패키지 매니저 비호환 / 스크립트 자체 오류) → `validate smoke test: WIRING FAIL` + 생성된 명령 + 실패 stderr + 제안 대체 (예: pnpm 비호환 → `npm run validate`). **stack-guard 산출물 수정 필요** — 종료.

   > 핵심 구분: stack-guard 의 책무는 *wiring* 까지. 프로젝트 실 위반은 *프로젝트 책무* 라 smoke test 가 잡되 stack-guard 가 차단하지 않는다.
```

**(2) `stack-guard/SKILL.md` "마지막 출력" 단락 (현재 line 49~54) 마지막 줄 (`- 다음 권장 단계 ...`) *직전* 에 1줄 추가**

현재 line 53 부근:
```markdown
- 매뉴얼 hook 등록 절차 SSOT 위치 (...) — 생성된 STACK_SETUP_PLAN.md에는 link만 박힘.
- 다음 권장 단계 (`/plan-workitem` 또는 `/implement-workitem`)
```

After (위 두 줄 사이 에 다음 1줄 삽입):
```markdown
- validate smoke test 결과 (PASS / PASS with warning / FAIL with stderr 요약)
```

### 검증
- 수행 단계가 5단계로 늘었고 5번이 `Smoke test (필수)`.
- "마지막 출력" 에 smoke test 결과 항목이 박혀 있다.

---

## Step 17. `/stack-guard` `${CLAUDE_PROJECT_DIR}` + exec form 표준화 (2 OS 예시) (IMPROVE-LIST #18)

### 대상 파일
- `docs/00-meta/GUARDRAILS_STRATEGY.md`
- `.claude/skills/stack-guard/SKILL.md`

### 현재 상태
- `GUARDRAILS_STRATEGY.md` line 80~92 의 PostToolUse hook 매뉴얼 등록 절차 예시: `"command": "pnpm validate"` — bare command. CWD drift 위험 + `.cmd` shim 호환 미보장.
- `stack-guard/SKILL.md` R0 단계 (line 31~35) 에 OS 분기 명시 없음.

### 변경 액션
(1) `GUARDRAILS_STRATEGY.md` 의 hook 예시를 Unix/macOS + Windows 두 예시로 교체 + `${CLAUDE_PROJECT_DIR}` 절대 경로 / `args` exec form 명시. `if` 필드는 *미사용* — Anthropic hooks docs 의 단일-rule 제약 (`|`/`&&` 미지원) 회피, 파일 확장자 필터는 verify 스크립트 내부에서 처리.
(2) `stack-guard/SKILL.md` R0 단계 에 OS 분기 blurb 추가.

### Before/After

**(1) `GUARDRAILS_STRATEGY.md` line 81~92 (`1. \`.claude/settings.local.json\` 생성 또는 수정:` ~ 마지막 코드 블록 + line 92 의 주의 단락)**

Before (현재 정확한 본문):
```markdown
1. `.claude/settings.local.json` 생성 또는 수정:
```json
{
  "hooks": {
    "PostToolUse": [
      { "matcher": "Edit|Write", "hooks": [{ "type": "command", "command": "pnpm validate" }] }
    ]
  }
}
```

2. 주의: `defaultMode: "acceptEdits"` 환경에서 PostToolUse hook은 매 Edit/Write마다 실행 → 비용 폭증 위험. 로컬에서만 활성화 권장.
```

After:
````markdown
1. `.claude/settings.local.json` 생성 또는 수정.

**Unix/macOS 예시:**

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
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
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "powershell",
        "args": ["-File", "${CLAUDE_PROJECT_DIR}/scripts/verify.ps1", "--changed"]
      }]
    }]
  }
}
```

> **본 hook 패턴의 핵심 3 가지**:
> - `${CLAUDE_PROJECT_DIR}` 절대 경로 — CWD drift 회피 (Anthropic open issue #50960 다중 reproducer 대응).
> - `args` 배열 (exec form) — shell escaping 회피, `.cmd` shim 대응 (Windows 는 `powershell` 또는 `node` 직접 호출).
> - `matcher: "Write|Edit"` — 도구 이름 필터. 파일 확장자 필터는 *verify 스크립트 내부* 에서 처리한다 (예: `verify.sh --changed` 가 `git diff --name-only` 로 변경 파일을 추려 확장자별 분기).
>
> **Schema 주의 — `if` 필드 미사용**: Anthropic [hooks docs](https://code.claude.com/docs/en/hooks) 에 따르면 hook 의 `if` 필드는 *정확히 하나의 permission rule* 만 받으며 `|`/`&&`/list 같은 결합 syntax 를 지원하지 않는다. 따라서 본 예시는 *`if` 없이 matcher 만 사용 + verify 스크립트 내부 확장자 필터링* 패턴으로 박는다. fork 사용자가 *Edit / Write 별로 다른 동작이 필요* 하면 **두 hook handler 로 분리** 한다 (`matcher: "Edit"` 1개 + `matcher: "Write"` 1개 — 각자 자기 `if` 단일 rule).

2. 주의: `defaultMode: "acceptEdits"` 환경에서 PostToolUse hook 은 매 Write/Edit 마다 실행 → 비용 폭증 위험. 로컬에서만 활성화 권장. (Step 19 의 `async`/`asyncRewake` 옵션 패턴으로 비용 폭증 완화 가능 — `asyncRewake` 는 exit code 2 에서 Claude 를 깨운다.)
````

> **IMPROVE-GUIDE 해설** (GUARDRAILS_STRATEGY.md 에는 박지 않음): 위 두 OS 예시의 변경 핵심은 인용 quote 블록 안의 3 가지 (`${CLAUDE_PROJECT_DIR}` / `args` exec form / `matcher` 만 사용 + verify 스크립트 내부 필터) 와 `if` 미사용 schema 주의 단락에 응축. 본 quote 블록은 GUARDRAILS_STRATEGY.md 의 운영 docs 에 *자연스러운 sub-단락* 으로 박힌다 — outer numbered list (`1.` / `2.`) 와 충돌하지 않는 형식.

**(2) `stack-guard/SKILL.md` R0 단계 (현재 line 31~35) 마지막 줄 (line 35 `.gitattributes 통일은 항상 ...`) *바로 뒤* 에 다음 블록 삽입**

After (line 35 뒤, 새 블록):
```markdown
- 단일 OS/셸이 Windows로 판정되면 `scripts/verify.ps1` 우선 + 매뉴얼 hook 예시는 PowerShell exec form ([GUARDRAILS_STRATEGY.md Windows 예시](../../../docs/00-meta/GUARDRAILS_STRATEGY.md)).
- macOS/Linux 판정 또는 mixed env면 `scripts/verify.sh` 우선 + exec form 그대로 (Unix/macOS 예시).
- mixed env면 *두 verify 스크립트 모두 생성* (`.sh` + `.ps1`) + 두 hook 예시 모두 출력.
- 두 OS 모두 매뉴얼 hook 예시 본문은 `${CLAUDE_PROJECT_DIR}` + `args` 배열로 박는다 (Anthropic open issue #50960 다중 reproducer 대응).
```

### 검증
- `GUARDRAILS_STRATEGY.md` hook 예시가 Unix + Windows 두 개로 분리됐고 `${CLAUDE_PROJECT_DIR}` / `args` exec form 박혀 있다. `if` 필드는 *없음* + Schema 주의 단락에서 `if` 미사용 사유 (단일-rule 제약) 박혀 있다.
- `stack-guard/SKILL.md` R0 단계에 OS 분기 4줄이 박혀 있다.

---

## Step 18. `/stack-guard` 스택별 verify 풀세트 — TS-first depth (IMPROVE-LIST #19)

### 대상 파일
- `.claude/skills/stack-guard/SKILL.md`

### 현재 상태
- "정적 분석 도구 권장" 표 (line 56~63) 만 있음. *lint + format + typecheck + unit + e2e* 풀세트 default 미명시.

### 변경 액션
"정적 분석 도구 권장" 표 (line 56~63) **바로 뒤** (line 64 위치) 에 새 단락 1개 추가.

### Before/After

**`stack-guard/SKILL.md` — 현재 line 63 (`| Rust | \`cargo deny\` + \`cargo udeps\` | ... |`) 뒤 (line 64 빈 줄 자리) 에 다음 블록 삽입**

After:
```markdown

## 스택별 verify 풀세트 (default template)

본 표는 *runtime / 언어* 축으로 verify 도구 default 를 박는다. [ADR-031](../../../docs/90-decisions/boilerplate/ADR-031-non-web-out-of-scope.md) 의 *프로젝트 유형 축* (web frontend / API server / CLI / monorepo / Supabase) 과는 *직교 차원* — 한 프로젝트는 *유형 1 + runtime 1* 의 조합으로 자기 verify 명령을 박는다 (예: *TS web frontend* = 유형 "web frontend" × runtime "TS" → TS web 행 적용). 본 표 자체는 ADR-031 의 직접 지원 5 유형을 *축소하거나 대체하지 않는다*.

| runtime / 언어 (예시 프로젝트 유형) | format | lint | typecheck | unit test | e2e test |
|------|--------|------|-----------|-----------|----------|
| TS web (Next/Vite — 유형: web frontend) | Biome (또는 Prettier) | Biome (또는 ESLint) | `tsc --noEmit` | Vitest | Playwright |
| TS API (Express/Fastify/Hono — 유형: API server) | Biome (또는 Prettier) | Biome (또는 ESLint) | `tsc --noEmit` | Vitest | supertest 또는 동등 |
| TS CLI (유형: CLI) | Biome (또는 Prettier) | Biome (또는 ESLint) | `tsc --noEmit` | Vitest | (선택, snapshot) |
| TS monorepo (유형: monorepo — Nx/Turbo) | Biome (또는 Prettier) | Biome (또는 ESLint) | `tsc --noEmit` (workspace 별) | Vitest | 패키지별 적용 |
| TS + Supabase (유형: Supabase 통합) | Biome (또는 Prettier) | Biome (또는 ESLint) | `tsc --noEmit` | Vitest | Supabase test runner |
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

**Dependency 설치 정책** (네트워크 / 환경 의존도 큼 — 기본은 *설치하지 않음*):

- `/stack-guard` 는 *직접 패키지를 install 하지 않는다*. 산출은 `package.json` 의 `scripts.validate` 진입점 + verify 스크립트 본문 + *권장 devDeps 목록* (예: `biome / typescript / vitest / @playwright/test`).
- 출력에 `필요한 devDependencies (사용자가 npm install / pnpm add -D 로 직접 설치)` 섹션을 박는다 — 설치 명령 텍스트는 권장이지 자동 실행 X.
- 이유: 네트워크 환경 / 사용자 승인 / 기존 lockfile 충돌 / monorepo 의 workspace 라우팅 등 도구가 자동 판단하기 어려운 변수 존재. 자동 설치는 sandbox 정책 위반 위험도.
- 이미 설치돼 있으면 별도 출력 없이 verify 스크립트만 박는다.
```

또한 **`stack-guard/SKILL.md` "마지막 출력" 단락** (Step 16 갱신 후 5줄) 마지막 줄 *직후* (= 다음 권장 단계 줄 뒤) 에 1줄 추가:

After (`stack-guard/SKILL.md` "마지막 출력" 단락 끝 에 추가):
```markdown
- 스택별 default verify template은 본 skill의 "스택별 verify 풀세트" 표 기준. 도구 변경 시 ARCHITECTURE_OVERVIEW.md ## 7-X 갱신.
```

### 검증
- `stack-guard/SKILL.md` 에 `## 스택별 verify 풀세트` 단락이 정적 분석 권장 표 뒤에 박혀 있다.
- "마지막 출력" 에 default verify template 안내 1줄이 박혀 있다.

---

## Step 19. PostToolUse hook async adapter — 옵션 출력 (IMPROVE-LIST #20)

### 대상 파일
- `docs/00-meta/GUARDRAILS_STRATEGY.md`
- `.claude/skills/stack-guard/SKILL.md`

### 현재 상태
- `GUARDRAILS_STRATEGY.md` line 50~60 "1단계 비범위 (prototyping 후 분리)" — 비용 폭증 우려로 자동 등록 보류 명시. `async`/`asyncRewake`/`if` 패턴 정보 없음.
- `stack-guard/SKILL.md` line 20 — 1단계 비범위 1줄.

### 변경 액션
(1) `GUARDRAILS_STRATEGY.md` "1단계 비범위" 단락의 사유 텍스트 (line 59~60) 를 *옵션 출력* surface 로 갱신.
(2) `stack-guard/SKILL.md` "마지막 출력" 단락에 *Claude PostToolUse async adapter 예시* 항목 추가 (옵션 출력).

### Before/After

**(1) `GUARDRAILS_STRATEGY.md` line 59~60 (현재 정확한 2줄)**

Before:
```markdown
**1단계 비범위 (prototyping 후 분리)**:
- PostToolUse hook 자동 등록 — `acceptEdits` 모드에서 매 Edit/Write마다 lint를 자동 실행하면 비용이 폭증할 수 있다(사용자가 수락 프롬프트로 차단할 기회조차 없음). hook 입출력 JSON 스키마와 settings.json patch 양식이 본 시점에 직접 검증되지 않았다. prototyping 단계에서 (1) hook 입출력 스키마와 settings.json patch 양식 (2) `acceptEdits` 모드의 실측 비용 (3) 단일 OS/셸 가정의 자동 감지 신뢰도를 측정한 뒤 별도 항목(`5b. /stack-guard hook 자동 등록`)으로 분리한다.
```

After:
```markdown
**1단계 비범위 (사용자 옵션 — shared 자동 등록 X)**:
- PostToolUse hook은 본 1단계에서 **`.claude/settings.json` shared에 자동 박지 않는다** ([ADR-010](../90-decisions/boilerplate/ADR-010-multi-agent-compatibility.md) multi-tool parity 정합 — canonical 검증은 `validate` 스크립트, hook은 Claude-only adapter).
- Anthropic 2026 hooks docs의 `async: true` / `asyncRewake: true` 2 패턴이 *비용 폭증 우려*를 완화한다 (async 백그라운드 실행 + 실패 시만 깨움). `/stack-guard`는 **이 패턴 예시를 *옵션 출력*으로 박는다** (사용자가 채택 시 `.claude/settings.local.json`에 복사). 파일 확장자 필터링은 verify 스크립트 내부 처리 — `if` 필드는 단일 permission rule 제약(`|`/`&&` 미지원)으로 미사용.
- **canonical 검증은 hook 도입 여부와 무관하게 작동** — `/validate-workitem`, `/finalize-workitem`, `/stabilize-milestone` 각각이 동기 `validate` 호출을 가짐 (ADR-007 lifecycle 정합).
```

**(2) `stack-guard/SKILL.md` "마지막 출력" 단락 끝 (Step 18 에서 추가한 default verify 안내 1줄 직후) 에 다음 블록 추가**

After (옵션 출력 블록):
````markdown
- **옵션: Claude PostToolUse async adapter 예시** (사용자가 채택 시 `.claude/settings.local.json` 에 복사). GUARDRAILS_STRATEGY.md 의 PostToolUse 동기 hook 예시와 동일하게 *Unix / Windows 2 OS 예시* 모두 제공 — 동일 schema 에 `async: true` + `asyncRewake: true` 만 추가:

**Unix/macOS 예시:**

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "${CLAUDE_PROJECT_DIR}/scripts/verify.sh",
        "args": ["--changed"],
        "async": true,
        "asyncRewake": true
      }]
    }]
  }
}
```

**Windows 예시 (PowerShell 또는 `verify.mjs` 대응):**

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "powershell",
        "args": ["-File", "${CLAUDE_PROJECT_DIR}/scripts/verify.ps1", "--changed"],
        "async": true,
        "asyncRewake": true
      }]
    }]
  }
}
```

**Schema 주의**: 위 GUARDRAILS_STRATEGY.md PostToolUse 동기 hook 예시와 동일 패턴 — `matcher` 만 사용 (도구 이름 필터). 파일 확장자 필터는 *verify 스크립트 내부* 에서 처리 — Anthropic [hooks docs](https://code.claude.com/docs/en/hooks) 의 `if` 필드 단일-rule 제약 (`|`/`&&` 미지원) 회피. `asyncRewake: true` 는 verify 가 **exit code 2** 로 종료 시 Claude 를 깨워 stderr 를 system reminder 로 주입. Windows `command`/`args` 조합은 fork 적용 시 docs 직접 확인 — **본 예시는 1 차 해석이며 schema variant 가 발견되면 SSOT 가 아님**.

도입은 사용자 결정. 본 hook은 *조기 피드백 adapter* — 실패 시 Claude를 깨워 stderr를 system reminder로 주입. **차단형 게이트 아님** (완료 판정은 동기 `validate-workitem` / `finalize-workitem` / `stabilize-milestone`이 책임).
````

### 검증
- `GUARDRAILS_STRATEGY.md` "1단계 비범위" 사유가 *옵션 surface* 로 갱신.
- `stack-guard/SKILL.md` "마지막 출력" 에 async adapter 옵션 블록이 박혀 있다.

---

## Step 20. `/stack-guard` Codex wrapper 승격 + ADR-010 Amendment 1 (IMPROVE-LIST #17)

### 대상 파일
- `.agents/skills/stack-guard/SKILL.md` (신설)
- `README.md`
- `README_ko.md`
- `docs/90-decisions/boilerplate/ADR-010-multi-agent-compatibility.md`
- `docs/90-decisions/boilerplate/README.md`

### 현재 상태
- `.agents/skills/` 에 wrapper 8 종 (`bootstrap-project/`, `bootstrap-stack/`, `finalize-workitem/`, `implement-workitem/`, `plan-workitem/`, `repair-workitem/`, `stabilize-milestone/`, `validate-workitem/`) — `stack-guard/` 없음.
- README.md line 94 / README_ko.md line 93: stack-guard 가 *자연어 호출* 목록.
- ADR-010 후속 작업 Phase 2 보류 4종에 stack-guard 포함. Amendment 없음.

### 변경 액션
(1) `.agents/skills/stack-guard/SKILL.md` 신규 생성 — 기존 wrapper 와 동일 패턴.
(2) **`.agents/skills/stack-guard/agents/openai.yaml` 신규 생성** — 기존 8개 wrapper 모두 *SKILL.md + agents/openai.yaml 2 파일 쌍* 구조. wrapper 신설 시 둘 다 생성해야 패턴 정합.
(3) README.md / README_ko.md 의 wrapper 목록 갱신 — stack-guard 를 core 로 이동, 자연어 호출 목록에서 제거.
(4) ADR-010 본문 끝에 `## Amendment 1` 추가.
(5) ADR 인덱스 README 의 ADR-010 행 Amendments 칼럼 갱신.

### Before/After

**(1) 신규 파일 `.agents/skills/stack-guard/SKILL.md` 생성**

내용 (다른 wrapper 와 동일 형식):
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

> 주의: `name:` 값은 `$` *없이* (`stack-guard`) 박는다 — 기존 wrapper 8종과 동일 패턴.

**(2) 신규 파일 `.agents/skills/stack-guard/agents/openai.yaml` 생성**

기존 8 종 wrapper 가 모두 동일 내용 보유 — *복사 생성*:

```yaml
policy:
  allow_implicit_invocation: false
```

> 디렉터리 신규 생성 필요: `.agents/skills/stack-guard/` 와 `.agents/skills/stack-guard/agents/` 모두 부재 — `mkdir -p` (Unix) 또는 PowerShell `New-Item -ItemType Directory -Force` 후 두 파일 박는다.

**(3-a) `README.md` line 94**

Before (현재 정확한 1줄):
```markdown
2. Documents and policies are equal. Core workflow skills have Codex wrappers ($-prefixed): $implement-workitem, $validate-workitem, $repair-workitem, $finalize-workitem, $plan-workitem, $bootstrap-project, $bootstrap-stack, $stabilize-milestone. Remaining skills (discover-product, stack-guard, review-doc, boilerplate-context, bootstrap-design) are invoked via natural language. See [WORKFLOW.md](docs/00-meta/WORKFLOW.md).
```

After:
```markdown
2. Documents and policies are equal. Core workflow skills have Codex wrappers ($-prefixed): $implement-workitem, $validate-workitem, $repair-workitem, $finalize-workitem, $plan-workitem, $bootstrap-project, $bootstrap-stack, $stabilize-milestone, $stack-guard. Remaining skills (discover-product, review-doc, boilerplate-context, bootstrap-design) are invoked via natural language. See [WORKFLOW.md](docs/00-meta/WORKFLOW.md).
```

**(3-b) `README.md` line 98 (remaining skills 자연어 호출 안내)**

Before:
```markdown
4. For remaining skills (`discover-product`, `stack-guard`, `review-doc`, `boilerplate-context`, `bootstrap-design`), invoke in natural language: *"Follow `.claude/skills/<name>/SKILL.md`"*.
```

After:
```markdown
4. For remaining skills (`discover-product`, `review-doc`, `boilerplate-context`, `bootstrap-design`), invoke in natural language: *"Follow `.claude/skills/<name>/SKILL.md`"*.
```

**(3-c) `README_ko.md` line 93**

Before:
```markdown
2. 문서와 정책은 동일. 핵심 workflow skill은 Codex wrapper ($-prefixed)로 제공: $implement-workitem, $validate-workitem, $repair-workitem, $finalize-workitem, $plan-workitem, $bootstrap-project, $bootstrap-stack, $stabilize-milestone. 나머지 skill (discover-product, stack-guard, review-doc, boilerplate-context, bootstrap-design)은 자연어로 호출. 자세한 워크플로우는 [WORKFLOW.md](docs/00-meta/WORKFLOW.md) 참조.
```

After:
```markdown
2. 문서와 정책은 동일. 핵심 workflow skill은 Codex wrapper ($-prefixed)로 제공: $implement-workitem, $validate-workitem, $repair-workitem, $finalize-workitem, $plan-workitem, $bootstrap-project, $bootstrap-stack, $stabilize-milestone, $stack-guard. 나머지 skill (discover-product, review-doc, boilerplate-context, bootstrap-design)은 자연어로 호출. 자세한 워크플로우는 [WORKFLOW.md](docs/00-meta/WORKFLOW.md) 참조.
```

**(3-d) `README_ko.md` line 97**

Before:
```markdown
4. 나머지 skill(`discover-product`, `stack-guard`, `review-doc`, `boilerplate-context`, `bootstrap-design`)은 자연어로 호출: *"Follow `.claude/skills/<name>/SKILL.md`"*
```

After:
```markdown
4. 나머지 skill(`discover-product`, `review-doc`, `boilerplate-context`, `bootstrap-design`)은 자연어로 호출: *"Follow `.claude/skills/<name>/SKILL.md`"*
```

**(4) `ADR-010-multi-agent-compatibility.md` 파일 끝 (마지막 line 65 뒤) 에 다음 단락 추가**

After (파일 끝에 append):
```markdown

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
```

**(5) `docs/90-decisions/boilerplate/README.md` line 17 (ADR-010 행)**

Before (현재 정확한 1줄):
```markdown
| 010 | Multi-agent compatibility (AGENTS.md as canonical entry) | accepted | — | AGENTS.md를 캐노니컬 진입 페이지로, Codex CLI도 동일 워크플로우 동작 |
```

After:
```markdown
| 010 | Multi-agent compatibility (AGENTS.md as canonical entry) | accepted | (+amend1: Phase 2.5 stack-guard wrapper 승격) | AGENTS.md를 캐노니컬 진입 페이지로, Codex CLI도 동일 워크플로우 동작 |
```

### 검증
- `.agents/skills/stack-guard/SKILL.md` 파일 존재 + 다른 wrapper 와 동일 형식.
- `.agents/skills/stack-guard/agents/openai.yaml` 파일 존재 + 기존 8 wrapper 와 동일 내용 (`policy.allow_implicit_invocation: false`).
- 기존 wrapper 8개 (`bootstrap-project / bootstrap-stack / finalize-workitem / implement-workitem / plan-workitem / repair-workitem / stabilize-milestone / validate-workitem`) 가 모두 *SKILL.md + agents/openai.yaml 쌍* 인지 `glob .agents/skills/*/agents/openai.yaml` 로 재확인 (9개 결과 = 새 stack-guard 포함).
- README.md / README_ko.md 의 wrapper 목록에 `$stack-guard` 가 포함되고 자연어 호출 목록에서는 제거됨.
- **README 전수 정합 점검**: `grep -n "stack-guard" README.md README_ko.md` 로 *모든 등장 위치* 회수 + 다음 3 종류 모두 정합 확인:
  1. Core workflow Codex wrapper 목록 (line ~94 / line ~93) — 포함.
  2. 자연어 호출 skill 목록 (line ~98 / line ~97) — 제외.
  3. 본문 다른 위치 (Step 2 / Quick Start 설명 등) — `stack-guard` 가 자연어 호출 가능 skill 로 잘못 분류되지 않는지 확인.
- **Wrapper 노출 검증** (Codex 환경 있으면): Codex CLI 에서 `/skills` 또는 `$stack-guard` 호출 시 wrapper 가 listing 에 나타나는지 확인 — fork 후 첫 사용자가 즉시 확인 가능.
- ADR-010 본문에 `## Amendment 1` 단락이 박혀 있다.
- ADR 인덱스 README 의 ADR-010 행 Amendments 가 `(+amend1: Phase 2.5 stack-guard wrapper 승격)`.

---

## ✅ 그룹 C 완료 — 커밋

```
feat(stack-guard): smoke test, exec-form hooks, stack verify defaults, Codex wrapper
```

---

# 그룹 D — stabilize-milestone + bootstrap-stack

`/stabilize-milestone` 의 cheap mechanical preflight + evaluator-optimizer 패턴 명명 + instruction improvement 보고 + bootstrap-stack 운영 기술 사실 체크리스트.

---

## Step 21. `/stabilize-milestone` deterministic pre-flight (Step 1.0) 신설 (IMPROVE-LIST #21)

### 대상 파일
- `.claude/skills/stabilize-milestone/SKILL.md`

### 현재 상태
- 수행 단계 1 (line 19) milestone 문서 회수 → 1.5 (line 21~25) Graduation pre-check.
- 1.5 *앞* 의 cheap mechanical check (link 유효성 / ADR ref / FAC unmapped / 모드 라벨) 자리 없음.

### 변경 액션
수행 단계 1 (line 19) 과 1.5 (line 21) 사이 (line 20 빈 줄 위치) 에 `### 1.0. Deterministic pre-flight` 단락 신설.

### Before/After

**`stabilize-milestone/SKILL.md` — 현재 line 19 (`1. milestone 문서를 읽고 ...`) 와 line 21 (`### 1.5. Graduation pre-check (ADR-014)`) 사이 (line 20 빈 줄 위치) 에 다음 블록 삽입**

After (삽입할 블록):
```markdown

### 1.0. Deterministic pre-flight (LLM 위임 전 cheap mechanical check)

LLM 호출 전 다음을 순서대로 점검 (모두 deterministic, fail-fast X — 보고만):

1. **docs/ 내부 markdown link 유효성** (기본: *내부 / ADR 참조 / 로컬 파일* 만 점검 — 외부 URL 검사는 optional):

   - **기본 (내부 link only — deterministic 보장)**: `markdown-link-check --config <(echo '{"ignorePatterns":[{"pattern":"^https?://"}]}') docs/**/*.md` (외부 URL 무시).
     - OS 별 glob 처리:
       - Unix/macOS bash: `markdown-link-check docs/**/*.md` (bash glob 자동 확장).
       - Windows PowerShell (glob 미확장 안전 패턴): `Get-ChildItem docs -Recurse -Filter *.md | ForEach-Object { markdown-link-check $_.FullName }`.
       - OS 무관 fallback: repo 의 `scripts/verify.{sh,ps1,mjs}` 에 한 줄 helper 박거나 `npx markdown-link-check` 를 *각 파일 인자로 직접 호출* — `glob` npm 패키지 의존 회피.
   - **optional (외부 URL 검사 — 네트워크 의존 / flaky)**: 위 명령에서 `ignorePatterns` 제거. 단 *deterministic preflight 의 기본 단계가 아님* — `--with-external-links` 플래그로 사용자 명시 발화 시만.
   - 깨진 link 발견 시 IMPROVEMENT_GUIDE.md 에 `P1 [Doc-link] <file:line> — <broken link>` 라벨 기록.
   - `markdown-link-check` 미설치 환경은 본 항목만 skip + 출력에 명시 (`Doc-link check skipped: markdown-link-check not installed`).
2. **ADR 참조 유효성**: `[ADR-NNN]` 패턴 grep + 실제 파일 존재 여부 매칭.
   - 누락 발견 시 IMPROVEMENT_GUIDE에 `P1 [ADR-ref] <file:line> — ADR-NNN 본문 부재` 기록.
3. **FAC ↔ AC unmapped 검출** ([ADR-037](../../../docs/90-decisions/boilerplate/ADR-037-spec-coverage-audit.md) amend1 영속 SSOT `## 7-1` 정합):
   - 본 마일스톤의 모든 feature 문서 `## 7-1. FAC ↔ AC 매핑표`에서 *unmapped* 또는 *비어 있음* 항목 회수.
   - 발견 시 IMPROVEMENT_GUIDE에 `P0 [Spec-gap] F-NNN:FAC-N → unmapped` 기록 + 미커버 task 추가 권장.
4. **모드 라벨 ↔ 본문 정합 휴리스틱** (ADR-012): 모든 `docs/00-meta/` 파일의 `> 모드: ...` 라벨이 본문과 명백히 어긋나는지 점검 (휴리스틱 한계 명시).
   - mismatch 시 P2 `[Doc-mode] <file>` 기록.

본 단계는 모두 *보고만* — 발견이 있어도 stabilize 후속 단계 차단 X (LLM 위임 단계로 계속). 다음 라운드의 `/plan-workitem`이 후속 task로 회수.

**review-doc 책임 분담**: [review-doc](../review-doc/SKILL.md)은 *단일 문서 ad-hoc 검토*에 한정. cross-doc / link / FAC↔AC는 본 deterministic preflight가 담당 — review-doc을 `--all`/`--milestone` 모드로 확장하지 않는다.
```

### 검증
- `stabilize-milestone/SKILL.md` 수행 단계가 `1.` → `### 1.0. Deterministic pre-flight` → `### 1.5. Graduation pre-check` 순서로 박혀 있다.
- `allowed-tools` (line 6 의 `Read Glob Grep Write Edit Bash Agent`) 가 이미 Bash 포함이므로 권한 갱신 불필요.

---

## Step 22. `/stabilize-milestone` evaluator-optimizer 패턴 명명 + ADR-014 Amendment 1 (IMPROVE-LIST #22)

### 대상 파일
- `docs/90-decisions/boilerplate/ADR-014-milestone-graduation.md`
- `docs/90-decisions/boilerplate/README.md`
- `.claude/skills/stabilize-milestone/SKILL.md`
- `docs/00-meta/DELEGATION_STRATEGY.md`

### 현재 상태
- ADR-014 본문에 evaluator-optimizer 패턴 명명 없음. Amendments 없음.
- ADR 인덱스 README 의 ADR-014 행: `| 014 | Milestone graduation contract | accepted | — | ...`.
- `stabilize-milestone/SKILL.md` 첫 단락 (line 10~13) 에 패턴 명명 없음.
- `DELEGATION_STRATEGY.md` 스킬 실행 순서 가이드 (line 80~91) 에 패턴 명명 없음.

### 변경 액션
(1) ADR-014 본문 끝에 `## Amendment 1` 추가.
(2) ADR 인덱스 README 의 ADR-014 행 Amendments 갱신.
(3) `stabilize-milestone/SKILL.md` 첫 단락에 evaluator orchestration 1줄 추가.
(4) `DELEGATION_STRATEGY.md` 스킬 실행 순서 가이드 마지막에 1줄 추가.

### Before/After

**(1) `ADR-014-milestone-graduation.md` 파일 끝 (현재 line 55 의 `- ADR-022 ...` 다음) 에 다음 단락 추가**

After:
```markdown

## Amendment 1 (2026-05-16) — Evaluator-Optimizer 패턴 명명

### 결정

`/stabilize-milestone`이 instantiate하는 패턴을 Anthropic "Building Effective AI Agents" 가이드의 **evaluator-optimizer pattern**으로 명명한다.

- **Generator** = [/implement-workitem](../../../.claude/skills/implement-workitem/SKILL.md) (이전 lifecycle 단계).
- **Evaluator** = qa + reviewer agent + deterministic preflight (본 skill이 위임/실행).
- **Optimizer** = [/repair-workitem](../../../.claude/skills/repair-workitem/SKILL.md) (다음 단계, 사용자 발화).

본 skill은 evaluator 단계의 *orchestration* — 코드 수정 X, 평가 + 보고만 (책임 경계는 본 ADR의 graduation contract 정합).

### 근거

- Anthropic 단일 source의 패턴 명명은 [ADR-022](ADR-022-ratchet-principle.md) "다중 repo 실증" 기준의 *외부실증* X — *명명 자체는 행동 변화 없는 citation*이라 evidence 부담 적음.
- ADR-007 lifecycle의 책임 분할(builder = 구현, validator = 판정 + report)은 이미 패턴 정합이지만 *milestone scope*의 명명이 빠짐.

### 적용 surface

- [.claude/skills/stabilize-milestone/SKILL.md](../../../.claude/skills/stabilize-milestone/SKILL.md) 본문 첫 단락에 *"본 skill은 evaluator-optimizer pattern의 evaluator orchestration이다 (ADR-014 amend 1)"* 1줄 추가.
- [DELEGATION_STRATEGY.md](../../00-meta/DELEGATION_STRATEGY.md) 스킬 실행 순서 가이드 단락에 동일 1줄.

### 후속 작업

없음 — citation 추가만.
```

**(2) `docs/90-decisions/boilerplate/README.md` line 20 (ADR-014 행)**

Before:
```markdown
| 014 | Milestone graduation contract | accepted | — | graduation checklist 5+1 + 회고 + pre-check + --dry-run |
```

After:
```markdown
| 014 | Milestone graduation contract | accepted | (+amend1: evaluator-optimizer pattern 명명) | graduation checklist 5+1 + 회고 + pre-check + --dry-run |
```

**(3) `stabilize-milestone/SKILL.md` 첫 단락 — 현재 line 10 (`이 skill은 **코드 수정·커밋·workitem status 변경을 하지 않는다.**`) *바로 앞* (line 9 빈 줄) 에 1줄 삽입**

After (line 9 자리 에 1줄):
```markdown
본 skill은 evaluator-optimizer pattern의 evaluator orchestration이다 (ADR-014 amend 1).
```

**(4) `DELEGATION_STRATEGY.md` — 현재 line 91 (`8. 마일스톤의 모든 task가 \`done\`이 되면 \`/stabilize-milestone\` — 통합 점검(...) ...`) 단락 *바로 뒤* (line 92 빈 줄) 에 1줄 추가**

After (line 92 자리 에 1줄, 들여쓰기 동일):
```markdown
   - `/stabilize-milestone`은 evaluator-optimizer pattern의 evaluator orchestration이다 (ADR-014 amend 1) — generator=`/implement-workitem`, optimizer=`/repair-workitem`.
```

### 검증
- ADR-014 본문에 `## Amendment 1` 단락이 박혀 있다.
- ADR 인덱스 README 의 ADR-014 행 Amendments 갱신.
- `stabilize-milestone/SKILL.md` 첫 단락에 evaluator orchestration 1줄.
- `DELEGATION_STRATEGY.md` 스킬 순서 8번 아래에 1줄.

---

## Step 23. `/stabilize-milestone` 에 instruction improvement 후보 보고 추가 (IMPROVE-LIST #9)

### 대상 파일
- `.claude/skills/stabilize-milestone/SKILL.md`

### 현재 상태
- "수행" 8번 (최종 출력, line 47~52) 에 instruction improvement 후보 항목 부재.

### 변경 액션
"수행" 8번 (최종 출력) 의 마지막 항목 *바로 뒤* 에 1항목 추가.

### Before/After

**`stabilize-milestone/SKILL.md` 현재 line 47~52 (`8. 최종 출력:` ~ `   - architect 호출 권장 (있으면)`) 의 마지막 항목 (line 52) 직후 (line 53 빈 줄) 에 다음 항목 추가**

현재 line 47~52:
```markdown
8. 최종 출력:
   - 통합 `validate` 결과 + E2E 결과 (있으면)
   - P0 / P1 / P2 후속 작업
   - QA_FINDINGS / IMPROVEMENT_GUIDE 갱신 위치
   - 다음 마일스톤으로 넘기는 항목
   - architect 호출 권장 (있으면)
```

After — 마지막 항목 뒤에 추가:
```markdown
   - instruction improvement 후보:
     본 마일스톤 동안 builder/validator/reviewer가 반복적으로 막힌 패턴,
     AGENTS.md 또는 agent/skill body 문구가 *비자명하거나 모호*했던 지점,
     새로 박을 만한 *self-check 항목* 후보를
     [IMPROVEMENT_GUIDE.md](../../../docs/40-validation/IMPROVEMENT_GUIDE.md)에 보고.
     각 항목에 [ADR-022](../../../docs/90-decisions/boilerplate/ADR-022-ratchet-principle.md) evidence label 부착.
     *AGENTS.md / agent / skill body는 자동 수정 X — 보고만*.
```

### 검증
- `stabilize-milestone/SKILL.md` 최종 출력 항목이 6개 (기존 5 + 신규 1).

---

## Step 24. `/bootstrap-stack` 산출물의 ARCHITECTURE_OVERVIEW 운영 기술 사실 체크리스트 강화 (IMPROVE-LIST #10)

### 대상 파일
- `.claude/skills/bootstrap-stack/output-checklist.md`

### 현재 상태
- `output-checklist.md` 에 *ports / 실행 명령 / 환경변수 이름 / 주요 파일 역할 / known gotcha* 체크리스트 부재.

### 변경 액션
`output-checklist.md` 파일 끝에 *스택 확정 후 ARCHITECTURE_OVERVIEW.md 에 채울 운영 기술 사실* 블록 추가.

> 주의: `bootstrap-stack/output-checklist.md` 의 현재 내용을 모두 보존하면서 *파일 끝에 append* 한다. ADR-027 amend 는 본 Step 범위 외 — checklist 텍스트가 ARCHITECTURE_OVERVIEW `## 7-X` 갱신을 *권장* 만 하므로 책임 분배 변경 없음.

### Before/After

**`bootstrap-stack/output-checklist.md` 파일 끝에 다음 블록 추가**

After (파일 끝에 append):
```markdown

## 스택 확정 후 ARCHITECTURE_OVERVIEW.md 운영 기술 사실 (체크리스트)

`/bootstrap-stack` 결과로 스택이 확정되면 ARCHITECTURE_OVERVIEW.md `## 7. 기술 선택` 하위에 다음 *구체적 기술 사실*을 함께 채운다. 1줄짜리라도 명시 — 빈 placeholder는 두지 않는다.

- [ ] 실행 명령 (`pnpm dev`, `make run`, `task validate` 등) — 스택 정합
- [ ] 주요 포트 (개발/스테이징/프로덕션 분리)
- [ ] 환경변수 이름 (값은 비워둠 — secrets는 `.env`, `.gitignore` 정합)
- [ ] 주요 파일/디렉터리 역할 (3~5개 핵심만)
- [ ] known gotcha / 자주 막히는 지점 (있으면, 없으면 생략)

비-UI 프로젝트는 7-4(프론트) 섹션 생략, CLI 프로젝트는 7-2 강화 등 스택 정합.

근거: 본 보일러플레이트는 *Living Doc* 패턴 정합 — 운영 기술 사실은 fork 직후 곧장 검증되는 surface로 박힌다. ADR-027 인터페이스 결정 책임 분배는 변경 없음 (체크리스트가 ARCHITECTURE_OVERVIEW `## 7-X` 갱신을 *권장* 만 한다 — 자동 작성 X).
```

### 검증
- `bootstrap-stack/output-checklist.md` 파일 끝에 운영 기술 사실 체크리스트가 박혀 있다.

---

## ✅ 그룹 D 완료 — 커밋

```
feat(stabilize-milestone): deterministic preflight, evaluator-optimizer naming; bootstrap-stack ops facts
```

---

# 전체 적용 후 최종 점검 체크리스트

다음 모든 항목이 ✅ 면 IMPROVE-LIST 24 항목 전부 반영 완료.

**그룹 A**
- [ ] `review-doc/SKILL.md` `allowed-tools` 에 `Write Edit` 추가
- [ ] `reviewer.md` `tools:` 에 `Write, Edit` 추가 + 본문에 *scoped Write 사용 룰* 1줄
- [ ] `review-doc/SKILL.md` 본문에 "Write 범위 제한" plain-text label 단락
- [ ] `stabilize-milestone/SKILL.md` 책임 경계 단락이 3 종 허용 변경 형태 + 본문 끝 단락이 도입부 참조 형태
- [ ] `FEATURE_TEMPLATE.md` 에 `## 7-1. FAC ↔ AC 매핑표 (subsection of ## 7)` 신설
- [ ] `plan-workitem/SKILL.md` "feature 분해 시" / "마지막 출력" 매핑표 단락 갱신 + Legacy fallback 명시
- [ ] **`ADR-037-spec-coverage-audit.md` Amendment 1 + ADR 인덱스 README ADR-037 행 갱신**
- [ ] `validate-workitem/SKILL.md` 검증 기준 8항목 + report 양식 `## Spec coverage` 섹션

**그룹 B1** (Surgical Changes 확산)
- [ ] ADR-006 본문에 `## Amendment 1 (2026-05-16) — Surgical Changes ...` 단락 (Document Consistency 는 surface 목록에서 제외 — reviewer.md 본문 SSOT)
- [ ] ADR 인덱스 README 의 ADR-006 행 Amendments 갱신
- [ ] `builder.md` self-check 8항목 (dead code 방향 교정 + diff trace + LOC heuristic 포함)
- [ ] `validate-workitem/SKILL.md` 에 *범위 밖 변경 + diff trace audit* 4 카테고리 + report 양식 `## Diff trace audit` 섹션
- [ ] `reviewer.md` 에 `## Scope Discipline 체크` + `## Document Consistency 체크` 별도 섹션
- [ ] `stabilize-milestone/SKILL.md` reviewer 위임 호출에 `review surface: code`
- [ ] `review-doc/SKILL.md` 검토 항목 마지막 bullet 으로 `review surface: doc` 명시

**그룹 B2** (Ambiguity surfacing + AGENTS.md SSOT)
- [ ] `implement-workitem/SKILL.md` Red phase plan 단락이 `Step → verify` 형식 권장 + ambiguity surfacing 단락
- [ ] `plan-workitem/SKILL.md` 9-1 `AC interpretation diversity self-check` 단락
- [ ] `AGENTS.md` 단순성·YAGNI 단락 마지막 항목 1줄 (Surgical Changes SSOT)

**그룹 C**
- [ ] `stack-guard/SKILL.md` 수행 단계 5번 `Smoke test (필수)` + "마지막 출력" smoke test 결과 항목
- [ ] `GUARDRAILS_STRATEGY.md` PostToolUse hook 예시가 Unix/macOS + Windows 2 예시 (`${CLAUDE_PROJECT_DIR}` / `args` exec form, `if` 필드 *미사용* + Schema 주의 단락)
- [ ] `stack-guard/SKILL.md` R0 단계에 OS 분기 4줄
- [ ] `stack-guard/SKILL.md` 에 `## 스택별 verify 풀세트` 표 + TS-first depth 권고 + 도구 감지 우선 순서
- [ ] `stack-guard/SKILL.md` "마지막 출력" 에 default verify 안내 + Claude PostToolUse async adapter 옵션 블록
- [ ] `GUARDRAILS_STRATEGY.md` "1단계 비범위" 사유가 *옵션 surface* 로 갱신
- [ ] `.agents/skills/stack-guard/SKILL.md` wrapper 신설
- [ ] **`.agents/skills/stack-guard/agents/openai.yaml` 신설** (기존 8 wrapper 와 동일 — `policy.allow_implicit_invocation: false`)
- [ ] `README.md` / `README_ko.md` wrapper 목록 갱신 (`stack-guard` core 이동)
- [ ] ADR-010 본문에 `## Amendment 1 — Phase 2.5: stack-guard wrapper 승격` + 인덱스 README 갱신

**그룹 D**
- [ ] `stabilize-milestone/SKILL.md` 에 `### 1.0. Deterministic pre-flight` 신설
- [ ] ADR-014 본문에 `## Amendment 1 — Evaluator-Optimizer 패턴 명명` + 인덱스 README 갱신
- [ ] `stabilize-milestone/SKILL.md` 첫 단락 + `DELEGATION_STRATEGY.md` 스킬 순서 가이드 evaluator-optimizer 1줄
- [ ] `stabilize-milestone/SKILL.md` 최종 출력 *instruction improvement 후보* 1항목
- [ ] `bootstrap-stack/output-checklist.md` 운영 기술 사실 체크리스트

# 적용 시 가드레일 (적용하지 *말 것*)

본 가이드 적용 중 *추가로* 도입하지 *말아야* 할 사항 — IMPROVE-LIST 1372 line 의 가드레일을 그대로 박는다.

- Karpathy CLAUDE.md를 그대로 복붙해 새 file로 두지 *않는다* — AGENTS.md 와 SSOT 충돌.
- *plugin marketplace install* / *`curl >> CLAUDE.md`* 적용하지 *않는다* — ADR-010 `AGENTS.md` 캐노니컬 진입 정책 위반.
- *"bias toward caution over speed" 메타-룰* 박지 *않는다* — `--fast` 옵션이 이미 사용자 선택 trade-off surface (ADR-009).
- *Product layer 7원칙* 박지 *않는다* — ADR-035 DISCOVERY=SSOT + ADR-036 12-섹션 PRD 가 이미 cover.
- star 수를 *outcome 검증 evidence* 로 사용하지 *않는다* — *adoption signal* 로만 인용 (ADR-022 정합).
- Clean Code 6항목을 7로 확장하지 *않는다* — Scope Discipline / Document Consistency 는 별도 차원.
- 단일 case-study / 단일 testimony 를 `[외부실증]` 으로 라벨링하지 *않는다* — ADR-022 "다중 repo 실증" 기준.
- PostToolUse hook 자동 등록을 P0 강제 정책으로 박지 *않는다* — `async`/`asyncRewake` 는 보조 루프 (Step 19 정합).
- `.claude/settings.json` *shared* 에 hook 을 박지 *않는다* — Codex parity 위반 (ADR-010).
- review-doc 을 *`--all` / `--milestone`* 모드로 확장하지 *않는다* — surface 비대화. cross-doc / link / FAC↔AC 는 stabilize deterministic preflight (Step 21) 에 흡수.
- stack-guard 가 *smoke test 미통과* `validate` 명령을 산출하지 *않는다* — Step 16 필수.
- 보일러플레이트 직접 지원 스택을 *TS web 단일* 로 좁히지 *않는다* — ADR-031 의 5 스택 유지. *TS-first depth* (Step 18) 는 기본 verify template 깊이 dialing.
- 두 번째 evaluator agent 로 *architect 를 일상 호출* 하지 *않는다* — architect 는 큰 tradeoff / 큰 설계만 (DELEGATION_STRATEGY.md).
