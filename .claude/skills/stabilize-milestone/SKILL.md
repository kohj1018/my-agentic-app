---
name: stabilize-milestone
description: Stabilize a milestone — run E2E + regression + refactoring/ADR review. No code changes, no commits.
argument-hint: "[milestone id] [--dry-run]"
disable-model-invocation: true
allowed-tools: Read Glob Grep Write Edit Bash Agent
context-pack: minimal
---

본 skill은 evaluator-optimizer pattern의 evaluator orchestration이다 (ADR-014 amend 1).

이 skill은 **코드 수정·커밋·workitem status 변경을 하지 않는다.**
다음 세 종류의 문서 갱신만 정상 책임이다:
1. `docs/40-validation/QA_FINDINGS.md` 누적 기록 (qa 위임 결과).
2. `docs/40-validation/IMPROVEMENT_GUIDE.md` 누적 기록 (reviewer 위임 결과 + deterministic preflight 결과).
3. milestone 문서의 `## 8. 회고` 섹션 자동 채움 ([ADR-014](../../../docs/90-decisions/boilerplate/ADR-014-milestone-graduation.md) graduation contract — status 변경 X, 본문 단락 갱신만).
   - 회고 본문 4 항목: 목표 달성도 / scope creep / 비목표 위반 / 핵심 학습 3개 이내.

그 외 변경은 금지한다 — milestone 문서의 `## 0. Status` / `## 1~7` 섹션 / 다른 workitem 문서 / 코드 일체.
후속 작업이 필요하면 `/repair-workitem` 또는 새 task로 텍스트 제안만 출력한다.

입력:
- `$ARGUMENTS`에는 milestone ID(예: `M1`)가 들어온다.
- `--dry-run` 플래그가 있으면 1.5 Graduation pre-check만 돌리고 종료(P0 검증 도구 — 전체 QA 없이 빠른 졸업 가능 여부 확인).

수행:
1. milestone 문서를 읽고 포함된 feature/task 목록을 회수한다.

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

### 1.5. Graduation pre-check (ADR-014)

MILESTONE 문서의 `## 5. 완료 기준` 각 항목을 다음 deterministic 평가로 체크 (LLM 즉흥 판정 금지 — ADR-014 *P0 검증 도구* 정합):

- `모든 task status: done` → 본 milestone에 속한 모든 task 파일(`docs/30-workitems/tasks/T-*.md`)의 `## 0. Status` 값이 모두 `done`.
- `통합 validate Pass` → 단계 3의 `validate` 명령 exit code 0. `--dry-run` 모드에서는 본 단계 안에서 1회 실행.
- `E2E Pass (스택 정의 시)` → 단계 3의 E2E 명령 exit code 0. E2E 미정의 스택은 *해당 없음*으로 처리(통과).
- `AC 매핑 100%` → 본 milestone의 모든 task의 최신 `docs/40-validation/reports/<task-id>.md` `## AC ↔ 테스트 매핑` 섹션 항목이 모두 `✅`. report 부재 task는 미충족 처리.
- `P0 severity finding 0건` → `docs/40-validation/QA_FINDINGS.md`의 본 milestone 헤더(`## M-N`) 아래 `### P0` 섹션 항목 수 0.
- `(선택) 본 마일스톤 한정 추가 기준` → 본문 텍스트 그대로 평가(사용자가 자유 기재한 영역 — 해당 항목만 LLM 해석 허용).

판정 출력:
- 미충족 항목 발견 시 `졸업 가능: NO` + 미충족 항목 목록을 출력하고 *조기 종료 옵션*을 사용자에게 제시한다(강제 종료 아님).
- 모든 항목 충족 시 `졸업 가능: YES` 출력 후 다음 단계 진행.
- `--dry-run` 플래그가 켜져 있으면 위 평가만 돌리고 즉시 종료(qa·reviewer 위임 단계 4~6 생략).

