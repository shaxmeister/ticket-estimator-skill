# Ticket Estimator: a Claude skill for story-point estimation

A [Claude Skill](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) that estimates Jira or Linear tickets on a **1/2/3/5/8** scale and explains why, so sprint planning starts from a consistent baseline instead of a blank stare.

## Why I built it

My team spent a lot of time in refinement debating points, and estimates varied a lot depending on who was in the room. I turned the team's informal rules ("easy change + hard testing = 3", "a 3 that depends on another team is a 5") into a rubric Claude applies the same way every time. I then calibrated it against ~10 past tickets with agreed scores.

The version in this repo uses **made-up calibration examples**, because the originals were internal tickets.

## How it works

1. Paste a ticket URL or its text.
2. Claude scores four dimensions on the same 1/2/3/5/8 scale:

   | Dimension | Weight |
   |---|---|
   | Scope of code changes | 40% |
   | Uncertainty / unknowns | 25% |
   | Testing / QA effort | 20% |
   | Cross-team dependencies | 15% |

3. It calculates a weighted score, maps it to points, then applies team rules (for example, "investigation needed = at least 5").
4. It returns the estimate, a breakdown table, the closest calibration example, risks, and ideas to shrink the ticket. Any 8 comes with a suggested way to split it.

## Install

```
skills/
└── ticket-estimator/
    └── SKILL.md
```

- **Claude app:** zip the `ticket-estimator` folder and upload it in Claude's Skills settings.
- **Claude Code:** copy the folder into `~/.claude/skills/` (personal) or `.claude/skills/` in your repo (shared with the team).

The file must be named `SKILL.md`. The folder name can be anything.

## Make it yours

The value is in calibration. Replace the examples in `SKILL.md` with 2 past tickets per point value that your team agreed on, and adjust the weights if, for example, testing is your bottleneck.

## Example output

> **Ticket:** Filter recommendations by product attributes from tenant config
> ## Estimate: 3 points
> | Dimension | Score | Rationale |
> |---|---|---|
> | Scope | 3 | New config read plus a filter in one service. |
> | Uncertainty | 3 | Unclear how the filter interacts with existing ranking logic. |
> | Testing | 3 | Needs before/after API comparison and regression on other tenants. |
> | Dependencies | 1 | None. |
>
> **Weighted score:** 2.70 → **3**
> **Closest example:** 3a
