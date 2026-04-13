# Stage 4 - LORE Documentation Production
# Read this file and execute it exactly. Do not skip steps.
# Do not begin writing output until Step 5.
# Pipeline reference: lore/LORE.md



---

# Purpose

This stage produces all final documentation files in the docs/ directory.
It reads the work artifacts produced by stages 1, 2, and 3 and renders them
into human-facing and AI-agent-facing documents using the scaffolded templates
in lore/templates/.

Documents are produced in a fixed order. Each document is validated before
moving to the next. The stage does not stop on minor validation failures -
it flags them and continues. Only a complete inability to produce a document
triggers an error state.

---

# Prerequisites

Before executing this stage:

1. Read lore/SCHEMAS.md in full. Know all schemas before writing anything.

2. Read docs/.LORE/work/JOURNAL.md. Verify:
   - status is IN_PROGRESS
   - last_stage_completed is stage_3_human_gate.md or stage_2b_continuation.md
     (stage 2b is acceptable only if stage 3 was skipped via the [K] path in
     the lore/LORE.md human gate menu)
   Also read the Ignored Subsystems section if present. Build a list of
   ignored subsystem names - these must be excluded from all output.
   Also read the Question Resolution Summary for the Subsystem dispositions
   block - build a lookup of each subsystem's disposition (active, deprecated,
   or ignored).
   If the prerequisite fails, output:
       "Stage 4 prerequisite failed: expected last_stage_completed to be
        stage_3_human_gate.md. Found: [value]. Check the pipeline state."
   Stop.

3. Read docs/.LORE/work/SKELETON.md in full.

4. Read docs/.LORE/work/ANALYSIS.md in full.
   Note: if completion_status is PARTIAL, documentation will be incomplete
   in the affected areas. Note which sections are incomplete before proceeding.
   You will use [INCOMPLETE - analysis not available] markers where needed.

5. Read docs/.LORE/work/QUESTIONS.md in full.
   For each question, record:
   - tier (1, 2, or 3)
   - whether ANSWER is blank or filled
   - for Tier 2 blank answers: the Default assumption field value
   Build a lookup table: Q[number] -> answer or default.

6. Read lore/templates/OVERVIEW_template.md.
7. Read lore/templates/ENCHIRIDION_template.md.
8. Read lore/templates/PRIMER_template.md.
9. Read lore/templates/ENTRY_README_template.md.

---

# Step 1 - Resolve Question Answers

Before writing any document, resolve all questions to a usable value.

For each question in QUESTIONS.md:

    If ANSWER is filled:
        Cross-reference the answer against ANALYSIS.md before accepting it.
        For each answer that names a specific technology, library, service,
        file path, port number, or configuration value:
            Search ANALYSIS.md for any observation about the same topic.
            If the answer is consistent with code observations: use it.
                Tag claims derived from this answer [src: QUESTIONS.md Q[number]].
            If the answer contradicts a code observation:
                Do not use the human's answer as stated.
                Use the code observation as ground truth.
                Note the contradiction explicitly in the relevant section:
                    [Note: Q[number] answer contradicts code observation.
                     Code observation used: [src: filename]. Human answer
                     recorded in QUESTIONS.md for reference.]
                Tag the claim [src: filename] (the code source, not QUESTIONS.md).
            If the answer covers a topic not addressed in ANALYSIS.md:
                Accept it and tag [src: QUESTIONS.md Q[number]].

        Also check the CONFIDENCE field:
            If CONFIDENCE is ???:
                Apply hedging language in the output document wherever this
                answer is used as a source.
                Example hedging: "reportedly", "as described by the operator",
                "may not reflect current state", "not confirmed in code".
                Also add a note: [unverified - low confidence answer]
            If CONFIDENCE is blank (and ANSWER is filled):
                Treat as authoritative. No hedging language needed.

    If ANSWER is blank AND tier is 2:
        Use the Default assumption field value.
        Tag claims derived from this assumption [inferred - Q[number] unanswered].

    If ANSWER is blank AND tier is 1:
        This is a blocking gap. Note it.
        Wherever documentation requires this answer, write:
            [AWAITING ANSWER - Q[number]: one-line question summary]
        Do not invent an answer. Do not skip the section entirely.

    If ANSWER is blank AND tier is 3:
        Treat as optional. If the information would improve a section, note it
        as an open question in the document using plain text, not a marker.

