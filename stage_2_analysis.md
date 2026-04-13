# Stage 2 - Deep Analysis
# Read this file and execute it exactly. Do not skip steps.
# Do not begin writing output until Step 6.
# Pipeline reference: lore/LORE.md



---

# Purpose

This stage reads the files flagged in docs/.LORE/work/SKELETON.md in priority order.
It produces docs/.LORE/work/ANALYSIS.md, which is the primary knowledge artifact
consumed by stage 4 (LORE production).

It also consolidates all human questions from SKELETON.md into
docs/.LORE/work/QUESTIONS.md, adding any new questions discovered during deep reading.

Token discipline is critical. Read files in priority order. Stop reading
when the context budget approaches its limit. Mark incomplete sections clearly.
Stage 2b exists to handle overflow.

---

# Prerequisites

Before executing this stage:

1. Read lore/SCHEMAS.md in full. You must know the exact schemas for
   ANALYSIS.md, QUESTIONS.md, and JOURNAL.md before writing anything.

2. Read docs/.LORE/work/JOURNAL.md. Verify:
   - status is IN_PROGRESS
   - last_stage_completed is stage_1_recon.md
   If status is not IN_PROGRESS, stop and output:
   "Stage 2 prerequisite failed: JOURNAL.md status is [status]. Expected IN_PROGRESS."

3. Read docs/.LORE/work/SKELETON.md in full. This is your primary input.
   Pay close attention to:
   - Files Flagged for Deep Read (your reading list, in order, with read policy)
   - Resource Estimate (sets expectations for how many passes are needed)
   - Subsystem Candidates (drives the Subsystems section of ANALYSIS.md)
   - Human Questions (consolidated into QUESTIONS.md)
   - Project Identity (drives the header of ANALYSIS.md)

---

# Step 1 - Establish Context Budget

Before reading any flagged file, estimate your remaining context budget.

Define two thresholds based on file count, which is concrete and observable:

    CAUTION threshold: you have started reading more than 15 files
        At this point: finish the current file, then do not start another.
        Proceed to Step 6 with completion_status PARTIAL.

    STOP threshold: you have started reading more than 20 files
        At this point: stop immediately, even mid-file.
        Write what you have, mark everything incomplete, proceed to Step 6.

Check the file count after each file is read.
Never start reading a new file if you are past the CAUTION threshold.

Note: these are conservative defaults. Smaller files consume less context
than large ones. If you are reading many small files (under 50 lines each),
you may extend the CAUTION threshold to 20 files and STOP to 25. If you are
reading large files (over 200 lines each), apply the thresholds more strictly.

HEAD-only files (policy HEAD in SKELETON.md) do not count toward the file
count thresholds. They consume minimal context and may be read at any point.

---

# Step 2 - Read Files in Priority Order

Read each file in the order listed under "Files Flagged for Deep Read"
in docs/.LORE/work/SKELETON.md. The list is already ordered by PRIMER score
(highest-score subsystems first). Do not reorder it.

## File read policy enforcement

Before reading each file, check its assigned read policy from SKELETON.md:

    FULL  - read the entire file. Extract all details below.
    HEAD  - read the first 20 lines only. Record the file in the
            inventory as a data file. Do not extract function details.
            Do not document any content beyond what the first 20 lines reveal.
    SKIP  - do not read. Do not record in any inventory.

No documentation is ever produced for a file that was not read with policy FULL.
If a file was read HEAD only, its entry in the file inventory is limited to:
    - [filepath]: [data file | generated file] - HEAD only: [what the first 20 lines revealed]

For each FULL file you read, extract and record the following. Keep notes in
working memory until Step 6 when you write ANALYSIS.md.

    a) What does this file do? (one paragraph max)
    b) What subsystem does it belong to? (match to SKELETON.md candidates)
    c) Key exported functions, classes, routes, or constants
       (name + one-line description for each)
    d) Notable imports - what does this file depend on?
    e) Any data structures, schemas, or type definitions visible
    f) Any external service calls (HTTP, database, cache, queue)
    g) Any configuration values loaded or expected
    h) Any hardcoded values, magic numbers, or obvious technical debt
    i) Any error handling patterns observed
    j) Anything that contradicts a claim in SKELETON.md

Do not summarize away important details. If a function's signature or
a schema definition is short enough to quote verbatim, quote it.
Tag every note [src: filename] or [src: filename:line] for specific lines.

## Reading order within a subsystem

