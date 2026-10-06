---
name: adopt
description: "Onboard a company/project to the marketing-kit agents: gather the business data sources, analyze website and marketing material, interview the user for what data can't answer, and generate the business context pack (.claude/docs/BUSINESS_CONTEXT.md) all agents read. Re-run anytime to refresh. Triggers on: /marketing-kit:adopt"
---

You are onboarding this project to the marketing-kit agent suite. The agents are generic; this skill generates the **business context pack** that adapts them to this specific company — its products, audience, brand voice, proof and data. Work directly — do not delegate the writing, but you may spawn Explore agents for analyzing material.

## Step 0: Detect mode

- If `.claude/docs/BUSINESS_CONTEXT.md` does not exist → **fresh adoption** (all steps).
- If it exists → **re-adoption**: re-verify every section against the current data sources, update stale facts. Compare its `CONTRACT_VERSION` with the one in the plugin's `docs/BUSINESS_CONTEXT_TEMPLATE.md`; if the template is newer, add any new required sections. Preserve hand-written content — refresh facts, don't rewrite prose. Skip Steps 4–5 unless missing.

## Step 1: Gather the data sources

Use the AskUserQuestion tool to ask where the company's data lives (multiSelect where sensible):
- Website URL(s)
- Folders with marketing material, brand guides, testimonials, case studies (offer any plausible directories you find in the project)
- Customer feedback sources (reviews, survey exports, support logs)
- Connected tools that may be read (e.g. Google Drive, Gmail, Slack via MCP) — only ones the user names

Record every confirmed source; agents will later read ONLY listed sources.

## Step 2: Analyze the data

Spawn 1–2 Explore agents (or read directly for small material sets) over the confirmed sources; fetch the website with WebFetch. Extract with concrete quotes/paths as evidence:

- Products/offers, pricing, guarantees as actually published
- The audience the material speaks to; recurring pains, desires, objections in customer feedback
- The de-facto brand voice: language, register (Du/Sie), tone, house terminology
- Proof inventory: every testimonial, case study, number, award — with its source location
- Existing funnels/pages and channels
- Legal footers, disclaimers, regulated-claim patterns

## Step 3: Interview for the gaps

Compare findings against the template's REQUIRED sections. Ask the user (AskUserQuestion, batched, most critical first) ONLY what the data could not answer — e.g. positioning intent, target segment priorities, tone aspiration vs. current reality, claims that must not be made. Do not re-ask what Step 2 already established; present those as facts to confirm.

## Step 4: Generate the context pack

Create `.claude/docs/` containing:

1. **`BUSINESS_CONTEXT.md`** — fill in the plugin's `docs/BUSINESS_CONTEXT_TEMPLATE.md` (find it in the installed plugin directory; if unavailable, reconstruct its section list: Identity, Audience, Brand Voice, Proof Library, Data Sources, Channels & Traffic, Competitors & Differentiation, Legal Constraints, Reference Documents — with `CONTRACT_VERSION: 1`). Every REQUIRED section must be filled with verified facts from Steps 2–3. Link to existing brand/style docs instead of duplicating them.
2. **`PROOF_LIBRARY.md`** — only if the proof inventory is too large for the table in BUSINESS_CONTEXT.md: the full library, referenced from the main file.

Everything you write must be traceable to a source or a user answer. Mark anything uncertain with `<!-- VERIFY: ... -->` and list those items in your final report.

## Step 5: Scaffold agent memory

Create empty directories (no files — memory grows organically):

```
.claude/agent-memory/{marketing-kit-funnel-planner,marketing-kit-funnel-content-creator,marketing-kit-conversion-reviewer,marketing-kit-copy-editor,marketing-kit-headline-reviewer}/
```

If the directories already exist, leave them completely untouched.

## Step 6: Register the marketplace in project settings

Merge into `.claude/settings.json` (create if missing, preserve existing keys):

```json
{
  "extraKnownMarketplaces": {
    "marketing-agent-kit": {
      "source": { "source": "github", "repo": "ChristianEichmueller/marketing-agent-kit" }
    }
  }
}
```

## Step 7: Report

Present to the user:
- The confirmed data sources and what each contributed
- Files created/updated (context pack, settings)
- Every `<!-- VERIFY: ... -->` item that needs their confirmation
- Any REQUIRED section you could not fill (e.g. no proof material exists yet) — with a recommendation
- Reminder: `/marketing-kit:funnel <product/goal>` is now ready to run

Do NOT commit anything — the user reviews and commits.
