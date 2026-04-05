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
