---
name: cover-letter
description: >-
  Write a short, tailored, human-sounding cover letter from a job description plus the user's
  resume, then render it to PDF with build_cover_letter.py and spot-check the output. TRIGGER
  when the user says things like "write a cover letter for this role", "draft a cover letter",
  "make a cover letter for <company>", or hands over a job posting and their resume and asks for
  a letter. Do NOT use for resume edits, generic writing help, or when the user only wants talking
  points rather than a finished letter.
---

# Cover Letter

Your job is to land the user an interview. Act as an experienced recruiter reading the posting:
figure out what this specific company is actually screening for, then surface the resume
achievements that prove the user has done exactly that. A tailored, concrete letter beats a
polished generic one every time.

## Step 1 — Read both inputs fully

- Read the **job description** and the **resume** before writing a word. Both are usually given as
  file paths.
- From the JD, extract the 3-4 things the role really centers on (the "you'll work on..." and
  "what we look for" sections). These are your targeting anchors.
- From the resume, find the achievements that map one-to-one onto those anchors, preferring ones
  with hard metrics (scale, latency, percentages, dollar/time saved).

## Step 2 — Draft, recruiter mindset

- **Lead each body paragraph with the strongest role-matching achievement**, not a generic intro.
  Open paragraph 1 by naming the role and tying the company's mission to what the user builds.
- **Latest company first**, then the next one. Only include companies that strengthen the match.
  Ask the user (or check prior instructions) which jobs to omit, but never pad with weak/old roles.
- **Highlight AI / recent work when relevant** to the role, with concrete mechanics and numbers
  rather than buzzwords.
- Keep it **short**: roughly 3-4 body paragraphs plus a one-line close. It must fit on one page.
- Map skills honestly to the company's stack (e.g. "my recent work is in Python" if the backend is
  Python) without overclaiming frameworks the user hasn't used.

## Step 3 — Style rules (always on)

- **No em dashes.** Use a comma, "which is", "namely", or split into two sentences instead.
- **No semicolons.** Rewrite as two sentences or use a comma.
- Professional but human: contractions are fine, avoid stiff corporate filler and clichés
  ("synergy", "fast-paced", "I am writing to express my interest"). Write like a competent person
  talking to another person.
- Check the user's saved style feedback (auto-memory) before finalizing; honor any standing
  preferences.

## Step 4 — Match the build script's markdown format

The renderer `build_cover_letter.py` parses a fixed structure. Produce the `.md` in exactly this
order, separated by blank lines:

```
Sender Name
Contact line (pipe-separated; markdown links like [text](url) are supported)
Date (e.g. May 26, 2026)

Recipient line 1 (e.g. Company Hiring Team)
Recipient line 2 (Company)
Recipient line 3 (City, State)

---

Dear <recipient>,

Body paragraph 1...

Body paragraph 2...

...

Sincerely,
Sender Name
```

Notes on the parser: blank lines separate paragraphs; consecutive short lines (under 60 chars) like
the closing get joined with line breaks, so keep "Sincerely," and the name on adjacent lines.
Name the file to match the script default if convenient (`<Company>_CoverLetter.md`).

## Step 5 — Render and spot-check (do not skip)

The script lives at `Jobs/build_cover_letter.py`, while each letter lives in its own company
subfolder (`Jobs/<Company>/<Company>_CoverLetter.md`). Run the script from the `Jobs/` folder and
pass the letter's relative path:

```
cd Jobs
python3 build_cover_letter.py <Company>/<Company>_CoverLetter.md
```

The PDF is written next to the `.md`. The verification commands below assume you `cd` into
`Jobs/<Company>/` first (or adjust the paths accordingly).

Then **verify two things against the real PDF**, not the HTML preview:

1. **Page count must be exactly 1.** Check the real PDF, not the `--preview` PNG (its page estimate
   is a rough heuristic and is misleading):
   `python3 -c "import re; d=open('<Company>_CoverLetter.pdf','rb').read(); m=re.search(rb'/Count\s+(\d+)', d); print(m.group(1).decode())"`
2. **Rasterize the actual PDF and read it** to confirm real margins and clean rendering (links
   resolve, no overflow, no stray markdown like literal `**` or `[]()`). On macOS:
   `sips -s format png <Company>_CoverLetter.pdf --out _check.png`, then read `_check.png`.
   Do NOT trust the script's `--preview` PNG for margins or pagination: that PNG is a screenshot of
   the HTML and ignores the print `@page` rules, so it can look fine while the PDF is 2 pages or has
   different margins.

If it spills onto a second page, prefer pulling it back without gutting content, in this order:
remove a full paragraph's worth of text or fold sentences together (trimming a few words rarely
drops a whole line); then, if still over by a line or two, tune the script's typography (it uses a
single CSS `@page` margin, a body `line-height`, and a `.body p` `margin-bottom`, e.g. line-height
1.4 and 9pt paragraph spacing fit a dense one-pager while still looking professional). Re-run and
re-check the real PDF each time.

## Known pitfall — where margins come from

Control print margins in ONE place. The script sets margins via the CSS `@page { margin: ... }`
rule and passes NO `margin` to `page.pdf` (Playwright defaults that to none). If you ever see the
HTML preview look like it fits one page (whitespace below the signature) but the PDF reports 2
pages, suspect margins being applied twice (a CSS `@page` margin AND a `page.pdf(margin=...)` both
active stack and squeeze the content area). Keep exactly one margin source. Fix the margin source
rather than mangling the letter to fit a squeezed area.

## Finish

Report what you produced (the `.md`, the `.pdf`), confirm it is one page and style-clean (no em
dash, no semicolon), and point the user at the files.
