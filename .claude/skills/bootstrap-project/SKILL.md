---
name: bootstrap-project
description: Initialize a new project from this boilerplate using one natural-language project brief.
argument-hint: "[project brief]"
disable-model-invocation: true
allowed-tools: Read Glob Grep Write Edit
model: claude-opus-4-6
effort: max
---

너의 역할은 이 보일러플레이트를 기준으로 새 프로젝트의 초기 문서 세팅을 완료하는 것이다.

입력:
- `$ARGUMENTS`에는 사용자의 프로젝트 설명이 자연어로 들어온다.

반드시 수행할 일:
1. 아래 문서를 먼저 읽어 구조를 이해한다.
   - `CLAUDE.md`
   - `docs/00-meta/TEMPLATE_GUIDE.md`
   - `docs/00-meta/WORKFLOW.md`
   - `docs/00-meta/GUARDRAILS_STRATEGY.md`
   - `docs/00-meta/NEW_PROJECT_CHECKLIST.md`

2. 사용자의 프로젝트 설명을 바탕으로 아래 문서를 갱신한다.
   - `README.md`
   - `docs/10-charter/PROJECT_CHARTER.md`
   - `docs/20-system/ARCHITECTURE_OVERVIEW.md`

3. 필요하다면 아래도 함께 갱신한다.
   - `docs/20-system/DESIGN_SYSTEM.md`
   - `docs/90-decisions/ADR-002-initial-project-decisions.md`

4. 최초 workitem 문서를 만든다.
   - `docs/30-workitems/milestones/M1-foundation.md`
   - `docs/30-workitems/features/F-001-core-value.md`

5. 반드시 지켜야 할 원칙:
   - 추측은 사실처럼 쓰지 말고 가정으로 표시한다.
   - 스택이 명시되지 않았다면 stack-specific 자동화는 만들지 않는다.
   - hooks, CI, lint/test 스크립트는 스택이 명확할 때만 추가한다.
   - 상위 문서와 하위 문서의 역할을 섞지 않는다.
   - 너무 많은 파일을 새로 만들지 않는다. 꼭 필요한 초기 파일만 만든다.

6. 출력 방식:
   - 무엇을 갱신했는지 요약한다.
   - 아직 비어 있는 결정 사항을 명확히 남긴다.
   - 다음 추천 단계가 있다면 3개 이하로 짧게 제안한다.
