# Improve Guide

이 가이드는 [IMPROVE-LIST.md](IMPROVE-LIST.md)의 20개 개선 항목(I-01 ~ I-20)을 실제로 적용하는 단계별 행동지침이다. **가이드만 따라 진행하면 개선을 완료할 수 있다.**

- 행동지침은 **8단계**로 구성된다. 각 단계는 여러 sub-step으로 나뉜다.
- 각 sub-step 끝에는 영어 한 줄 커밋 메시지를 박는다.
- 모든 파일 경로는 **프로젝트 루트 기준 상대 경로**다.
- before/after 코드 블록은 *해당 sub-step 직전 상태*와 일치한다. **일부 before는 선행 단계 적용 후 상태 기준**이다 — 예: 4단계 ARCHITECTURE 7-1 before는 3단계 agent rename 적용 후(`architect`) 기준. 막힐 때는 *선행 단계 완료 상태*인지 먼저 확인.
- Windows 환경 사용자는 **부록 B: PowerShell 대안**을 참고. 본문 명령은 Bash / Git Bash / WSL 기준.
- IMPROVE-LIST.md / IMPROVE-GUIDE.md 자체는 **sweep 대상에서 제외** — 두 파일은 *과거 분석 + 가이드*라 옛 용어 인용이 정당. 8단계 grep 검증에 `--exclude="IMPROVE-*.md"` 박혀 있음.

---

## 사전 준비

1. 작업 중 새 디렉터리·파일 생성 전 `ls`로 부모 디렉터리 존재 확인.
2. 단계마다 1 커밋이 기본 단위. 단계 안에 sub-step이 여러 개면 sub-step마다 커밋.
3. **모든 `grep` / `git mv` / `mkdir` 명령은 프로젝트 루트(`my-agentic-app/`)에서 실행한다.** 하위 디렉터리에서 실행하면 grep 0건이 잘못 통과될 수 있다.

---

## 1단계 — baseline / generated 경계 정리

대상 항목: **I-18, I-03, I-04** (I-02는 I-18 적용으로 자동 해소).

### 1.1 STACK_SETUP_PLAN.md를 template + GUARDRAILS로 분리 (I-18)

**목적**: baseline에서 STACK_SETUP_PLAN.md를 빼고 *보일러플레이트 정책*은 GUARDRAILS_STRATEGY.md로, *generated 부분*은 `_templates/`로 분리. 결과적으로 ADR-012의 *"00-meta 6개 파일"* 원칙도 자동 복원 (= I-02 해소).

**행동지침**:

1. `docs/00-meta/_templates/` 디렉터리 생성.
   ```bash
   mkdir -p docs/00-meta/_templates
   ```

2. 현재 `docs/00-meta/STACK_SETUP_PLAN.md` 본문을 두 영역으로 분리.
   - **GUARDRAILS_STRATEGY.md로 이동할 부분**: `## PostToolUse hook 매뉴얼 등록 절차` 단락 전체 (50줄 부근).
   - **template으로 남을 부분**: `## 외부 의존 부트업`, `## 통합 명령 사용법`, `## CI 권장 출력` 세 단락.

3. `docs/00-meta/GUARDRAILS_STRATEGY.md` 끝부분에 `## PostToolUse hook 매뉴얼 등록 절차` 단락을 그대로 옮긴다. 본문 첫 줄에 한 줄 추가:
   ```markdown
   > 본 단락은 STACK_SETUP_PLAN_TEMPLATE.md에서 이관됨 (I-18, IMPROVE-LIST). hook 자동 등록 정책의 SSOT는 본 파일.
   ```

4. `docs/00-meta/STACK_SETUP_PLAN.md`를 `docs/00-meta/_templates/STACK_SETUP_PLAN_TEMPLATE.md`로 이동.
   ```bash
   git mv docs/00-meta/STACK_SETUP_PLAN.md docs/00-meta/_templates/STACK_SETUP_PLAN_TEMPLATE.md
   ```

5. `STACK_SETUP_PLAN_TEMPLATE.md` 본문 최상단(`# Stack Setup Plan` 다음 줄)에 주석 추가.
   ```markdown
   <!-- 본 파일은 /bootstrap-stack이 docs/00-meta/STACK_SETUP_PLAN.md를 *최초 생성*할 때 복사하는 template.
        baseline에는 본 template만 존재. 실제 STACK_SETUP_PLAN.md는 스택 결정 후 생성된다. -->
   ```

6. `.claude/skills/bootstrap-stack/SKILL.md`의 "반드시 수행할 일" 단락에 1행 추가.
   ```markdown
   - `docs/00-meta/_templates/STACK_SETUP_PLAN_TEMPLATE.md`를 복사해 `docs/00-meta/STACK_SETUP_PLAN.md`를 생성 (이미 있으면 갱신 제안만).
   ```

7. `docs/00-meta/STRUCTURE.md` 산출물 표에서 `stack setup plan` 행을 두 행으로 분리. **본 변경은 4컬럼 형식 — 1.3에서 5번째 컬럼 `presence` 박을 예정**.
   - 기존:
     ```
     | stack setup plan | docs/00-meta/STACK_SETUP_PLAN.md | /bootstrap-stack, /stack-guard | Reference |
     ```
   - 변경 후 (4컬럼, 1.3 적용 전):
     ```
     | stack setup plan template | docs/00-meta/_templates/STACK_SETUP_PLAN_TEMPLATE.md | 수동 (boilerplate 제공) | Reference |
     | stack setup plan | docs/00-meta/STACK_SETUP_PLAN.md | /bootstrap-stack, /stack-guard | Reference |
     ```
   - 1.3 적용 후 (5컬럼, 최종 형식):
     ```
     | stack setup plan template | docs/00-meta/_templates/STACK_SETUP_PLAN_TEMPLATE.md | 수동 (boilerplate 제공) | Reference | baseline |
     | stack setup plan | docs/00-meta/STACK_SETUP_PLAN.md | /bootstrap-stack, /stack-guard | Reference | generated |
     ```

8. `ls docs/00-meta/` 결과로 다음 6개 + `_templates/` 1개를 확인.
   - AGENT_EXECUTION_STRATEGY.md (3단계에서 DELEGATION_STRATEGY.md로 rename 예정)
   - GLOSSARY.md
   - GUARDRAILS_STRATEGY.md
   - NEW_PROJECT_CHECKLIST.md (6단계에서 PROJECT_START_CHECKLIST.md로 rename 예정)
   - STRUCTURE.md
   - WORKFLOW.md
   - `_templates/`
   
   *(파일이 정확히 6개로 줄어들어 ADR-012의 "6개 파일" 원칙 자동 복원 = I-02 해소.)*

**커밋 메시지**:
```
refactor(meta): extract STACK_SETUP_PLAN to template and move hook policy to GUARDRAILS
```

---

### 1.2 DESIGN.md를 baseline placeholder로 명시 (I-03)

**목적**: baseline에 존재하는 DESIGN.md가 *"UI 프로젝트에서만 만들어진다"*고 모순적으로 적힌 문제 해소. 안 B(현재 위치 유지 + 명시 주석) 채택 — ADR-006 단순성 정합.

**행동지침**:

1. `docs/20-system/DESIGN.md` 본문 최상단 주석 8행 부근을 다음으로 교체.
   - **before** (line 7-10 부근):
     ```markdown
     <!-- 본 문서는 UI 프로젝트일 때만 만들어진다. /bootstrap-design이 R0~R4 라운드를 통해 채운다.
          비-UI 프로젝트(API 서버 / CLI 도구)는 본 파일을 만들지 않는다. -->
     ```
   - **after**:
     ```markdown
     <!-- 본 문서는 UI 프로젝트의 시각 결정 SSOT.
          baseline에는 placeholder로 존재한다 (presence: conditional, STRUCTURE.md 참조).
          - UI 프로젝트: /bootstrap-design이 R0~R4 라운드로 본 파일을 채운다.
          - 비-UI 프로젝트(API 서버 / CLI 도구 등): fork 직후 본 파일을 삭제한다. -->
     ```

2. `.claude/skills/bootstrap-design/SKILL.md` 본문에 1행 추가.
   ```markdown
   - 본 skill은 baseline placeholder DESIGN.md를 *채우는* 흐름. 비-UI 프로젝트는 fork 직후 DESIGN.md를 삭제했음을 전제. 파일 부재 시 작업 중단 + 사용자에게 보고.
   ```

**커밋 메시지**:
```
docs(design): clarify DESIGN.md as baseline placeholder with deletion path for non-UI forks
```

---

### 1.3 STRUCTURE.md 산출물 표에 `presence` 컬럼 추가 (I-04)

**목적**: baseline / generated / conditional / reserved 4종 산출물이 한 표에 섞여 *AI agent가 누락인지 미생성인지 판단 불가*한 문제 해소.

**행동지침**:

