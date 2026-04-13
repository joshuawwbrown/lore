# LORE Pipeline Concepts and Terminology
# Schema Version: 1.6
# Self-contained reference for all proprietary terms used in this pipeline.
# Read this file if you are new to LORE or if a term is unclear.
# This file is for AI and human readers alike. It has no effect on pipeline state.



---

# what is LORE

LORE is a staged documentation pipeline that runs inside an AI assistant
context window. It reads a software project and produces a set of reference
documents without requiring build tools, servers, or external services.

The pipeline is designed to span multiple context windows. Each stage produces
artifacts that the next stage consumes. A handoff note at the end of each stage
tells the next context window exactly where to resume.

LORE lives in a directory called lore/ at the root of (or adjacent to) the
project being documented. It writes work files to docs/.LORE/work/ and final output
to docs/ in the project root.

A core principle: no documentation is ever produced for a file that has not
been read. Every entry in a PRIMER file directory must be sourced from a file
that was read with policy FULL during analysis. This is enforced by the
analysis_complete gate in stage 2c before documentation production begins.

---

# pipeline stages

The pipeline has six numbered stages plus two overflow stages:

	stage_1_recon.md
	    Reconnaissance. Reads project structure without reading implementation
	    files. Produces SKELETON.md, QUESTIONS.md, and initializes JOURNAL.md.
	    Also produces a resource estimate (files, context windows, tokens) so
	    the human can decide whether to proceed.

	stage_2_analysis.md
	    Deep analysis. Reads the files flagged by stage 1 in PRIMER-score
	    priority order (highest-score subsystems first). Produces ANALYSIS.md
	    and a consolidated QUESTIONS.md.

	stage_2b_continuation.md
	    Overflow pass. Runs only when stage 2 ran out of context budget before
	    reading all flagged files. Appends to ANALYSIS.md without rewriting it.
	    May be run multiple times until the flagged file list is exhausted.

	stage_2c_subsystem_completion.md
	    Subsystem completion gate. Runs after stage 2 (and any 2b passes).
	    Reads all remaining unread files in every PRIMER-warranted subsystem
	    until every such subsystem has analysis_complete: YES. May be run
	    multiple times. Stage 3 cannot proceed until this gate is cleared.
	    This is the enforcement stage for the "no undocumented files" principle.

	stage_3_human_gate.md
	    Human gate. Presents unanswered questions to the human, records answers,
	    and routes the pipeline to stage 4. Does not answer questions itself.
	    Hard prerequisite: all PRIMER-warranted subsystems must have
	    analysis_complete YES before this stage may run.

	stage_4_lore.md
	    Documentation production. Reads all work artifacts and renders final
	    output documents using templates from lore/templates/. Only produces
	    PRIMERs for subsystems with analysis_complete YES.

	stage_5_qa.md
	    Adversarial QA loop. Checks output documents against a binary rubric,
	    applies corrections, and produces REVIEW_NOTES.md. Runs up to 3 passes.
	    Includes check B6: no PRIMER may document a file that was not read
	    with policy FULL.

---

# work files

