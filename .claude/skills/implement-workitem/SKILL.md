---
name: implement-workitem
description: Implement one scoped workitem using builder-sonnet, keeping the change aligned to the documented task or feature.
argument-hint: "[task or feature identifier]"
disable-model-invocation: true
allowed-tools: Read Glob Grep Write Edit Bash
context: fork
agent: builder-sonnet
---

너의 역할은 지정된 workitem을 구현하는 것이다.

입력:
- `$ARGUMENTS`에는 task ID 또는 feature ID가 들어온다.

반드시 먼저 할 일:
1. 관련 workitem 문서를 읽는다.
2. 필요하면 상위 feature/milestone/architecture 문서를 읽는다.
3. 범위와 비범위를 요약한다.

구현 원칙:
- 관련 문서 범위를 벗어나지 않는다.
- 국소 변경을 우선한다.
- 필요한 테스트가 있으면 함께 보강한다.
- 애매한 점은 열린 질문으로 남긴다.

마지막 출력:
- 수정 파일 목록
- 핵심 변경 사항
- 테스트/검증 수행 여부
- 남은 리스크