1. `docs/00-meta/STRUCTURE.md` 산출물 표 헤더를 4컬럼 → 5컬럼으로 갱신.
   - **before**:
     ```
     | 산출물 | 위치 | 생성 주체 | 라이프사이클 |
     |--------|------|-----------|--------------|
     ```
   - **after**:
     ```
     | 산출물 | 위치 | 생성 주체 | 라이프사이클 | presence |
     |--------|------|-----------|--------------|----------|
     ```

2. 표 직전(`## 산출물 표` 헤더와 표 사이)에 `presence` 값 정의 단락 추가.
   ```markdown
   `presence` 컬럼은 산출물이 *어떤 상태로 존재하는가*를 표시한다.
   
   - **baseline**: 보일러플레이트가 *이미 박아 둔* 파일 (템플릿 / 정책 placeholder 포함).
   - **generated**: skill 호출 시 *최초 생성*되는 파일. baseline에는 없음.
   - **conditional**: 특정 스택에서만 존재 (예: UI 한정).
   - **reserved**: 번호 placeholder. 미생성. fork 사용자가 채우거나 dropped 처리.
   - **boilerplate-only**: 보일러플레이트 자체 검증·메타 자료. fork 후 read-only. 프로젝트 산출물 아님.
   ```

3. 표의 각 행 끝에 presence 값 박음. 매핑은 다음과 같다.

   | 산출물 | presence | 후속 단계 영향 |
   |--------|----------|---------------|
   | project charter | baseline | — |
   | discovery | generated | — |
   | architecture overview | baseline | — |
   | design (UI only) | conditional | — |
   | bootstrap-design skill | baseline | — |
   | milestone | generated | — |
   | feature | generated | — |
   | task | generated | — |
   | validation report | generated | — |
   | qa findings | baseline | — |
   | simulation run | (행 자체를 2.1에서 본 표에서 *제거* → 별도 *보일러플레이트 메타 산출물* 표로 이관) | 2.1 step 3·4 |
   | improvement guide | baseline | — |
   | ADR | baseline (ADR-002 / ADR-003은 legacy reserved — 자세한 사유는 2.2 step 6-C *Reserved / Parked / Dropped* 표) | 2.2 step 13에서 *ADR (boilerplate)* / *ADR (project)* 2 행으로 분리 |
   | stack setup plan template | baseline | — |
   | stack setup plan | generated | — |
   | verify scripts | generated | — |
   | AGENTS.md | baseline | — |
   | Codex 프로젝트 설정 | baseline | — |
   | Codex skill wrapper | baseline | — |
   
   **시점 주의**: *simulation run*과 *ADR* 두 행은 1.3 시점에는 위처럼 한 행으로 박되, 후속 단계(2.1 / 2.2)에서 *별도 표 분리* 또는 *2 행 분할*로 갱신된다. 1.3에서 임시 박은 값이 *나중에 사라지거나 분할*되는 게 정상 흐름.

**커밋 메시지**:
```
docs(structure): add presence column to artifact inventory for baseline/generated/conditional disambiguation
```

---

## 2단계 — 보일러플레이트 영역 분리

대상 항목: **I-14, I-16** + **I-07 / I-08 설계 + 박음** (7단계는 *완성 검증*만).

> 모든 grep / git 명령은 **프로젝트 루트**(`my-agentic-app/`)에서 실행한다.

### 2.1 SIMULATION_RUN.md를 .boilerplate/validation/로 이동 (I-16)

**목적**: 보일러플레이트 자체 dogfood 기록이 fork 사용자의 QA 영역(`docs/40-validation/`)과 동거하는 모순 해소.

**행동지침**:

1. `.boilerplate/validation/` 디렉터리 생성.
   ```bash
   mkdir -p .boilerplate/validation
   ```

2. SIMULATION_RUN.md 이동.
   ```bash
   git mv docs/40-validation/SIMULATION_RUN.md .boilerplate/validation/SIMULATION_RUN.md
   ```

3. `docs/00-meta/STRUCTURE.md`에 *보일러플레이트 메타 산출물* 별도 표 신설. 위치는 *Canonical Owner 매핑(SSOT 부록)* 단락 바로 위.
   ```markdown
   ## 보일러플레이트 메타 산출물
   
   `.boilerplate/` 디렉터리는 보일러플레이트 *자체 검증·메타 자료* 영역이다.
   fork 후 read-only로 취급한다 — 프로젝트 산출물이 아니다.
   
   | 산출물 | 위치 | 생성 주체 | 라이프사이클 | presence |
   |--------|------|-----------|--------------|----------|
   | simulation run | .boilerplate/validation/SIMULATION_RUN.md | 수동 (보일러플레이트 진화 라운드별 누적) | Record | boilerplate-only |
   ```

4. STRUCTURE.md 본래 산출물 표에서 *simulation run* 행 삭제 (위 별도 표로 이관).

5. `docs/90-decisions/ADR-017-dogfood-simulation.md`의 `## 결정 → 3. 산출물` 단락에서 경로 갱신.
   - **before**: `\`docs/40-validation/SIMULATION_RUN.md\` (Record 타입, 회차별 누적)`
   - **after**: `\`.boilerplate/validation/SIMULATION_RUN.md\` (Record 타입, 회차별 누적. fork 사용자 영역 아님)`

6. `ADR-017` 본문 끝부분 *상세 기록 링크*도 갱신. **주의**: 본 단계 시점에 ADR-017은 아직 `docs/90-decisions/`에 있다. 2.2에서 `docs/90-decisions/boilerplate/`로 이동하면 상대 경로 1단계 더 깊어진다 — **2.2 step 14에서 재보정**.
   - **before**: `상세 기록: [SIMULATION_RUN.md](../40-validation/SIMULATION_RUN.md)`
   - **after** (2.1 시점, ADR-017이 `docs/90-decisions/`에 있을 때): `상세 기록: [SIMULATION_RUN.md](../../.boilerplate/validation/SIMULATION_RUN.md)`
   - 2.2 이동 후 최종 경로: `상세 기록: [SIMULATION_RUN.md](../../../.boilerplate/validation/SIMULATION_RUN.md)` (2.2 step 14에서 재보정)

7. `README.md` 디렉터리 트리 단락에 `.boilerplate/` 1줄 추가. **현재 트리의 `└── scripts/`(마지막 행)를 `├── scripts/`로 바꾸고 `.boilerplate/`를 새 마지막 `└──` 행으로 박음.**
   - **before** (현재 트리 마지막 2행):
     ```
     │   └── 90-decisions/  # ADR records
     └── scripts/           # Project-specific automation (after stack is chosen)
     ```
   - **after**:
     ```
     │   └── 90-decisions/  # ADR records
     ├── scripts/           # Project-specific automation (after stack is chosen)
     └── .boilerplate/      # Boilerplate self-validation/meta artifacts. Read-only after fork. Not a project artifact.
     ```

8. `README_ko.md`의 디렉터리 트리도 동일 패턴으로 갱신 (한글 설명). `└── scripts/`를 `├── scripts/`로 바꾸고 마지막 `└──`로 `.boilerplate/` 박음.
   ```
   │   └── 90-decisions/  # ADR 기록
   ├── scripts/           # 프로젝트별 자동화 (스택 확정 후 추가)
   └── .boilerplate/      # 보일러플레이트 자체 검증·메타 자료. fork 후 read-only. 프로젝트 산출물 아님.
   ```

**커밋 메시지**:
```
refactor(meta): move SIMULATION_RUN.md to .boilerplate/validation/ and document boundary
```

---

### 2.2 docs/90-decisions/를 boilerplate/ + project/로 분리 (I-14, I-07, I-08 통합)

**목적**: 보일러플레이트 ADR과 프로젝트 ADR이 같은 폴더에 섞이는 문제 해소. 동시에 새 인덱스에 *Reserved/Parked/Dropped 번호*와 *Amendments 컬럼*도 함께 박는다(I-07, I-08).

**행동지침**:

1. 두 sub-folder 생성.
   ```bash
   mkdir -p docs/90-decisions/boilerplate
   mkdir -p docs/90-decisions/project
   ```

2. 보일러플레이트 ADR 25개 + ADR 가이드를 `boilerplate/`로 이동.
   ```bash
   git mv docs/90-decisions/ADR-000-boilerplate-decision-policy.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-001-doc-hierarchy.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-004-model-alias-policy.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-005-ssot.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-006-simplicity-and-architecture.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-007-workitem-lifecycle.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-008-commit-convention.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-009-tdd-default.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-010-multi-agent-compatibility.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-011-agents-md-line-cap.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-012-doc-architecture-cleanup.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-014-milestone-graduation.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-017-dogfood-simulation.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-019-context-packs-and-jit.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-020-incremental-validate.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-021-static-analysis-recommendation.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-022-ratchet-principle.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-024-plan-mode-out-of-lifecycle.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-025-external-deps-and-ci-recommendation.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-026-plan-workitem-schema.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-027-interface-decision-allocation.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-031-non-web-out-of-scope.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-035-continuous-discovery.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-036-feature-level-prd.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/ADR-037-spec-coverage-audit.md docs/90-decisions/boilerplate/
   git mv docs/90-decisions/_ADR_GUIDE.md docs/90-decisions/boilerplate/
   ```

