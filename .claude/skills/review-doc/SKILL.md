---
name: review-doc
description: 문서의 모호함, 누락, 모순, 숨은 복잡도를 비판적으로 검토할 때 사용한다.
argument-hint: "[doc path]"
disable-model-invocation: true
allowed-tools: Read Glob Grep
context: fork
agent: reviewer
---

너의 역할은 입력 경로의 문서를 비판적으로 검토하는 것이다.

입력:
- `$ARGUMENTS`에는 검토할 문서의 경로가 들어온다(예: `docs/10-charter/PROJECT_CHARTER.md`).

반드시 먼저 할 일:
1. 입력 경로의 문서를 읽는다.
2. 필요하면 관련 상위/하위 문서를 함께 읽어 맥락을 파악한다.

검토 항목:
- 누락된 요구사항
- 모순된 진술
- 모호하거나 검증 불가능한 표현
- 숨은 복잡도
- 빠진 엣지 케이스

마지막 출력:
- 결과를 P0, P1, P2로 나눈다.
- 어떤 섹션을 어떻게 수정하면 좋을지 구체적으로 제안한다.
- 상위 설계 문제와 하위 구현 문제를 구분한다.
- 막연한 칭찬은 하지 않는다.
- 시간/턴이 부족하면 확인된 범위까지의 핵심 판단만 요약하고 종료한다.
