# Per-bullet review mode

A human-in-the-loop alternative to the autonomous mutation loop. Use when the user wants direct control over every change to a resume variant, when the variant is scoped to a single opening (no multi-company convergence), or when length is constrained and Highlight-altitude calls need explicit user judgment per bullet.

Structurally distinct from:

- **Autonomous mutation loop** (the default `autowrite` flow): the parent skill picks mutations, applies them, scores, keeps or discards. The user reviews the changelog at the end.
- **Career-intake Q&A rounds** (`career-intake` skill): the user answers open questions; the assistant drafts bullets from the candidate's framing; the candidate confirms or corrects.
- **Per-bullet review** (this mode): the assistant proposes a small set of variant changes up front (typically 4-8), then walks the user through them one at a time -- before/after in monospace, brief tradeoff statement, 3-4 numbered options, user picks a number, the edit applies, the next change comes up.

The user types `1` or `2` (or whatever). No prose required. The pace is fast and the user retains every change-level decision.

---

## When to use this mode

- Single-company variant where the user wants control without the multi-hour autonomous loop.
- Variants where the candidate has strong opinions about specific bullets and would override most autonomous mutations anyway.
- Director-level or sensitive openings where each Highlight altitude call matters and a single bad mutation tanks the variant's voice.
- After a manual one-shot audit (see manual one-shot review mode in `SKILL.md`) has surfaced a top-3 gap list -- per-bullet review is the natural follow-on to address them one at a time.
- When the candidate has explicitly asked for a per-bullet walk-through ("walk me through each change," "let me approve every edit," "per-bullet review").

**When NOT to use this mode:**

- Multi-company convergence runs -- use the autonomous loop.
- Initial substrate building -- use career-intake.
- Quick polish passes where the candidate has high trust in the autonomous loop's mutation quality.

---

## Prerequisites

Before starting per-bullet review, the assistant should have:

1. **A target resume variant draft.** Either the autonomous loop produced one and the user wants to per-bullet review it; or the assistant ran a manual one-shot audit and produced a candidate variant; or the user supplied a draft directly. The variant lives at a known path -- this mode edits that file in place.
2. **A short list of change points** (typically 4-8). Each change is one before/after pair. More than ~10 changes in a single per-bullet review session is too long; split across sessions or drop the lower-priority ones.
3. **A clear ordering.** Walk top-down through the document (title → summary → highlights → experience top-to-bottom → patents → education). Within a section, walk in document order. Don't jump around -- the cognitive cost of context-switching per change is real.

If any of the three are missing, surface them before starting the review.

---

## The presentation format per change

Each change point gets its own message. The shape:

```
## Change [N] -- [short label]

```
LOCKED [original / canonical]:

[The original sentence or bullet, verbatim, in monospace.]


VARIANT (drafted):

[The proposed replacement, verbatim, in monospace.]
```

**[2-4 sentence tradeoff statement.] What changes, why, what it costs.

**Options:**
1. **[First option.]** [One-line description of what this does.]
2. **[Second option.]** [One-line description.]
3. **[Third option.]** [One-line description.]
4. **[Optional fourth.]** [One-line description.]

Your call?
```

The user replies with a number. The assistant applies the corresponding edit (or no edit, for a `revert`/`keep-as-locked` option), confirms in one line ("Locked as drafted -- no edit needed" or "Title swapped"), and moves to the next change.

### Why this exact shape

- **Monospace before/after** so the diff is unambiguous. Prose paraphrases of changes lose nuance.
- **2-4 sentence tradeoff** is enough to make the call without overwhelming. More than that and the user starts skimming.
- **Numbered options** because a single number reply is the lowest-friction interaction the user can give. No prose required, no copy-paste, no confusion about which version got picked.
- **Always include a revert option** (or a "keep as locked" option). Some changes the user will reject outright -- the option must exist.
- **3-4 options is the sweet spot.** 2 is binary; 5+ is decision-paralysis. Three covers most cases; four when there's a genuinely useful hybrid.

### Common option shapes

- **Keep canonical / revert.** Always include this -- some changes will be rejected.
- **Keep variant as drafted.** The proposed change applies as-is.
- **Hybrid.** A middle option that takes part of the variant but preserves part of the canonical.
- **Different wording.** Sometimes the right call is a third phrasing neither the canonical nor the drafted variant got right.
- **Cut entirely.** When the change point is whether to retain a bullet at all, "cut" is a legitimate option.

---

## What the assistant does during the review

For each change:

1. **Post the change message** in the exact shape above.
2. **Wait for the user's reply.** A single digit or short phrase is the expected format.
3. **Apply the edit** (or no-op for revert/keep) using the file-edit tool. One edit per response.
4. **Confirm in one line.** What changed, or no edit needed.
5. **Move to the next change** without waiting for the user to prompt.

