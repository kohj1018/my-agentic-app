# ADR-012 — docs/00-meta 문서 아키텍처 정리

> scope: boilerplate

## Status
accepted

## 배경
- [외부실증] Diátaxis (https://diataxis.fr) — 문서를 Tutorial / How-to / Reference / Explanation 4종으로 분류하면 독자가 "이 문서에서 무엇을 얻을 수 있는가"를 즉시 파악한다.
- [관측됨] `docs/00-meta/`에 9개 문서 중 4개(TEMPLATE_GUIDE / LOCAL_SETTINGS_EXAMPLE / BOOTSTRAP_PROMPT_EXAMPLES / 30-workitems/README)가 다른 문서의 내용과 중복되어 SSOT 위반(ADR-005).
- 흡수 후에도 모드 라벨 없이 6개 파일이 남으면 독자가 "읽어야 할 문서인가 / 참고 문서인가"를 파악하기 어렵다.

## 결정

### 결정 A — docs/00-meta 9→6 흡수 (Step 3.2)
다음 4개 파일을 흡수 후 삭제한다.
- `TEMPLATE_GUIDE.md` → STRUCTURE.md(네이밍 규칙) + bootstrap-project skill 참조 교체
- `LOCAL_SETTINGS_EXAMPLE.md` → GUARDRAILS_STRATEGY.md "local 자동화 권장 원칙" 단락
- `BOOTSTRAP_PROMPT_EXAMPLES.md` → NEW_PROJECT_CHECKLIST.md 1단계 예시 코드블록
- `docs/30-workitems/README.md` → 단순 삭제 (STRUCTURE.md가 SSOT)

### 결정 B — Diátaxis 모드 라벨 추가 (Step 3.3)
남은 6개 파일 각각에 첫 줄 모드 라벨 추가:

| 파일 | 모드 |
|------|------|
| `STRUCTURE.md` | Reference (산출물 인벤토리) |
| `WORKFLOW.md` | Reference + How-to (워크플로우 정의 + 단계별 절차) |
| `DELEGATION_STRATEGY.md` | Reference (위임 트리거 + 메인 세션 역할) |
| `GUARDRAILS_STRATEGY.md` | Explanation (guardrail 운영 원칙의 근거) |
| `NEW_PROJECT_CHECKLIST.md` | How-to (새 프로젝트 시작 체크리스트) |
| `GLOSSARY.md` | Reference (용어 정의) |

### 비결정 (영구 No)
- `docs/00-meta/` 폴더명 변경 — ADR-001 계층 정의와 충돌, 불필요한 마이그레이션 비용.
- `_templates/` 통합 — 템플릿은 각 계층에 분산이 자연스럽다(SSOT는 계층별 _templates).

## 결과
- `docs/00-meta/`에 정확히 6개 파일.
- 각 파일 첫 줄에 모드 라벨 → 독자가 1초 안에 문서 성격 파악.
- `/review-doc`이 모드 mismatch 발견 시 IMPROVEMENT_GUIDE에 P1로 자동 보고.

## 후속 작업
없음

## 참고
- ADR-005 (SSOT 원칙)
- ADR-022 (Ratchet Principle — [외부실증] 라벨)
- Diátaxis: https://diataxis.fr
