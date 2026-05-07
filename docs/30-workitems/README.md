# 작업 단위 문서 가이드

## 목적
이 디렉터리는 실제 구현과 검증이 가능한 작업 단위 문서를 저장한다.

## 디렉터리 구성
- `milestones`: 큰 단계 목표
- `features`: 사용자 가치 단위 기능 문서
- `tasks`: 실제 구현 단위 작업 문서
- `plans`: Claude Code Plan 모드가 자동 생성하는 실행 계획. 경로는 `.claude/settings.json`의 `plansDirectory`가 canonical이다(현재 `./docs/30-workitems/plans`).
- `_templates`: 새 문서를 만들 때 복사할 템플릿

## 권장 생성 순서
1. milestone 생성
2. feature 생성
3. task 생성

## 연결 원칙
- milestone은 여러 feature를 가질 수 있다.
- feature는 여러 task를 가질 수 있다.
- task는 관련 milestone/feature/ADR를 링크한다.

## 상태 관리
workitem 상태값과 전이 규칙은 [../00-meta/WORKFLOW.md#문서-상태-전이](../00-meta/WORKFLOW.md#문서-상태-전이)가 SSOT다.