Store this lookup. Reference it throughout document production.

---

# Step 2 - Establish Output Paths

Confirm that a docs/ directory exists at the same level as lore/.
If docs/ does not exist, create it now.

The canonical output paths are:

    docs/LORE_ABSTRACT.md              - primary human-facing overview
    docs/LORE_ENCHIRIDION.md           - AI-agent-facing reference
    docs/LORE_INDEX.md                 - entry index for the docs/ directory
    docs/LORE_PRIMER_[name].md         - one file per PRIMER-warranted subsystem
    docs/LORE_GUIDE_[name].md          - one file per user-runnable subsystem

All PRIMER and GUIDE files live in docs/ alongside the other LORE files,
not in a subdirectory.

---

# Step 3 - Identify PRIMER and GUIDE Targets

## PRIMER targets

From ANALYSIS.md, find every subsystem entry where PRIMER warranted is YES.
These are subsystems with a score of 2 or higher.

For each PRIMER target, record:
    - subsystem name (will become part of the filename)
    - PRIMER score
    - key files list
    - purpose paragraph
    - external dependencies
    - integration points

If ANALYSIS.md has no PRIMER-warranted subsystems, skip PRIMER production
entirely. Note this in the journal.

## GUIDE targets

From ANALYSIS.md, find every subsystem entry where operator_facing is YES,
OR where dev_entry_points is non-empty.

For each GUIDE target, record:
    - subsystem name (will become part of the filename)
    - runnable_entry_points
    - operator_prerequisites
    - known_operator_issues
    - purpose paragraph
    - key files list (filtered to operator-relevant files)

A subsystem may be both a PRIMER target and a GUIDE target. Both files
will be produced. They serve different audiences: the PRIMER is for a
developer reading the code; the GUIDE is for an operator running the tool.

For subsystems where operator_facing is YES: produce a full GUIDE covering
the entire subsystem.
For subsystems where operator_facing is NO but dev_entry_points is non-empty:
produce a GUIDE scoped to those entry points only (the executables and commands
section covers only the listed dev_entry_points; other sections are abbreviated).

If ANALYSIS.md has no operator_facing or dev_entry_points subsystems, skip
GUIDE production entirely. Note this in the journal.

---

# Step 4 - Check docs/ for Existing Files

Read the Existing Documentation Inventory section from SKELETON.md now.
This was written by stage 1 and is your authoritative record of what was
already in docs/ before this run.

List the docs/ directory. For each file that already exists:

    If the file is one of the canonical output files
    (LORE_ABSTRACT.md, LORE_BRIEF.md, LORE_ENCHIRIDION.md, LORE_INDEX.md,
     LORE_PRIMER_*.md, LORE_GUIDE_*.md):
        Note: PRIMERs are only produced for subsystems with analysis_complete YES
        and disposition: active.
        Subsystems with disposition: deprecated get a stub entry only (no PRIMER).
        Subsystems with disposition: ignored are excluded from all output entirely.
        If a warranted subsystem has analysis_complete NO, no PRIMER is produced
        and the ENCHIRIDION entry is marked [ANALYSIS INCOMPLETE - PRIMER not produced].
        Note it. You will overwrite it in this stage.
        This is expected behavior - docs are always regenerated from work files.

    If the file is NOT one of the canonical output files (a user-created file):
        Do not modify or delete it.
        Note it in the journal as a pre-existing file preserved.

## Reconcile Existing Documentation Claims

Read the "Claims extracted from existing documentation" list from the
Existing Documentation Inventory section of docs/.LORE/work/SKELETON.md.

If the list is "none extracted" or empty: skip this section entirely.

If claims are present, for each claim:

    Compare the claim against what ANALYSIS.md says about the same topic.

    If the claim is consistent with ANALYSIS.md:
        No action required. The existing documentation agrees with analysis.

    If the claim contradicts ANALYSIS.md:
        Record a reconciliation conflict note. Format:
            RECONCILIATION CONFLICT: [claim text] [src: existing doc filename]
            Contradicts: [what ANALYSIS.md says] [src: ANALYSIS.md section]
            Resolution: new documentation reflects analysis; existing doc is outdated.
        Append this note to SKELETON.md Corrections section.
        Note it in JOURNAL.md last_error field with severity MINOR.
        The new output document will reflect ANALYSIS.md, not the old claim.

    If the claim covers a topic that ANALYSIS.md does not address:
        Treat it as supplementary context. You may use it as [src: existing doc]
        in the relevant output section, with a note that it was not verified
        against code in this run.

