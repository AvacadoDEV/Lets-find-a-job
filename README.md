# Lets Find a Job 🎯

> An AI-powered job search pipeline built on [Claude Code](https://claude.ai/code) — structured to find, evaluate, shortlist, and apply to roles with precision, not volume.

**Forked from and built on [career-ops](https://github.com/santifer/career-ops) by [Santiago Ferrer (santifer)](https://santifer.io).**

---

## What This Is

A job search system you configure once for your role, location, and salary target — then let run. Fork it, point it at your industry, and it handles the grunt work.

The recommended entry point is a single command:

```
/career-ops hermes
```

Hermes orchestrates the entire pipeline end to end. You make two decisions:
1. **YES / NO / MAYBE** on each shortlisted job
2. **"Apply Now"** on the preview card before each submission

Everything else — scanning, verification, ranking, tailoring, quality-checking — is automated.

---

## Architecture

```
┌─────────────────────────────────────────────────┐
│                    HERMES                        │
│             Orchestrator Agent                   │
│  Coordinates all phases, applies threshold       │
│  routing, maintains session log                  │
└────┬──────────────────┬──────────────────────────┘
     │                  │
     ▼                  ▼
┌──────────┐    ┌────────────────────┐
│ Agent 1  │    │     Agent 2        │
│  Job     │    │  Resume Reviewer   │
│Verifier  │    │                    │
│          │    │ Independent review │
│· Live?   │    │ of tailored docs   │
│· Real?   │    │ before preview:    │
│· Eligible│    │ · No invented facts│
│· Fit score    │ · ATS coverage     │
│          │    │ · Cover letter JD- │
│PASS/SKIP │    │   specific?        │
└──────────┘    │ APPROVED/REVISE    │
                └────────────────────┘
```

### Full pipeline flow

```
/career-ops hermes
       │
       ▼
Phase 1 — DISCOVER
  Scan 30+ portals → raw job list

       │
       ▼
Phase 2 — VERIFY  [Agent 1: Job Verifier]
  Per job: liveness check · authenticity · location eligibility · fit score
  PASS → scored job    SKIP → logged with reason

       │ PASS jobs only
       ▼
Phase 3 — RANK
  Tier 1  (4.5+)    → fast-lane
  Tier 2  (3.5–4.4) → standard review
  Tier 3  (<3.5)    → auto-skip, logged

       │ Tier 1 + 2 only
       ▼
Phase 4 — HUMAN REVIEW  ← your first decision
  Ranked shortlist presented, Tier 1 first.
  You mark YES / NO / MAYBE per job.
  Say "submit" to proceed.

       │ YES jobs only
       ▼
Phase 5 — TAILOR
  Per YES job: re-verify liveness · extract JD · tailor CV ·
  generate PDF · write cover letter · answer form questions

       │
       ▼
Phase 6 — QUALITY GATE  [Agent 2: Resume Reviewer]
  Checks tailored docs against original CV:
  · No invented metrics or experience
  · ATS keyword coverage
  · Cover letter is JD-specific, not templated
  APPROVED → preview card    REVISE → fix + re-check

       │ APPROVED
       ▼
Phase 7 — PREVIEW & APPLY  ← your second decision
  Preview card: CV path · cover letter snippet · salary · form Q&As
  Say "Apply Now" → system opens URL, pastes content
  You click the final Submit button yourself.
  Tracker updated to Applied.
```

---

## Tier Routing

| Score | Tier | Flow |
|-------|------|------|
| 4.5–5.0 | Tier 1 — Fast-lane | Preview card → "Apply Now" (one word) |
| 3.5–4.4 | Tier 2 — Standard | Preview card → confirm |
| < 3.5 | Tier 3 — Skip | Auto-skipped, reason logged to `data/hermes-log.md` |

You always see the preview card before anything is submitted. The tier only affects how much friction is in the flow — not whether you're in control.

---

## Credit & Foundation

**The scanning, evaluation, CV generation, and portal infrastructure is built entirely on [career-ops](https://github.com/santifer/career-ops) by [Santiago Ferrer (santifer)](https://santifer.io).**

Santiago built career-ops to solve his own job search — and it worked:
- Evaluated **740+ job offers**
- Generated **100+ tailored CVs**
- Landed a **Head of Applied AI** role

The multi-language support (German/French), scoring framework, negotiation scripts, and ethical guardrails are all from the original.

**New additions in this repo:**
- `modes/hermes.md` — Orchestrator agent coordinating the full pipeline
- `modes/agents/job-verifier.md` — Agent 1: liveness, authenticity, eligibility, fit scoring
- `modes/agents/resume-reviewer.md` — Agent 2: independent CV/cover letter quality gate
- `modes/shortlist.md` — Shortlist management with YES/NO/MAYBE decisions
- `modes/apply-batch.md` — Fast-lane apply with preview card

> ⭐ Star the original: **[github.com/santifer/career-ops](https://github.com/santifer/career-ops)**

---

## Stack

| Tool | Role |
|------|------|
| [Claude Code](https://claude.ai/code) | Hermes orchestrator + all agents |
| Node.js + Playwright | PDF generation, browser automation, liveness checks |
| Markdown + YAML | Data storage and configuration |
| HTML/CSS | CV template and design |

---

## Setup

### Prerequisites

- [Claude Code](https://claude.ai/code) (Claude subscription required)
- Node.js 18+
- Git

### Install

```bash
git clone https://github.com/AvacadoDEV/Lets-find-a-job.git
cd Lets-find-a-job
npm install
npx playwright install chromium
npm run doctor
```

### Configure

1. **CV** — Create `cv.md` with your experience in markdown (see `examples/cv-example.md`)
2. **Profile** — Copy `config/profile.example.yml` → `config/profile.yml` and fill in your details
3. **Narrative** — Copy `modes/_profile.template.md` → `modes/_profile.md` and customise your archetypes, superpowers, and story bank
4. **Portals** — Copy `templates/portals.example.yml` → `portals.yml` and adjust companies and search keywords for your target roles

> **Note:** `cv.md`, `config/profile.yml`, `modes/_profile.md`, `portals.yml`, and all job data are gitignored — your personal information stays local.

### Run the doctor check

```bash
npm run doctor
```

All checks should pass before you start.

---

## Commands

Open Claude Code in this directory (`claude`), then:

### Hermes (recommended)

| Command | What it does |
|---------|--------------|
| `/career-ops hermes` | Full pipeline: scan → verify → rank → shortlist → tailor → review → apply |
| `/career-ops hermes shortlist` | Skip scan, start from existing shortlist |
| `/career-ops hermes apply` | Skip to apply phase for existing YES jobs |
| `/career-ops hermes status` | Print current session log summary |

### Manual controls

| Command | What it does |
|---------|--------------|
| `/career-ops scan` | Scan portals, add raw jobs to pipeline |
| `/career-ops shortlist review` | Review pending jobs, mark YES / NO / MAYBE |
| `/career-ops shortlist add <url>` | Manually score and add a specific job |
| `/career-ops apply-batch` | Fast-lane apply for all YES jobs |
| `/career-ops tracker` | Application status overview |
| `/career-ops deep` | Deep company research |
| `/career-ops contacto` | Find LinkedIn contacts + draft outreach |
| `/career-ops <JD url or text>` | Single-job auto-pipeline |

---

## File Structure

```
├── cv.md                            # Your canonical CV (gitignored)
├── config/
│   ├── profile.yml                  # Identity, targets, comp (gitignored)
│   └── profile.example.yml          # Template
├── modes/
│   ├── hermes.md                    # ★ Orchestrator (new)
│   ├── agents/
│   │   ├── job-verifier.md          # ★ Agent 1: Job Verifier (new)
│   │   └── resume-reviewer.md       # ★ Agent 2: Resume Reviewer (new)
│   ├── shortlist.md                 # ★ Shortlist mode (new)
│   ├── apply-batch.md               # ★ Apply-batch fast-lane (new)
│   ├── _profile.md                  # Your archetypes + story bank (gitignored)
│   ├── _profile.template.md         # Template
│   ├── _shared.md                   # Scoring logic, global rules
│   └── ...                          # Other career-ops modes
├── data/
│   ├── hermes-log.md                # Session log: scans, verdicts, applications
│   ├── shortlist.md                 # Job shortlist (gitignored)
│   ├── applications.md              # Tracker (gitignored)
│   └── pipeline.md                  # URL inbox (gitignored)
├── portals.yml                      # Portal config (gitignored)
├── templates/
│   ├── cv-template.html             # CV design
│   └── portals.example.yml          # Portals template
├── output/                          # Generated PDFs (gitignored)
└── reports/                         # Evaluation reports (gitignored)
```

---

## Design Principles

- **Human in the loop** — you make exactly two decisions per application: YES on the shortlist, Apply Now on the preview
- **Independent quality gate** — Agent 2 reviews tailored documents it didn't produce, catching what a self-reviewing agent would miss
- **Rank-based routing** — high-confidence matches get less friction, not zero oversight
- **No invented content** — Agent 2 hard-blocks any CV that contains metrics or experience not in your original
- **No auto-submit** — the system opens the form and pastes content; you click Submit
- **Data stays local** — CV, profile, and job data are gitignored and never committed

---

## Acknowledgements

This project is a fork of and extension to **[career-ops](https://github.com/santifer/career-ops)** by **[Santiago Ferrer](https://santifer.io)**. The portal scanner, evaluation engine, CV generation pipeline, application tracker, batch processing, and multi-language support are all from Santiago's original work.

His portfolio is also open source: **[cv-santiago](https://github.com/santifer/cv-santiago)**.

---

*Built with [Claude Code](https://claude.ai/code) · Based on [career-ops](https://github.com/santifer/career-ops) by [santifer](https://santifer.io)*