3. 기존 `docs/90-decisions/README.md`를 `docs/90-decisions/boilerplate/README.md`로 이동.
   ```bash
   git mv docs/90-decisions/README.md docs/90-decisions/boilerplate/README.md
   ```

4. 새로운 *허브* `docs/90-decisions/README.md` 생성. 내용:
   ```markdown
   # ADR Index Hub
   
   > scope 정책의 SSOT는 [boilerplate/ADR-000](boilerplate/ADR-000-boilerplate-decision-policy.md) 참조.
   
   본 디렉터리는 두 종류의 ADR을 분리해 둔다.
   
   - **[Boilerplate ADR Index](boilerplate/README.md)** — 보일러플레이트 자체 정책 (000~099). fork 후 read-only. supersede는 project/에서 박는다.
   - **[Project ADR Index](project/README.md)** — fork 사용자가 박는 프로젝트 결정 (100+).
   
   ## 새 ADR을 어디 박는가
   
   - 새로 시작한 프로젝트의 첫 결정 (스택, 데이터 모델 등)을 박을 때 → `project/ADR-100-<slug>.md`. **Project ADR은 무조건 100+ 번호**. ADR-002 / ADR-003은 legacy reserved placeholder라 새로 박지 않는다.
   - 보일러플레이트 자체 정책을 갱신·추가할 때 → `boilerplate/ADR-NNN-<slug>.md` (100 미만 빈 번호 사용 또는 새 번호).
   - 기존 boilerplate 결정을 뒤집을 때 → `project/ADR-1NN-<slug>.md` (예: ADR-100, ADR-101) 본문 첫 줄에 `Supersedes ADR-NNN (boilerplate)` 명시.
   ```

5. `docs/90-decisions/project/README.md` 생성. 내용:
   ```markdown
   # Project ADR Index
   
   > 이 디렉터리는 fork된 프로젝트의 자체 결정 ADR을 박는 영역.
   > 첫 ADR은 ADR-100부터 시작 (정책: [boilerplate/ADR-000](../boilerplate/ADR-000-boilerplate-decision-policy.md)).
   
   ## ADR 목록
   
   (아직 박힌 ADR이 없습니다. fork 사용자가 첫 결정부터 채웁니다.)
   
   | # | 제목 | 상태 | 한 줄 요약 |
   |---|------|------|-----------|
   ```

6. `docs/90-decisions/boilerplate/README.md`(2단계 3번에서 이동한 파일) 갱신:
   
   **6-A. 표 헤더에 `Amendments` 컬럼 추가 (I-08).**
   - **before**:
     ```
     | # | 제목 | 상태 | 한 줄 요약 |
     |---|------|------|-----------|
     ```
   - **after**:
     ```
     | # | 제목 | 상태 | Amendments | 한 줄 요약 |
     |---|------|------|------------|-----------|
     ```

   **6-B. 각 ADR 행에 Amendments 채움.** 본 시점에서 *이미 박혀 있는* amend 라벨만 박음. 후속 단계 amend는 *해당 단계에서 박을 때 본 인덱스 행도 동시 갱신*.
   - `000`: `(+amend1: 폴더 분리)` ← 본 단계 step 8에서 박힘
   - `007`: `(+amend1: lock file whitelist 11종)` ← 기존 amend
   - `008`: `(+amend1: monorepo scope, +amend2: Refs footer)` ← 기존 amend
   - `009`: `(+amend1: AC ID 컨벤션)` ← 기존 amend
   - `017`: `(+amend1: 위치 경로 .boilerplate/)` ← 본 단계에서 박힘 (2.1 step 5/6 + 2.2 step 14)
   - `021`: `(+amend1: secret scanner)` ← 기존 amend
   - `026`: `(+amend1: planner self-check + architect 호출 신호)` ← 기존 amend
   - 나머지 ADR은 빈 칸(`—`) 박음.
   
   **후속 단계에서 추가될 amend** (해당 단계에서 박을 때 본 인덱스 행도 동시 갱신):
   - `004`: `(+amend1: agent 이름 역할 중심)` — 3단계 3.1 step 5에서 박을 때 인덱스 동시 갱신.
   - `007 amend2`: `(+amend2: agent 단위 판정 경계 SSOT)` — 3단계 3.3 step 4에서 박을 때 인덱스 동시 갱신.
   - `035`: `(+amend1: Charter staleness 보고)` — 5단계 5.2 step 2에서 박을 때 인덱스 동시 갱신.
   
   **최종 amend 대상은 9개** (000 / 004 / 007 / 008 / 009 / 017 / 021 / 026 / 035) — 7단계 7.1 검증에서 최종 확인.

   **6-C. README 끝부분에 `## Reserved / Parked / Dropped 번호` 섹션 신설 (I-07).**
   ```markdown
   ## Reserved / Parked / Dropped 번호
   
   본 보일러플레이트 진화 과정에서 *번호는 잡혔지만 ADR이 만들어지지 않은* 경우를 추적한다.
   fork 사용자는 이 번호들을 *자기 ADR 번호로 재사용하지 않는다* — Project ADR은 ADR-100부터.
   
   | # | 상태 | 사유 |
   |---|------|------|
   | ADR-002 | legacy reserved | deprecated placeholder for initial project decisions. **새 project ADR은 ADR-100+에 박음** (ADR-000 amend 1 참조). 본 번호는 재사용 X. |
   | ADR-003 | legacy reserved | deprecated placeholder for stack selection. **새 project ADR은 ADR-100+에 박음**. 본 번호는 재사용 X. |
   | ADR-013 | dropped | Phase 진화 중 fold됨 (git log: `git log --all --diff-filter=D -- "**/ADR-013*"`로 사유 확인) |
   | ADR-015 | dropped | Phase 진화 중 fold됨 |
   | ADR-016 | dropped | Phase 진화 중 fold됨 |
   | ADR-018 | parked | CODE_LINEAGE.md (Refs footer SSOT). P1 트리거 보류. ADR-008 amend 2가 인용. |
   | ADR-023 | dropped | Phase 진화 중 fold됨 |
   | ADR-028 | dropped | Phase 진화 중 fold됨 |
   | ADR-029 | dropped | Phase 진화 중 fold됨 |
   | ADR-030 | dropped | Phase 진화 중 fold됨 |
   | ADR-032 | dropped | Phase 진화 중 fold됨 |
   | ADR-033 | dropped | Phase 진화 중 fold됨 |
   | ADR-034 | dropped | Phase 진화 중 fold됨 |
   ```

   *(dropped 사유는 추후 git log 조회 후 정확히 채울 수 있음. 일단 *"Phase 진화 중 fold됨"*으로 통일.)*

7. `docs/90-decisions/boilerplate/_ADR_GUIDE.md`의 *상태값* 단락에 2 라인 추가.
   - **before**:
     ```markdown
     ## 상태값
     - `proposed`: 제안됨, 아직 확정 아님
     - `accepted`: 팀/개인이 수용함
     - `superseded`: 새 ADR로 대체됨
     - `deprecated`: 더 이상 유효하지 않음
     ```
   - **after**: 위 4 라인 + 아래 2 라인 추가.
     ```markdown
     - `reserved`: 번호만 잡힌 placeholder. 본문 미작성.
     - `parked`: 본문은 있으나 트리거 미달로 보류.
     ```

8. `docs/90-decisions/boilerplate/ADR-000-boilerplate-decision-policy.md` 끝부분에 Amendment 1 단락 추가.
   ```markdown
   ## Amendment 1 (YYYY-MM-DD) — docs/90-decisions/ 폴더 분리
   
   ### 결정
   `docs/90-decisions/`를 다음 2 sub-folder로 분리한다.
   - `boilerplate/` — 보일러플레이트 자체 ADR (000~099). fork 후 read-only.
   - `project/` — fork 사용자가 박는 프로젝트 ADR (100+).
   - `docs/90-decisions/README.md` — 두 인덱스 허브.
   
   ### 근거
   라벨 수준 분리(`> scope:`)는 *읽어야 알 수 있는* signal이지만 폴더 분리는 *읽지 않고도 보이는* signal. 6개월 운영 후 fork 사용자 시야에서 *내 ADR vs 보일러플레이트 ADR* 즉시 구분.
   
   ### supersede 흐름
   `project/ADR-100-...`에서 `Supersedes ADR-006 (boilerplate)` 형식으로 cross-folder 참조 유지. 본 amend로 ADR-000 D 정책의 *boilerplate ADR 자유 supersede* 권한 그대로 작동.
   ```
   *(YYYY-MM-DD는 작업 시점의 실제 날짜로 교체.)*

