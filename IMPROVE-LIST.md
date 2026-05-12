# IMPROVE-LIST

> 이 보일러플레이트의 개선 주제 모음. 각 항목은 "현재 상태 → 한계 진단 → 개선 방향 → 트레이드오프/고민" 순서로 정리한다.
> 우선 채택 여부를 결정하기 전 단계의 검토용 리스트이며, 결정된 항목은 ADR로 박는다.

> **재작성 이력 (시간 역순)**:
> - **2026-05-09**: 3·5·6번 두괄식 재작성 + 모든 트레이드오프를 *결정*으로 변환. TL;DR + 핵심 결정 표(P0/P1/P2/No) + 결정 근거 표가 섹션 진입점. 외부 자료 합의는 표로 압축. 명시적 No 결정에는 *영구 No / 조건부 No / 데이터 트리거 보류* 구분. ADR 후보 모두 *결정 1개에 ADR 1개* 분할(ADR-011~022).
> - **2026-05-09**: 7번 두괄식 재구성(스택 시나리오 5종 × lifecycle 8단계 빈자리 매핑) + 8번 두괄식 재작성(기획 PO·PM 두 트랙).
> - **2026-05-08**: 7번(스택 시나리오 사고 실험) 및 8번(기획 빈자리) 신설 초안.
> - **2026-05-07**: 3·5·6번 외부 자료 합의 정렬(라운드 1) + 자기 비판 6항목 반영(라운드 2). 섹션 간 이동 — AGENTS.md slim은 6→3으로, dogfood 시뮬레이션은 6→5로.
>
> 외부 자료 인덱스는 우선순위 요약 끝 단락 참조.

---

## 1. 인터페이스 결정(시각·API·CLI)을 먼저 하는 흐름

### 결정 (TL;DR)

| # | 변경 | 영향 |
| --- | --- | --- |
| 1 | `docs/20-system/DESIGN_SYSTEM.md` → **`docs/20-system/DESIGN.md`로 rename + UI 한정** | UI 프로젝트에서만 생성 |
| 2 | `ARCHITECTURE_OVERVIEW.md`에 **`## 7-1. API 컨벤션`, `## 7-2. CLI 컨벤션` 신설** | 해당 스택일 때만 채움 |
| 3 | **신규 `/bootstrap-design` skill** (UI 한정, R0~R4 발굴 라운드) | DESIGN.md 채움 |
| 4 | **`/bootstrap-stack` 확장** — API/CLI 감지 시 architect-opus 단발 호출로 7-1/7-2 채움 | ARCHITECTURE 갱신 |
| 5 | **repo root `DESIGN.md` 두지 않음** (ADR-005 진입 페이지 1개 원칙) | — |
| 6 | **ADR-011** 신설 — 위 결정의 SSOT + 마이그레이션 가이드 | `docs/90-decisions/ADR-011.md` + ADR README |

### 한 줄 근거 (5가지)
1. **이름 직관성** — DESIGN.md는 시각, ARCHITECTURE는 시스템 결정. 백엔드 개발자가 헷갈리지 않는다.
2. **결합도 정합** — API envelope/error/naming은 ARCHITECTURE의 도메인 모델·외부 연동과 강결합 → 같이 읽혀야 한다.
3. **SSOT 중복 자동 해소** — 기존 DESIGN_SYSTEM "API 로깅/관측성" ↔ ARCHITECTURE "운영성" 중복이 사라진다.
4. **비-UI 프로젝트 단순성** — 백엔드/CLI 단독 프로젝트는 DESIGN.md 자체가 안 만들어진다 (파일 1개 감소).
5. **결정 프로세스 정합** — UI는 시각 의도 발굴(R0~R4) vs API/CLI는 스택 결정 부산물(단발 호출). 본질이 다른 비용을 한 파일에 묶을 이유 없음.

### 현재 상태 (요약)
- `docs/20-system/DESIGN_SYSTEM.md`는 UI / API/백엔드 / CLI 그룹별 빈 placeholder. 토큰·envelope·CLI 출력 어떤 것도 머신 리더블 형태가 아니다.
- 본문이 *"프로젝트의 인터페이스 설계 규칙... 안 쓰는 그룹은 통째 삭제"*로 광의 SSOT를 의도. 그러나 ARCHITECTURE_OVERVIEW의 "운영성"·"외부 연동"·"도메인 모델"과 책임이 중첩.
- `/bootstrap-project`의 output checklist에서 "선택 갱신 문서"라 거의 비어 통과 / `/bootstrap-stack`은 인터페이스 결정 비범위.
- 결과: 첫 컴포넌트/엔드포인트/명령어 구현 시점에야 색·envelope·flag를 즉흥 결정한다.

### 한계 진단

