# Stage 2c - Subsystem Completion Gate
# Read this file only after stage_2_analysis.md (and any stage_2b passes) complete.
# Execute it exactly. Do not skip steps.
# Pipeline reference: lore/LORE.md



---

# Purpose

This stage is the quality gate between analysis and documentation production.

Its only job is to ensure that every PRIMER-warranted subsystem has
analysis_complete: YES before the pipeline advances to stage 3.

A PRIMER cannot be produced for a subsystem until every file in that subsystem
has been read with policy FULL (or acknowledged as HEAD/SKIP where appropriate).
This stage enforces that guarantee. It reads files, updates ANALYSIS.md, and
loops until all warranted subsystems are complete.

Stage 2c may be run multiple times. Each pass picks up where the last left off.
It does not re-read files already in the file_inventory. It does not rewrite
sections already written. It only adds.

---

# Prerequisites

Before executing this stage:

1. Read lore/SCHEMAS.md in full. You must know the exact schemas for
   ANALYSIS.md, SKELETON.md, and JOURNAL.md before writing anything.

2. Read docs/.LORE/work/JOURNAL.md. Verify:
   - status is IN_PROGRESS
   - last_stage_completed is stage_2_analysis.md, stage_2b_continuation.md,
     or stage_2c_subsystem_completion.md (a prior pass of this stage)

   If last_stage_completed is stage_3_human_gate.md or later, this stage
   has already been passed. Output and stop:
       "Stage 2c skipped: pipeline is already past the analysis phase."

3. Read docs/.LORE/work/ANALYSIS.md in full. For every subsystem entry, record:
   - subsystem name
   - PRIMER warranted (YES or NO)
   - analysis_complete (YES or NO)
   - files_read (N of M)
   - the list of files already in file_inventory

4. Read docs/.LORE/work/SKELETON.md in full. For every subsystem candidate, record:
   - the full list of files belonging to that subsystem
   - the read policy (FULL | HEAD | SKIP) for each file

---

# Step 1 - Assess Completion State

State the following explicitly before doing anything else:

    PRIMER-warranted subsystems: [count]
    [For each warranted subsystem:]
        - [name]: analysis_complete [YES | NO], files_read [N of M]
          [If NO:] files remaining: [list unread files with their read policy]

    Subsystems already complete: [count]
    Subsystems still incomplete: [count]

If all PRIMER-warranted subsystems have analysis_complete YES:

    Output:
        All PRIMER-warranted subsystems are complete.
        Stage 2c is not needed.
    Update JOURNAL.md: append "- stage_2c_subsystem_completion.md: SKIPPED - all subsystems already complete"
    Proceed to Step 5 (write handoff note for stage 3).
    Stop.

If any PRIMER-warranted subsystem has analysis_complete NO:
    Continue to Step 2.

---

# Step 2 - Build the Read List

For each PRIMER-warranted subsystem where analysis_complete is NO:

    Compare the full file list from SKELETON.md against the file_inventory
    entries already present in ANALYSIS.md for that subsystem.

    Files that appear in the SKELETON.md subsystem file list but do NOT
    appear in the ANALYSIS.md file_inventory are unread. These are your
    targets for this pass.

    Order the read list:
        1. Complete the highest-PRIMER-score incomplete subsystem first.
           Read all its unread files before moving to the next subsystem.
        2. Then move to the next-highest-score incomplete subsystem.
        3. Continue until context budget is reached or all files are read.

State the read list explicitly:

    Files to read this pass (in order):
    [For each file:]
        [N]. [filepath] ([subsystem name]) - policy: [FULL | HEAD | SKIP]

Monitor context budget after each file. If CAUTION threshold (15 FULL files
read this pass) is reached: finish the current file and stop reading further.

---

# Step 3 - Read Files and Update ANALYSIS.md

For each file in the read list, in order:

## Reading

Apply the file's read policy from SKELETON.md:

    FULL  - read the entire file.
    HEAD  - read the first 20 lines only.
    SKIP  - do not read. Mark in the inventory as:
              [filepath]: SKIP - lock file / compiled output / not analyzed

For FULL reads, extract:
    a) What does this file do? (one paragraph max)
    b) What subsystem does it belong to?
    c) Key exported functions, classes, routes, or constants
       (name + one-line description for each)
    d) Notable imports - what does this file depend on?
    e) Any data structures, schemas, or type definitions visible
    f) Any external service calls (HTTP, database, cache, queue)
    g) Any configuration values loaded or expected
    h) Any hardcoded values, magic numbers, or obvious technical debt
    i) Any error handling patterns observed
    j) Anything that contradicts existing claims in ANALYSIS.md

For HEAD reads, record only:
    - what the file appears to be (data file, lookup table, fixture, etc.)
    - the first recognizable field or entry name if visible

No documentation is ever produced for a file that was not read with
policy FULL. HEAD files get a one-line inventory entry only.

Tag every extracted fact [src: filename] or [src: filename:line].

## Updating ANALYSIS.md

After reading each file, immediately update the relevant subsystem entry:

    Update ANALYSIS.md IN PLACE for the relevant subsystem entry:

    Add to file_inventory directly in the existing block (do not create
    a separate ADDENDUM block):
        FULL read:
            - [filepath]: [one-line description of what the file does] [src: filepath]
              key functions:
                - [name]([params if known]): [one-line description] [src: filepath]
              [If no notable exports:]
                none identified [src: filepath]

        HEAD read:
            - [filepath]: [data file | generated file] - HEAD only: [what first 20 lines revealed]

        SKIP:
            - [filepath]: SKIP - [reason]

    Update files_read count IN PLACE (overwrite the prior value):
        files_read: [new N] of [M] [updated: stage_2c pass N]

    Update analysis_complete IN PLACE:
        If new N equals M: set analysis_complete to YES
        Otherwise: leave as NO

    Do not append ADDENDUM blocks. ANALYSIS.md must remain a single
    coherent reference document, not a log of diffs.