After processing all claims, append to docs/.LORE/work/JOURNAL.md under a section titled
"Existing Docs Reconciliation":
    - Claims reviewed: [count]
    - Conflicts found: [count]
    - Supplementary claims used: [count]
    - Conflicts detail: [one line per conflict, or "none"]

---

# Step 5 - Produce LORE_ABSTRACT.md

Load lore/templates/ABSTRACT_template.md as your scaffolding.
Replace every placeholder in the template with real content.
Do not preserve template scaffolding markers in the output.
The output is a finished document, not a template.

## Source material for LORE_ABSTRACT.md

    Project name and identity:  SKELETON.md Project Identity section
    What the project does:      ANALYSIS.md Theory of Operation section
    Architecture overview:      ANALYSIS.md Subsystems section (summary level)
    Data layer:                 ANALYSIS.md Data Paradigm section
    Infrastructure:             ANALYSIS.md IT Paradigm section
    Setup and run instructions: SKELETON.md Entry Point Summaries + any README
                                content observed in stage 1
    Key dependencies:           SKELETON.md Dependency Summary section
    Human answers:              resolved Q values from Step 1

## Content rules

    - Audience: a new developer joining the project.
    - Length: sufficient to understand the project without reading code.
      Target 400-800 words for the narrative sections, not counting tables.
    - Every factual claim must be tagged [src: filename] or [inferred].
      Source tags are required in the output file. Do not omit them.
      This policy matches ENCHIRIDION.md and ensures cross-document
      consistency and verifiability during QA.
    - Where ANALYSIS.md has [INCOMPLETE] markers, propagate them.
    - Where Tier 1 questions are unanswered, write the [AWAITING ANSWER] marker.
    - Use only ASCII characters.
    - Use tabs for indentation in any code blocks or structured lists.

Write the file to docs/LORE_ABSTRACT.md.
Validate it exists before moving on.

---

# Step 6 - Produce LORE_ENCHIRIDION.md

Load lore/templates/ENCHIRIDION_template.md as your scaffolding.
Replace every placeholder with real content.
The output is a finished document.

## Source material for ENCHIRIDION.md

    Subsystem catalog:          ANALYSIS.md Subsystems section (full detail)
    Key files directory:        ANALYSIS.md Key Files and Functions Directory
    Data paradigm:              ANALYSIS.md Data Paradigm section
    IT paradigm:                ANALYSIS.md IT Paradigm section
    Code style reference:       ANALYSIS.md Code Style and Organization section
    Bugs and risks:             ANALYSIS.md Observed Bugs and Risks section
    Entry points:               SKELETON.md Entry Point Summaries
    Dependencies:               SKELETON.md Dependency Summary section
    Human answers:              resolved Q values from Step 1

## Content rules

    - Audience: an AI coding assistant or a developer using it as a lookup
      reference. Precision and completeness matter more than narrative flow.
    - Structure: prefer labeled sections, bullet lists, and short paragraphs
      over long prose. Tables are acceptable for dependency lists and
      function directories.
    - Every factual claim must be tagged [src: filename] or [inferred].
      Source tags are NOT omitted in ENCHIRIDION.md. They are required in output.
    - Function signatures and type definitions should be quoted verbatim where
      short enough to include without overwhelming the document.
    - Where ANALYSIS.md has [INCOMPLETE] markers, propagate them.
    - Where Tier 1 questions are unanswered, write the [AWAITING ANSWER] marker.
    - Use only ASCII characters.

## Template comment stripping

    The ENCHIRIDION template begins with a [TEMPLATE INSTRUCTIONS] block.
    This block starts with the line "[TEMPLATE INSTRUCTIONS - remove this
    entire block before writing output]" and ends with the line
    "[END TEMPLATE INSTRUCTIONS]".
    Remove this entire block, inclusive of both marker lines, before writing
    the output file. The blank line after [END TEMPLATE INSTRUCTIONS] is also
    The output file must not contain any TEMPLATE INSTRUCTIONS markers.
    The first line of docs/LORE_ENCHIRIDION.md must be the "# LORE_ENCHIRIDION" heading.

