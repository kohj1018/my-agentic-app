# 템플릿 가이드

## 목적
이 저장소는 Claude Code 기반의 문서 중심 개발 보일러플레이트다.
새 프로젝트를 시작할 때 이 저장소를 복제한 뒤, 문서 구조와 에이전트 구조를 그대로 재사용하는 것을 목표로 한다.

## 문서 계층
- `docs/00-meta`: 템플릿 사용법과 공통 운영 규칙
- `docs/10-charter`: 프로젝트 범위, 목표, 문제 정의
- `docs/20-system`: 시스템 구조, 설계 개요, 디자인 시스템
- `docs/30-workitems`: 마일스톤, 기능, 작업 단위 문서
- `docs/40-validation`: QA 결과, 개선 가이드
- `docs/90-decisions`: 중요한 의사결정 기록

## 새 프로젝트 시작 순서
1. `docs/10-charter/PROJECT_CHARTER.md`를 프로젝트에 맞게 수정한다.
2. `docs/20-system/ARCHITECTURE_OVERVIEW.md`를 작성한다.
3. 필요 시 `docs/20-system/DESIGN_SYSTEM.md`를 채운다.
4. `docs/30-workitems/_templates`를 참고해 milestone/feature/task 문서를 만든다.
5. 구현 후 `docs/40-validation/QA_FINDINGS.md`와 `docs/40-validation/IMPROVEMENT_GUIDE.md`를 갱신한다.
6. 중요한 선택은 `docs/90-decisions`에 ADR로 남긴다.

## 네이밍 규칙
- 마일스톤: `M1-xxx.md`, `M2-xxx.md`
- 기능: `F-001-xxx.md`, `F-002-xxx.md`
- 작업: `T-001-xxx.md`, `T-002-xxx.md`
- ADR: `ADR-001-xxx.md`, `ADR-002-xxx.md`

## 문서 연결 원칙
- 상위 문서는 하위 문서를 링크한다.
- 기능 문서는 관련 마일스톤, 설계 문서, ADR을 링크한다.
- QA 문서는 기능/작업 ID를 기준으로 역참조한다.

## 상태값 예시
- draft
- ready
- in-progress
- blocked
- done
- deprecated

## Guardrail 운영 원칙
기본 보일러플레이트는 cross-platform 재사용성을 위해
OS/셸/런타임 종속적인 hook와 검증 스크립트를 shared 기본값으로 포함하지 않는다.

자동화 전략은 docs/00-meta/GUARDRAILS_STRATEGY.md를 따른다.
프로젝트의 실제 스택이 정해진 뒤 그 스택에 맞는 scripts/hooks/CI를 생성한다.
