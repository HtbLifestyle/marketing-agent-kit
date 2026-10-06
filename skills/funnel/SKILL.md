---
name: funnel
description: "Create a high-converting website funnel by spawning the marketing-kit agent pipeline: funnel-planner (strategy + page list), user approval gate, funnel-content-creator (structure + publish-ready copy into marketing/funnel/<name>/), then conversion-reviewer + copy-editor in parallel as blocking quality gates with a capped fix loop. Triggers on: /marketing-kit:funnel <product/goal>"
---

You are now the funnel orchestrator. Follow the steps below exactly. You coordinate by spawning agents and reading a shared context file between steps. Do NOT delegate orchestration to another agent — YOU execute these steps directly.

## Setup

Precondition: the project must contain `.claude/docs/BUSINESS_CONTEXT.md` — the business context pack every agent reads (company, audience, brand voice, proof library, data sources). If missing, stop and tell the user to run `/marketing-kit:adopt` first.

Agent-type resolution: the agent names in this workflow are marketing-kit plugin agents, which appear plugin-namespaced in your available-agents list (e.g. `marketing-kit:funnel-planner`). For every spawn, prefer a project-local agent with the plain name if one is available (that is a deliberate per-project override); otherwise use the `marketing-kit:`-namespaced type. Never skip an agent because the plain name is missing.

1. Generate a timestamp: run `date '+%Y-%m-%d-%H-%M-%S'` via Bash
2. Derive a short kebab-case funnel name from the user's task (e.g., "coaching-launch", "saas-trial")
3. Create the shared context file at `.claude/agents-log/<timestamp>-<funnel-name>.md` with this initial content:

```markdown
# Funnel: <funnel name>

## Task
<paste the user's full task description here>

## File References
<list any files, briefs, existing pages or assets the user mentioned>

## Deliverables Directory
marketing/funnel/<funnel-name>/
```

Store the context file path — you will pass it to every agent.

## Workflow

Execute these steps in order. Between each step, read the context file to check results before proceeding.

### Step 1: Funnel Planner

Spawn the planner:
```
Agent(subagent_type: "funnel-planner")
```

Prompt must include:
- The user's task description and all file references
- Instruction: "Read the context file at `<path>`. Run your intake: if core facts (product, price, conversion goal, audience, language/tone) are missing, append them under `## Funnel Planner — Questions` and end with `FUNNEL PLANNER QUESTIONS PENDING`. Otherwise write the full funnel plan under `## Funnel Plan` per your output format — structure table, per-page briefs, Voice & Language, Open Items — and end with `FUNNEL PLAN COMPLETE`."

After it completes, read the context file:
- **Questions pending** → relay every question to the user via AskUserQuestion, append the answers under `## Funnel Planner — Answers`, re-spawn the planner pointing at the answers. Repeat until the plan exists.
- **Plan exists** → verify it contains the Funnel Structure table, per-page briefs and a Voice & Language section. If anything is missing, re-spawn with corrective instruction.

### Step 2: User Gate — Plan Approval

Present the plan summary (structure table + open items) to the user, then ask via AskUserQuestion:
- "go" — approve the plan
- "adjust" — the user types changes; append them under `## Funnel Plan — User Adjustments`, re-spawn the planner to integrate them, then repeat this gate

Do NOT proceed to Step 3 without explicit approval.

### Step 3: Funnel Content Creator

Spawn the content creator:
```
Agent(subagent_type: "funnel-content-creator")
```

Prompt must include:
- Instruction: "Read the context file at `<path>`. Implement `## Funnel Plan` as your contract: write `marketing/funnel/<funnel-name>/README.md` (funnel overview) plus one `NN-<page>.md` per page in the plan's structure table, with complete publish-ready copy per your output format. Log each page under `## Funnel Content — <Page>` in the context file. If you believe the plan is wrong, do NOT restructure silently — append under `## Content Creator Plan Concern` and implement the rest. End with `FUNNEL CONTENT COMPLETE`."

After it completes, read the context file and verify:
- Every page in the plan's structure table has a deliverable file AND a `## Funnel Content — <Page>` entry (list the directory to confirm)
- If `## Content Creator Plan Concern` exists → relay it to the user, get a decision, and if the plan changes re-run the affected pages before reviewing.

