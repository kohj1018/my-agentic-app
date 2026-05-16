# ADR-035 — DISCOVERY.md Living Doc + Assumption Tracker

> scope: boilerplate

## Status
accepted

## 배경
- [관측됨+외부실증] `DISCOVERY_TEMPLATE.md`는 11섹션 placeholder. STRUCTURE.md는 "Living Doc"으로 분류했지만 *어떻게 살아 있는가* 정의 부재. mid-project pivot 시 재호출 절차 부재.
- [외부실증] Cagan dual-track Agile + Teresa Torres continuous discovery — discovery는 1회성 event가 아니라 ongoing. assumption tracker가 없으면 가설이 검증 없이 구현으로 이어진다.
- [관측됨] DISCOVERY → Charter *1방향 박기*만 정의 → 피벗 시 SSOT 모호.

## 결정

### 1. DISCOVERY_TEMPLATE 13섹션 (기존 11 + 신설 2)
- `## 12. Assumption Tracker` — 핵심 가정의 검증 결과 누적. 빈 결과 = "미검증 - 행동 차단", stabilize가 보고.
- `## 13. Opportunity Backlog` — 기각·검증실패 후보까지 보존 (Torres OST opportunity space).

### 2. `/discover-product` `--update` 모드
- 기존 DISCOVERY.md 있으면: R0→R1·R2→R3→R4 갱신 경로.
- `--fast --update`: assumption tracker만 갱신 (가장 빈번한 mid-project use case).

### 3. Idempotency
ID 매칭 — 기존 ID(A-1·A-2)면 *검증일·다음 행동만 갱신*, 새 가정이면 새 ID 부여.

### 4. DISCOVERY=SSOT / Charter=snapshot 명문화
- DISCOVERY.md = persona/scenario/assumption SSOT.
- Charter는 snapshot view — DISCOVERY 갱신 시 Charter는 자동 sync 안 됨.
- AGENTS.md에 1줄 명시. PROJECT_CHARTER.md 본문 끝에 안내 comment.

## 결과
- mid-project pivot 시 `/discover-product --update`로 DISCOVERY.md 갱신 → Charter 갱신 제안 흐름 확보.
- assumption tracker로 "왜 이걸 만들었지?" 질문에 즉각 답 가능.

## 잔여 모니터링
assumption tracker 빈 결과율 — stabilize가 "미검증 가정 N건" 형태로 보고.

## 참고
- ADR-022 (Ratchet Principle — [관측됨+외부실증] 라벨)
- ADR-007 (workitem lifecycle)
