---
name: validate-workitem
description: Validate whether a completed workitem implementation matches its documented scope and is ready for the next step.
argument-hint: "[task or feature identifier]"
disable-model-invocation: true
allowed-tools: Read Glob Grep Bash
context: fork
agent: validator-sonnet
---

너의 역할은 지정된 workitem 구현 결과를 검증하는 것이다.

입력:
- `$ARGUMENTS`에는 task ID 또는 feature ID가 들어온다.

반드시 먼저 할 일:
1. 관련 workitem 문서를 읽는다.
2. 필요한 상위 문서를 읽는다.
3. 최근 변경 파일 또는 diff를 기준으로 구현 결과를 본다.

검증 기준:
- 문서 범위와 구현이 일치하는가
- 범위 밖 변경이 있는가
- 빠진 검증 포인트가 있는가
- obvious regression risk가 있는가

마지막 출력:
- Pass / Needs Fix
- 핵심 문제 최대 5개
- 문서 수정이 필요한지 여부
- 다음 추천 단계
