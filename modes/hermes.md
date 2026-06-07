# Hermes — Pipeline Orchestrator

Hermes is the top-level coordinator for the entire job search pipeline.
Run `/career-ops hermes` and it drives everything — scan, verify, rank,
shortlist, review, tailor, quality-check, and apply — in one continuous flow.

You make two decisions:
  1. YES / NO / MAYBE on each shortlisted job
  2. "Apply Now" on the preview card before each submission

Everything else is automated.

---

## Full Pipeline Flow

```
/career-ops hermes
       │
       ▼
  ┌─────────────────────────────────────┐
  │  PHASE 1 — DISCOVER                 │
  │  Scan portals → raw job list        │
  └──────────────┬──────────────────────┘
                 │ raw jobs
                 ▼
  ┌─────────────────────────────────────┐
  │  PHASE 2 — VERIFY  (Agent 1)        │
  │  Job Verifier runs per job:         │
  │  · Still live?                      │
  │  · Real company, not a repost?      │
  │  · Location/remote eligible?        │
  │  · Initial fit score                │
  │                                     │
  │  PASS → scored job                  │
  │  SKIP → logged with reason          │
  └──────────────┬──────────────────────┘
                 │ PASS jobs only
                 ▼
  ┌─────────────────────────────────────┐
  │  PHASE 3 — RANK & SHORTLIST         │
  │  Tier 1  4.5+   Fast-lane           │
  │  Tier 2  3.5–4.4 Standard           │
  │  Tier 3  <3.5   Auto-skip           │
  │                                     │
  │  Tier 3 jobs logged and skipped.    │
  │  Tier 1 + 2 added to shortlist.     │
  └──────────────┬──────────────────────┘
                 │ Tier 1 + Tier 2 jobs
                 ▼
  ┌─────────────────────────────────────┐
  │  PHASE 4 — HUMAN REVIEW             │
  │  Show ranked shortlist to user.     │
  │  User marks YES / NO / MAYBE.       │
  │  "Submit" triggers Phase 5.         │
  └──────────────┬──────────────────────┘
                 │ YES jobs only
                 ▼
  ┌─────────────────────────────────────┐
  │  PHASE 5 — TAILOR                   │
  │  Per YES job (one at a time):       │
  │  · Re-verify liveness               │
  │  · Extract full JD                  │
  │  · Tailor CV to JD                  │
  │  · Generate PDF                     │
  │  · Write cover letter               │
  │  · Answer custom form questions     │
  └──────────────┬──────────────────────┘
                 │ tailored documents
                 ▼
  ┌─────────────────────────────────────┐
  │  PHASE 6 — QUALITY GATE  (Agent 2)  │
  │  Resume Reviewer checks:            │
  │  · No invented metrics or roles     │
  │  · ATS keyword coverage             │
  │  · Cover letter is JD-specific      │
  │  · Summary matches role             │
  │                                     │
  │  APPROVED → preview card            │
  │  REVISE   → apply fixes, re-check   │
  └──────────────┬──────────────────────┘
                 │ APPROVED
                 ▼
  ┌─────────────────────────────────────┐
  │  PHASE 7 — PREVIEW & APPLY          │
  │  Show preview card to user.         │
  │  User says "Apply Now".             │
  │  System opens URL, pastes content.  │
  │  User clicks final Submit button.   │
  │  Tracker updated to Applied.        │
  └─────────────────────────────────────┘
```

---

## Phase 1 — Discover

Read `modes/scan.md` and execute a portal scan exactly as that mode describes.
Collect raw job list: title, company, URL, source portal.

If the user says `/career-ops hermes` without scanning first AND
`data/shortlist.md` already has PENDING or YES rows → ask:

```
Shortlist has {N} jobs already pending. Want to:
  1. Continue with existing shortlist (skip scan)
  2. Run a fresh scan first
```

---

## Phase 2 — Verify (Agent 1)

For each raw job from the scan, spawn **Agent 1: Job Verifier**.

```
Agent(
  subagent_type="general-purpose",
  prompt="[content of modes/agents/job-verifier.md]\n\nJob to verify:\nTitle: {title}\nCompany: {company}\nURL: {url}\n\nCandidate profile:\n[content of config/profile.yml]",
  description="Hermes · Agent 1 · Job Verifier · {company}"
)
```

Collect result: `PASS {score} {tier} {reason}` or `SKIP {reason}`.

Process jobs sequentially if Playwright is needed. Batch with WebFetch where Playwright is not required.

Log all SKIPs to `data/hermes-log.md` with reason.

---

## Phase 3 — Rank & Shortlist

After all Agent 1 results are collected:

**Tier assignment:**
| Score | Tier | Action |
|-------|------|--------|
| 4.5–5.0 | Tier 1 | Add to shortlist, flag as fast-lane |
| 3.5–4.4 | Tier 2 | Add to shortlist, standard review |
| < 3.5 | Tier 3 | Auto-skip, log reason, do not show user |

