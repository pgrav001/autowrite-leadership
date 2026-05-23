# Company Profile Template

Every cached profile under `<resume-parent-dir>/profiles/<slug>.md` must follow this schema. A research subagent produces a file matching this template; a recruiter subagent reads it to know how to score. Profiles cache co-located with the resume markdown they're scored against -- not in the plugin tree.

The skeleton below is the canonical structure. Do not add sections that aren't here; do not remove sections that are. Empty a section only if no information was found -- and flag that explicitly with "Not found during research; revisit on profile refresh."

---

## skeleton

```markdown
---
name: <slug>
company: <Full Company Name>
role_qualifier: <optional, e.g., "AI Engineer / Applied Research" or "any">
director_level: <true if role qualifier signals director, VP, head of, senior manager, or chief officer; omit or false for IC/manager roles>
researched_at: <ISO 8601 date, e.g., 2026-05-12>
sources:
  - <URL or short citation>
  - <URL or short citation>
---

# <Full Company Name>

## Hiring values

Bullet list of 3-6 explicit hiring values, drawn from public sources (engineering blog, "what it's like to work here" pages, hiring manager interviews, careers page). Each bullet should be one sentence and cite the source inline if non-obvious.

## Technical bar

**IC and manager roles only** (`director_level: false` or absent). What this company screens for at the role level. 4-8 bullets. Be specific -- "ships systems end-to-end" beats "strong engineering"; "delivers commercial projects under $5M without surprises" beats "good PM." Reference any named archetypes the company is hiring for if you can find them in their materials.

## Leadership bar

**Director-level roles only** (`director_level: true`). Replace `## Technical bar` with this section. What organizational capability does this company screen for at director level? 4-8 bullets. Cover: expected team size and headcount, whether the company hires builders vs. operators vs. fixers, P&L or budget ownership expectations, whether ICs or managers report directly, functional scope. Be specific -- "leads a team of 10-20 engineers" beats "strong people manager"; "owns $5M-$10M opex budget" beats "budget experience"; "built two functions from scratch in the last 5 years" beats "builder mindset."

## Stakeholder environment

**Director-level roles only** (`director_level: true`). 3-5 bullets. Who does this role manage up to (C-suite, board, specific executive)? Does it drive cross-functional decisions without owning all the functions? What external-representation expectations exist -- customer relationships, board or investor appearances, public-facing work, partner relationships? This section gives the recruiter subagent the context to evaluate whether the resume surfaces the right scope of influence.

## Recent strategic priorities

What is this company investing in *right now* (within the last ~6 months)? Look at product launches, hiring posts, research focus areas, leadership statements. 3-6 bullets. This is the section most likely to go stale -- it's why profiles cache but support refresh.

## Anti-patterns / red flags

What this company explicitly does NOT hire for, or what gets resumes rejected. 3-6 bullets. Draw from interview reports, engineering blog "what we don't do" posts, hiring manager statements.

## Binary eval criteria

6-12 evals, each in the format defined in [eval-guide.md](eval-guide.md). Numbered. Each must be:
- Binary yes/no
- Specific enough that two recruiter subagents would agree
- Tied to this company's actual signals
- Not gameable by trivial keyword stuffing

```

EVAL 1: <Short name>
Question: <Yes/no>
Pass: <Specific>
Fail: <Specific>
Source: <Where this came from>

EVAL 2: ...

```

## Scoring notes

Any additional context for the recruiter subagent about *how* to score this company specifically. Examples:
- "This company weights research artifacts heavily; if the resume cites a paper or draft, lean toward passing the related eval."
- "This company is highly skeptical of cross-functional/managerial language for IC roles; treat such language as failing the IC-fit eval."

This section is optional. Use it only when scoring nuance can't be captured cleanly in the eval pass/fail definitions.

```

---

## validation checklist

Before saving a profile, the autowrite loop validates:

- [ ] All sections present and non-empty (or explicitly flagged as "not found")
- [ ] 6 to 12 evals (no fewer, no more)
- [ ] Every eval is binary yes/no
- [ ] At least 3-4 evals are clearly company-specific (would not appear unchanged in another company's profile)
- [ ] Each eval has a `Source:` line citing a real, current public signal
- [ ] `researched_at` date is set
- [ ] `sources` frontmatter lists at least 3 URLs or citations
- [ ] **Director-level profiles only:** `director_level: true` is set; `## Leadership bar` and `## Stakeholder environment` are present; `## Technical bar` is absent
- [ ] **Director-level profiles only:** at least 3-4 evals target organizational-scope signals (headcount, budget, cross-functional reach, people development, strategic decisions) rather than technical-output signals

If any of these fail, the loop re-spawns the research subagent with the validation feedback.

---

## example profiles (sketches only -- not real research)

Two examples in two different industries to show the same schema applies broadly.

### Example A: research-driven tech company (AI engineering hire)