When the flagged list contains multiple files from the same subsystem,
read all of them before moving to the next subsystem. Completing one
subsystem at a time is better than partial coverage of many subsystems,
because a PRIMER cannot be produced for a subsystem until all its files
are read. Track the running count for each subsystem: files_read / files_total.

## Transitive imports for config and schema files

While reading flagged files, you will encounter imports of other files.
Apply this rule unconditionally:

    If a flagged file imports another file that is a config, schema,
    constants, or globals file - and that file has not already been read -
    read it immediately before continuing.

    This rule applies regardless of how the file is named or where it lives.
    Config files whose purpose is unclear from their name are especially
    important to read, because misunderstanding their roles leads to
    documentation errors that are hard to catch later.

    Do not skip a config or schema import because it appears to contain
    secrets. Instead, read it for structure and key names only:
        - Record every key name and its inferred type.
        - Do not record the values of fields whose names suggest credentials
          (patterns: password, secret, key, token, apiKey, privateKey,
          credentials, auth, pass, pwd, connectionString, dsn).
        - Note in your working memory: "values redacted - credential fields"
          for any field whose value was skipped.
        - Tag the file [src: filename] as normal.

    Add any such file to your read list and note it was read transitively.

---

# Step 3 - Assess Subsystem Coverage

After reading all files you can fit, evaluate your coverage of each
subsystem candidate from SKELETON.md.

For each candidate:
    - How many of its key files did you read? (N of M files)
    - Is the subsystem's purpose now clear, partially clear, or still opaque?
    - Should its PRIMER score be revised up or down? State your reasoning.
    - Does it have integration points not visible from structure alone?

If a subsystem is still opaque after reading its files, flag it in
ANALYSIS.md and add a Tier 2 question to QUESTIONS.md asking the human
to clarify its purpose.

For each subsystem, assess the following fields:

    disposition: active | deprecated | ignored
        active     - normal treatment; full analysis and documentation.
        deprecated - subsystem still exists but is being replaced. Infer
                     from README comments, Q13-style human answers, or
                     directory names like "legacy", "old", "archive".
                     A stub entry is produced; no PRIMER.
        ignored    - foreign code, vendored samples, or unrelated projects
                     that should not be documented. Infer from Q13-style
                     human answers or obvious non-project directories.
                     Excluded from all output. Noted in JOURNAL.md only.
        Default: active. If uncertain, use active and add a Tier 3 question.

    operator_facing: YES | NO | MAYBE
        YES   - the subsystem's primary purpose is to be run directly by
                an operator or developer. The whole directory is a tool
                collection, deployment system, or admin executable set.
                A GUIDE will be produced. Examples: srvr/tools/, deploy/
        NO    - internal only; not directly invoked by humans.
        MAYBE - unclear from files read; add a Tier 3 question to QUESTIONS.md.

    dev_entry_points: [list] or "none"
        Admin executables, initializers, connectivity tests, or one-off
        migration runners that are not the subsystem's main function but
        may be invoked directly. A GUIDE section is produced for these
        even if operator_facing is NO.

Also track files_read as a ratio for each subsystem:
    files_read: [N read so far] of [M total files in subsystem]
    analysis_complete: YES if N equals M, NO otherwise.
    If the directory file count is uncertain, default to NO until confirmed.

A PRIMER must NOT be produced for a subsystem where analysis_complete is NO.
If this pass cannot complete a subsystem, its ANALYSIS.md entry is written
with analysis_complete: NO and the subsystem is picked up by stage 2b or 2c.

---

# Step 4 - Identify Observed Bugs and Risks

While reading, maintain a running list of problems. Do not hunt for bugs -
only record what you naturally observe during reading.

Classify each by severity:

    HIGH    - likely to cause data loss, security breach, or outage
    MEDIUM  - likely to cause incorrect behavior under normal use
    LOW     - code smell, maintainability concern, or edge case gap

For each problem, record:
    - severity
    - file and approximate line number [src: file:line]
    - plain description of the problem
    - brief suggestion for how it might be fixed

If nothing is observed, record "none observed" explicitly.
Do not fabricate or speculate about bugs that are not directly visible.

---

# Step 5 - Generate Additional Questions

Review your notes from reading. Identify any new questions that:
    a) could not have been identified from structure alone (stage 1 would have caught them)
    b) materially affect the accuracy of documentation

Apply the same tier classification rules as stage 1:
    TIER 1: blocking
    TIER 2: enhancing (include a default assumption)
    TIER 3: cosmetic

