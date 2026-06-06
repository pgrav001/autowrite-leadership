# Cover-letter subagent prompt template

Use this template when spawning a cover-letter subagent (via the Agent tool) to produce a role-scoped cover letter alongside an opening-tailored resume. The letter is bound to one specific job opening -- one company, one role title, one hiring-manager profile, one resume variant. There is no generic "company-level" cover letter; a JD-less letter has nothing concrete to echo and would invert the discipline autowrite enforces everywhere else.

The output is a markdown letter body (250-400 words) that the parent skill saves to `applications/<company-slug>/<role-slug>/cover-letter.md` and then renders to `cover-letter.html` via `cover-letter-html-template.md`.

---

## Invocation pattern

Tool: `Agent`
`subagent_type`: `Explore` (read-only synthesis -- all source content is passed inline; no WebFetch or WebSearch required)
`description`: `Draft cover letter for <Company> / <Role>`
`run_in_background`: `false` (parent saves the output and proceeds to the next role in the batch)

Spawn one cover-letter subagent per role in parallel within each Step 6 batch, the same batch shape used for the hiring-manager and recruiter subagents at 6b/6c/6d. The letter is a per-role artifact (peer to `resume.html`); it has no company-level analog.

`prompt`:

```
You are drafting a cover letter for one specific job opening. The letter ships alongside a role-tailored resume. The hiring-manager profile already extracted what this role's evaluator cares about; your job is to write a letter that echoes 2-3 of those load-bearing signals in prose, using only facts present in the resume or the supplementary-context library passed below.

**Target company:** [COMPANY NAME]
**Role title:** [ROLE TITLE]
**JD URL:** [URL]
**Team (if named in JD):** [TEAM NAME or "not named"]
**Today's date:** [YYYY-MM-DD]

**Hiring-manager profile (full markdown -- the role-scoped evals to draw from):**

<<<HIRING-MANAGER PROFILE BEGIN>>>
[HIRING-MANAGER PROFILE MARKDOWN PASTED INLINE]
<<<HIRING-MANAGER PROFILE END>>>

**Job description (full markdown extract from discovery):**

<<<JD BEGIN>>>
[JD MARKDOWN PASTED INLINE]
<<<JD END>>>

**Resume variant for this role (markdown -- the source of every claim you may make about the candidate):**

<<<RESUME BEGIN>>>
[RESUME MARKDOWN PASTED INLINE]
<<<RESUME END>>>

**Supplementary-context library (every markdown file the parent loaded from `bullets/`, `interview-notes/`, `narratives/`, `context/` -- additional factual phrasings you may quote or paraphrase from):**

<<<LIBRARY BEGIN>>>
[LIBRARY FILES CONCATENATED INLINE, each prefixed with its relative path on a separator line like `--- bullets/meta-ise.md ---`]
<<<LIBRARY END>>>

**Candidate review flag (only set when the resume variant was marked for manual review by the secondary-loop sub-mutation budget exhaustion):** [yes | no]

**Director-level flag (set from the hiring-manager profile's `director_level` frontmatter):** [yes | no]

**Core through-line(s) (optional; passed from the parent skill's invocation):** [list one or two strings, or empty]

If one or more through-line strings are passed:

- **You MUST weave each through-line into the opening paragraph** (verbatim or close-paraphrase that preserves the candidate's phrasing). For a single through-line, the opening paragraph should anchor the candidate's positioning on it before pivoting to the role-specific evidence. For paired through-lines, both must appear in the opening or the first body paragraph -- never collapse them into one.
- **Subsequent paragraphs do the eval-echo work** as described in the letter shape below.
- **Do NOT paraphrase the through-line into your own framing.** The candidate has authored the through-line as a specific positioning claim. Rewording it -- even to what sounds like a tighter version -- strips the candidate's voice from the sentence the entire letter is anchored on. The exact wording the candidate passed is the wording that goes in the letter.
- **Through-line + JD coherence:** if the through-line and the JD-specific evals are in tension (e.g., the through-line frames the candidate as an infrastructure builder but the JD is for a product-management role), the letter should acknowledge the through-line first, then explicitly bridge to the role's responsibilities ("...and the bridge to <JD framing> is..."). Do not drop the through-line in favor of JD fit; the user passed it as a non-negotiable.

If no through-line is passed, proceed with the standard opening shape below.

## Your output

Return the cover letter body as markdown. Return ONLY the markdown -- no preamble, no closing notes, no rationale paragraph. The parent skill will save it.

If `Candidate review flag` is `yes`, prepend a `## Manual review needed` heading with one sentence noting the resume was budget-exhausted on this role and the letter should be reviewed for any structural mismatch before submission. Then continue with the normal letter structure below.

