# New Project Checklist

## 1. 저장소 복제 직후
- [ ] 프로젝트 이름에 맞게 `README.md`를 수정했다
- [ ] `docs/10-charter/PROJECT_CHARTER.md`를 새 프로젝트 내용으로 바꿨다
- [ ] `docs/20-system/ARCHITECTURE_OVERVIEW.md`를 새 프로젝트 기준으로 정리했다
- [ ] 필요하면 `docs/20-system/DESIGN_SYSTEM.md`를 채웠다
- [ ] 필요 없는 예시 문구를 제거했다

## 2. 작업 구조 준비
- [ ] `docs/30-workitems/milestones`에 첫 milestone 문서를 만들었다
- [ ] `docs/30-workitems/features`에 첫 feature 문서를 만들었다
- [ ] 필요하면 `docs/30-workitems/tasks`에 task 문서를 만들었다

## 3. 운영 결정
- [ ] 운영 OS/셸 전제를 정했다
- [ ] 언어/프레임워크를 정했다
- [ ] 패키지 매니저를 정했다
- [ ] 테스트 도구를 정했다
- [ ] lint/typecheck 도구를 정했다

## 4. guardrail 추가
- [ ] `docs/00-meta/GUARDRAILS_STRATEGY.md`를 읽었다
- [ ] 프로젝트 스택에 맞는 scripts를 만들었다
- [ ] 필요하면 `.claude/settings.local.json`에 개인 자동화를 추가했다
- [ ] shared 설정에 환경 종속적인 hook를 바로 넣지 않았다

## 5. 의사결정 기록
- [ ] 중요한 선택을 `docs/90-decisions`에 ADR로 남겼다

## 6. 첫 커밋 전
- [ ] 예전 프로젝트 예시 문구가 남아 있지 않다
- [ ] 불필요한 템플릿 placeholder가 과하게 남아 있지 않다
- [ ] 새 프로젝트의 핵심 범위와 비범위가 명확하다
