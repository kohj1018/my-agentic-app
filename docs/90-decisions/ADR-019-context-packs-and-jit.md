# ADR-019 — Context Packs frontmatter + JIT 로딩

> scope: boilerplate

## Status
accepted

## 배경
- [외부실증] Anthropic effective context engineering — 매 task마다 모든 ADR/architecture를 fork-load하면 컨텍스트 창 낭비 + 추론 노이즈 증가. 최소 충분(minimal sufficiency) 원칙이 agent 품질 상한을 올린다.
- [관측됨] 각 skill의 "반드시 먼저 읽을 파일" 목록이 *최소 충분*인지 점검 없음 → 과도한 사전 로딩 위험.

## 결정

### 1. JIT 로딩 정책 명문화
`반드시 먼저 읽을 파일`은 *최소 충분*. 추가 ADR/architecture 섹션은 task 본문에서 발화 시 인용 — 사전 fork-load 금지.

모든 skill SKILL.md 본문 끝에 `## Context 정책 (ADR-019)` 섹션으로 명문화.

### 2. Context Packs 2종

| pack | 포함 | 용도 |
|------|------|------|
| **minimal** *(default)* | AGENTS.md + task 본문 | 모든 일반 skill |
| **full** | 모든 docs/ | architect-opus 디폴트 |

- skill frontmatter에 `context-pack: minimal` 필드 추가 (13개 skill 일괄).
- architect-opus agent frontmatter에 `context-pack: full`.

## 비결정 (No)
- frontend/backend 영역별 pack 사전 정의 — 과설계. 사용자가 필요 시 fork 프로젝트에서 자체 정의.

## 토큰 절감 추정
- minimal: ~5K / full: ~30K → 호출당 5~25K 절감.

## 결과
- 모든 skill이 JIT 로딩 정책을 명문화 → 과도한 사전 로딩 방지.
- context-pack frontmatter로 도구가 로딩 범위를 명시적으로 제어 가능.

## 후속 작업
없음

## 참고
- ADR-022 (Ratchet Principle — [외부실증] 라벨)
- ADR-010 (multi-tool 호환)
