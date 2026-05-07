# ADR-006 단순성·Clean Code·Clean Architecture 우선순위

## 상태
accepted

## 배경
이 보일러플레이트의 핵심 가치는 fork된 미래 프로젝트에서 AI 에이전트가 "요구한 범위만, 오버엔지니어링 없이, 가독성 좋게" 코드를 작성하게 만드는 것이다.

이 요구는 다음 세 층으로 분해된다.
1. **단순성·YAGNI** — 요구한 범위만 구현, 추측성 추상화 금지, 미래 대비 코드 금지.
2. **Clean Code** — 명확한 네이밍, 한 함수 한 가지 일, 중복 회피, WHY 주석.
3. **Clean Architecture** — 의존성 규칙, 레이어 경계 — 단 프로젝트 규모가 정당화할 때만.

순서가 중요하다. Clean Architecture를 단순성 위에 두면 작은 prototype에 4-layer를 강제해 사용자의 "오버하지 않으면서"와 직접 충돌한다.

## 결정
보일러플레이트는 위 세 층을 다음 우선순위로 적용한다(우선순위가 높을수록 강하게 강제한다).

1. **단순성·YAGNI (1순위)** — 모든 코드 변경에 강제 적용.
2. **Clean Code 6항목 (2순위)** — reviewer agent가 호출될 때마다 라벨링.
3. **Clean Architecture (3순위)** — 모듈이 3개 이상이거나 프로젝트 규모가 정당화할 때만 적용.

각 층의 surface 매핑:

| 층 | Surface | 형태 |
|----|---------|------|
| 단순성·YAGNI | `AGENTS.md` | 5개 항목, fork된 새 세션이 자동 로드. |
| 단순성·YAGNI | `.claude/agents/builder-sonnet.md` | 구현 출력 직전 self-check 4항목. 미통과 항목은 "남은 정리 항목"에 명시. |
| Clean Code 6항목 | `.claude/agents/reviewer.md` | 6항목 체크리스트, P0/P1/P2 + 항목 라벨링. |
| Clean Architecture | `.claude/agents/architect-opus.md` | "프로젝트 규모가 정당화하는가" self-check. |
| Clean Architecture | `docs/20-system/ARCHITECTURE_OVERVIEW.md`의 `## 3-1. 레이어 경계 + 의존성 규칙` | 모듈 ≥3 시 채울 것을 권장. |

## 근거
- 사용자가 명시적으로 "오버엔지니어링 회피·가독성 우선"을 핵심 가치로 강조했다.
- prototype 단계에서 4-layer를 강제하면 ceremony가 가치를 압도한다 — 단순성을 1순위로 두는 이유.
- Clean Code는 라벨링 비용이 낮으므로 reviewer가 매번 적용해도 비용 부담이 작다.
- Clean Architecture는 적용 비용이 크므로 규모가 정당화할 때만 강제한다.

## 결과
- `AGENTS.md`에 단순성 5개 항목 단락 (`CLAUDE.md`는 `@AGENTS.md` import).
- builder-sonnet, validator-sonnet, reviewer, architect-opus의 규칙에 self-check / 체크리스트 / 규모 점검 추가.
- `/implement-workitem` skill이 구현 시 단순성 self-check + Clean Code 6항목을 참조.
- ARCHITECTURE_OVERVIEW의 `## 3-1. 레이어 경계 + 의존성 규칙` 섹션이 "프로젝트 규모가 정당화될 때만 채운다" YAGNI 보호 단서와 함께 도입된다(이 ADR과 같은 적용 사이클의 후속 변경).
- `/stabilize-milestone`이 reviewer 입력에 Clean Code 6항목 체크리스트를 명시한다.

## 후속 작업
- 단순성 self-check 4항목이 builder-sonnet 출력 비용을 늘리는지 측정. 비용이 크면 축약 검토.
- `legacy 코드`의 premature abstraction은 즉시 제거하지 않고 `/stabilize-milestone`이 후보로만 보고하는 정책을 유지한다(사용자 결정 우선).
