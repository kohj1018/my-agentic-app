---
name: write-architecture
description: 프로젝트 charter를 바탕으로 시스템 구조와 상위 설계 문서를 정리할 때 사용한다.
---

# 목적
`docs/20-system/ARCHITECTURE_OVERVIEW.md`를 생성하거나 갱신한다.

## 작업 절차
1. `docs/10-charter/PROJECT_CHARTER.md`를 먼저 읽는다.
2. 시스템 경계, 상위 구조, 도메인 모델, 데이터 흐름, 외부 연동 지점을 정리한다.
3. 리스크와 미확정 사항을 명시한다.
4. 과도한 구현 세부사항은 피한다.

## 출력 원칙
- 이 문서는 상위 설계 문서다.
- task 수준 구현 체크리스트를 넣지 않는다.
- 필요한 하위 구현 단위는 별도 workitem 문서로 분리한다.
