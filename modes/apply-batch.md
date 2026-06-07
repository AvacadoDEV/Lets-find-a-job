# Mode: apply-batch

## Purpose

For every `✅ YES` row in `data/shortlist.md` that does not yet have a generated PDF, produce:
1. A tailored CV (HTML → PDF via Playwright)
2. A cover letter (1 page, JD-specific)
3. A pre-filled application checklist

**HARD RULE: This mode NEVER submits, clicks Apply, or sends anything.**
It stops at "documents ready — review before applying."

---

## Trigger

User says: "apply-batch", "generate documents for my YES jobs", "prepare applications"

---

## Steps

### 0. Guard check
Read `data/shortlist.md`. Find all rows where Decision = `✅ YES`.
If none: tell the user "No approved jobs yet. Review your shortlist first with `/career-ops shortlist review`."

### 1. For each YES row (process one at a time — never parallel Playwright)

#### 1a. Verify the job is still active
- Navigate to the URL with `browser_navigate` + `browser_snapshot`
- If the page shows only header/footer with no JD → mark as `❌ CLOSED` in shortlist and skip
- If active → continue

#### 1b. Extract the full JD
Read title, responsibilities, requirements, nice-to-haves, and any application questions.

#### 1c. Read candidate files
- `cv.md` — canonical CV
- `article-digest.md` — proof points (if exists)
- `modes/_profile.md` — archetypes, superpowers, story bank
- `config/profile.yml` — identity and comp
- `templates/cv-template.html` — design template

#### 1d. Tailor the CV
Reorder and rewrite bullets to mirror JD language. Rules:
- NEVER invent experience or metrics — only reframe what exists in cv.md
- Lead Professional Summary with the role title and company name
- Surface the 2–3 proof points most relevant to this specific JD
- Match keywords from the JD for ATS (exact phrases, not synonyms)
- Keep to 1 page if possible; 2 pages max
- Keep the Skills section last — lead with impact, not tools

#### 1e. Generate PDF
Write tailored HTML to `output/temp-{num}-{slug}.html`
Run: `node generate-pdf.mjs output/temp-{num}-{slug}.html output/{num}-{slug}-{date}.pdf`
Confirm PDF exists before continuing.

#### 1f. Write cover letter
File: `output/{num}-{slug}-cover-{date}.md`
Format:
```
{Company Hiring Team / Hiring Manager if known},

[Para 1 — 2 sentences] Why this role, why this company. Reference something specific from the JD or company (not generic).

[Para 2 — 3 sentences] Your strongest proof point for THIS role. Quote a JD requirement, map it to a real result from your CV. Use numbers.

[Para 3 — 2 sentences] What you bring that others don't. Your differentiator (technical PM from X, ad-serving scale, self-serve data analysis).

[Closing — 1 sentence] Direct ask for conversation.

Anup Rao
905-782-5239 · Raonoops@gmail.com · hire.anuprao.dev
```

Rules:
- 250 words max
- No "I am passionate about" or corporate filler
- Every sentence earns its place — cut anything that doesn't add signal
- Reference the JD by name or one specific detail to show it's not a template

#### 1g. Write application checklist
File: `output/{num}-{slug}-checklist-{date}.md`
```markdown
# Application Checklist — {Company} · {Role}

- [ ] Review tailored CV: output/{num}-{slug}-{date}.pdf
- [ ] Review cover letter: output/{num}-{slug}-cover-{date}.md
- [ ] Application URL: {url}
- [ ] Confirm salary field: target $X CAD (minimum $75K CAD)
- [ ] Note any custom questions in the application form (answer below)
- [ ] Click Apply yourself — career-ops does NOT submit

## Custom Application Questions
{list any questions found on the application page, with drafted answers}

## Notes
{any flags: visa question, salary transparency, referral needed, etc.}
```

#### 1h. Update shortlist
Add PDF filename to the row in `data/shortlist.md`. Change Decision cell to `✅ YES · PDF ready`.

#### 1i. Update applications tracker
Write TSV to `batch/tracker-additions/{num}-{slug}.tsv` with status `Evaluated` (not Applied — user applies manually).

### 2. Summary after all jobs processed

```
Done. Generated documents for {N} roles:

  ✅ {Company} — {Role} → output/{filename}.pdf
  ✅ {Company} — {Role} → output/{filename}.pdf

Next step: review each PDF and cover letter, then apply yourself at the URLs in each checklist.
Run `node merge-tracker.mjs` to sync the tracker.
```

---

## Single-job variant

User says: "apply for job 003" or "generate documents for PointClickCare"

Same steps as above but only for the specified row.
The row must be `✅ YES` in the shortlist — if it's still PENDING, say:
"That job hasn't been approved yet. Mark it YES in the shortlist first."