9. ADR 본문 내부 cross-link 갱신.
   - 보일러플레이트 ADR 끼리의 `[ADR-NNN](ADR-NNN-...)` 링크는 *같은 폴더 안*에서 작동하므로 변경 불필요.
   - 단, 다음 docs 안의 `../90-decisions/ADR-NNN-...` 형식 링크는 `../90-decisions/boilerplate/ADR-NNN-...`로 갱신:
     ```bash
     # 변경 surface 검색
     grep -rn "../90-decisions/ADR-" docs/00-meta/ docs/10-charter/ docs/20-system/ docs/30-workitems/ docs/40-validation/
     ```
   - 각 일치하는 곳을 `../90-decisions/boilerplate/ADR-`로 일괄 변경.

10. `AGENTS.md` 안의 `docs/90-decisions/ADR-...` 링크도 동일 갱신.
    ```bash
    grep -n "docs/90-decisions/ADR-" AGENTS.md
    ```
    각 일치하는 곳을 `docs/90-decisions/boilerplate/ADR-`로 변경.

11. `README.md` / `README_ko.md` 안의 `docs/90-decisions/ADR-...` 링크도 동일 갱신.
    ```bash
    grep -n "docs/90-decisions/ADR-" README.md README_ko.md
    ```
    각 일치 줄에서 `docs/90-decisions/ADR-` → `docs/90-decisions/boilerplate/ADR-`로 변경.

12. `docs/00-meta/STRUCTURE.md` *Canonical Owner 매핑* 표 갱신.
    - **12-A. ADR 경로 행 일괄 갱신** (약 20 행). 예: `문서 계층 정의` 행의 `../90-decisions/ADR-001-doc-hierarchy.md` → `../90-decisions/boilerplate/ADR-001-doc-hierarchy.md`. 동일하게 표의 모든 ADR 경로 행을 `boilerplate/ADR-` 형식으로 갱신.
    - **12-B. `ADR 인덱스` 행을 2 행으로 분리**:
      - **before**: `| ADR 인덱스 | docs/90-decisions/README.md |`
      - **after**:
        ```
        | ADR 인덱스 허브 | docs/90-decisions/README.md |
        | ADR 인덱스 (boilerplate) | docs/90-decisions/boilerplate/README.md |
        ```
      *(Project ADR 인덱스는 fork 사용자가 채우는 영역이라 Canonical Owner 매핑 등록 X — 빈 placeholder.)*

13. STRUCTURE.md 산출물 표의 `ADR` 행 위치 갱신.
    - **before**: `| ADR | docs/90-decisions/ADR-*.md (인덱스: docs/90-decisions/README.md) | architect-opus, /bootstrap-project 등 | Record | baseline |`
    - **after**:
      ```
      | ADR (boilerplate) | docs/90-decisions/boilerplate/ADR-*.md (인덱스: docs/90-decisions/boilerplate/README.md) | 수동 (boilerplate 진화) | Record | baseline |
      | ADR (project) | docs/90-decisions/project/ADR-1NN-*.md (인덱스: docs/90-decisions/project/README.md) | architect, /bootstrap-project 등 | Record | generated |
      ```

14. **2.1에서 박은 ADR-017 SIMULATION_RUN.md 링크 경로 재보정**. ADR-017이 본 단계 2번에서 `boilerplate/`로 이동했으므로 본문 안의 `.boilerplate/...` 링크 상대 경로가 1단계 더 깊어진다.
    - 2.1 step 5에서 박은 *산출물 단락*: `\`.boilerplate/validation/SIMULATION_RUN.md\`` 절대 경로 표기는 그대로 유지 (변경 X — 절대 경로라 ADR 위치 영향 받지 않음).
    - 2.1 step 6에서 박은 *상세 기록 링크*: `[SIMULATION_RUN.md](../../.boilerplate/validation/SIMULATION_RUN.md)` → `[SIMULATION_RUN.md](../../../.boilerplate/validation/SIMULATION_RUN.md)`로 보정 (`..` 1단계 추가).
    - 검증: 새 위치 `docs/90-decisions/boilerplate/ADR-017-dogfood-simulation.md`에서 IDE 링크 hover 시 `.boilerplate/validation/SIMULATION_RUN.md`로 도달.

15. *(`.claude/skills/`, `.claude/agents/`, `.agents/skills/` 본문 안의 ADR 링크 갱신은 8단계 sweep에서 일괄 처리. 본 단계에서는 docs/ 경로만 갱신.)*

**커밋 메시지**:
```
refactor(adr): split 90-decisions into boilerplate/ and project/ with amendment + reserved tracking
```

---

## 3단계 — multi-tool agent 정합성

대상 항목: **I-15, I-19, I-20**.

### 3.1 agent 이름에서 모델 별칭 제거 (I-15)

**목적**: `architect-opus`, `builder-sonnet`, `validator-sonnet`의 모델 별칭을 이름에서 빼고 frontmatter `model:` 필드로만 표기.

**행동지침**:

1. agent 파일 3개 rename.
   ```bash
   git mv .claude/agents/architect-opus.md .claude/agents/architect.md
   git mv .claude/agents/builder-sonnet.md .claude/agents/builder.md
   git mv .claude/agents/validator-sonnet.md .claude/agents/validator.md
   ```

2. 각 파일의 frontmatter `name:` 필드 갱신.
   - `architect.md`의 `name: architect-opus` → `name: architect`
   - `builder.md`의 `name: builder-sonnet` → `name: builder`
   - `validator.md`의 `name: validator-sonnet` → `name: validator`
   - `model: opus|sonnet` 필드는 **그대로 유지**.

3. 모든 SKILL.md의 `agent:` frontmatter 필드 갱신.
   ```bash
   grep -rln "agent: architect-opus\|agent: builder-sonnet\|agent: validator-sonnet" .claude/skills/ .agents/skills/
   ```
   일치하는 모든 파일에서:
   - `agent: architect-opus` → `agent: architect`
   - `agent: builder-sonnet` → `agent: builder`
   - `agent: validator-sonnet` → `agent: validator`

4. docs 본문 안의 agent 이름 참조 갱신.
   ```bash
   grep -rn "architect-opus\|builder-sonnet\|validator-sonnet" docs/ AGENTS.md README.md README_ko.md
   ```
   각 일치하는 줄에서 동일한 3가지 치환. **중요**: 변경 대상은 ADR 본문도 포함.
   - `docs/90-decisions/boilerplate/ADR-004-model-alias-policy.md`
   - `docs/90-decisions/boilerplate/ADR-006-simplicity-and-architecture.md`
   - `docs/90-decisions/boilerplate/ADR-007-workitem-lifecycle.md`
   - `docs/90-decisions/boilerplate/ADR-026-plan-workitem-schema.md`
   - `docs/00-meta/AGENT_EXECUTION_STRATEGY.md` (3.2에서 rename 예정이지만 본 단계에서는 현재 이름으로 작업)
   - 기타.

5. `docs/90-decisions/boilerplate/ADR-004-model-alias-policy.md` 끝부분에 Amendment 1 추가.
   ```markdown
   ## Amendment 1 (YYYY-MM-DD) — agent 이름에서 모델 별칭 제거
   
   ### 결정
   agent 이름은 **역할 중심**(`architect` / `builder` / `validator` / `planner` / `reviewer` / `qa`)으로 한다. 모델 선택은 agent 파일의 frontmatter `model:` 필드에서만 표기한다.
   
   ### 근거
   - Codex 사용자의 의미 혼선 차단 — `$builder-sonnet`이 Codex에서 *어떤 모델*인지 자명하지 않은 문제 해소.
   - 모델 갱신 시 이름 변경 비용 0 — ADR-004 본 정책의 *별칭 자동 갱신* 의도와 정합.
   - ADR-006 단순성 1순위 — 이름은 한 가지 의미만 운반.
   ```

6. **`docs/90-decisions/boilerplate/README.md`** 인덱스 표 (Boilerplate ADR 표)의 ADR-004 행 Amendments 컬럼에 `(+amend1: agent 이름 역할 중심)` 추가. *(허브 `docs/90-decisions/README.md`는 인덱스 표가 없는 링크 페이지라 갱신 X.)*

7. 검증.
   ```bash
   grep -rn "architect-opus\|builder-sonnet\|validator-sonnet" . --include="*.md" --include="*.toml" --include="*.yaml" --include="*.json" --exclude="IMPROVE-*.md"
   ```
   결과 0건이 나와야 한다. *(IMPROVE-*.md는 옛 용어 인용이 정당하므로 exclude.)*

**커밋 메시지**:
```
refactor(agents): drop model alias suffix from agent names (architect-opus → architect)
```