If any file read contradicts an existing claim in ANALYSIS.md:
    Append a CORRECTION note to SKELETON.md (same format as stage 2 corrections).
    Update the contradicted claim in ANALYSIS.md inline with a note:
        [corrected by stage_2c: see SKELETON.md Corrections]

If any file read raises a new question:
    Add it to QUESTIONS.md (same tier rules as stage 2, continue Q-numbering).

---

# Step 4 - Update MANIFEST.md and JOURNAL.md

## Update MANIFEST.md

For each file read in this pass (FULL or HEAD):
    If the file already has an entry in MANIFEST.md: update its entry.
    If new: append an entry.

    Format:
        - [filepath]: mtime [timestamp or "unknown"] | subsystem [name] | stage 2c

Update the manifest_produced field to the current timestamp.

If any HEAD reads occurred:
Set file_read_policy_applied to HEAD_APPLIED.
Otherwise leave it at STANDARD (or HEAD_APPLIED if already set from prior pass).

## Update JOURNAL.md

last_updated: current timestamp
last_stage_completed: stage_2c_subsystem_completion.md
files_read_total: [add the count of FULL reads completed this pass
    to the existing files_read_total value in JOURNAL.md]

    Append to Completed Stages:
        - stage_2c_subsystem_completion.md: [DONE | PARTIAL] - pass [N]
          [N subsystems complete, M subsystems remaining]

    Update Artifacts section with current state of all files.

    If any new Tier 1 questions were added: update Unanswered Questions section.

---

# Step 5 - Determine Exit Condition

Count PRIMER-warranted subsystems where analysis_complete is NO.

If count is 0 (all warranted subsystems are complete):
    Set exit_condition: COMPLETE
    Update JOURNAL.md completion_status note:
        "All PRIMER-warranted subsystems have analysis_complete YES.
         Pipeline may advance to stage 3."

If count is greater than 0 (some subsystems still incomplete):
    Set exit_condition: PARTIAL
    If this was caused by context budget:
        Note: another pass of stage_2c is required.
    If this was caused by files not appearing in any subsystem list:
        Note: review SKELETON.md subsystem candidate file lists for accuracy.

---

# Step 6 - Validate Output

Before writing the handoff note, verify:

    [ ] Every file read this pass appears in the relevant subsystem's
        file_inventory in ANALYSIS.md
    [ ] No file inventory entry exists for a file that was not read
    [ ] files_read counts are accurate (count the inventory entries)
    [ ] analysis_complete is YES for every subsystem whose files_read N equals M
    [ ] MANIFEST.md has been updated with all files read this pass
    [ ] JOURNAL.md last_stage_completed is stage_2c_subsystem_completion.md
    [ ] No HEAD-read file has function documentation in its inventory entry
    [ ] No SKIP file appears in any file inventory

If any check fails: fix it before writing the handoff note.

---

# Step 7 - Write the Handoff Note

If exit_condition is COMPLETE:

    ---
    Stage 2c complete - Subsystem Completion Gate
    Codebase: [total_source_files] files ([languages])
    LORE: Stage [2/5] Pass [2c] Context [N]
    Status: COMPLETE - all PRIMER-warranted subsystems fully analyzed

    Subsystems complete: [count] ([names])
    Files read this pass: [count]
    Total files read across all analysis passes: [count] of [total_source_files]
    Tokens this context: ~[N]k (estimated)
    Remaining tokens (est.): [low]k-[high]k across [N] more contexts

    Please start a new context with this text:

    Please follow `lore/LORE.md` and CONTINUE.
    ---

If exit_condition is PARTIAL:

    ---
    Stage 2c partial - Subsystem Completion Gate (budget reached)
    Codebase: [total_source_files] files ([languages])
    LORE: Stage [2/5] Pass [2c] Context [N]
    Status: PARTIAL - [count] subsystems still incomplete, run Stage 2c again

    Subsystems complete: [count] ([names])
    Subsystems still incomplete: [count]
    [For each incomplete subsystem:]
        - [name]: [files_read N of M], next file: [filename]
    Tokens this context: ~[N]k (estimated)
    Remaining tokens (est.): [low]k-[high]k across [N] more contexts

    Please start a new context with this text:

    Please follow `lore/LORE.md` and CONTINUE.
    ---

Copy the handoff note into the Last Handoff Note section of JOURNAL.md.
Output it as the final output of this stage.

---

# DO NOT

- Do not read files in node_modules/, .git/, dist/, build/, or target/
- Do not re-read files already present in any subsystem's file_inventory
- Do not rewrite or delete any section of ANALYSIS.md already written
- Do not create ADDENDUM blocks - update subsystem entries in place
- Do not assign function documentation to any file read with policy HEAD
- Do not create file inventory entries for files that were not read
- Do not produce PRIMER-warranted status for a subsystem until
  analysis_complete is YES
- Do not advance to stage_3_human_gate.md until all PRIMER-warranted
  subsystems have analysis_complete YES
- Do not answer human gate questions yourself
- Do not write any files in docs/ - that is stage 4's job
- Do not proceed to stage 3 in this same context window
  (output the handoff note and stop)