#### "AI 느낌"이 나는 5가지 구조적 원인 (외부 자료 합의)
LLM은 학습 코퍼스의 *median*을 재생산하는 statistical pattern matcher다. "랜딩 페이지 만들어"는 purple gradient + Inter + indigo-500 + 3-column으로 수렴한다 ([prg.sh](https://prg.sh/ramblings/Why-Your-AI-Keeps-Building-the-Same-Purple-Gradient-Website), [GenDesigns](https://gendesigns.ai/blog/ai-generated-ui-mistakes-how-to-fix)). 이를 깨려면 다음 5축을 *명시적 제약*으로 박아야 한다:

1. **레퍼런스 부재** — 좋아하는 제품의 시각 메커니즘("split-screen + all-caps sans + serif body + monochrome + 1 accent")을 텍스트로 분해하지 않으면 평균 미감으로 회귀.
2. **머신 리더블 토큰 부재** — Figma/Notion 문서는 LLM이 못 읽는다. plain markdown + fenced YAML이 표준 ([alexlavaee.me](https://alexlavaee.me/blog/lessons-learned-designing-with-ai/)).
3. **Don'ts 부재** — 외부 글의 일관된 결론: explicit 금지 목록이 LLM 정확도 향상의 *단일 최대 기여* ([designproject.io](https://designproject.io/blog/design-md-file/)).
4. **상태 변형 부재** — empty/loading/error 미정의 → 즉흥 생성 ([NN/G](https://www.nngroup.com/articles/empty-state-interface-design/), [Carbon](https://carbondesignsystem.com/patterns/empty-states-pattern/)).
5. **Motion policy 부재** — duration/easing/`prefers-reduced-motion` 없으면 컴포넌트마다 자체 transition. Material 3: 라우팅 UI 160~240ms, entrance/exit 240~360ms ([m3.material.io](https://m3.material.io/styles/motion/easing-and-duration)).

#### DESIGN_SYSTEM ↔ ARCHITECTURE 책임 중첩 (결정 결합도)

| DESIGN_SYSTEM 항목 | ARCHITECTURE에 자리 | 결합도 | 결정 시점 | 결정 비용 |
| --- | --- | --- | --- | --- |
| **UI 시각** (색·typo·spacing·motion·컴포넌트·상태) | 없음 | 스택 약결합 | 별도 라운드 | R0~R4 (시각 의도 발굴) |
| **API envelope** | "외부 연동/데이터 흐름" 인접 | 프레임워크 강결합 | bootstrap-stack | 단발 호출 |
| **API error code** | 없음 | 도메인 강결합 | bootstrap-stack | 단발 호출 |
| **API 네이밍** | "도메인 모델" 인접 | 도메인 강결합 | bootstrap-stack | 단발 호출 |
| **API 로깅/관측성** | "운영성"과 **중복** | — | — | 이미 ARCHITECTURE 책임 |
| **CLI 출력 포맷** | 없음 | 라이브러리 강결합 | bootstrap-stack | 단발 호출 |
| **CLI 플래그/명령어** | "도메인 모델" 인접 | 도메인 강결합 | bootstrap-stack | 단발 호출 |

→ UI는 시각 의도 중심이라 별도 발굴 라운드 필요. API/CLI는 스택·도메인 결정의 부산물이라 bootstrap-stack에서 같이 결정. **결정 비용·프로세스가 본질적으로 달라 한 파일에 둘 이유 없음.**

#### 4가지 가능한 구조 — A 채택

| 옵션 | 설명 | 판단 |
| --- | --- | --- |
| **A** ✓ | UI=DESIGN.md (UI 한정), API/CLI=ARCHITECTURE 7-1/7-2 흡수 | **채택** (위 5가지 근거) |
| B | DESIGN_SYSTEM 광의 SSOT 유지 | 백엔드 misnomer + 운영성 중복 잔존 |
| C | 영역별 3개 파일 분리 | 단순성 1순위 위반 |
| D | UI까지 ARCHITECTURE 흡수 | UI는 시스템 구조와 직교 → 잡음비 폭락, 50+ 섹션 비대 |

업계의 repo root `DESIGN.md` 컨벤션(Google Stitch 2026-04, [google-labs-code/design.md](https://github.com/google-labs-code/design.md))은 *위치까지* 따라가지 않는다 — ADR-001(문서 계층)·ADR-005 패턴 5(진입 페이지 1개)·ADR-010(AGENTS.md 캐노니컬 진입) 위반.

### 개선 방향

#### A. 책임 분배

| 영역 | skill | 산출물 | 트리거 |
| --- | --- | --- | --- |
| UI 시각 | **신규 `/bootstrap-design`** | `docs/20-system/DESIGN.md` | stack에 React/Next.js/Vue/Svelte/SwiftUI/Flutter/Expo/Astro/RN |
| API 컨벤션 | `/bootstrap-stack` 확장 | `ARCHITECTURE_OVERVIEW.md` `## 7-1` | API 스택 (FastAPI/Express/Spring/Django/Rails 등) |
| CLI 컨벤션 | `/bootstrap-stack` 확장 | `ARCHITECTURE_OVERVIEW.md` `## 7-2` | CLI 라이브러리 (clap/cobra/click/oclif 등) |
| API 로깅/관측성 | (변경 없음) | `ARCHITECTURE_OVERVIEW.md` `## 8 운영성` | (기존) |

`/bootstrap-design` skill 사양:
- frontmatter: `disable-model-invocation: true`, `agent: architect-opus`. fork 격리 풀고 메인 세션이 R0~R4 운전 (`discover-product` 패턴).
- 트리거: `/bootstrap-stack` 종료 출력에 "frontend 감지됨. `/bootstrap-design` 권장" 텍스트 한 줄 → 사용자가 발화. ADR-007 8단계 lifecycle 변경 없음.
- 비-UI 프로젝트는 호출되지 않음 → DESIGN.md 자체가 만들어지지 않음.

`/bootstrap-stack` 확장 사양:
- API 스택 감지 시 architect-opus 단발 sub-call로 ARCHITECTURE 7-1 채움 (envelope, status code 매핑, error 레지스트리, 네이밍, 페이지네이션, **Don'ts**).
- CLI 스택 감지 시 같은 방식으로 7-2 채움 (출력 포맷, 플래그/명령어, TTY/ANSI 정책, **Don'ts**).
- skill 본문 한 줄: "API/CLI 인터페이스 컨벤션은 ARCHITECTURE_OVERVIEW.md의 7-1/7-2에 박는다."

#### B. UI R0~R4 라운드 (`/bootstrap-design`)

**R0 — 레퍼런스 추출 + 안티-레퍼런스**
- 좋아하는 제품 1~3개 (Linear/Notion/Stripe/Vercel/Arc/Things)의 시각 메커니즘을 architect-opus 단발 sub-call로 분해 (color signature / typography pairing / density / motion 톤).
- **안티-레퍼런스 1~2개 필수**: "purple gradient generic SaaS 같지 말 것", "indigo-on-slate Tailwind 디폴트 회피".
- 입력 후보 라이브러리: VoltAgent의 [awesome-design-md](https://github.com/VoltAgent/awesome-design-md) (70+) 또는 [getdesign.md](https://getdesign.md/).

**R1 — 디자인 원칙 3~5개**
- actionable verb. 모호어("modern/clean/sleek") 금지. 예: "정보 밀도 우선", "monochrome + 1 accent", "motion은 의미 전달용만".

**R2 — 디자인 토큰 (W3C DTCG + Stitch 정렬)**
- **3-tier 토큰 모델** ([DTCG 2025.10 stable](https://www.w3.org/community/design-tokens/2025/10/28/design-tokens-specification-reaches-first-stable-version/)): primitive(`blue-100..900`) → semantic(`color/text/primary`, 컴포넌트는 이 계층만 참조) → component(`button/bg/primary`, 필요시만).
- color: brand 1 + neutral 1 + accent 1 + semantic 4 (success/warning/error/info), 12~16 hex.
- typography: 1~2 family, 4~5 size scale, modular ratio (1.125/1.25/1.333), weight pair.
- spacing: 4 또는 8 단위 base, t-shirt scale 또는 numeric.
- radius / shadow / motion (duration·easing·`prefers-reduced-motion`).
- WCAG 4.5:1 텍스트 대비 검증.

**R3 — 컴포넌트 인벤토리 + 상태 매트릭스**
- primitives (Button/Input/Text/Icon), composites (Card/Modal/Toast), patterns (Form/EmptyState/ErrorState/LoadingState).
- 각 컴포넌트마다 **상태 매트릭스 강제**: default / hover / active / focus / disabled / loading / error / empty.
- 스택별 시작점 권장 테이블 (skill이 보유):

| 스택 | 시작점 |
| --- | --- |
| React/Next.js | shadcn/ui (Radix + CSS 변수 + March 2026 shadcn-skills) |
| Vue | shadcn-vue |
| Svelte | shadcn-svelte |
| RN/Expo | Tamagui (Tamagui Studio 토큰 export 호환) |
| Flutter | ShadCN-Flutter 또는 Material 3 |
| SwiftUI | Apple HIG 토큰 직접 정의 |
| Astro | shadcn 패턴 + Astro 어댑터 |

**R4 — `docs/20-system/DESIGN.md` 저장**
- 섹션 순서를 [Stitch DESIGN.md canonical](https://github.com/google-labs-code/design.md/blob/main/docs/spec.md)에 정렬: **Overview / Colors / Typography / Layout / Elevation & Depth / Shapes / Components / Do's and Don'ts**.
- 토큰: frontmatter YAML 또는 fenced ` ```yaml ` 블록. UI 한정 단일 파일이라 frontmatter도 자연스럽다.

#### C. Don'ts 섹션 (영역별 SSOT 끝에 필수)

외부 자료 합의 ([designproject.io](https://designproject.io/blog/design-md-file/)): explicit prohibitions가 LLM 정확도 향상의 단일 최대 기여.

| 영역 | 위치 | 예시 |
| --- | --- | --- |
| UI Don'ts | DESIGN.md 끝 | 색 5색 이내 / raw hex 금지 / Inter·Roboto·Arial 디폴트 금지 / 3-column icon grid 디폴트 금지 / hierarchy는 size+weight+color 중 2축 이상 / 한 화면 primary CTA 2개 이상 금지 / 모든 motion에 `prefers-reduced-motion` 분기 / 모든 컴포넌트에 empty/loading/error 상태 정의 |
| API Don'ts | ARCHITECTURE 7-1 끝 | envelope 변경 금지 / error code ad-hoc 추가 금지 / endpoint 단/복수형 혼용 금지 / 비차단 fail이 200 OK로 가는 패턴 금지 |
| CLI Don'ts | ARCHITECTURE 7-2 끝 | JSON/표 출력 모드 일관성 위반 금지 / TTY 감지 없는 ANSI 색 금지 / interactive prompt가 `--yes`로 우회되지 않는 패턴 금지 |

agent self-check 보강:
- **builder-sonnet**: "이번 task의 인터페이스 요소(컴포넌트/엔드포인트/명령어)가 해당 SSOT의 토큰·컨벤션·Don'ts를 위반하지 않는가?"
- **validator-sonnet**: "UI: 컴포넌트가 R3 인벤토리 등록 + 상태 매트릭스 충족? / API: 7-1 envelope·error 컨벤션 준수? / CLI: 7-2 출력 포맷 컨벤션 준수?"
- **선택적 lint** (스택 정해진 후 `/stack-guard`가 권장): ESLint/stylelint로 raw 색·spacing 잡기 / API contract test로 envelope 일관성 / CLI snapshot test로 출력 일관성.

#### D. 마이그레이션 (보일러플레이트 자체 변경)

| # | 변경 | 영향 surface |
| --- | --- | --- |
| 1 | `docs/20-system/DESIGN_SYSTEM.md` → `docs/20-system/DESIGN.md` rename + 본문을 UI 그룹만 남김 (Stitch canonical 섹션 순서로 placeholder 재구성) | 1 파일 |
| 2 | `docs/20-system/ARCHITECTURE_OVERVIEW.md`에 `## 7-1. API 컨벤션`, `## 7-2. CLI 컨벤션` placeholder 신설 (각 끝에 Don'ts 가이드 한 줄) | 1 파일 |
| 3 | `docs/00-meta/STRUCTURE.md` 산출물 표 — `design system` 행 → `design (UI only)`, 위치 `docs/20-system/DESIGN.md`, 생성 주체 `/bootstrap-design`, 생성 조건 "UI 스택 포함 시", 라이프사이클 Living | 1 파일 |
| 4 | STRUCTURE.md Canonical Owner 매핑 — "UI 시각 디자인" → DESIGN.md, "API/CLI 인터페이스 컨벤션" → ARCHITECTURE 7-1/7-2 | (위와 같은 파일) |
| 5 | `.claude/skills/bootstrap-project/output-checklist.md`에서 `DESIGN_SYSTEM.md` 라인 제거 | 1 파일 |
| 6 | `AGENTS.md` "깊은 운영 원칙" 인덱스에 한 줄 추가 (UI 프로젝트일 때): `[시각 디자인](docs/20-system/DESIGN.md)` | 1 파일 |
| 7 | `.claude/skills/bootstrap-design/SKILL.md` 신설 (R0~R4 본문 + 스택별 시작점 테이블) | +1 파일 |
| 8 | `.claude/skills/bootstrap-stack/SKILL.md` 본문에 7-1/7-2 채움 책임 한 줄 추가 | 1 파일 |
| 9 | `docs/90-decisions/ADR-011-interface-decision-allocation.md` 작성 + ADR README에 한 줄 등록 | +1 파일 + README |

**총 9개 surface 변경. 신규 2 파일, rename 1, 본문 갱신 6.**

#### E. ADR-011 결정 사항 (10개)
1. `docs/20-system/DESIGN_SYSTEM.md` → `docs/20-system/DESIGN.md` rename + UI 한정.
2. ARCHITECTURE_OVERVIEW.md에 7-1 (API), 7-2 (CLI) 신설.
3. `/bootstrap-design` skill 신설 (UI 한정).
4. `/bootstrap-stack`이 7-1/7-2 채움 책임.
5. UI DESIGN.md는 Stitch DESIGN.md canonical 섹션 순서 채택.
6. 3-tier DTCG 토큰 모델.
7. 영역별 Don'ts 섹션 필수.
8. **repo root `DESIGN.md` 두지 않음** (외부 도구 자동 발견 마찰은 사용자가 root stub으로 ad-hoc 해결).
9. **운영성 ↔ 7-1 경계 가이드**: trace ID/log 포맷/관측 stack=운영성. 응답 envelope/error 레지스트리/네이밍=7-1. 흐릿한 영역(예: error 응답에 trace ID)은 *주된 결정 맥락*으로 판단 — error 일관성 핵심이면 7-1, 관측 인프라 핵심이면 운영성.
10. **charter 비목표 가이드**: "미적 트렌드 추종(neumorphism, glassmorphism, 그날의 dribbble)"을 비목표로 박을 것 권장.

### 시나리오 검증

| # | 프로젝트 | DESIGN.md | ARCHITECTURE 7-1 | ARCHITECTURE 7-2 | 흐름 |
| --- | --- | --- | --- | --- | --- |
| 1 | Next.js SaaS | 채움 (R0~R4) | 없음 | 없음 | bootstrap → `/bootstrap-design` |
| 2 | FastAPI/Spring 백엔드 | **파일 없음** | 채움 (envelope/error/Don'ts) | 없음 | bootstrap-stack(7-1 자동 채움) |
| 3 | Rust/Go CLI 도구 | **파일 없음** | 없음 | 채움 (출력/플래그/Don'ts) | bootstrap-stack(7-2 자동 채움) |
| 4 | 풀스택 (Next.js + FastAPI + admin CLI) | 채움 | 채움 | 채움 | bootstrap-stack(7-1+7-2) → `/bootstrap-design`(UI) |
| 5 | RN/Expo | 채움 (시작점 Tamagui) | (BFF 있으면 채움) | 없음 | `/bootstrap-design`이 RN 디폴트 권장 |
| 6 | 라이브러리/SDK | **파일 없음** | (Public API면 채움) | 없음 | `/bootstrap-design` 미발화 |
| 7 | UI 단독 + 외부 도구가 root DESIGN.md 요구 | 채움 | 없음 | 없음 | 시나리오 1 + 사용자가 root에 한 줄 stub 직접 추가 |
| 8 | 프로토타입 (`--fast`) | UI 토큰만 채움 | (스택에 따라) | (스택에 따라) | `/bootstrap-design --fast`: R0(1개) + R2 → R3·R4 생략 |

### 결정된 트레이드오프

기존 미결정 사항을 모두 결정으로 박음. 결정 근거는 위 본문·다른 항목 맥락(시뮬레이션 위주의 5번, 단순성 1순위의 ADR-006, 진입 페이지 1개의 ADR-005·010) 종합.

| 미결정 사항 | **결정** | 근거 |
| --- | --- | --- |
| DESIGN_SYSTEM.md 파일명 유지 vs rename | **rename → DESIGN.md** | 이름 misnomer 해소가 결정 근거 5개 중 1번. 보일러플레이트라 rename 비용은 9 surface·1회성. |
| API/CLI에도 R0~R4 같은 라운드 도입 | **지금은 도입 안 함** | 스택·도메인 강결합이라 단발 호출로 충분 (YAGNI, ADR-006). 5번 시뮬레이션이 가치 입증 시 후속 ADR로 `/bootstrap-stack --kind=api\|cli` 모드 분기 추가. |
| 운영성 ↔ 7-1 경계 흐릿 영역 처리 | **주된 결정 맥락으로 판단 (ADR-011 결정 9에 명시)** | error 응답에 trace ID 박는 결정은 → error 일관성 핵심이면 7-1, 관측 인프라 핵심이면 운영성. |
| 외부 도구 root DESIGN.md 발견 마찰 | **사용자가 root에 한 줄 stub 직접 추가** | 보일러플레이트는 sync 책임 안 짐. stub은 1줄(`See docs/20-system/DESIGN.md`)이라 drift 위험 사실상 0. |
| Stitch DESIGN.md alpha 채택 위험 | **섹션 순서만 채택, frontmatter는 옵션** | 스펙 변경 비용 미미. plain markdown + YAML이라 마이그레이션 비용 낮음. |
| shadcn/ui 권장의 lock-in | **권장이지 강제 아님** | shadcn ownership 모델(코드 복사)이라 lock-in 약함. 스택별 시작점 테이블이 RN/SwiftUI/Flutter/Vue/Astro 등 커버. |
| `/bootstrap-design` 자동 트리거 false positive | **권장 출력만, 사용자가 발화** | RN/Ink 같은 비웹 React 환경에서 자동 발화 실패 위험. ADR-007 lifecycle 변경 없이 옵션 단계로. |
| `/bootstrap-design --fast` 도입 | **도입** | R0(레퍼런스 1개) + R2(토큰)만 → R3·R4 생략. discover-product의 단계별 출구 보장 패턴과 동일. |
| `/bootstrap-design`의 fork 격리 | **풀고 메인 세션이 운전** | R0~R4가 인터랙션 길어지므로 메인이 운전. 종료 후 `/clear` 권장 (discover-product와 동일 트레이드오프). |
| ARCHITECTURE 11→13 섹션 비대화 | **수용** | 7-1/7-2 모두 "있을 때만 채움"이라 비-API/비-CLI는 빈 헤더 또는 통째 삭제. 신호 잡음비 영향 미미. 분리 유지(B)의 misnomer·중복 비용보다 net win. |

---

## 2. ADR이 보일러플레이트 자체 결정을 담는 게 맞는가

### 현재 상태
- `docs/90-decisions/ADR-001~010` 중 ADR-002, ADR-003은 placeholder (fork 후 새 프로젝트가 채울 자리), 나머지 8개(ADR-001/004~010)는 모두 **이 보일러플레이트 자체의 정책 결정**이다.
  - ADR-001: 문서 계층 6분할
  - ADR-004: 모델 별칭 정책
  - ADR-005: SSOT 패턴 5종
  - ADR-006: 단순성·Clean Code·Clean Architecture 우선순위
  - ADR-007: workitem 8단계 lifecycle
  - ADR-008: Conventional Commits
  - ADR-009: TDD default + opt-out
  - ADR-010: 다중 에이전트 도구 호환
- ADR README는 보일러플레이트 결정과 새 프로젝트 결정을 같은 표에 섞어 둔다.

### 한계 진단 (사용자 의문 그대로 풀이)
- **혼동 지점 1 — "이게 내 프로젝트의 결정인가, 보일러플레이트의 결정인가?"**: fork 직후 사용자는 ADR-001~010을 보면서 "이 문서들이 내 새 프로젝트의 의사결정인가, 아니면 이 템플릿을 만든 사람의 의사결정인가" 직관적으로 구분이 안 된다.
- **혼동 지점 2 — supersede 권한**: 사용자가 ADR-006(단순성 정책)을 자기 프로젝트에서 뒤집고 싶다면, "보일러플레이트가 박은 ADR을 supersede해도 되는가?" 정신적 마찰이 생긴다.
- **혼동 지점 3 — 새 ADR 번호의 충돌**: 새 프로젝트가 첫 ADR을 쓸 때 "ADR-011"부터 시작해야 하나, 새로 ADR-001부터 시작해야 하나? 현재 가이드는 명시 안 됨.
- **장점도 있음**: 보일러플레이트 자체 결정을 ADR로 박아두면 fork된 사용자가 "왜 이 정책이 있는가"를 추적 가능 — 단순 한 줄 가이드보다 권위가 높다.

### 개선 방향 (3가지 옵션)
1. **옵션 A — 라벨링만 추가 (가성비 최고)**:
   - `docs/90-decisions/README.md`에 "Boilerplate ADR (fork 후 supersede 가능)" 섹션과 "Project ADR (이 fork의 결정)" 섹션을 분리.
   - ADR-001/004~010 frontmatter에 `scope: boilerplate` 같은 필드 추가.
   - fork 후 새 프로젝트는 ADR-100부터 시작 (또는 boilerplate scope를 그대로 두고 자기 결정만 추가).
2. **옵션 B — 디렉터리 분리 (구조 변경)**:
   - `docs/90-decisions/boilerplate/ADR-001~010.md` (보일러플레이트 결정, fork 후 immutable이 디폴트)
   - `docs/90-decisions/project/ADR-001~.md` (fork 후 새 프로젝트가 채움)
   - 단점: SSOT/단순성과 충돌. ADR-005의 "정책=ADR" 패턴 재서술 필요.
3. **옵션 C — 메타 ADR-000 (가장 가벼움)**:
   - `ADR-000-boilerplate-decision-policy.md`를 만들어 "이 디렉터리의 ADR-001/004~010은 보일러플레이트의 자체 결정이며, fork된 프로젝트는 supersede ADR을 새로 만들어 자기 정책으로 덮을 수 있다"를 명시.
   - 새 프로젝트는 ADR-002, ADR-003을 채우거나 ADR-011부터 시작.

### 트레이드오프 / 고민
- **권장은 옵션 A + 옵션 C 결합**: 라벨링으로 시각적 구분 + ADR-000 메타로 권위 있는 정책 명시. 디렉터리 분리(옵션 B)는 단순성 1순위 위반 가능성이 커 보류.
- **fork 후 ADR 번호 정책**: "보일러플레이트 ADR 번호는 그대로 두고, 새 프로젝트는 ADR-100부터" 또는 "초기화 시 보일러플레이트 ADR을 별도 폴더로 옮기는 reset script" 중 선택. 후자는 다운스트림 사용자가 정책 컨텍스트를 잃을 위험.
- **사용자 의문에 대한 직접 답**: 보일러플레이트 결정을 ADR로 박는 것은 **추적성과 권위 측면에서 정당하다** — 다만 fork 사용자의 인지 부담을 줄이려면 라벨링과 메타 ADR이 필요하다.

---

## 3. 문서 아키텍처 — AGENTS.md slim cap + 00-meta 9→5 + Diátaxis 모드 라벨

> **TL;DR**: 본 보일러플레이트는 ADR-005 SSOT를 이미 박았으므로 *불필요한 진입 surface 4건*만 흡수하면 된다. **P0 3개**(AGENTS.md 100줄 hard cap / 00-meta 9→5 흡수 / `/review-doc`에 모드 mismatch 검출), **P1 1개**(`/stabilize-milestone` SSOT drift 점검), **명시적 No 2개**(00-meta 폴더명 변경 / `_templates/` 통합). **새 ADR 3개**(ADR-011 / ADR-012 / ADR-013).
>
> **본 섹션의 위치** — *진입 표면 정합과 slim cap*에만 초점. 6번에 흩어져 있던 "AGENTS.md 슬림화"는 본 섹션으로 이관됐다(진입 surface는 한 곳에서 관리).

### 핵심 결정 (한눈)

| 분류 | # | 결정 | 핵심 산출물 | ADR |
|------|---|------|-----------|-----|
| **P0** | A | AGENTS.md 100줄 hard cap + 80줄 soft cap | ADR-011 본문 + `/review-doc`에 cap 점검 1줄 | ADR-011 신설 |
| **P0** | B | docs/00-meta 9문서 → 5문서 흡수 | 4 파일 삭제(아래 표) + AGENTS.md "깊은 운영 원칙" 6→4 링크 | ADR-012 신설 |
| **P0** | C | 각 docs/00-meta/*.md 첫 줄에 Diátaxis 모드 라벨 + `/review-doc` 모드 mismatch 검출 | 9 파일 1줄씩 추가 + review-doc skill 본문 2줄 추가 | ADR-012 본문 |
| **P1** | D | `/stabilize-milestone`에 SSOT drift 점검 단계 추가 | stabilize-milestone skill 본문 1단락 (5단어+ 시퀀스가 2 파일에 본문 정의로 등장하면 보고) | ADR-013 신설 |
| **P1** | E | AGENTS.md 기존 5개 핵심 행동 규율 trace-to-failure retrospective | AGENTS.md PR 본문에 각 줄별 근거 ADR/사건 1회 정리 | (ADR-011 본문) |
| **No** | F | 00-meta 폴더명 변경(`operations/`/`process/`) | 영구 보류 — Diátaxis 공식 가이드 *"폴더명보다 내용의 mode가 강한 신호"* | (cut) |
| **No** | G | `_templates/` 분산 통합 | 영구 보류 — fork 사용자가 처음 1회만 보는 영역, 가성비 낮음 | (cut) |

### 근거 (외부 + 내부)

| 근거 | 출처 | 본 절의 결정 |
|------|------|-------------|
| AGENTS.md 100~150줄 sweet spot, 150줄 초과 시 정확도 reverse + inference 비용 +20~23% | Augment Code 2,500 repo 분석 | 100줄 hard cap 채택 (*결정 A*) |
| 좋은 AGENTS.md = 명령어 최상단 / 코드 스니펫 / 3-tier 경계 / 6 핵심 영역(commands·testing·structure·style·git·boundaries) | GitHub Copilot 2,500+ repo 분석 | 본 보일러플레이트 35줄은 이미 정합. cap만 박으면 됨 |
| AGENTS.md ≤ 60줄 *"pilot's checklist"* + Ratchet Principle | Addy Osmani | 60줄은 단순성·TDD·SSOT·lifecycle·multi-tool 5층 surface에 빡빡 → 100을 채택. Ratchet은 6번 Pillar 1로 이관 |
| Codex 32 KiB 누적 cap | Codex 공식 | 100줄 = Codex limit의 ~10%, 안전 마진 |
| Diátaxis는 *폴더 X, 모드 라벨 O* | diataxis.fr 공식 가이드 | 폴더 신설 안 함, 첫 줄 라벨만 (*결정 C*) |
| ADR-005 SSOT 패턴 1 (정의 1곳, 다른 곳은 링크) | 본 보일러플레이트 | 단발 위임 nav 4 파일은 흡수 정당 (*결정 B*) |
| Spotify × Anthropic *"good set of CLAUDE MD + good set of skills"* | engineering.atspotify.com | 단순한 설정의 재현성 = 본 절의 방향 |

### 구체적 변경 사항

**파일 변경 표** — 결정 B (00-meta 흡수):

| 현재 파일 | 라인 | 흡수 대상 | 결과 |
|----------|------|---------|------|
| `docs/00-meta/TEMPLATE_GUIDE.md` | 52 | *문서 계층 6분할*은 ADR-001에 이미 있음(중복 제거). *네이밍 규칙* 4줄은 `STRUCTURE.md`의 "절차" 섹션 위로 흡수 | 삭제 |
| `docs/00-meta/LOCAL_SETTINGS_EXAMPLE.md` | 21 | 전체를 `GUARDRAILS_STRATEGY.md`의 "local 자동화 권장 원칙" 단락 아래로 7줄 압축 흡수 | 삭제 |
| `docs/00-meta/BOOTSTRAP_PROMPT_EXAMPLES.md` | 36 | 전체를 `NEW_PROJECT_CHECKLIST.md`의 "1. 저장소 복제 직후" 단계에 인라인 코드블록으로 흡수 | 삭제 |
| `docs/30-workitems/README.md` | 25 | `STRUCTURE.md` 산출물 표가 이미 SSOT. 흡수 불필요 | 삭제 |

**최종 docs/00-meta** = STRUCTURE / WORKFLOW / AGENT_EXECUTION_STRATEGY / GUARDRAILS_STRATEGY / NEW_PROJECT_CHECKLIST / GLOSSARY = **6문서** (GLOSSARY 빈 placeholder 포함). AGENTS.md "깊은 운영 원칙" 인덱스 6→4 링크로 슬림화.

**Diátaxis 모드 라벨** — 결정 C: 각 docs/00-meta/*.md 첫 줄(`# 제목` 다음 줄)에 다음 형식 1줄 추가:
```
> 모드: Reference (산출물 인벤토리)
```
혼재 문서는 `> 모드: Reference + How-to (구조 정의 + 갱신 절차)` 형식. **`/review-doc` 본문 보강** — 모드 mismatch 발견 시 `IMPROVEMENT_GUIDE.md`에 **P1 severity + Q3 (Inadvertent+Reckless) quadrant** 2축 라벨로 보고 (severity × quadrant 직교 — §5-D 정합).

**AGENTS.md cap 운영 절차** — 결정 A:
- 100줄 초과 PR 발생 시 `/review-doc` 또는 reviewer가 P0(Q1)로 분류.
- 새 정책은 항상 ADR로(ADR-005 패턴 4 정합), AGENTS.md surface는 1줄 단락만.

**SSOT drift 점검** — 결정 D: `/stabilize-milestone` 본문에 단계 6과 7 사이 추가:
- 5단어+ 동일 시퀀스가 2 파일에 *본문 정의*로 등장 + 둘 중 하나가 ADR이 아니면 `IMPROVEMENT_GUIDE.md`에 **P1 severity + Q1 (Deliberate+Reckless) quadrant** 2축 라벨로 보고 (drift는 SSOT 패턴을 알면서도 방치한 경우 = Deliberate. 잊고 있던 경우면 reviewer가 Q3로 재분류).
- 단순 링크·요약은 false positive 제외.

### ADR 분할 — *결정 1개에 ADR 1개* (ADR-005 패턴 4 정합)

| ADR | 결정 |
|-----|------|
| **ADR-011** | AGENTS.md 100줄 hard cap (단일 결정) |
| **ADR-012** | docs/00-meta 흡수(9→5) + Diátaxis 모드 라벨 도입 (한 흐름 — *문서 정리* 1 ADR) |
| **ADR-013** | `/stabilize-milestone`에 SSOT drift 점검 단계 추가 (단일 결정) |

### 결정 근거 — 왜 이 형태인가

| 결정 | 근거 |
|------|------|
| A. 100줄 hard cap | Augment 100~150줄 *외부실증* + Codex 32 KiB의 ~10% *안전 마진*. ADR-005 패턴 4 정합(정책=ADR이므로 본문 추가 ergonomics 손실 없음). |
| B. 9→5 흡수 | ADR-005 패턴 1(정의 1곳, 다른 곳은 링크)의 정합성 검증 PR. 단발 위임 nav 4건만 흡수, 핵심 책임 문서(STRUCTURE/WORKFLOW/AGENT_EXECUTION_STRATEGY/GUARDRAILS_STRATEGY)는 SRP로 유지. |
| C. 모드 라벨 + reviewer hook | Diátaxis 가이드 *"폴더 X, 모드 O"* 정렬. *라벨링만으로는 노이즈*이므로 `/review-doc` 통합이 필수 — 노이즈가 신호로 변환. |
| D. drift 점검 자동화 | ADR-005가 SSOT 패턴을 박았지만 *시간이 지나면서* 같은 사실이 다시 2곳에 박힐 가능성을 점검하는 절차가 부재. stabilize에 추가가 자연(매 마일스톤 점검). |
| E. trace-to-failure retrospective | Osmani Ratchet 정신상 기존 5개 행동 규율도 *왜 박혔는가*를 사후 정리하면 본문이 자기 정화 — 이후 추가 시도가 *왜*에서 막힘. 6번 Pillar 1과 정합. |
| F. 00-meta 폴더명 변경 No | Diátaxis 가이드 *"user type/폴더명보다 내용의 mode가 강한 신호"* + cross-reference 일괄 변경 비용 큼. 효용은 *처음 fork 시 1회* 직관 향상에 한정. |
| G. `_templates/` 통합 No | charter·workitems 분산이 헷갈리는 빈도 = 처음 1회. fork 사용자 동선이 처음 1회 후 재방문 거의 없음. ADR-006 단순성 — 비용 < 효용 시 보류. |

### 잔여 모니터링

- AGENTS.md 100줄 cap이 fork된 다운스트림에서 어떻게 작동하는지 — fork 사례 0건이라 *[가설]* 라벨. 5번 P0-Gate 시뮬레이션이 첫 [관측됨] 데이터 회수 경로.
- 흡수 후 `/review-doc` 의 모드 mismatch 검출 false positive 비율 — 첫 마일스톤 stabilize에서 측정 후 6번 Pillar 1 Ratchet 적용 범위 한정 표 갱신 후보.

---

## 4. plan-workitem 강화 + Claude Code plan 모드 lifecycle 비범위

> **TL;DR**: 두 흐름이다. **(1)** Claude Code 빌트인 plan 모드를 lifecycle에서 *분리* — 설정 1줄·디렉터리 1개·문서 4곳을 정리하고 본질 효용(think-before-edit)은 `/implement-workitem` 1줄로 흡수(ADR-024). 근거: ADR-010(Codex 호환) + ADR-006(단순성) + 외부 버그 [#19537](https://github.com/anthropics/claude-code/issues/19537). **(2)** `/plan-workitem`을 *측정 가능한 분해기*로 승격 — TASK_TEMPLATE에 sizing 한계·AC verb whitelist·Given-When-Then 강제·`## 9. 의존성`을 박고, planner skill에 charter 정합 self-check + architect-opus 호출 신호 4종을 추가(ADR-026).
>
> **P0 5개**(plan 모드 분리 4건 + TASK_TEMPLATE schema 강화), **P1 2개**(charter 정합 self-check / architect-opus 신호), **명시적 No 4개**(plan 모드 lifecycle 통합 / 2-pass planning / risk·effort 추정 / `/plan-workitem` rename).
>
> **본 섹션의 위치** — *plan 단계의 결정*만 다룬다. 실제 RGR 구현(/implement-workitem)은 6번. lifecycle 8단계 중 3단계(plan)와 4단계(implement)의 *경계* 자리.

### 핵심 결정 (한눈)

| 분류 | # | 결정 | 핵심 산출물 | ADR |
|------|---|------|-----------|-----|
| **P0** | A | `.claude/settings.json`에서 `plansDirectory` 라인 삭제 → 빌트인 디폴트(`~/.claude/plans/`) 회귀 | settings.json 1줄 삭제 | ADR-024 본문 |
| **P0** | B | `docs/30-workitems/plans/` 삭제 + STRUCTURE.md "plan (Claude Code Plan 모드)" 행 삭제 + `30-workitems/README.md` `plans` 줄 삭제 | 디렉터리 1개 + 2 문서 3줄 정리 | ADR-024 본문 |
| **P0** | C | AGENTS.md에 plan 모드 자율성 1단락 명시 | "Claude Code plan 모드(Shift+Tab)는 사용자 자율. lifecycle은 의존하지 않는다." | ADR-024 본문 |
| **P0** | D | `/implement-workitem`에 think-before-edit 1줄 흡수 — Red phase 직전 첫 단락에 "어떤 테스트를 어떤 순서로 작성할 것인가" 1~3문장 명시 | implement-workitem skill 본문 1줄 | ADR-024 본문 |
| **P0** | E | TASK_TEMPLATE schema 강화: sizing 한계 3종 + AC verb whitelist + Given-When-Then 강제 + `## 9. 의존성` 추가 | TASK_TEMPLATE 4 단락 갱신 + planner skill self-check 3줄 | ADR-026 신설 |
| **P1** | F | planner skill에 charter 정합 self-check 1단락 — `## 비목표` 키워드 매칭 + feature ↔ milestone "포함되는 기능" 매핑 | planner skill 본문 1단락(2문장) | ADR-026 본문 |
| **P1** | G | architect-opus 호출 권장 신호 4종 감지 시 출력 마지막 1줄 텍스트 제안 (자동 호출 X — ADR-007 정합) | planner skill 본문 1단락 + 신호 4개 명시 | ADR-026 본문 |
| **No** | H | plan 모드 lifecycle 통합(이전 안: 정합 정책 4축 추가) | 영구 보류 — ADR-010 위반 + ADR-006 단순성 위반. plan 모드 본질이 *단일 task 도구*이므로 결정 D로 동등 효용 확보 | (cut) |
| **No** | I | 2-pass planning (reviewer pass) | 영구 보류 — 토큰 2배 + 5번 stabilize-milestone reviewer가 누락 검토 책임 보유. 단계 중복 | (cut) |
| **No** | J | TASK_TEMPLATE `## 10. 추정` (risk·effort) 선택 섹션 | 영구 보류 — 보일러플레이트가 정확도 보장 불가. ADR-006 YAGNI. 사용자 프로젝트 단위에서 자유 추가 | (cut) |
| **No** | K | `/plan-workitem` → `/decompose-workitem` rename | 영구 보류 — plan 모드 비범위 결정 후 충돌 = 자동완성 표면 1건. cross-ref 일괄 변경 비용 > 효용. skill `description`에 "Claude Code plan 모드와 다름 — milestone/feature/task 분해기" 1줄로 충분 | (cut) |

### 근거 (외부 + 내부)

| 근거 | 출처 | 본 절의 결정 |
|------|------|-------------|
| Claude Code plan 모드의 본질은 *permission mode + edit-file로 plan 파일 자체 편집*이라는 단순 메커니즘 — 분해기가 아니라 단일 task scope 도구 | [lucumr.pocoo.org/2025/12/17](https://lucumr.pocoo.org/2025/12/17/what-is-plan-mode/) | lifecycle 비범위 (*결정 A~D, H*) |
| `plansDirectory` 프로젝트 설정이 무시되고 글로벌 `~/.claude/plans/`로 fallback되는 빌드 존재 (closed bug, recurring) | [anthropics/claude-code#19537](https://github.com/anthropics/claude-code/issues/19537) | 외부 미해결 버그에 lifecycle 묶지 않음 (*결정 A*) |
| Codex CLI는 plan 모드 없음 — 동등 흐름 보장 불가 | 본 보일러플레이트 ADR-010 | plan 모드 의존 시 Codex 사용자 흐름 깸 (*결정 A~D*) |
| 단순성·YAGNI 1순위 — 정합 비용 4축(이름/위치/핸드오프/fallback)이 0축으로 감소 | 본 보일러플레이트 ADR-006 | 통합 거부 (*결정 H*) |
| TDD Red phase는 *기계적으로 파싱 가능한* AC 전제 — measurable verb whitelist는 LLM TDD 실패의 단일 최대 원인 해소 | 본 보일러플레이트 ADR-009 + 외부 SDD 자료([Fowler SDD analysis](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html)) | Given-When-Then 강제 + verb whitelist (*결정 E*) |
| skill `allowed-tools` frontmatter가 read-only 강제를 이미 보장 (planner: Read/Glob/Grep/Write/Edit. validator: 더 좁다) | 본 보일러플레이트 .claude/skills/* | plan 모드의 read-only enforcement 효용은 이미 흡수 (*결정 D*) |
| 스킬 간 흐름은 *자동 호출이 아니라 텍스트 제안 → 사용자/메인이 발화* | 본 보일러플레이트 ADR-007 | architect-opus 자동 호출 거부, 텍스트 제안만 (*결정 G*) |

### 구체적 변경 사항

**Plan 모드 분리** — 결정 A~D:

```diff
# .claude/settings.json
{
  "$schema": "...",
  "model": "opus",
  "defaultMode": "acceptEdits",
- "plansDirectory": "./docs/30-workitems/plans",
  "permissions": { ... }
}
```

```diff
# 디렉터리
- docs/30-workitems/plans/.gitkeep   (디렉터리 통째 삭제)
```

```diff
# docs/00-meta/STRUCTURE.md 산출물 표
- | plan (Claude Code Plan 모드) | `docs/30-workitems/plans/` | Claude Code Plan 모드 | ephemeral |
```

```diff
# docs/30-workitems/README.md
- - `plans`: Claude Code Plan 모드가 자동 생성하는 실행 계획. 경로는 `.claude/settings.json`의 `plansDirectory`가 canonical이다(현재 `./docs/30-workitems/plans`).
```

```diff
# AGENTS.md (단순성·YAGNI 단락 아래에 신설)
+ ## Claude Code plan 모드
+ Claude Code의 빌트인 plan 모드(Shift+Tab)는 사용자 자율 도구다. 이 보일러플레이트의 lifecycle은 plan 모드를 의무화하지 않으며 산출물 경로(`plansDirectory`)도 강제하지 않는다. Codex 사용자도 동등한 흐름을 갖는다(ADR-010, ADR-024).
```

```diff
# .claude/skills/implement-workitem/SKILL.md (Red phase 진입 직전 단락에 1줄)
+ Red phase 진입 직전, 출력의 첫 단락으로 "이 task에서 어떤 테스트를 어떤 순서로 작성할 것인가"를 1~3문장으로 명시한다(plan 모드 의존 없이 think-before-edit 규율 확보).
```

**TASK_TEMPLATE schema 강화** — 결정 E:

```diff
# docs/30-workitems/_templates/TASK_TEMPLATE.md ## 6. Acceptance Criteria
+ <!-- AC는 Given-When-Then *형식 강제*. measurable verb whitelist만 사용:
+      허용: returns, displays, persists, rejects, emits, responds with, contains, matches
+      금지: works, looks good, is correct, is fine, handles, supports
+      AC 3개 이하 권장(4개 이상이면 task 분리 — ADR-026 sizing 한계).
+      위반 시 planner가 분해 거부, builder-sonnet이 RGR Red phase 진입 거부. -->
- - AC-1:
- - AC-2:
+ - AC-1 [Given] ... [When] ... [Then] ...
+ - AC-2 [Given] ... [When] ... [Then] ...

+ ## 9. 의존성
+ <!-- 형식: `- T-002: T-001의 X 정의 후 시작 가능`. 비어 있으면 병렬 가능으로 간주. -->
```

```diff
# .claude/skills/plan-workitem/SKILL.md "반드시 수행할 일" 단락에 추가
+ 8. **분해 후 sizing self-check** — 다음 3 한계 중 하나라도 초과 시 추가 분해 권장 텍스트를 출력에 명시:
+    - 1 task = 1 RGR 사이클(반나절~1일).
+    - AC 3개 이하.
+    - 변경 예정 파일(TASK_TEMPLATE `## 4-1`) 5개 이하.
+ 9. **AC 형식 강제** — 모든 AC는 Given-When-Then + measurable verb whitelist(TASK_TEMPLATE 주석 참조). 위반 시 planner가 분해 거부.
+ 10. **task 의존성 채움** — TASK_TEMPLATE `## 9. 의존성`을 분해 시 명시. 병렬 가능 task는 비워둔다.
```

**planner skill 행동 강화** — 결정 F·G:

```diff
# .claude/skills/plan-workitem/SKILL.md "반드시 지킬 원칙" 단락 아래 추가
+ ## 정합성 self-check (분해 직후 1회 실행)
+ - charter `## 비목표` 단락 키워드와 분해된 feature/task를 매칭. 위반 의심 시 출력의 "남은 미결정 사항"에 명시.
+ - feature 범위가 상위 milestone `## 3. 포함되는 기능`에 매핑되는지 확인. 매핑 실패 시 동일 위치에 명시.
+
+ ## architect-opus 호출 권장 신호 (감지 시 텍스트 제안만, 자동 호출 금지 — ADR-007)
+ 다음 4 신호 중 하나라도 감지되면 출력 마지막에 `architect-opus 호출 권장: <이유>` 1줄 추가:
+ 1. 새 모듈 디렉터리 생성(`src/<new>/` 또는 동등 경로).
+ 2. charter `## 제약`에 없는 새 외부 의존(npm/pip/cargo) 추가.
+ 3. ARCHITECTURE_OVERVIEW.md `## 3-1. 레이어 경계` 변경.
+ 4. "패턴 변경" / "새 boundary" / "도메인 경계" 키워드 등장.
```

**plan summary 표** — 결정 E의 일부. planner 출력 마지막에 다음 매트릭스 1개 추가:

```
| Milestone | Feature | Task  | AC 수 | 의존성  |
|-----------|---------|-------|-------|--------|
| M1        | F-001   | T-001 | 2     | -      |
| M1        | F-001   | T-002 | 3     | T-001  |
```

### ADR 분할 — *결정 1개에 ADR 1개* (ADR-005 패턴 4 정합)

| ADR | 결정 |
|-----|------|
| **ADR-024** | Claude Code plan 모드 lifecycle 비범위 — 사용자 자율 도구로 분리. settings 1줄 + 디렉터리 1개 + 문서 4곳 정리 + AGENTS.md 1단락 + /implement-workitem 1줄 흡수 (한 흐름 — *분리 결정* 1 ADR) |
| **ADR-026** | plan-workitem 강화 — TASK_TEMPLATE schema 4종(sizing/AC verb/Given-When-Then/의존성) + planner skill 행동 2종(charter 정합/architect-opus 신호) (한 흐름 — *분해 deterministic화* 1 ADR) |

### 결정 근거 — 왜 이 형태인가

| 결정 | 근거 |
|------|------|
| A. plansDirectory 삭제 | #19537 외부 미해결 버그 + ADR-010 호환(Codex 무관). 보일러플레이트가 동작 보장 가능한 영역만 SSOT로 박는 ADR-005 정신. |
| B. plans/ 디렉터리 + 문서 정리 | workitem 트리는 *Living Doc only*가 ADR-005 라이프사이클 모델. ephemeral은 사용자 홈으로 자연 분리. |
| C. AGENTS.md 1단락 | 명시적 No이므로 진입 surface에서 한 번 박는다. 3번-A의 AGENTS.md 100줄 cap과 정합 — short surface 1단락이면 충분. |
| D. /implement-workitem think-before-edit | plan 모드의 본질 효용(read-only + 사고 우선)은 skill `allowed-tools` + prompt 1줄로 동등 확보. 외부 도구 의존 0. |
| E. TASK_TEMPLATE schema | builder-sonnet의 Red phase는 Given-When-Then이 *기계적으로 파싱 가능*해야 deterministic 동작. measurable verb whitelist는 외부 SDD 자료 합의(Fowler) — vague AC가 LLM TDD 실패의 단일 최대 원인. |
| F. charter 정합 self-check | 보일러플레이트가 SSOT를 박았지만 *위반 검출은 사람*에 의존하던 빈자리. self-check 2문장은 ADR-006 단순성 위반 아님(planner가 이미 charter 읽음 — 추가 토큰 0). |
| G. architect-opus 신호 텍스트 제안 | 자동 호출은 false positive 시 비용(opus) 큼. ADR-007의 "스킬 간 흐름은 자동 호출이 아니라 텍스트 제안 → 사용자/메인이 발화" 정합. 5번-H(stabilize 단계 architect-opus 신호)와 동일 정신. |
| H. plan 모드 lifecycle 통합 No | ADR-010 위반 + ADR-006 단순성 4축 정합 정책 추가. plan 모드 본질이 *단일 task 도구*이므로 lifecycle 의존 가치 없음(*결정 D로 동등 효용 확보*). |
| I. 2-pass planning No | 토큰 2배 + 5번 stabilize-milestone reviewer가 누락 검토 책임 보유 → 단계 중복. ADR-006 단순성. |
| J. risk·effort 추정 No | 보일러플레이트가 추정 정확도 보장 불가. 사용자 프로젝트 단위에서 자유 추가 가능. ADR-006 YAGNI. |
| K. rename No | plan 모드 비범위 결정 후 충돌 = 자동완성 표면 1건. cross-ref 일괄 변경(ADR-007/AGENTS.md/WORKFLOW.md/skill 디렉터리/README) > 효용. skill `description`에 "Claude Code plan 모드와 다름 — 분해기" 1줄로 충분. |

### 잔여 모니터링

- **Given-When-Then 강제 후 첫 마일스톤의 task 분해 거부율** `[가설→데이터 트리거]` — planner의 vague AC 거부율이 50% 초과 시 verb whitelist 갱신 필요. 5번 P0-Gate 시뮬레이션이 첫 데이터 회수.
- **architect-opus 신호 4종의 false positive 비율** `[가설]` — 첫 마일스톤 stabilize에서 측정. 50% 초과 시 신호 정밀도 강화(예: AST 기반 모듈 디렉터리 감지).
- **plan 모드 비범위 결정의 사용자 영향** `[가설]` — fork 다운스트림 사례 0건. 첫 fork 사용자 피드백이 [관측됨] 데이터 회수 경로. 재검토 트리거 3종: (a) Codex가 동등 plan 모드 도입, (b) plan 모드가 milestone/feature/task 계층 분해 제공, (c) #19537 fix·안정화.

---

## 5. stabilize-milestone — Graduation Contract + Tech Debt Quadrant + 4지표 + Dogfood Gate

> **TL;DR**: stabilize-milestone은 *데이터 누적*만 하고 *처리 메커니즘*이 없는 상태다. **P0-Gate 1개**(보일러플레이트 자체 dogfood 시뮬레이션 — 다른 P0 채택의 *선행 조건*), **P0-Post-Gate 4개**(milestone graduation checklist / retrospective 섹션 / IMPROVEMENT_GUIDE Fowler quadrant 직교 추가 / `/stabilize-milestone` graduation pre-check 단계), **P1 3개**(METRICS.md 4지표 / carry-over 자동 등록 플래그 / architect-opus auto-escalation 신호), **명시적 No 3개**(release-level DoD / 자체 발명 매트릭스 / auto-escalation 자동 호출). **새 ADR 4개**(ADR-014~017).
>
> **본 섹션의 위치** — *milestone 단위 sprint contract*에 초점. graduation 기준이 명문화되지 않은 빈 자리를 외부 표준 4종(Anthropic sprint contract / Atlassian multi-level DoD / Fowler quadrant / DORA 원칙)으로 채운다.

### 핵심 결정 (한눈)

| 분류 | # | 결정 | 핵심 산출물 | ADR |
|------|---|------|-----------|-----|
| **P0-Gate** | A | 보일러플레이트 자체 dogfood 시뮬레이션 (todo CLI / 8단계 lifecycle 1회 통과) | `docs/40-validation/SIMULATION_RUN.md` (Record) + 성공 기준 3개 | ADR-017 신설 |
| **P0** | B | MILESTONE_TEMPLATE `## 5. 완료 기준` graduation checklist 강화 | 5+1 항목(아래 산출물 표) + `/plan-workitem` 자동 채움 | ADR-014 신설 |
| **P0** | C | MILESTONE_TEMPLATE `## 8. 회고` 추가 (4 항목) | stabilize 자동 채움 + 사용자 검토 1라운드 | ADR-014 본문 |
| **P0** | D | IMPROVEMENT_GUIDE에 Fowler quadrant *직교* 추가 (P0/P1/P2 severity는 유지) | severity × quadrant 2축 분류 (아래 형식) | ADR-015 신설 |
| **P0** | E | `/stabilize-milestone`에 graduation pre-check 단계 추가 + `--dry-run` 플래그 | skill 본문 단계 1·2 사이 삽입 | ADR-014 본문 |
| **P1** | F | METRICS.md (Living Doc) 신설 + 4지표 + trend 악화 신호 | `docs/40-validation/METRICS.md` + stabilize 매 회 1줄 append | ADR-016 신설 |
| **P1** | G | `--apply-carryover` 플래그 (default OFF, 명시 발화만 자동 등록) | stabilize skill 본문 1단락 | ADR-014 본문 |
| **P1** | H | reviewer가 architect-opus 호출 권장 신호 4종 감지 시 텍스트 제안 | stabilize 출력 1줄 추가 (자동 호출 X) | (skill amend) |
| **No** | I | Release-level DoD 도입 | stabilize 출력에 자연 흡수(carry-over 0건 + ADR 후보 0건 = release-ready) | (cut) |
| **No** | J | 자체 발명 (영향, 노력) 매트릭스 | Fowler quadrant + P0/P1/P2 severity로 충분. 외부 표준 학습 비용 0 | (cut) |
| **No** | K | architect-opus auto-escalation *자동 호출* | ADR-007 정책 정합 — 텍스트 제안만, 자동 호출 없음 | (cut) |

### 근거 (외부 + 내부)

| 근거 | 출처 | 본 절의 결정 |
|------|------|-------------|
| sprint contract: *"the generator and evaluator negotiated a sprint contract: agreeing on what 'done' looked like... before any code was written"* | Anthropic 3-agent harness ([anthropic.com/engineering/harness-design-long-running-apps](https://www.anthropic.com/engineering/harness-design-long-running-apps)) | milestone-level sprint contract = graduation checklist (*결정 B*) |
| Multi-level DoD = story / sprint / release 3계층 | Atlassian / Agile Alliance | task AC = story-level (이미 있음), milestone graduation = sprint-level (*결정 B*). release-level은 stabilize 출력에 자연 흡수 (*결정 I cut*) |
| Tech Debt Quadrant 2축 분류 (Deliberate/Inadvertent × Reckless/Prudent) | Martin Fowler ([martinfowler.com/bliki/TechnicalDebtQuadrant](https://martinfowler.com/bliki/TechnicalDebtQuadrant.html)) | 외부 표준이 이미 있음 — 자체 발명 (*결정 J cut*) |
| 적은 수의 잘 정의된 지표(4개) > 많은 모호한 지표 | DORA 4 metrics 원칙 ([dora.dev](https://dora.dev/guides/dora-metrics/)) | METRICS.md 4지표(AC 통과율 / P0 수 / opt-out 비율 / repair 평균). 단 DORA *지표*는 1:1 매핑 X — 정직한 framing(*결정 F*) |
| *"Implement to learn"* — 짓고 나서야 무엇이 정말 필요한지 안다 | Drew Breunig ([dbreunig.com 2026-05](https://www.dbreunig.com/2026/05/04/10-lessons-for-agentic-coding.html)) | 시뮬레이션 = gate (*결정 A*) — 데이터 없는 정책은 추측 |
| ADR-006 *"추측에 기반한 추상화 금지"* | 본 보일러플레이트 | 시뮬레이션이 *현 IMPROVE-LIST의 P0 가설을 데이터로 환원* (*결정 A의 근거*) |

### 구체적 변경 사항

#### 결정 A — P0-Gate (시뮬레이션 dogfood)

**왜 todo CLI인가** — 4-연산 calculator는 stateless·single-function이라 lifecycle 단계를 충분히 굴리지 못한다. todo CLI는 CRUD 4종 + persistence가 들어가 8단계 lifecycle의 *모든 결정 포인트*(AC 분해 / E2E / 회귀)를 자연스럽게 자극한다. 본 보일러플레이트의 일반 사용 케이스(SaaS / API / CLI)에 더 가까움.

```
실행 절차
1. mock charter / architecture / M1 / F-001 / T-001~T-003 생성 (스택은 사용자 환경 자연스러운 1종 — Node/Python/Rust 중)
2. 8단계 lifecycle 1회 통과:
   discover → bootstrap-project → bootstrap-stack → stack-guard
   → plan-workitem → implement-workitem → validate-workitem
   → finalize-workitem → stabilize-milestone
3. 각 단계의 입출력이 다음 단계 입력으로 의도대로 들어가는지 명시적 체크포인트에서 검증

성공 기준 (3개 모두 충족 시 gate 통과)
- 사용자 개입(수동 편집) ≤ 1회
- 산출물 placeholder 충원율 ≥ 80% (빈 채로 통과한 섹션 ≤ 20%)
- graduation pre-check 미통과 사유 ≤ 2개

기록 산출물
docs/40-validation/SIMULATION_RUN.md (Record)
- 단계별 1줄: "어디서 끊겼는지 / 사용자 개입 필요한 지점 / 산출물 비어 있는 지점"

실패 처리 — gate 정신
- gate 미통과 시 본 IMPROVE-LIST의 다른 P0 채택을 보류
- 발견된 lifecycle 깨짐을 ADR 후보로 즉시 박고, 결합된 P0 재평가

재시뮬레이션 트리거
- 새 ADR 도입 / lifecycle 단계 변경 / skill 본문 큰 변경 — 셋 중 하나
- 매번 SIMULATION_RUN에 회차 헤더 추가
```

#### 결정 B+C — P0 (MILESTONE_TEMPLATE 강화)

**MILESTONE_TEMPLATE.md** — 현재 placeholder 그대로 사용, 새 섹션 추가 안 함(단순성):

```markdown
## 5. 완료 기준 (graduation checklist)
> sprint contract: 본 마일스톤이 "done"이라고 합의되는 외부 검증 가능한 기준.
- [ ] 모든 task status: done
- [ ] 통합 validate Pass
- [ ] E2E Pass (스택에 정의된 경우)
- [ ] AC 매핑 100% (validation report 기준)
- [ ] P0/Q1 finding 0건 (QA_FINDINGS의 본 마일스톤 헤더 기준)
- [ ] (선택) 본 마일스톤 한정 추가 기준

## 8. 회고 (stabilize 자동 채움)
- 목표 달성도: <정량/정성 1줄>
- scope creep 사례: <있으면 1줄, 없으면 "없음">
- 비목표(charter ## 5) 위반 사례: <있으면 1줄>
- 핵심 학습 3개 이내
```

`/plan-workitem`이 milestone 생성 시점에 위 default를 자동 채움. 사용자가 추가 기준을 협상.

#### 결정 D — P0 (IMPROVEMENT_GUIDE Fowler quadrant *직교* 추가)

**핵심 통찰** — P0/P1/P2(*언제 처리?*)와 Q1~Q4(*왜 발생?*)는 직교 정보. 둘 다 유지 = 정보 손실 0. 자체 매트릭스 폐기 = ADR-005 SSOT 정합:

```markdown
### M1
#### P0 (severity)
- (Q1: Deliberate+Reckless) <항목>
- (Q3: Inadvertent+Reckless) <항목>

#### P1 (severity)
- (Q2: Deliberate+Prudent) <항목>

#### P2 (severity)
- (Q4: Inadvertent+Prudent) <항목>
```

reviewer가 항목 분류 시 quadrant 라벨링. **stale 자동 처리** — 6개월 이상 손대지 않은 P0/P1 항목은 stabilize가 *명시적 보류 사유 1줄 추가 요구*. 미답 시 P2로 강등 + Q4("관찰만")로 분류. "고대 P1" 자동 정리.

#### 결정 E — P0 (`/stabilize-milestone` graduation pre-check)

skill 본문 단계 1과 2 사이 삽입 — 각 graduation checklist 항목 자동 체크 → 미충족 시 *"졸업 가능: NO, <미충족 항목>"* 출력 후 *조기 종료 옵션*. `--dry-run` 플래그 = pre-check만 돌리고 종료(P0 검증 도구).

#### 결정 F — P1 (METRICS.md)

**framing 정정** — DORA *지표*가 아니라 *원칙*만 차용:
- DORA 4지표(deploy / lead time / CFR / MTTR) = *delivery* 지표.
- 본 절 4지표 = *quality leading* 지표. 측정 대상이 다름 — "DORA 정신 차용"은 framing fraud.

**`docs/40-validation/METRICS.md`** (Living Doc) — stabilize가 매 회 1줄 append:

| 지표 | 정의 | trend 악화 신호 (reviewer 보고) |
|------|------|-----------------------------|
| AC 통과율 | (AC 통과 / 총 AC) × 100% | 직전 mile 대비 −10pp 이상 하락 |
| P0 finding 수 | QA_FINDINGS 본 mile 헤더 P0(또는 Q1) 라벨 수 | +50% 또는 절대값 ≥ 5 |
| TDD opt-out 비율 | opt-out task / 총 task | 절대값 ≥ 30% (TDD 형해화 신호) |
| Repair 라운드 평균 | task당 `/repair-workitem` 호출 평균 | 절대값 ≥ 1.5 (분해 부족 신호) |

누적 형식 — `| M1 | 92% | 2 | 18% | 0.7 | 2026-05-09 |`. 시각화는 외부 도구(github-insights / Grafana 등) 사용자 결정.

#### 결정 G — P1 (`--apply-carryover`)

stabilize의 "다음 마일스톤 carry-over"를 다음 milestone의 `## 3. 포함되는 기능`에 자동 등록 옵션. **default OFF**(텍스트 출력만), 명시 `--apply-carryover` 발화 시만 자동 박음. carry-over 항목에 `(carry-over from M1)` 주석 자동 추가 + revert 1줄 안내.

#### 결정 H — P1 (architect-opus auto-escalation *신호*)

reviewer가 다음 신호 감지 시 stabilize 출력에 *"architect-opus 호출 권장: <이유>"* 텍스트만:
- 누적 P0 ≥ 5건 (마일스톤 한정).
- 모듈 수 +50% 이상 증가.
- layer 경계 위반 ≥ 3건 (정적 분석, 6번 P0와 연결).
- TDD opt-out 비율 ≥ 30%.

**자동 호출 없음** (*결정 K cut*) — ADR-007 정책 정합. false positive 시 사용자 결정.

### ADR 분할 — *결정 1개에 ADR 1개*

| ADR | 결정 |
|-----|------|
| **ADR-014** | Milestone graduation contract — `## 5. 완료 기준` checklist + `## 8. 회고` 4 항목 + stabilize pre-check 단계 + `--apply-carryover` (한 흐름 — *milestone-level sprint contract*) |
| **ADR-015** | IMPROVEMENT_GUIDE Fowler Tech Debt Quadrant *직교* 추가 + stale 자동 강등 |
| **ADR-016** | METRICS.md 신설 + 4지표 + trend 악화 신호 |
| **ADR-017** | 시뮬레이션 dogfood 1회 의무 + 재실행 트리거 |

### 결정 근거 — 왜 이 형태인가

| 결정 | 근거 |
|------|------|
| A. todo CLI / 8단계 / 성공 기준 3개 | calculator는 stateless라 lifecycle 자극 불충분. todo는 CRUD+persistence로 8단계 모두 굴림. 성공 기준 3개는 *주관 평가 위험* 차단(*"잘 됐다/안 됐다"* 대신 *"개입 횟수·충원율·미통과 사유"*). |
| B. graduation checklist 5+1 | 4개는 task-level과 차별 없음, 6개부터 prototype 속도 저하. 5+1(선택 1) = sprint contract의 *명문화* + *조정* 둘 다 보장. |
| C. 회고 4 항목 cap | SPACE framework 6차원은 회고에 과잉(Atlassian retrospective best practice — fork 사용자가 4축 이상 안 채움). 4항목 = 압축된 sprint contract 회고. |
| D. severity × quadrant 직교 | P0/P1/P2(언제)와 Q1~Q4(왜)는 정보 차원이 다름. 직교 = 두 차원 다 보존. 자체 매트릭스 폐기 = ADR-005 SSOT 정합. |
| E. graduation pre-check 단계 | sprint contract *"what 'done' looked like before any code"* 패턴의 *milestone scale* 적용. `--dry-run`은 P0 검증 도구로 분리. |
| F. DORA *원칙*만 차용 framing | "DORA 정신"으로 framing하면 fork 사용자가 dora.dev에서 1:1 매핑 시도 → ADR-005 표현 drift 위험. 정직한 framing이 그 질문 자체 차단. |
| G. carry-over default OFF | 사용자 통제권 우선 — `--apply-carryover`는 명시 발화만. ADR-007의 *"자동 호출 아니라 텍스트 제안"* 정신 정합. |
| H. auto-escalation 신호만 | 자동 호출은 ADR-007 정책 위반 + false positive 비용 큼. 텍스트 제안은 사용자 결정 보존. |
| I. release-level DoD No | Atlassian 3계층은 enterprise scale. 본 보일러플레이트의 release = stabilize 출력의 carry-over 0건 + ADR 후보 0건. 자연 흡수. 별도 계층 도입은 단순성 위반. |
| J. 자체 매트릭스 No | Fowler quadrant가 외부 표준 — fork 사용자 학습 비용 0. 자체 발명은 ADR-005 SSOT + ADR-006 단순성 둘 다 위반. |
| K. auto-escalation 자동 호출 No | ADR-007 정책 — *"자동 호출 아니라 텍스트 제안"*. 기존 정책 깨면서 ergonomics만 약간 더 얻음 = ROI 음수. |

### 잔여 모니터링

- 시뮬레이션 1회 통과 후 *어떤 P0가 [관측됨]으로 승격되고 어떤 게 P2로 강등되는지* — 본 IMPROVE-LIST의 evidence label이 첫 데이터 회수.
- METRICS.md trend 악화 신호 임계값(−10pp / +50% / 30% / 1.5)이 적정한지 — 첫 마일스톤 stabilize 후 6번 Pillar 1 Ratchet의 enabling 정책 약 적용으로 조정.
- graduation pre-check 미통과 사유의 *반복 패턴* — 같은 사유가 3회 이상 반복되면 lifecycle 단계 자체의 결함 신호 → ADR 후보.

---

## 6. 하네스 엔지니어링 — 4 Pillars 정렬 + Context Packs + CODE_LINEAGE.md

> **TL;DR**: 본 보일러플레이트 자산을 2026 표준 4 Pillars(system prompt / tools / context / subagents)에 매핑하면 **Context pillar가 가장 약하다**. 거대 codebase 안전의 핵심도 Context. **P0 5개**(Ratchet 명문화·범위 한정 / 정적 분석 권장 / JIT 로딩 명문화 / CODE_LINEAGE.md / 코드 organization 가이드), **P1 5개**(`validate --changed` / Context Packs frontmatter / dependency hygiene / sub-agent 출력 cap / AC ID 컨벤션 강제), **명시적 No 5개**(복잡도 예산 / agent concurrency guard / Hindsight memory / 6 agent → 통합 / ADR 카테고리화는 ADR ≥ 15 트리거까지 보류). **새 ADR 5개**(ADR-018~022) + **기존 ADR amend 2건**(ADR-008·ADR-009).
>
> **본 섹션의 위치** — *거대 codebase에서도 작동하는 구조*에 초점. AGENTS.md slim 정책은 3번으로, 시뮬레이션 dogfood는 5번으로 이관. 6번은 4 pillars × *Context engineering 4 strategies*(Write/Select/Compress/Isolate) 정렬에 집중.

### 핵심 결정 (한눈)

| 분류 | # | Pillar | 결정 | 핵심 산출물 | ADR |
|------|---|--------|------|-----------|-----|
| **P0** | A | 1. System Prompt | Ratchet Principle 명문화 + 적용 범위 *제약 강·enabling 약* | _ADR_GUIDE 1단락 + builder-sonnet self-check 1줄 + 적용 범위 표 | ADR-022 신설 |
| **P0** | B | 2. Tools | `/stack-guard`가 스택별 정적 분석 *1종*만 권장 (paralysis 차단) | TS=`dependency-cruiser`, Python=`import-linter`, Go=`go vet`, Rust=`cargo deny`+`cargo udeps` | ADR-021 신설 |
| **P0** | C | 3. Context | JIT 로딩 정책 명문화 — `반드시 먼저 읽을 파일`은 *최소 충분*, 추가는 발화 시 인용 | 모든 skill SKILL.md 본문 1줄 추가 | ADR-019 본문 |
| **P0** | D | 3. Context | CODE_LINEAGE.md (git footer 기반) — 5결정 spec | `docs/40-validation/CODE_LINEAGE.md` (Living, 자동 재생성) + Conventional Commits footer 컨벤션 | ADR-018 신설 + ADR-008 amend |
| **P0** | E | 횡단 | `/bootstrap-stack`이 스택별 디폴트 디렉터리 구조 권장 출력 | Next.js / FastAPI / Rust CLI 등 디폴트 트리 | (skill amend) |
| **P1** | F | 2. Tools | `validate --changed` (incremental, git diff 기반) | `/stack-guard`가 옵션 추가, `/finalize-workitem` 직전엔 `--changed`만 | ADR-020 신설 |
| **P1** | G | 3. Context | Context Packs frontmatter | skill frontmatter `context-pack: minimal/frontend/backend/full` (default `minimal`) | ADR-019 신설 |
| **P1** | H | 2. Tools | dependency hygiene — `npm audit`/`pip-audit` + 6개월 unused | stabilize가 매 회 1회 실행, IMPROVEMENT_GUIDE Q3/Q4 자동 등록 | (skill amend) |
| **P1** | I | 4. Subagents | sub-agent 출력 cap 1~2K 명문화 | 각 agent 본문 1줄 추가 (Anthropic 가이드 인용) | (agent amend) |
| **P1** | J | 4. Subagents | AC ID 컨벤션 강제 — `AC_N` / `[AC-N]` 누락 시 validator P1 경고 | ADR-009 후속 작업의 명문화 | ADR-009 amend |
| **No** | K | 횡단 | codebase 복잡도 예산 (모듈 수·파일 라인) | *영구 No* — 측정-경고 only는 ROI 의문, 관측된 실패 0건 (Ratchet 강 적용) | (cut) |
| **No** | L | 4. Subagents | agent concurrency guard (multi-fork 차단) | *영구 No* — AGENT_EXECUTION_STRATEGY 병렬 패턴 3종이 이미 worktree 권장. 관측 실패 0건 | (cut) |
| **No** | M | 횡단 | Hindsight 동적 memory layer 도입 | *영구 No* — Claude Code 전용(Codex 비호환) → ADR-010 위반. 정적 인덱스(CODE_LINEAGE/METRICS)로 80% 효용 확보 | (cut) |
| **No** | N | 4. Subagents | 6 agent → 3 agent 통합 | *데이터 트리거 보류* — 시뮬레이션이 validator/reviewer 출력 중복률 ≥30% 보고 시 검토. 30% 미만이면 분리 유지 | (P2 트리거) |
| **No** | O | 횡단 | ADR 카테고리화(`category: policy/arch/stack/tooling/misc`) | *조건부 No* — ADR ≥ 15 도달 시까지 보류 (현재 10개, YAGNI) | (P2 트리거) |

### 근거 (외부 + 내부)

| 근거 | 출처 | 본 절의 결정 |
|------|------|-------------|
| 4 Pillars: *system prompt / tools / context / subagents* | Addy Osmani, WaveSpeed Claude Code 분석 ([wavespeed.ai](https://wavespeed.ai/blog/posts/claude-code-agent-harness-architecture/)) | 본 절의 결정 분류 축 |
| *"only 1.6% of Claude Code's codebase is AI decision logic. The other 98.4% is deterministic infrastructure"* | WaveSpeed Claude Code 분석 | 하네스 = 결정 로직이 아니라 infrastructure → 4 pillars 정렬에 집중하는 게 본질 |
| Context engineering 4 strategies — Write / Select / Compress / Isolate | Drew Breunig × Lance Martin × Anthropic ([rlancemartin.github.io](https://rlancemartin.github.io/2025/06/23/context_engineering/)) | Context Packs(*결정 G*, Select+Isolate) + JIT 로딩(*결정 C*, Select) + CODE_LINEAGE(*결정 D*, Write) + sub-agent cap(*결정 I*, Compress) — 4 strategies 모두 사용 |
| *"maintain lightweight identifiers... use these references to dynamically load data into context at runtime"* | Anthropic effective context engineering ([anthropic.com/engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)) | JIT 로딩 우선 (*결정 C*) — 사전 fork-load 금지 |
| *"return only a condensed, distilled summary of its work (often 1,000-2,000 tokens)"* | Anthropic 가이드 | sub-agent 출력 cap 1~2K (*결정 I*) — 100K 내부 reasoning이 1~2K 요약으로 회수 |
| Ratchet Principle: *"every line in a good AGENTS.md should be traceable back to a specific thing that went wrong"* | Addy Osmani | 명문화 + 적용 범위 한정 (*결정 A*) |
| 정적 분석 = TDD for architecture | dependency-cruiser ([npm](https://www.npmjs.com/package/dependency-cruiser)), import-linter | 스택별 1종 권장 (*결정 B*) — paralysis 차단 |
| `/stabilize-milestone`이 finalize의 *명시적 file add* 산출물(ADR-007)을 lineage SSOT로 활용 | 본 보일러플레이트 ADR-007 | CODE_LINEAGE는 git footer 기반 derived view (*결정 D*) — task 문서 4-1은 SSOT가 아니라 *참조 목록* |

### Multi-agent Boilerplate 비교 — 우리는 어디 있는가

| 도구 | 디렉터리 패턴 | 강점 | 약점 |
|------|-------------|------|------|
| **GitHub spec-kit** | `.specify/{memory,specs,templates,scripts}/` | 4단 워크플로우 명문화 + `constitution.md` immutable principles | "광범위 markdown" 검토 부담, 작은 prototype에 과잉 (Martin Fowler) |
| **BMAD-Method** | 다중 specialized agent 페르소나 | 풀 agile 팀 시뮬레이션, 큰 프로젝트 적합 | 단일 개발자엔 무겁고 학습 곡선 큼 |
| **Agent OS (buildermethods)** | `commands/agent-os/`, `profiles/default/global/` | "Inject standards" 패턴 — 코드베이스에서 standards 추출 후 주입 | 본문/내부 구조 미공개 |
| **shinpr/ai-coding-project-boilerplate** | `.claude/{agents,commands,skills}/` + `docs/{guides,adr,design,prd}/` | sub-agent 격리로 770K+ token 세션 무열화 | TS 한정 |
| **본 보일러플레이트** | `docs/{00-meta,10-charter,20-system,30-workitems,40-validation,90-decisions}/` | ADR-005 SSOT / ADR-006 단순성 1순위 / ADR-010 multi-tool 호환 / 8단계 lifecycle / TDD default | (3·5·6 항목들) |

**본 보일러플레이트의 정체성** = *"단순성·SSOT·multi-tool 호환을 1순위로 박은 최소 하네스"*. spec-kit/BMAD가 *완전 sound*를 추구한다면 본 보일러플레이트는 *fork-friendly + tool-portable*. 4 pillars 정렬은 *내부 구조 강제*가 아니라 *판정 도구로 사용* — docs 6분할은 그대로 유지.

### 한계 진단 — 4 Pillars 매핑

| Pillar | 본 보일러플레이트의 자산 | 본 절의 P0/P1로 메우는 약점 |
|--------|------------------------|--------------------------|
| **1. System Prompt** | AGENTS.md 35줄 / ADR-006 본문 / skill SKILL.md 본문 | Ratchet Principle 미명문화(*결정 A*). 길이 cap은 3번 P0가 처리. |
| **2. Tools** | `validate` 통합 / `/stack-guard` / `.codex/config.toml` permissions | 정적 분석 비통합(*결정 B*) / `validate --changed` 부재(*결정 F*) / dependency hygiene 부재(*결정 H*). |
| **3. Context** | 각 skill `반드시 먼저 읽을 파일` / fork된 sub-agent SKILL 자동 로드 | JIT 로딩 미명문화(*결정 C*) / Context Packs 미정의(*결정 G*) / lineage 인덱스 부재(*결정 D*). |
| **4. Subagents** | 6 agents (architect/builder/validator/qa/reviewer/planner) / 병렬 패턴 3종 | sub-agent 출력 cap 미명문화(*결정 I*) / AC ID 컨벤션 미강제(*결정 J*). 동시성 가드는 *결정 L cut*. |

### 구체적 변경 사항

#### 결정 A — Ratchet Principle 명문화 + 적용 범위 한정

**왜 필요한가** — 본 IMPROVE-LIST 자체에 Ratchet Principle을 *문자 그대로* 적용하면 본 절의 P0 다수가 *과거 관측된 실패가 아니라 가설적 미래 실패*에 근거함이 드러나 P2로 강등된다(예: AGENTS.md 100줄 cap, sub-agent 출력 cap, agent concurrency guard, 복잡도 예산). 본 보일러플레이트는 fork 사례 0건이라 *모든 예방적 정책*이 [가설]이다. 그러면 *fork된 새 프로젝트는 보호 못함*이라는 본 보일러플레이트의 존재 의의 자체와 모순. **적용 범위가 명문화되어야 자기 일관성이 선다**.

**적용 범위 표** (`docs/90-decisions/_ADR_GUIDE.md`에 박음):

| 정책 종류 | 정의 | Ratchet 강도 |
|---------|------|----------------|
| **제약 정책** (constraint) | 새 reviewer 룰 / self-check / ADR 본문 / P0 분류 기준 — *agent 행동을 좁히는 정책* | **강** — 관측된 실패 사례 필수 |
| **enabling 정책** (enabling) | cap·budget·convention·tooling 권장 — *agent에게 도구/한계 제공* | **약** — 외부 다중 repo 실증으로 충분 |

**`_ADR_GUIDE.md` 1단락 추가**:
> *"새 ADR의 '배경' 섹션은 (a) 본 보일러플레이트/fork에서 관측된 실패·발견·이슈, 또는 (b) 외부 다중 repo 실증 자료의 출처를 1~3문장으로 명시한다. 둘 다 비어 있고 *예방적 가설*만 있다면 본 ADR은 *제약*이 아닌 *enabling*(소프트 권장)이어야 한다."*

**builder-sonnet self-check 4항목에 1줄 추가**:
> *"이번 추가/변경이 어떤 구체적 실패를 막는가? 관측된 실패가 없고 가설적 예방이라면 *제약 형태로 강제하지 말고 권장 형태로*."*

**evidence label 도입** — 본 IMPROVE-LIST의 각 P0/P1 항목에 `[관측됨]`/`[외부실증]`/`[가설]` 라벨. 우선순위 요약에 반영됨.

#### 결정 B — `/stack-guard` 스택별 정적 분석 *1종*만 권장

paralysis 방지를 위해 *대안 나열* 안 함. 다음 1종만:

| 스택 | 도구 | 비고 |
|------|------|------|
| TypeScript / JS | `dependency-cruiser` | layer 위반 룰을 ARCHITECTURE_OVERVIEW `## 3-1` 채움 시 함께 권장. madge·skott은 사용자 결정 옵션. |
| Python | `import-linter` | 동일 layer 룰 패턴 |
| Go | `go vet` (built-in) | 후속 보강 가능 |
| Rust | `cargo deny` + `cargo udeps` | unused deps + license/advisory 동시 점검 |

`validate` 명령에 lint 단계로 통합 → CI fail로 잡힘. **강제 X, 권장만** (ADR-010 multi-tool 정합, GUARDRAILS_STRATEGY *"OS/셸 종속 hook 강제 X"* 정신).

#### 결정 C — JIT 로딩 정책 명문화

모든 skill SKILL.md에 한 줄 추가:
> *"`반드시 먼저 읽을 파일`은 *최소 충분*. 추가 ADR/architecture 섹션은 task 본문에서 발화 시 인용 — 사전 fork-load 금지."*

**예시** — builder-sonnet이 task 본문을 읽고 charter `## 5. 비목표`만 추가로 필요하다 판단 시 *그 시점에* Read. 매 task 모든 ADR을 fork-load하던 패턴 차단.

#### 결정 D — CODE_LINEAGE.md (git footer 기반, 5결정 spec)

> 본 절의 *가장 load-bearing 결정*. *추적성 vs 단순성 충돌*(ADR-006 WHAT 주석 금지인데 6개월 뒤 *"이 파일 왜 있지?"* 답해야 함)의 구조적 답안.

**5결정**:

1. **소스 = git log + Conventional Commits footer** (task `## 4-1` *아님*).
   `## 4-1`은 본질적으로 drift 가능(TASK_TEMPLATE 본문에도 *"git 실제 변경과 어긋나면..."* 명시). `/finalize-workitem`이 명시적 file add를 강제하므로 git이 1차 SSOT. ADR-008 amend로 footer 컨벤션 추가:
   ```
   feat(auth): implement /me endpoint
   
   Refs: T-003 (AC-2, AC-3)
   ```

2. **형식 = 4 필드 표** (방향성·활동성·현재성):
   ```
   | path                 | first       | last                          | touches | active-AC      |
   | src/auth/me.ts       | T-003 (M1)  | T-014 (M2)                    | 3       | T-014:AC-1     |
   | src/auth/oauth.ts    | T-007 (M1)  | T-007 (M1)                    | 1       | (none)         |
   | src/legacy/v1.ts     | T-001 (M1)  | (deleted in M3, last T-021)   | 4       | (deleted)      |
   ```
   - `first` = *"이 파일이 왜 생겼는가"* 1차 답.
   - `last` / `touches` = 활동성·시점.
   - `active-AC` = *결정 J*의 AC ID 컨벤션과 정합.

3. **regeneration = stabilize가 매 회 *전량 재생성***. idempotent. merge conflict는 재실행 1회로 해소.

4. **deleted file = 행 보존 + last 필드에 `(deleted in M3, last T-021)` 토큰**. *"왜 사라졌는가"*가 lineage의 가장 자주 묻는 질문.

5. **git 추적함** (.gitignore *아님*). 본문 첫 줄: `<!-- GENERATED by /stabilize-milestone — do not edit by hand -->`.

**단순성 위반 아닌 이유** — 사람이 안 만짐(git이 SSOT, view는 자동 생성) + 코드에 task ID 주석 안 박음 + DB의 view-table과 동일 패턴.

#### 결정 E — 코드 organization 가이드

`/bootstrap-stack`이 스택별 디폴트 디렉터리 구조를 *권장 출력*:

| 스택 | 디폴트 트리 |
|------|-----------|
| Next.js | `app/` `components/` `lib/` `tests/` |
| FastAPI | `app/{api,core,domain,infra}/` `tests/` |
| Rust CLI | `src/{cli,core,...}/` `tests/` |

ARCHITECTURE_OVERVIEW.md `## 3-1` 채움 시 함께 박음. 사용자 즉흥 결정 → 스파게티 차단.

#### 결정 F — `validate --changed` (incremental)

git diff 기반 변경 파일만 lint/typecheck/test. Nx affected / Turbo affected 패턴 차용. **사용 시점**:
- `/finalize-workitem` 직전 → `--changed`만(빠른 회전).
- `/stabilize-milestone` → full validate(누락 차단).

#### 결정 G — Context Packs frontmatter

skill frontmatter `context-pack` 필드:

| pack | 포함 | 용도 |
|------|------|------|
| **minimal** *(default)* | AGENTS.md + task 본문 | 단순 task (단순성 1순위) |
| frontend | + charter + ARCHITECTURE `## 3-1` + DESIGN_SYSTEM(UI 그룹) | UI task |
| backend | + charter + ARCHITECTURE `## 3-1`,`## 4`,`## 6` + DESIGN_SYSTEM(API 그룹) | API task |
| full | 모든 docs/ | architect-opus 디폴트 |

토큰 절감 추정 — minimal ~5K vs full ~30K, builder-sonnet 호출당 5~25K 절감.

#### 결정 H — Dependency hygiene

stabilize가 매 회 1회 실행 — `npm audit` / `pip-audit` (스택별). 결과를 IMPROVEMENT_GUIDE Q3(Inadvertent+Reckless)로 카테고리화. 6개월 unused deps는 Q4(관찰만) 자동 등록.

#### 결정 I — Sub-agent 출력 cap 1~2K 명문화

각 agent 본문에 한 줄 추가:
> *"반환 요약은 1,000~2,000 토큰. 긴 reasoning은 본 sub-agent 안에 둔다."*

근거 — Anthropic 가이드 *"return only a condensed, distilled summary (often 1,000-2,000 tokens)"*. 메인 컨텍스트 토큰 경합 방지.

#### 결정 J — AC ID 컨벤션 강제 (ADR-009 amend)

validator-sonnet이 AC ↔ 테스트 매핑 시 테스트 이름에 `AC_N` 또는 `[AC-N]` 식별자 누락 시 P1 경고. 시뮬레이션(P0-Gate) 통과 후 `[관측됨]` 데이터 회수 시 P0로 격상. 자연어 매칭 false positive 차단.

### Anthropic 3-agent vs 본 보일러플레이트 6-agent — 정직한 정당화

| Anthropic 3-agent | 본 보일러플레이트 매핑 | 분리 정당성 |
|------------------|--------------------|-----------|
| planner | architect-opus + planner | 큰 결정(아키텍처) ↔ 분해(workitem)는 cognitive scope가 다름 |
| generator | builder-sonnet | 1:1 매핑 |
| evaluator | validator-sonnet + qa + reviewer | *3-way 분리* — 본 보일러플레이트의 특이점 |

**3-way evaluator 분리의 정당성 = *증거 베이스 차이***:
- **validator-sonnet** = 형식 매핑(AC ↔ 테스트). 증거 베이스: AC 텍스트 + 테스트 함수 이름.
- **qa** = 동적 분석(엣지케이스·회귀). 증거 베이스: 코드 실행 결과. (Anthropic evaluator도 Playwright로 동작 검증).
- **reviewer** = 정적 분석(Clean Code + Tech Debt Quadrant). 증거 베이스: PR diff 패턴.

**통합 가능성 검토 = 데이터 트리거** (*결정 N*) — 시뮬레이션 dogfood가 validator/reviewer 출력 중복률 ≥30% 보고 시 통합 ADR 검토. 30% 미만이면 분리 유지가 데이터로 정당화. **추측이 아니라 데이터 기반**.

### ADR 분할 — *결정 1개에 ADR 1개*

| ADR | 결정 |
|-----|------|
| **ADR-018** | CODE_LINEAGE.md (git footer 기반, 4 필드, 전량 재생성) — 단일 결정. **ADR-008 amend** 함께 — Conventional Commits footer 컨벤션 추가. |
| **ADR-019** | Context Packs frontmatter + JIT 로딩 정책 (한 흐름 — *Context pillar의 read 정책*) |
| **ADR-020** | `validate --changed` (incremental) |
| **ADR-021** | `/stack-guard`가 스택별 정적 분석 1종 권장 |
| **ADR-022** | Ratchet Principle 명문화 + 적용 범위 한정 + evidence label 도입 |
| **ADR-009 amend** | AC ID 컨벤션 강제 (`AC_N`/`[AC-N]` 누락 시 validator P1) |

### 결정 근거 — 왜 이 형태인가

| 결정 | 근거 |
|------|------|
| A. Ratchet 적용 범위 한정 | *문자 그대로의 Ratchet*은 본 보일러플레이트 자체를 무너뜨림(fork 사례 0). YAGNI는 제약에 강하고 enabling에 약 — 도구는 옵트인 가능하므로 해악 작음. ADR-006 단순성 정신 정합. |
| B. 스택별 1종 (대안 나열 X) | *paralysis 차단* — 사용자가 *3종 중 어느 것?*을 묻기 시작하면 도구 채택 자체가 미뤄짐. ADR-010 multi-tool 정합 — 권장만, 강제 X. |
| C. JIT 로딩 정책 | Anthropic 공식 가이드 *"reference 기반 dynamic load"* 직접 차용. 매 task 모든 ADR fork-load 금지가 컨텍스트 토큰 경제의 핵심. |
| D. CODE_LINEAGE git footer 기반 | task 4-1은 drift 가능 → SSOT 약함. git commit은 immutable history → SSOT 강. ADR-007의 *"명시적 파일 add"*가 이미 강제하므로 footer 컨벤션 1줄만 추가하면 lineage SSOT 완성. ADR-005 패턴 1 정합. |
| E. 디렉터리 구조 권장 | 즉흥 결정 → 스파게티화는 *대규모 codebase에서 가장 자주 관측되는 실패 패턴*([외부실증]). `/bootstrap-stack`이 *권장만* — 사용자 override 가능. |
| F. `validate --changed` | 큰 codebase에서 full validate 비용 폭증 → AI가 검증 skip 위험. Nx/Turbo affected 패턴이 외부실증. finalize 직전엔 빠른 회전, stabilize는 누락 차단 — 분기. |
| G. default `minimal` | 단순성 1순위 — 사용자가 *의도적으로 풍부한 pack*을 발화할 때만 늘림. Anthropic JIT 정책 정합. |
| H. dependency hygiene | npm audit / pip-audit는 외부 표준. Q-quadrant에 자동 분류는 5번 결정 D와 정합. |
| I. sub-agent 출력 cap | Anthropic 가이드 직접 인용. 1줄 명문화 비용 미미, 메인 컨텍스트 보호 효용 큼. |
| J. AC ID 컨벤션 (ADR-009 amend) | ADR-009 후속 작업에 이미 *"권장 강화"*가 명시됨 — 본 amend는 그 후속 작업의 명문화. P1 경고 → 시뮬레이션 후 P0 격상으로 단계적 도입. |
| K. 복잡도 예산 No | 측정-경고 only는 *행동 변경 없는 텔레메트리* — Ratchet 강 적용 영역. 본 보일러플레이트 모듈 폭증 사건 0건. fork 시점에서야 실측 시작 가능. |
| L. concurrency guard No | 병렬 패턴 3종(AGENT_EXECUTION_STRATEGY)이 이미 worktree 권장. 동일 task multi-fork는 사용자 결정 영역 — 보일러플레이트가 강제하면 ADR-010 도구 중립 깸. 관측 실패 0건. |
| M. Hindsight memory No | Hindsight = SessionStart/Stop hook 기반 — Claude Code 전용. ADR-010 multi-tool 호환 위반. 정적 인덱스(CODE_LINEAGE/METRICS/IMPROVEMENT_GUIDE Q-quadrant)로 80% 효용을 도구 중립으로 확보. |
| N. 6 → 3 통합 No (데이터 트리거) | 분리 정당성이 *증거 베이스 차이*라는 검증 가능 명제 — 시뮬레이션 데이터가 결정 근거. 30% 트리거는 ADR-006 *"동일 패턴 3회 이상 추출"*의 역방향. |
| O. ADR 카테고리화 No (ADR ≥ 15 트리거) | 현재 ADR 10개. 3·5·6 결정으로 ~10개 추가 → ADR-022까지 도달. 본 PR 머지 직후 트리거 발동 가능. *진짜 ratchet*(자라난 후 정리). |

### 잔여 모니터링

- **CODE_LINEAGE.md 첫 마일스톤 회전 후 정확도** — 5결정 spec이 git 실제 변경과 일치하는지. footer 누락률이 P1 신호.
- **Context Packs 채택률** — `minimal` default가 충분한지, 사용자가 자주 `frontend`/`backend`로 override하는지. 자주 override되면 default 재검토.
- **validator/reviewer 출력 중복률** — 시뮬레이션 후 30% 트리거 충족 시 *결정 N* 발동.
- **AC ID 컨벤션 P1→P0 격상 시점** — 시뮬레이션 통과 + 후속 마일스톤 1회에서 누락률 ≤ 5% 도달 시 격상.

---

## 7. 스택별 시나리오 시뮬레이션 — 5 시나리오 × 8 lifecycle 빈자리 매핑 + 결정

> **TL;DR**: 5 시나리오(백엔드 / 프론트 / 풀스택 모노레포 / 프론트+Supabase MCP / 비웹) × lifecycle 8단계 사고 실험으로 **P0 4개**(비웹 cut / lock file 자동 포함 / monorepo scope / secret scanner), **P1 6개**(MCP baseline / 외부 의존·CI 권장 / ARCHITECTURE 백엔드·프론트 sub-section / bootstrap-stack monorepo 라운드 / plan-workitem sizing 가이드), **P2 3개**(language 1줄 / frontend +1 지표 / PII 안내), **명시적 cut 3개**(`--pr` / Supabase RLS 자동 검증 / Plan 모드는 4번에 흡수). **새 ADR 3개**(ADR-023 / ADR-025 / ADR-031), **기존 ADR amend 3개**(ADR-007·008·021). 모든 발견은 `[가설]` 또는 `[외부실증]` — 5번 P0-Gate 통과 후 [관측됨]으로 승격.
>
> **본 섹션의 위치** — 5번 P0-Gate가 *짓고 측정*이라면 본 7번은 *짓기 전 사고 실험*. 짓기 전 사고 실험은 *결정 후보*만 제공하지만 *결정 자체*는 ADR-005(SSOT) + ADR-006(단순성 1순위) + 6번 Pillar 1 Ratchet의 *enabling 정책에 약·제약 정책에 강* 기준에 따라 본 섹션 안에서 *모두 닫는다*. 트레이드오프는 결정 후 잔여 모니터링 항목으로 남긴다.

### 핵심 결정 (한눈)

| 분류 | # | 결정 | 핵심 산출물 | ADR |
|------|---|------|-----------|-----|
| **P0** | A | 비웹 스택을 *명시적 비목표*로 cut | ADR-031 본문 + AGENTS.md "직접 지원 범위" 1줄 | ADR-031 신설 |
| **P0** | B | finalize의 lock file 자동 포함 | ADR-007 amend (제외 규칙에 lock 파일 화이트리스트 11종) | ADR-007 amend |
| **P0** | C | monorepo Conventional Commits scope = 패키지명 | ADR-008 amend | ADR-008 amend |
| **P0** | D | secret scanner(`gitleaks`/`trufflehog`) 권장 | ADR-021 amend (정적 분석 권장 list에 1줄 추가) | ADR-021 amend |
| **P1** | E | MCP baseline + Supabase service_role 가드 | settings.json `mcpServers: {}` + Codex 매핑 + 권한 분류(read/mutation/admin) + secret 값 분류(`*_PUBLIC_*` vs `*_SERVICE_ROLE_*`) | ADR-023 신설 |
| **P1** | F | bootstrap-stack 외부 의존 권장 + stack-guard CI 권장 | STACK_SETUP_PLAN.md 부트업 절차 1단락 + `.github/workflows/validate.yml` *권장 출력*(생성 X) | ADR-025 신설 |
| **P1** | G | ARCHITECTURE 백엔드 sub-section 자리 | `## 7-1. 백엔드 결정` 8 prompt(DB migration / 인증 / 트랜잭션 / Idempotency / Rate limit / Async / Caching / API versioning) | (템플릿 amend) |
| **P1** | H | ARCHITECTURE 프론트 sub-section 자리 | `## 7-2. 프론트 결정` 7 prompt(라우팅 / 상태관리 / SSR-CSR / i18n / SEO / 인증 / 폼 validation) | (템플릿 amend) |
| **P1** | I | bootstrap-stack monorepo 라운드 | skill 본문 1단락(orchestrator: turbo/nx/pnpm-only / shared 패키지 위치·버전 / publish 정책) | (skill amend) |
| **P1** | J | plan-workitem sizing/분리 가이드 | skill 본문 1단락(monorepo: 패키지당 5 파일 / OpenAPI·migration task 분리) | (skill amend) |
| **P2** | K | 프로젝트 language 명시 | charter `## 0. Status` 위 `> 작성 언어: <언어>` 1줄 + AGENTS.md 1줄 | (템플릿 amend) |
| **P2** | L | frontend 한정 +1 지표 | METRICS.md 5번-E에 bundle / Lighthouse / a11y 회귀 후보 1줄 | (5번-E amend) |
| **P2** | M | PII/privacy 자리 안내 | ARCHITECTURE `## 8. 품질 속성/보안` 안내문 1줄 | (템플릿 amend) |
| **cut** | N | `--pr` 플래그 | finalize 책임 경계가 commit까지(ADR-007). push/PR은 사용자 결정 영역 | (cut) |
| **cut** | O | Supabase RLS 자동 검증 | Supabase-특이 패턴은 보일러플레이트 도구 중립 깸 | (cut) |
| **cut** | P | Plan 모드 lifecycle 통합 | 4번 P0와 동일 결정 — 같은 결정 두 번 박지 않음 | (4번에 흡수) |

### 시뮬레이션 방법론

각 시나리오를 8단계 — `discover → bootstrap-project → bootstrap-stack → stack-guard → plan → implement → validate → finalize → stabilize` — 에 통과시키며 다음 4축에서 *마찰점*을 식별한다:

1. **산출물 자리 부재** — 결정해야 할 것이 ARCHITECTURE / DESIGN_SYSTEM / TASK_TEMPLATE 등 어느 섹션에도 위치가 없다.
2. **즉흥 결정 위험** — 결정 자리는 있지만 가이드가 없어 첫 task 구현 시점에 LLM이 즉흥 결정한다.
3. **검증 누락** — validator/finalize가 그 영역의 정합성을 점검하지 않는다.
4. **단순성·SSOT 위반** — 같은 사실이 두 곳에 박힐 수밖에 없거나, 검증된 사실이 여러 단계에서 drift한다.

### 통합 lifecycle × 시나리오 마찰 매트릭스

각 cell은 *그 단계에서 가장 자주 발생할 마찰*. 빈 cell = 현 보일러플레이트로 충분.

| 단계 | 백엔드 | 프론트 | 모노레포 | Supabase MCP | 비웹 |
| --- | --- | --- | --- | --- | --- |
| discover | API consumer 페르소나 미반영 | 디바이스 폼팩터 결정 자리 0 | 단일 product 가정 | 5-tier 우선순위 자리 0 | 스택별(아래 *비웹 한정* 참조) |
| bootstrap-project | charter `## 8` 화면 가정 | DESIGN_SYSTEM placeholder 통과(1번이 다룸) | charter 단일 product 가정 | charter + Supabase 5-tier 압축 | (스택별) |
| bootstrap-stack | 외부 의존(Postgres/Redis/S3) 부트업 가이드 0 | frontend 기술 결정 자리 0 | orchestrator 결정 1줄에 압축 | anon vs service_role 분류 0 | `validate` JS-bias |
| stack-guard | OpenAPI/migration validate 통합 0 | Storybook/RTL/a11y 통합 0 | `validate --changed` 부재(6번 P1) | supabase migrations 0 | xcodebuild·cargo 미언급 |
| plan | endpoint+migration 분리 신호 0 | UI AC 표현 가이드 0 | 5파일 cap 자주 깨짐 | RLS task 분리 0 | (스택별) |
| implement | integration test TDD 회피(opt-out 누적) | 컴포넌트 TDD(RTL+MSW) 가이드 0 | RGR 사이클 시간 폭증(`pnpm -r`) | MCP prod mutation 가드 0 | binary asset diff 무력 |
| validate | OpenAPI ↔ 코드 ↔ 테스트 3-way 점검 0 | 토큰 위반(raw hex) 점검 0 | cross-package 정합 점검 0 | RLS ↔ 코드 점검 0 | 비웹 검증 도구 누락 |
| finalize | migration 파일 `## 4-1` 누락 시 `Needs Review` | (정상) | scope 컨벤션 즉흥 | service_role hardcode 사고 | (정상) |
| stabilize | API breaking·security scan 0 | bundle/Lighthouse/a11y 회귀 측정 0 | turbo/nx graph 측정 0 | Realtime cleanup 검사 0 | (스택별) |

### 시나리오별 *고유* 빈자리 (위 매트릭스 외 추가 발견)

- **백엔드** — ARCHITECTURE에 핵심 결정 8개 자리 부재(*결정 G*가 채움): DB migration / 인증·인가 / 트랜잭션 경계 / Idempotency / Rate limit / Async job / Caching / API versioning. 외부 의존 부트업(*결정 F*) + secret hardcode 검출(*결정 D*) + integration test의 TDD opt-out 회피책(P0-Gate가 [관측됨] 데이터 회수).
- **프론트** — ARCHITECTURE에 핵심 결정 7개 자리 부재(*결정 H*가 채움): 라우팅 / 상태관리 / SSR-CSR / i18n / SEO / 인증 / 폼 validation. 컴포넌트 TDD(RTL+MSW) 가이드는 implement-workitem 본문 1단락(*결정 J*에 흡수). RN/Expo 분기는 1번 시나리오 6에 이미 흡수.
- **모노레포** — orchestrator·shared 패키지·publish 결정(*결정 I*가 채움) + 패키지 간 의존성 규칙은 ARCHITECTURE `## 3-1`이 *layer 경계 + 패키지 의존성*을 동시 다룸을 1줄 단서로 amend(*결정 H의 보조*) + scope 컨벤션(*결정 C*) + AGENTS.md cap의 모노레포 위반 위험은 3번 ADR-011 본문에 *모노레포는 패키지별 docs/ 분리* 1줄 추가 권장(이미 ADR-010 D7 결과 단락에 있는 정책 재활용).
- **Supabase MCP** — service_role 가드(*결정 E*) + Edge Functions의 ARCHITECTURE 위치는 *결정 G*의 백엔드 sub-section에 흡수(Edge Function = 서버리스 백엔드). RLS 자동 검증·Realtime cleanup은 *결정 O*로 cut.
- **비웹** — `validate` JS-bias·DESIGN_SYSTEM 그룹 부재·binary asset 검증·cross-compile·LFS·hardware-in-loop·ML reproducibility 모두 *결정 A*(비웹 cut)로 한 번에 정리. `validate` 명령 자체는 `make validate`/`task validate`로 흡수 가능 — 가이드는 `bootstrap-stack`이 자연어로 받아 처리(현행대로).

### 횡단 공통 빈자리 — 모든 시나리오에 공통

| # | 빈자리 | 결정으로 매핑 |
| --- | --- | --- |
| A | MCP 통합 정책 0건(settings.json·AGENTS.md·ADR-010 어디에도 등장 X) | *결정 E* (ADR-023 신설) |
| B | Secret hardcode 검출 부재(`.env*` Read deny + finalize 경로 차단만) | *결정 D* (ADR-021 amend) |
| C | 외부 의존(DB·Redis·S3) 로컬 부트업 가이드 부재 | *결정 F* (ADR-025 신설) |
| D | CI 자리 부재(`.github/workflows/`의 자리 0) | *결정 F* (ADR-025 신설, F가 둘을 한 결정으로 묶음) |
| E | lock file 처리 정책 부재(finalize `Needs Review` 마찰) | *결정 B* (ADR-007 amend) |
| F | Plan 모드 path leakage(`settings.json`·`.gitignore`가 lifecycle 끌어안음) | *결정 P* (4번에 흡수) |
| G | 프로젝트 language 명시 부재(보일러플레이트 한국어 → 영문권 fork도 한국어 시작) | *결정 K* (charter `## 0` 위 1줄) |
| H | PR/리뷰 흐름 부재(finalize commit까지만) | *결정 N* (cut) |
| I | PII/privacy 자리 부재(ARCHITECTURE `## 8.보안` 단서 0) | *결정 M* (안내문 1줄) |

### 결정 근거 — 왜 이 형태인가

각 결정이 *왜 이 형태로 닫혔는가*를 ADR-005·006 + 6번 Pillar 1 Ratchet 정신과 정합으로 한 줄씩.

| 결정 | 근거 |
| --- | --- |
| A. 비웹 cut | ADR-006 단순성 1순위. 6 sub-stack(Mobile/ML/embedded/game/desktop) 직접 지원은 *최소 하네스 + fork-friendly + tool-portable* 정체성(6번 본문) 깸. *명시적 비목표* > *암묵적 빈자리*. |
| B. lock file ADR-007 amend | 새 ADR 신설보다 amend가 SSOT 정합. lock file은 task와 무관한 자동 생성물 → `## 4-1` 강제는 단순성 위반. |
| C. monorepo scope ADR-008 amend | scope 정책은 ADR-008 본문에 자연. ADR ≥ 15 카테고리화 트리거(6번 P2) 압박 회피. |
| D. secret scanner ADR-021 amend | 6번 P0(정적 분석 권장)와 한 흐름. 별도 ADR은 카테고리화 트리거 압박. |
| E. MCP baseline 신설(ADR-023) | 2026 MCP 표준화는 *외부실증* — 6번 Pillar 1 Ratchet의 enabling 정책에 약하게 적용 = 선제 도입 정당. ADR-010 도구 표면 매핑 자연 확장. *강제 X, 권장만*. |
| F. 외부 의존·CI 권장 통합(ADR-025) | 둘 다 "stack-guard/bootstrap-stack의 *권장 출력 확장*" 한 흐름 → 한 ADR로 묶는 게 SSOT. GUARDRAILS의 "OS/셸 종속 hook 강제 X" 정신상 *권장*만 박는 게 단순성 정합. |
| G·H. ARCHITECTURE sub-section | 1번 DESIGN_SYSTEM의 *그룹 패턴*(UI/API/CLI 그룹 → 사용 안 하면 통째 삭제) 재활용. *질문 prompt만*, 답은 사용자 — 깊이 박지 않는 enabling. |
| I·J. skill 본문 흡수 | sizing/구조 가이드라인은 *정책*이 아니라 *운영 가이드* → ADR-005 패턴 4(정책=ADR)의 경계 영역. 추적성은 git history + ADR-007 amend 후속 작업 단락에 "monorepo-aware sizing은 plan-workitem skill 본문 SSOT" 명시로 보존. |
| K. language 1줄 | ADR로 박을 만한 정책 깊이 없음. 가장 가벼운 enabling. |
| L. frontend +1 지표 | METRICS.md 5번-E의 *4 ± 1* cap 정신 정합 — frontend 한정 +1. 강제 X. |
| M. PII 안내 | 깊이 박으면 단순성 위반. *위치 안내만*. |
| N. --pr cut | finalize 책임 경계가 commit까지(ADR-007). push/PR은 사용자 결정 영역 — `.claude/settings.local.json` alias로 fork 사용자 자율. |
| O. RLS 자동 검증 cut | Supabase-특이 패턴은 보일러플레이트가 *모든 BaaS*를 다룰 수 없음 → 도구 중립(ADR-010) 정신 깸. |
| P. Plan 모드 = 4번 흡수 | ADR-005: 같은 결정 두 번 박지 않음. |

### 다른 섹션과의 정합

- **1번** — DESIGN_SYSTEM의 UI/API/CLI 3 그룹 그대로. 비웹 그룹 추가 X(*결정 A*).
- **3번 ADR-011** — AGENTS.md 100줄 cap이 모노레포에서 충돌 → ADR-011 본문에 *모노레포는 패키지별 docs/ 하위 분리*(ADR-010 D7 결과 단락 재활용) 1줄 단서 추가 권장.
- **4번** — Plan 모드 비범위(*결정 P*) 동일 결정.
- **5번 P0-Gate** — 본 7번이 *시나리오 후보 목록* 제공. 5번 P0-Gate 본문에 *"가능하면 시나리오 5종 중 1종 선택(default = trivial CLI / 옵션 = 백엔드 또는 모노레포)"* 1줄 권장 추가.
- **6번 ADR-019(Context Packs)** — frontend/backend pack의 정의가 *결정 G·H*의 ARCHITECTURE sub-section과 정렬 — pack에 백엔드/프론트 sub-section 포함.
- **6번 ADR-021(정적 분석 권장)** — *결정 D*(secret scanner) 흡수 amend.

### ADR 후보 (3개 신설 + 3개 amend, 본문 amend 6건)

**신설**
- **ADR-023** — MCP integration baseline + tool 권한 분류(read-only / mutation / admin) + secret 값 분류(`*_PUBLIC_*` vs `*_SERVICE_ROLE_*`) (단일 결정 — 결정 E).
- **ADR-025** — `/bootstrap-stack`의 외부 의존 권장 + `/stack-guard`의 CI 권장 출력 (단일 결정 — 결정 F. 둘 다 "권장 출력 확장" 한 흐름).
- **ADR-031** — 비웹 스택을 *명시적 비목표*로 cut. 보일러플레이트 직접 지원 = web frontend / API server / CLI / monorepo / Supabase 통합. mobile / ML / embedded / game / desktop은 fork 사용자가 stack-guard 출력 override (단일 결정 — 결정 A).

**amend**
- **ADR-007 amend** — finalize 우선순위 (3) 제외 규칙에 lock file 자동 포함 화이트리스트 11종(`pnpm-lock.yaml`·`package-lock.json`·`yarn.lock`·`bun.lockb`·`Cargo.lock`·`Gemfile.lock`·`composer.lock`·`go.sum`·`Pipfile.lock`·`poetry.lock`·`uv.lock`). 그 외 신규 dependency 변경(예: `package.json` deps key 추가)은 reviewer P1 (결정 B).
- **ADR-008 amend** — 모노레포 감지 시 scope = 패키지명(`feat(api):`·`feat(web):`·`feat(shared):`). bootstrap-stack의 monorepo 라운드 결과를 commit scope vocabulary로 박음 (결정 C).
- **ADR-021 amend** — 정적 분석 권장 list에 `gitleaks` / `trufflehog` 1줄 추가. *강제 X, 권장만* (결정 D).

**본문 amend (ADR 없이)**: 결정 G·H(ARCHITECTURE 백엔드/프론트 sub-section), I(bootstrap-stack monorepo 라운드), J(plan-workitem sizing 가이드), K(charter language 1줄), L(METRICS frontend +1 지표), M(ARCHITECTURE PII 안내) — 모두 템플릿/skill 본문 갱신 PR 1회로 처리.

### 트레이드오프 (결정 후 잔여 모니터링)

본 섹션은 *모든 결정을 닫음*. 다음은 결정 후 추적할 항목 (P0-Gate 시뮬레이션 또는 fork 데이터로 확인).

- **결정 A 잠재 fork 사용자 손실** — Mobile/ML/embedded/game/desktop 사용자가 보일러플레이트 채택 안 할 위험. *명시적 비목표*가 *암묵적 빈자리*보다 정직 — fork 직후 불일치 발견 시간 단축. ADR-031 본문에 *override 절차*(stack-guard 출력 사용자 무시) 1단락 권장.
- **결정 B false positive** — 11종 lock 화이트리스트 외 신규 매니저 등장 시 누락. 대응: stabilize가 staged된 *.lock·*-lock.* 패턴 중 화이트리스트 미일치 1건 발견 시 P1 보고.
- **결정 E lock-in 우려** — settings.json `mcpServers` 형식이 Claude Code-specific. Codex의 MCP 형식이 다르면 SSOT 깨짐. 대응: ADR-023 본문에 *baseline은 형식이 아닌 정책*(권한 분류 / secret 값 분류)으로 정의 + ADR-010 도구 표면 매핑 표에 한 행 추가.
- **결정 I·J skill 본문 흡수의 추적성 손실** — git history만 추적. ADR-005 정책=ADR 패턴 경계. 대응: ADR-007 amend 후속 작업 단락에 *"monorepo-aware sizing은 plan-workitem skill 본문 SSOT"* 명시.

---

## 8. 기획 — Discovery 강화(PO 면) + AI-readable PRD(PM 면) + Dual-track 핸드오프

> **TL;DR**: 본 보일러플레이트는 *build it right*(품질·lifecycle·SSOT)는 깊지만 *build the right it*(PMF·iteration·AI-readable spec)는 얕다. **P0 2개**(FEATURE_TEMPLATE PRD 강화 + DISCOVERY.md 진짜 living doc), **P1 2개**(spec self-audit + PMF 측정 opt-in), **P2 3개**(Switch Interview / Story Map+Pitch / PMF feedback loop). 새 ADR 후보 6개(ADR-035~040). 모든 항목이 `[관측됨+외부실증]` 또는 `[외부실증]` 카테고리, 순수 `[가설]`은 P2 G 1개뿐.
>
> **사용자 직관 검증** — "PO 기획(PMF 발견) vs PM 기획(시나리오·task 분해)" 분리는 Marty Cagan(SVPG) / Jeff Patton / Teresa Torres가 정의한 *Dual Track Agile* 정설 그대로다 ([SVPG](https://www.svpg.com/dual-track-agile/), [Patton](https://jpattonassociates.com/dual-track-development/)). 1인 개발자도 *역할 분리가 아니라 트랙(시간) 분리*로 동시 운영. 본 보일러플레이트의 현 lifecycle은 *single-track delivery* 위주이며 discovery가 bootstrap 직전 1회로 묶인 게 본 절의 출발점.

### 핵심 결정 (P0/P1/P2 한눈)

| 분류 | # | 결정 | 핵심 산출물 | ADR |
|------|---|------|-----------|-----|
| **P0** | A | FEATURE_TEMPLATE PRD 강화 | 10→12섹션 (신설 3 + 흡수 1 + 강화 1) | ADR-036 |
| **P0** | B | DISCOVERY.md 진짜 living doc | Assumption Tracker + Opportunity Backlog + `/discover-product --update` 모드 | ADR-035 |
| **P1** | C | Spec coverage self-audit | validator 1 step + plan-workitem FAC↔AC 매핑표 | ADR-037 |
| **P1** | D | PMF 측정 권장 opt-in | `docs/40-validation/PMF_PLAYBOOK.md` + Charter 1줄 + stabilize `--pmf` 플래그 | ADR-038 |
| **P2** | E | Switch Interview 1pager | `docs/10-charter/_templates/SWITCH_INTERVIEW.md` (트리거: 인터뷰 시도 1회) | ADR-039 |
| **P2** | F | Story Map + Shape Up Pitch | MILESTONE_TEMPLATE 옵션 3섹션 (트리거: milestone 3+) | ADR-040 |
| **P2** | G | PMF feedback loop in lifecycle | (트리거: fork SaaS의 PMF 환류 1건. 현재 보류) | — |

### 현재 상태

| 자산 | 위치 | 진단 |
|------|------|------|
| `/discover-product` (R0~R4) | `.claude/skills/discover-product/SKILL.md` | one-shot 가정. 재호출 절차 없음 |
| DISCOVERY_TEMPLATE 11섹션 | `docs/10-charter/_templates/DISCOVERY_TEMPLATE.md` | "Living Doc" 명목 분류(STRUCTURE.md), 갱신 절차 비어 있음 |
| persona / pain 템플릿 | `docs/10-charter/_templates/persona-template.md`, `pain-template.md` | 페르소나 6필드 + Pain 5열 표. 기각 후보 보존 자리 없음 |
| PROJECT_CHARTER 11섹션 | `docs/10-charter/PROJECT_CHARTER.md` | DISCOVERY.md → Charter *1방향 박기*. 역방향 sync 미정의. SSOT 모호 |
| FEATURE_TEMPLATE 10섹션 | `docs/30-workitems/_templates/FEATURE_TEMPLATE.md` | User Story / 시나리오 / Feature-level AC / NFR 자리 부재 |
| MILESTONE_TEMPLATE 7섹션 | `docs/30-workitems/_templates/MILESTONE_TEMPLATE.md` | Appetite(시간 budget) / Rabbit holes(함정) 자리 부재 |
| AC | TASK_TEMPLATE `## 6` (ADR-009) | task 한정. 시나리오 수준 AC(FAC) 부재 |
| validator-sonnet | `.claude/agents/validator-sonnet.md` | AC 통과만 점검. spec ↔ 구현 매핑 누락 미점검 |

### 외부 자료 합의 (정렬 기준 7개)

| 영역 | 권위 | 핵심 메시지 |
|------|------|------------|
| **Dual-track Agile** | [SVPG Cagan](https://www.svpg.com/dual-track-agile/) / [Patton](https://jpattonassociates.com/dual-track-development/) / [Torres](https://www.producttalk.org/rise-modern-product-discovery/) | Discovery·Delivery는 *동시* 트랙. 1인이라도 *time-split*. *"validate ideas the fastest, cheapest way"* (Cagan). |
| **PMF 측정** | [Sean Ellis 40% test](https://review.firstround.com/how-superhuman-built-an-engine-to-find-product-market-fit/) / [Vohra Coda 4-step](https://coda.io/@rahulvohra/superhuman-product-market-fit-engine) | "very disappointed" ≥ 40% = PMF 도달, 25~40% fixable, ≤25% 멀었음. Superhuman 22→58% 실증. YC·Sequoia·a16z portfolio 채택. |
| **Continuous Discovery** | [Torres OST](https://www.producttalk.org/opportunity-solution-trees/) / [airfocus 2026](https://airfocus.com/resources/events/behind-the-build-teresa-torres-first-ai-product/) | Outcome→Opportunity→Solution→Test 4계층. *every week interview*. AI 시대 *AI 추론 페르소나 vs 실제 인터뷰 페르소나* 격차 위험. |
| **AI-readable Spec** | [Osmani 6 core](https://addyosmani.com/blog/good-spec/) / [O'Reilly Radar](https://www.oreilly.com/radar/how-to-write-a-good-spec-for-ai-agents/) | 6 core(Commands·Testing·Structure·Style·Git·Boundaries) + 3-tier(✅Always/⚠️Ask/🚫Never) + self-audit("compare result, list unmet"). PRD-vs-SRS hybrid. |
| **SDD 표준** | [GitHub spec-kit](https://github.com/github/spec-kit) / [ChatPRD](https://www.chatprd.ai/learn/prd-for-ai-codegen) / [Amazon Kiro](https://kiro.dev/) | constitution → specify → plan → tasks. *user story + AC를 spec 단계에서* 박는다 (본 보일러플레이트는 task 단계만). Kiro: vibe mode vs spec mode 분기. |
| **Vibe → Agentic** | [Karpathy 2026 (TheNewStack)](https://thenewstack.io/vibe-coding-is-passe/) / [Lenny](https://www.lennysnewsletter.com/p/everyone-should-be-using-claude-code) | "vibe coding is passé" — spec discipline + scrutiny가 default. Lenny: *"Nailing the problem statement is the single most important step."* |
| **Shape Up Pitch** | [Basecamp Shape Up](https://basecamp.com/shapeup/shape-up.pdf) | 5요소: Problem·Appetite(시간 budget)·Solution·Rabbit holes·No-gos. MILESTONE_TEMPLATE은 Appetite·Rabbit holes 부재. |

### 한계 진단 (핵심 빈자리 6개)

#### PO 트랙
- **#1. Discovery는 1회용** `[관측됨+외부실증]` — `/discover-product`가 bootstrap 직전 1회. mid-project pivot에 *재호출 절차 없음*. STRUCTURE.md는 DISCOVERY.md를 "Living Doc"으로 분류했지만 *어떻게 살아 있는가* 정의 비어 있음.
- **#2. PMF 측정 외부 표준 인용 부재** `[외부실증]` — Sean Ellis 40% / Superhuman 4-step이 *무료·15분 setup*인데 안내 없음. Charter `## 6. 성공 기준`이 자체 발명 metric(*"DAU 100명"* 같은)으로 채워질 위험 — fork 사용자가 매번 새로 학습.
- **#3. Assumption tracker drift** `[관측됨+외부실증]` — `## 9. 핵심 가정`(DISCOVERY.md/Charter)에 검증 방법은 적지만 *언제·결과·다음 행동* 추적 안 됨. 6개월 뒤 *"이 가정은 검증 끝났나?"* 답 불가. Continuous discovery *every week interview* 정신상 가장 큰 빈자리.

#### PM 트랙
- **#4. Feature template의 narrative 부재** `[관측됨+외부실증]` — F-xxx에 User Story / 시나리오 / Feature-level AC / NFR 자리 없음. AI agent가 implement하기엔 *who·why·시나리오 측정 기준* 부족. Charter `## 3.1`은 *제품 전체* 1개라 *feature 단위* 시나리오 자리 어디에도 없음. spec-kit/Kiro/ChatPRD 모두 *feature 단위 user story+AC를 spec 단계에서* 박는데 본 보일러플레이트는 task 단계에서만.
- **#5. Spec ↔ 구현 누락 점검 없음** `[외부실증]` — validator는 AC 통과만 본다. feature `## 5. 요구사항` 중 task로 분해 안 된 요구사항이 *조용히 누락*된다. Osmani self-audit 정신 부재.

#### 횡단
- **#6. Discovery 갱신 시 Charter sync 절차 없음 (SSOT 모호)** `[관측됨]` — DISCOVERY → Charter *1방향 박기*만 정의, 역방향 sync 미정의. PO 결정(페르소나 변경·핵심 pain 변경)이 milestone/feature를 *역방향 갱신*하는 절차 없음. *DISCOVERY.md가 SSOT인지 Charter가 SSOT인지 불명* — ADR-005 SSOT 패턴 1 위반 위험.

### 개선 방향

#### A. P0 — FEATURE_TEMPLATE을 AI-readable PRD로 강화 `[관측됨+외부실증]`

**결정**: 10섹션 → 12섹션. **신설 3개**(`## 3 핵심 시나리오 (Feature-level)` / `## 7 Feature-level AC` / `## 8 NFR`) + **흡수 1개**(기존 `## 8 검증 방법` → 새 `## 7 FAC`로 흡수, 측정 가능 형식화) + **강화 1개**(기존 `## 2 사용자 가치` → User Story 형식 강제).

**산출물 — `docs/30-workitems/_templates/FEATURE_TEMPLATE.md` 갱신**:
```markdown
## 1. 요약                                       # 변경 없음
## 2. 사용자 가치 (User Story)                    # 강화
<!-- "As a <persona>, I want to <goal>, so that <benefit>." 1개 이상.
     persona는 PROJECT_CHARTER.md `## 2.1` ID 인용 — 자체 발명 X. -->
## 3. 핵심 시나리오 (Feature-level)               # 신설
<!-- happy / alternate / fail 각 3~5단계.
     Charter `## 3.1`(제품 전체)과 다른 *이 feature 한정* 시나리오. -->
## 4. 범위 / 5. 비범위 / 6. 요구사항             # 변경 없음
## 7. Feature-level Acceptance Criteria           # 신설(구 `## 8 검증 방법` 흡수)
<!-- FAC-1, FAC-2 ... 시나리오 수준 측정 가능 기준.
     task `## 6 AC`는 FAC를 만족시키는 구현 단위. -->
## 8. Non-functional Requirements                 # 신설
<!-- 성능·접근성·보안·i18n. 해당 없으면 "(해당 없음)" 명시. -->
## 9. 엣지 케이스 / 10. 의존성 / 11. 관련 문서 / 12. 열린 질문   # 변경 없음
```

**`--fast` 회피 보장**: prototype은 신설 3섹션을 1줄씩만 채워도 OK(*"해당 없음"* / *"M2 이후 검토"*). YAGNI 정합 — 5번 graduation contract의 *시작 시점 budget*과 동등 정신.

**Boundaries 3-tier (Osmani) — AGENTS.md에만 1회 적용, Feature/Task 본문에는 강제 X**: AGENTS.md "핵심 행동 규율" 섹션에 ✅/⚠️/🚫 라벨 도입. 100줄 cap(3번 P0) 깨지 않는 선에서 *기존 5~7개 규율 라벨링만*, 추가 항목 0개. ADR-005 패턴 1(SSOT) 정합.

**plan-workitem 본문 1줄 추가**: *"feature 분해 시 12섹션 모두 채운다. `## 7 FAC`는 task `## 6 AC`로 분해되며 매핑 누락 시 plan 출력의 *남은 미결정 사항*에 명시."*

**근거**: Osmani 6 core + ChatPRD 6 sections(외부실증) + FEATURE_TEMPLATE의 narrative·NFR 자리 비어 있음(관측됨). 추가 비용 2섹션, --fast로 prototype 부담 0.

#### B. P0 — DISCOVERY.md 진짜 living doc + Assumption Tracker `[관측됨+외부실증]`

**결정**: 11섹션 → 13섹션. **신설 2개**(`## 12 Assumption Tracker` / `## 13 Opportunity Backlog`). `/discover-product`에 `--update` 모드 추가. **DISCOVERY.md = persona/scenario/assumption SSOT, Charter는 snapshot view**(역방향 SSOT 아님)을 ADR-035에 명문화 — *한계 진단 #6 SSOT 모호의 직접 해소*.

**산출물 — `docs/10-charter/_templates/DISCOVERY_TEMPLATE.md` 갱신**:
```markdown
## 12. Assumption Tracker          # 신설
<!-- ## 10 핵심 가정의 *검증 결과 누적*. 빈 결과 = "미검증 - 행동 차단", stabilize가 보고. -->
| ID  | 가정                        | 검증 방법    | 검증 결과   | 검증일      | 다음 행동       |
| A-1 | 타겟이 매주 이력서 갱신     | 5명 인터뷰   | true (4/5) | 2026-05-15  | F-002 진행      |
| A-2 | 무료→$5 전환 ≥ 10%         | landing A/B  | false (3%) | 2026-06-01  | pivot 필요      |

## 13. Opportunity Backlog          # 신설
<!-- 기각·검증실패 후보까지 보존 (Torres OST opportunity space 정신). -->
| Pain                       | 빈도×고통  | 현재 상태       | 비고                    |
| 매일 이력서 버전 관리       | 매일×중   | active (F-001) | 핵심                    |
| 협업자 권한 관리           | 가끔×하   | parked         | M3 이후 재평가          |
| AI 자동 작성               | -          | rejected       | A-2 검증 결과 false     |
```

**`/discover-product` skill 갱신**:
- **`--update` 모드**: 기존 DISCOVERY.md 있으면 R0(페르소나 재확인) → R1·R2(opportunity backlog 갱신·새 pain 추가) → R3(assumption tracker 갱신) → R4 저장.
- **`--fast --update`**: assumption tracker만 갱신(가장 빈번한 mid-project use case).
- **Idempotency**: ID 매칭. 기존 ID(A-1·A-2)면 *검증일·다음 행동만 갱신*, 새 가정이면 새 ID 부여. skill 본문에 명문화.

**Charter sync 절차 (AGENTS.md "깊은 운영 원칙" 인덱스 1줄 추가)**: *"DISCOVERY.md 갱신 시 Charter는 자동 sync 안 됨. 사용자가 `/bootstrap-project --apply`로 갱신 제안을 받거나 직접 편집. Charter는 DISCOVERY.md의 snapshot — 역방향 SSOT 아님."*

**근거**: Cagan/Torres dual-track(외부실증) + DISCOVERY.md "Living Doc" 분류했는데 *살아 있는 절차 비어 있음*(관측됨). DISCOVERY=SSOT 박음으로써 ADR-005 정합 강화.

#### C. P1 — Spec coverage self-audit `[외부실증]`

**결정**: validator-sonnet에 1 step 추가, plan-workitem 출력에 매핑표 추가. **자동 차단 X(제안만)** — ADR-007 validator 책임 경계(*판정+report 기록 전용*) 정합.

- **validator-sonnet 본문 1줄 추가**: *"feature `## 7 FAC`의 각 항목이 task `## 6 AC`로 매핑됐는가? 매핑 안 된 FAC가 있으면 report에 `Spec Gap: FAC-N → unmapped` 명시 + 미커버 task 추가 권장."*
- **plan-workitem 출력 형식 1줄 추가**: *"plan 출력에 새 섹션 `## 8. FAC ↔ AC 매핑표`. 형식: `FAC-1 → T-001:AC-1, T-002:AC-2`. 미커버 FAC는 `unmapped`."*

**근거**: Osmani self-audit(외부실증). 토큰 비용 ~1K/task. 6개월 뒤 누락 발견 비용보다 작음.

#### D. P1 — PMF 측정 권장 (opt-in) `[외부실증]`

**결정**: `docs/40-validation/PMF_PLAYBOOK.md`(Reference) 신설. **강제 X, 권장만**. 부적합 프로젝트는 Charter `## 6`에 *"PMF 측정 비목표"* 명시 — multi-stack 정체성(API 서버·CLI·internal admin) 보호.

**산출물 — `docs/40-validation/PMF_PLAYBOOK.md` (Reference, ~80줄)**:
- **Sean Ellis 40% test 1pager**: 질문 5개(NPS-style + "very disappointed" 핵심 질문) / threshold 해석(<25% 멀었음 / 25~40% fixable / ≥40% 도달).
- **Superhuman 4-step engine 절차**: (1) 설문 전체 발송, (2) "very disappointed" 응답자만 high-expectation segment, (3) 그들의 *love it about*과 *main benefit* 분석 → 페르소나 sharpening, (4) roadmap을 *retention 강화* + *acquisition 확대*로 분리.
- **적용 가능 판단 1pager**: ✅ 권장 = B2C / B2B SaaS / freemium → 유료 전환. 🚫 비목표 = anonymous CLI / 강제 사용 internal admin / API 서버(end-user 없음) / mobile native 비SaaS.

**Charter `## 6. 성공 기준` 안내 1줄 추가**: *"PMF 측정 적용 가능한 프로젝트는 PMF_PLAYBOOK.md 권장. 부적합은 *PMF 측정 비목표*로 명시."*

**stabilize-milestone `--pmf` 플래그 (옵션)**: 출력에 *"PMF score 한 줄 입력 권장"* 안내. METRICS.md(5번 P1)에 PMF score 행 옵션 추가(선택 행 — 부적합 프로젝트는 비움).

**근거**: Sean Ellis 40% test(외부실증, YC·Sequoia·a16z 채택) + Vohra 4-step(Superhuman 22→58% 실증) + multi-stack 정체성 보호(opt-in). enabling 정책이라 Ratchet 약 적용(6번 Pillar 1).

#### E. P2 — Switch Interview 1pager `[외부실증]`

**결정**: `docs/10-charter/_templates/SWITCH_INTERVIEW.md` (Reference). **트리거**: 사용자 인터뷰 시도 1회 발생 시.

- **5단계 timeline 질문**: *첫 생각 → 검색 → 평가 → 선택 → 사용*.
- **4-forces 모델 (Bob Moesta)**: *push*(현 상황 불만족) / *pull*(새 솔루션 매력) / *anxiety*(전환 두려움) / *habit*(현 솔루션 관성).
- **결과 환류**: DISCOVERY.md `## 13. Opportunity Backlog`에 새 pain 추가, `## 12. Assumption Tracker`에 검증 결과 행 추가.

**`/discover-product` R0/R1에 1줄 추가**: *"페르소나가 AI 추론만으로 생성됐다면 *가정*으로 표시. 인터뷰 1~3건 후 `--update`. SWITCH_INTERVIEW.md 참조."*

**근거**: Bob Moesta·Klement JTBD(외부실증). AI 시대 *AI 추론 페르소나 vs 실제 인터뷰 페르소나* 격차 위험(Torres airfocus 경고).

#### F. P2 — Story Map + Shape Up Pitch (milestone scoping) `[외부실증]`

**결정**: MILESTONE_TEMPLATE 7섹션 → 9섹션. 신설 3개(`## 1-1 Appetite` / `## 2-1 Story Map` / `## 4-1 Rabbit holes`) 모두 *옵션*. 비웹·CLI는 비워둬도 통과. **트리거**: milestone 3개 이상 잡힌 프로젝트.

```markdown
## 1-1. Appetite (시간 budget)               # 신설(옵션)
<!-- Shape Up: small batch 2주 / big batch 6주. 명시 안 하면 ∞. -->
## 2-1. Story Map (옵션)                     # 신설(옵션)
<!-- 사용자 여정 시간축 표. UI/SaaS에 가치, internal/CLI는 생략 OK. -->
| 단계 | 1.시작 | 2.핵심 동작 | 3.마무리 |
| MVP (M1) | 회원가입 | 이력서 1개 작성 | 다운로드 |
| 강화 (M2) | OAuth | 다중 버전 | 공유 링크 |

## 4-1. Rabbit holes (조심할 함정)            # 신설(옵션)
<!-- 깊이 빠지면 안 되는 영역. builder가 회피. -->
```

**5번 graduation contract와 정합**: *완료 기준*은 graduation(P0), *시작 budget*은 appetite — milestone 양 끝 명시되면 scope 변동 가능 영역 명확.

**근거**: Shape Up 5요소 + Patton story mapping(외부실증). 옵션이라 enabling — Ratchet 약. 1인 prototype은 모두 비우면 OK.

#### G. P2 — PMF feedback loop in lifecycle `[가설]`

**결정**: 현 시점 채택 *보류*. 본 보일러플레이트 fork 사례 0이라 [관측됨] 승격 안 됨.
- **트리거**: fork된 SaaS 프로젝트가 PMF 측정 결과를 charter에 환류한 사례 1건.
- **트리거 도달 시 결정**: stabilize에 *product metric 보고* step 추가 + DISCOVERY.md `--update` 자동 권장.

#### H. ADR 후보 (총 6개, "결정 1개 = ADR 1개" 정합)

- **ADR-035** — Continuous discovery (DISCOVERY.md living doc + Assumption Tracker + Opportunity Backlog + `--update` 모드 + DISCOVERY=SSOT/Charter=snapshot 명문화).
- **ADR-036** — Feature-level PRD (FEATURE_TEMPLATE 12섹션 + plan-workitem FAC↔AC 매핑 강제 + AGENTS.md boundaries 3-tier).
- **ADR-037** — Spec coverage self-audit (validator 1 step + plan-workitem 매핑표 출력).
- **ADR-038** — PMF 측정 권장 정책 (PMF_PLAYBOOK.md + Charter 1줄 + stabilize `--pmf` + 부적합 비목표 명시).
- **ADR-039** — JTBD Switch Interview 1pager (P2, 트리거 도달 시).
- **ADR-040** — Milestone scoping 도구 (Story Map + Appetite + Rabbit holes, P2, 트리거 도달 시).

### 시나리오 검증 (5종)

| # | 프로젝트 | 흐름 요약 | 결과 |
|---|---------|----------|------|
| 1 | B2C SaaS (Next.js+Supabase) | discover → bootstrap → M1 → stabilize `--pmf` → Sean Ellis 28% → `/discover-product --update`(페르소나 sharpening) → M2 | DISCOVERY.md A-1=false 박힘, PMF_PLAYBOOK 적용. lifecycle 8단계 변경 없음. |
| 2 | B2B internal admin tool | Charter `## 6`에 *PMF 측정 비목표* 명시 → 정상 lifecycle | PMF_PLAYBOOK 미사용. FEATURE 12섹션·DISCOVERY 13섹션 모두 적용. |
| 3 | Rust/Go CLI 도구 | NFR=*"해당 없음"* / Story Map 비움 / PMF 비목표 / SWITCH_INTERVIEW은 anonymous 배포라 적용 어려움 명시 | 핵심 보일러플레이트 흐름 동일. |
| 4 | 빠른 prototype `--fast` | discover-product `--fast` → R1+R2+R4만 + FEATURE 신설 3섹션 1줄씩만 → spec self-audit skip | placeholder 채워지지만 prototype 깊이. Assumption Tracker 빈 채. |
| 5 | Mid-project pivot | M2 stabilize 후 인터뷰에서 가정 false → `/discover-product --update --fast` → A-1 갱신 → `/bootstrap-project --apply` charter 갱신 → M3 plan | 첫 *continuous discovery 실사용*. lifecycle 변경 없음, 단계 1 재진입만 추가. |

### 다른 섹션과의 관계

| 섹션 | 정합 |
|------|------|
| **3번 (문서 아키텍처)** | DISCOVERY/Charter/FEATURE/TASK는 *Reference + Tutorial* 모드. AGENTS.md 100줄 cap을 본 8번이 깨지 않음(boundaries 3-tier는 *기존 5~7개 라벨링*만). |
| **4번 (plan-workitem)** | 본 8번의 핵심 진입점. task sizing/AC 품질(4번 P0)과 Feature PRD(8번 P0)는 *다른 계층 보강*이라 시너지. |
| **5번 (graduation contract)** | graduation checklist에 *FAC 매핑 100%* 항목 추가 권장. METRICS.md에 *PMF score* 행 옵션 추가(--pmf 시). |
| **5번 P0-Gate (dogfood)** | mock charter/FEATURE 작성 시 본 8번 12섹션 모두 채움 강제 → enabling 정책의 [관측됨] 승격 자연 경로. |
| **6번 (4 Pillars)** | Pillar 3에 *spec packs* 추가 가능(`discovery` pack / `feature` pack). Pillar 4에 validator self-audit. Ratchet 약 적용(대부분 enabling). |
| **7번 (시나리오 시뮬레이션)** | 시나리오 검증표 참조. 시나리오 5(비웹) cardinality 충돌 회피 — PMF·Story Map 모두 옵션. |

### 본 섹션의 우선순위 요약 통합 방식

본 섹션 항목 evidence label은 대부분 `[관측됨]`(placeholder 비어 있음) + `[외부실증]`(7개 권위) 둘 다인 *강한 카테고리*. 순수 `[가설]`은 G(PMF feedback loop) 1개뿐.

| 그룹 | 항목 | 트리거 |
|------|------|--------|
| **P0 즉시 (this PR)** | A: FEATURE_TEMPLATE PRD 강화 / B: DISCOVERY.md 진짜 living doc | 즉시 |
| **P1 다음 마일스톤** | C: spec coverage self-audit / D: PMF 측정 권장 opt-in | 다음 마일스톤 plan 시 |
| **P2 트리거 도달** | E: Switch Interview / F: Story Map + Pitch / G: PMF feedback loop | 인터뷰 1회 / milestone 3+ / fork SaaS PMF 환류 1건 |

### 결정 근거 (구 트레이드오프 → 결론으로 변환)

이전 초안의 12개 트레이드오프를 모두 *결정*으로 변환. 본 보일러플레이트 1순위 정책(ADR-006 단순성·ADR-005 SSOT·ADR-010 multi-tool 호환) + 외부 자료 + 다른 섹션 맥락을 종합한 *추천 결정*.

| # | 이슈 | 결정 |
|---|------|------|
| 1 | "PO/PM 분리"가 1인 개발자에게 과잉인가? | **트랙만 분리, 역할 X.** Cagan dual-track 원래 정의도 *time-split for one person*. P0 두 항목은 *문서 구조*만 바꾸므로 1인 비용 0. |
| 2 | PMF 측정 강제 vs 권장? | **opt-in 권장.** PMF_PLAYBOOK.md에 *적용 가능 판단* 1pager 포함 → 적용 가능한데 skip 위험 차단. 부적합 프로젝트(API 서버·CLI·internal admin) 보호. |
| 3 | `--update` 모드 idempotency? | **ID 매칭.** 기존 ID 있으면 검증일·다음 행동만 갱신, 없으면 새 ID 부여. Skill 본문 명문화. |
| 4 | FAC vs AC 경계? | **FAC=시나리오 수준, AC=구현 수준.** 1:1 매핑이면 FAC=AC 1개. unmapped FAC만 분해 강제. self-audit 통과 = unmapped 0. |
| 5 | Self-audit false positive? | **제안만, 자동 차단 X.** ADR-007 validator 책임 경계(*판정+report 기록 전용*) 정합. 최종 판정은 사용자. |
| 6 | Osmani 6 core 매핑 격차? | **격차 = Boundaries 1개만.** Commands·Testing·Structure·Style·Git Workflow는 AGENTS.md+STRUCTURE+ARCHITECTURE+ADR-008 분산 박힘 → 매핑만 명시. Boundaries 3-tier 신규는 AGENTS.md 한정. |
| 7 | 외부 자료 지역 편향(서구 SaaS)? | **PMF_PLAYBOOK.md에 *비목표 옵션* 박음.** 한국·아시아 시장 특수성(B2B 영업·internal tool 비중)은 *비목표 명시*로 회피. |
| 8 | Karpathy "vibe coding is passé" 함의? | **`--fast`로 vibe/spec 분기.** prototype은 `--fast`(1줄씩 채움), production은 full spec. Kiro 분기와 동치. |
| 9 | spec-kit / Kiro 흡수 깊이? | **정신만 차용, 구조는 본 보일러플레이트 유지.** constitution.md ↔ AGENTS.md+ADR-006 매핑(별도 파일 추가 X). 4-phase ↔ 8단계 lifecycle 매핑. ADR-005·006·010 정합. |
| 10 | Lenny "nail problem statement" 매핑? | **Assumption Tracker가 그 압력 형식화.** Charter `## 3` + DISCOVERY `## 1`은 이미 정합. 본 8번이 *지속 검증 압력* 추가. |
| 11 | PMF feedback loop 채택 시점? | **P2 보류, 트리거 도달 시.** fork SaaS의 PMF 환류 1건이 트리거. 보일러플레이트 fork 0이라 [관측됨] 승격 불가. |
| 12 | PMF_PLAYBOOK + SWITCH_INTERVIEW 통합? | **분리 유지.** verb 다름(*측정* vs *인터뷰*), lifecycle 시점 다름(post-launch vs pre-discovery). ADR-006 단일 책임 정합. 통합 시 단일 파일이 비대화. |

---

## 우선순위 요약 (모든 항목 종합)

> 3·5·6번 재구성 + 자기 비판 6항목 반영 후 갱신. 외부 자료 합의(Anthropic 4 pillars / context engineering / Augment·GitHub 2,500-repo / Fowler quadrant / DORA *원칙만* / Drew Breunig "Implement to learn" / Addy Osmani Ratchet) 반영.

### Evidence label 범례

각 항목 끝에 다음 라벨을 박는다 (Ratchet Principle의 적용 범위 한정에 따른 자기 일관성 장치 — 6번 Pillar 1):

- `[관측됨]` = 본 보일러플레이트 또는 fork에서 *실제로 발생한 실패·불편*에 근거.
- `[외부실증]` = Augment / GitHub / Anthropic / Fowler 같은 *외부 다중 repo 실증 자료*에 근거.
- `[가설]` = 예방적 가설 — 시뮬레이션(P0-Gate) 결과로 `[관측됨]`으로 승격 또는 P2로 강등.

### P0-Gate (다른 P0 시작 *전*에 통과해야 하는 단일 항목)

이 항목 하나만 P0-Gate다. 통과 못하면 아래 P0-Post-Gate 항목들의 채택을 보류한다 — *깨진 lifecycle 위에 새 정책을 짓지 않는다*.

- **5번 보일러플레이트 자체 dogfood 시뮬레이션** `[가설→실증 변환 도구]`
  - 8단계 lifecycle 1회 통과 + `SIMULATION_RUN.md` 기록.
  - 성공 기준: 사용자 개입 ≤ 1회 / placeholder 충원율 ≥ 80% / graduation pre-check 미통과 사유 ≤ 2개.
  - 실패 시: 발견된 깨짐을 ADR 후보로 박고, 본 IMPROVE-LIST의 결합된 P0 재평가.

### P0-Post-Gate — Gate 통과 후 즉시 검토

#### 단순성·SSOT 정합 cleanup (외부실증 + 관측 약함, 비용 작음)

- **3번 docs/00-meta 흡수 PR 1회** `[외부실증]` — TEMPLATE_GUIDE / LOCAL_SETTINGS_EXAMPLE / BOOTSTRAP_PROMPT_EXAMPLES / 30-workitems/README 흡수 → 9문서 → 5문서. ADR-005 SSOT 정합 검증 PR.
- **3번 AGENTS.md 100줄 hard cap 정책** `[외부실증]` — Augment 2,500-repo 연구의 sweet spot. ADR-011a로 박음.
- **3번 AGENTS.md 항목별 trace-to-failure 정리** `[관측됨]` — 기존 5개 행동 규율이 어느 ADR/사건 근거인지 1회 retrospective.

#### Milestone graduation 정합 (관측됨 — 본 보일러플레이트의 빈 placeholder 자체가 증거)

- **5번 milestone graduation contract** `[관측됨]` — MILESTONE_TEMPLATE `## 5. 완료 기준` checklist 강화 + stabilize의 graduation pre-check 단계.
- **5번 milestone retrospective 4항목** `[외부실증]` — MILESTONE_TEMPLATE `## 8. 회고`(stabilize 자동 채움).
- **5번 IMPROVEMENT_GUIDE Tech Debt Quadrant 재구조** `[외부실증]` — Fowler 4-quadrant로 분류, 자체 매트릭스 폐기.

#### 4 pillars 정렬 — context pillar 우선

- **6번 Ratchet Principle 명문화 + 적용 범위 한정** `[관측됨]` — _ADR_GUIDE 1단락 + builder-sonnet self-check 1줄 + 적용 범위 표(제약 강 / enabling 약).
- **6번 task ↔ 코드 lineage 인덱스 (git footer 기반)** `[외부실증]` — `CODE_LINEAGE.md` 자동 생성(stabilize). git log + Conventional Commits footer가 SSOT, 4 필드 형식, 전량 재생성.
- **6번 정적 분석 권장 (강제 X)** `[외부실증]` — `/stack-guard`가 dependency-cruiser/import-linter 등 권장.
- **6번 코드 organization 가이드** `[외부실증]` — `/bootstrap-stack`이 스택별 디폴트 디렉터리 구조 권장.

#### 다른 항목들 (1·2·4번에서 결정된 P0)

- **1번 인터페이스 결정 흐름** `[외부실증]` — UI=`/bootstrap-design` + `docs/20-system/DESIGN.md`(rename, UI 한정), API/CLI=`/bootstrap-stack` 확장 + `ARCHITECTURE_OVERVIEW.md` 신설 7-1/7-2 흡수. 운영성 SSOT 중복 자동 해소 + 비-UI 프로젝트 파일 1개 감소.
- **4번 plan 모드 lifecycle 비범위 (ADR-024)** `[관측됨]` — settings 1줄 삭제 + `docs/30-workitems/plans/` 제거 + STRUCTURE/README/AGENTS.md 정리 + /implement-workitem 1줄 흡수. ADR-010(Codex 호환) + ADR-006(단순성) + 외부 버그 #19537 정합.
- **4번 TASK_TEMPLATE schema 강화 (ADR-026)** `[관측됨+외부실증]` — sizing 한계 3종(1 RGR / AC ≤3 / 변경 파일 ≤5) + AC verb whitelist + Given-When-Then 강제 + `## 9. 의존성`. planner skill self-check 3줄.

#### 7번 스택 시뮬레이션 — P0 4개

- **7번-A 비웹 스택 *명시적 비목표* cut** `[외부실증]` — ADR-031 신설. 보일러플레이트 직접 지원 = web frontend / API server / CLI / monorepo / Supabase. mobile / ML / embedded / game / desktop은 fork 사용자가 stack-guard 출력 override. *명시적 비목표*가 *암묵적 빈자리*보다 ADR-006 단순성 정합.
- **7번-B finalize lock file 자동 포함** `[가설→Gate 후 실증]` — ADR-007 amend. 11종 화이트리스트(`pnpm-lock.yaml`·`package-lock.json`·`yarn.lock`·`bun.lockb`·`Cargo.lock`·`Gemfile.lock`·`composer.lock`·`go.sum`·`Pipfile.lock`·`poetry.lock`·`uv.lock`). 첫 finalize 시점에 발견.
- **7번-C monorepo Conventional Commits scope 컨벤션** `[외부실증]` — ADR-008 amend. scope = 패키지명(`feat(api):`·`feat(web):`·`feat(shared):`).
- **7번-D Secret scanner(`gitleaks`/`trufflehog`) 권장** `[외부실증]` — ADR-021 amend(6번 정적 분석 권장 list에 1줄 추가). *강제 X, 권장만*.

### P1 — 다음 마일스톤

- **3번 Diátaxis 모드 라벨 1줄 + reviewer 모드 mismatch 검출** `[외부실증]` — 각 docs/00-meta 문서 첫 줄 라벨 + `/review-doc`이 모드 mismatch 시 P1 보고. *행동 변경 없는 라벨링은 노이즈*이므로 reviewer 통합이 P1 자격.
- **3번 SSOT drift 점검 자동화** `[가설]` — `/stabilize-milestone`에 키워드 중복 검출 + `/review-doc` 강화.
- **5번 METRICS.md (4 ± 1 지표, DORA *원칙만*)** `[외부실증]` — AC 통과율 / P0 수 / opt-out 비율 / repair 라운드 평균. trend 악화 신호 명문화.
- **5번 carry-over 자동 등록** `[관측됨]` (`--apply-carryover` 플래그).
- **5번 architect-opus auto-escalation 신호** `[가설]`.
- **6번 Context Packs** `[외부실증]` — skill frontmatter `context-pack: minimal | frontend | backend | full`. JIT 로딩 정책 명문화.
- **6번 incremental validate** `[외부실증]` — `validate --changed` (Nx affected / Turbo affected 패턴).
- **6번 dependency hygiene** `[외부실증]` — `npm audit`/`pip-audit` + 6개월 unused deps Q4 자동.
- **6번 AC ID 컨벤션 강제** `[관측됨]` — validator가 `AC_N`/`[AC-N]` 누락 시 P1. ADR-009 후속 작업의 명문화.
- **6번 sub-agent 출력 cap 1~2K 명문화** `[외부실증]`.
- **2번 ADR 라벨링** `[관측됨]` (옵션 A + 옵션 C).
- **4번 planner charter 정합 self-check + architect-opus 신호 4종 (ADR-026 본문)** `[관측됨]` — planner skill 본문 1단락(2문장 self-check) + 신호 감지 시 텍스트 제안 1줄(자동 호출 X — ADR-007 정합).

#### 7번 스택 시뮬레이션 — P1 6개

- **7번-E MCP baseline + Supabase service_role 가드** `[외부실증]` — ADR-023 신설. settings.json `mcpServers: {}` + Codex 매핑 + 권한 분류(read/mutation/admin) + secret 값 분류(`*_PUBLIC_*` vs `*_SERVICE_ROLE_*`). *강제 X, 권장만*. Supabase MCP 시나리오의 가장 큰 보안 가드.
- **7번-F bootstrap-stack 외부 의존 권장 + stack-guard CI 권장** `[외부실증]` — ADR-025 신설. STACK_SETUP_PLAN.md 부트업 절차 1단락 + `.github/workflows/validate.yml` *권장 출력*(생성 X). 둘 다 "권장 출력 확장" 한 흐름.
- **7번-G ARCHITECTURE 백엔드 sub-section 자리** `[가설]` — `## 7-1. 백엔드 결정` 8 prompt(DB migration / 인증 / 트랜잭션 / Idempotency / Rate limit / Async / Caching / API versioning). 1번 DESIGN_SYSTEM 그룹 패턴 재활용 — 비백엔드는 통째 삭제 가능. ADR 없이 템플릿 amend.
- **7번-H ARCHITECTURE 프론트 sub-section 자리** `[가설]` — `## 7-2. 프론트 결정` 7 prompt(라우팅 / 상태관리 / SSR-CSR / i18n / SEO / 인증 / 폼 validation). 1번이 *시각*만 다룸 → 본 결정은 *기술 결정*. ADR 없이 템플릿 amend.
- **7번-I bootstrap-stack monorepo 라운드** `[외부실증]` — skill 본문 1단락(orchestrator: turbo/nx/pnpm-only / shared 패키지 / publish). ADR 없이 skill 본문 갱신.
- **7번-J plan-workitem sizing/분리 가이드** `[관측됨]` — skill 본문 1단락(monorepo: 패키지당 5 파일 / OpenAPI·migration task 분리). 4번 sizing 휴리스틱이 monorepo·백엔드에서 깨지는 *관측된* 문제 해소.

### P2 — 트리거 도달 시 재검토

- **6번 ADR 카테고리화** `[외부실증]` — ADR ≥ 15 시 트리거. 현재 10개로 YAGNI.
- **6번 codebase 복잡도 예산** `[가설]` — 측정-경고 only.
- **6번 agent concurrency guard** `[가설]` — 같은 task에 multi-fork 시 worktree 강제.
- **6번 validator + reviewer 통합 가능성** `[가설→데이터 트리거]` — 시뮬레이션이 출력 중복률 ≥ 30% 보고 시 통합 ADR 검토.
- **3번 00-meta 이름 변경** `[가설]` — Diátaxis 가이드상 폴더명보다 모드 라벨이 더 강한 신호. 보류 정당.
- **(cut) 4번 risk·effort 추정** — 보일러플레이트가 정확도 보장 불가. 사용자 프로젝트 단위 자유 추가. ADR-006 YAGNI.
- **(cut) 4번 2-pass planning** — 토큰 2배 + 5번 stabilize-milestone reviewer 책임 중복.
- **(cut) 4번 `/plan-workitem` → `/decompose-workitem` rename** — plan 모드 비범위 결정 후 충돌 = 자동완성 표면 1건. cross-ref 일괄 변경 비용 > 효용. skill `description`에 1줄로 충분.

#### 7번 스택 시뮬레이션 — P2 3개 + cut 3개

- **7번-K 프로젝트 language 명시** `[관측됨]` — charter `## 0. Status` 위 `> 작성 언어: <언어>` 1줄 + AGENTS.md 1줄. ADR로 박을 깊이 없음 — 가장 가벼운 enabling.
- **7번-L frontend +1 지표** `[가설]` — METRICS.md 5번-E에 bundle / Lighthouse / a11y 회귀 후보 1줄. 5번-E의 *4 ± 1* cap 정합. 강제 X.
- **7번-M PII/privacy 자리 안내** `[외부실증]` — ARCHITECTURE `## 8.보안` 안내문 1줄. 깊이 박지 않고 *위치 안내*만.
- **(cut N) `--pr` 플래그** `[관측됨]` — finalize 책임 경계가 commit까지(ADR-007). push/PR은 fork 사용자가 `.claude/settings.local.json` alias로 자율.
- **(cut O) Supabase RLS 자동 검증** `[가설]` — Supabase-특이 패턴은 보일러플레이트가 *모든 BaaS*를 다룰 수 없음 — 도구 중립(ADR-010) 깸.
- **(cut) 시나리오 5 비웹 직접 지원** — 7번-A로 단일 ADR-031에 통합 cut.

---

### ADR 후보 일람 — *결정 1개에 ADR 1개* (ADR-005 패턴 4 정합)

이전 초안의 ADR-011/012/013가 각 4~6 결정을 묶었던 것을 *단일 결정 1 ADR* 원칙으로 분할. ADR이 12개로 늘지만 (a) supersede 시 부분 뒤집기 가능, (b) 각 결정이 독립적으로 리뷰·인용 가능, (c) ADR 인덱스가 길어지면 6번 P2 *"ADR ≥ 15 시 카테고리화"* 트리거가 자연 발동 — *진짜 ratchet*(자라난 후에 정리).

**3번 — 문서 아키텍처**:
- **ADR-011** — AGENTS.md 100줄 hard cap + Ratchet 적용 (단일 결정).
- **ADR-012** — docs/00-meta 흡수 (9→5) + Diátaxis 모드 라벨 도입 (단일 결정 — 둘 다 *문서 정리* 한 흐름).
- **ADR-013** — `/stabilize-milestone`에 SSOT drift 점검 단계 추가 (단일 결정).

**5번 — Milestone graduation**:
- **ADR-014** — Milestone graduation contract (MILESTONE_TEMPLATE `## 5. 완료 기준` checklist 강화 + stabilize pre-check) (단일 결정).
- **ADR-015** — IMPROVEMENT_GUIDE Tech Debt Quadrant 재구조 (Fowler 4-quadrant) (단일 결정).
- **ADR-016** — METRICS.md 신설 + 4지표 + trend 악화 신호 (단일 결정).
- **ADR-017** — 시뮬레이션 dogfood 1회 의무 + 재실행 트리거 (단일 결정).

**6번 — 4 Pillars 정합**:
- **ADR-018** — CODE_LINEAGE.md (git footer 기반, 4 필드, 전량 재생성) + Conventional Commits footer 컨벤션 (단일 결정 — 둘 다 *git이 lineage SSOT* 한 흐름. ADR-008 amend로도 가능).
- **ADR-019** — Context Packs frontmatter + JIT 로딩 정책 (단일 결정).
- **ADR-020** — `validate --changed` (incremental validate) (단일 결정).
- **ADR-021** — `/stack-guard`가 정적 분석 도구 권장 (단일 결정 — 강제 X, 권장만).
- **ADR-022** — AC ID 컨벤션 강제 (`AC_N`/`[AC-N]` 누락 시 P1) — ADR-009 amend 형태로도 가능.

**1·4번 — 별도 결정**:
- 1번 본문 — 인터페이스 결정 책임 분배 ADR (DESIGN_SYSTEM.md→DESIGN.md rename·UI 한정, ARCHITECTURE 7-1/7-2 신설, `/bootstrap-design` 도입).
- **ADR-024** *(신설, 4번)* — Claude Code plan 모드 lifecycle 비범위 (settings·디렉터리·문서 4곳 정리 + AGENTS.md 1단락 + /implement-workitem 1줄 흡수).
- **ADR-026** *(신설, 4번)* — plan-workitem 강화 (TASK_TEMPLATE schema 4종 + planner skill 행동 2종 — 한 흐름).

**7번 — 스택별 시나리오 시뮬레이션** (3 신설 + 3 amend, 본문 6건은 ADR 없이 amend):
- **ADR-023** *(신설)* — MCP integration baseline + tool 권한 분류(read-only / mutation / admin) + secret 값 분류(`*_PUBLIC_*` vs `*_SERVICE_ROLE_*`) (결정 E — 횡단 A·B 시나리오 4).
- **ADR-025** *(신설)* — `/bootstrap-stack` 외부 의존 권장 + `/stack-guard` CI 권장 출력 (결정 F — 횡단 C·D).
- **ADR-031** *(신설)* — 비웹 스택을 *명시적 비목표*로 cut. 보일러플레이트 직접 지원 = web frontend / API server / CLI / monorepo / Supabase. mobile / ML / embedded / game / desktop은 fork 사용자가 stack-guard override (결정 A — 시나리오 5).
- **ADR-007 amend** — finalize 우선순위 (3) 제외 규칙에 lock file 화이트리스트 11종 (결정 B — 횡단 E).
- **ADR-008 amend** — 모노레포 scope = 패키지명(`feat(api):`·`feat(web):`·`feat(shared):`) (결정 C — 시나리오 3).
- **ADR-021 amend** — 정적 분석 권장 list에 `gitleaks` / `trufflehog` 1줄 추가 (결정 D — 횡단 B).
- **본문 amend (ADR 없이 6건)**: ARCHITECTURE 백엔드/프론트 sub-section(G·H) / bootstrap-stack monorepo 라운드(I) / plan-workitem sizing 가이드(J) / charter language 1줄(K) / METRICS frontend +1 지표(L) / ARCHITECTURE PII 안내(M) — 모두 단일 PR로 처리.

### 외부 자료 인덱스 (3·5·6번 본문에서 참조)
- Anthropic — [effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) / [harness design long-running apps](https://www.anthropic.com/engineering/harness-design-long-running-apps) / [effective harnesses](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) / [2026 Agentic Coding Trends Report](https://resources.anthropic.com/2026-agentic-coding-trends-report).
- Drew Breunig × Lance Martin — [context engineering 4 strategies](https://rlancemartin.github.io/2025/06/23/context_engineering/) / [10 lessons for agentic coding](https://www.dbreunig.com/2026/05/04/10-lessons-for-agentic-coding.html).
- Addy Osmani — [agent harness engineering](https://addyosmani.com/blog/agent-harness-engineering/) / [long-running agents](https://addyosmani.com/blog/long-running-agents/).
- Augment Code — [2,500-repo AGENTS.md analysis](https://www.augmentcode.com/blog/how-to-write-good-agents-dot-md-files).
- GitHub Blog — [2,500+ repo AGENTS.md study](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/).
- GitHub spec-kit / Agent OS / BMAD-Method — [Martin Fowler's SDD analysis](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html).
- Diátaxis — [official guide](https://diataxis.fr/) / [start here](https://diataxis.fr/start-here/).
- Martin Fowler — [Tech Debt Quadrant](https://martinfowler.com/bliki/TechnicalDebtQuadrant.html) / [ADR](https://martinfowler.com/bliki/ArchitectureDecisionRecord.html).
- DORA / SPACE — [dora.dev](https://dora.dev/guides/dora-metrics/) / [getdx.com SPACE](https://getdx.com/blog/space-metrics/).
- Atlassian — [Definition of Done](https://www.atlassian.com/agile/project-management/definition-of-done).
- Spotify × Anthropic — [agentic development at Spotify](https://engineering.atspotify.com/2026/4/anthropic-agentic-development).
- Geoffrey Huntley — [Ralph Wiggum](https://ghuntley.com/ralph/).
- Hindsight — [agent harness needs memory](https://hindsight.vectorize.io/blog/2026/05/04/agent-harness-needs-memory).
- Static analysis — [dependency-cruiser](https://www.npmjs.com/package/dependency-cruiser) / [ArchUnitTS](https://github.com/LukasNiessen/ArchUnitTS) / [Nx affected](https://nx.dev/docs/features/ci-features/affected).