---

### 3.2 AGENT_EXECUTION_STRATEGY.md → DELEGATION_STRATEGY.md + 본문 일반화 (I-19)

**목적**: 파일명·본문에서 Claude 도구별 표현을 *역할 기반*으로 일반화. multi-tool 정합 회복.

**행동지침**:

1. 파일 rename.
   ```bash
   git mv docs/00-meta/AGENT_EXECUTION_STRATEGY.md docs/00-meta/DELEGATION_STRATEGY.md
   ```

2. `docs/00-meta/DELEGATION_STRATEGY.md` 본문에서 다음 표현을 일반화.

   **2-A. `## 병렬 패턴 3종` 표**의 1행 변경.
   - **before**:
     ```
     | 1 | 한 메시지에서 Agent 호출 병렬 | 메인이 한 turn에 Agent 도구를 여러 번 호출. 결과만 통합. | 일상 위임의 기본. 독립적인 짧은 sub-agent 작업 여러 개. |
     ```
   - **after**:
     ```
     | 1 | 한 turn에 독립 sub-agent 다중 호출 | 메인이 한 turn에 sub-agent 도구를 여러 번 호출 (Claude: Agent tool). Codex는 wrapper skill로 같은 본문을 실행하지만 sub-agent / 병렬 위임 parity는 도구별 다름 — [boilerplate/ADR-010](../90-decisions/boilerplate/ADR-010-multi-agent-compatibility.md) 매핑 참조. 결과만 통합. | 일상 위임의 기본. 독립적인 짧은 sub-agent 작업 여러 개. |
     ```

   **2-B. `## 병렬 패턴 3종` 표**의 2행 변경.
   - **before**:
     ```
     | 2 | `isolation: "worktree"` 인자로 단일 Agent 호출 | Agent 도구 호출 시 인자로 격리 git worktree 지정. ...
     ```
   - **after**:
     ```
     | 2 | 격리 git worktree 분기 단일 호출 | sub-agent 호출 시 격리 git worktree 지정 (도구별 지원은 [boilerplate/ADR-010](../90-decisions/boilerplate/ADR-010-multi-agent-compatibility.md) 매핑 표 참조). 변경 없으면 자동 cleanup, 있으면 worktree 경로·브랜치를 결과에 포함.
     ```

   **2-C. `## 병렬 패턴 3종` 표**의 3행 변경.
   - **before**: `bundled \`/batch\` 호출 | Claude Code의 **공식 bundled skill**. ...`
   - **after**: `도구별 bundled batch | Claude Code: 공식 \`/batch\` skill. Codex: 동등 기능 미지원 (자연어 분기). ...`

   **2-D. 마지막 단락의 `/batch` 한정 문구** 갱신.
   - **before**: `\`/batch\`가 본 보일러플레이트의 custom skill이 아니라 Claude Code bundled skill임을 명시한다 ...`
   - **after**: `도구별 bundled batch 지원은 Claude Code의 \`/batch\`가 유일한 1차 출처다 (Codex 동등 기능 도입 시 본 단락 갱신). 도구별 매핑 SSOT는 [boilerplate/ADR-010](../90-decisions/boilerplate/ADR-010-multi-agent-compatibility.md).`

3. **모든** 파일에서 `AGENT_EXECUTION_STRATEGY` 참조를 `DELEGATION_STRATEGY`로 갱신.
   ```bash
   grep -rln "AGENT_EXECUTION_STRATEGY" . --exclude="IMPROVE-*.md"
   ```
   *(IMPROVE-*.md는 옛 용어 인용이 정당하므로 갱신 대상에서 제외.)* 다음 영역이 일치할 가능성 큼:
   - `README.md`, `README_ko.md`
   - `AGENTS.md`
   - `docs/00-meta/STRUCTURE.md` (*Canonical Owner 매핑* 표 1행 + 본문 1~2회)
   - `docs/00-meta/NEW_PROJECT_CHECKLIST.md`
   - `docs/00-meta/WORKFLOW.md`
   - `docs/90-decisions/boilerplate/ADR-005-ssot.md` 등
   - `.claude/skills/*/SKILL.md` 일부 (8단계에서도 sweep)
   
   모든 일치 줄에서 `AGENT_EXECUTION_STRATEGY` → `DELEGATION_STRATEGY` 그리고 `AGENT_EXECUTION_STRATEGY.md` → `DELEGATION_STRATEGY.md`.

4. STRUCTURE.md *Canonical Owner 매핑* 표 1행 갱신.
   - **before**: `| 위임 트리거 + 메인 세션 역할 | docs/00-meta/AGENT_EXECUTION_STRATEGY.md |`
   - **after**: `| 위임 트리거 + 메인 세션 역할 | docs/00-meta/DELEGATION_STRATEGY.md |`

5. 검증.
   ```bash
   grep -rn "AGENT_EXECUTION_STRATEGY" . --include="*.md" --exclude="IMPROVE-*.md"
   ```
   결과 0건이 나와야 한다.

**커밋 메시지**:
```
refactor(docs): rename AGENT_EXECUTION_STRATEGY to DELEGATION_STRATEGY and generalize tool-specific wording
```

---

### 3.3 validator / reviewer / qa 판정 범위 경계 명시 (I-20)

**목적**: 세 agent의 *판정 범위*를 *판정 단위 / 판정 종류 / 책임 제약* 3축으로 명시.

**행동지침**:

1. `docs/00-meta/DELEGATION_STRATEGY.md`(3.2에서 rename된 파일)의 *위임 트리거* 표에서 다음 3 행을 갱신.
   
   - **before**:
     ```
     | 구현 완료 후 범위 검증 | validator | diff 기반 검증 |
     | 문서/코드의 모순·누락·숨은 복잡도 검토 | reviewer | |
     | 구현 후 회귀 위험·엣지 케이스 점검 | qa | |
     ```
   - **after**:
     ```
     | 구현 완료 후 범위 검증 | validator | **단위**: workitem 단위 / **종류**: 판정 + report 전용 / **제약**: 코드·문서 수정 금지 (ADR-007). AC ↔ 테스트 매핑, 문서 범위 정합. |
     | 문서/코드의 모순·누락·숨은 복잡도 검토 | reviewer | **단위**: 코드·문서 단위 / **종류**: 구조적 모순 + 숨은 복잡도 + 정책 drift / **제약**: 수정 권장만, 직접 수정 X. Clean Code 6항목 (ADR-006). |
     | 구현 후 회귀 위험·엣지 케이스 점검 | qa | **단위**: milestone / user-flow 단위 / **종류**: 회귀 + 엣지 케이스 + 사용자 위험 / **제약**: 보고만, Write 권한 없음 (stabilize-milestone이 받아 적음). |
     ```

2. `docs/00-meta/STRUCTURE.md` *Canonical Owner 매핑* 표에 새 행 추가 (`agent 단위 책임 경계` 행).
   ```
   | agent 단위 책임 경계 (validator/reviewer/qa) | docs/00-meta/DELEGATION_STRATEGY.md (위임 트리거 표) |
   ```

3. `docs/90-decisions/boilerplate/ADR-007-workitem-lifecycle.md`에 Amendment 추가 (기존 Amendment 1 다음).
   ```markdown
   ## Amendment 2 (YYYY-MM-DD) — agent 단위 판정 범위 경계 SSOT
   
   ### 결정
   본 ADR의 8단계 lifecycle 표는 *skill 단위* 책임을 정의한다. *agent 단위* 판정 범위 경계는 다음과 같이 별도 SSOT를 둔다.
   
   - **위치**: `docs/00-meta/DELEGATION_STRATEGY.md`의 위임 트리거 표.
   - **형식**: validator / reviewer / qa 세 agent에 *판정 단위 / 판정 종류 / 책임 제약* 3축으로 행 박음.
   
   ### 근거
   - fork 사용자가 *milestone stabilize 시 qa vs reviewer 어느 쪽 호출*을 매번 판단하지 않도록 경계 규칙 명문화.
   - skill 단위 책임(본 ADR)과 agent 단위 경계(DELEGATION_STRATEGY)가 분리 SSOT라 변경 비용 분리.
   ```

4. boilerplate/README.md 인덱스의 ADR-007 행 Amendments 컬럼 갱신.
   - **before**: `(+amend1: lock file whitelist 11종)`
   - **after**: `(+amend1: lock file whitelist 11종, +amend2: agent 단위 판정 경계 SSOT)`

**커밋 메시지**:
```
docs(delegation): clarify validator/reviewer/qa scope boundaries (unit, judgment, constraint)
```

---

## 4단계 — 조건부 산출물 실행 경로

대상 항목: **I-05** + **I-13 일부** (`/bootstrap-design`, `/bootstrap-stack`).

### 4.1 ARCHITECTURE 7-1~7-4 자동 삭제 책임 명시 (I-05)

**행동지침**:

1. `docs/20-system/ARCHITECTURE_OVERVIEW.md`의 7-1 sub-section 주석을 갱신.
   - **before**:
     ```markdown
     ## 7-1. API 컨벤션
     <!-- API 스택일 때만 채운다. /bootstrap-stack이 architect 단발 호출로 채울 수 있다.
          비-API 프로젝트는 통째 삭제. -->
     ```
   - **after**:
     ```markdown
     ## 7-1. API 컨벤션
     <!-- API 스택일 때만 채운다. /bootstrap-stack이 architect 단발 호출로 채운다.
          **비-API 프로젝트는 스택 확정 시 /bootstrap-stack이 본 sub-section을 통째 삭제.** -->
     ```

2. 동일 패턴으로 7-2 (CLI), 7-3 (백엔드), 7-4 (프론트) sub-section 주석도 갱신. 각각 *"비-CLI 프로젝트는 스택 확정 시 /bootstrap-stack이 본 sub-section을 통째 삭제"*, *"비-백엔드..."*, *"비-프론트..."* 패턴.

3. `.claude/skills/bootstrap-stack/SKILL.md` "반드시 수행할 일" 단락에 1행 추가.
   ```markdown
   - 스택 확정 시 ARCHITECTURE_OVERVIEW.md의 비해당 7-1~7-4 sub-section을 통째 삭제 (예: API 미포함 프로젝트는 `## 7-1` sub-section 삭제).
   ```

4. `.claude/skills/stabilize-milestone/SKILL.md` 본문 step 6 부근에 1행 추가.
   ```markdown
   - ARCHITECTURE_OVERVIEW.md에 비해당 7-x sub-section이 *잔존*하면 IMPROVEMENT_GUIDE.md에 P2 보고 — *"조건부 sub-section 미삭제. /bootstrap-stack 재실행 또는 수동 삭제 권장."*
   ```

**커밋 메시지**:
```
docs(architecture): make conditional 7-x section deletion an explicit /bootstrap-stack responsibility
```

---

### 4.2 /bootstrap-design baseline 흐름 정합 (I-13 일부)

**행동지침**:

1. `.claude/skills/bootstrap-design/SKILL.md` 본문의 "반드시 수행할 일" 단락에 다음 2 라인 추가.
   ```markdown
   - 본 skill은 baseline placeholder `docs/20-system/DESIGN.md`를 *채운다* (생성 X). 파일이 없으면 fork 사용자가 비-UI 프로젝트로 판단해 삭제한 경우 — 작업 중단 + 사용자에게 *"본 프로젝트는 비-UI라 판단됨. /bootstrap-design 실행 의도 확인 필요"* 보고.
   - DESIGN.md 본문 상단 주석(`baseline placeholder`)을 변경하지 않는다 — 정책 SSOT는 STRUCTURE.md presence 컬럼 + 본 파일 주석.
   ```

**커밋 메시지**:
```
docs(skills): align bootstrap-design SKILL with DESIGN.md baseline placeholder policy
```

---

## 5단계 — 검증 / 누적 기록

대상 항목: **I-06, I-12** + **I-13 일부** (`/stabilize-milestone`, `/review-doc`).

### 5.1 QA / IMPROVEMENT 항목 스키마 정의 (I-06)

**행동지침**:

1. `docs/40-validation/QA_FINDINGS.md` 상단에 *항목 스키마* 단락 신설 (`## 다운스트림 마이그레이션 가이드` 위).
   ```markdown
   ## 항목 스키마
   
   각 발견 항목은 다음 형식으로 박는다.
   
   - 필수 4필드: `ID | severity | evidence label | linked workitem`
   - 권장 2필드: `status | decision`
   - evidence label은 [boilerplate/ADR-022](../90-decisions/boilerplate/ADR-022-ratchet-principle.md)의 `[관측됨]` / `[외부실증]` / `[가설]` (+ 합성 표기) 중 1개.
   
   예시:
   ​```
   - **F-M1-001** | P1 | [관측됨] | linked: T-002 | status: open
     - 발견: FAC-4 → T-002:AC-N 매핑 누락, validate 통과인데 spec gap.
     - 결정: 다음 라운드 plan에서 T-002에 AC-3 추가.
   ​```
   ```

2. `docs/40-validation/IMPROVEMENT_GUIDE.md` 상단에도 동일 스키마 단락 박음 (`## 0. 요약` 위).

**커밋 메시지**:
```
docs(validation): define entry schema for QA_FINDINGS and IMPROVEMENT_GUIDE
```

---

### 5.2 /stabilize-milestone에 DISCOVERY ↔ Charter staleness 보고 step 추가 (I-12)

**행동지침**:

1. `.claude/skills/stabilize-milestone/SKILL.md`의 step 6과 step 7 사이에 새 sub-step 추가.
   ```markdown
   ### 6.5. DISCOVERY ↔ Charter staleness 감지 (ADR-035 amend 1)
   
   다음 3 시그널을 점검한다 (보고만, 자동 차단 X — validator 책임 경계 정합).
   
   1. `docs/10-charter/DISCOVERY.md`의 mtime이 `docs/10-charter/PROJECT_CHARTER.md`의 mtime보다 최신인지.
   2. DISCOVERY.md `## 12. Assumption Tracker` 표에서 *"미검증"* 결과 항목 수.
   3. PROJECT_CHARTER.md `## 2.1 페르소나` / `## 3.1 핵심 시나리오` / `## 9 핵심 가정` 섹션 중 비어 있거나 DISCOVERY.md와 명백히 어긋난 섹션 수.
   
   위 3 시그널 중 1개라도 *stale 의심* 판정 시 IMPROVEMENT_GUIDE.md에 P1 보고:
   *"DISCOVERY ↔ Charter drift 의심 — /bootstrap-project --apply 또는 수동 갱신 권장."*
   ```

2. `docs/90-decisions/boilerplate/ADR-035-continuous-discovery.md` 끝부분에 Amendment 1 추가.
   ```markdown
   ## Amendment 1 (YYYY-MM-DD) — Charter 본문 staleness 보고 흡수
   
   ### 결정
   ADR-035 *잔여 모니터링*의 *"assumption tracker 빈 결과율 보고"*를 다음 3 시그널로 확장한다.
   
   - DISCOVERY.md mtime > PROJECT_CHARTER.md mtime
   - Assumption Tracker 미검증 항목 수
   - PROJECT_CHARTER `## 2.1 / 3.1 / 9` 섹션 stale
   
   `/stabilize-milestone` step 6.5에서 점검 + IMPROVEMENT_GUIDE.md에 P1 보고.
   
   ### 근거
   mid-project pivot 시 DISCOVERY만 갱신하고 Charter는 그대로일 경우 SSOT silent divergence 차단.
   ```

3. boilerplate/README.md 인덱스 ADR-035 행 Amendments 컬럼 갱신: `(+amend1: Charter staleness 보고)` *(이미 2단계 6-B에서 박았으면 확인만)*.

**커밋 메시지**:
```
feat(stabilize): detect DISCOVERY-Charter staleness and report to IMPROVEMENT_GUIDE
```

---

### 5.3 /review-doc skill에 ADR 메타 점검 반영 (I-13 일부)

**행동지침**:

1. `.claude/skills/review-doc/SKILL.md` 본문의 "반드시 수행할 일" 단락에 다음 4 점검 항목 추가.
   ```markdown
   - boilerplate/README.md의 *Reserved / Parked / Dropped 번호* 표가 git log의 실제 누락 번호와 일치하는지 점검. 새 dropped 번호 발견 시 P2 보고.
   - boilerplate/README.md ADR 표의 *Amendments* 컬럼이 각 ADR 본문의 실제 `## Amendment N` 단락과 일치하는지 점검. 누락 발견 시 P1 보고.
   - AGENTS.md가 100줄 hard cap을 준수하는지 (ADR-011) — 초과 시 P0, 80~100줄 사이는 P1.
   - `docs/00-meta/` 파일 수가 ADR-012의 *6개* 원칙과 일치하는지 (`_templates/`는 카운트 제외). 위반 시 P0 보고.
   ```

**커밋 메시지**:
```
docs(skills): extend review-doc to audit ADR amendment visibility and reserved/parked tracking
```

---

## 6단계 — 진입 문서 / 사용자 동선

대상 항목: **I-17, I-10, I-11, I-09, I-01**.

### 6.1 NEW_PROJECT_CHECKLIST → PROJECT_START_CHECKLIST rename (I-17)

**행동지침**:

1. 파일 rename.
   ```bash
   git mv docs/00-meta/NEW_PROJECT_CHECKLIST.md docs/00-meta/PROJECT_START_CHECKLIST.md
   ```

2. 모든 참조 갱신.
   ```bash
   grep -rln "NEW_PROJECT_CHECKLIST" . --exclude="IMPROVE-*.md"
   ```
   *(IMPROVE-*.md는 옛 용어 인용이 정당하므로 갱신 대상에서 제외.)* 다음 영역에서 일치 가능:
   - `README.md`, `README_ko.md`
   - `docs/00-meta/STRUCTURE.md` (*Canonical Owner 매핑* 표)
   - `docs/00-meta/WORKFLOW.md`
   - `docs/00-meta/DELEGATION_STRATEGY.md` (3단계에서 rename됨)
   - `.claude/skills/bootstrap-project/SKILL.md`
   - `.claude/skills/bootstrap-stack/SKILL.md`
   
   모든 일치 줄에서 `NEW_PROJECT_CHECKLIST` → `PROJECT_START_CHECKLIST`.

3. `docs/00-meta/PROJECT_START_CHECKLIST.md`(방금 rename된 파일) 본문 처리 — 다음 2 규칙으로 통일.
   - 본문 안의 *"새 프로젝트 시작"* 표현은 그대로 둔다 (*의미가 자연스러움 — "프로젝트가 새로 시작됨"이라는 사건*).
   - 첫 줄 모드 라벨 *"모드: How-to (새 프로젝트 시작 체크리스트)"*도 그대로 유지. 파일명만 `PROJECT_START_CHECKLIST.md`로 바뀌고, *문서 성격*(*"새 프로젝트 시작 체크리스트"*)은 변하지 않는다.

4. STRUCTURE.md *Canonical Owner 매핑* 표의 다음 2행 갱신.
   - **before**: `| 새 프로젝트 시작 절차 (체크리스트) | docs/00-meta/NEW_PROJECT_CHECKLIST.md |`
   - **after**: `| 새 프로젝트 시작 절차 (체크리스트) | docs/00-meta/PROJECT_START_CHECKLIST.md |`
   - **before**: `| Bootstrap 입력 예시 | docs/00-meta/NEW_PROJECT_CHECKLIST.md (1단계 예시 흡수) |`
   - **after**: `| Bootstrap 입력 예시 | docs/00-meta/PROJECT_START_CHECKLIST.md (1단계 예시 흡수) |`

5. 검증.
   ```bash
   grep -rn "NEW_PROJECT_CHECKLIST" . --include="*.md" --exclude="IMPROVE-*.md"
   ```
   결과 0건.

**커밋 메시지**:
```
refactor(docs): rename NEW_PROJECT_CHECKLIST to PROJECT_START_CHECKLIST
```

---

### 6.2 PROJECT_START_CHECKLIST에 /plan-workitem + Status 전환 체크 추가 (I-10)

**행동지침**:

1. `docs/00-meta/PROJECT_START_CHECKLIST.md`의 `## 2. 작업 구조 준비` 단락 첫 줄 직전에 다음 2 체크 항목 추가.
   ````markdown
   ## 2. 작업 구조 준비
   - [ ] `/plan-workitem [milestone-id]`를 실행해 milestone/feature/task 문서를 분해했다
     ```
     /plan-workitem M1
     ```
   - [ ] bootstrap 후 PROJECT_CHARTER.md / ARCHITECTURE_OVERVIEW.md / M1 / F-001의 `## 0. Status`를 `draft → ready`로 전환했다
   - [ ] `docs/30-workitems/milestones`에 첫 milestone 문서가 있다
   - [ ] `docs/30-workitems/features`에 첫 feature 문서가 있다
   - [ ] 필요하면 `docs/30-workitems/tasks`에 task 문서를 만들었다
   ````
   *(기존 3 체크 항목은 그대로 유지, 위 2 체크가 가장 위로.)*

