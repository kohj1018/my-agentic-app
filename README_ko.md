<!-- 구조 변경 시 README.md와 README_ko.md를 동시에 갱신한다. drift를 막기 위해 두 README 본문은 짧게 유지하고 깊은 정의는 docs/ 링크로 둔다. -->
# Claude Code Agentic Boilerplate

**Language: [English](README.md) | 한국어**

새 프로젝트를 Claude Code로 시작할 때, 문서 구조와 서브에이전트 워크플로우를 한 번에 세팅하는 보일러플레이트다.

> **한 줄 요약**: 이 저장소를 fork → 필요하면 `/discover-product`로 사용자 데이터 기반 발굴 → `/bootstrap-project` 실행 → charter·architecture·초기 workitem을 한 번에 생성. 메인 세션은 오케스트레이션, 실작업은 서브에이전트가 수행한다.

## 이런 경우에 적합하다

- 새 프로젝트를 자주 시작하는 개인 개발자
- 문서 구조와 작업 분해를 표준화하고 싶은 팀
- 메인 세션이 모든 컨텍스트를 떠안지 않고, 서브에이전트 위임 방식으로 운영하고 싶은 사용자

## 전체 흐름

```
/discover-product (선택)
  → /bootstrap-project → /bootstrap-stack → /stack-guard
  → /plan-workitem → /implement-workitem
  → /validate-workitem → /repair-workitem (Needs Fix 시) → /finalize-workitem
  → /stabilize-milestone
```

각 단계 상세는 [WORKFLOW.md](docs/00-meta/WORKFLOW.md), 서브에이전트 위임은 [AGENT_EXECUTION_STRATEGY.md](docs/00-meta/AGENT_EXECUTION_STRATEGY.md)를 참조한다.
아래 빠른 시작에서 이 명령들을 0~3단계로 따라갈 수 있다.

새 프로젝트는 `/discover-product`로 페르소나·pain·시나리오를 먼저 발굴해 charter의 신뢰도를 높이는 것을 권장한다. 발굴 결과는 `docs/10-charter/DISCOVERY.md`에 저장되고, `/bootstrap-project`가 이를 charter/architecture/초기 workitem으로 변환한다. 빠른 prototype에서는 `/discover-product`를 건너뛰고 `/bootstrap-project`에 자연어 설명을 바로 줘도 된다.

## 빠른 시작

### 전제 조건

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)가 설치되어 있어야 한다
- 이 저장소를 GitHub 템플릿으로 새 저장소에 적용하거나, 복제해서 시작한다

### 0단계 (선택): 발굴

페르소나·pain 데이터를 기반으로 charter 신뢰도를 높이려면:

```text
/discover-product [제품 설명]
```

빠른 prototype이라면 이 단계를 건너뛰고 `/bootstrap-project`에 brief를 바로 넘기면 된다.

### 1단계: 프로젝트 초기화

```text
/bootstrap-project [프로젝트 설명 또는 비워두면 DISCOVERY.md 사용]
```

생성 결과: `README.md`, `docs/10-charter/PROJECT_CHARTER.md`, `docs/20-system/ARCHITECTURE_OVERVIEW.md`, 초기 milestone/feature 문서.

**팁 — brief에 포함하면 좋은 것**: 무엇을 만드는지, 누가 쓰는지, 어떤 문제를 푸는지, 확정된 것과 미정인 것. 자세한 예시는 [BOOTSTRAP_PROMPT_EXAMPLES.md](docs/00-meta/BOOTSTRAP_PROMPT_EXAMPLES.md)를 참고한다.

### 2단계: 스택 세팅

스택이 정해지면:

```text
/bootstrap-stack [스택/런타임 설명]
/stack-guard
```

`/bootstrap-stack`은 스택 선택을 문서화하고 필요한 자동화 방향을 정리한다. `STACK_SETUP_PLAN.md`를 검토한 뒤 `/stack-guard`를 실행하면 통합 `validate` 진입점과 verify 스크립트가 생성된다.

### 3단계: 분해 → 구현 → 마감

```text
# 분해 + 구현
/plan-workitem [milestone 또는 feature id]
/implement-workitem [task id]
/validate-workitem [task id]

# Pass 시: 마감하고 다음으로
/finalize-workitem [task id]

# Needs Fix 시: 수정 후 재검증
/repair-workitem [task id]
/validate-workitem [task id]

# milestone의 모든 task가 done이 되면:
/stabilize-milestone [milestone id]
```

## 구조

산출물 전체 인벤토리(위치·생성 주체·라이프사이클)는 [STRUCTURE.md](docs/00-meta/STRUCTURE.md)를 참조한다.

```
.
├── CLAUDE.md          # 프로젝트 공통 지침
├── .claude/           # 서브에이전트, 스킬, 설정
├── docs/
│   ├── 00-meta/       # 워크플로우, guardrail, 템플릿, 운영 가이드
│   ├── 10-charter/    # 프로젝트 범위, 목표, 문제 정의
│   ├── 20-system/     # 아키텍처 개요, 디자인 시스템
│   ├── 30-workitems/  # milestone, feature, task
│   ├── 40-validation/ # QA 결과, 개선 가이드, reports
│   └── 90-decisions/  # ADR 기록
└── scripts/           # 프로젝트별 자동화 (스택 확정 후 추가)
```

## Guardrail 원칙

이 템플릿은 cross-platform 재사용성을 우선한다 — shared 기본값에 OS/셸/런타임 종속적인 hook를 포함하지 않는다. 자세한 내용은 [GUARDRAILS_STRATEGY.md](docs/00-meta/GUARDRAILS_STRATEGY.md)를 참고한다.

## 처음 시작할 때 먼저 볼 문서

- [NEW_PROJECT_CHECKLIST.md](docs/00-meta/NEW_PROJECT_CHECKLIST.md) — 새 프로젝트 시작 체크리스트
- [TEMPLATE_GUIDE.md](docs/00-meta/TEMPLATE_GUIDE.md) — 문서 구조와 네이밍 규칙
- [WORKFLOW.md](docs/00-meta/WORKFLOW.md) — 단계별 워크플로우
- [BOOTSTRAP_PROMPT_EXAMPLES.md](docs/00-meta/BOOTSTRAP_PROMPT_EXAMPLES.md) — 입력 예시

## Contributing

개선 제안이나 버그 제보는 [이슈 템플릿](.github/ISSUE_TEMPLATE)을, 구조 변경은 [PR 템플릿](.github/PULL_REQUEST_TEMPLATE)을 참고한다.

## License

MIT