Work files live in docs/.LORE/work/. They are written by stages and consumed by
subsequent stages. Do not edit them manually unless recovering from an error.

	JOURNAL.md
	    The persistent state file for the entire pipeline run. Contains:
	    pipeline status, timestamps, list of completed stages, list of
	    artifacts and their states, files_read_total (cumulative count of
	    FULL-read source files across all analysis passes), unanswered
	    questions, ignored subsystems list, the last handoff note, QA pass
	    summaries, and the pipeline critique (after stage 5). Every stage
	    reads this file first to verify prerequisites.

	SKELETON.md
	    The structural reconnaissance artifact from stage 1. Contains a
	    compressed map of the project: identity, dependency summary, directory
	    map, entry point summaries, flagged files for deep read (with read
	    policy per file), resource estimate, subsystem candidates with PRIMER
	    scores, existing documentation inventory, domain jargon candidates,
	    and all human questions. Read by stages 2, 2b, 2c, and 4.

	ANALYSIS.md
	    The deep knowledge artifact from stages 2, 2b, and 2c. Contains:
	    theory of operation, subsystem entries (each with a file_inventory,
	    files_read N/M count, analysis_complete flag, and disposition field),
	    data paradigm, IT paradigm, code style, key files directory, and
	    observed bugs. This is the primary source of truth for stage 4.
	    Has a completion_status field (COMPLETE or PARTIAL). A subsystem
	    entry with analysis_complete: NO must not be used to produce a
	    PRIMER. A subsystem with disposition: ignored must not appear in
	    any output document. A subsystem with disposition: deprecated gets
	    a stub entry only.

	QUESTIONS.md
	    The consolidated question list produced by stages 1 and 2, answered
	    by the human during stage 3. Questions are grouped by tier and carry
	    ANSWER and CONFIDENCE fields. Stage 4 uses resolved answers to fill
	    documentation. Q13-style answers may set subsystem dispositions
	    (deprecated or ignored), recorded in JOURNAL.md.

	MANIFEST.md
	    The file manifest produced by stage 2 and updated by stages 2b, 2c,
	    and 6. Records every file read during analysis with its mtime, assigned
	    subsystem, stage, and read policy. Also carries mtime_available (YES/NO),
	    file_read_policy_applied (STANDARD/HEAD_APPLIED), and a Preserved Files
	    section listing non-LORE docs/ files. Used by stage 6 for update
	    detection.

	REVIEW_NOTES.md
	    The QA artifact produced by stage 5. Contains the rubric check table,
	    defects found, corrections applied, and next-pass instructions.
	    Overwritten on each QA pass. Per-pass summaries are preserved in
	    JOURNAL.md.

---

# output documents

Output documents live in docs/ at the project root. They are produced by
stage 4 and checked by stage 5.

All output filenames are prefixed with LORE_ to avoid namespace collisions
with any existing files in docs/.

	docs/LORE_ABSTRACT.md
	    The primary human-facing document. Audience: a new developer joining
	    the project. Written for readability - uses a mix of prose paragraphs,
	    structured sections, tables, and bullet lists as appropriate.
	    Covers what the project does, how it works, architecture overview,
	    data layer, infrastructure, getting-started instructions, key
	    dependencies, and known risks. Source tags may be omitted from this
	    document's final rendered form (unlike LORE_ENCHIRIDION.md where
	    tags are mandatory).

	docs/LORE_ENCHIRIDION.md
	    The AI-agent-facing technical reference. Audience: an AI coding
	    assistant or an experienced developer using it as a lookup reference.
	    Precision and completeness matter more than narrative flow. Contains
	    the full subsystem catalog, key files directory, data paradigm, IT
	    paradigm, code style reference, and observed bugs. Every factual
	    claim carries a mandatory [src:] or [inferred] tag.

	docs/LORE_INDEX.md
	    The entry index for the docs/ directory. Audience: anyone arriving at
	    docs/ for the first time. Lists every document with a one-line
	    description and explains how to regenerate the documentation. Short
	    by design - it is navigation, not content.

	docs/LORE_[name]_PRIMER.md
	    A deep-reference document for a single subsystem. One file per
	    subsystem whose PRIMER score is 2 or higher. Audience: a developer
	    working on or integrating with that specific subsystem. Deeper than
	    the ENCHIRIDION entry for the same subsystem.
	    Example: docs/LORE_auth_PRIMER.md

---

# key concepts

## PRIMER score

An integer from 0 to 5 assigned to each subsystem candidate during stage 1
and potentially revised upward in stage 2. It measures how strongly a
dedicated PRIMER document is warranted. One point for each criterion met:

	+1  five or more files are dedicated to this subsystem
	+1  referenced by three or more other subsystems (estimated from imports)
	+1  has a non-obvious pattern or framework convention
	+1  has its own external dependencies
	+1  is a standalone operational tool with its own configuration and more
	    than 5 non-trivial files, independent of the main application
	    (examples: deploy/, tools/, cli/, scripts/)

The fifth criterion exists specifically for operator tooling and CLI toolkits
that run independently of the application and therefore score 0 on the
"referenced by multiple other parts" criterion. Without this criterion,
such subsystems are systematically under-scored and missed.

A score of 2 or higher means stage 4 will produce a LORE_PRIMER_[name].md
file for that subsystem. A score of 0 or 1 means no PRIMER is produced.

Stage 1 scores conservatively. Stage 2 may revise scores upward after
reading actual files.

A PRIMER is only produced after the subsystem has analysis_complete: YES.
If a warranted subsystem is not fully analyzed when stage 4 runs, no PRIMER
is produced and the ENCHIRIDION entry is marked [ANALYSIS INCOMPLETE].

