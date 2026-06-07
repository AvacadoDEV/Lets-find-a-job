# Mode: shortlist

## Purpose

Manage `data/shortlist.md` — the human-gated queue between job discovery and application.

This mode handles three sub-commands:
- `add` — Score a new job and add it to the shortlist
- `review` — Show pending rows with a summary, prompt for YES/NO/MAYBE
- `status` — Print a count of PENDING / YES / NO / MAYBE rows

---

## add — Score and add a job to the shortlist

**Trigger:** User pastes a job URL or says "add this to shortlist"

**Steps:**
1. Read `cv.md`, `config/profile.yml`, `modes/_profile.md`
2. Fetch the JD (WebFetch → fallback WebSearch)
3. Score it using the scoring dimensions in `_shared.md`
4. Determine salary estimate (from JD or WebSearch if not listed)
5. Check remote/Canada eligibility explicitly — flag ⚠️ if US-only
6. Write one new row to the **Pending Review** table in `data/shortlist.md`

**Row format:**
```
| {next_num} | {YYYY-MM-DD} | {Company} | {Role Title} | {score}/5 | {salary range} | {✅ or ⚠️ + note} | {1-sentence why it fits Anup} | [Apply]({url}) | ⬜ PENDING |
```

**Rules:**
- Use 3-digit zero-padded number, continuing from max existing
- Keep "Why it fits" under 120 characters — be specific, cite one proof point
- NEVER add a row without verifying the URL loads a real JD
- If score < 3.5, add a clear note in "Why it fits" and still add it — let the user decide

---

## review — Show pending jobs, collect YES/NO/MAYBE

**Trigger:** User says "review shortlist", "show me what to review", or "let's go through the jobs"

**Steps:**
1. Read `data/shortlist.md`
2. Find all rows where Decision = `⬜ PENDING`
3. For each pending row, present a numbered card:

```
---
[001] StackAdapt — Technical Product Manager, AI
Score: 4.5/5 · ~$100K–$130K CAD · ✅ Remote Canada
Why: Canadian AdTech co, campaign platform + AI — direct match to X ad-serving + technical PM background
URL: https://job-boards.greenhouse.io/stackadapt/jobs/4111884009

→ Your call: YES / NO / MAYBE
```

4. Collect user's decision for each (can be all at once: "1=YES, 2=NO, 3=MAYBE...")
5. Update `data/shortlist.md`:
   - YES → move row to **Approved** section, change Decision cell to `✅ YES`
   - NO → move row to **Archive** section, change Decision to `❌ NO`
   - MAYBE → keep in Pending, change Decision to `🔶 MAYBE`
6. After collecting all decisions, show a submit prompt for each YES job:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  {N} job(s) approved:

  [001] StackAdapt — Technical PM, AI
  [003] Grafana Labs — Senior PM

  Say "submit" to fast-lane all YES jobs (recheck → tailor CV →
  generate PDF → cover letter → preview → one click to apply).
  Or name one: "submit 001"
  Or say "later" to stop here and run /career-ops apply-batch when ready.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

7. If user says "submit" or "submit {num}": hand off to `apply-batch` mode for those jobs.

**This step collects decisions and optionally triggers the fast-lane. It never submits applications itself.**

---

## status — Quick summary

**Trigger:** User says "shortlist status" or "how many jobs in the shortlist"

Print a one-line summary:
```
Shortlist: {N} pending · {N} approved (YES) · {N} maybe · {N} archived
```
Then list the YES rows as a compact checklist showing which ones have had CVs generated vs. not.
