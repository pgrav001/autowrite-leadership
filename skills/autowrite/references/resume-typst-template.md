# Resume Typst template

A designed-PDF alternative to the HTML-rendered template. Use when the deliverable bar is "a recruiter opens this PDF and the typography signals senior-level care" and the pandoc-default PDF output reads visually dense or generic.

The HTML template (`resume-html-template.md`) remains the right default for most users: zero install, portable, prints via any browser. Typst is the right pick when the user wants explicit control over accent color, section rules, date alignment, and consistent typography across the document -- the kind of design pass that lands at director level and above.

---

## What this template gives you

- A single-file `.typ` source that compiles to a 2-page PDF.
- Accent color, section rule, and right-aligned date convention -- configurable in three lines at the top.
- Typography: a configurable sans-serif font stack, with the same body font used throughout for ATS legibility.
- Section header style: uppercase + tracking + a thin underline rule, drawn from the accent color.
- Right-aligned dates / locations / metadata next to each company and role row -- the convention recruiters scan for.

Compare to HTML-via-pandoc: HTML wins on portability and zero install. Typst wins on layout precision (right-aligned dates are a one-liner in Typst, awkward in pandoc HTML), on accent color consistency, and on the page-break behavior that keeps role headers from orphaning at the bottom of page 1.

---

## When to pick which

**Pick HTML** when:

- The user has no design preference beyond "looks clean."
- Deliverables need to render in a browser as well as PDF.
- The user is uncomfortable installing additional tooling.
- The user already prints to PDF via Chrome / Safari / Firefox and that workflow works.

**Pick Typst** when:

- The user wants accent color, section rules, and right-aligned dates as a single design system.
- The user is applying at director level or above where document design itself reads as a signal.
- The deliverable will be attached to a job application portal that wants a PDF (not HTML).
- The user has pushed back on pandoc-default PDF output as "visually dense" or "generic."

Both can coexist: render the same markdown source through HTML for portable preview and Typst for the designed submission PDF. The markdown is the canonical content; both presentation layers track it.

---

## Install (one-time)

Typst is a single binary, available via the most common package managers.

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

Confirm the install:

```
typst --version
```

---

## Template (`resume.typ`)

Drop this file alongside the resume markdown. Adapt the configuration block at the top (accent color, font stack, content) to the candidate.

```typst
// Resume -- Typst presentation layer
// Content source of truth: resume.md (in this directory)
// Regenerate the PDF with:
//   typst compile resume.typ
//
// The .typ is a SEPARATE presentation layer. If resume.md content changes,
// update this file to match, then re-compile.

// --- Configuration --------------------------------------------------------
#let accent = rgb("#1f3a5c")          // accent color (used for name, section headers, rules)
#let muted = rgb("#5a5a5a")           // dates next to companies
#let faint = rgb("#777777")           // role metadata (location, sub-dates)
#let body_color = rgb("#262626")      // main body text

#let body_font = ("Helvetica Neue", "Arial")   // sans-serif fallback chain
#let body_size = 9.7pt
#let leading = 0.58em
#let list_spacing = 0.46em

// --- Page setup -----------------------------------------------------------
#set page(paper: "us-letter", margin: (x: 1.5cm, top: 1.2cm, bottom: 1.1cm), numbering: "1")
#set text(font: body_font, size: body_size, fill: body_color)
#set par(justify: false, leading: leading, spacing: leading)
#set list(marker: text(fill: accent)[•], spacing: list_spacing, body-indent: 0.5em)

// --- Reusable section / company / role helpers ---------------------------
#let section(title) = {
  v(5pt)
  text(size: 10.5pt, weight: "bold", fill: accent, tracking: 1pt)[#upper(title)]
  v(2pt)
  line(length: 100%, stroke: 0.5pt + rgb("#c7d0d8"))
  v(4pt)
}

#let company(name, dates) = {
  grid(columns: (1fr, auto), column-gutter: 8pt,
    align(left, text(weight: "bold", size: 10.5pt)[#name]),
    align(right + horizon, text(size: 9.5pt, fill: muted)[#dates]))
  v(1pt)
}

#let role(title, meta) = {
  grid(columns: (1fr, auto), column-gutter: 8pt,
    align(left, text(style: "italic", fill: rgb("#383838"))[#title]),
    align(right + horizon, text(size: 9pt, fill: faint)[#meta]))
  v(3pt)
}

// --- Header --------------------------------------------------------------
#text(size: 23pt, weight: "bold", fill: accent)[Candidate Name]
#v(3pt)
#text(size: 10.5pt, weight: "medium", fill: rgb("#3a3a3a"))[Candidate title line / positioning]
#v(4pt)
#text(size: 9pt, fill: faint)[email\@example.com  ·  linkedin.com/in/candidate  ·  City, State]
#v(5pt)
#line(length: 100%, stroke: 1pt + accent)
#v(7pt)

// --- Summary paragraph ---------------------------------------------------
A short summary paragraph here -- 2-4 sentences. Lead with the claim, prove with the proof.

// --- Sections ------------------------------------------------------------
#section("Selected Highlights")

- *Bullet 1 headline* -- short prose backing the claim.
- *Bullet 2 headline* -- short prose backing the claim.
- *Bullet 3 headline* -- short prose backing the claim.

#section("Experience")

#company("Company Name", "Start – End")
#role("Title", "Location · Date range")

- Bullet 1 for this role.
- Bullet 2 for this role.

#v(4pt)
#role("Earlier title at same company", "Location · Earlier date range")

- Bullet 1.
- Bullet 2.

#v(6pt)
#company("Prior Company", "Start – End")
#role("Title", "Location")

- Bullet 1.

#section("Education")

*University Name* -- Degree; relevant minor or field
```