## Letter shape -- IC and manager roles (Director-level flag = no)

```
[Date in "Month DD, YYYY" form]

Dear <salutation>,

<Opening paragraph -- 2-3 sentences. Lead with the role title and one anchor claim from the resume that maps to the strongest hiring-manager-profile eval. No fluff openers ("I am writing to apply...", "I was excited to see..."). The opening sentence should be the one a hiring manager skimming would remember.>

<Body paragraph 1 -- 3-5 sentences. Echo a load-bearing hiring-manager-profile eval. Cite specific resume evidence (project name, shipped artifact, quantified outcome) drawn verbatim from the resume or library. No restating the resume -- pick one or two claims and add the context the bullet could not fit.>

<Body paragraph 2 -- 3-5 sentences. Echo a second hiring-manager-profile eval, ideally one that pairs with the JD's stated responsibilities (not just company-level inherited evals). Same evidence rule. If the JD names a specific initiative or product the candidate's background touches, name it.>

<Optional body paragraph 3 -- 2-3 sentences. Used only if a third hiring-manager-profile eval is too load-bearing to omit and not covered above. Otherwise skip this paragraph.>

<Closing paragraph -- 2 sentences. State availability for next steps. No "I look forward to your response" filler. One concrete sentence about what comes next.>

Sincerely,
[Candidate Name -- pulled from the resume's top-level heading]
```

## Letter shape -- Director-level roles (Director-level flag = yes)

Director-level cover letters do different work than IC cover letters. They are read by hiring executives or search committees who already know the candidate has the technical chops on paper -- otherwise the resume wouldn't have surfaced. The letter's job is to demonstrate executive presence, strategic positioning, and organizational judgment in prose. The artifact-anchored opening that works for IC letters reads as junior at director level.

```
[Date in "Month DD, YYYY" form]

Dear <salutation>,

<Opening paragraph -- 2-3 sentences. Lead with strategic positioning, not an artifact. The opening should answer "what is the organizational problem this candidate solves, and why is it relevant to this specific role at this specific moment?" Reference the company's current strategic situation as the anchor (drawn from the hiring-manager profile's `## Stakeholder environment` or `## Recent strategic priorities` sections, or from the JD's framing of the role), then state what the candidate brings to that situation. No fluff openers, no artifact-first framing ("I built X"), no aspirational language ("I have always wanted to..."). The opening sentence is the one a hiring executive skimming would remember -- and it should sound like a peer speaking to a peer, not a candidate selling themselves.>

<Body paragraph 1 -- 4-6 sentences. Echo a load-bearing org-scope hiring-manager-profile eval (team size, budget, cross-functional reach, organizational adversity, or strategic decision-making). Cite specific resume evidence: not what the candidate built, but what the candidate decided, restructured, hired, or sponsored, and what changed because of that decision. Quantify scope (headcount, budget, scope of decision) wherever the resume or library supports it. This paragraph is where the candidate proves the resume's scope claims are real and operationally lived, not just numbers.>

<Body paragraph 2 -- 4-6 sentences. Echo a second hiring-manager-profile eval, ideally one that pairs with the JD's stated responsibilities. If the JD describes a specific organizational situation ("scale the team from 12 to 30 in 18 months", "consolidate three teams into one platform org", "build out a new function"), this is the paragraph where the candidate names the closest analog from their own history. Same evidence rule -- specific decisions, specific outcomes, no fabrication.>

<Optional body paragraph 3 -- 3-4 sentences. Used when an external-value differentiator is a load-bearing signal for this role (cross-industry transition, specific external practice, public artifact, scale shift). Most director-level letters benefit from including this paragraph; skip only when the previous two paragraphs have already saturated the strongest signals.>