## human confidence model

When a human answers a question at the human gate (stage 3), LORE treats all
answers as authoritative by default. No numeric confidence score is required.

If the human is uncertain about an answer, they append ??? to it:

    ANSWER: I think it deploys to Cloud Run but it might be Cloud Functions ???

The ??? suffix signals low confidence. Stage 4 applies hedging language to
claims derived from that answer (e.g. "reportedly", "as described by the
operator", "not confirmed in code"). Stage 5 cross-references low-confidence
answers against code observations before accepting them as authoritative.

Answers without ??? are treated as confidence 4 (certain) internally.
Answers with ??? are treated as confidence 1 (low) internally.

The CONFIDENCE field still exists in QUESTIONS.md for internal pipeline use,
but it is derived automatically - humans never fill it in directly.

---

## question tiers

Human questions are classified into three tiers that control how they are
handled at the human gate (stage 3) and in documentation production (stage 4).

	Tier 1 - Blocking
	    Documentation cannot be accurate without an answer. Stage 4 will not
	    fabricate an answer. Instead it writes an [AWAITING ANSWER] marker in
	    the relevant section of the output document. The maximum number of
	    Tier 1 questions across the entire QUESTIONS.md file is 5. Questions
	    that seem blocking but can be worked around must be Tier 2.

	Tier 2 - Enhancing
	    Documentation is more accurate with an answer, but a reasonable
	    default assumption exists. QUESTIONS.md carries a "Default assumption
	    if unanswered" field for every Tier 2 question. If unanswered, stage 4
	    applies the default and tags the resulting claim [inferred - Q[N]
	    unanswered].

	Tier 3 - Cosmetic
	    Would add polish but has no accuracy impact. Safe to skip entirely.
	Unanswered Tier 3 questions appear as open questions in LORE_ABSTRACT.md
	    and LORE_ENCHIRIDION.md but do not block or degrade any section.

## subsystem disposition

Every subsystem identified during analysis carries a disposition field that
controls how it is treated in documentation production.

	active
	    Normal treatment. Full analysis, PRIMER if warranted, GUIDE if
	    operator-facing. This is the default for all subsystems unless
	    the human or code evidence indicates otherwise.

	deprecated
	    The subsystem still exists in the codebase but is being replaced
	    or removed. Stage 4 produces a stub entry in the ENCHIRIDION and
	    a single row in the OVERVIEW subsystems table, both marked
	    "(deprecated)". No PRIMER is produced. The stub names the
	    replacement and explains why the deprecated code still exists.
	    Example: google/ legacy deploy scripts - superseded by deploy/.

	ignored
	    Foreign code, vendored samples, or unrelated projects that should
	    not be documented as part of this project. Excluded from all output
	    documents entirely. Noted in JOURNAL.md Ignored Subsystems section
	    only. Example: noccrm-google-sample/ copied from another project.

Disposition is set in ANALYSIS.md by stage 2 (inferred from code and
directory names) and confirmed or changed by the human at stage 3 via
Q13-style answers. Stage 4 reads the disposition for every subsystem
before producing any output.

## file read policy

Every file encountered during analysis is assigned one of three read policies.
The policy is recorded in SKELETON.md's Files Flagged for Deep Read section
and enforced throughout the analysis stages.

	FULL
	    Read the entire file. Applies to all source code files (.js, .mjs,
	    .ts, .py, .go, .sh), config files, Dockerfiles, markdown docs within
	    a subsystem, and YAML/JSON config files under approximately 100 lines.
	    Only FULL-read files may appear in PRIMER file directories with
	    function-level documentation.

	HEAD
	    Read the first 20 lines only. Applies to data files, large lookup
	    tables, large JSON arrays, CSV files, and any JSON/YAML file that is
	    clearly a data dump rather than configuration. The HEAD read confirms
	    the file's purpose and records it in the inventory with a one-liner.
	    No function documentation is ever produced for a HEAD-read file.

	SKIP
	    Do not read. Applies to lock files (package-lock.json, yarn.lock),
	    compiled output, minified JS, binary files, and anything in
	    node_modules/, .git/, dist/, build/, target/. SKIP files do not
	    appear in any file inventory.

No documentation is ever produced for a file that was not read with policy
FULL. This is an absolute rule enforced by stage 2c and verified by QA
check B6. Analysis stages update ANALYSIS.md in place — new findings from
stage 2b and 2c are merged directly into existing subsystem entries rather
than appended as separate ADDENDUM blocks. ANALYSIS.md must remain readable
as a single coherent reference document throughout the pipeline.

## analysis_complete

A boolean field on each subsystem entry in ANALYSIS.md. Set to YES when the
number of files in the subsystem's file_inventory (FULL reads) equals the
total file count for that subsystem from SKELETON.md. Set to NO otherwise.

Stage 2c runs until all PRIMER-warranted subsystems have analysis_complete YES.
Stage 3 will not proceed until this condition is met. Stage 4 will not produce
a PRIMER for any subsystem where analysis_complete is NO.

## token self-reporting

Each stage's handoff note includes an estimated token count for that context
window ("Tokens this context: ~Nk"). This is self-reported by the AI based
on its sense of context consumption. The estimate is approximate but useful
for calibrating the stage 1 resource estimate across multiple runs. The
stage 5 final handoff also reports files_read_total from JOURNAL.md, giving
a complete picture of run cost.

## resource estimate

Produced by stage 1 Step 10b. Records the estimated cost of a full
documentation run: total source files to read, PRIMER-warranted subsystem
count (preliminary — stage 2 regularly revises upward), estimated context
windows, and estimated tokens. Displayed prominently in the stage 1 handoff
note so the human can decide whether to proceed before committing to the
analysis passes.

## source tagging

Every factual claim in work files, LORE_ABSTRACT.md, and LORE_ENCHIRIDION.md
must carry one of two tags:

	[src: filename]
	    The claim is directly supported by a named file that was read during
	    analysis. Use the shortest unambiguous path relative to the project
	    root. For a specific line: [src: filename:line].

	[inferred]
	    The claim is a reasonable inference from structure, context, or
	    observed patterns. It has not been directly verified in source code.
	    Inferred claims must be plausible - do not use [inferred] to cover
	    fabricated content.

Source tags are required in both LORE_ABSTRACT.md and LORE_ENCHIRIDION.md.
They appear in the output files, not just in working memory. This policy
applies uniformly across all output documents so that the same fact is
equally verifiable regardless of which document a reader is using.

## answer confidence

The CONFIDENCE field in QUESTIONS.md signals how certain the human is about
an answer. Humans do not use a numeric scale - they use the ??? suffix instead.

    Blank CONFIDENCE (answered question):
        Treated as authoritative. Stage 4 uses the answer without hedging.

    ??? suffix on the answer:
        Treated as low confidence. Stage 4 applies hedging language wherever
        the answer is used ("reportedly", "as described by the operator",
        "not confirmed in code"). Stage 5 cross-references the answer against
        code observations before accepting it as authoritative.

Internally, the pipeline maps these to numeric confidence levels for QA logic:
    blank (answered) = high confidence (3)
    ???              = low confidence (1)

This mapping is not exposed to the human. Humans only ever write answers
and optionally append ???. They never fill in a number.

See also: human confidence model (earlier in this section).

## domain jargon candidates

A list of project-specific and domain-specific terms collected by stage 1
during reconnaissance. Stored in a "Domain Jargon Candidates" section of
SKELETON.md. Consumed by stage 5 QA check C7.

A jargon candidate is any term that a developer from outside the project's
industry would not recognize. The test is: "would a competent backend
developer, hired today, be confused by this term when reading the code?"

A jargon candidate IS:
    - an industry abbreviation or acronym (DIP, LRN, NPA-NXX, NANP, OCN)
    - a domain term specific to the project's vertical (carrier, ported,
      deactivation record, litigator, test-drive in telecom context)
    - a project-internal name for a non-obvious concept (grocer, deac)

