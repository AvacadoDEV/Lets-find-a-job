# Lets Find a Job 🎯

> An AI-powered job search pipeline built on [Claude Code](https://claude.ai/code) — structured to find, evaluate, shortlist, and apply to roles with precision, not volume.

**Forked from and built on [career-ops](https://github.com/santifer/career-ops) by [Santiago Ferrer (santifer)](https://santifer.io).**

---

## What This Is

A job search system you configure once for your role, location, and salary target — then let run. Fork it, point it at your industry, and it handles the grunt work. It's a 3-layer pipeline:

```
Layer 1 — Discover    /career-ops scan
           Searches 30+ company portals and job boards for matching roles.
           Filters by title, location, and salary range automatically.

Layer 2 — Shortlist   /career-ops shortlist review
           Every found job is scored and summarized. You mark each one
           YES / NO / MAYBE. Nothing moves forward without a YES.

Layer 3 — Apply       submit → /career-ops apply-batch
           Mark YES and hit submit. The system instantly rechecks the
           posting, tailors your CV to the JD, generates a PDF, and
           writes a cover letter. A preview card appears — glance at
           it, say "Apply Now", and you're done. One confirmation,
           not a manual process.
```

**The goal is quality, not volume.** A well-targeted application to 5 companies beats a generic blast to 50.

---

## Credit & Foundation

**Layer 1 is built entirely on [career-ops](https://github.com/santifer/career-ops) by [Santiago Ferrer (santifer)](https://santifer.io).**

Career-ops is an open-source AI job search pipeline that Santiago built and used personally to:
- Evaluate **740+ job offers**
- Generate **100+ tailored CVs**
- Land a **Head of Applied AI** role

It provides the evaluation engine, CV generation, portal scanning, batch processing, and application tracking that powers the foundation of this system. The multi-language support (German/French), the scoring framework, negotiation scripts, and the ethical guardrails are all from the original.

**Layers 2 and 3** — the shortlist approval gate (`modes/shortlist.md`) and the apply-batch automation (`modes/apply-batch.md`) — are original additions built on top of that foundation.

> ⭐ If you find this useful, go star the original: **[github.com/santifer/career-ops](https://github.com/santifer/career-ops)**

---

## Stack

| Tool | Role |
|------|------|
| [Claude Code](https://claude.ai/code) | AI agent driving the entire pipeline |
| Node.js + Playwright | PDF generation, browser automation |
| Markdown + YAML | Data storage and configuration |
| HTML/CSS | CV template and design |

---

## How It Works

### Layer 1 — Discovery *(from career-ops by santifer)*

Scans 30+ job portals and company career pages configured in `portals.yml`. Each scan:
- Checks company career pages directly via Playwright
- Runs targeted search queries on Greenhouse, Ashby, Lever, and Wellfound
- Filters results by title keywords and location eligibility
- Deduplicates against scan history so you never see the same listing twice

### Layer 2 — Shortlist *(new)*

Every job found by the scanner is added to `data/shortlist.md` with:
- A fit score (1–5) based on skills, archetype match, salary, and remote eligibility
- Salary estimate (from the JD or researched via web search)
- A one-line "why it fits" tied to a real proof point from your CV
- A ⚠️ flag if the role is US-only or has other eligibility concerns

You review each card and mark **YES / NO / MAYBE**. Only YES rows move to Layer 3. Nothing is generated, sent, or filed without your explicit approval.

### Layer 3 — Apply *(new)*

Mark a job YES and say "submit". Within ~30 seconds:

1. **Rechecks** the posting is still live (Playwright browser snapshot) — closed roles are skipped automatically
2. **Tailors** your CV — reorders bullets to mirror JD language, surfaces the 2–3 most relevant proof points, matches ATS keywords exactly without inventing anything
3. **Generates** a PDF from the tailored HTML via Playwright
4. **Writes** a cover letter (250 words max, JD-specific, no generic filler)
5. **Answers** any custom application questions found on the form
6. **Shows a preview card** — cover letter snippet, PDF path, salary field, custom Q&As
7. **Waits for "Apply Now"** — you glance at the preview and confirm with one word

The system opens the application URL with everything ready to paste. You click the final Submit button yourself — one deliberate action, not a hands-off blast.

---

## Setup

### Prerequisites

- [Claude Code](https://claude.ai/code) (Claude subscription required)
- Node.js 18+
- Git

### Install

```bash
git clone https://github.com/anup1/lets-find-a-job.git
cd lets-find-a-job
npm install
npx playwright install chromium
npm run doctor
```

### Configure

1. **CV** — Create `cv.md` with your experience in markdown (see `examples/cv-example.md`)
2. **Profile** — Copy `config/profile.example.yml` → `config/profile.yml` and fill in your details
3. **Narrative** — Copy `modes/_profile.template.md` → `modes/_profile.md` and customize your archetypes, superpowers, and story bank
4. **Portals** — Copy `templates/portals.example.yml` → `portals.yml` and adjust companies and keywords for your target roles

> **Note:** `cv.md`, `config/profile.yml`, `modes/_profile.md`, `portals.yml`, and all job data are gitignored — your personal information stays local.

### Run the doctor check

```bash
npm run doctor
```

All checks should pass before you start.

---

## Commands

Open Claude Code in this directory (`claude`), then:

| Command | What it does |
|---------|--------------|
| `/career-ops scan` | Search portals for new roles → adds to shortlist |
| `/career-ops shortlist review` | Review pending jobs, mark YES / NO / MAYBE |
| `/career-ops shortlist add <url>` | Manually add and score a specific job URL |
| `/career-ops shortlist status` | Count: pending / YES / NO / MAYBE |
| `/career-ops apply-batch` | Generate tailored CV + cover letter for all YES jobs |
| `/career-ops apply-batch job 003` | Generate documents for one specific job |
| `/career-ops <JD url or text>` | Full auto-pipeline for a single job |
| `/career-ops oferta` | Evaluate a role (A–F) without generating a PDF |
| `/career-ops tracker` | Application status overview |
| `/career-ops deep` | Deep company research before an interview |
| `/career-ops contacto` | Find LinkedIn contacts + draft outreach message |

---

## File Structure

```
├── cv.md                        # Your canonical CV (gitignored — stays local)
├── config/
│   ├── profile.yml              # Your identity, targets, comp (gitignored)
│   └── profile.example.yml      # Template to copy from
├── modes/
│   ├── _profile.md              # Your archetypes + story bank (gitignored)
│   ├── _profile.template.md     # Template to copy from
│   ├── _shared.md               # Scoring logic, global rules
│   ├── shortlist.md             # ★ Shortlist mode (Layer 2, new)
│   ├── apply-batch.md           # ★ Apply-batch mode (Layer 3, new)
│   └── ...                      # Other career-ops modes (from santifer)
├── data/
│   ├── shortlist.md             # Your job shortlist (gitignored)
│   ├── applications.md          # Application tracker (gitignored)
│   └── pipeline.md              # URL inbox (gitignored)
├── portals.yml                  # Company portal config (gitignored)
├── templates/
│   ├── cv-template.html         # CV HTML/CSS design
│   └── portals.example.yml      # Portals template
├── output/                      # Generated PDFs (gitignored)
└── reports/                     # Evaluation reports (gitignored)
```

---

## Design Principles

- **Human in the loop** — every decision point requires explicit approval
- **Quality over quantity** — roles scoring below 3.5/5 are flagged; below 3.0 are not recommended
- **No auto-submit** — the system generates, you send
- **Respect everyone's time** — a recruiter's attention is valuable; only send what's worth reading
- **Data stays local** — your CV, profile, and job data are gitignored and never committed

---

## Acknowledgements

This project is a fork of and extension to **[career-ops](https://github.com/santifer/career-ops)** by **[Santiago Ferrer](https://santifer.io)**. The portal scanner, evaluation engine, CV generation pipeline, application tracker, batch processing, and multi-language support are all from Santiago's original work.

Santiago built career-ops to solve his own job search — and it worked. His portfolio is also open source: **[cv-santiago](https://github.com/santifer/cv-santiago)**.

The shortlist approval layer (Layer 2) and apply-batch automation (Layer 3) in this repo are new additions contributed back to the community.

---

*Built with [Claude Code](https://claude.ai/code) · Based on [career-ops](https://github.com/santifer/career-ops) by [santifer](https://santifer.io)*
