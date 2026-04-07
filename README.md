# Claude Code Agentic Boilerplate

Claude Code 기반의 문서 중심 개발 보일러플레이트 저장소다.

## 목적
- 새 프로젝트를 시작할 때 문서 구조, 에이전트 구조, skill 구조를 재사용하기 위한 템플릿이다.
- 범위 정의 → 시스템 설계 → 작업 단위 분해 → QA/개선의 흐름을 일관되게 유지하는 것을 목표로 한다.

## 주요 구성
- `CLAUDE.md`: 프로젝트 공통 지침
- `.claude/agents`: 역할별 subagent
- `.claude/skills`: 반복 가능한 작업 절차
- `docs`: 계층화된 문서 구조
- `docs/90-decisions`: ADR 기록

## 문서 계층
- `docs/00-meta`
- `docs/10-charter`
- `docs/20-system`
- `docs/30-workitems`
- `docs/40-validation`
- `docs/90-decisions`

## 사용 방식
1. 이 저장소를 복제한다.
2. 새 프로젝트에 맞게 `PROJECT_CHARTER.md`를 수정한다.
3. 아키텍처 개요와 디자인 시스템을 채운다.
4. workitem 템플릿으로 마일스톤/기능/작업 문서를 만든다.
5. 이후 구현과 검증은 문서 구조를 기준으로 진행한다.

## Guardrails philosophy
이 템플릿은 cross-platform 재사용성을 우선한다.
그래서 shared 기본값에는 OS/셸/런타임 종속적인 hook와 검증 스크립트를 포함하지 않는다.

자동화는 프로젝트의 실제 스택이 정해진 뒤 추가한다.
자세한 내용은 docs/00-meta/GUARDRAILS_STRATEGY.md를 참고한다.

## 이런 경우에 적합함
- Claude Code 기반으로 문서 중심 개발을 하고 싶을 때
- 새 프로젝트마다 같은 문서 구조와 에이전트 구조를 재사용하고 싶을 때
- 범위 정의, 설계, 작업 분해, QA/개선 흐름을 반복 가능한 형태로 만들고 싶을 때

## 시작 순서
1. 이 저장소를 템플릿으로 사용해 새 저장소를 만든다
2. docs/00-meta/NEW_PROJECT_CHECKLIST.md를 따라 초기 설정을 한다
3. 프로젝트 스택이 정해지면 guardrail을 그 스택에 맞게 추가한다

## One-shot project bootstrap

새 프로젝트를 시작할 때는 Claude Code에서 아래 명령 중 하나로 시작한다.

### 1) 프로젝트 초기 세팅
```text
/bootstrap-project [프로젝트 설명]
```

예시:
```text
/bootstrap-project 개인 커리어 관리 SaaS. 사용자는 JD와 이력서를 비교하고, 부족한 역량을 추적하고, 주간 액션 플랜을 관리한다. 초기 타깃은 취준생. 아직 스택은 미정.
```

### 2) 스택 확정 후 세팅
```text
/bootstrap-stack [스택/런타임 설명]
```

예시:
```text
/bootstrap-stack Next.js 16 + TypeScript + pnpm + Supabase + Playwright + Vercel
```
중요한 기획과 설계는 Opus 기반 skill/subagent가 담당하도록 설계되어 있다.

## Better prompt quality
한 줄 입력만으로도 시작할 수 있지만, 아래 4가지를 포함하면 결과 품질이 더 좋아진다.

- 무엇을 만드는지
- 누가 쓰는지
- 어떤 문제를 푸는지
- 현재 확정된 것과 아직 미정인 것

예시:
```text
/bootstrap-project 개인 회고 SaaS. 사용자는 하루 회고와 주간 회고를 기록하고, 원인 분석과 개선 추적을 한다. 초기 타깃은 자기관리 욕구가 높은 직장인과 학생. 아직 스택은 미정이고, 모바일 우선 UX를 원한다.
```

## 에이전트 실행

이 보일러플레이트는 메인 세션이 오케스트레이션을 담당하고, 서브에이전트가 실작업을 수행하는 방식을 우선한다.

```text
/implement-workitem T-001-auth-session   # 구현
/validate-workitem T-001-auth-session    # 검증
```

상세 전략과 위임 조건은 [docs/00-meta/AGENT_EXECUTION_STRATEGY.md](docs/00-meta/AGENT_EXECUTION_STRATEGY.md)를 참조한다.
