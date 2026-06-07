# Mode: apply-batch

## Purpose

Fast-lane application flow. When a job is marked `✅ YES` and the user triggers apply-batch,
the system immediately:
1. Rechecks the posting is still live
2. Tailors the CV to the JD
3. Generates a PDF
4. Writes a cover letter
5. Answers any custom application questions
6. Presents a **preview card** for a quick glance
7. Waits for a single **"Apply Now"** confirmation before opening the application URL

**The user sees everything before it goes out. One final confirmation, then done.**

---

## Trigger

- User marks a job YES in the shortlist and says "submit", "apply", or "go"
- User says "apply-batch" to process all YES jobs in sequence
- User says "apply for job 003" to fast-lane a specific job

---

## Flow

```
YES + Submit
     ↓
[~30 seconds]
Liveness check → Extract JD → Tailor CV → Generate PDF → Write cover letter
     ↓
Preview card shown to user
     ↓
User says "Apply Now" / "looks good" / "send it"
     ↓
System opens application URL and provides all documents ready to paste
     ↓
Tracker updated to Applied
```

---

## Steps

### 0. Guard check
Read `data/shortlist.md`. Find rows where Decision = `✅ YES` and status is not already `Applied`.
If none: "No approved jobs ready. Review your shortlist first with `/career-ops shortlist review`."

### 1. For each YES job (one at a time — never parallel Playwright)

#### 1a. Recheck liveness
- `browser_navigate` to the job URL + `browser_snapshot`
- Only footer/navbar with no JD content → mark `❌ CLOSED` in shortlist, skip, notify user
- Active JD found → continue immediately

#### 1b. Extract full JD
Title, company, responsibilities, requirements, nice-to-haves, and any visible application form questions.

#### 1c. Read candidate files
- `cv.md` — canonical CV (NEVER modify this file)
- `article-digest.md` — proof points (if exists)
- `modes/_profile.md` — archetypes, superpowers, story bank
- `config/profile.yml` — identity, comp targets
- `templates/cv-template.html` — design template

#### 1d. Tailor the CV
- NEVER invent experience or metrics — only reframe what exists in cv.md
- Lead Professional Summary with the exact role title and company name
- Surface the 2–3 proof points most relevant to this JD
- Mirror JD keyword language exactly for ATS (not synonyms)
- 1 page preferred; 2 pages max
- Skills section last — lead with impact

#### 1e. Generate PDF
Write tailored HTML to `output/temp-{num}-{slug}.html`
Run: `node generate-pdf.mjs output/temp-{num}-{slug}.html output/{num}-{slug}-{date}.pdf`
Confirm file exists before continuing.

#### 1f. Write cover letter
File: `output/{num}-{slug}-cover-{date}.md`

Structure:
```
{Hiring Team / Hiring Manager name if found in JD},

[Para 1 — 2 sentences] Why this role, why this company. One specific detail
from the JD or company — not generic.

[Para 2 — 3 sentences] Strongest proof point for THIS role. Quote a JD
requirement, map it to a real result from the CV with a number.

[Para 3 — 2 sentences] The differentiator — what the candidate brings that
others don't. Specific, not vague.

[Closing — 1 sentence] Direct ask for a conversation.

{candidate name}
{phone} · {email} · {portfolio}
```

Rules: 250 words max · no filler phrases · every sentence earns its place ·
reference one specific JD detail so it's clearly not a template.

#### 1g. Answer custom application questions
If the JD page or application form has visible custom questions (e.g. "Why do you want
to work here?", "Describe a product you've shipped"), draft concise answers now.
Include them in the preview card.

#### 1h. Show preview card

Present this to the user before doing anything else:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  READY TO APPLY — {Company} · {Role}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  CV:           output/{filename}.pdf  ← tailored to this JD
  Cover letter: output/{filename}-cover-{date}.md
  URL:          {application url}
  Salary field: {target from profile.yml}

  Cover letter preview:
  ┌─────────────────────────────────────────┐
  │ {first 3 lines of cover letter}...      │
  └─────────────────────────────────────────┘

  {If custom questions found:}
  Custom questions answered:
  Q: {question}
  A: {drafted answer}

  Say "Apply Now" to proceed, or "skip" to move to the next job.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**WAIT for user response. Do not proceed until confirmed.**

#### 1i. On "Apply Now" confirmation

- Navigate browser to the application URL
- Provide all content ready to paste: cover letter text, custom question answers, salary figure
- Guide user through the form if needed (field by field)
- **STOP before clicking the final Submit/Apply button** — user clicks it themselves
- After user confirms they submitted: update shortlist Decision to `✅ Applied`

#### 1j. Update tracker
Write TSV to `batch/tracker-additions/{num}-{slug}.tsv` with status `Applied`.

### 2. Summary after all jobs processed

```
Done. Processed {N} roles:

  ✅ Applied  — {Company} · {Role}
  ❌ Closed   — {Company} · {Role} (posting no longer active)
  ⏭ Skipped  — {Company} · {Role} (user skipped)

Run `node merge-tracker.mjs` to sync the tracker.
```

---

## Single-job fast-lane

"Apply for job 003" or "submit PointClickCare" → same flow, single job.
Row must be `✅ YES` — if still PENDING: "Mark it YES in the shortlist first."

---

## On "skip"

User says "skip" at the preview card → move to next job, mark shortlist as `⏭ SKIPPED`.
User can revisit skipped jobs later with "apply for job {num}".
