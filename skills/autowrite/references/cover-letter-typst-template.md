# Cover-letter Typst template

A designed-PDF cover letter that matches the resume Typst template's visual system. Use when the candidate is shipping a Typst-rendered resume and wants the cover letter to read as the second half of a matched pair -- same accent color, same font stack, same section-rule treatment -- rather than two unrelated documents.

The HTML cover-letter template (`cover-letter-html-template.md`) remains the right default for portability and zero install. Typst is the right pick when the resume is already rendered in Typst and the deliverable expectation is a single visually-coherent submission pair.

---

## When to pair Typst resume + Typst cover letter

- The candidate's resume is rendered from `resume-typst-template.md` (or a hand-rolled Typst variant of similar shape).
- The deliverable expectation is two PDFs that read as one design system.
- Director-level or higher openings where design coherence reads as a signal.
- The application portal accepts PDFs (not HTML) and the candidate wants the visual match.

**When NOT to pair:**

- Resume is rendered from HTML / pandoc / Word. The Typst cover letter will read as visually disjoint from the resume; use the matching HTML template instead.
- The candidate is uncomfortable installing Typst. Use HTML for both halves.
- The application portal flattens cover letters to plain text in its UI (common on Greenhouse / Lever / Ashby form fields). Render to PDF anyway as a fallback attachment, but expect the visual design to be invisible in the primary submission path.

---

## Install (same as resume Typst template)

Typst is a single binary, available via the most common package managers. If you already installed it for the resume template, you're done.

```
# macOS (Homebrew)
brew install typst

# Linux (Cargo)
cargo install typst-cli

# Windows (Winget)
winget install typst.typst

# Or download a release directly from
# https://github.com/typst/typst/releases
```

---

## Template (`cover-letter.typ`)

Drop this file alongside the cover-letter markdown. The configuration block at the top mirrors the resume template -- keep the values identical between the two files so the visual system holds.

```typst
// Cover letter -- Typst presentation layer
// Content source of truth: cover-letter.md (in this directory)
// Pair-rendered with: resume.typ in the same directory (shared accent + font stack)
//
// Regenerate the PDF with:
//   typst compile cover-letter.typ
//
// The .typ is a SEPARATE presentation layer. If cover-letter.md content changes,
// update this file to match, then re-compile.

// --- Configuration (mirror resume.typ exactly) ---------------------------
#let accent = rgb("#1f3a5c")          // accent color -- same value as resume.typ
#let muted = rgb("#5a5a5a")
#let faint = rgb("#777777")
#let body_color = rgb("#262626")

#let body_font = ("Helvetica Neue", "Arial")   // same font stack as resume.typ
#let body_size = 10pt                          // slightly larger than resume (resume is content-dense; letter is prose)
#let leading = 0.65em                          // slightly looser leading for prose readability

// --- Page setup ---------------------------------------------------------
#set page(paper: "us-letter", margin: (x: 1.5cm, top: 1.5cm, bottom: 1.2cm))
#set text(font: body_font, size: body_size, fill: body_color)
#set par(justify: false, leading: leading, spacing: 1em, first-line-indent: 0pt)

// --- Header (mirrors resume header for visual continuity) ----------------
#text(size: 23pt, weight: "bold", fill: accent)[Candidate Name]
#v(3pt)
#text(size: 10.5pt, weight: "medium", fill: rgb("#3a3a3a"))[Candidate title line / positioning]
#v(4pt)
#text(size: 9pt, fill: faint)[email\@example.com  ·  linkedin.com/in/candidate  ·  City, State]
#v(5pt)
#line(length: 100%, stroke: 1pt + accent)
#v(14pt)

// --- Date -----------------------------------------------------------------
#text(fill: muted)[Month DD, YYYY]
#v(10pt)

// --- Salutation ----------------------------------------------------------
Dear Hiring Manager,
#v(8pt)

// --- Body paragraphs -----------------------------------------------------
// Each paragraph stands alone -- Typst's #set par(spacing: 1em) handles spacing.
// Use *bold* and _italic_ for emphasis sparingly; cover letters carry less inline
// emphasis than resumes.

The opening paragraph goes here -- strategic positioning, anchor on the company's
current organizational situation, weave in the candidate's through-line.

The first body paragraph echoes a load-bearing hiring-manager-profile eval.
Specific resume evidence (project name, shipped artifact, quantified outcome) drawn
from the resume or supplementary library.

The second body paragraph echoes a second hiring-manager-profile eval, ideally one
that pairs with the JD's stated responsibilities. Same evidence rule.

Optional third body paragraph -- the external-value differentiator. Used when
something the candidate brings that an internally-promoted candidate at the target
company would not have by default is load-bearing for this role.

The closing paragraph -- two sentences. Peer-to-peer cadence, one concrete sentence
about next steps. No "thank you for your consideration" filler.

#v(14pt)

Sincerely,
#v(20pt)
Candidate Name
```

