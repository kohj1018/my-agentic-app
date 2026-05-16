# ADR-027 — 인터페이스 결정 책임 분배

> scope: boilerplate

## Status
accepted

## 배경
- [외부실증] [prg.sh — Why Your AI Keeps Building the Same Purple Gradient](https://prg.sh/ramblings/Why-Your-AI-Keeps-Building-the-Same-Purple-Gradient-Website) — LLM이 시각 결정 입력 없이 생성하면 median 미감(purple gradient generic SaaS)으로 수렴한다. 명시적 결정 자리가 없으면 매 task마다 LLM이 즉흥 결정한다.
- [관측됨] `DESIGN_SYSTEM.md`가 UI / API·백엔드 / CLI 3 그룹 placeholder를 한 파일에 담아 "광의 SSOT" 시도 → 백엔드 개발자에게 misnomer + ARCHITECTURE 운영성 섹션과 책임 중복.
- [관측됨] ARCHITECTURE_OVERVIEW.md에 API 컨벤션·CLI 출력 포맷·백엔드 핵심 결정·프론트 결정 자리 부재 → 첫 구현 시점에 LLM 즉흥 결정.

## 결정 (15개)

1. `DESIGN_SYSTEM.md` → `DESIGN.md` rename + UI 한정.
2. ARCHITECTURE 7-1(API 컨벤션), 7-2(CLI 컨벤션), 7-3(백엔드 결정), 7-4(프론트 결정) sub-section 신설.
3. `/bootstrap-design` skill 신설 (UI 한정, R0~R4 라운드).
4. `/bootstrap-stack`이 7-1/7-2/7-3/7-4 채움 책임 (스택 감지 → architect-opus 단발 sub-call).
5. UI DESIGN.md는 Stitch DESIGN.md canonical 섹션 순서 채택 (Overview / Colors / Typography / Layout / Elevation & Depth / Shapes / Components / Motion / Do's and Don'ts).
6. 3-tier DTCG 토큰 모델 (primitive → semantic → component).
7. 영역별 Don'ts 섹션 필수 — LLM 정확도 향상의 단일 최대 기여.
8. **repo root `DESIGN.md` 두지 않음** — 외부 도구 자동 발견 마찰은 사용자가 root stub으로 ad-hoc 해결.
9. **운영성 ↔ 7-1 경계 가이드**: trace ID/log 포맷/관측 stack=운영성. 응답 envelope/error 레지스트리/네이밍=7-1. 흐릿한 영역(error 응답에 trace ID)은 *주된 결정 맥락*으로 판단.
10. **charter 비목표 가이드**: "미적 트렌드 추종(neumorphism, glassmorphism, 그날의 dribbble)"을 비목표로 박을 것 권장.
11. **API/CLI에 R0~R4 같은 라운드 도입 안 함** (YAGNI, ADR-006). API/CLI는 스택·도메인 강결합이라 `/bootstrap-stack`의 architect-opus 단발 sub-call로 충분.
12. **shadcn/ui 권장이지 강제 아님** — ownership 모델(코드 복사)이라 lock-in 약함. R3 스택별 시작점 표 7개 항목이 대안 커버.
13. **`/bootstrap-design --fast` 도입** — R0(레퍼런스 1개) + R1(원칙 1줄 minimal) + R2(토큰). R3·R4 생략.
14. **fork 격리 풀고 메인 세션이 R0~R4 운전** (discover-product 패턴) — `context: fork` 미명시. R0/R1 무거운 추론은 architect-opus 단발 sub-call. 종료 후 사용자 `/clear` 권장.
15. **ARCHITECTURE 11→14 섹션 비대화 수용** — 7-1~7-4는 모두 "있을 때만 채움" 가이드라 비해당 프로젝트는 통째 삭제 가능.

## 비결정 (영구 No)
- DESIGN_SYSTEM 광의 SSOT 유지 — misnomer + 운영성 중복 비용.
- 영역별 3개 파일 분리 (DESIGN / ARCHITECTURE_API / ARCHITECTURE_BACKEND) — 단순성 1순위(ADR-006) 위반.
- UI까지 ARCHITECTURE 흡수 — "시각"과 "구조" 혼합으로 가독성 저하.

## 마이그레이션 (9 surface 변경)
1. `DESIGN_SYSTEM.md` → `DESIGN.md` rename + UI 한정 본문
2. ARCHITECTURE_OVERVIEW.md 7-1/7-2/7-3/7-4 sub-section 신설
3. STRUCTURE.md 산출물 표 갱신
4. STRUCTURE.md Canonical Owner 매핑 4줄 추가
5. bootstrap-project output-checklist.md 갱신
6. AGENTS.md 인덱스 1줄 추가
7. `/bootstrap-design` skill 신설
8. `/bootstrap-stack` skill 본문 갱신 (인터페이스 채움 + 프론트 감지 시 권장 출력)
9. builder-sonnet / validator-sonnet self-check 각 1줄 추가

## 시나리오 검증 표

| 시나리오 | DESIGN.md | 7-1 | 7-2 | 7-3 | 7-4 |
|---------|-----------|-----|-----|-----|-----|
| Next.js SaaS | `/bootstrap-design` 채움 | - | - | - | `/bootstrap-stack` 채움 |
| FastAPI 백엔드 | 미생성 | `/bootstrap-stack` 채움 | - | `/bootstrap-stack` 채움 | - |
| Rust CLI | 미생성 | - | `/bootstrap-stack` 채움 | - | - |
| 풀스택 (Next.js + FastAPI) | `/bootstrap-design` 채움 | `/bootstrap-stack` 채움 | - | `/bootstrap-stack` 채움 | `/bootstrap-stack` 채움 |
| RN+Expo (ADR-031 override) | `/bootstrap-design` 채움 (Tamagui) | - | - | - | `/bootstrap-stack` 채움 |
| 라이브러리 패키지 | 미생성 | - | - | - | - |
| UI + root stub | `/bootstrap-design` 채움 + root에 stub 1줄 별도 | - | - | - | - |
| `--fast` prototype | R2(토큰)만 | - | - | - | - |

## 외부 근거

- [Stitch DESIGN.md spec (Google Labs)](https://github.com/google-labs-code/design.md/blob/main/docs/spec.md) — canonical 섹션 순서 채택 (결정 5).
- [W3C DTCG 2025.10 stable](https://www.w3.org/community/design-tokens/) — 3-tier 토큰 모델 표준 (결정 6).
- [designproject.io — How to write a design.md AI agents follow](https://designproject.io/blog/design-md-file/) — Don'ts가 LLM 정확도 향상의 단일 최대 기여 (결정 7).
- [Brad Frost — Agentic Design Systems in 2026](https://bradfrost.com/blog/post/agentic-design-systems-in-2026/) — deliberate constraint 안에서 AI 출력 품질 상한 (결정 1·11 근거).
- [Material 3 — Motion easing/duration](https://m3.material.io/styles/motion/easing-and-duration) — 라우팅 UI 160~240ms / entrance·exit 240~360ms (DESIGN.md `## 8. Motion` 근거).
- [prg.sh — Why Your AI Keeps Building the Same Purple Gradient](https://prg.sh/ramblings/Why-Your-AI-Keeps-Building-the-Same-Purple-Gradient-Website) — LLM median 회귀 진단 (배경).
- [Smashing — Naming best practices for tokens](https://www.smashingmagazine.com/2024/05/naming-best-practices/) — 3-tier kebab-case 컨벤션 (결정 6).

## 후속 작업
- ADR-017 시뮬레이션 Round 2에서 DESIGN.md 채움 효과 측정 (LLM 시각 결정 일관성 delta).
- ADR-017 Round 2 결과에 따라 shadcn 채움 자동화 가치 입증 시 후속 ADR 검토.

## 참고
- ADR-006 (단순성 1순위)
- ADR-022 (Ratchet Principle — [외부실증] 라벨)
- ADR-031 (비웹 스택 override 경로)
