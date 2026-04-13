# Stage 5 - Adversarial Quality Control
# Read this file and execute it exactly. Do not skip steps.
# This stage is an adversarial loop. Assume the documentation is wrong until proven otherwise.
# Pipeline reference: lore/LORE.md

---

# Purpose

This stage runs an adversarial quality control loop over every document
produced by stage 4. It scores output against a binary rubric defined below.
It makes inline corrections to output documents and records all findings in
docs/.LORE/work/REVIEW_NOTES.md.

Exit condition: all rubric checks pass, OR 3 loop iterations have been completed,
whichever comes first.

## When to run this stage

Stage 5 should be run BEFORE the human reviews the output documents, not after.
Running it after human corrections have been applied inflates the score and
prevents the pipeline from catching errors that the human had to find manually.

The intended sequence is:
    1. Stage 4 completes and outputs its handoff note.
    2. Stage 5 runs immediately (this stage) - this is the baseline QA pass.
    3. The human reviews the output documents and makes any corrections.
    4. If corrections were made, stage 5 may be re-run to verify them (optional).

If stage 5 is run after human corrections: note this in REVIEW_NOTES.md with
the line "Note: this pass ran after manual corrections were applied. Score
reflects post-correction state, not stage 4 output quality." This preserves
an honest record of what the pipeline caught versus what the human caught.

Confidence scale used throughout this stage:
	0 = no confidence (0%)
	1 = low confidence (~25%)
	2 = moderate confidence (~50%)
	3 = high confidence (~75%)
	4 = certain (100%)

---

# Prerequisites

Before executing this stage:

1. Read lore/SCHEMAS.md in full. You must know the exact schema for
   REVIEW_NOTES.md before writing anything.

2. Read docs/.LORE/work/JOURNAL.md. Verify:
   - status is IN_PROGRESS
   - last_stage_completed is stage_4_lore.md
   If the prerequisite fails, output:
       "Stage 5 prerequisite failed: expected last_stage_completed to be
        stage_4_lore.md. Found: [value]. Check the pipeline state."
   Stop.

3. Read docs/.LORE/work/ANALYSIS.md. This is your ground truth for factual
   verification. Every claim in the output documents will be checked
   against this file.

4. Read docs/.LORE/work/SKELETON.md. This is your secondary ground truth
   for structural and identity claims.

5. Read docs/.LORE/work/QUESTIONS.md. Note which Tier 1 questions were answered
   and what the answers were. These feed into accuracy checks.

6. Read docs/LORE_ABSTRACT.md.
7. Read docs/LORE_ENCHIRIDION.md.
8. Read docs/LORE_INDEX.md.
9. Read each docs/LORE_*_PRIMER.md file listed in JOURNAL.md artifacts.

   If any expected document is MISSING from docs/:
       Record it as a BLOCKING defect immediately.
       Set overall_score to 0 of [total checks] for that document's checks.
       Continue checking all other documents.

10. Check whether docs/.LORE/work/REVIEW_NOTES.md already exists.
    If it exists: read it. Note the qa_pass_number. This pass will be
    qa_pass_number + 1. Note the defects from the prior pass - they must
    all be re-checked in this pass.
    If it does not exist: this is pass 1.

---

# Step 1 - Establish Pass Number and Prior State

State explicitly before any checking:

	This is QA pass [N] of a maximum of 3.

	[If pass > 1:]
	Prior pass found [count] defects. [count] were BLOCKING or MAJOR.
	Defects carried forward from prior pass:
	[list each defect by number and one-line description]
	These must be verified as corrected or re-recorded as still present.

	[If pass 1:]
	No prior pass. Running full rubric check from scratch.

---

# Step 2 - Run the Rubric

Run every check in the rubric below against every applicable document.
The rubric is binary: each check is PASS, FAIL, or PARTIAL.

PARTIAL is used only when a check applies to multiple sections and some
pass while others fail. Record which sections passed and which failed.

For each check, record:
	- Check ID (from the table below)
	- Result: PASS | FAIL | PARTIAL
	- Confidence: 0 to 4
	- Notes: brief description of what was found

After running all checks, compute the overall score:
	total checks = count of all checks run
	passed = count of PASS results + (0.5 * count of PARTIAL results, rounded down)
	overall_score = "[passed] of [total] checks passed"