Do not duplicate questions already in SKELETON.md.
Continue the Q-number sequence from where SKELETON.md left off.

Apply the same discipline: resist more than 2 new Tier 1 questions from this
stage. If code is ambiguous, prefer a Tier 2 question with a reasonable default.

---

# Step 6 - Write Output Files

DO NOT begin writing until Steps 2 through 5 are complete (or context budget
was reached). Write all output files in this order:

## Write docs/.LORE/work/ANALYSIS.md

Use the ANALYSIS.md schema from lore/SCHEMAS.md exactly.

Set completion_status at the top:
    - COMPLETE if all flagged files were read
    - PARTIAL if context budget was reached before finishing

### Theory of Operation
Write a narrative of what the application does and how it works end to end.
Audience: technically literate reader who has not seen the codebase.
Length: 200 to 500 words.
Tag every factual claim [src: filename] or [inferred].
Do not include claims you cannot support.

### Subsystems
One entry per subsystem candidate from SKELETON.md.
If a subsystem had files read: fill all fields from your notes.
    - Set files_read to "N of M" where N is the count of files you read
      and M is the total file count for this subsystem from SKELETON.md.
    - Set analysis_complete to YES if N equals M, NO if N is less than M.
    - Populate file_inventory with one entry per file read (FULL or HEAD).
      FULL reads get full detail. HEAD reads get a one-liner only.
      Do not create entries for files not yet read.

If a subsystem had no files read: write a minimal entry:
    files_read: 0 of M
    analysis_complete: NO
    file_inventory: none read this pass
    [all other fields]: [NOT READ - stage 2b or 2c will complete this subsystem]

Include the revised PRIMER score with justification.
Include disposition, operator_facing, and dev_entry_points assessments from Step 3.

### Data Paradigm
Describe how data is structured, stored, and moved.
Covers: database type, schema approach, data models, data flow between layers.
If this could not be determined: write [INCOMPLETE - no data layer files read]
and flag it in Incomplete Sections.

### IT Paradigm
Describe infrastructure and deployment model.
Covers: runtime, containerization, cloud services, process management, networking.
Draw from any Dockerfile, docker-compose, config files, or README claims.
Tag all claims.

### Code Style and Organization
Describe conventions observed in the codebase.
Covers: naming, module patterns, async patterns, error handling, test conventions.
Base this only on files actually read. Do not generalize from one file alone.

### Key Files and Functions Directory
Curated reference list. Only entries important enough to appear in docs.
Not exhaustive. Format from SCHEMAS.md.

### Observed Bugs and Risks
From your Step 4 running list. Format from SCHEMAS.md.
If none: write "none observed".

### Incomplete Sections
Only include this section if completion_status is PARTIAL.
List every section or subsystem that is incomplete and why.

## Write docs/.LORE/work/QUESTIONS.md

Write the complete consolidated QUESTIONS.md using the schema from SCHEMAS.md.

Merge questions from:
    - SKELETON.md (Tier 1, Tier 2, Tier 3 sections) - copy verbatim, preserve Q-numbers
    - Your Step 5 new questions - append after the last SKELETON.md question

If QUESTIONS.md already exists (from a prior partial run), read it first
and preserve any ANSWER fields that are already filled in.

Preserve Q-number order within each tier. Tier 1 first, then Tier 2, then Tier 3.

## Write docs/.LORE/work/MANIFEST.md

Write docs/.LORE/work/MANIFEST.md using the MANIFEST.md schema from lore/SCHEMAS.md.

    manifest_produced: current timestamp (YYYY-MM-DD HH:MM)

    ### Files Read
    One entry per file read during Step 2 of this stage.
    For each file:
        - Obtain the file's mtime using bash ls -la if available.
          If bash is not available, write "unknown" for mtime.
        - Assign the subsystem by matching the file against the key files
          lists in the Subsystem Candidates section of SKELETON.md.
          If no match, write "none".
        - Write stage as "2".

    ### Manual Overrides
    Write: none

If MANIFEST.md already exists (from a prior partial run), overwrite it in full.
The manifest always reflects the complete set of files read across all passes
completed so far. Re-read the prior manifest before overwriting and carry
forward any entries for files read in previous passes of this stage.

## Update docs/.LORE/work/SKELETON.md

If Step 2 found any contradictions between observed code and SKELETON.md claims:
    Append a Corrections section to SKELETON.md (do not edit existing sections).
    Use the Corrections format from the SKELETON.md schema in SCHEMAS.md.