---

## Compile

From the directory holding `cover-letter.typ`:

```
typst compile cover-letter.typ
```

This produces `cover-letter.pdf` in the same directory. For a watch-and-rebuild loop while iterating:

```
typst watch cover-letter.typ
```

The watch process rebuilds the PDF on every save.

---

## Sync discipline (same as resume Typst)

The Typst file is a **presentation layer**. The markdown cover letter is the **content source of truth**. When the markdown changes, update the Typst file to match -- the Typst file is not auto-regenerated from markdown.

A sustainable sync protocol:

1. **Make the content change in markdown first.** Use the cover-letter subagent or per-bullet review for the editing pass.
2. **Mirror the change in the `.typ` file.** Same words, same paragraph breaks.
3. **Re-compile.** `typst compile cover-letter.typ`.
4. **Visually inspect** the rendered PDF -- catch any spacing or paragraph-break drift before submission.

If both `resume.typ` and `cover-letter.typ` are maintained off the same per-opening directory, keep their configuration blocks identical. Drift in accent color, font stack, or header style breaks the matched-pair illusion the Typst template is supposed to produce.

---

## Common adjustments

- **Page count over 1pp.** Cover letters should fit on one page. If the body runs long, trim the optional third paragraph first, then tighten the body paragraphs (the cover-letter subagent prompt's 250-400 word target for IC / manager and 350-500 for director roles is the operative discipline -- if the markdown is within range and the PDF still overflows, increase top/bottom margins by 0.2cm before adjusting font size).
- **Header feels too heavy.** The matched-pair design uses the resume's header verbatim on the cover letter for continuity. If the cover letter would land better with a lighter header, reduce `#text(size: 23pt, ...)` for the name to 18pt and drop the title line to 9pt; keep the accent color and rule.
- **Signature block looks cramped.** Increase `#v(20pt)` between "Sincerely," and the name to 30pt or 40pt. Some candidates leave space for a scanned signature image to be dropped in post-print; if so, increase to 60pt.
- **Date format.** US convention is "Month DD, YYYY" (e.g., "June 5, 2026"). UK / commonwealth convention is "DD Month YYYY" (e.g., "5 June 2026"). Match the convention of the target country.

---

## What this template is NOT

- **Not a substitute for the markdown.** The markdown cover letter is the canonical content. Downstream tools (mutation loops, audits, library-loading) operate on the markdown. The Typst file is a presentation layer; deleting it loses no content.
- **Not a layout engine for cover-letter variations.** Single-column, single-page, prose. Multi-column cover letters or cover letters with sidebars / graphics are out of scope. ATS portals that scrape cover-letter PDFs strip layout before keyword-matching; non-conventional layouts add no signal and frequently break the scrape.
- **Not styled to mimic Word's default.** Word's default cover letter (Calibri, single-spaced, left-aligned date) is what every other candidate uses. The Typst pair is a deliberate departure from that default -- accent color, designed header, deliberate spacing. Treat the visual distinction as intentional.