<Closing paragraph -- 2-3 sentences. State availability and one concrete sentence about what comes next, framed as a peer-to-peer next step rather than a candidate-to-employer one ("Happy to talk through any of the specifics above when we connect" or "I can be reached at [contact] for next steps in the search process"). No "I look forward to hearing from you" filler. No "Thank you for your consideration" -- that framing collapses peer-to-peer voicing into supplicant-voicing.>

Sincerely,
[Candidate Name -- pulled from the resume's top-level heading]
```

The director-level letter's voice is the most important variable. Read each draft sentence and ask: "Could this sentence appear in a peer's note to the hiring executive?" If the answer is no -- if it reads as a candidate addressing a recruiter -- rewrite at peer voice. Executive presence on paper is the highest-leverage signal a cover letter can carry at this level.

## Salutation rules

- If `Team (if named in JD)` is set to a real team name, write `Dear <Team Name> team,` (e.g., `Dear Applied AI team,`).
- Otherwise, write `Dear Hiring Manager,`.
- Never invent a specific hiring-manager name. The discovery layer does not surface names, and inventing one is fabrication. If the candidate later learns the hiring manager's name, they can replace the salutation line manually.

## Word count

- **IC and manager roles (Director-level flag = no): 250-400 words** (excluding the date line, salutation, and signature). Letters under 200 words read as low-effort; letters over 450 words read as filler-heavy.
- **Director-level roles (Director-level flag = yes): 350-500 words.** Director letters carry more strategic context, more org-scope evidence, and frequently a third paragraph for the external-value differentiator. Letters under 300 words at director level read as undercooked; letters over 550 words read as belabored. The mid-range (380-450) is the sweet spot for most director letters.
- Count words once before returning; if outside the range, tighten or expand by trimming or restoring one body sentence at a time, not by adding new claims.

## Hard constraints (the no-fabrication rule)

- **Every factual claim must trace to the resume or to a file in the supplementary library.** If you want to write "led the migration to Rust" but the resume says "contributed to the Rust migration" and the library doesn't go further, write "contributed to" -- not "led."
- **If a hiring-manager-profile eval would be best addressed by a claim the resume and library do not support, do not invent the claim.** Instead, append a line at the end of your response, after the signature, under a `## Candidate questions flagged` heading -- one bullet per missing fact, in the form `[ ] <fact in question form>`. The parent skill will read these from the saved file and log them in `sub-changelog.md`. The letter itself stays clean.
- **Use the candidate's own framing.** If the resume says "audience shift from humans to agents," the letter can say "audience shift" or "shift from humans to agents" -- it should not paraphrase to "audience adaptation" or "behavioral change."
- **No quantification you didn't see in the source.** If the resume says "shipped 14 products," the letter can echo "14 products"; it cannot upgrade to "over a dozen" or downgrade to "several."
- **No causality upgrades.** If the resume or library frames a relationship as concurrence ("the team's work landed in the same window as the company's broader AI investment surge"), the letter cannot upgrade to causation ("our team drove the company's AI shift"). Concurrence and causation are different claims; the candidate has already authored the framing they're comfortable defending in interviews. Preserve the verb. Common shape of the error: the resume says "our work coincided with X" / "landed in the same window as X" / "alongside X"; the letter writes "drove X" / "caused X" / "led the shift to X." If the resume's framing is restrained, the letter's framing must match. The candidate's restraint is itself the credibility signal -- collapsing it into a stronger claim damages credibility on the interview side even if it scores higher on a recruiter scan.
- **No voice-file phrasings paraphrased.** If a `voice.md` file exists in the supplementary library and authorizes specific verbatim phrasings for a context type (e.g., "use verbatim in: cover letters for AI-company openings"), use the phrasing verbatim. Do not "improve" or shorten it. Voice-file phrasings have been pre-vetted by the candidate as defensible in interviews; paraphrases lose that vetting.

## Tone

- Direct, specific, concrete. The letter does work, not theater.
- No fluff openers. No emojis. No exclamation points (an exclamation point in a cover letter is an error).
- No corporate cliches ("results-driven", "passionate about", "deep expertise", "rockstar", "team player").
- No "I am uniquely positioned" framing. No "I have always been fascinated by X" backstory unless the resume itself documents it.
- One anchor claim per paragraph, supported by one piece of resume or library evidence. The structure does the persuading; the language does not have to.
- Address the hiring manager as a peer who will skim. Not a sales target. Not a god.

## When the hiring-manager profile has a "## Scoring notes" or "## Profile limitations" section

- **Scoring notes:** treat as load-bearing. If the profile says "weight production-engineering evidence over research artifacts," the letter's evidence should be production-engineering examples, not research-paper-style framing.
- **Profile limitations:** when the JD was thin and the profile relies mostly on inherited company evals, the letter still gets written but leans on the candidate's strongest resume bullets that align with the company-level bar. Do not invent role-specific framing the profile couldn't extract.

## Examples of opening sentences that work (for shape -- do not copy)

**IC and manager (artifact-anchored openings):**

- "The Applied AI Engineer posting describes evaluation-harness work as the load-bearing piece -- the Agentic Forensics research and the A/B/C harness across Claude Code, OpenCode, and Codex sit exactly on that surface."
- "Your engineering team is hiring for someone who can ship multi-agent reasoning systems end-to-end; the five-stage reasoning framework I deployed across four agent harnesses at <current role> is the closest analog I can offer."
- "The role calls for a Rust engineer who has built production agent infrastructure -- the orchestration server I shipped (Rust, SQL-backed job queue, concurrent multi-agent execution) was built to that brief in a different setting."

**Director-level (strategic-positioning openings):**

- "The Director of Engineering search at <Company> is, from the public material around it, a consolidation play -- three formerly-independent teams folding into one platform org under a single director. The closest parallel in my own history is the 18-month consolidation I led at <prior company>, which is where the operating muscle for this kind of work lived for me."
- "<Company>'s recent reorganization put the Director of AI Platform role directly under the CTO with a mandate to ship the first revenue-bearing product on the new stack within nine months -- the operational pattern (small senior team, executive-direct cadence, hard external deadline) matches the one I ran at <prior company> when we shipped <product> on a similar timetable."
- "The Head of Product Engineering opening at <Company> is unusually scope-defined for a public posting -- 22 engineers, $5M opex, two-quarter delivery commitment on the new agent platform. I have run a team of comparable size and budget through a similarly constrained delivery before, and the muscle that mattered most was the one for cutting scope deliberately rather than negotiating extensions."

(These are shape illustrations only. The actual opening must use phrasing drawn from the candidate's resume + library, not these example shapes verbatim. Note that director-level openings name a specific organizational situation at the target company before claiming relevance -- this is the strategic-positioning move that distinguishes a director letter from an IC one.)
```

---

## How this fits into the secondary loop

Step 6.5 runs **after** Step 6e (per-role artifacts) and **before** Step 6f (per-role results-openings.tsv row), inside each Step 6 batch:

1. 6a discovers roles for one locked company.
2. 6b builds a hiring-manager profile per role (batch).
3. 6c scores the locked variant against each hiring-manager profile (batch).
4. 6d runs a sub-mutation loop on roles below threshold (batch).
5. 6e renders `resume.md` -> `resume.html` per role (batch).
6. **6.5 spawns a cover-letter subagent per role in the same batch shape, saves `cover-letter.md`, renders `cover-letter.html`, appends a note to `sub-changelog.md`.**
7. 6f writes the `results-openings.tsv` row including the `cover_letter` column.
8. 6g (after all batches in a company complete) renders the company-level `index.html` with a Cover letter column linking to each `cover-letter.html`.

The cover-letter subagent reads the resume variant produced at 6d/6e, not the `_company-locked.md` -- the per-role resume is the source of truth for what the candidate is claiming for THIS role.

## When a role is flagged for manual review

Roles that exhaust the per-role sub-mutation budget without locking are tagged `flag_for_review` in `results-openings.tsv`. Step 6.5 still generates a cover letter for these roles -- the letter is still a useful artifact even when the resume is flagged -- but:

- The `Candidate review flag` field in the subagent prompt is set to `yes`.
- The subagent prepends a `## Manual review needed` heading to the markdown output.
- The `results-openings.tsv` row's `cover_letter` column is set to `flagged` instead of `yes`.

This keeps the artifact discoverable without falsely advertising it as submission-ready.

## When a role's resume.md does not exist

If 6e was skipped (closed role, excluded role, structural disqualifier that prevented sub-mutation from running at all), Step 6.5 is also skipped for that role. The `cover_letter` column gets `skipped` and the company index's Cover letter column shows `-`.
