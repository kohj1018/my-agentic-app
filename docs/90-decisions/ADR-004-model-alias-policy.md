# ADR-004 모델 별칭 우선 정책

> scope: boilerplate

## 상태
accepted

## 배경
이 보일러플레이트의 핵심 가치는 "여러 프로젝트에서 반복 재사용"이다.
모델 ID를 전체 버전 표기(`claude-opus-4-N` 같은 형태)로 고정하면, fork된 새 프로젝트에서
시간이 지남에 따라 staleness가 누적되어 모델 갱신을 사람이 매번 기억해야 한다.

`.claude/settings.json`, agent 정의, skill 정의에 모델 표기가 흩어져 있고,
일부는 별칭(`sonnet`)을, 일부는 전체 ID를 사용하고 있어 표기 일관성이 없다.

## 결정
shared 기본값에서는 모델 별칭(`sonnet`, `opus`, `haiku`)만 사용한다.
특정 버전을 강제해야 하는 이유가 있으면 별도 ADR로 남기고 그 자리에서만 전체 ID를 사용한다.

## 근거
- Claude Code의 별칭은 자동 최신 매핑을 제공한다([model-config 문서](https://code.claude.com/docs/en/model-config)).
- 보일러플레이트의 "재사용 가능" 약속과 자동 최신 매핑이 가장 잘 맞는다.
- 사람이 모델 갱신을 잊어 staleness가 누적되는 것을 저비용으로 막는다.

## 결과
- `.claude/settings.json`: `"model": "opus"` (default 별칭 — 어느 별칭이 default인지는 본 ADR의 강제 사항이 아니다. 정책 본질은 "shared 기본값에서 별칭만 쓴다".)
- `.claude/agents/architect-opus.md`: `model: opus`
- `.claude/skills/bootstrap-project/SKILL.md`, `.claude/skills/bootstrap-stack/SKILL.md`: `model: opus`
- 다른 sub-agent의 `model: sonnet` 표기는 그대로 유지(이미 별칭).

## 후속 작업
- 별칭 정책을 깰 때(특정 버전 강제)의 절차: 새 ADR로 이유와 갱신 책임자를 기록하고, 그 자리에서만 전체 ID를 사용한다.
- 사용자가 fork 직후 자기 환경의 비용 정책을 강제해야 하면 `.claude/settings.local.json`에서 model을 override한다.
- **Provider별 별칭 해석 차이 주의**: Anthropic API와 Bedrock/Vertex/Foundry에서 별칭이 매핑되는 ID·시점이 다를 수 있다. 특정 provider에서 재현성이 중요한 시점(릴리스 직전, 회계 감사 등)에는 그 provider 환경 변수 또는 settings 단에서 전체 ID로 임시 pinning한다(별도 ADR로 기록).