A jargon candidate is NOT:
    - an internal module or library name (zero, fmt, biz) - these belong
      in the PRIMER file directory, not the jargon list
    - a common programming term used normally (handler, router, schema)
    - a universally understood acronym (API, HTTP, JSON, DB, URL)

Stage 1 populates the list from directory names, file names, README content,
and config key names. Stage 5 check C7 verifies that each candidate term
appearing more than 3 times in output documents is defined on first use.
C7 accepts both acronym expansions ("Location Routing Number (LRN)") and
plain-language glosses ("DIP (real-time carrier lookup)") as valid
definitions, since some terms are product labels rather than true acronyms.

## completion_status

A field at the top of ANALYSIS.md. One of two values:

	COMPLETE
	    All files flagged by stage 1 were read in stage 2 (or stage 2b).
	    No sections are marked [INCOMPLETE].

	PARTIAL
	    Context budget was reached before all flagged files were read.
	    Incomplete sections are marked [INCOMPLETE - reason] inline.
	    Stage 2b should be run before proceeding to stage 3.

Stage 4 propagates [INCOMPLETE] markers from ANALYSIS.md into output
documents. Stage 5 checks that this propagation happened correctly.

## context budget thresholds

Stage 2 monitors context consumption continuously and uses two thresholds:

	CAUTION - approximately 60% consumed
	    Finish reading the current file. Do not start another.
	    Set completion_status to PARTIAL and proceed to write output.

	STOP - approximately 75% consumed
	    Stop immediately, even mid-file.
	    Write what is available, mark everything else INCOMPLETE.