If no contradictions were found: do not modify SKELETON.md.

---

# Step 7 - Update JOURNAL.md

Update docs/.LORE/work/JOURNAL.md:

    last_updated: current timestamp (same method as stage 1)
    last_stage_completed: stage_2_analysis.md
    files_read_total: [cumulative count of FULL reads completed this pass;
        add to any prior value already in this field from a previous pass]

    If completion_status is COMPLETE:
        status: IN_PROGRESS  (pipeline continues)
        Completed Stages: append "- stage_2_analysis.md: DONE"

    If completion_status is PARTIAL:
        status: IN_PROGRESS  (stage 2b will continue)
        Completed Stages: append "- stage_2_analysis.md: PARTIAL - [brief reason]"

    Artifacts section: overwrite with current state of all known files.
    Include docs/.LORE/work/ANALYSIS.md as EXISTS or PARTIAL.
    Include docs/.LORE/work/QUESTIONS.md as EXISTS.
    Include docs/.LORE/work/MANIFEST.md as EXISTS.
    Include all expected final output docs as MISSING.

    Unanswered Questions: overwrite with all Tier 1 questions from QUESTIONS.md
    that have blank ANSWER fields.

---

# Step 8 - Validate Output

Before writing the handoff note, verify:

    [ ] docs/.LORE/work/ANALYSIS.md exists
    [ ] completion_status field is present and set to COMPLETE or PARTIAL
    [ ] Theory of Operation is 200-500 words and every claim is tagged
    [ ] Every subsystem from SKELETON.md has an entry in ANALYSIS.md
    [ ] Observed Bugs and Risks section is present (even if "none observed")
    [ ] docs/.LORE/work/QUESTIONS.md exists with all tiers present
    [ ] No question from SKELETON.md was dropped
    [ ] Q-numbers are unique and sequential
    [ ] docs/.LORE/work/MANIFEST.md exists with at least one entry in Files Read
    [ ] JOURNAL.md last_stage_completed is stage_2_analysis.md

If any check fails: fix it before writing the handoff note.
If a check cannot be fixed, note it in JOURNAL.md last_error and continue.

---

# Step 9 - Write the Handoff Note

If completion_status is COMPLETE, write this block exactly:

---
Stage 2 complete - Deep Analysis
Codebase: [total_source_files] files ([comma-separated languages])
LORE: Stage [2/5] Pass [1] Context [2]
Status: COMPLETE - all flagged files read, proceed to Stage 2c

Subsystems fully analyzed: [count of analysis_complete YES]
Subsystems still incomplete: [count of analysis_complete NO] ([names])
Files read this pass: [N] of [total_source_files] total
Tokens this context: ~[N]k (estimated)
Remaining tokens (est.): [low]k-[high]k across [N] more contexts

Please start a new context with this text:

Please follow `lore/LORE.md` and CONTINUE.
---

If completion_status is PARTIAL, write this block instead:

---
Stage 2 partial - Deep Analysis (budget reached)
Codebase: [total_source_files] files ([comma-separated languages])
LORE: Stage [2/5] Pass [1] Context [2]
Status: PARTIAL - context budget reached, continue with Stage 2b

Subsystems fully analyzed this pass: [count] ([names])
Subsystems incomplete: [count] ([names])
Files read this pass: [N of flagged]
Next file: [filename]
Tokens this context: ~[N]k (estimated)
Remaining tokens (est.): [low]k-[high]k across [N] more contexts

Please start a new context with this text:

Please follow `lore/LORE.md` and CONTINUE.
---

Copy the handoff note into the Last Handoff Note section of JOURNAL.md.
Then output it as the final output of this stage.

---

# DO NOT

- Do not read files in node_modules/, .git/, dist/, build/, or target/
- Do not read files not on the flagged list unless they are imported by a
  flagged file and the import cannot be understood without reading it
  (in that case, read only the relevant export, not the entire file)
- Do not start a new file read after passing the CAUTION context threshold
- Do not answer the human gate questions yourself
- Do not write any files in docs/ - that is stage 4's job
- Do not produce the final documentation in this stage
- Do not modify existing sections of SKELETON.md - only append Corrections
- Do not assign function documentation to any file read with policy HEAD
- Do not create file inventory entries for files that were not read
- Do not produce a PRIMER entry for any subsystem where analysis_complete is NO
- Do not proceed to stage 3 in this same context window
  (output the handoff note and stop)