Do NOT batch multiple changes in one message. Do NOT skip the wait. Do NOT propose new changes mid-review that weren't in the original change list -- if a new idea surfaces, log it and address it in a follow-on session.

If the user pushes back on the change list itself (e.g., "actually I don't want to touch the summary at all"), drop the affected changes from the list and re-confirm the remaining order before continuing.

---

## After the last change

When the final change is locked, the review ends with a summary table:

```
## Per-bullet review complete

All [N] decisions locked. Summary:

| # | Item | Decision |
|---|---|---|
| 0 | [Change 0] | [Decision -- what was applied] |
| 1 | [Change 1] | [Decision] |
| ... | ... | ... |
```

Then a short close-out checklist:

1. **Variant frontmatter cleanup.** The variant's frontmatter likely still describes the pre-review state. Update `status` (e.g., `draft` -> `reviewed`), expand the diffs list to capture every applied change, move resolved open questions to a `resolutions` field.
2. **Decision log.** If the user maintains a decision log (e.g., a career-database `DECISIONS.md`), offer to add an entry recording the per-bullet review and the substantive decisions.
3. **Deliverable port.** If the variant is the markdown source for a designed PDF (Typst, HTML, or other presentation layer), offer to re-render. The presentation layer must be synced manually to the markdown.
4. **Tracker update.** If the variant is tied to a specific job application, update the application tracker's `last touch` and `next step` columns.

Offer all four; do not assume which the user wants. The review is the substantive work; the close-out is procedural and the user picks what they want done.

---

## Pacing and turn discipline

- **One change per turn.** Never more. The point of per-bullet review is per-bullet control; batching defeats it.
- **No commentary between changes** unless the user asks. A user who just typed `1` does not want a paragraph of agreement -- they want change 2.
- **Acknowledge dissent without re-arguing.** If the user picks an option the assistant would have argued against, apply the edit and move on. The user retains the call.
- **If the user types a non-numeric response,** treat as a freeform option specification ("actually, can we drop this entirely?") -- adapt the edit to match, confirm in one line, move on.
- **If the user pauses mid-review,** the file state should be self-evident from the changes already applied. Don't try to summarize where you are; the user can scroll up. When the user returns, pick up at the next unprocessed change.

---

## Worked example (shape only -- do not copy verbatim)

The user has a director-level resume variant draft and four proposed changes from a manual audit. The assistant opens:

> Going top-down through the doc, one decision at a time. I'll present each as before/after; you call keep / modify / cut and we move down.

Change 0 message:

```
## Change 0 -- Title line

```
LOCKED [canonical]:
[The candidate's locked title line for their canonical resume.]

VARIANT (drafted):
[A proposed alternative title that mirrors the JD's title verbatim.]
```

**Tradeoff.** Mirroring the JD title verbatim is a recruiter-skim hit. But the locked title carries additional positioning the variant title drops -- specifically [the dropped signal]. The summary still carries [the dropped signal], so the variant doesn't lose it entirely; it demotes it from the title to body altitude.

**Three viable calls:**
1. Keep locked title verbatim (no change).
2. Swap to the verbatim JD-mirror.
3. Hybrid: lead with the JD title, keep the dropped signal as the second clause.

Your call?
```

User types `2`. Assistant applies the title swap, confirms in one line ("Title swapped to the JD-mirror version"), proceeds to change 1.

---

## The test

A good per-bullet review:

1. **Walked top-down through the variant.** No jumping around.
2. **Presented every change in the same shape** (before/after, tradeoff, numbered options) so the user got into a rhythm.
3. **Applied edits incrementally** -- one per user response.
4. **Did not propose new changes mid-review** outside the original list.
5. **Closed out with a summary table + procedural checklist** (frontmatter, decision log, deliverable port, tracker).

The user should have spent more time deciding than reading. If the user is reading paragraphs of assistant prose between changes, the format has drifted.

---

## How this mode connects to the autonomous loop

The autonomous loop (the default `autowrite` flow) and per-bullet review are not competitors; they are different tools for different jobs.

- **Autonomous loop** optimizes across many companies in parallel, mutates aggressively, finds gaps the candidate would not have noticed. Use when scoring against a multi-company target set.
- **Per-bullet review** ratifies (or rejects) a small set of pre-identified changes against one variant. Use when the candidate wants the discipline of named tradeoffs and the control of per-change approval.

A common end-to-end flow uses both:

1. Run the autonomous loop against the target set; let it converge per company.
2. Pick one company's locked variant.
3. Run a manual one-shot audit on it (or use the secondary loop's hiring-manager profile) to surface the top 4-8 gaps for the specific opening.
4. Per-bullet review walks the user through addressing those gaps.
5. Render the deliverable (HTML, Typst, or other).

Each stage hands off cleanly to the next. The per-bullet review is the final human-in-the-loop step before submission.
