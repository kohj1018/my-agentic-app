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
새 프로젝트 시작 체크리스트는 [NEW_PROJECT_CHECKLIST.md](NEW_PROJECT_CHECKLIST.md)가 SSOT다. 본 문서에서는 정의를 복제하지 않는다.

## 네이밍 규칙
- 마일스톤: `M1-xxx.md`, `M2-xxx.md`
- 기능: `F-001-xxx.md`, `F-002-xxx.md`
- 작업: `T-001-xxx.md`, `T-002-xxx.md`
- ADR: `ADR-001-xxx.md`, `ADR-002-xxx.md`

## 문서 연결 원칙
- 상위 문서는 하위 문서를 링크한다.
- 기능 문서는 관련 마일스톤, 설계 문서, ADR을 링크한다.
- QA 문서는 기능/작업 ID를 기준으로 역참조한다.

## 문서 상태값
workitem 상태값과 전이 규칙은 [WORKFLOW.md#문서-상태-전이](WORKFLOW.md#문서-상태-전이)가 SSOT다.
ADR 전용 상태값(`proposed`/`accepted`/`superseded`/`deprecated`)은 [_ADR_GUIDE.md](../90-decisions/_ADR_GUIDE.md)가 SSOT다.

## Guardrail 운영 원칙
shared 기본값과 stack-specific 자동화의 분리 원칙은 [GUARDRAILS_STRATEGY.md](GUARDRAILS_STRATEGY.md)가 SSOT다.

## 새 프로젝트 시작 체크리스트
실제 새 프로젝트를 시작할 때는 docs/00-meta/NEW_PROJECT_CHECKLIST.md를 기준으로 진행한다.

## 산출물 인벤토리
모든 산출물의 위치·생성 주체·라이프사이클은 [STRUCTURE.md](STRUCTURE.md)가 SSOT다.

## 권장 시작 방식
이 템플릿은 수동으로 문서를 하나씩 고치기보다,
Claude Code에서 /bootstrap-project [프로젝트 설명]으로 시작하는 것을 권장한다.

스택이 확정되면 /bootstrap-stack [스택 설명]으로 다음 단계를 진행한다.
