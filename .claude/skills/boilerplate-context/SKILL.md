---
name: boilerplate-context
description: Use when working in this repository to understand the boilerplate's layered documentation system, workitem flow, and guardrail philosophy.
user-invocable: false
---

이 저장소는 Claude Code용 문서 중심 보일러플레이트다.

핵심 구조:
- `docs/00-meta`: 템플릿 사용법, 워크플로우, guardrail 철학
- `docs/10-charter`: 프로젝트 범위, 문제, 목표, 제약
- `docs/20-system`: 시스템 구조와 설계 개요
- `docs/30-workitems`: milestone, feature, task
- `docs/40-validation`: QA와 개선
- `docs/90-decisions`: ADR

기본 원칙:
- 상위 문서 없이 하위 문서를 먼저 만들지 않는다.
- 프로젝트 초기화 시에는 charter와 architecture를 먼저 정리한다.
- 실제 구현 단위는 workitems로 분해한다.
- stack-specific 자동화는 프로젝트 스택이 확정된 뒤에 생성한다.
- shared 기본값에는 OS/셸/런타임 종속적인 자동화를 강제로 넣지 않는다.

프로젝트 초기 세팅이 필요하면 `/bootstrap-project`를 우선 사용한다.
스택 자동화 세팅이 필요하면 `/bootstrap-stack`을 사용한다.
