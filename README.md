<!-- 구조 변경 시 README.md와 README_ko.md를 동시에 갱신한다. drift를 막기 위해 두 README 본문은 짧게 유지하고 깊은 정의는 docs/ 링크로 둔다. -->
# Claude Code Agentic Boilerplate

**Language: English | [한국어](README_ko.md)**

A boilerplate that sets up the document structure and sub-agent workflow all at once when starting a new project with Claude Code.

> **In short**: Fork this repo → optionally run `/discover-product` to ground your charter in real user data → run `/bootstrap-project` → get charter, architecture, and initial workitems in one shot. The main session orchestrates; sub-agents do the work.

## Who is this for

- Individual developers who frequently start new projects
- Teams wanting to standardize document structure and work breakdown
- Users who want the main session to delegate via sub-agents rather than carrying all context

## Overall Flow

```
/discover-product (optional)
  → /bootstrap-project → /bootstrap-stack → /stack-guard
  → /plan-workitem → /implement-workitem
  → /validate-workitem → /repair-workitem (if Needs Fix) → /finalize-workitem
  → /stabilize-milestone
```

For step-by-step details, see [WORKFLOW.md](docs/00-meta/WORKFLOW.md).
For sub-agent delegation, see [DELEGATION_STRATEGY.md](docs/00-meta/DELEGATION_STRATEGY.md).
The Quick Start below walks through these commands as Steps 0–3.

`/discover-product` is recommended for new projects to ground charter in concrete persona/pain/scenarios. It writes `docs/10-charter/DISCOVERY.md`, which `/bootstrap-project` then converts into charter/architecture/initial workitems. For a quick prototype, you can skip `/discover-product` and pass a natural-language brief directly to `/bootstrap-project`.

## Quick Start

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) must be installed
- Apply this repository as a GitHub template to a new repo, or clone it to get started

### Step 0 (Optional): Discover

Run discovery to ground your charter in real persona/pain data:

```text
/discover-product [product description]
```

Skip this step for quick prototypes — pass a brief directly to `/bootstrap-project` instead.

### Step 1: Initialize Project

```text
/bootstrap-project [project brief or empty to use DISCOVERY.md]
```

Generates: `README.md`, `docs/10-charter/PROJECT_CHARTER.md`, `docs/20-system/ARCHITECTURE_OVERVIEW.md`, and initial milestone/feature documents.

**Tip — what to include in the brief**: what you're building, who uses it, the problem it solves, and what's already decided vs. undecided. See [PROJECT_START_CHECKLIST.md](docs/00-meta/PROJECT_START_CHECKLIST.md) for examples.

### Step 2: Set Up Stack

Once your stack is decided:

```text
/bootstrap-stack [stack/runtime description]
/stack-guard
```

`/bootstrap-stack` documents stack choices and outlines needed automation. Then run `/stack-guard` after reviewing `STACK_SETUP_PLAN.md` — it generates the unified `validate` entrypoint and verify scripts.

### Step 3: Plan → Implement → Ship

```text
# Plan + implement
/plan-workitem [milestone or feature id]
/implement-workitem [task id]
/validate-workitem [task id]

# If Pass: finalize and move on
/finalize-workitem [task id]

# If Needs Fix: repair, then re-validate
/repair-workitem [task id]
/validate-workitem [task id]

# Once all tasks in the milestone are done:
/stabilize-milestone [milestone id]
```

## Using with Codex CLI (alternate entry)

When you hit Claude Code's usage limit or prefer Codex:

1. Run `codex` in the same repo — `AGENTS.md` is auto-loaded.
2. Documents and policies are equal. Core workflow skills have Codex wrappers ($-prefixed): $implement-workitem, $validate-workitem, $repair-workitem, $finalize-workitem, $plan-workitem, $bootstrap-project, $bootstrap-stack, $stabilize-milestone. Remaining skills (discover-product, stack-guard, review-doc, boilerplate-context, bootstrap-design) are invoked via natural language. See [WORKFLOW.md](docs/00-meta/WORKFLOW.md).
3. Core workflow skills are callable via Codex Skills:
   - Inner loop: `$implement-workitem T-001`, `$validate-workitem T-001`, `$repair-workitem T-001`, `$finalize-workitem T-001`
   - Planning / bootstrap / stabilize: `$plan-workitem M1`, `$bootstrap-project <brief>`, `$bootstrap-stack <stack>`, `$stabilize-milestone M1`
4. For remaining skills (`discover-product`, `stack-guard`, `review-doc`, `boilerplate-context`, `bootstrap-design`), invoke in natural language: *"Follow `.claude/skills/<name>/SKILL.md`"*.

> Note: docs in `docs/` use Claude's `/<skill-name>` slash syntax. Read these as `$<skill-name>` when working in Codex.

For full policy, see [ADR-010](docs/90-decisions/boilerplate/ADR-010-multi-agent-compatibility.md).

## Structure

For a full inventory of all artifacts (location, owner, lifecycle), see [STRUCTURE.md](docs/00-meta/STRUCTURE.md).

```
.
├── AGENTS.md          # Canonical entry instructions (tool-neutral)
├── CLAUDE.md          # Imports AGENTS.md (Claude Code entry)
├── .claude/           # Sub-agents, skills, settings
├── docs/
│   ├── 00-meta/       # Workflow, guardrails, templates, operational guides
│   ├── 10-charter/    # Project scope, goals, problem definition
│   ├── 20-system/     # Architecture overview, UI design
│   ├── 30-workitems/  # Milestones, features, tasks
│   ├── 40-validation/ # QA findings, improvement guide, reports
│   └── 90-decisions/  # ADR records
├── scripts/           # Project-specific automation (after stack is chosen)
└── .boilerplate/      # Boilerplate self-validation/meta artifacts. Read-only after fork. Not a project artifact.
```

## Guardrail Principles

This template prioritizes cross-platform reusability — shared base settings do not include OS/shell/runtime-dependent hooks. See [GUARDRAILS_STRATEGY.md](docs/00-meta/GUARDRAILS_STRATEGY.md) for details.

## Where to Start

- [PROJECT_START_CHECKLIST.md](docs/00-meta/PROJECT_START_CHECKLIST.md) — New project startup checklist (with input examples)
- [STRUCTURE.md](docs/00-meta/STRUCTURE.md) — Document structure, naming conventions, and artifact inventory
- [WORKFLOW.md](docs/00-meta/WORKFLOW.md) — Step-by-step workflow

## Contributing

For improvement suggestions or bug reports, see the [issue templates](.github/ISSUE_TEMPLATE). For structural changes, see the [PR template](.github/PULL_REQUEST_TEMPLATE).

## License

MIT