2. 각 task의 status를 점검 — `done`이 아닌 항목이 있으면 명단을 출력하고 종료(완료를 강제하지 않음).
3. 통합 `validate` 명령을 실행한다 + (있으면) E2E 명령을 실행한다.
4. **qa agent에 회귀·엣지케이스 점검 위임** — qa는 보고만 한다(qa.md의 tools에 Write 없음). 반환된 보고를 본 skill이 받아 `docs/40-validation/QA_FINDINGS.md`에 누적 기록한다.
5. **reviewer agent에 리팩토링 후보·아키텍처 부채 점검 위임** — reviewer 입력에 Clean Code 6항목 체크리스트(ADR-006) + `review surface: code` 를 명시 전달한다. reviewer도 보고만 한다. 반환된 보고를 본 skill이 받아 `docs/40-validation/IMPROVEMENT_GUIDE.md`에 정리.
   - reviewer 결과에 구조 변경이 필요해 보이면 메인 세션에 architect 추가 호출을 텍스트로 제안.
6. 미흡한 ADR 후보 제안 — 마일스톤 중에 내려진 결정인데 ADR이 없는 것을 식별. ADR 후보 기준에 "layer 경계·의존성 규칙 변경"도 포함(ADR-006 정책).
   - ARCHITECTURE_OVERVIEW.md에 비해당 7-x sub-section이 *잔존*하면 IMPROVEMENT_GUIDE.md에 P2 보고 — *"조건부 sub-section 미삭제. /bootstrap-stack 재실행 또는 수동 삭제 권장."*
   - layer 경계·의존성 규칙 변경(ARCHITECTURE_OVERVIEW의 ## 3-1)이 마일스톤 중에 발생했으면 ADR 후보로 표시한다(정책: ADR-006).
### 6.5. DISCOVERY ↔ Charter staleness 감지 (ADR-035 amend 1)

다음 3 시그널을 점검한다 (보고만, 자동 차단 X — validator 책임 경계 정합).

1. `docs/10-charter/DISCOVERY.md`의 mtime이 `docs/10-charter/PROJECT_CHARTER.md`의 mtime보다 최신인지.
2. DISCOVERY.md `## 12. Assumption Tracker` 표에서 *"미검증"* 결과 항목 수.
3. PROJECT_CHARTER.md `## 2.1 페르소나` / `## 3.1 핵심 시나리오` / `## 9 핵심 가정` 섹션 중 비어 있거나 DISCOVERY.md와 명백히 어긋난 섹션 수.

위 3 시그널 중 1개라도 *stale 의심* 판정 시 IMPROVEMENT_GUIDE.md에 P1 보고:
*"DISCOVERY ↔ Charter drift 의심 — /bootstrap-project --apply 또는 수동 갱신 권장."*

7. ARCHITECTURE_OVERVIEW의 `## 3-1. 레이어 경계 + 의존성 규칙` 섹션이 비어 있고 모듈 수가 3개 이상이면 채울 것을 권장 출력한다(정책: ADR-006).
8. 최종 출력:
   - 통합 `validate` 결과 + E2E 결과 (있으면)
   - P0 / P1 / P2 후속 작업
   - QA_FINDINGS / IMPROVEMENT_GUIDE 갱신 위치
   - 다음 마일스톤으로 넘기는 항목
   - architect 호출 권장 (있으면)
   - instruction improvement 후보:
     본 마일스톤 동안 builder/validator/reviewer가 반복적으로 막힌 패턴,
     AGENTS.md 또는 agent/skill body 문구가 *비자명하거나 모호*했던 지점,
     새로 박을 만한 *self-check 항목* 후보를
     [IMPROVEMENT_GUIDE.md](../../../docs/40-validation/IMPROVEMENT_GUIDE.md)에 보고.
     각 항목에 [ADR-022](../../../docs/90-decisions/boilerplate/ADR-022-ratchet-principle.md) evidence label 부착.
     *AGENTS.md / agent / skill body는 자동 수정 X — 보고만*.

책임 경계:
- 코드 수정·커밋·workitem status 변경 금지.
- 누적 문서 갱신 + milestone `## 8. 회고` 자동 채움.
- *상세 SSOT 는 본 skill 도입부 책임 경계 단락* — 본 단락은 단순 재확인.

E2E 명령이 없는 스택은 3단계에서 통합 `validate`만 돌리고 E2E는 skip한다(출력에 명시).

## Dependency hygiene
- `npm audit` / `pip-audit` (스택별 대응) 1회 실행.
- 결과를 IMPROVEMENT_GUIDE.md에 P1 severity로 보고.
- 6개월 unused deps는 P2로 자동 등록.

## Context 정책 (ADR-019)
`반드시 먼저 읽을 파일`은 *최소 충분*. 추가 ADR/architecture 섹션은 task 본문에서 발화 시 인용 — 사전 fork-load 금지.