```markdown
---
name: <slug>
company: <Tech Company>
role_qualifier: AI Engineer
researched_at: <YYYY-MM-DD>
sources:
  - <careers page URL>
  - <engineering blog URL>
  - <hiring manager interview citation>
---

# <Tech Company>

## Hiring values
- Treats AI safety as a first-class engineering concern, not a separate function.
- Selects for thoughtfulness about LLM limitations over raw output speed.
- Hires generalists who can pivot across infra, eval, and product work.

## Technical bar
- Ships systems end-to-end, including the messy operational parts.
- Demonstrates eval fluency -- can describe specific eval methodologies they've used.
- Familiar with research literature on the company's active areas.

## Binary eval criteria

EVAL 1: Multi-agent shipping evidence
Question: Does the resume describe a multi-agent system the candidate built and shipped in the last 18 months?
Pass: A specific system named, with deployment surface and adoption signal.
Fail: Only generic "agentic AI" framing.
Source: <citation>

(...further evals follow the same pattern)
```

### Example B: large general contractor (construction PM hire)

```markdown
---
name: <slug>
company: <Construction Firm>
role_qualifier: Senior Project Manager
researched_at: <YYYY-MM-DD>
sources:
  - <careers page URL>
  - <industry publication / project case study URL>
  - <hiring manager LinkedIn post citation>
---

# <Construction Firm>

## Hiring values
- Selects PMs who run jobs to schedule and budget without escalation drama.
- Prefers candidates with experience in the specific subtrade and jurisdiction (commercial vs. residential, union vs. open shop, target market segment).
- Hires for client-facing maturity as much as technical PM skills.

## Technical bar
- Has delivered projects at or above the firm's typical scale ($X-$Y range; team size N+).
- Holds current OSHA-30 and any state-specific certifications.
- Familiar with the schedule and cost-control tools the firm uses (Procore, Primavera, MS Project, etc.).

## Binary eval criteria

EVAL 1: Project-scale evidence
Question: Does the resume describe at least one delivered project of comparable scale (budget, duration, crew size) to the firm's typical engagement?
Pass: A specific project named with two or more quantified scale dimensions and a delivered outcome.
Fail: Only project counts or general scope descriptions without scale dimensions.
Source: <citation: firm's project portfolio page or job posting>

(...further evals follow the same pattern)
```

### Example C: director-level role at a mid-stage AI company

Note the differences from Example A (same industry, IC role): `director_level: true` is set; `## Technical bar` is replaced by `## Leadership bar`; `## Stakeholder environment` is added; the evals target organizational scope, people development, and strategic decision-making instead of technical-output evidence.

