---
name: career-ops
description: AI job search command center -- evaluate offers, generate CVs, scan portals, track applications
user_invocable: true
args: mode
---

# career-ops -- Router

## Mode Routing

Determine the mode from `{{mode}}`:

| Input | Mode |
|-------|------|
| (empty / no args) | `discovery` -- Show command menu |
| JD text or URL (no sub-command) | **`auto-pipeline`** |
| `oferta` | `oferta` |
| `ofertas` | `ofertas` |
| `contacto` | `contacto` |
| `deep` | `deep` |
| `pdf` | `pdf` |
| `training` | `training` |
| `project` | `project` |
| `tracker` | `tracker` |
| `pipeline` | `pipeline` |
| `apply` | `apply` |
| `apply-batch` | `apply-batch` |
| `shortlist` | `shortlist` |
| `shortlist add` | `shortlist` (sub: add) |
| `shortlist review` | `shortlist` (sub: review) |
| `shortlist status` | `shortlist` (sub: status) |
| `scan` | `scan` |
| `batch` | `batch` |

**Auto-pipeline detection:** If `{{mode}}` is not a known sub-command AND contains JD text (keywords: "responsibilities", "requirements", "qualifications", "about the role", "we're looking for", company name + role) or a URL to a JD, execute `auto-pipeline`.

If `{{mode}}` is not a sub-command AND doesn't look like a JD, show discovery.

---

## Discovery Mode (no arguments)

Show this menu:

```
career-ops -- Command Center

PIPELINE (recommended flow):
  1. /career-ops scan          → Discover new jobs from portals → added to shortlist
  2. /career-ops shortlist     → Review jobs, mark YES / NO / MAYBE
  3. /career-ops apply-batch   → Generate tailored CV + cover letter for YES jobs
     (You review documents, then apply yourself)

SINGLE JOB:
  /career-ops {JD or URL}      → AUTO-PIPELINE: evaluate + report + PDF + tracker
  /career-ops oferta           → Evaluation only A-F (no auto PDF)
  /career-ops pdf              → Generate tailored CV PDF for a specific job

RESEARCH & OUTREACH:
  /career-ops ofertas          → Compare and rank multiple offers
  /career-ops contacto         → LinkedIn: find contacts + draft message
  /career-ops deep             → Deep company research

TRACKING:
  /career-ops tracker          → Application status overview
  /career-ops shortlist status → Count: pending / YES / NO / MAYBE

ADVANCED:
  /career-ops pipeline         → Process pending URLs from inbox (data/pipeline.md)
  /career-ops apply            → Live application assistant (reads form + drafts answers)
  /career-ops training         → Evaluate course/cert against North Star
  /career-ops project          → Evaluate portfolio project idea
  /career-ops batch            → Batch processing with parallel workers

Inbox: add URLs to data/pipeline.md → /career-ops pipeline
Or paste a JD directly to run the full pipeline.
```

---

## Context Loading by Mode

After determining the mode, load the necessary files before executing:

### Modes that require `_shared.md` + their mode file:
Read `modes/_shared.md` + `modes/{mode}.md`

Applies to: `auto-pipeline`, `oferta`, `ofertas`, `pdf`, `contacto`, `apply`, `apply-batch`, `shortlist`, `pipeline`, `scan`, `batch`

### Standalone modes (only their mode file):
Read `modes/{mode}.md`

Applies to: `tracker`, `deep`, `training`, `project`

### Modes delegated to subagent:
For `scan`, `apply` (with Playwright), `apply-batch` (with Playwright), and `pipeline` (3+ URLs): launch as Agent with the content of `_shared.md` + `modes/{mode}.md` injected into the subagent prompt.

```
Agent(
  subagent_type="general-purpose",
  prompt="[content of modes/_shared.md]\n\n[content of modes/{mode}.md]\n\n[invocation-specific data]",
  description="career-ops {mode}"
)
```

Execute the instructions from the loaded mode file.
