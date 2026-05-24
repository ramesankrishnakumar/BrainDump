# Resume Builder

Generates an ATS-friendly **PDF** and **DOCX** from a markdown resume.

- **PDF** — headless Chrome (Playwright), styled for human reviewers. Single column, real selectable text.
- **DOCX** — python-docx, no tables/images, standard headings. Opens in Word or Google Docs (drag into Google Drive).

Both files are generated on every run.

## Usage

```bash
python3 build_resume.py                          # default: last-resume-revised-4.md
python3 build_resume.py my-resume.md             # -> my-resume.pdf + my-resume.docx
python3 build_resume.py my-resume.md --out Resume  # -> Resume.pdf + Resume.docx
```

Output files are written next to the input markdown, named after it (or `--out`).

### Validating the layout

Chrome can't screenshot a PDF directly (it downloads instead), so use `--preview` to
render the same HTML to a PNG and inspect it:

```bash
python3 build_resume.py --preview   # also writes <base>-preview.png
```

The console prints an approximate page count. Open the PNG and check: centered name,
section rules, right-aligned dates, clean page break, no clipping. (See the
`render_preview` "MUST DO" comment in the script — an agent editing the CSS should
always regenerate and read this PNG.)

## Markdown format

```
Krishnakumar Ramesan                  <- line 1: name
Hayward, CA | (256) ... | [linkedin](url)   <- contact (supports [text](url) links)
**Senior Software Engineer | ...**    <- tagline (bold)
---
## SECTION                            <- uppercase section header
### Company — Role | Aug 2022 – Present   <- role; date after "|" goes right-aligned
- **Bold lead-in** rest of bullet     <- bullet (supports **bold** + links)
```

Plain lines under a section (e.g. PROFILE, SKILLS) render as paragraphs.

## Setup

```bash
python3 -m pip install --index-url https://pypi.org/simple/ python-docx playwright
python3 -m playwright install chromium
```