These thresholds exist because writing ANALYSIS.md also consumes context.
Stopping too late leaves no budget to write the output.

## confidence scale

Used in stage 5 QA rubric checks to qualify how certain the AI is about
a check result. Applies especially to subjective checks.

	0 = no confidence (0%)   - result is a guess
	1 = low confidence (25%) - some basis but highly uncertain
	2 = moderate (50%)       - reasonable basis, meaningful uncertainty
	3 = high confidence (75%) - strong basis, minor uncertainty
	4 = certain (100%)       - directly verifiable, no ambiguity

A confidence of 2 or lower makes a check result advisory only. It is still
recorded in REVIEW_NOTES.md but is not used to block pipeline completion.

## defect severity

Used in stage 5 to classify problems found in output documents.

	BLOCKING
	    Must be corrected before documentation is considered usable.
	    Stage 5 attempts auto-correction on all blocking defects.
	    If any blocking defects remain after 3 QA passes, the pipeline
	    sets JOURNAL.md status to ERROR.

	MAJOR
	    Should be corrected before documentation is distributed.
	    Stage 5 attempts auto-correction on major defects.
	    If uncorrected after 3 passes, they are listed for manual review.

	MINOR
	    Should be noted but does not block use or distribution.
	    Stage 5 logs minor defects but does not auto-correct them unless
	    the fix is trivial (one word or one line).

## inline markers

Markers written into output documents to signal deliberate gaps or conditions.
They are never removed by stage 4. Stage 5 checks that they are present
where they should be and absent where they should not be.

	[AWAITING ANSWER - Q[N]: summary]
	    Written where a Tier 1 question was left unanswered. Signals to the
	    human that this section is inaccurate until Q[N] is answered.

	[INCOMPLETE - reason]
	    Written where analysis did not cover a section (context budget reached,
	    or files were not read). Propagated from ANALYSIS.md into output docs.

## handoff note

The block of text output at the end of every stage. It is also written into
the Last Handoff Note section of JOURNAL.md. Its purpose is to give the next
context window everything it needs to resume without re-reading all work files.

Every handoff note contains:
	- which stage just completed
	- which stage to run next (and which file to read)
	- project name and root
	- schema version
	- current state of all work files and output files

The human copies the handoff note into a fresh context window and says:

    "Please follow lore/START.md to continue the pipeline."

## canonical output files

The specific set of files stage 4 is authorized to produce and overwrite.
Stage 4 overwrites these files on every run. Stage 5 is the only other stage
authorized to modify them (corrections only).

	docs/LORE_ABSTRACT.md
	docs/LORE_ENCHIRIDION.md
	docs/LORE_INDEX.md
	docs/LORE_PRIMER_[name].md  (one per PRIMER-warranted subsystem, score 2+)
	docs/LORE_GUIDE_[name].md   (one per user-runnable subsystem)

The general naming convention for all canonical output files is:

	LORE_[type]_[label].md

where type is one of: OVERVIEW, ENCHIRIDION, INDEX, PRIMER, GUIDE
and label is the lowercase underscored subsystem name (omitted for singletons).

Any file in docs/ that does not match the LORE_[type]* pattern is a
user-created file. Stage 4 must never modify or delete user-created files.

## GUIDE documents

A GUIDE is an operator-facing document for a subsystem that has
human-runnable entry points, multi-step workflows, or configuration a
human must fill in before running. One GUIDE per qualifying subsystem.

