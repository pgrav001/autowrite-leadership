# Blind comparison panel mode

A multi-reviewer comparison of two structurally different drafts of the same artifact. Use it when two versions exist and the choice between them is not obvious: a canonical rewrite, a major variant, a restructured summary, or two LinkedIn About drafts. It isn't worth running for line edits; per-bullet review covers those.

The recruiter evals score one resume against one bar. This mode asks a different question: **of two drafts, which one should be the base, and what is each reviewer's context telling you that your own priorities missed?** Its most valuable output is usually not the verdict but the accuracy findings and the shape of the disagreement.

## Setup

1. **Stage the drafts blind.** Copy them to `DRAFT-A.md` and `DRAFT-B.md` with **all frontmatter, changelogs, and version markers stripped**, and randomize which is which. Reviewers who can see which draft is newer, or why changes were made, ratify the reasoning instead of judging the document.
2. **Write one shared brief** that every reviewer receives:
   - Who the candidate is, in three or four sentences.
   - The narratives the candidate says must land (from the substrate's identity / strengths files).
   - The page or length limit.
   - The fixed output format below.
3. **Pick 6-8 lenses, each a different hiring context, not a different mood.** A workable default set:
   - Hiring manager in the candidate's primary target lane
   - Hiring manager in an adjacent lane (for example, the IC-track version of a management role)
   - In-house technical recruiter doing the first screen
   - Senior leader in a partner function the role works across
   - Practitioner peer who does the same work at the same level
   - **Adversarial auditor with read access to the candidate's source materials** (the substrate, review archive, evidence files)
   - Non-specialist skip-level executive
   - Narrative editor focused on structure and flow
4. **Give exactly one reviewer source access**: the auditor. That reviewer checks every claim against the record and tends to produce the highest-value findings by a wide margin. The others stay honest cold readers; give them source access and they stop reading the way a real reviewer reads.
5. **Run all reviewers in parallel** as independent subagents. They don't see each other's output.

## Reviewer output format (fixed)

Every reviewer returns:

1. **Verdict: A or B.** One letter. "Both have merits" is a failed review. If the reviewer would graft part of the losing draft onto the winner, they name the base first, then the graft.
2. **Top three reasons** for the verdict, each citing a line.
3. **Accuracy and overclaim findings.** Any line that asserts more than it can support, contradicts another line, or would not survive the obvious follow-up question. (The auditor's version cites the source.)
4. **Additions, each naming its displacement.** At a page limit, every proposed addition must say which existing line it replaces or shrinks. Without this rule, a findings list silently becomes an addition list and the artifact bloats.
5. **Cuts**, with the reason.
6. **Where the candidate's stated priorities are wrong for this reviewer's context.** Ask this explicitly. Reviewers who disagree with each other here are often mapping a real difference between lanes, not a difference in taste.

## Synthesis

The parent session synthesizes; reviewers don't.

- **Accuracy findings outrank the verdict.** Fix confirmed accuracy defects in whichever draft ships today, even before the structural choice is made. If the defect is in a file that's already been sent, say so plainly.
- **Report the tally and where it splits.** "6-2 for B" matters less than *which* lenses chose A. A split that falls cleanly along a lane boundary (IC lane versus everyone else) is a positioning finding: the right base may differ by target.
- **Count convergence.** A cut or fix named independently by three or more reviewers is strong signal. A finding from one reviewer is a candidate, not a mandate.
- **Check proposed cuts against the candidate's stated purpose.** A reviewer can be right by their own criterion and wrong against what the candidate says the work is for. Flag those cuts rather than applying them.
- **Report only.** Present the verdict, the split, the accuracy findings, and the convergent changes. The candidate decides the base and which changes to apply. A common good outcome is a third draft: one draft's structure with the other's evidence density.

## Output

Save under the working directory as `panel-<YYYY-MM-DD>/`:

- `DRAFT-A.md`, `DRAFT-B.md`, `brief.md` (exactly what reviewers saw)
- `reviewer-<lens-slug>.md` per reviewer
- `synthesis.md`: tally, split analysis, accuracy findings ranked by severity, convergent changes, flagged cuts, and the open decisions for the candidate
