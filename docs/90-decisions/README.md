# ADR Index

> 이 디렉터리의 ADR을 한눈에 본다. ADR scope 정책은 [ADR-000](ADR-000-boilerplate-decision-policy.md) 참조.

## Boilerplate ADR (fork 후 supersede 가능)

| # | 제목 | 상태 | 한 줄 요약 |
|---|------|------|-----------|
| 000 | Boilerplate decision policy | accepted | scope 라벨링 + supersede + 번호 정책 |
| 001 | Doc hierarchy | accepted | docs/ 디렉터리 6분할 결정 |
| 004 | Model alias policy | accepted | shared 기본값에서 모델 별칭(`sonnet`, `opus`, `haiku`)만 사용 |
| 005 | Single Source of Truth (SSOT) | accepted | 같은 사실은 1곳에서 정의, 다른 곳은 한 줄 + 링크. 정책=ADR 패턴. |
| 006 | Simplicity, Clean Code, and Clean Architecture priority | accepted | 단순성 1순위, Clean Code 2순위, Clean Architecture 3순위 (정당화 시) |
| 007 | Workitem lifecycle | accepted | discover→bootstrap→plan→implement→validate→repair→finalize→stabilize 8단계 |
| 008 | Commit convention | accepted | Conventional Commits 기본 채택 |
| 009 | TDD default + opt-out | accepted | /implement-workitem 디폴트는 Red→Green→Refactor 사이클, opt-out은 사유+follow-up 모두 필요 |
| 010 | Multi-agent compatibility (AGENTS.md as canonical entry) | accepted | AGENTS.md를 캐노니컬 진입 페이지로, Codex CLI도 동일 워크플로우 동작 |
| 011 | AGENTS.md 100줄 hard cap | accepted | AGENTS.md 최대 100줄, 신규 정책은 ADR + 1줄 링크 |
| 012 | docs/00-meta 문서 아키텍처 정리 | accepted | 9→6 흡수 + Diátaxis 모드 라벨 추가 |
| 014 | Milestone graduation contract | accepted | graduation checklist 5+1 + 회고 + pre-check + --dry-run |
| 017 | Dogfood 시뮬레이션 의무 + 재실행 트리거 | accepted | todo CLI baseline 시뮬레이션 + 성공 기준 3개 + 재실행 트리거 3종 |
| 019 | Context Packs + JIT 로딩 | accepted | minimal/full 2종 context-pack + 사전 fork-load 금지 정책 |
| 022 | Ratchet Principle | accepted | 정책의 제약 강도를 *제약(강)/enabling(약)*으로 차등 적용 |
| 024 | Claude Code plan 모드 lifecycle 비범위 | accepted | plan 모드 비의무화, plansDirectory 제거, think-before-edit 규율 확보 |
| 026 | plan-workitem 강화 (TASK_TEMPLATE schema) | accepted | AC GWT 형식 + sizing 3한계 + 의존성 섹션 + planner self-check |
| 027 | 인터페이스 결정 책임 분배 | accepted | DESIGN.md(UI) + ARCHITECTURE 7-1~7-4(API/CLI/백엔드/프론트) + /bootstrap-design 신설 |
| 031 | Non-web stacks out of direct support scope | accepted | 비웹 스택은 기본 자동화 직접 지원 범위 밖, override 경로 제공 |

## Project ADR (fork된 프로젝트가 채움, ADR-100부터)

> 아래 항목은 `scope: project` placeholder. 파일은 fork 사용자가 `/bootstrap-project` / `/bootstrap-stack` 실행 시점에 생성되며, 생성 시 첫 줄에 `> scope: project`를 박는다 (ADR-000).

| # | 제목 | 상태 | 한 줄 요약 |
|---|------|------|-----------|
| 002 | Initial project decisions | (placeholder) | bootstrap 단계 초기 결정 모음 |
| 003 | Stack selection | (placeholder) | 스택 선택과 근거 |

## 신규 ADR 추가 절차
1. `_ADR_GUIDE.md`의 "권장 섹션"을 따라 ADR 본문 작성.
2. Boilerplate ADR은 위 "Boilerplate ADR" 표에, Project ADR은 "Project ADR" 표에 한 줄 추가.
3. 관련 agent/skill 본문에 ADR 링크를 박는다.
4. scope 정책은 ADR-000 참조.
