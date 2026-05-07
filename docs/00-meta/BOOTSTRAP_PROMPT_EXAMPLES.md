# Bootstrap Prompt Examples

## 프로젝트 초기화 예시 1
/bootstrap-project 개인 회고 SaaS. 사용자는 하루 회고와 주간 회고를 기록하고, 원인 분석과 개선 추적을 한다. 초기 타깃은 자기관리 욕구가 높은 직장인과 학생. 아직 스택은 미정이고, 모바일 우선 UX를 원한다.

## 프로젝트 초기화 예시 2
/bootstrap-project 취준생 커리어 관리 서비스. 사용자는 JD와 이력서를 비교하고 부족한 역량을 추적하며, 주간 학습 액션 플랜을 관리한다. 초기 타깃은 한국 취준생. 배포는 웹 우선이며 스택은 아직 미정이다.

## 스택 세팅 예시 1
/bootstrap-stack Next.js 16 + TypeScript + pnpm + Supabase + Playwright + Vercel

## 스택 세팅 예시 2
/bootstrap-stack FastAPI + PostgreSQL + pytest + Ruff + Docker + GitHub Actions

## /discover-product 라운드 인터랙션 예시

```
사용자: /discover-product 개인 회고 SaaS. 사용자는 하루/주간 회고를 기록하고, 원인 분석과 개선 추적을 한다.

[R0] 메인이 한 줄을 되돌리고 페르소나 후보 3개 제시
사용자: "1번 + 직장인 강조로 합쳐줘"

[R1] 메인이 pain 8개를 빈도×고통으로 정렬해 제시
사용자: "1번, 3번이 핵심"
[R1 계속] JTBD 한 줄 + happy/alternate/fail 5단계 작성

[R2] MVP 범위 vs 비범위, 성공 기준 1~3개
사용자: "good"

[R3] 가정 5개, 위험한 가정 2개에 검증 방법 1줄
사용자: "good"

[R4] DISCOVERY.md 작성
출력: docs/10-charter/DISCOVERY.md 저장됨. 다음 액션: /bootstrap-project (DISCOVERY.md 사용)
```
