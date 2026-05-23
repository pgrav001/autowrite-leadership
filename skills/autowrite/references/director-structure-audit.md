# Director resume structural audit

When the active autowrite run has `director_level: true`, the parent skill runs a one-pass structural audit of the resume in Step 4 (baseline), AFTER the baseline scoring completes and BEFORE the user-confirmation gate that precedes the mutation loop. This is the most consequential addition to the director-level pipeline -- it catches resume-structure failures upstream of the line-level mutation loop, which can only re-shape bullets, not re-architect a resume.

The audit is **director-only**. Do NOT run it on IC runs -- the structural patterns it checks for are wrong-level for IC resumes and would propose mutations that would actively hurt an IC candidate.

The audit produces a structured findings list and surfaces it to the user with a single choice: bundle the structural fixes as experiment 0.5 before the line-level loop begins, or skip and let the loop discover them via eval scores.

---

## Why a structural audit (and why now)

Director resumes fail more often for structural reasons than for bullet-level reasons. The most common failure modes:

1. **No executive summary** -- the resume jumps from contact line straight to the first role. Director recruiters scan for a 3-5 line summary stating scope + arc + lens; without one, the resume reads as a senior IC resume that was relabeled.
2. **Output-first framing** -- bullets lead with what shipped, not what the candidate decided, restructured, hired, or sponsored. Reads as an IC who happened to manage a team, not a director who runs an organization.
3. **Headcount and budget invisible** -- the candidate's actual scope (team of 18, $4.2M opex) appears nowhere in the resume, even when they own it. Recruiters infer scope downward by default; if you don't surface yours, they assume it's smaller than it is.
4. **Title progression buried** -- the candidate's IC -> manager -> director arc isn't visible at a glance. A reader has to read all four roles to see it.
5. **No external-value differentiator** -- for any director search where an internal candidate is in play (most of them), the resume needs to surface what the external candidate brings that an internal promotion would not. Cross-industry experience, scale at a different stage of company, specific external practices, public artifacts.
6. **No "Selected Accomplishments" or "Career Highlights" section** -- 8+ year careers benefit from a non-chronological top section that surfaces the 3-5 strongest wins. Without it, a recruiter who only reads the most-recent role misses the candidate's strongest work.

The line-level mutation loop can fix bullets. It cannot add a missing section, re-order the resume, or convert output-first framing to scope-first framing at the section level. Those changes need to happen before the loop runs.

---

## When the audit runs

In autowrite SKILL.md Step 4 (establish baseline), inserted as **Step 4.5** between the baseline scoring (current sub-step 9) and the user-confirmation gate (current sub-step 10).

Pseudo-flow:

1. (Existing) Spawn all recruiter subagents in parallel; collect reports.
2. (Existing) Compute baseline aggregate; write results.tsv row; update results.json.
3. **(New) If `director_level: true`: run the audit (this file). Surface findings, get user choice.**
4. **(New) If user chose "apply now": run experiment 0.5 (structural bundle); re-score; keep/discard.**
5. (Existing) User confirms baseline; proceed to Step 5 (mutation loop) from the post-audit working file.

The audit is read-only at the file level until the user accepts. No structural mutation is applied without explicit user consent.

---

## What to check

For each check below: detect the pattern in the resume's markdown, score it as `pass | weak | missing`, and -- if non-pass -- formulate a proposed structural mutation.

### Check 1: Executive summary

**Detection:**
- Look for a section heading at the top of the resume named: `Summary`, `Profile`, `Executive Summary`, `Professional Summary`, `About`, or similar variant.
- OR a paragraph block between the contact line (the `<p class="contact">`-equivalent paragraph(s) under the H1) and the first H2 section.

**Pass criteria:**
- Block exists with 3-5 lines (sentences, not bullets).
- Names a scope dimension: team size OR budget OR headcount OR functional span ("two product orgs across 35 engineers").
- Names a functional arc: at least one prior-role context ("12 years building consumer products" or "Engineering leader who has scaled three teams from 0 to 20+").
- Names a strategic lens: what the candidate is currently building / focused on / known for ("Focus on shipping consumer AI at internet scale", "Specializes in turning around underperforming teams without backfilling externally").

**Weak (one or two of the three scope/arc/lens dimensions are present but not all three):** finding "executive summary is present but underspecified" -- proposed mutation: revise the summary to add the missing dimension(s), drawing from the candidate's current role bullets and title progression.

