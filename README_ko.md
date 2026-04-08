# Claude Code Agentic Boilerplate

**Language: [English](README.md) | 한국어**

새 프로젝트를 Claude Code로 시작할 때, 문서 구조와 서브에이전트 워크플로우를 한 번에 세팅하는 보일러플레이트다.

범위 정의 → 시스템 설계 → 작업 분해 → 구현 → 검증 흐름을 일관된 구조로 반복할 수 있게 해준다.
메인 세션은 오케스트레이션에 집중하고, 실작업은 서브에이전트와 스킬에 위임하는 방식을 기본으로 둔다.

## 이런 경우에 적합하다

- 새 프로젝트를 자주 시작하는 개인 개발자
- 문서 구조와 작업 분해를 표준화하고 싶은 팀
- 메인 세션이 모든 컨텍스트를 떠안지 않고, 서브에이전트 위임 방식으로 운영하고 싶은 사용자

## 빠른 시작

### 전제 조건

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)가 설치되어 있어야 한다
- 이 저장소를 GitHub 템플릿으로 새 저장소에 적용하거나, 복제해서 시작한다

### 1단계: 프로젝트 초기화

새 저장소에서 Claude Code를 열고 아래를 실행한다.

```text
/bootstrap-project [프로젝트 설명]
```

예시:

```text
/bootstrap-project 개인 커리어 관리 SaaS. 사용자는 JD와 이력서를 비교하고, 부족한 역량을 추적하고, 주간 액션 플랜을 관리한다. 초기 타깃은 취준생. 아직 스택은 미정.
```

이 명령은 아래 문서들의 초안을 자동으로 정리하고, 프로젝트 시작 구조를 만들어준다.

- `README.md`
- `docs/10-charter/PROJECT_CHARTER.md`
- `docs/20-system/ARCHITECTURE_OVERVIEW.md`
- 초기 milestone / feature 문서

### 2단계: 스택 확정 후 세팅

스택이 정해지면 아래를 실행한다.

```text
/bootstrap-stack [스택/런타임 설명]
```

예시:

```text
/bootstrap-stack Next.js 16 + TypeScript + pnpm + Supabase + Playwright + Vercel
```

스택 선택을 문서화하고, 필요한 자동화와 guardrail 방향을 정리한다.

## 전체 흐름

```
/bootstrap-project → /bootstrap-stack → /plan-workitem → /implement-workitem → /validate-workitem → qa/reviewer
```

각 단계의 상세 운영 방식은 아래 문서를 참고한다.

- [워크플로우](docs/00-meta/WORKFLOW.md)
- [에이전트 실행 전략](docs/00-meta/AGENT_EXECUTION_STRATEGY.md)

## 구현과 검증

workitem 문서를 만든 뒤, 구현은 `/implement-workitem`으로 시작한다.
구현 후 범위 일치와 검증은 `/validate-workitem`으로 확인한다.
필요하면 `qa`나 `reviewer` 서브에이전트를 추가로 사용한다.

```text
/implement-workitem T-001-auth-session
/validate-workitem T-001-auth-session
```

## 구조

```
.
├── CLAUDE.md                          # 프로젝트 공통 지침
├── .claude/
│   ├── settings.json                  # 공유 프로젝트 설정
│   ├── agents/                        # 역할별 서브에이전트
│   └── skills/                        # 반복 가능한 작업 절차 (slash commands)
├── docs/
│   ├── 00-meta/                       # 템플릿 사용법, 워크플로우, 운영 원칙
│   ├── 10-charter/                    # 프로젝트 범위, 목표, 문제 정의
│   ├── 20-system/                     # 시스템 구조, 설계 개요
│   ├── 30-workitems/                  # milestone / feature / task / plans
│   ├── 40-validation/                 # QA 결과, 개선 가이드
│   └── 90-decisions/                  # ADR 기록
└── scripts/                           # 프로젝트별 자동화 스크립트 (스택 확정 후 추가)
```

## Guardrail 원칙

이 템플릿은 cross-platform 재사용성을 우선한다.
팀 전체가 공유하는 기본 설정에는 OS/셸/런타임 종속적인 hook와 검증 스크립트를 넣지 않는다.
자동화는 프로젝트의 실제 스택이 정해진 뒤 추가한다.

자세한 내용은 [GUARDRAILS_STRATEGY.md](docs/00-meta/GUARDRAILS_STRATEGY.md)를 참고한다.

## 처음 시작할 때 먼저 볼 문서

- [NEW_PROJECT_CHECKLIST.md](docs/00-meta/NEW_PROJECT_CHECKLIST.md) — 새 프로젝트 시작 체크리스트
- [TEMPLATE_GUIDE.md](docs/00-meta/TEMPLATE_GUIDE.md) — 문서 구조와 네이밍 규칙
- [WORKFLOW.md](docs/00-meta/WORKFLOW.md) — 단계별 워크플로우

## 입력 팁

한 줄 입력만으로도 시작할 수 있지만, 아래 4가지를 포함하면 결과 품질이 올라간다.

- 무엇을 만드는지
- 누가 쓰는지
- 어떤 문제를 푸는지
- 현재 확정된 것과 아직 미정인 것

더 많은 예시는 [BOOTSTRAP_PROMPT_EXAMPLES.md](docs/00-meta/BOOTSTRAP_PROMPT_EXAMPLES.md)를 참고한다.

## Contributing

개선 제안이나 버그 제보는 [이슈 템플릿](.github/ISSUE_TEMPLATE)을, 구조 변경은 [PR 템플릿](.github/PULL_REQUEST_TEMPLATE)을 참고한다.

## License

MIT