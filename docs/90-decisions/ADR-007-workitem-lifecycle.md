# ADR-007 워크아이템 라이프사이클

## 상태
accepted

## 배경
이 보일러플레이트의 워크아이템 흐름은 단순한 "구현 → 검증" 두 단계가 아니라 다음 8단계로 정의되어야 한다. 각 단계의 책임이 분리되어 있어야 sub-agent fork 환경에서 결과 회수와 책임 경계가 명확해진다.

본 ADR 이전의 워크플로우에서는 검증과 마감이 한 명령에 묶이지 않아 사람이 매번 status 갱신과 커밋을 수동으로 했고, `/validate-workitem`이 검증 외 작업까지 떠안는 경향이 생겼다.

## 결정
워크아이템 라이프사이클을 다음 8단계로 정의한다.

| # | 단계 | skill | 주체 agent | 책임 경계 |
|---|------|-------|-----------|----------|
| 1 | discover | `/discover-product` | (메인 세션 운전) | persona/pain/JTBD/시나리오 발굴 → DISCOVERY.md |
| 2 | bootstrap | `/bootstrap-project` | architect-opus | DISCOVERY.md → charter/architecture/M1/F-001 |
| 3 | plan | `/plan-workitem` | planner | milestone/feature/task 분해 |
| 4 | implement | `/implement-workitem` | builder-sonnet | task 구현 (Red→Green→Refactor 사이클, ADR-009) |
| 5 | validate | `/validate-workitem` | validator-sonnet | 판정 + report 기록. **status 변경·코드 수정·커밋 금지.** |
| 6 | repair (Needs Fix일 때만) | `/repair-workitem` | builder-sonnet | report의 실패 항목만 수정. **자동 커밋 금지, 새 기능 금지, 범위 밖 변경 금지.** |
| 7 | finalize (Pass일 때) | `/finalize-workitem` | builder-sonnet | status `done` 갱신 + 명시적 파일 add + Conventional Commits 커밋 |
| 8 | stabilize | `/stabilize-milestone` | (qa, reviewer를 위임) | 마일스톤 단위 종합 점검. **코드 수정·커밋·status 변경 금지.** |

skill 간 흐름은 **자동 호출이 아니라 텍스트 제안 → 사용자/메인이 발화**한다. 예: validate Pass 출력은 "다음 액션: `/finalize-workitem T-001`"을 텍스트로 제안한다. 자동 호출이 아니다.

## 근거
- 책임 분리로 sub-agent fork 환경에서 각 단계의 입출력이 명확해진다.
- "검증" 단계와 "마감" 단계 분리로 "검증은 통과인데 status가 in-progress"인 모순이 사라진다.
- repair 단계가 명시적이라 무한 루프(repair → validate → repair)에 가드를 둘 수 있다(연속 3회 시 사용자 확인).
- 라이프사이클 정의가 ADR로 박혀 있으면 fork된 미래 프로젝트에서 6개월 뒤 사용자가 "왜 이런 단계 분리인가"를 추적할 수 있다.

## 결과
- 8개 skill이 각 단계에 1:1로 대응한다.
- `docs/00-meta/WORKFLOW.md`가 이 라이프사이클을 단계별 사용법으로 풀어 적는다.
- `docs/00-meta/AGENT_EXECUTION_STRATEGY.md`가 단계별 위임 대상을 정의한다.
- `/validate-workitem`은 **판정 + report 기록 전용**. 자동 수정·자동 마감 금지.
- 무한 루프 가드: repair 한 라운드는 P0/P1만 처리하고 P2 이하는 다음 라운드 추천. 같은 task에서 repair → validate 사이클이 연속 3회 이상이면 사용자 확인 요구.

## 후속 작업
- `/finalize-workitem`이 통합 검증 명령(`validate`)을 한 번 더 돌리는 정책 — 직전 `/validate-workitem` 통과 후에도 안전성을 위해 한 번 더(상태가 변했을 수 있음).
- `/stabilize-milestone`은 코드 수정·커밋을 하지 않고 점검 결과를 누적 기록한다 — 후속 작업이 필요하면 `/repair-workitem` 또는 새 task로 연결.