Write all Tier 1 and Tier 2 jobs to `data/shortlist.md` using the standard row format plus a **Tier** column.

Print a discovery summary:
```
Hermes scan complete:
  {N} jobs found · {N} verified · {N} skipped · {N} auto-ranked Tier 3

  Tier 1 (fast-lane, 4.5+):  {N} jobs
  Tier 2 (standard, 3.5–4.4): {N} jobs
```

---

## Phase 4 — Human Review

Show the shortlist grouped by tier. Tier 1 jobs listed first.

For each job, show a card:

```
[T1 · 4.7/5] StackAdapt — Technical PM, AI
~$110K–$130K CAD · ✅ Remote Canada
Why: Canadian AdTech, campaign platform + AI matches X ad-serving background exactly
→ YES / NO / MAYBE
```

Collect decisions. Tier 1 jobs should be presented with a nudge:
`(Tier 1 — strong match, fast-lane apply ready)`

After all decisions, show submit prompt (same as shortlist review mode):
```
{N} jobs approved. Say "submit" to fast-lane all, "submit 001" for one,
or "later" to stop here.
```

---

## Phase 5 — Tailor

For each YES job:
1. Re-verify liveness (Playwright snapshot)
2. Extract full JD (responsibilities, requirements, form questions)
3. Read `cv.md`, `article-digest.md` (if exists), `modes/_profile.md`, `config/profile.yml`, `templates/cv-template.html`
4. Tailor CV — mirror JD language, surface 2–3 most relevant proof points, match ATS keywords exactly. NEVER invent experience or metrics.
5. Write tailored HTML → `output/temp-{num}-{slug}.html`
6. Run `node generate-pdf.mjs output/temp-{num}-{slug}.html output/{num}-{slug}-{date}.pdf`
7. Write cover letter → `output/{num}-{slug}-cover-{date}.md`
8. Draft answers to any custom application questions found on the form

---

## Phase 6 — Quality Gate (Agent 2)

Spawn **Agent 2: Resume Reviewer** with the tailored documents.

```
Agent(
  subagent_type="general-purpose",
  prompt="[content of modes/agents/resume-reviewer.md]\n\nOriginal CV:\n[content of cv.md]\n\nJob Description:\n{jd_text}\n\nTailored CV HTML:\n[content of output/temp-{num}-{slug}.html]\n\nCover letter:\n[content of output/{num}-{slug}-cover-{date}.md]",
  description="Hermes · Agent 2 · Resume Reviewer · {company}"
)
```

**On APPROVED:** Proceed to Phase 7.

**On REVISE:** Agent 2 returns specific notes. Apply fixes inline (do not re-tailor from scratch unless instructed). Re-run Agent 2 once. If still REVISE after second pass → show user the issue and ask how to proceed.

---

## Phase 7 — Preview & Apply

Show preview card:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  [T{tier}] {Company} — {Role}  ·  Score: {score}/5
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  CV:     output/{filename}.pdf  ✅ Agent 2 approved
  Letter: output/{filename}-cover-{date}.md
  URL:    {application url}
  Salary: {target from profile.yml}

  Cover letter opening:
  ┌───────────────────────────────────────────────┐
  │ {first 3 lines}...                            │
  └───────────────────────────────────────────────┘

  {if custom questions:}
  Form Q&As ready: {N} questions answered

  Say "Apply Now" to open the form with everything ready.
  Say "skip" to defer this job.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

On "Apply Now":
- Navigate browser to application URL
- Provide all content ready to paste (cover letter, Q&A answers, salary)
- Guide through form fields if needed
- **STOP before final Submit button — user clicks it**
- After user confirms submitted: update shortlist to `✅ Applied`, write tracker TSV

On "skip": mark `⏭ SKIPPED`, move to next job.

---

## Hermes Log

Maintain `data/hermes-log.md` across sessions:

```markdown
# Hermes Log

## {YYYY-MM-DD} Session

### Scanned
- {N} portals · {N} raw jobs found

### Agent 1 — Verified
| Job | Result | Reason |
|-----|--------|--------|
| StackAdapt TPM | PASS T1 4.7 | Strong AdTech + Canada match |
| Reddit Staff PM | SKIP | US-only, no Canada eligibility |

### Agent 2 — Quality
| Job | Result | Notes |
|-----|--------|-------|
| StackAdapt TPM | APPROVED | — |

### Applied
| Job | Status |
|-----|--------|
| StackAdapt TPM | ✅ Submitted |
```

---

## Hermes Commands

| Command | Behaviour |
|---------|-----------|
| `/career-ops hermes` | Run full pipeline from scan |
| `/career-ops hermes shortlist` | Skip scan, go from Phase 4 with existing shortlist |
| `/career-ops hermes apply` | Skip to Phase 5 for existing YES jobs |
| `/career-ops hermes status` | Print Hermes log summary for current session |
| `/career-ops hermes reset` | Clear hermes-log.md and reset pipeline state |