**커밋 메시지**:
```
docs(checklist): add /plan-workitem command box and Status transition check
```

---

### 6.3 WORKFLOW.md에 4-1 작성 타이밍 + 문서 선갱신 예외 단락 추가 (I-11)

**행동지침**:

1. `docs/00-meta/WORKFLOW.md` `## 4. 구현 및 검증` 단락 마지막에 1행 추가.
   ```markdown
   - task `## 4-1. 변경 예정 파일/경로`는 implement 중 채운다 — plan 단계에서 미리 채울 의무 없음.
   ```

2. `## 4-1. 마감 (finalize)` 단락 위에 새 단락 `## 4-A. 문서 선갱신 예외` 추가.
   ```markdown
   ## 4-A. 문서 선갱신 예외
   
   AGENTS.md의 *"상위 문서 없이 하위 문서를 먼저 만들지 않는다"* 규칙은 다음 케이스에서 면제한다.
   
   1. 보안 hotfix
   2. 단순 typo / 오타 수정
   3. 명시적으로 비목표(charter `## 5`)에 박힌 영역의 긴급 패치
   
   면제 적용 시 `/finalize-workitem` 단계에서 IMPROVEMENT_GUIDE.md에 *"상위 문서 후행 갱신 필요"* P2 보고 — 다음 stabilize 라운드에서 상위 문서 sync 추적.
   ```

**커밋 메시지**:
```
docs(workflow): document 4-1 timing and document-precedence exceptions
```

---

### 6.4 README "Same workflow applies" 표현 정밀화 (I-09)

**행동지침**:

1. `README.md` "Using with Codex CLI" 단락에서 "Same workflow applies" 부분 갱신.
   - **before**:
     ```markdown
     2. Same workflow applies: see [WORKFLOW.md](docs/00-meta/WORKFLOW.md).
     ```
   - **after**:
     ```markdown
     2. Documents and policies are equal. Core workflow skills have Codex wrappers ($-prefixed): $implement-workitem, $validate-workitem, $repair-workitem, $finalize-workitem, $plan-workitem, $bootstrap-project, $bootstrap-stack, $stabilize-milestone. Remaining skills (discover-product, stack-guard, review-doc, boilerplate-context, bootstrap-design) are invoked via natural language. See [WORKFLOW.md](docs/00-meta/WORKFLOW.md).
     ```

2. `README_ko.md`의 동일 단락도 같은 의미로 갱신 (한국어).
   - **before**:
     ```markdown
     2. 워크플로우는 동일: [WORKFLOW.md](docs/00-meta/WORKFLOW.md) 참조.
     ```
   - **after**:
     ```markdown
     2. 문서와 정책은 동일. 핵심 workflow skill은 Codex wrapper ($-prefixed)로 제공: $implement-workitem, $validate-workitem, $repair-workitem, $finalize-workitem, $plan-workitem, $bootstrap-project, $bootstrap-stack, $stabilize-milestone. 나머지 skill (discover-product, stack-guard, review-doc, boilerplate-context, bootstrap-design)은 자연어로 호출. 자세한 워크플로우는 [WORKFLOW.md](docs/00-meta/WORKFLOW.md) 참조.
     ```

**커밋 메시지**:
```
docs(readme): clarify Codex compatibility as core-wrappers-plus-natural-language-fallback
```

---

### 6.5 README_ko.md "디자인 시스템" → "UI 디자인" (I-01)

**행동지침**:

1. `README_ko.md`의 디렉터리 트리 단락에서 line 115 부근 1행 갱신.
   - **before**:
     ```
     │   ├── 20-system/     # 아키텍처 개요, 디자인 시스템
     ```
   - **after**:
     ```
     │   ├── 20-system/     # 아키텍처 개요, UI 디자인 (UI 프로젝트 한정)
     ```

**커밋 메시지**:
```
docs(readme-ko): align design wording with ADR-027 (UI design, UI-only)
```

---

## 7단계 — ADR 메타 추적성 완성 검증

대상 항목: **I-07, I-08** (실제 박는 작업은 2단계 6-B / 6-C에서 완료. 본 단계는 *완성 검증*만).

### 7.1 boilerplate/README.md 완성 검증

**행동지침**:

1. `docs/90-decisions/boilerplate/README.md`를 열고 다음 3가지 확인.
   - [ ] ADR 표에 `Amendments` 컬럼이 있고, amend된 **9개 ADR**(000 / **004** / 007 / 008 / 009 / 017 / 021 / 026 / 035)에 값이 박혀 있다. ADR-004 / ADR-007 amend2 / ADR-035는 3·5단계에서 박힌 후 인덱스 행도 동시 갱신됐어야 한다.
   - [ ] `## Reserved / Parked / Dropped 번호` 섹션이 있고, 13개 번호(002, 003, 013, 015, 016, 018, 023, 028, 029, 030, 032, 033, 034)가 모두 등록되어 있다. ADR-002 / ADR-003은 `legacy reserved` 상태로 표시.
   - [ ] _ADR_GUIDE.md *상태값*에 `reserved` / `parked`가 추가되어 있다.

2. `docs/90-decisions/project/README.md`를 열고 다음 확인.
   - [ ] *"ADR-100부터 시작"* 안내 단락이 있다.
   - [ ] 빈 ADR 목록 표가 있다.