A subsystem qualifies for a GUIDE if it has any of:
	- operator-invocable scripts, CLIs, or npm targets
	- a multi-step process a human must follow in order
	- configuration a human must fill in before the subsystem works
	- known failure modes or blocking issues an operator needs before running

The GUIDE audience is an operator running the tool, not a developer reading
the code. Tone is plain language and imperative. It covers the whole
subsystem in one document - all executables and workflows together.

A subsystem may have both a PRIMER and a GUIDE. They serve different
audiences: PRIMER for developers reading code, GUIDE for operators running it.

Stage 2 records a user_runnable field (YES | NO | MAYBE) on each subsystem
entry in ANALYSIS.md. Stage 4 produces a GUIDE for every YES subsystem.
MAYBE subsystems generate a Tier 3 question at stage 3.

## schema version

A version string declared at the top of lore/SCHEMAS.md. Written into
JOURNAL.md when the pipeline starts. On resume, the pipeline checks that
the journal's schema_version matches the current SCHEMAS.md version. A
mismatch triggers MENU F (schema migration) in lore/LORE.md.

Current version: 1.6

## scoped update vs full re-scan

When documentation exists and project files have changed, the pipeline offers
two options:

	[U] Scoped update
	    Stage 6 reads docs/.LORE/work/MANIFEST.md and compares recorded mtimes against
	    current filesystem mtimes using bash ls -la if available, or git diff if
	    available. Only files whose mtime is newer than the recorded value are
	    re-read. ANALYSIS.md is patched in place. Only affected output document
	    sections are regenerated. Faster and cheaper than a full re-scan.
	    Best for routine code changes to known subsystems.

	    MANIFEST.md carries a mtime_available field (YES | NO) that stage 6
	    checks first. If NO, mtime comparison is skipped and git diff is used
	    immediately. MANIFEST.md also carries a Preserved Files section listing
	    all non-LORE files in docs/ at the time of the last run, so stage 6
	    can detect user files that were renamed or deleted between runs.

	[F] Full re-scan
	    All analysis work files (SKELETON.md, ANALYSIS.md, QUESTIONS.md,
	    MANIFEST.md) are discarded. The pipeline restarts from stage 1.
	    JOURNAL.md is preserved. Use this when the project structure has
	    changed significantly, many files have changed, or the scoped update
	    is producing unreliable results.

Stage 6 will warn the user and suggest [F] if more than 8 files have changed
or if more than 5 files are new (not previously in the manifest).

## LORE_BRIEF.md

The AI session entry-point document. Audience: an AI coding assistant opening
a fresh session on this project. It is the first file the AI reads before
beginning development work.

LORE_BRIEF.md contains:
    - an ingestion acknowledgement prompt (the AI responds with a short token
      to confirm it has read the file)
    - AI collaboration guidelines: a generic base ruleset plus project-specific
      rules derived from ANALYSIS.md (code style, framework conventions,
      observed patterns)
    - a concise project overview sufficient to orient the AI without re-reading
      all documentation
    - a directed reading sequence listing which LORE documents to read next,
      in order, before beginning work
    - a completion acknowledgement prompt

Produced by stage 7. Added to the canonical output file set in schema 1.6.
Listed in LORE_INDEX.md. The reading sequence it contains references
LORE_ABSTRACT.md, LORE_ENCHIRIDION.md, and any relevant PRIMERs.

Usage:

    Share lore/LORE_BRIEF.md with your AI assistant at the start of a
    development session. The AI reads it, follows the reading sequence,
    and acknowledges completion before beginning work.

---

## the pipeline critique

A section appended to JOURNAL.md by stage 5 on its final pass (when
exit_condition_met is YES). It is a structured self-assessment of the LORE
pipeline instructions themselves - not of the output documents.

It covers: friction points in each stage, context budget behavior, question
tier quality, template fit, rubric fit, correction effectiveness, schema
gaps, naming issues, and suggested improvements.

It is the primary input for meta-analysis of the pipeline after a run.
It has no effect on pipeline state and is never read by any stage.

---

