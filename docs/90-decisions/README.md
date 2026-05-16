# ADR Index Hub

> scope 정책의 SSOT는 [boilerplate/ADR-000](boilerplate/ADR-000-boilerplate-decision-policy.md) 참조.

본 디렉터리는 두 종류의 ADR을 분리해 둔다.

- **[Boilerplate ADR Index](boilerplate/README.md)** — 보일러플레이트 자체 정책 (000~099). fork 후 read-only. supersede는 project/에서 박는다.
- **[Project ADR Index](project/README.md)** — fork 사용자가 박는 프로젝트 결정 (100+).

## 새 ADR을 어디 박는가

- 새로 시작한 프로젝트의 첫 결정 (스택, 데이터 모델 등)을 박을 때 → `project/ADR-100-<slug>.md`. **Project ADR은 무조건 100+ 번호**. ADR-002 / ADR-003은 legacy reserved placeholder라 새로 박지 않는다.
- 보일러플레이트 자체 정책을 갱신·추가할 때 → `boilerplate/ADR-NNN-<slug>.md` (100 미만 빈 번호 사용 또는 새 번호).
- 기존 boilerplate 결정을 뒤집을 때 → `project/ADR-1NN-<slug>.md` (예: ADR-100, ADR-101) 본문 첫 줄에 `Supersedes ADR-NNN (boilerplate)` 명시.