3. `docs/90-decisions/README.md` (허브)를 열고 다음 확인.
   - [ ] *Boilerplate ADR Index* 링크가 `boilerplate/README.md`로 작동.
   - [ ] *Project ADR Index* 링크가 `project/README.md`로 작동.
   - [ ] *"새 ADR을 어디 박는가"* 단락이 있다.

4. *(만약 위 어느 항목이라도 빠졌으면 2단계 6번/2단계 4번/2단계 5번으로 돌아가 채운다.)*

**커밋 메시지**:
```
docs(adr): verify amendment column and reserved/parked tracking completeness
```

*(2단계에서 모두 박혔으면 본 단계는 빈 커밋 — `git commit --allow-empty -m "..."` 또는 skip.)*

---

## 8단계 — skill / agent 본문 일관성 sweep (I-13 마지막)

**목적**: 위 7단계 모든 변경 후 skill / agent 본문이 *문서 정책과 정합*하는지 마지막 sweep.

### 8.1 ADR 경로 일괄 갱신

**행동지침**:

1. `.claude/` 와 `.agents/` 안의 모든 SKILL.md / agent .md에서 ADR 경로를 새 `boilerplate/` 경로로 갱신.
   ```bash
   grep -rln "docs/90-decisions/ADR-" .claude/ .agents/
   ```
   각 일치 파일에서 모든 `docs/90-decisions/ADR-` → `docs/90-decisions/boilerplate/ADR-`로 변경.
   
   상대 경로 케이스도 일괄 처리 — `.claude/skills/<name>/SKILL.md`는 `../../../docs/...` 패턴도 사용한다:
   ```bash
   grep -rln "\.\./\.\./\.\./docs/90-decisions/ADR-\|\.\./\.\./docs/90-decisions/ADR-\|\.\./90-decisions/ADR-" .claude/ .agents/
   ```
   각 일치 파일에서 다음 3 패턴을 *모두* 갱신:
   - `../../../docs/90-decisions/ADR-` → `../../../docs/90-decisions/boilerplate/ADR-`
   - `../../docs/90-decisions/ADR-` → `../../docs/90-decisions/boilerplate/ADR-`
   - `../90-decisions/ADR-` → `../90-decisions/boilerplate/ADR-`

2. 검증.
   ```bash
   grep -rn "90-decisions/ADR-" .claude/ .agents/ docs/ --exclude="IMPROVE-*.md"
   ```
   모든 일치 줄이 `boilerplate/` 또는 `project/` sub-folder를 포함해야 한다. *(IMPROVE-LIST.md / IMPROVE-GUIDE.md는 옛 경로 인용이 정당하므로 exclude.)*

### 8.2 최종 grep 검증

다음 명령을 모두 실행해서 0건이 나와야 한다. **IMPROVE-LIST.md / IMPROVE-GUIDE.md는 옛 용어를 인용하는 *과거 분석·가이드* 문서이므로 `--exclude="IMPROVE-*.md"`로 sweep 대상에서 제외**.

```bash
# 1. agent 이름 모델 별칭이 남아 있는지
grep -rn "architect-opus\|builder-sonnet\|validator-sonnet" . --include="*.md" --include="*.toml" --include="*.yaml" --include="*.json" --exclude="IMPROVE-*.md"

# 2. 옛 파일명 참조가 남아 있는지
grep -rn "AGENT_EXECUTION_STRATEGY\|NEW_PROJECT_CHECKLIST" . --include="*.md" --exclude="IMPROVE-*.md"

# 3. SIMULATION_RUN.md 옛 경로 참조가 남아 있는지
grep -rn "docs/40-validation/SIMULATION_RUN" . --include="*.md" --exclude="IMPROVE-*.md"

# 4. STACK_SETUP_PLAN.md baseline 참조가 남아 있는지 (template/generated 분리 후)
grep -rn "docs/00-meta/STACK_SETUP_PLAN.md" . --include="*.md" --exclude="IMPROVE-*.md"
# 결과: skill 본문(/bootstrap-stack 등)에서 *생성 대상 경로*로만 등장해야 함. 다른 곳은 _templates/STACK_SETUP_PLAN_TEMPLATE.md 참조여야.
```

### 8.3 AGENTS.md 100줄 cap 점검

```bash
wc -l AGENTS.md
```
결과가 100 이하여야 한다. 초과 시 — 본 가이드 진행 중 새 내용 추가했다면 ADR로 옮긴다 (ADR-011 정합).

### 8.4 STRUCTURE.md Canonical Owner 매핑 표 최종 점검

`docs/00-meta/STRUCTURE.md` Canonical Owner 매핑 표를 열고 다음 행이 모두 갱신되었는지 확인.

- [ ] `위임 트리거 + 메인 세션 역할` → `docs/00-meta/DELEGATION_STRATEGY.md`
- [ ] `새 프로젝트 시작 절차 (체크리스트)` → `docs/00-meta/PROJECT_START_CHECKLIST.md`
- [ ] `Bootstrap 입력 예시` → `docs/00-meta/PROJECT_START_CHECKLIST.md`
- [ ] `ADR 인덱스` → `docs/90-decisions/README.md` (허브) + `docs/90-decisions/boilerplate/README.md` (실제 보일러플레이트 인덱스) 2 행
- [ ] `agent 단위 책임 경계 (validator/reviewer/qa)` → `docs/00-meta/DELEGATION_STRATEGY.md` (신규 행)
- [ ] 모든 ADR 참조 경로가 `boilerplate/ADR-NNN-...` 형식

**커밋 메시지**:
```
chore(skills): sweep skill/agent bodies for consistency with new structure
```

---

## 부록 A — 작업 중 막혔을 때

- **ADR 본문 cross-link이 동시에 너무 많은 곳을 가리킴**: 한 ADR씩 처리. 2단계 9번 `grep` 결과를 한 파일씩 갱신.
- **AGENTS.md cap 초과 위험**: ADR-011 정책 — 새 정책은 ADR로 박고 AGENTS.md에는 1줄 + 링크만.
- **README.md / README_ko.md cross-language drift**: 한 변경마다 *영문 → 한국어* 순서로 박고 즉시 비교.
- **STRUCTURE.md presence 컬럼 값 결정 모호**: 4 값 정의를 다시 본다 (1.3 step 2). 못 정하면 *baseline*으로 임시 박고 `<!-- TODO: presence 재검토 -->` 주석.
- **dropped ADR 번호의 정확한 사유**: 2단계 6-C의 *"Phase 진화 중 fold됨"* 일반 표현으로 충분. git log 정밀 조회는 별도 후속 작업.

---

## 부록 B — PowerShell 대안

본문은 Bash / Git Bash / WSL 기준. Windows PowerShell 사용자는 다음 등가 명령을 사용한다.

### 디렉터리 생성

| Bash | PowerShell |
|------|-----------|
| `mkdir -p docs/00-meta/_templates` | `New-Item -ItemType Directory -Force -Path docs/00-meta/_templates` |
| `mkdir -p .boilerplate/validation` | `New-Item -ItemType Directory -Force -Path .boilerplate/validation` |
| `mkdir -p docs/90-decisions/boilerplate` | `New-Item -ItemType Directory -Force -Path docs/90-decisions/boilerplate` |

### 파일 이동 / rename

`git mv`는 PowerShell에서도 동일하게 작동한다 (git이 OS 중립). 그대로 사용 가능.

### 패턴 검색 (grep 대안)

PowerShell 기본 `Select-String`보다 [ripgrep (`rg`)](https://github.com/BurntSushi/ripgrep)을 권장 — 더 빠르고 grep과 syntax 유사.

| Bash | ripgrep | PowerShell `Select-String` |
|------|---------|---------------------------|
| `grep -rn "pattern" . --include="*.md" --exclude="IMPROVE-*.md"` | `rg "pattern" -g "*.md" -g "!IMPROVE-*.md"` | `Get-ChildItem -Recurse -Filter "*.md" -Exclude "IMPROVE-*.md" \| Select-String "pattern"` |
| `grep -rln "pattern" .claude/ .agents/` | `rg -l "pattern" .claude/ .agents/` | `Get-ChildItem -Recurse -Path .claude/,.agents/ -Filter "*.md" \| Select-String "pattern" \| Select-Object -Unique Path` |

### 줄 수 확인

| Bash | PowerShell |
|------|-----------|
| `wc -l AGENTS.md` | `(Get-Content AGENTS.md \| Measure-Object -Line).Lines` |

### 파일 수 확인

| Bash | PowerShell |
|------|-----------|
| `ls docs/00-meta/ \| grep -v _templates \| wc -l` | `(Get-ChildItem docs/00-meta/ -Exclude _templates).Count` |

### 권장

대량 grep / sweep 작업이 많으므로 **ripgrep 설치를 권장**한다. `winget install BurntSushi.ripgrep.MSVC` 또는 `scoop install ripgrep`.
