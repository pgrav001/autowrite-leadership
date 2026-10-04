# Autowrite

A Claude Code plugin that autonomously improves a resume by spinning up company-specific recruiter subagents, scoring the resume against binary hiring evals, mutating one line at a time, and converging across multiple target companies in parallel.

Modeled on Andrej Karpathy's autoresearch methodology, adapted from skill-prompt optimization to resume content.

---

## What it does

You give autowrite a resume markdown file and a list of target companies. It runs a two-stage autonomous loop -- expect an hour or more for a full multi-company end-to-end run.

### Stage 1: Primary loop (per-company convergence)

1. For each company, it **researches** the hiring profile on first run (using WebSearch and WebFetch against careers pages, engineering blogs, public statements from leadership, recent product launches). Profiles cache to disk and reuse.
2. Each profile contains 6-12 **binary yes/no eval criteria** specific to that company's bar.
3. autowrite spawns one **recruiter subagent per company in parallel**, each scoring the resume against its profile's evals.
4. **Profile-overalignment safeguard:** if you target only one company, autowrite automatically adds a triangulation profile to detect false 100% baselines (a real failure mode from production -- a single research subagent can produce evals that align too closely with the resume's own language).
5. The mutation loop mutates **one line of the resume at a time** to address cross-profile gaps. Improvements are kept; non-improvements are reverted.
6. **Per-profile lock:** when a company's profile holds at ≥90% pass rate for 3 consecutive experiments, that variant **LOCKS**. autowrite saves it as `applications/<company>/_company-locked.md` and removes that profile from the active mutation pool. The loop continues against un-locked profiles.
7. Different companies have divergent bars -- so the output is per-company resume variants, not a single one-size-fits-all resume.

### Stage 2: Secondary loop (per-opening tailoring)

For each locked company:

1. A **job-openings investigator** subagent discovers currently-open roles at that company (discovery only -- no scoring, to avoid bias).
2. For each role: a **hiring-manager profile** subagent builds a role-scoped profile that blends company-bar evals with JD-specific evals.
3. A recruiter subagent scores the locked company variant against the hiring-manager profile.
4. If the score is below the lock threshold, a per-role sub-mutation loop runs against the hiring-manager profile until it locks or the per-role budget exhausts.
5. A **cover-letter subagent** drafts a 250-400 word role-scoped letter that echoes 2-3 of the hiring-manager-profile evals in prose, anchoring every claim to the resume or supplementary-context library (no fabrication; any missing facts surface as candidate questions in the role's `sub-changelog.md`).
6. Output per role: a `resume.md` AND a `cover-letter.md` (plus print-ready `.html` siblings), ready to submit as a job application.

### Outputs

- Working primary-loop tree with mutation history (`autowrite-<resume-slug>/`)
- Profile cache (`profiles/`)
- Applications tree (`applications/<company>/<role>/`) containing the submission set per opening: `resume.md` + `resume.html` (print-ready resume) and `cover-letter.md` + `cover-letter.html` (print-ready role-scoped cover letter) -- open each HTML file in a browser, Ctrl+P -> Save as PDF
- Company-level navigation pages (`applications/<company>/index.html`) listing every discovered opening with active-status tags, final scores, and direct links to the print-ready resume + cover letter pair
- Live HTML dashboard with a top-of-page TOC linking to per-company index pages and each locked variant's HTML; per-profile score lanes and per-company role breakdowns underneath
- Full discovery audit trail: `openings-<YYYY-MM-DD>.json` (machine-readable) and `openings-<YYYY-MM-DD>.md` (scannable) showing every role the discovery subagent considered, including roles tagged closed, excluded, or unverified -- nothing is dropped silently
- changelog.md (primary loop) and sub-changelog.md (per role, including a `## Cover letter generated` block recording which evals each letter echoed) -- causal logs of every mutation for interview prep

**The original resume is never modified.** Mutations happen on working copies. The user reviews, diffs, and manually decides whether to apply changes.

---

## What it does NOT do

- Send your resume to anyone. Everything runs locally on your machine.
- Pretend a resume passes evals it doesn't. The point is to find the gaps, not paper over them.
- Add fabricated content. If a recruiter subagent suggests a capability the resume should claim, you decide whether that's true and add it yourself -- autowrite proposes mutations, you approve them in the loop's keep/discard logic, but you remain accountable for accuracy.
- Replace human judgment. The mutation loop optimizes for what the recruiter subagents detect. Your hiring manager will see things subagents can't. Treat autowrite as a sparring partner, not an oracle.

---

## Installation

This is a Claude Code plugin. It lives at `~/.claude/plugins/autowrite/` (or `%USERPROFILE%\.claude\plugins\autowrite\` on Windows). After placing or cloning the directory there, Claude Code picks it up on next launch.

Quickest path -- clone directly into the plugin directory:

```
# macOS / Linux
git clone https://github.com/<your-username>/autowrite ~/.claude/plugins/autowrite

# Windows (PowerShell)
git clone https://github.com/<your-username>/autowrite "$env:USERPROFILE\.claude\plugins\autowrite"
```

Or install manually (if you downloaded a zip):

```
# macOS / Linux
mkdir -p ~/.claude/plugins
# place the autowrite directory inside ~/.claude/plugins/

# Windows (PowerShell)
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\plugins"
# place the autowrite directory inside %USERPROFILE%\.claude\plugins\
```

The plugin should resolve to:

```
~/.claude/plugins/autowrite/
  .claude-plugin/
    plugin.json
  README.md
  skills/
    autowrite/
      SKILL.md
      references/
        eval-guide.md
        profile-template.md
        recruiter-subagent-prompt.md
        research-subagent-prompt.md
        job-openings-subagent-prompt.md          # v0.3+
        hiring-manager-subagent-prompt.md        # v0.3+
        resume-html-template.md                  # v0.4+
        cover-letter-subagent-prompt.md          # v0.6+
        cover-letter-html-template.md            # v0.6+
    career-intake/                               # v0.5+
      SKILL.md
      references/
        session-template.md
        interview-rounds.md
        resume-ingest-template.md                # v0.6+
```

The plugin ships two skills. `autowrite` is the autonomous evaluator + mutator (and, as of v0.6, the cover-letter generator). `career-intake` is a human-in-the-loop interview skill that builds the supplementary-context library `autowrite` reads from; v0.6 added a preflight **Round 0** that parses an existing resume into seeded bullet files first, then asks whether specific roles need deeper Q&A -- so candidates whose resume is already in decent shape skip the full 5-round interview entirely. Recommended flow for a fresh resume update: run `/career-intake` first with your existing resume, decide whether Round 0's seed is enough or whether the current role needs depth, then `/autowrite` against the resulting draft.

After installing, **restart Claude Code** so the plugin registry picks up the new skill.

---

## Quickstart: your first run

The minimum-viable invocation. If you have a resume and two target companies, this is the path of least resistance.

1. **Convert your resume to markdown if it isn't already.** Either point autowrite at a `.pdf` (it converts at Step 1.0; the original is never modified) or run a one-shot conversion in a separate session. The markdown is the canonical working format the loop operates on.
2. **Pick 2-4 target companies** with a clear role qualifier (e.g., `"Anthropic / AI Engineer"`, `"Google DeepMind / Research Engineer"`). Two is the minimum that produces a meaningful aggregate; four is the comfortable maximum for a first run -- more profiles equal more wall-clock time and more API spend.
3. **Invoke the skill.** Either type `/autowrite` and answer the STOP-and-confirm prompts, or invoke in natural language:

   ```
   Run autowrite on ~/path/to/resume.md for Anthropic / AI Engineer
   and OpenAI / Member of Technical Staff.
   ```

4. **Confirm the defaults** when the skill prompts. For a first run, accept `runs_per_experiment: 1`, no budget cap, `profile_refresh: false`. Adjust later if you see specific signals; the defaults are tuned for most candidates.
5. **Watch the dashboard.** When the loop starts, autowrite opens a local HTML dashboard in your browser. The dashboard refreshes every ~10 seconds and shows per-profile scores, the current mutation under evaluation, and the keep / discard verdict per experiment. You can walk away -- the loop runs autonomously.
6. **Expect an hour or more for full multi-company convergence.** Each per-company variant locks when its profile holds at ≥90% pass rate for 3 consecutive experiments. Locked variants get saved as `applications/<company>/_company-locked.md` and the loop continues against any still-unlocked profiles.
7. **After convergence:** the skill enters the secondary loop -- discovering active openings at each locked company, building per-role hiring-manager profiles, and rendering per-opening resume + cover-letter pairs as the final submission artifacts. The full output tree lands at `applications/<company>/<role>/`.

For the first run, watch the dashboard for the first ~5 minutes to confirm the loop is healthy (profiles loaded, baselines scored, first mutation under consideration) -- then walk away. If anything looks off in those first 5 minutes (no profiles loaded, baseline scores all 0, dashboard not refreshing), interrupt and check the file-creation fallback notes in `skills/autowrite/SKILL.md`.

A first run that produces locked variants for all targets and per-opening artifacts for the active roles is the success signal. Anything less is a setup issue -- check the SKILL's "before starting: gather context" section, restart, and re-invoke.

---

## Usage

Once installed and Claude Code has been restarted, invoke the skill in any Claude Code session:

> /autowrite

or in natural language:

> Run autowrite on `~/path/to/resume.md` for <Company A>, <Company B>, and <Company C>.

(Examples: "for Anthropic, OpenAI, and Google DeepMind" for an AI-engineering search; "for Turner Construction, Skanska, and Mortenson" for a construction-management search; "for Riot, Insomniac, and Bungie" for a game-development search. The pattern is the same; only the targets change.)

The skill enters its STOP-and-confirm gate before running anything (mirrored from autoresearch). It will ask:

1. **Target resume.** Path to a `.md` or `.pdf` file. If you pass a PDF, autowrite converts it to markdown as Step 1.0 (the original PDF is never modified) and saves the converted file alongside it. Markdown is the canonical working format -- the loop operates on it from that point forward.
2. **Target companies.** One or more, optionally qualified by role (examples: `"Anthropic / AI Engineer"`, `"Turner Construction / Senior Project Manager"`, `"Riot Games / Gameplay Engineer"`). Recommend 2-4 targets for a meaningful aggregate.
3. **Runs per experiment.** Default 1. Bump to 3 if you see eval results flip between runs without resume changes.
4. **Budget cap.** Optional. Default: no cap (runs until convergence).
5. **Profile refresh.** Default: use cache if available. Set to `true` to force re-research even when a cached profile exists.

Once you confirm, the loop runs autonomously. The dashboard opens in your browser; you can walk away.

**Other modes.** Besides the autonomous loop: *manual one-shot review* (one JD, one audit, one variant draft), *per-bullet review* (walk proposed edits one at a time), and *blind comparison panel* (6-8 parallel reviewers, each a different hiring context, choose between two drafts with frontmatter stripped; one adversarial auditor checks every claim against your sources). Ask for any of them by name. See `skills/autowrite/SKILL.md`.

---

## How company research works

When autowrite encounters a target company with no cached profile (or with `profile_refresh: true`), it spawns a research subagent that:

1. Uses **WebSearch** to find the company's careers / engineering / culture pages.
2. Uses **WebFetch** to read the most signal-dense URLs: job postings, "how we hire" content, engineering blog posts from the last ~12 months, public statements from leadership.
3. Synthesizes the findings into a profile markdown file containing:
   - Hiring values
   - Technical bar
   - Recent strategic priorities (last ~6 months)
   - Anti-patterns / red flags
   - **6-12 binary eval criteria** tied to specific public signals (each with a `Source:` line citing where the criterion came from)
4. Saves the profile to `<resume-parent-dir>/profiles/<slug>.md` -- co-located with the resume, not in the plugin tree. This sidesteps harness write-restrictions on `~/.claude/` and keeps profile caches travelling with the resume they were researched against.

The skill validates each profile before using it -- all sections populated, 6-12 evals, every eval binary and clearly company-specific (no generic checks that could appear in any company's profile), every eval cited. If validation fails, the research subagent runs again with explicit feedback.

**Profiles cache for ~3 months.** After that, refresh them -- companies' strategic priorities shift, especially in fast-moving fields. You can also delete a profile file at any time to force a re-research.

You can also **hand-author profiles** if you want. Follow the schema in `skills/autowrite/references/profile-template.md` and drop the file into `<resume-parent-dir>/profiles/`. The skill will load it the same way as a researched one.

---

## How the recruiter eval works

For each profile (cached or freshly researched), autowrite spawns a **recruiter subagent in parallel** (single message, multiple Agent tool calls -- they run concurrently, not serially). Each subagent receives:

- The full resume markdown content
- The full profile markdown content

It assists a senior recruiter at the target company, scoring the resume against the profile's binary evals only -- it does not invent new criteria. It returns a structured JSON-shaped report with:

- Per-eval pass/fail and evidence
- Pass count and pass rate
- Top 3 strengths
- Top 3 gaps (prioritized by impact at this company)
- Single highest-priority revision suggestion (specific section, specific bullet, specific swap)
- Optional evaluator notes (eval ambiguity, scoring confidence, etc.)

The skill aggregates these reports, identifies the most-shared gap across profiles, and proposes the next mutation.

---

## Outputs

After a run, you'll find these artifacts in `<resume-directory>/autowrite-<resume-slug>/`:

```
autowrite-<resume-slug>/
  dashboard.html                   # live dashboard, auto-refreshes every 10s
  results.json                     # data file powering the dashboard
  results.tsv                      # tab-separated score log
  changelog.md                     # detailed mutation log with reasoning
  <chosen-name>.md                 # working revised resume (the artifact)
  <chosen-name>.md.baseline        # original resume at start-of-run
  recruiter-reports/
    experiment-000/
      <profile-slug>.md            # structured report per profile per experiment
      ...
    experiment-001/
      ...
```

And the profile cache at `<resume-parent-dir>/profiles/` grows as you target new companies. Profiles are co-located with the resume they were researched against -- not in the plugin tree.

---

## Outputs you should care about

1. **`applications/<company>/<role>/resume.html` + `cover-letter.html`** -- the per-opening submission pair. Print each to PDF and submit. The markdown sources live alongside for re-editing.
2. **`<chosen-name>.md`** -- the working revised resume that fed the locked variants. Diff against your original, decide what to keep, apply manually to your canonical resume. autowrite never overwrites your original.
3. **`changelog.md` + per-role `sub-changelog.md`** -- the research logs. Each kept mutation has a reason; each generated cover letter records which hiring-manager-profile evals it echoed and any candidate questions the subagent flagged rather than invent. When an interviewer asks "why is your resume structured this way?" you have receipts.
4. **`recruiter-reports/<final>/<profile>.md`** -- per-profile remaining gaps. These are your **interview prep talking points**. You know in advance what each target company will probe, because their recruiter subagent flagged it.
5. **`<resume-parent-dir>/profiles/<slug>.md`** -- the cached company profile. Reusable on every future run against this resume. Treat it as a living document.

---

## Customization

### Adjust convergence threshold

Edit `skills/autowrite/SKILL.md` step 5. Default is 90% aggregate for 3 consecutive experiments. Bump to 85% if you want shorter runs; bump to 92% if you want stricter convergence.

### Weight some profiles more heavily

When invoking, pass weights: `"anthropic: 2x, openai: 1x, google-deepmind: 1x"`. The aggregate becomes weighted. Useful when you have a primary target and secondary comparison points.

### Add a custom profile

Author a markdown file at `<resume-parent-dir>/profiles/<your-slug>.md` matching the schema in `skills/autowrite/references/profile-template.md`. The skill will use it without re-researching.

### Force a profile refresh

Pass `profile_refresh: true` at invocation, or delete the cached profile file. The research subagent re-runs.

---

## Maintenance cadence

autowrite is designed to be re-run periodically as the search progresses, not just once at the start. The cached profiles, locked variants, and per-opening artifacts compound when maintained on a cadence; they decay when left alone for months.

**Weekly (during active search):**

- Review the per-company directories for any opening discovered since last run -- the discovery subagent only fires when the loop runs, so new openings at locked companies need a fresh invocation to surface.
- Update `applications/<company>/<role>/` directories with submission status (submitted, recruiter screen scheduled, rejected, advanced, offer). The tracker is the source of truth for what's live.
- Archive submitted variants you've already heard back on, so the active surface stays clean.

**Monthly (during active search):**

- Re-read locked variants against the canonical resume. If you've made substantive content changes to the canonical (added a project, dropped a role), the locked variants are out of sync; either re-run autowrite to refresh them or accept that they're frozen at the prior canonical's snapshot.
- Scan the changelog for mutation patterns. If the same gap keeps surfacing across companies, it's a canonical-level signal worth addressing in the upstream resume rather than per-variant.

**Quarterly (or on a major company news event):**

- **Refresh stale profiles.** The README's default `profile_refresh: false` reuses cached profiles indefinitely; companies' priorities shift. Set `profile_refresh: true` (or delete the profile file) before re-running. Fast-moving sectors (AI, gaming, biotech, fintech) need quarterly refresh; slow-moving sectors (utilities, government, established trades) can hold profiles 6 months.
- **Re-research on major news.** If a target company has a CEO change, a reorg, a layoff wave, a new product launch, or a significant funding event, the profile is stale regardless of cache age -- force refresh.

**End-of-search (after accepting an offer or pausing the search):**

- Archive the working tree (`autowrite-<slug>/`) to a dated archive directory so it doesn't get re-baselined on the next search cycle.
- Keep the cached profiles and per-company `_company-locked.md` files -- they're valuable substrate for the next search and refresh fast.
- Capture lessons in the changelog: which mutations correlated with advance-to-loop outcomes, which were performative (improved scores but didn't change interview signal), which patterns are worth preserving across searches.

## Outcome capture

The autonomous loop optimizes against profile evals; it has no visibility into what actually happened when the resume was submitted. The highest-value signal in an active search is whether the submitted variant got an advance, a rejection, or silence -- and connecting that signal back to the loop is on the candidate.

After each application outcome lands, append a row to `applications/<company>/results-outcomes.tsv`:

```
role	submitted_variant	submission_date	outcome	outcome_stage	outcome_date	notes
director-of-eng	applications/<company>/director-of-eng/resume.md	2026-06-10	rejected	recruiter	2026-06-21	no feedback
staff-engineer	applications/<company>/staff-engineer/resume.md	2026-06-12	advanced	hm	2026-06-19	HM cited the AI infra bullet
```

On subsequent autowrite runs for the same company:

- Roles with `outcome: rejected` at the recruiter stage signal the variant's hiring-manager profile is too generous; the next sub-mutation pass should re-weight the evals the role specifically screened against.
- Roles with `outcome: advanced` at the hiring-manager stage signal the variant is at the right shape; no further mutation needed for similar future roles at the same company.
- Roles with `outcome: ghosted` (no response after the company's typical response window) should be treated as soft rejections for the purposes of mutation weighting.

The loop doesn't auto-read this file (yet); the operator reads it before re-invoking and adjusts the `core_through_line` or surfaces patterns to the parent skill as a manual note. Future versions of the skill may read `results-outcomes.tsv` directly to bias the mutation engine on re-runs.

## Limitations

- **Subagent variance.** Recruiter subagents are deterministic enough for single runs in most cases, but agentic eval scoring carries some noise. If you see implausible result flips without resume changes, bump `runs per experiment` to 3.
- **Research recency.** Profiles capture what was public at `researched_at`. Fast-moving fields (tech, AI, gaming, fashion, biotech) shift priorities quickly; slower fields (utilities, government, established trades) stay stable longer. Refresh quarterly for the former, semi-annually for the latter, or after major company news.
- **Single-document scope for scoring.** autowrite scores one resume against profiles. As of v0.6 it also generates a role-scoped cover letter per opening at the end of the secondary loop, but cover letters are an output artifact -- they are not scored or mutated against their own profile set. LinkedIn summaries and portfolio sites still aren't supported as scored targets, though the architecture would extend cleanly.
- **No verification.** A recruiter subagent scores a claim as it appears in the resume. If your resume says "shipped X" and you didn't, the subagent has no way to know. autowrite optimizes the document's pass rate; you remain accountable for accuracy.
- **English-only profiles right now.** The research subagent prompts in English and looks for English-language sources. Multilingual research is a future extension.

---

## Pattern lineage

autowrite is a direct adaptation of the autoresearch skill structure (binary evals, STOP-and-confirm gate, baseline-before-mutation, one-mutation-at-a-time, keep/discard logic, autonomous loop, never-stop posture, dashboard pattern, changelog format). The novel piece is the multi-subagent orchestration -- spawning one recruiter per profile in parallel, aggregating across profiles, mutating against the cross-profile gap.

If autoresearch optimizes a skill prompt against fixed evals, autowrite optimizes a resume against *researched, company-specific* evals across multiple targets simultaneously. Same loop, different artifact, broader eval surface.

---

## Related

`autowrite` covers the artifact-generation side of the search workflow -- autonomous resume mutation, per-opening tailoring, cover-letter generation. A separate skill called `career-database` covers the upstream substrate layer: a Claim-to-Proof markdown database of strengths, evidence, stories, voice patterns, and operational session-continuity files that resumes are generated *from* rather than maintained alongside.

The two skills are independent (no code-level dependencies) but compose cleanly. A typical end-to-end search workflow looks like:

1. **`career-database`** -- build or maintain the substrate (the durable career facts; interview prep substrate; voice file; canonical artifact).
2. **`career-intake`** (this plugin) -- optional bridge: if the candidate's substrate is in the resume but not yet in the supplementary library autowrite reads, run career-intake first to itemize the resume into seeded `bullets/` + `voice.md`.
3. **`autowrite`** (this plugin) -- run against the canonical artifact + target companies. Per-opening variants land in `applications/<company>/<role>/`.
4. **Per-opening tailoring + interview prep** -- back to `career-database` for the per-opening review, post-application feedback capture, and pre-interview prep cycle against the substrate.

If you want the substrate layer: see `career-database`. If you want autonomous resume tailoring across multiple targets: stay here. Either or both, in any order.

## License

MIT. Use it, fork it, send improvements.
