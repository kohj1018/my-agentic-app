---
name: bootstrap-project
description: Initialize a new project from this boilerplate using one natural-language project brief.
argument-hint: "[project brief]"
disable-model-invocation: true
allowed-tools: Read Glob Grep Write Edit
context: fork
agent: architect-opus
model: opus
effort: max
---

너의 역할은 이 보일러플레이트를 기준으로 새 프로젝트의 초기 문서 세팅을 완료하는 것이다.

입력:
- `$ARGUMENTS`에는 사용자의 프로젝트 설명이 자연어로 들어온다.
- 입력이 짧더라도 스스로 구조화해서 해석한다.
- 필요하면 `brief-template.md`의 구조를 내부적으로 참고해 누락된 요소를 정리한다.

반드시 먼저 읽을 파일:
- `CLAUDE.md`
- `docs/00-meta/TEMPLATE_GUIDE.md`
- `docs/00-meta/WORKFLOW.md`
- `docs/00-meta/GUARDRAILS_STRATEGY.md`
- `docs/00-meta/NEW_PROJECT_CHECKLIST.md`
- `brief-template.md`
- `output-checklist.md`
- `examples/career-saas-example.md`

반드시 수행할 일:
1. 사용자의 프로젝트 설명을 구조화한다.
2. 아래 문서를 갱신한다.
   - `README.md`
   - `docs/10-charter/PROJECT_CHARTER.md`
   - `docs/20-system/ARCHITECTURE_OVERVIEW.md`
3. 필요하면 아래도 함께 갱신한다.
   - `docs/20-system/DESIGN_SYSTEM.md`
   - `docs/90-decisions/ADR-002-initial-project-decisions.md`
4. 최초 workitem 문서를 만든다.
   - `docs/30-workitems/milestones/M1-foundation.md`
   - `docs/30-workitems/features/F-001-core-value.md`

반드시 지켜야 할 원칙:
- 추측은 사실처럼 쓰지 말고 가정으로 표시한다.
- 스택이 명시되지 않았다면 stack-specific 자동화는 만들지 않는다.
- hooks, CI, lint/test 스크립트는 스택이 명확할 때만 추가한다.
- 상위 문서와 하위 문서의 역할을 섞지 않는다.
- 꼭 필요한 초기 파일만 만든다.

마지막 출력:
- 갱신한 파일 목록
- 핵심 가정
- 남은 미결정 사항
- 다음 추천 단계 최대 3개