# file inventory

	lore/README.md              - human-facing overview
	lore/LORE.md                - pipeline reference document; state detection and menus
	lore/SCHEMAS.md             - single source of truth for all work file schemas
	lore/CONCEPTS.md            - this file; glossary and terminology reference
	lore/stage_1_recon.md       - reconnaissance stage instructions
	lore/stage_2_analysis.md    - deep analysis stage instructions
	lore/stage_2b_continuation.md - overflow continuation stage instructions
	lore/stage_2c_subsystem_completion.md - subsystem completion gate instructions
	lore/stage_3_human_gate.md  - human gate stage instructions
	lore/stage_4_lore.md        - documentation production stage instructions
	lore/stage_5_qa.md          - adversarial QA stage instructions
	lore/stage_6_update.md      - scoped update pass instructions
	lore/stage_7_brief.md       - project brief generation stage instructions
	lore/templates/ABSTRACT_template.md     - scaffold for LORE_ABSTRACT.md
	lore/templates/ENCHIRIDION_template.md  - scaffold for LORE_ENCHIRIDION.md
	lore/templates/PRIMER_template.md       - scaffold for LORE_PRIMER_[name].md
	lore/templates/GUIDE_template.md        - scaffold for LORE_GUIDE_[name].md
	lore/templates/ENTRY_README_template.md - scaffold for LORE_INDEX.md
	lore/templates/BRIEF_template.md        - scaffold for LORE_BRIEF.md

	docs/.LORE/work/            - pipeline work files (created at runtime)
	    JOURNAL.md              - persistent pipeline state
	    SKELETON.md             - structural reconnaissance artifact
	    ANALYSIS.md             - deep analysis artifact
	    QUESTIONS.md            - human question list
	    MANIFEST.md             - file manifest with mtimes for update detection
	    REVIEW_NOTES.md         - QA findings (most recent pass)

	docs/                       - output documents (created at runtime)
	    LORE_ABSTRACT.md        - human-facing project overview
	    LORE_BRIEF.md           - AI session entry point and reading sequence
	    LORE_ENCHIRIDION.md     - AI-agent-facing technical reference
	    LORE_INDEX.md           - documentation directory index
	    LORE_[name]_PRIMER.md   - subsystem deep references (one per warranted subsystem)

---

## git integration

LORE can read git metadata to enrich documentation with authorship and change
history. Git integration is optional - the pipeline degrades gracefully when
git is unavailable.

When git is available, stage 1 collects:
    - git_available flag (YES | NO), recorded in SKELETON.md
    - primary authors per subsystem (from git shortlog)
    - last commit date per subsystem (from git log)
    - overall contributor list (from git shortlog -sn)

This data feeds:
    - an Authors section in LORE_ABSTRACT.md
    - per-subsystem primary author fields in LORE_ENCHIRIDION.md
    - LORE_BRIEF.md collaboration guidelines (who to consult per subsystem)

Stage 1 asks permission before running any git command. If the user declines,
git_available is set to NO and all authorship fields are omitted from output.

---

# how to start the pipeline

Open a fresh context window with your AI assistant.
Share lore/LORE.md and say:

    Please follow `lore/LORE.md` to document this project.

The pipeline detects its state from docs/.LORE/work/JOURNAL.md automatically.

To continue after a stage completes, open a fresh context and say:

    Please follow `lore/LORE.md` and CONTINUE.

The word CONTINUE causes the pipeline to skip the menu and execute the next
stage automatically. Each stage's handoff note provides the exact text to
paste - you do not need to construct the prompt yourself.

Stage 1 will produce a resource estimate (files, context windows, tokens)
before any analysis begins, so you can decide whether to proceed.

---

# how to read this pipeline as an AI


If you are an AI assistant executing this pipeline:

1. Read lore/LORE.md first, always. It is the pipeline reference document.
   lore/README.md is for humans and does not contain pipeline instructions.
2. Read lore/SCHEMAS.md before writing any work file.
3. Read lore/CONCEPTS.md (this file) if any term is unclear.
4. Execute only the stage file you are instructed to execute.
5. Do not read ahead into future stage files.
6. Do not modify files outside docs/.LORE/work/ and docs/ unless explicitly
   instructed by the stage file you are executing.
7. Assign read policy FULL to source code files, HEAD to data files, SKIP
   to lock files and compiled output. Never assign FULL to a data file or
   HEAD/SKIP to a source file.
8. Never document a file that was not read with policy FULL. No inferred
   function entries, no placeholder descriptions. If a file was not read,
   it does not appear in the file directory.
9. Tag every factual claim [src: filename] or [inferred]. No exceptions
   in work files or in LORE_ENCHIRIDION.md.
10. Output the handoff note as the final output of each stage. Nothing else
    should follow it. Use the exact format specified in the stage file.