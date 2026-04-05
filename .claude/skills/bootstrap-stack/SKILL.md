---
name: bootstrap-stack
description: Add stack-specific setup guidance and project automation after the stack and runtime are explicitly chosen.
argument-hint: "[stack and runtime summary]"
disable-model-invocation: true
allowed-tools: Read Glob Grep Write Edit
model: claude-opus-4-6
effort: max
---

너의 역할은 프로젝트 스택이 명확해진 이후, 이 보일러플레이트에 맞게 stack-specific 초기 세팅 문서를 정리하는 것이다.

입력:
- `$ARGUMENTS`에는 언어, 프레임워크, 패키지 매니저, 테스트 도구, 배포 환경 등이 자연어로 들어온다.

반드시 수행할 일:
1. 아래 문서를 먼저 읽는다.
   - `docs/00-meta/GUARDRAILS_STRATEGY.md`
   - `docs/00-meta/WORKFLOW.md`
   - `docs/10-charter/PROJECT_CHARTER.md`
   - `docs/20-system/ARCHITECTURE_OVERVIEW.md`

2. 사용자의 스택 정보를 바탕으로 아래 문서를 갱신한다.
   - `docs/20-system/ARCHITECTURE_OVERVIEW.md`
   - `docs/90-decisions/ADR-003-stack-selection.md`

3. 필요하면 아래 문서도 만든다.
   - `docs/00-meta/STACK_SETUP_PLAN.md`

4. 반드시 지켜야 할 원칙:
   - shared 기본값에 OS/셸 종속 hook를 강제로 넣지 않는다.
   - 대신 어떤 scripts, hooks, CI가 필요한지 문서로 정리한다.
   - 실제 실행 스크립트가 필요한 경우, 해당 스택에서 자연스러운 런타임을 기준으로 제안한다.
   - 확실하지 않은 환경 전제는 가정으로 표시한다.

5. 출력 방식:
   - 스택 선택 요약
   - 추가할 guardrail 목록
   - 바로 구현 가능한 스크립트/CI 후보
   - 남은 불확실성