**Missing (no block at all):** finding "no executive summary" -- proposed mutation: add a summary block at the top, drawing scope from the current role's bullets and supplementary library (if loaded), arc from the title progression, and lens from the role qualifier the user provided OR the most-recent role's strategic focus.

### Check 2: Scope metadata at role headings

**Detection:**
- For each role in the Experience section: read the line(s) directly under the role's H3 heading. Is there an italic or metadata line that names team size, headcount, budget, or scope?

**Pass criteria:**
- For each role at director-equivalent level (Director, VP, Head of, Senior Manager, or any role with stated leadership scope): the heading or the metadata line below it includes at least one quantified scope dimension (team of N, $X budget, N reports, N+ engineers, or similar).

**Weak (1-2 of the director-equivalent roles surface scope but the rest don't):** finding "role scope inconsistent across the experience section" -- proposed mutation: add a `*<dates> | Team of N | $X opex*` line under each director-level role heading where it's missing.

**Missing (no roles surface scope metadata):** finding "role scope not surfaced at heading level" -- proposed mutation: add scope metadata lines under each director-level role heading.

**Pulling the numbers:** the candidate may not have stated team size or budget in the resume body. Check the supplementary library (`bullets/`, `interview-notes/`, `narratives/`). If still not found, flag a candidate question in the changelog (`Candidate question: what was the team size and budget at <Role> at <Company>? Not in resume or library.`) -- do not invent numbers.

### Check 3: Decision verbs vs IC verbs in bullets

**Detection:**
- Sample 8-15 bullets across the candidate's most recent 2-3 director-level roles (skip earlier IC roles -- those bullets correctly use IC verbs).
- For each bullet, classify its leading verb:
  - **Decision/leadership verbs:** Restructured, Reorganized, Hired, Built [the team / the function], Sponsored, Decided to, Recovered, Negotiated, Aligned, Influenced, Persuaded, Promoted, Mentored, Coached, Cut, Killed, Wound down, Scaled, Doubled, Tripled, Reduced [headcount/spend], Owned, Delivered against [a strategic outcome], Drove [a cross-functional outcome].
  - **IC verbs:** Built [a system], Shipped, Implemented, Coded, Designed [a specific technical artifact], Optimized, Refactored, Debugged, Deployed, Wrote, Tested, Reviewed.

**Pass criteria:**
- At least 50% of sampled bullets in recent director-level roles lead with decision/leadership verbs.

**Weak (30-49% decision verbs):** finding "bullets read as mixed IC/leader contributions" -- proposed mutation: rewrite the 3-5 lowest-signal IC-verb bullets to lead with the decision and its org consequence, using framings from the supplementary library where available.

**Missing (under 30% decision verbs):** finding "bullets read predominantly as IC contributions" -- proposed mutation: rewrite the top 5-7 bullets across recent director-level roles to lead with decision verbs. This is the most common director-resume failure and the highest-leverage structural fix.

### Check 4: Title progression legibility

**Detection:**
- Read role titles in Experience section in order.
- Is there a visible IC -> manager -> director progression OR a clear lateral movement at the director level?
- Are titles formatted consistently so a recruiter scanning vertical alignment can read the arc?

**Pass criteria:**
- Progression is apparent within ~5 seconds of scanning the resume. Either:
  - Chronological titles show clear upward progression (Engineer -> Senior Engineer -> Engineering Manager -> Director), OR
  - The candidate has been at director level for multiple roles, and the breadth of those roles is apparent (Director of X at Company A -> Director of Y at Company B).

**Weak (progression exists but is hard to read because of formatting or company-prefixed titles):** finding "title progression is structurally present but hard to read" -- proposed mutation: reformat the role headings so titles are visually dominant over company names, or add a `Career Highlights` section that surfaces the arc explicitly.

**Missing (progression is not legible -- e.g., the candidate has multiple director roles but they read like a flat horizontal list, or there are gaps that aren't explained):** finding "title progression not legible" -- proposed mutation: add a 3-5 line `Career Highlights` section after the summary that names the arc explicitly.

### Check 5: External-value differentiator

This is the check that is most commonly missing AND most often the difference between losing to an internal candidate and winning the role.

**Detection:**
- Read the entire resume and ask: what does this candidate offer that an internally-promoted candidate at the target company would not?
- Surface signals to look for: cross-industry transitions, scale shifts (startup -> enterprise or vice versa), specific external practices the candidate brought to past roles (an eval methodology, a hiring framework, a specific incident-response playbook), public artifacts (talks, papers, OSS, podcasts, recognized presence in a professional community), board / advisory / startup-founder experience.

**Pass criteria:**
- At least one differentiator is explicitly visible in the summary or the top of the experience section.

**Weak (a differentiator exists in the candidate's history but isn't surfaced -- e.g., the candidate ran a startup but it's buried as a one-line entry):** finding "external-value differentiator present but buried" -- proposed mutation: surface the differentiator in the summary OR add it as a `Selected Accomplishments` entry near the top.

**Missing (no differentiator visible at all):** finding "no external-value signal" -- proposed mutation: identify a differentiator from the supplementary library (if loaded) and surface it. If nothing surfaces from the resume or library, flag a candidate question (`Candidate question: is there a cross-industry, scale-shift, or public-artifact differentiator we should surface? Not in resume or library.`) -- do not invent one.

### Check 6: Selected Accomplishments / Career Highlights section

**Detection:**
- Is there a section between the summary and the Experience section that surfaces 3-5 strongest accomplishments, non-chronologically?

**Pass criteria:**
- Section exists with 3-5 entries. Each entry is one line, lead with the outcome (not the activity).

**Weak (section exists but has more than 7 entries, or entries lead with activities rather than outcomes):** finding "Career Highlights section present but unfocused" -- proposed mutation: trim to 3-5 strongest, reframe each to lead with outcome.

**Missing AND the candidate has 8+ years of experience:** finding "no Career Highlights section" -- proposed mutation: add a section after the summary with 3-5 top accomplishments drawn from across the candidate's career (not just the most recent role).

**Missing AND the candidate has under 8 years of experience:** skip this finding. The candidate's chronological experience section is sufficient.

---

## How to present findings

After all six checks complete, post a single chat message:

> Director-level structural audit (resume: `<filename>`):
>
> Found `<N>` structural findings before line-level mutation begins. The line-level loop can re-shape bullets but cannot add sections, re-order structure, or convert output-first framing to scope-first at the section level. These need to be addressed structurally OR the loop will spend experiments on bullets that work at the wrong abstraction.
>
> 1. **`<Check name>`** (`<missing | weak>`): `<one-sentence finding>`. Proposed mutation: `<one-sentence proposal>`.
>
> 2. **`<Check name>`** (`<missing | weak>`): `<one-sentence finding>`. Proposed mutation: `<one-sentence proposal>`.
>
> ...
>
> Choices:
>
> - **`apply now`** -- I bundle these as experiment 0.5, apply them all at once, re-score against all profiles. If the bundle's aggregate score holds or improves, the regular mutation loop continues from this new baseline. If it drops, all structural changes revert and the regular loop runs against the original baseline. Recommended for director-level runs with 2+ findings.
> - **`show me the text`** -- I post the proposed text for each mutation in chat before applying. You can edit, confirm, or cancel each one.
> - **`skip`** -- I proceed to the regular mutation loop without structural changes. The loop may eventually surface these via eval scores, but it can spend 3-5 experiments on bullet-level mutations that the structural fixes would obviate. Recommended only when you've already iterated structurally on the resume and trust the structure.

Wait for the user's response. Do not start experiment 0.5 or the regular loop without their answer.

---

## Output when "apply now"

If the user picks "apply now":

1. Generate the full text of each proposed structural mutation. Pull from:
   - The resume's current content (re-arrange / re-frame existing material when possible).
   - The supplementary library (`bullets/`, `interview-notes/`, `narratives/` -- loaded at Step 1.1) for missing specifics.
   - Candidate questions flagged in the changelog (do NOT invent specifics that aren't sourced).

2. Apply ALL mutations to the working copy in a single edit pass. This is the only place in the entire autowrite loop where multiple mutations bundle -- the rest of the loop maintains one-mutation-at-a-time discipline. The bundling here is deliberate: structural changes are pre-requisite to causal mutation tracking.

3. Spawn all recruiter subagents in parallel against the modified working file. Single message, multiple Agent tool calls.

4. Compute new aggregate.

5. **Keep/discard decision:**
   - Aggregate improved or held within 2 points of baseline -> **KEEP**. The post-audit working file becomes the new baseline. Continue to Step 5.
   - Aggregate dropped more than 2 points -> **REVERT**. Restore from the `.baseline` file. Announce: "Structural bundle reverted; aggregate dropped from X% to Y% (>2pt loss). The structural mutations may have removed signal the profiles were scoring on. Proceeding to regular line-level loop against original baseline. Findings preserved in changelog for future reference."

6. Log to `changelog.md` as a single experiment-0.5 entry:

   ```
   ## Experiment 0.5 -- structural audit bundle [keep / revert]

   **Aggregate:** [baseline]% -> [new]% (delta [+/-X.X])
   **Per-profile deltas:** anthropic [prev->new], openai [prev->new], ...
   **Mutations bundled:** N structural changes
     - Added executive summary (3 lines, scope/arc/lens; sourced from current role + bullets/meta-eng-lead.md)
     - Added scope metadata lines under 3 director-level role headings (team sizes pulled from interview-notes/2026-04-team-building.md)
     - Rewrote 5 bullets from IC framing to decision framing (top of current role + previous director role)
     - Added "Career Highlights" section with 4 accomplishments drawn from across career
   **Reasoning:** Director-level resumes that lack scope-first framing read as senior-IC resumes regardless of bullet quality. The structural fixes establish the right abstraction level before line-level mutation begins.
   **Result:** Which evals flipped -- which were now passing, which now failing (or were unaffected).
   **Decision:** kept (aggregate improved or held within 2pt) OR reverted (aggregate dropped >2pt).
   ```

7. Update `results.tsv` and `results.json` with the experiment-0.5 row.

8. Proceed to Step 5 (mutation loop) from the post-audit baseline.

---

## Output when "show me the text"

If the user picks "show me the text":

1. Post each proposed mutation's full text in chat as a numbered list. Each entry shows: the section affected, the current content (if any), and the proposed replacement / addition.
2. Use `AskUserQuestion` (if available) with options: `apply all`, `apply subset` (multi-select follow-up), `cancel and skip`. If unavailable, ask in plain text.
3. Honor the response. If `apply subset`, narrow the bundle to the selected mutations only, then proceed as in "apply now."

---

## Output when "skip"

If the user picks "skip":

1. Announce in chat: "Skipping structural audit. Findings logged to `changelog.md` for reference. Proceeding to mutation loop. The loop may surface these via eval scores; if you see consistent low scores on org-scope evals after 3-5 experiments, consider re-running and choosing `apply now`."
2. Append the findings list to `changelog.md` under a `## Structural audit (skipped)` heading, so the audit isn't lost.
3. Proceed to Step 5 (mutation loop) from the original baseline.

---

## Why bundle (and why this is the ONLY place we bundle)

The autowrite loop's central discipline is "one mutation at a time so the changelog stays causal." Structural changes violate this because they touch multiple parts of the document at once -- but only at the top of the run, and only because the structural baseline matters more than per-mutation attribution for these specific changes.

Bundling them into a single experiment 0.5 entry keeps the rest of the loop causal. The changelog notes "bundled structural audit" as the source of the changes. Subsequent line-level mutations continue to be tracked individually. If the bundle is later reverted, every later mutation still has a clear causal record against the post-audit (or original) baseline.

If the user wants to undo specific structural mutations later, they have the per-mutation list in the changelog and can apply them surgically to a fresh working copy. The bundle is reversible at the file level (via `.baseline`); the audit-as-individual-mutations record persists in the changelog for surgical review.

---

## Hard constraints

- **Director-only.** Do not run this audit on `director_level: false` runs. The findings would mislead an IC run.
- **No fabrication.** Structural mutations may re-frame, re-order, and add sections from existing material in the resume + supplementary library. They may NOT invent specifics (team sizes, budgets, accomplishments) that aren't sourced. Flag missing specifics as candidate questions.
- **One audit per run.** The audit runs once, in Step 4.5, between baseline and the line-level loop. It does not re-run mid-loop, even if structural drift becomes visible later.
- **Reversible.** The `.baseline` file is the revert target. If the structural bundle drops the aggregate by >2pt, the entire bundle reverts in one motion. The user does not need to manually undo anything.
- **No editorial.** The audit reports what it found and proposes mutations. It does not editorialize ("this resume is weak", "this candidate should rebrand"). State findings, propose changes, defer to the user.
