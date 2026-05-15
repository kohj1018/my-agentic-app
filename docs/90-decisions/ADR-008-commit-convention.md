# ADR-008 Conventional Commits 기본 채택

> scope: boilerplate

## 상태
accepted

## 배경
`/finalize-workitem`이 도입되면서 커밋 메시지 양식의 표준이 필요해졌다. 일관된 양식은 다음을 가능하게 한다.
- changelog 자동화 가능성 (스택 확정 후)
- 커밋 단위 작업 추적
- 변경 종류(feat/fix/chore/...)에 따른 리뷰 우선순위 판단

## 결정
보일러플레이트의 기본 커밋 컨벤션은 [Conventional Commits](https://www.conventionalcommits.org)다.

기본 형식:
```
<type>(<scope>): <summary>

<optional body>
<optional footer (e.g., task ID 참조)>
```

기본 type 어휘: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `perf`, `build`, `ci`, `style`, `revert`.

## 근거
- 커뮤니티 표준이라 진입 비용이 낮다.
- 단순한 형식으로 메시지의 의도가 한눈에 보인다.
- changelog 도구 생태계가 잘 정비되어 있다.
- 보일러플레이트 자체가 다언어/다스택을 지원하므로 stack-agnostic한 표준이 필요하다.

## 결과
- `/finalize-workitem`이 커밋 메시지 초안을 Conventional Commits 형식으로 생성한다.
- charter에서 다른 컨벤션을 명시적으로 override하지 않는 한 기본은 Conventional Commits.

## 후속 작업
- 사용자가 다른 컨벤션을 원하면 charter에서 명시적으로 override + 새 ADR로 결정 보존.
- changelog 자동화는 스택 확정 후 별도 task로 도입.