### Step 4: Quality Gates (PARALLEL)

Spawn ALL THREE reviewers in parallel (in a single message with three Agent tool calls):

**Conversion Reviewer:**
```
Agent(subagent_type: "conversion-reviewer")
```
Prompt: "Read the context file at `<path>` and review the deliverable files in `marketing/funnel/<funnel-name>/` against `## Funnel Plan` per your checklist. Append your verdict and findings under `## Conversion Review`. End with `CONVERSION REVIEW COMPLETE — PASS` or `— FAIL`."

**Copy Editor:**
```
Agent(subagent_type: "copy-editor")
```
Prompt: "Read the context file at `<path>` and review the deliverable files in `marketing/funnel/<funnel-name>/` against the plan's Voice & Language contract per your checklist. Append your verdict and findings under `## Copy Review`. End with `COPY REVIEW COMPLETE — PASS` or `— FAIL`."

**Headline Reviewer:**
```
Agent(subagent_type: "headline-reviewer")
```
Prompt: "Read the context file at `<path>` and review ONLY the headline layer (headlines, sublines, CTA texts, card titles) of the deliverable files in `marketing/funnel/<funnel-name>/` per your checklist — punchiness, front-loading, memorability, canonical promise/claim lines, tone alignment with the plan's reference sites. Every blocking finding ships with 2–3 concrete alternatives. Append your verdict and findings under `## Headline Review`. End with `HEADLINE REVIEW COMPLETE — PASS` or `— FAIL`."

After ALL THREE complete, read the context file.

### Step 5: Fix Loop (max 2 rounds)

If either gate is FAIL, track the round count (start at 1):

1. Spawn the content creator: "Read the context file at `<path>`. Fix every Blocking Finding in `## Conversion Review`, `## Copy Review` and/or `## Headline Review` in the deliverable files (headline findings come with proposed alternatives — choose or adapt them). Log each finding → change under `## Content Fix — Runde <n>`. End with `CONTENT FIX COMPLETE`."
2. Re-spawn ONLY the reviewer(s) that failed, instructing them to verify each previous finding explicitly and append under `## <Review> — Runde <n+1>`.
3. Both PASS → proceed to Step 6. Any FAIL and round < 2 → repeat with round +1.
4. Round 2 still FAIL → STOP. Summarize both positions under `## Escalation to User` and ask the user how to proceed (accept as-is / one more round / change the plan). Do not loop further on your own.

### Step 6: Final Report

Read the context file one final time and present:

```
## Funnel Complete

### Delegation Report
| # | Agent (subagent_type) | Agent ID | Status | Summary |
|---|----------------------|----------|--------|---------|
| 1 | funnel-planner | <id> | completed | <one line> |
| 2 | funnel-content-creator | <id> | completed | <N pages written> |
| 3 | conversion-reviewer | <id> | completed | <PASS/FAIL + rounds> |
| 4 | copy-editor | <id> | completed | <PASS/FAIL + rounds> |
| 5 | headline-reviewer | <id> | completed | <PASS/FAIL + rounds> |
| ... | ... | ... | ... | ... |

**Deliverables:** `marketing/funnel/<funnel-name>/` — <page list>
**Open VERIFY items:** <list, or none>
**Fix rounds:** <0 | 1 | 2 | escalated>
**Context file:** `<path>`
```

Rules for the report:
- List EVERY agent spawn in chronological order (including re-spawns for questions and fix rounds)
- Agent ID is the ID returned by each Agent tool call — this proves real delegation
- The report MUST contain at minimum: 1x funnel-planner, 1x funnel-content-creator, 1x conversion-reviewer, 1x copy-editor, 1x headline-reviewer

## Rules

- You MUST spawn each agent as a separate Agent tool call with the correct `subagent_type`
- You MUST read the context file between steps to verify progress
- The plan comes BEFORE content, and the user approves it — never skip the Step 2 gate
- conversion-reviewer, copy-editor and headline-reviewer are NEVER optional; all three must end at PASS (or the user explicitly accepts a FAIL via escalation)
- Fix loop is capped at 2 rounds. Escalate to the user after that.
- If any agent asks a question, relay it to the user and wait for the answer
- Never commit anything — the user reviews and commits manually
- Never skip the final report