---

# Rubric

## Category A - Structural Completeness

	A1  Every canonical output document listed in JOURNAL.md artifacts exists on disk.
	    FAIL condition: any document is MISSING.

	A2  docs/LORE_ABSTRACT.md contains all required sections from the template.
	    Sections required: project identity, what it does, architecture overview,
	    data layer summary, infrastructure summary, setup instructions, key dependencies.
	    FAIL condition: any required section is absent or empty.

	A3  docs/LORE_ENCHIRIDION.md contains all required sections from the template.
	    Sections required: subsystem catalog (one entry per subsystem in ANALYSIS.md),
	    key files directory, data paradigm, IT paradigm, code style reference,
	    bugs and risks, entry points, dependencies.
	    FAIL condition: any required section is absent or empty, or any subsystem
	    from ANALYSIS.md is missing from the catalog.

	A4  docs/LORE_INDEX.md lists every document in the docs/ directory with a
	    one-line description.
	    FAIL condition: any docs/ LORE file is not listed, or any entry has no description.

	A5  Every LORE_PRIMER_[name].md file expected by JOURNAL.md exists.
	    FAIL condition: any expected PRIMER is missing.
	    FAIL condition: any PRIMER file uses the old LORE_[name]_PRIMER.md
	    naming pattern instead of LORE_PRIMER_[name].md.
	    SKIP condition (mark as N/A): no PRIMERs were warranted.

	A6  Every PRIMER contains all required sections from the PRIMER template.
	    Sections required: subsystem identity, purpose, key files, key functions
	    or exports, external dependencies, integration points, configuration
	    reference, usage notes.
	    FAIL condition: any required section is absent or empty in any PRIMER.
	    SKIP condition: no PRIMERs were warranted.

	A7  Every LORE_GUIDE_[name].md file expected by JOURNAL.md exists.
	    FAIL condition: any expected GUIDE is missing.
	    FAIL condition: any GUIDE file uses a non-standard naming pattern.
	    SKIP condition (mark as N/A): no admin executable or operator-facing
	    subsystems were identified (operator_facing: NO and dev_entry_points: none
	    for all subsystems).

	A8  Every GUIDE contains all required sections from the GUIDE template.
	    Sections required: what this subsystem does, prerequisites, executables
	    and commands, configuration reference, known issues.
	    FAIL condition: any required section is absent or empty in any GUIDE.
	    SKIP condition: no admin executable or operator-facing subsystems
	    were identified.