Write the file to docs/LORE_ENCHIRIDION.md.
Validate it exists before moving on.

---

# Step 7 - Produce LORE_INDEX.md

Load lore/templates/ENTRY_README_template.md as your scaffolding.
Replace every placeholder with real content.
The output is a finished document.

## Content rules

    - Audience: anyone arriving at the docs/ directory for the first time.
    - Purpose: orient the reader and point them to the right document.
    - List every document in docs/ with a one-line description of what it contains.
      Include any pre-existing user files noted in Step 4.
    - Keep it short. This is a directory guide, not documentation itself.
    - No source tags required in this file - it contains no factual claims
      about the codebase, only navigation guidance.
    - Use only ASCII characters.

Write the file to docs/LORE_INDEX.md.
Validate it exists before moving on.

---

# Step 8 - Produce LORE_PRIMER files

## PRIMER eligibility gate

Before producing any PRIMER, verify in ANALYSIS.md that the subsystem has:
    - analysis_complete: YES
    - disposition: active

A PRIMER must never be produced for a subsystem where analysis_complete is NO
or where disposition is deprecated or ignored.

If a PRIMER-warranted subsystem has analysis_complete NO:
    Do not produce a PRIMER for it.
    In the ENCHIRIDION subsystem catalog entry for that subsystem, write:
        [ANALYSIS INCOMPLETE - PRIMER not produced. Run stage_2c_subsystem_completion.md
         to complete analysis, then regenerate documentation.]
    Note this in JOURNAL.md last_error as MINOR.

If a subsystem has disposition: deprecated:
    Do not produce a PRIMER for it.
    In the ENCHIRIDION subsystem catalog, write a stub entry:
        ## [Subsystem Name] (deprecated)
        status: deprecated
        superseded_by: [name of replacement subsystem, or "none identified"]
        retained_because: [why it still exists in the codebase, or "unknown [inferred]"]
        note: this subsystem is being replaced and is excluded from full documentation.
    Include a single row in the OVERVIEW subsystems table marked "(deprecated)".
    Do not produce a GUIDE for this subsystem.

If a subsystem has disposition: ignored:
    Do not produce any entry for it in any output document.
    Do not produce a PRIMER, GUIDE, or catalog entry.
    It must be completely absent from all output.

## PRIMER batching

If more than 5 PRIMERs are warranted, do not attempt to produce all of them
in a single pass. Produce them in batches:

    Batch 1: the 5 highest-scoring PRIMERs (alphabetical for ties).
    Batch 2+: remaining PRIMERs in subsequent sub-agent delegations or
              a follow-up stage 4 pass in a fresh context window.

For each PRIMER produced, immediately record it in JOURNAL.md Artifacts as
EXISTS before moving to the next one. This ensures partial progress is
preserved if context budget is exhausted mid-batch.

If delegating to sub-agents: each sub-agent should receive the ANALYSIS.md
subsystem entry for its assigned subsystem, the resolved question answers
from Step 1, and the PRIMER template. It should produce only its one PRIMER
file and nothing else.

After all batches are complete, verify that every expected PRIMER exists
before writing the handoff note.

---

