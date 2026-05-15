# ADR-021 — `/stack-guard` 정적 분석 권장 + secret scanner

> scope: boilerplate

## Status
accepted

## 배경
- [외부실증] dependency-cruiser / import-linter — layer 경계 위반을 코드 리뷰 없이 자동 검출.
- [외부실증] gitleaks / trufflehog — secret hardcode 검출 부재가 보안 사고의 단일 최대 원인 중 하나.
- `/stack-guard`가 정적 분석 도구를 권장하지 않아 layer 경계 위반·secret hardcode 검출 자리 없음.

## 결정

### 1. 스택별 정적 분석 1종 권장
paralysis 방지 원칙: 대안 나열 X, 스택별 1종 권장.

| 스택 | 도구 | 비고 |
|------|------|------|
| TypeScript / JS | `dependency-cruiser` | layer 위반 룰을 ARCHITECTURE_OVERVIEW `## 3-1` 채움 시 함께 권장 |
| Python | `import-linter` | 동일 layer 룰 패턴 |
| Go | `go vet` (built-in) | 후속 보강 가능 |
| Rust | `cargo deny` + `cargo udeps` | unused deps + license/advisory |

### 2. 적용 원칙
- *강제 X, 권장만* (ADR-010 multi-tool 호환, GUARDRAILS_STRATEGY "OS/셸 종속 hook 강제 X" 정신 정합).
- `validate` 명령에 lint 단계로 통합 → CI fail로 잡힘.
- 영역별 lint(UI=design tokens / API=Pact / CLI=snapshot test)는 *과설계로 cut* — Step 5.4의 builder/validator self-check가 동등 효용을 더 가벼운 비용으로 확보.

## 결과
- `/stack-guard` 종료 출력에 스택별 권장 정적 분석 1종 + `validate` 통합 방법 포함.
- layer 경계 위반을 코드 리뷰 전에 검출 가능.

## 후속 작업
없음

## 참고
- ADR-022 (Ratchet Principle — [외부실증] 라벨)
- ADR-010 (multi-tool 호환)
- GUARDRAILS_STRATEGY.md

## Amendment 1 (2026-05-15) — secret scanner 추가

### 결정
- `gitleaks` / `trufflehog` 1종 권장 (전 스택).
- finalize 직전 staged 파일 점검.
- *강제 X, 권장만*.

### 근거
- secret hardcode 검출 부재 빈자리 보강.
- `/stack-guard` 출력에 secret scanner 권장 단락 추가.
