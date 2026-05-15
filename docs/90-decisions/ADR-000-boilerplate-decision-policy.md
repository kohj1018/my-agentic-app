# ADR-000 — Boilerplate decision policy (메타)

> scope: boilerplate

## Status
accepted

## 배경
- 본 디렉터리의 ADR-001/004~010은 *이 보일러플레이트 자체*의 정책 결정이다.
- fork된 프로젝트가 자기 ADR을 박을 때 (a) 보일러플레이트 결정과 자기 결정의 시각적 구분 부재, (b) supersede 권한 모호, (c) 새 ADR 번호 시작점 모호의 마찰을 갖는다.

## 결정

### A. scope 라벨링
모든 ADR 첫 줄(제목 다음)에 다음 표시를 둔다.
- `> scope: boilerplate` — 본 보일러플레이트 자체 결정. fork 후 supersede 가능.
- `> scope: project` — fork된 프로젝트의 자체 결정.

본 보일러플레이트가 박는 모든 ADR은 `scope: boilerplate`로 박는다. **예외 — ADR-002, ADR-003**: 이 두 번호는 *보일러플레이트가 박아둔 fork-사용자용 placeholder slot*이라 `scope: project`로 표기한다 (placeholder를 fork 사용자가 채우면 그 결정은 project scope). 현재 ADR-002/003 파일은 미생성 상태 — fork 사용자가 `/bootstrap-project` / `/bootstrap-stack` 실행 시 생성된다.

### B. README 섹션 분리
`docs/90-decisions/README.md`를 두 섹션으로 분리.
1. **Boilerplate ADR** (fork 후 supersede 가능)
2. **Project ADR** (fork된 프로젝트의 결정)

### C. fork 후 ADR 번호 정책
- fork 사용자는 보일러플레이트 ADR 번호 범위(001~099)를 그대로 사용하지 않는다.
- 새 프로젝트 ADR은 **ADR-100부터** 시작한다(예: `ADR-100-stack-selection.md`).
- 보일러플레이트 ADR을 supersede할 경우 본인 번호(ADR-100+)에서 박은 뒤 본문 첫 줄에 `Supersedes ADR-NNN (boilerplate)` 표기.
- **ADR-002, ADR-003 예외**: fork 사용자가 채우는 첫 결정은 002/003을 채워도 되고, ADR-100부터 새로 시작해도 된다.

### D. supersede 권한
- fork 사용자는 boilerplate ADR을 자유롭게 supersede할 수 있다.
- supersede ADR은 *왜* 뒤집는지(프로젝트 컨텍스트·제약)를 본문에 1단락 명시.

## 결과
- fork 직후 사용자가 *어느 ADR이 내 결정인가*를 1초 안에 식별 가능.
- ADR 번호 충돌 영구 회피.

## 후속 작업
없음
