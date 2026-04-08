# Claude Code Agentic Boilerplate

**Language: English | [한국어](README_ko.md)**

A boilerplate that sets up the document structure and sub-agent workflow all at once when starting a new project with Claude Code.

It lets you consistently repeat the flow of scope definition → system design → work breakdown → implementation → validation.
The main session focuses on orchestration, delegating actual work to sub-agents and skills by default.

## Who is this for

- Individual developers who frequently start new projects
- Teams wanting to standardize document structure and work breakdown
- Users who want the main session to delegate via sub-agents rather than carrying all context

## Quick Start

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) must be installed
- Apply this repository as a GitHub template to a new repo, or clone it to get started

### Step 1: Project Initialization

Open Claude Code in your new repository and run:

```text
/bootstrap-project [project description]
```

Example:

```text
/bootstrap-project A personal career management SaaS. Users compare JDs with resumes, track skill gaps, and manage weekly action plans. Initial target is job seekers. Stack is undecided.
```

This command automatically drafts the following documents and creates the project's starting structure:

- `README.md`
- `docs/10-charter/PROJECT_CHARTER.md`
- `docs/20-system/ARCHITECTURE_OVERVIEW.md`
- Initial milestone / feature documents

### Step 2: Set Up After Choosing a Stack

Once your stack is decided, run:

```text
/bootstrap-stack [stack/runtime description]
```

Example:

```text
/bootstrap-stack Next.js 16 + TypeScript + pnpm + Supabase + Playwright + Vercel
```

This documents the stack choices and outlines the direction for automation and guardrails.

## Overall Flow

```
/bootstrap-project → /bootstrap-stack → /plan-workitem → /implement-workitem → /validate-workitem → qa/reviewer
```

For details on each step, see:

- [Workflow](docs/00-meta/WORKFLOW.md)
- [Agent Execution Strategy](docs/00-meta/AGENT_EXECUTION_STRATEGY.md)

## Implementation and Validation

After creating a workitem document, start implementation with `/implement-workitem`.
After implementation, verify scope alignment with `/validate-workitem`.
Use `qa` or `reviewer` sub-agents as needed.

```text
/implement-workitem T-001-auth-session
/validate-workitem T-001-auth-session
```

## Structure

```
.
├── CLAUDE.md                          # Shared project instructions
├── .claude/
│   ├── settings.json                  # Shared project settings
│   ├── agents/                        # Role-based sub-agents
│   └── skills/                        # Repeatable task procedures (slash commands)
├── docs/
│   ├── 00-meta/                       # Template usage, workflow, operational principles
│   ├── 10-charter/                    # Project scope, goals, problem definition
│   ├── 20-system/                     # System architecture, design overview
│   ├── 30-workitems/                  # Milestone / feature / task / plans
│   ├── 40-validation/                 # QA results, improvement guides
│   └── 90-decisions/                  # ADR records
└── scripts/                           # Project-specific automation scripts (added after stack is chosen)
```

## Guardrail Principles

This template prioritizes cross-platform reusability.
Shared base settings do not include OS/shell/runtime-dependent hooks or validation scripts.
Automation is added only after the project's actual stack is decided.

For details, see [GUARDRAILS_STRATEGY.md](docs/00-meta/GUARDRAILS_STRATEGY.md).

## Where to Start

- [NEW_PROJECT_CHECKLIST.md](docs/00-meta/NEW_PROJECT_CHECKLIST.md) — New project startup checklist
- [TEMPLATE_GUIDE.md](docs/00-meta/TEMPLATE_GUIDE.md) — Document structure and naming conventions
- [WORKFLOW.md](docs/00-meta/WORKFLOW.md) — Step-by-step workflow

## Input Tips

You can start with a single line of input, but including these four elements improves result quality:

- What you're building
- Who will use it
- What problem it solves
- What's decided and what's still undecided

For more examples, see [BOOTSTRAP_PROMPT_EXAMPLES.md](docs/00-meta/BOOTSTRAP_PROMPT_EXAMPLES.md).

## Contributing

For improvement suggestions or bug reports, see the [issue templates](.github/ISSUE_TEMPLATE). For structural changes, see the [PR template](.github/PULL_REQUEST_TEMPLATE).

## License

MIT