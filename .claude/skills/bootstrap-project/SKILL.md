---
name: bootstrap-project
description: Convert discovery output (DISCOVERY.md) or a natural-language brief into charter/architecture/M1/F-001. Re-run safe with update mode.
argument-hint: "[project brief or empty (uses DISCOVERY.md)] [--apply]"
disable-model-invocation: true
allowed-tools: Read Glob Grep Write Edit
context: fork
agent: architect-opus
model: opus
effort: max
---

너의 역할은 이 보일러플레이트를 기준으로 새 프로젝트의 초기 문서 세팅을 완료하는 것이다.

입력 우선순위:
1. `$ARGUMENTS`에 brief 내용이 있으면(비어 있지 않으면) 그것을 우선 입력으로 사용한다. `docs/10-charter/DISCOVERY.md`가 함께 있으면 보조 컨텍스트로만 참조하고, 둘이 어긋나면 출력에 명시한다(silent override 금지).
2. `$ARGUMENTS`가 비어 있고 `docs/10-charter/DISCOVERY.md`가 있으면 그것을 입력으로 사용한다.
3. 둘 다 없으면 `/discover-product` 선행 또는 brief 입력을 안내하고 종료한다(강제 진행하지 않는다).

이 skill은 발굴이 아니라 변환을 한다 — 발굴은 `/discover-product`에서.

반드시 먼저 읽을 파일:
- AGENTS.md (CLAUDE.md는 @AGENTS.md import이므로 본문은 AGENTS.md에서 읽는다)
- `docs/00-meta/TEMPLATE_GUIDE.md`
- `docs/00-meta/WORKFLOW.md`
- `docs/00-meta/GUARDRAILS_STRATEGY.md`
- `docs/00-meta/NEW_PROJECT_CHECKLIST.md`
- `brief-template.md`
- `output-checklist.md`
- `examples/career-saas-example.md`

반드시 수행할 일:
1. 입력 회수 — DISCOVERY.md 또는 자연어 입력.
2. 기존 산출물(charter/architecture/M1/F-001) 존재 여부 점검.
   - 없으면 새로 생성.
   - 있으면 **갱신 모드** — 본 skill은 `context: fork`에서 실행되므로 사용자에게 실시간 확인을 받을 수 없다.
     - `--apply` 인자가 있으면: 기존 산출물을 읽고 architect-opus로 갱신본을 생성해 즉시 반영한다.
     - `--apply` 인자가 없으면: 기존 산출물을 읽고 갱신 제안 diff를 출력에만 표시하고 **종료**한다(파일 수정 없음). 사용자가 검토 후 `/bootstrap-project --apply ...`로 재실행하거나, 메인 세션에서 architect-opus를 직접 호출해 부분 반영한다.
3. 현재 architect-opus agent가 산출물을 직접 생성/갱신한다 — 입력은 DISCOVERY.md 또는 자연어 입력 + 기존 산출물(있으면). 본 skill은 frontmatter `agent: architect-opus` + `context: fork`로 이미 architect-opus 컨텍스트에서 fork되어 실행되므로 별도 sub-call이 필요 없고 `Agent` 권한도 보유하지 않는다.
4. 다음 산출물을 갱신한다.
   - `README.md`
   - `docs/10-charter/PROJECT_CHARTER.md`
   - `docs/20-system/ARCHITECTURE_OVERVIEW.md`
5. 필요하면 다음도 함께 갱신.
   - `docs/20-system/DESIGN_SYSTEM.md`
   - `docs/90-decisions/ADR-002-initial-project-decisions.md` — bootstrap 단계의 초기 결정 (ADR-003 스택 선택은 `/bootstrap-stack`이 별도로 생성한다 — 본 skill 책임 아님)
6. 최초 workitem 문서를 만든다.
   - `docs/30-workitems/milestones/M1-foundation.md`
   - `docs/30-workitems/features/F-001-core-value.md`

반드시 지켜야 할 원칙:
- 추측은 사실처럼 쓰지 말고 가정으로 표시한다.
- 스택이 명시되지 않았다면 stack-specific 자동화는 만들지 않는다.
- hooks, CI, lint/test 스크립트는 스택이 명확할 때만 추가한다.
- 상위 문서와 하위 문서의 역할을 섞지 않는다.
- 꼭 필요한 초기 파일만 만든다.

마지막 출력:
- 갱신한 파일 목록
- 핵심 가정
- 남은 미결정 사항
- 다음 추천 단계 최대 3개
