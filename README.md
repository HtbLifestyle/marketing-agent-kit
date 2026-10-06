# marketing-agent-kit

A [Claude Code plugin](https://code.claude.com/docs/en/plugins) providing a multi-agent pipeline for high-converting marketing content — starting with website funnels. One shared, versioned agent setup with per-project agent memory. Sibling of [mobile-agent-kit](https://github.com/ChristianEichmueller/mobile-agent-kit), same conventions.

## What's inside

**5 agents** (`agents/`):

| Agent | Role |
|---|---|
| `funnel-planner` | Audience/offer intake + funnel strategy: stages, page list, per-page job, core message and CTA — the plan is the contract |
| `funnel-content-creator` | Implements the plan: page structure + complete publish-ready copy into `marketing/funnel/<name>/`; raises plan concerns instead of deviating |
| `conversion-reviewer` | Blocking CRO quality gate: hook, value argumentation, proof, objection coverage, CTA clarity, friction |
| `copy-editor` | Blocking language quality gate: tone consistency, terminology, clarity, grammar, readability |
| `headline-reviewer` | Blocking headline-layer gate: punchiness, front-loading, memorability, canonical promise lines, reference-tone alignment — always proposes stronger alternatives |

**2 skills** (`skills/`):

| Command | Workflow |
|---|---|
| `/marketing-kit:adopt` | Onboard a company: gather data sources, analyze website + marketing material, interview for the gaps, generate the business context pack (`.claude/docs/BUSINESS_CONTEXT.md`), scaffold agent memory |
| `/marketing-kit:funnel <product/goal>` | plan → user approval gate → content for every page → conversion + copy + headline review in parallel → capped fix loop (max 2 rounds, then escalate) → delegation report |

Key rules across the workflow: **the plan is written and user-approved before any content exists**; the three quality gates are never optional and must all end at PASS; product facts and proof are never invented — they trace to the business context pack's Proof Library, or ship as `<!-- VERIFY: ... -->` items the user confirms.

## Install

```
/plugin marketplace add ChristianEichmueller/marketing-agent-kit
/plugin install marketing-kit@marketing-agent-kit
/marketing-kit:adopt
```

`adopt` gathers the company's data sources (website, marketing material, customer feedback), analyzes them, interviews you for what the data can't answer, and generates `.claude/docs/BUSINESS_CONTEXT.md` — the context pack all agents read at runtime. Review and commit the generated files. Deliverables of funnel runs land in `marketing/funnel/<funnel-name>/` inside the project.

## How adaptation works (3 layers)

1. **Shared (this plugin):** agent roles, the plan-first discipline, the review gates. Identical everywhere, updated centrally.
2. **Per company (generated, committed):** `.claude/docs/BUSINESS_CONTEXT.md` — identity, audience, brand voice, proof library, permitted data sources, legal constraints. Agents refuse to invent anything this pack should answer.
3. **Per project (runtime):** every agent has `memory: project` — audience insights, brand voice, style rules and recurring review findings accumulate in `.claude/agent-memory/marketing-kit-<agent>/` inside each project and never leave it. Each workflow run additionally logs to `.claude/agents-log/<timestamp>-<funnel-name>.md` — every agent appends under its own heading, so the run is fully auditable.

## Updating

Improve agents/skills here, push, then on each consuming machine:

```
/plugin marketplace update marketing-agent-kit
claude plugin update marketing-kit
```

## Overriding per project

Project-level definitions always win. To customize one agent for one project, copy it to `<project>/.claude/agents/<name>.md` and edit — the plugin version is ignored for that project while the rest stay shared. (A local override uses the unprefixed memory dir `.claude/agent-memory/<name>/`.)

## Adding agents

New agents are added via the `/add-agent` skill, which analyzes the pipeline, writes the agent to these conventions and wires it into the workflow — an agent no workflow spawns is not integrated.

## Known issues

Claude Code (observed on 2.1.195) injects a broken Persistent Agent Memory path for plugin agents — a doubled `.claude/.claude` segment — so automatic MEMORY.md loading silently fails. The agents in this kit work around it by self-managing memory: they read/write `.claude/agent-memory/marketing-kit-<agent>/` explicitly and ignore the injected path. No action needed; this note can be dropped once the upstream bug is fixed.

## License

MIT