## Category B - Source Integrity

	B1  Every factual claim in LORE_ENCHIRIDION.md and LORE_ABSTRACT.md is tagged
	    [src: filename] or [inferred].
	    FAIL condition: any untagged factual claim in either document.
	    Confidence note: spot-check at least 10 claims per section, or all claims
	    if a section has fewer than 10.

	B2  Every [src: filename] tag in any output document refers to a file that
	    actually exists in the project (was listed in SKELETON.md or read in ANALYSIS.md).
	    FAIL condition: any src tag names a file not in the project.

	B3  No factual claim in any output document contradicts a statement in
	    ANALYSIS.md or SKELETON.md.
	    FAIL condition: any claim that directly conflicts with ground truth.
	    Confidence note: this check is subjective. Record your confidence level.
	    A confidence of 2 or lower makes the check result advisory only.

	    Structured sub-checklist for B3 - verify each of the following explicitly:

	    B3a  Config file roles: for each config file mentioned in the documentation,
	         does its documented purpose match what was observed in the file's
	         header comment or structure? A file documented as holding secrets
	         must not actually hold only safe values, and vice versa.
	         [src: the actual config file, not just ANALYSIS.md]

	    B3b  Port numbers: do documented port numbers match what was observed
	         in entry point files, Dockerfile EXPOSE directives, or server
	         configuration files?
	         [src: entry point files, Dockerfile]

	    B3c  Database names: do documented database or collection names match
	         what was observed in the database configuration file (dbConfig.js
	         or equivalent)?
	         [src: dbConfig or equivalent]

	    B3d  External service URLs and providers: for each external service named
	         in the documentation, is that service actually referenced in the
	         code? If a human answer named a provider that does not appear in
	         any [src: filename] tag, flag it as unverified.
	         [src: API call sites, Dockerfile, config files]

	    B3e  Environment file paths: do documented file paths (config files,
	         data files, certificates) match what was observed in Dockerfile,
	         docker-compose, or config files?
	         [src: Dockerfile, docker-compose, config files]

	    Record each sub-check as PASS, FAIL, or N/A in the B3 row notes.
	    If any sub-check is FAIL, B3 overall is FAIL.
	    If all sub-checks are PASS or N/A, B3 overall is PASS.

	B6  No PRIMER file documents a file that was not read with policy FULL.
	    For each LORE_PRIMER_[name].md, read the file inventory section.
	    For each file listed in the inventory with function-level detail,
	    verify that the file appears in the corresponding subsystem's
	    file_inventory in ANALYSIS.md with a FULL read entry (not HEAD,
	    not absent).
	    FAIL condition: any file documented with functions or code detail
	    in a PRIMER that does not have a FULL read entry in ANALYSIS.md.
	    Severity: BLOCKING - documentation based on unread files is fabricated.
	    SKIP condition: no PRIMERs were produced.

	B4  Unanswered Tier 1 questions from QUESTIONS.md are marked with
	    [AWAITING ANSWER - Q[number]: ...] in the output, not silently omitted
	    and not fabricated.
	    FAIL condition: a Tier 1 unanswered question produced neither a marker
	    nor a human-provided answer in the relevant section.
	    SKIP condition: all Tier 1 questions were answered.

	B5  Unanswered Tier 2 questions from QUESTIONS.md have their default
	    assumptions applied, not left blank.
	    FAIL condition: a section that required a Tier 2 default is empty or
	    contains a raw placeholder.
	    SKIP condition: all Tier 2 questions were answered.

	B7  Answers marked ??? in QUESTIONS.md have hedging language applied in
	    the output documents wherever that answer was used as a source.
	    Acceptable hedging forms: "reportedly", "as described by the operator",
	    "not confirmed in code", "may not reflect current state".
	    FAIL condition: a ??? answer was used as a source but the corresponding
	    claim in the output carries no hedging language and no [unverified] note.
	    SKIP condition: no answers are marked ???.

