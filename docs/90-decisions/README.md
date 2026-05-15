# ADR Index

> 이 디렉터리의 모든 ADR을 한눈에 본다. 새 ADR 추가 시 이 표에 한 줄을 추가하는 것이 기본 절차다(상세는 [_ADR_GUIDE.md](_ADR_GUIDE.md) 참조).

| # | 제목 | 상태 | 한 줄 요약 |
|---|------|------|-----------|
| 001 | Doc hierarchy | accepted | docs/ 디렉터리 6분할 결정 |
| 002 | Initial project decisions | (placeholder, `/bootstrap-project`가 새 프로젝트에서 생성) | bootstrap 단계의 초기 결정 모음 |
| 003 | Stack selection | (placeholder, `/bootstrap-stack`가 새 프로젝트에서 생성) | 스택 선택과 근거 |
| 004 | Model alias policy | accepted | shared 기본값에서 모델 별칭(`sonnet`, `opus`, `haiku`)만 사용 |
| 005 | Single Source of Truth (SSOT) | accepted | 같은 사실은 1곳에서 정의, 다른 곳은 한 줄 + 링크. 정책=ADR 패턴. |
| 006 | Simplicity, Clean Code, and Clean Architecture priority | accepted | 단순성 1순위, Clean Code 2순위, Clean Architecture 3순위 (정당화 시) |
| 007 | Workitem lifecycle | accepted | discover→bootstrap→plan→implement→validate→repair→finalize→stabilize 8단계 |
| 008 | Commit convention | accepted | Conventional Commits 기본 채택 |
| 009 | TDD default + opt-out | accepted | /implement-workitem 디폴트는 Red→Green→Refactor 사이클, opt-out은 사유+follow-up 모두 필요 |
| 010 | Multi-agent compatibility (AGENTS.md as canonical entry) | accepted | AGENTS.md를 캐노니컬 진입 페이지로, Codex CLI도 동일 워크플로우 동작 |
| 022 | Ratchet Principle | accepted | 정책의 제약 강도를 *제약(강)/enabling(약)*으로 차등 적용 |
| 031 | Non-web stacks out of direct support scope | accepted | 비웹 스택은 기본 자동화 직접 지원 범위 밖, override 경로 제공 |

## 신규 ADR 추가 절차
1. `_ADR_GUIDE.md`의 "권장 섹션"을 따라 ADR 본문 작성.
2. ADR 번호는 가장 큰 기존 번호 + 1.
3. 본 README의 표에 한 줄 추가(번호, 제목, 상태, 한 줄 요약).
4. 관련 agent/skill 본문에 ADR 링크를 박는다.
