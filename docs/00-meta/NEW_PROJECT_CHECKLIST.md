# New Project Checklist

## 1. 저장소 복제 직후
- [ ] Claude Code에서 `/bootstrap-project [프로젝트 설명]`을 실행했다
- [ ] `README.md`가 새 프로젝트 기준으로 갱신되었다
- [ ] `docs/10-charter/PROJECT_CHARTER.md`가 새 프로젝트 내용으로 채워졌다
- [ ] `docs/20-system/ARCHITECTURE_OVERVIEW.md`가 초기 구조를 반영한다
- [ ] 첫 milestone/feature 문서가 생성되었다

## 2. 작업 구조 준비
- [ ] `docs/30-workitems/milestones`에 첫 milestone 문서가 있다
- [ ] `docs/30-workitems/features`에 첫 feature 문서가 있다
- [ ] 필요하면 `docs/30-workitems/tasks`에 task 문서를 만들었다

## 3. 운영 결정
- [ ] 운영 OS/셸 전제를 정했다
- [ ] 언어/프레임워크를 정했다
- [ ] 패키지 매니저를 정했다
- [ ] 테스트 도구를 정했다
- [ ] lint/typecheck 도구를 정했다

## 4. guardrail 추가
- [ ] `docs/00-meta/GUARDRAILS_STRATEGY.md`를 읽었다
- [ ] 스택이 정해진 뒤 `/bootstrap-stack [스택 설명]`을 실행했다
- [ ] 필요하면 `.claude/settings.local.json`에 개인 자동화를 추가했다
- [ ] shared 설정에 환경 종속적인 hook를 바로 넣지 않았다

## 5. 의사결정 기록
- [ ] 중요한 선택을 `docs/90-decisions`에 ADR로 남겼다

## 6. 첫 커밋 전
- [ ] 예전 프로젝트 예시 문구가 남아 있지 않다
- [ ] 불필요한 템플릿 placeholder가 과하게 남아 있지 않다
- [ ] 새 프로젝트의 핵심 범위와 비범위가 명확하다

## 권장 원칙
- 먼저 수동으로 여러 문서를 고치기보다 /bootstrap-project를 먼저 실행한다.
- 스택이 정해지기 전에는 stack-specific 자동화를 추가하지 않는다.
- 중요한 기획/설계 변경은 Opus 기반 흐름을 우선 사용한다.

## 실행 원칙
- 메인 세션이 모든 구현을 직접 처리하지 않는다.
- 문서 작성, 구현, 검증, QA는 가능한 한 관련 서브에이전트/skill에 먼저 위임한다.
- 메인 세션은 목표 정리, 결과 통합, 다음 결정에 집중한다.