---

## Compile

From the directory holding `resume.typ`:

```
typst compile resume.typ
```

This produces `resume.pdf` in the same directory. For a watch-and-rebuild loop while iterating:

```
typst watch resume.typ
```

The watch process rebuilds the PDF on every save.

---

## Sync discipline

The Typst file is a **presentation layer**. The markdown resume is the **content source of truth**. When the markdown changes, update the Typst file to match -- the Typst file is not auto-regenerated from markdown.

A sustainable sync protocol:

1. **Make the content change in markdown first.** Use the per-bullet review or autonomous loop for the editing pass.
2. **Mirror the change in the `.typ` file.** Same words, same order. The Typst syntax (`*bold*`, `_italic_`, `-` for bullets) maps closely to markdown.
3. **Re-compile.** `typst compile resume.typ`.
4. **Visually inspect** the rendered PDF -- catch any layout drift (orphaned section headers, page breaks landing mid-bullet) before submission.

If the user routinely runs both HTML and Typst presentation layers off the same markdown, build the habit: any content edit ships with both presentation-layer updates in the same change. Otherwise the layers drift and the deliverable PDFs go stale.

---

## Common adjustments

- **Page count over 2pp.** Trim the summary first, then merge or split bullets, then drop the lowest-priority section. Do not shrink body font below 9.5pt at letter size -- recruiters notice.
- **Accent color too loud / too dim.** Adjust `#let accent = rgb("#...")` at the top. Three viable palettes: muted navy (`#21425d`), forest green (`#2d4a3e`), and warm charcoal (`#3a3a3a`). Stay away from saturated red, orange, or pure black; the first two read as aggressive at director level and the third loses the design signal entirely.
- **Date column too wide.** Trim the date format -- `Jan 2024 – May 2026` is the recruiter-readable convention; longer formats start consuming bullet width.
- **Orphan section headers** (a section title at the bottom of page 1 with no bullets under it). Add `#v(...)` spacing above the orphan, or reflow the prior section's bullets so the header falls naturally on page 2.

---

## What this template is NOT

- **Not a Word substitute.** Typst exports PDF; it does not export `.docx`. If the user needs a Word-editable deliverable, use pandoc to render the markdown to `.docx` separately (`pandoc resume.md -o resume.docx`). Treat the docx as the "user edits this in Word" fallback; Typst handles the designed PDF.
- **Not a layout engine for arbitrary content.** This template assumes a single-column resume with the conventional sections (summary, highlights, experience, education, sometimes patents / awards / publications). Multi-column layouts, sidebar formats, or graphical resumes are out of scope -- and rightly so. Designed PDFs that depart from single-column convention often fail ATS parsing on portals that strip layout before keyword-matching.
- **Not a permanent fork.** The Typst file is a sibling to the markdown, not a replacement. The markdown remains the source of truth that downstream tools (mutation loops, audits, cover-letter generation) operate on.