For each PRIMER target identified in Step 3, in order of PRIMER score
(highest score first, alphabetical by name for ties):

    Load lore/templates/PRIMER_template.md as your scaffolding.
    Replace every placeholder with real content for this specific subsystem.

    ## Source material per PRIMER

        Subsystem entry:    ANALYSIS.md Subsystems section for this subsystem
        Key files:          ANALYSIS.md Key Files and Functions Directory
                            (filtered to files in this subsystem)
        Dependencies:       ANALYSIS.md Subsystems entry - external dependencies
        Integration points: ANALYSIS.md Subsystems entry - integration points
        Human answers:      resolved Q values from Step 1

    ## Content rules

        - Audience: a developer working on or integrating with this specific
          subsystem. May or may not have read the ENCHIRIDION.
        - Depth: deeper than ENCHIRIDION entries. Include usage examples
          where the analysis supports them, using [inferred] if not sourced.
        - If a subsystem has important configuration values, list them
          with their expected types and defaults if known.
        - Every factual claim must be tagged [src: filename] or [inferred].
        - Where ANALYSIS.md has [INCOMPLETE] markers, propagate them.
        - Use only ASCII characters.

    ## Template comment stripping

        The PRIMER template begins with a [TEMPLATE INSTRUCTIONS] block.
        This block starts with the line "[TEMPLATE INSTRUCTIONS - remove this
        entire block before writing output]" and ends with the line
        "[END TEMPLATE INSTRUCTIONS]".
        Remove this entire block, inclusive of both marker lines, before writing
        each PRIMER output file. The blank line after [END TEMPLATE INSTRUCTIONS]
        is also removed. The output file must not contain any TEMPLATE
        INSTRUCTIONS markers. The first line of each docs/LORE_*_PRIMER.md
        file must be the "# [Subsystem Name] - PRIMER" heading.

    Filename: docs/LORE_PRIMER_[subsystem_name_lowercase_underscored].md
    Example: docs/LORE_PRIMER_auth.md

    Only produce this file if the subsystem has analysis_complete YES
    and disposition: active.
    If analysis_complete is NO, skip and write the ENCHIRIDION marker instead.
    If disposition is deprecated or ignored, skip entirely per the eligibility
    gate rules above.

    Write the file. Validate it exists before moving to the next PRIMER.

If there are no PRIMER targets, skip this step entirely.

---

# Step 8b - Produce LORE_GUIDE files

For each GUIDE target identified in Step 3, in order of subsystem name
(alphabetical):

    Load lore/templates/GUIDE_template.md as your scaffolding.
    Replace every placeholder with real content for this specific subsystem.

    ## Source material per GUIDE

        Subsystem entry:        ANALYSIS.md Subsystems section for this subsystem
        runnable_entry_points:  ANALYSIS.md user_runnable fields
        operator_prerequisites: ANALYSIS.md user_runnable fields
        known_operator_issues:  ANALYSIS.md user_runnable fields and Observed Bugs
        Key files:              ANALYSIS.md Key Files directory
                                (filtered to operator-relevant files)
        Human answers:          resolved Q values from Step 1

    ## Content rules

        - Audience: an operator or developer who needs to RUN this subsystem,
          not just read its code. Assume they understand the project at a high
          level but may not know the internals.
        - Tone: plain language. Imperative where giving instructions.
          Short sentences. No jargon without definition.
        - Cover the whole subsystem in one document. Do not produce one GUIDE
          per executable - group all executables and their workflows here.
        - Every factual claim must be tagged [src: filename] or [inferred].
        - Where ANALYSIS.md has [INCOMPLETE] markers relevant to operation,
          propagate them with a note: [INCOMPLETE - verify before running].
        - Use only ASCII characters.

    ## Template comment stripping

        The GUIDE template begins with a [TEMPLATE INSTRUCTIONS] block.
        Remove this entire block, inclusive of both marker lines, before
        writing the output file. The first line of each LORE_GUIDE_*.md
        file must be the "# [Subsystem Name] - Operator Guide" heading.

    Filename: docs/LORE_GUIDE_[subsystem_name_lowercase_underscored].md
    Example: docs/LORE_GUIDE_deploy.md

    Write the file. Validate it exists before moving to the next GUIDE.

If there are no GUIDE targets, skip this step entirely.

---

# Step 9 - Update JOURNAL.md

Update docs/.LORE/work/JOURNAL.md:

    last_updated: current timestamp (YYYY-MM-DD HH:MM)
    last_stage_completed: stage_4_lore.md
    status: IN_PROGRESS

    Completed Stages: append one of:
        - stage_4_lore.md: DONE
        - stage_4_lore.md: PARTIAL - [brief reason, list missing docs]

    Artifacts section: overwrite in full with all files and their states.
    List all docs/ output files. Mark each EXISTS, PARTIAL, or MISSING.
    List PRIMER files as LORE_PRIMER_[name].md and GUIDE files as
    LORE_GUIDE_[name].md.

    If any Tier 1 questions were unanswered and produced [AWAITING ANSWER]
    markers: note in docs/.LORE/work/JOURNAL.md which documents contain these
    markers and which question IDs they correspond to.

    Update docs/.LORE/work/MANIFEST.md:
        - Set the mtime_available field to YES if bash ls -la was available
          during stage 2, or NO if all mtimes in MANIFEST.md are "unknown".
          Check the existing mtime values in the Files Read section to determine
          this: if every entry has mtime "unknown", set NO; otherwise set YES.
        - Write or overwrite the Preserved Files section with every file
          currently in docs/ that is NOT a canonical LORE output file
          (i.e. every file whose name does not match LORE_*.md).
          Format each entry as: - [filepath]: [approximate line count or "empty"]
          If no such files exist, write: none