```markdown
---
name: <slug>
company: <AI Company>
role_qualifier: Director of Engineering
director_level: true
researched_at: <YYYY-MM-DD>
sources:
  - <careers page URL for the specific director posting>
  - <leadership team page URL>
  - <press / podcast about the company's current org build-out>
  - <earnings call transcript or investor letter, if public>
---

# <AI Company>

## Hiring values
- Hires director-level engineers who have built (not just inherited) a team at this scale before.
- Selects for operators who can absorb a 0-to-1 function while also running existing teams; the company is consolidating, not expanding headcount.
- Treats CEO-direct reporting as the norm at this layer -- candidates must be able to communicate strategically up and laterally without lengthy preparation.

## Leadership bar
- Has led an engineering organization of 15-40 people (the role's expected span), including both ICs and at least one EM tier.
- Demonstrates P&L or opex ownership at the $3M-$10M range (mentioned explicitly in the director JD's responsibilities section).
- Builder archetype, not operator -- the company has stated publicly that it is "building from scratch in three functional areas," and the role is one of them.
- Has navigated at least one significant organizational adversity (reorg, layoff, headcount freeze, strategic pivot) and can describe the decision made.
- Comfortable with player-coach mode: the team is small enough that the director is expected to be in code review and architectural decisions on the highest-stakes work, not just managing managers.

## Stakeholder environment
- Reports directly to the CTO; peers are 2 other engineering directors and the VP of Product.
- Owns one cross-functional initiative per quarter that requires alignment with Product, Research, and Go-to-Market.
- Expected to represent the company in 2-4 public-facing engagements per year (podcasts, conference talks, customer escalations).
- Hiring committee for this role includes a board observer for the final round -- candidates should be ready for strategic, not tactical, screening at the panel.

## Recent strategic priorities
- (3-6 bullets covering the company's current bets and what this director is being hired to solve specifically)

## Anti-patterns / red flags
- Candidates who present as "have managed managers" but cannot describe specific decisions about hiring, performance, or org design they personally made.
- Resume language that reads as senior IC ("built X system") without surfacing org-scope (team that built it, decisions to scope it that way).
- Vague leadership philosophy with no specific example of changing course on a team or strategy.
- Multiple director roles with short tenures (<18 months each) without a clear narrative for the transitions.

## Binary eval criteria

EVAL 1: Stated organizational scope
Question: Does the resume name the candidate's directly-owned headcount at director-level roles with a specific number, not "a team"?
Pass: At least one director-level role entry states team size as a specific integer (e.g., "team of 18", "4 EMs and 14 ICs").
Fail: All director-level role descriptions use vague scope language ("a team", "multiple engineers", "the engineering org").
Source: <citation: the director JD mentions "lead a 15-30 person team"; without scope in the resume, the recruiter cannot screen for match>

EVAL 2: Budget or P&L ownership evidence
Question: Does the resume name a specific budget, opex figure, or P&L scope the candidate has owned?
Pass: At least one bullet or role-metadata line states a budget or opex amount (e.g., "$4.2M opex", "owned $8M annual engineering spend").
Fail: No budget figures present anywhere; only headcount or activity descriptions.
Source: <citation: the director JD lists "manage the engineering budget" as a responsibility>

EVAL 3: People-development evidence
Question: Does the resume describe at least one person the candidate hired, promoted, or developed -- naming an outcome?
Pass: At least one bullet names a specific person-development outcome (e.g., "hired two senior engineers who became EMs within 18 months", "promoted three ICs to staff level").
Fail: People-development language is generic ("mentored team", "developed talent") without specific outcomes.
Source: <citation: the company's hiring values page references "leaders who build the leaders below them">

EVAL 4: Decision evidence (not just delivery evidence)
Question: Does at least 50% of the resume's recent-role bullets lead with a decision verb (Restructured, Hired, Sponsored, Cut, Built [the team], Decided to) rather than an IC verb (Built [a system], Shipped, Optimized)?
Pass: Sampling 8-12 bullets from the candidate's recent director-level roles, at least 50% lead with decision verbs.
Fail: Under 50% decision verbs; reads as a senior IC resume that was relabeled.
Source: <citation: hiring manager interview from podcast -- "I want to see what they decided, not what they shipped">

EVAL 5: Organizational adversity narrative
Question: Does the resume describe at least one situation of navigating significant organizational constraint (reorg, headcount freeze, layoff, strategic pivot, budget cut) and name the decision made?
Pass: At least one bullet or summary line references navigating a constraint and the decision in response.
Fail: No constraint narrative present; resume reads as if every role was steady-state growth.
Source: <citation: the company has been through a notable reorg in the last 12 months -- they will screen for this>

EVAL 6: Cross-functional reach evidence
Question: Does the resume describe at least one initiative the candidate drove across functions they did not own -- with named partner functions and a specific outcome?
Pass: At least one bullet names a cross-functional initiative with partner functions (Product, Research, GTM, Legal, Finance, etc.) and an outcome.
Fail: All bullets describe work inside the candidate's own org boundary.
Source: <citation: the JD lists "drive cross-functional initiatives across Product and Research" as a responsibility>

EVAL 7: External-value differentiator
Question: Does the resume surface at least one signal of value the candidate brings that an internally-promoted candidate at this company would not have by default?
Pass: The summary or top of the experience section names a cross-industry transition, scale shift, public artifact (talk/paper/OSS), or specific external practice the candidate brought to past roles.
Fail: Resume reads as someone whose entire career is within the same company stage and industry; no differentiator visible.
Source: <citation: the company is currently running a search committee that explicitly includes an internal candidate -- this is public via a recent board meeting summary>

EVAL 8: Title progression legibility
Question: Is the candidate's IC -> manager -> director progression (or sustained director-level breadth) readable within a 5-second scan of the resume?
Pass: Either the role headings show clear upward progression, or a Career Highlights section surfaces the arc explicitly.
Fail: Progression is buried, gaps are unexplained, or director roles read as flat lateral movement without clear narrative.
Source: <citation: standard director-search screening behavior>

EVAL 9: Strategic-lens signal in summary
Question: Does the resume open with a 3-5 line executive summary that names scope + arc + strategic lens?
Pass: Summary block exists, names a quantified scope dimension, references the candidate's functional arc, and states a strategic focus the candidate is known for.
Fail: No summary block, or summary is generic ("results-driven engineering leader") without specific dimensions.
Source: <citation: the director-search process at this company includes a 90-second resume screen before any deep review>

## Scoring notes
- The company is currently in a "consolidation, not expansion" mode publicly. Weight evals about navigating constraint (EVAL 5) and cross-functional reach (EVAL 6) above pure scope evidence (EVALS 1-2) when scoring -- a candidate with smaller-scope but constraint-navigation experience may screen better than a larger-scope candidate with only growth-mode experience.
- The board-observer-on-panel detail in the Stakeholder environment section means the recruiter should also flag whether the resume reads as polished enough for executive-level review (typography, consistency, no typos). This is a screening signal for this specific company that does not apply universally.
```

All three examples use the same schema. The differences are in the *signals* (what the company actually cares about), the *examples in each eval's Pass/Fail clauses* (specific to the role's measurable artifacts), and -- for director-level profiles -- the *section vocabulary* (Leadership bar + Stakeholder environment instead of Technical bar) and the *eval categories* (organizational scope, people development, decisions, adversity, cross-functional reach, external value). The shape -- frontmatter, sections, binary evals with sources -- is identical.

---

## maintenance

A cached profile is good for ~3 months. After that, refresh it -- companies' strategic priorities shift, especially in AI. The autowrite loop accepts a `profile_refresh: true` flag in step 1 to force re-research even when a cached file exists.
