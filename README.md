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
