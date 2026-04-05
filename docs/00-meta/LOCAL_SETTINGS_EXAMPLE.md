# Local Settings Example

## 목적
개인별 hook, shell 경로, 실험적 자동화는 shared 설정이 아니라
`.claude/settings.local.json`에 둔다.

## 예시 흐름
1. `.claude/settings.local.json` 파일을 직접 만든다.
2. 본인 환경에서만 동작하는 hook를 넣는다.
3. 이 파일은 Git에 커밋하지 않는다.

## 예시 상황
- Windows에서만 PowerShell hook를 쓰고 싶을 때
- macOS/Linux에서만 bash hook를 쓰고 싶을 때
- 특정 로컬 도구 경로를 잡아야 할 때
- 실험적인 guardrail을 먼저 써보고 싶을 때

## 주의
shared 템플릿에 local 자동화를 바로 반영하지 않는다.
팀 전체에 적용할 검증은 프로젝트 스택이 확정된 뒤 별도로 설계한다.