---

# Step 10 - Validate Output

Before writing the handoff, verify each produced document:

    [ ] docs/LORE_ABSTRACT.md exists and is not empty
    [ ] docs/LORE_ENCHIRIDION.md exists and is not empty
    [ ] docs/LORE_INDEX.md exists and is not empty
    [ ] Every expected LORE_PRIMER_[name].md file exists (if any were warranted)
    [ ] Every expected LORE_GUIDE_[name].md file exists (if any were warranted)
    [ ] No PRIMER or GUIDE file uses the old LORE_[name]_PRIMER.md naming pattern
    [ ] No template scaffolding markers remain in any output file
        (check for strings like "[PLACEHOLDER]", "[INSERT", "{{", "}}")
    [ ] No unicode characters or emojis appear in any output file
    [ ] LORE_ENCHIRIDION.md contains [src:] tags on factual claims
    [ ] LORE_ABSTRACT.md contains [src:] tags on factual claims
    [ ] LORE_ABSTRACT.md does not contain raw template syntax
    [ ] MANIFEST.md mtime_available field is present and set to YES or NO
    [ ] MANIFEST.md Preserved Files section is present
    [ ] JOURNAL.md last_stage_completed is stage_4_lore.md

For each failed check: attempt to fix it inline before writing the handoff.
If a check cannot be fixed, record it in JOURNAL.md last_error and mark
the relevant artifact as PARTIAL.

---

# Step 11 - Write the Handoff Note

Write the following block exactly, filling in the bracketed fields.
Copy it into the Last Handoff Note section of JOURNAL.md.
Output it as the final output of this stage.

---
Stage 4 complete - Documentation Production
Codebase: [total_source_files] files ([languages])
LORE: Stage [4/5] Pass [1] Context [N]
Status: COMPLETE - documentation produced, proceed to Stage 5 QA

Documents produced:
    - docs/LORE_ABSTRACT.md:          [EXISTS | PARTIAL]
    - docs/LORE_ENCHIRIDION.md:       [EXISTS | PARTIAL]
    - docs/LORE_INDEX.md:             [EXISTS | PARTIAL]
    [for each PRIMER produced:]
    - docs/LORE_PRIMER_[name].md:     [EXISTS]
    [for each GUIDE produced:]
    - docs/LORE_GUIDE_[name].md:      [EXISTS]
    [for each warranted subsystem where analysis_complete NO:]
    - docs/LORE_PRIMER_[name].md:     NOT PRODUCED - analysis incomplete
    [for each deprecated subsystem:]
    - docs/LORE_PRIMER_[name].md:     NOT PRODUCED - deprecated (stub in ENCHIRIDION)
    [for each ignored subsystem:]
    - [name]:                         EXCLUDED - ignored per operator instruction

Unanswered Tier 1 questions that left markers in output: [count or "none"]
Tokens this context: ~[N]k (estimated)
Remaining tokens (est.): [low]k-[high]k across [N] more contexts

Please start a new context with this text:

Please follow `lore/LORE.md` and CONTINUE.
---

---

# DO NOT

- Do not invent facts not supported by ANALYSIS.md, SKELETON.md, or resolved
  question answers
- Do not remove [INCOMPLETE] or [AWAITING ANSWER] markers - they are
  intentional signals to the human and to stage 5
- Do not modify any file in lore/ other than docs/.LORE/work/JOURNAL.md
- Do not delete pre-existing files in docs/ that are not canonical LORE output
- Do not produce PRIMER files for subsystems with a score below 2
- Do not produce GUIDE files for subsystems where operator_facing is NO and
  dev_entry_points is empty
- Do not use the old LORE_[name]_PRIMER.md naming pattern for any output file
- Do not produce a PRIMER for any subsystem where analysis_complete is NO
- Do not omit source tags from LORE_ENCHIRIDION.md
- Do not use unicode characters or emojis in any output
- Do not proceed to stage 5 in this same context window
  (output the handoff note and stop)