## Category C - Content Quality


	C1  The Theory of Operation (or equivalent section in LORE_ABSTRACT.md)
	    is coherent and self-consistent. A reader unfamiliar with the project
	    could understand what it does after reading it.
	    FAIL condition: the section is incoherent, self-contradictory, or so
	    vague as to convey no useful information.
	    Confidence note: this check is subjective. Record your confidence level.

	C2  The ENCHIRIDION.md subsystem catalog entries are internally consistent
	    and bidirectionally accurate.
	    Check 1: No subsystem is listed as both a dependency of another and
	    having no integration points.
	    Check 2: For each subsystem X that lists subsystem Y as an integration
	    point, verify that subsystem Y's entry also acknowledges a relationship
	    with subsystem X. A one-sided integration claim is a defect.
	    FAIL condition: any detectable internal contradiction, or any integration
	    point claim that is not acknowledged by the other subsystem's entry.

	C3  The docs/LORE_INDEX.md navigation guide is accurate. Each document
	    description matches the document's actual contents.
	    FAIL condition: a description that is misleading or factually wrong
	    about the document it describes.

	C4  No output document contains raw template syntax.
	    Markers to check for: [PLACEHOLDER], [INSERT, {{, }}, <<<, >>>,
	    [TEMPLATE INSTRUCTIONS, [END TEMPLATE INSTRUCTIONS]
	    FAIL condition: any of these strings appear in any output document,
	    including LORE_PRIMER_*.md and LORE_GUIDE_*.md files.

	C5  No output document contains unicode characters or emojis.
	    Check for any non-ASCII byte in the output files.
	    FAIL condition: any non-ASCII character found.

	C6  No output document is empty or under 50 words in its narrative sections.
	    An empty document or a stub document is a production failure, not a
	    content quality issue.
	    FAIL condition: any document has fewer than 50 words of narrative content.

	C7  No abbreviation or domain-specific term appears more than 3 times across
	    LORE_ABSTRACT.md and LORE_ENCHIRIDION.md without being defined on first use.
	    Use the Domain Jargon Candidates list from docs/.LORE/work/SKELETON.md as a
	    starting point, but also scan the output documents independently for
	    any additional terms not caught in stage 1.
	    A term is "defined on first use" if its full meaning or plain-language
	    equivalent is written out the first time it appears. Two forms are acceptable:
	        Acronym expansion: "Location Routing Number (LRN)"
	        Plain-language gloss: "DIP (real-time carrier lookup)" - acceptable
	        even if DIP is a product label rather than a true acronym.
	    FAIL condition: any term from the jargon candidates list, or any
	    abbreviation in ALL CAPS that requires domain knowledge to understand,
	    appears more than 3 times without a definition in either form above.
	    Note: universally understood acronyms (API, HTTP, JSON, DB, URL) are
	    exempt from this check.
	    Severity: MINOR (log for human review; auto-correct only if the fix is
	    one line - add the gloss in parentheses on first use).

## Category D - Pipeline Integrity


	D1  docs/.LORE/work/JOURNAL.md status field reflects the actual state of
	    the pipeline. If all stage 4 artifacts exist, status should be
	    IN_PROGRESS (this stage has not yet set it to COMPLETE).
	    FAIL condition: status is ERROR or AWAITING_HUMAN unexpectedly.

	D2  ANALYSIS.md completion_status is consistent with what stage 4 produced.
	    If ANALYSIS.md is PARTIAL, the output documents should have [INCOMPLETE]
	    markers in the relevant sections.
	    FAIL condition: ANALYSIS.md is PARTIAL but output has no [INCOMPLETE]
	    markers, suggesting stage 4 fabricated content for incomplete sections.

	D3  Every document listed in JOURNAL.md artifacts matches its actual state
	    on disk (EXISTS vs PARTIAL vs MISSING).
	    FAIL condition: any discrepancy between journal and disk state.

---

# Step 3 - Classify Defects

For each FAIL or PARTIAL result from Step 2, classify the defect:

	BLOCKING - must be fixed before documentation is usable
	    Applies to: A1, A2 (missing section), A3 (missing section or subsystem),
	    A7 (missing GUIDE), B2, B4, B6, B7, C4, C5, C6, D2

	MAJOR - should be fixed before documentation is distributed
	    Applies to: A4, A5, A6, A8, B1, B3 (confidence 3 or 4), C1 (confidence 3 or 4),
	    C2, C3, D3

	MINOR - should be noted but does not block use
	    Applies to: B5, B3 (confidence 2 or lower), C1 (confidence 2 or lower),
	    D1, any PARTIAL result on an otherwise passing check

---

# Step 4 - Apply Corrections

For each BLOCKING or MAJOR defect, apply a correction to the relevant
output document now, in this pass.

Rules for corrections:

	- Make the minimum change required to fix the defect.
	  Do not rewrite sections that are not defective.
	- Every correction must be traceable to a defect ID (A1, B3, etc.).
	- If a correction requires information not available in work files,
	  write [INCOMPLETE - correction blocked: reason] instead of fabricating.
	- After applying a correction, note it in your working list of
	  corrections applied.
	- Do not apply corrections to MINOR defects unless the fix is trivial
	  (one word or one line). Log MINOR defects for the human's review instead.

For each defect where a correction was NOT applied (blocked or MINOR):
	Record the defect in the Next Pass Instructions section of REVIEW_NOTES.md
	with a specific instruction for how to fix it.

---

# Step 5 - Determine Exit Condition

After Step 4, evaluate the exit condition:

	Condition 1 - All checks pass:
	    Overall score equals total checks (all PASS, no PARTIAL, no FAIL).
	    Exit: YES
	    Set JOURNAL.md status to COMPLETE.

	Condition 2 - Maximum iterations reached:
	    This is pass 3 (qa_pass_number == 3).
	    Exit: YES regardless of score.
	    Set JOURNAL.md status to COMPLETE if no BLOCKING defects remain.
	    Set JOURNAL.md status to ERROR if any BLOCKING defects remain,
	    and set last_error to a description of the remaining blocking defects.

	Condition 3 - Neither condition met:
	    Exit: NO
	    Set JOURNAL.md status to IN_PROGRESS.
	    The next pass must be run in a fresh context window using the
	    handoff note produced by this stage.

---

# Step 6 - Write REVIEW_NOTES.md

Write docs/.LORE/work/REVIEW_NOTES.md using the REVIEW_NOTES.md schema from
lore/SCHEMAS.md exactly.

	qa_pass_number: [this pass number]
	overall_score: [N] of [total] checks passed
	exit_condition_met: [YES | NO]

	### Checks Performed
	[complete table of all checks run this pass - see schema for format]

	### Defects Found
	[numbered list - see schema for format]
	[if none: write "none"]

	### Corrections Applied
	[list of corrections successfully made this pass - see schema for format]
	[if none: write "none"]

	### Corrections Deferred
	[list of defects that could not be auto-corrected this pass - see schema]
	[if none: write "none"]

	### Next Pass Instructions
	[only if exit_condition_met is NO]
	[numbered list of what the next pass must address - drawn from Corrections
	 Deferred entries and any remaining FAIL/PARTIAL checks]

If REVIEW_NOTES.md already exists from a prior pass, overwrite it in full
with this pass's data. The file always reflects the most recent pass only.
Prior pass data is preserved in JOURNAL.md (see Step 7).

---

# Step 7 - Update JOURNAL.md

Update docs/.LORE/work/JOURNAL.md:

	last_updated: current timestamp (YYYY-MM-DD HH:MM)
	last_stage_completed: stage_5_qa.md

	status:
	    COMPLETE if exit_condition_met is YES and no BLOCKING defects remain
	    ERROR if exit_condition_met is YES and BLOCKING defects remain
	    IN_PROGRESS if exit_condition_met is NO

	last_error:
	    "none" if status is COMPLETE
	    description of remaining BLOCKING defects if status is ERROR

	Completed Stages: append one of:
	    - stage_5_qa.md: DONE - pass [N], [passed] of [total] checks passed, [defect count] defects
	    - stage_5_qa.md: PARTIAL - pass [N] of 3, [defect count] defects remain, re-run required
	    (note: last_stage_completed for stage 4 should read stage_4_lore.md)

	Artifacts section: overwrite in full.
	    Update any document whose state changed during corrections.
	    Add docs/.LORE/work/REVIEW_NOTES.md: EXISTS.

	Append a QA summary to the journal under a section titled
	"QA Pass [N] Summary". This section is never overwritten -
	each pass appends a new summary. Format:

	    ### QA Pass [N] Summary
	    Date: [timestamp]
	    Checks: [passed] of [total] passed
	    BLOCKING defects: [count]
	    MAJOR defects: [count]
	    MINOR defects: [count]
	    Corrections applied: [count]
	    Exit condition met: [YES | NO]
	    [If NO:] Next pass must address: [brief list]

---

# Step 8 - Validate REVIEW_NOTES.md

Before writing the handoff, verify:

	[ ] docs/.LORE/work/REVIEW_NOTES.md exists
	[ ] qa_pass_number matches this pass number
	[ ] overall_score format is correct ([N] of [total] checks passed)
	[ ] exit_condition_met is YES or NO
	[ ] Checks Performed table has one row per check run
	[ ] All Result values are PASS, FAIL, or PARTIAL
	[ ] All Confidence values are 0, 1, 2, 3, or 4
	[ ] Defects Found section is present (even if "none")
	[ ] Corrections Applied section is present (even if "none")
	[ ] Next Pass Instructions present if and only if exit_condition_met is NO
	[ ] JOURNAL.md status reflects the exit condition correctly

If any check fails: fix it before writing the handoff note.

---

# Step 8b - Write Pipeline Critique (final pass only)

Run this step only if exit_condition_met is YES.
If exit_condition_met is NO, skip this step entirely.

This step produces a structured self-critique of the LORE pipeline instructions
themselves - not of the documentation output. It is the raw material for
meta-analysis of the pipeline's design. It is appended to JOURNAL.md and
is never used by any other pipeline stage. It is for the human operator only.

Read the following before writing the critique:
	- docs/.LORE/work/REVIEW_NOTES.md (all passes, via the QA Pass summaries in JOURNAL.md)
	- docs/.LORE/work/ANALYSIS.md (note any sections that were difficult to populate)
	- docs/.LORE/work/QUESTIONS.md (note any questions that felt wrong in tier or wording)
	- docs/.LORE/work/SKELETON.md (note any sections that were hard to fill from structure alone)

Append the following section to docs/.LORE/work/JOURNAL.md:

	### Pipeline Critique
	Produced by: stage_5_qa.md (pass [N], final)
	This section is for human meta-analysis of the pipeline only.
	It does not affect pipeline state or schema validation.

	#### Stage 1 - Reconnaissance
	friction points:
	[List any instructions in lore/stage_1_recon.md that caused confusion, produced
	 poor output, or were ambiguous. Be specific: name the step and describe
	 the problem. If none: "none observed".]

	output quality:
	[Assess whether SKELETON.md was a useful input for stage 2. Note any
	 sections that were sparse, over-filled, or not used by downstream stages.
	 If no issues: "output quality adequate".]

	#### Stage 2 - Analysis
	friction points:
	[List any instructions in lore/stage_2_analysis.md that caused confusion,
	 produced poor output, or were ambiguous. Be specific. If none: "none observed".]

	context budget behavior:
	[Did the context budget thresholds (60%/75%) produce good cutoff decisions?
	 Were important files left unread? Were low-value files read too early?
	 If no issues: "budget thresholds performed adequately".]

	output quality:
	[Assess whether ANALYSIS.md was a useful input for stage 4. Note any
	 sections that were sparse, over-filled, or not used. If no issues:
	 "output quality adequate".]

	#### Stage 2b - Continuation
	used this run: [YES | NO]
	[If YES:]
	friction points:
	[List any issues with the continuation logic - files re-read, sections
	 overwritten, addendum blocks that were awkward. If none: "none observed".]
	[If NO:]
	not applicable

	#### Stage 2c - Subsystem Completion Gate
	used this run: [YES | NO]
	[If YES:]
	friction points:
	[List any issues with the completion gate - subsystems that required many
	 passes, files that were difficult to classify as FULL vs HEAD, any
	 analysis_complete flags that were set incorrectly. If none: "none observed".]
	completeness achieved: [YES - all warranted subsystems reached analysis_complete YES |
	                        NO - some subsystems still incomplete at stage 3]
	[If NO:]
	not applicable

	#### Stage 3 - Human Gate
	question quality:
	[Were the Tier 1 questions genuinely blocking? Were any Tier 2 questions
	 that should have been Tier 1, or vice versa? Were any questions redundant
	 or unanswerable by the human? If no issues: "question tiers appropriate".]

	gate behavior:
	[Did the gate present questions clearly? Were any human answers ambiguous
	 or hard to record? If no issues: "gate behavior adequate".]

	#### Stage 4 - Documentation Production
	friction points:
	[List any instructions in lore/stage_4_lore.md that caused confusion or
	 produced poor output. Note any template sections that did not fit the
	 project. Be specific. If none: "none observed".]

	template fit:
	[For each template used, assess whether its sections matched the project
	 well. Note any sections that were always empty, always incomplete, or
	 always redundant for this type of project.
	 Format: - [template name]: [assessment]]

	#### Stage 5 - QA
	rubric fit:
	[Were the rubric checks appropriate for this project? Note any checks
	 that were always N/A, always trivially passing, or that missed real
	 problems. If no issues: "rubric checks appropriate".]

	correction effectiveness:
	[Did auto-corrections in Step 4 actually fix defects, or did they tend
	 to produce new problems? If no corrections were applied: "no corrections
	 applied this run".]

	#### Cross-cutting observations
	schema gaps:
	[Note any information that was needed during the run but had no place
	 in any work file schema. Suggest field or section additions if warranted.
	 If none: "no schema gaps observed".]

	naming and terminology:
	[Note any terms used in pipeline instructions that were ambiguous,
	 inconsistent across stages, or confusing in practice. If none: "none".]

	suggested improvements:
	[A numbered list of specific, actionable changes to the pipeline
	 instructions or schemas that would have improved this run.
	 Format: [N]. [stage or file] - [specific change suggested]
	 If none: "none".]

---

# Step 9 - Write the Handoff Note

If exit_condition_met is YES and status is COMPLETE:

	Write the following block exactly, filling in the bracketed fields.
	Copy it into the Last Handoff Note section of JOURNAL.md.
	Output it as the final output of this stage.

	---
	Stage 5 complete - QA pass [N] - Pipeline finished
	Codebase: [total_source_files] files ([languages])
	LORE: Stage [5/5] Pass [N] Context [N]
	Status: COMPLETE - documentation ready

	Final score: [passed] of [total] checks passed
	BLOCKING defects remaining: 0
	MAJOR defects remaining: [count]
	MINOR defects remaining: [count]
	Total files read (all analysis passes): [files_read_total from JOURNAL.md]
	Tokens this context: ~[N]k (estimated)

	Documents delivered:
	    - docs/LORE_ABSTRACT.md:          EXISTS
	    - docs/LORE_ENCHIRIDION.md:       EXISTS
	    - docs/LORE_INDEX.md:             EXISTS
	    - docs/LORE_BRIEF.md:             [EXISTS | MISSING - run stage 7]
	    [for each PRIMER:]
	    - docs/LORE_PRIMER_[name].md:     EXISTS
	    [for each GUIDE:]
	    - docs/LORE_GUIDE_[name].md:      EXISTS

	[If any MAJOR or MINOR defects remain:]
	    Note: [count] non-blocking defects were not auto-corrected.
	    See docs/.LORE/work/REVIEW_NOTES.md for details.

	The pipeline is complete. Documentation is ready for use.
	---

If exit_condition_met is YES and status is ERROR (BLOCKING defects remain
after 3 passes):

	Write the following block exactly, filling in the bracketed fields.
	Copy it into the Last Handoff Note section of JOURNAL.md.
	Output it as the final output of this stage.

	---
	Stage 5 complete - QA pass 3 - Max iterations reached with blocking defects
	Codebase: [total_source_files] files ([languages])
	LORE: Stage [5/5] Pass [3] Context [N]
	Status: ERROR - [count] blocking defects remain after 3 passes

	Final score: [passed] of [total] checks passed
	BLOCKING defects remaining: [count]
	Total files read (all analysis passes): [files_read_total from JOURNAL.md]
	Tokens this context: ~[N]k (estimated)

	The following blocking defects could not be auto-corrected:
	[For each remaining BLOCKING defect:]
	    [N]. [Check ID] - [description]
	         Location: [file and section]
	         Correction required: [what must change]

	Recommended action:
	    1. Review the defects listed above.
	    2. Make the corrections manually to the files in docs/.
	    3. Re-run QA only by choosing option [Q] from MENU D in lore/LORE.md.

	Pipeline status set to ERROR.
	See docs/.LORE/work/REVIEW_NOTES.md for full defect details.
	---

If exit_condition_met is NO (passes remaining):

	Write the following block exactly, filling in the bracketed fields.
	Copy it into the Last Handoff Note section of JOURNAL.md.
	Output it as the final output of this stage.

	---
	Stage 5 partial - QA pass [N] complete - additional pass required
	Codebase: [total_source_files] files ([languages])
	LORE: Stage [5/5] Pass [N] Context [N]
	Status: PARTIAL - [count] defects remain, run Stage 5 again (pass [N+1] of 3)

	Pass [N] score: [passed] of [total] checks passed
	Defects corrected this pass: [count]
	Defects deferred this pass: [count]
	Defects remaining: [count] ([BLOCKING count] BLOCKING, [MAJOR count] MAJOR, [MINOR count] MINOR)
	Tokens this context: ~[N]k (estimated)

	Next pass must address:
	[copy the Next Pass Instructions list from REVIEW_NOTES.md verbatim]

	Please start a new context with this text:

	Please follow `lore/LORE.md` and CONTINUE.
	---

---

# DO NOT

- Do not fabricate corrections when information is not available in work files
- Do not rewrite sections of output documents that are not defective
- Do not run more than 3 passes total across all context windows for this pipeline run
- Do not modify any file in lore/ other than docs/.LORE/work/JOURNAL.md and
  docs/.LORE/work/REVIEW_NOTES.md during this stage
- Do not skip BLOCKING defect corrections to hit a passing score
- Do not mark a check PASS if you have low confidence - use PARTIAL or record
  your confidence level honestly
- Do not run Step 8b (pipeline critique) unless exit_condition_met is YES
- Do not proceed to any further stage after this one
  (stage 5 is the final stage - output the handoff note and stop)