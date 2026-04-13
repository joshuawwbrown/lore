# Stage 2b - Analysis Continuation (Overflow Pass)
# Read this file only if stage_2_analysis.md ran out of context budget before
# completing all flagged files. Execute it exactly. Do not skip steps.
# Pipeline reference: lore/LORE.md



---

# Purpose

This stage is a direct continuation of stage_2_analysis.md. It picks up deep
reading from the first file in the SKELETON.md flagged list that was not yet
read in stage 2. It appends its findings to docs/.LORE/work/ANALYSIS.md.

Stage 2b is not a new analysis pass. It does not re-read files already read.
It does not re-write sections already written. It only extends ANALYSIS.md
with information from the remaining unread files.

---

# Prerequisites

Before executing this stage:

1. Read lore/SCHEMAS.md in full. You must know the exact schema for
   ANALYSIS.md, SKELETON.md, and JOURNAL.md before writing anything.

2. Read docs/.LORE/work/JOURNAL.md. Confirm:
   - status is IN_PROGRESS
   - last_stage_completed is stage_2_analysis.md or stage_2b_continuation.md
   - ANALYSIS.md is listed in Artifacts as PARTIAL

   If ANALYSIS.md is listed as EXISTS (not PARTIAL), stage 2 completed
   successfully and all subsystems with analysis_complete NO have been
   handled. You do not need this stage. Output the following and stop:

       Stage 2b skipped: ANALYSIS.md is already complete.
       Proceed to stage_2c_subsystem_completion.md.

3. Read docs/.LORE/work/SKELETON.md in full. Identify every file in the
   "Files Flagged for Deep Read" section.

4. Read docs/.LORE/work/ANALYSIS.md in full. Note:
   - Which files have already been read (check file_inventory entries and
     [src: filename] tags in sections written so far)
   - Which subsystems have analysis_complete: NO
   - Which sections are marked [INCOMPLETE - context budget reached]
   - The completion_status field value (it must be PARTIAL to be here)

   Build a list: files flagged in docs/.LORE/work/SKELETON.md that do NOT yet appear in
   any subsystem's file_inventory in ANALYSIS.md. This is the remaining
   read list. Work through it in the same priority order as SKELETON.md
   (highest PRIMER score subsystems first, all files of one subsystem
   before moving to the next).

---

# Step 1 - Confirm Remaining Work

State explicitly, before any file reading:

    Files already read in stage 2 / prior 2b passes: [list them]
    Files remaining to read: [list them in priority order, subsystem by subsystem]
    Subsystems with analysis_complete NO: [list them with files_read ratio]
    Sections in ANALYSIS.md marked INCOMPLETE: [list them]

If the remaining read list is empty and no sections are marked INCOMPLETE:

    Output:
        ANALYSIS.md appears complete. No remaining files to read.
        Verify ANALYSIS.md completion_status field and update it to COMPLETE.
        Update JOURNAL.md: last_stage_completed to stage_2b_continuation.md.
        Proceed to stage_3_human_gate.md.
    Stop.

---

# Step 2 - Read Remaining Files

For each file in the remaining read list, in priority order:

    Check the file's read policy from SKELETON.md Files Flagged for Deep Read:
        FULL  - read the entire file and extract all details below.
        HEAD  - read the first 20 lines only. Record as a data file in the
                inventory. Do not extract function details.
        SKIP  - do not read. Do not record in any inventory.

    No documentation is ever produced for a file that was not read with
    policy FULL. HEAD files get a one-line inventory entry only.

    For each FULL file read, extract:
        a) Any information relevant to sections already written in ANALYSIS.md
           that were marked INCOMPLETE or that could be augmented.
        b) Any new subsystems, data flows, or integration points not yet noted.
        c) Any bugs or risks not yet noted.
        d) Any key functions or exports not yet listed.
        e) Any correction to claims in SKELETON.md (record for Step 4).

    Tag every extracted fact with [src: filename] immediately as you note it.
    Do not carry untagged facts forward.

    After reading each file, update the files_read count for its subsystem:
        files_read: [new N] of [M]
        analysis_complete: YES if new N equals M, NO otherwise.

    Monitor context budget continuously.
    If budget is approaching:
        Stop reading further files.
        Note the first unread file.
        Mark any sections you cannot complete as [INCOMPLETE - stage 2b context exhausted].

---

# Step 3 - Append to ANALYSIS.md

DO NOT rewrite ANALYSIS.md from scratch.
DO NOT remove or alter sections already written.
DO NOT re-state facts already present unless correcting an error.

For each subsystem entry whose files were read in this pass:

    Update the file_inventory block IN PLACE: add one entry per newly read file
    directly into the existing file_inventory block for this subsystem.
    Do not append a separate ADDENDUM block. Edit the subsystem entry directly.
        FULL reads: full detail (description, key functions, src tags).
        HEAD reads: one-liner only.
        No entry for files not yet read.

    Update files_read to the new N of M count IN PLACE (overwrite the prior value).
    Update analysis_complete to YES if N now equals M.
    Add an inline note on the updated line: [updated: stage_2b pass N]

    If all files for this subsystem are now read (analysis_complete YES):
        Remove any [INCOMPLETE - not read in this pass] markers from
        this subsystem's entry.
        The subsystem is now eligible for PRIMER production.

For each cross-cutting section in ANALYSIS.md that was marked INCOMPLETE:

    Remove the [INCOMPLETE - context budget reached] marker.
    Append the new information gathered in this pass directly into the section.
    If still incomplete after this pass, restore the marker with updated reason:
        [INCOMPLETE - stage 2b context exhausted, [N] files remaining]

For cross-cutting sections that were complete but have new supporting detail:

    Add the new detail directly into the relevant paragraph or list in place.
    Mark the addition with a trailing inline note: [added: stage_2b pass N]
    Do not create a separate ADDENDUM block. ANALYSIS.md must remain readable
    as a single coherent reference document, not a log of diffs.

Update the completion_status field at the top of ANALYSIS.md:

    If all subsystems have analysis_complete YES and no cross-cutting section
    is still marked INCOMPLETE: set completion_status to COMPLETE.

    Otherwise: leave it as PARTIAL and update the Incomplete Sections list
    to reflect which subsystems still have analysis_complete NO.

---

# Step 4 - Check for Skeleton Corrections

Review everything read during this stage against the claims in SKELETON.md.

For each contradiction found:

    Append a CORRECTION entry to the Corrections section of SKELETON.md.
    Format:
        CORRECTION [stage 2b]: [section name] - [what changed and why]
        Source: [file and line or range that contradicts the original claim]

    Also note the correction in ANALYSIS.md in the relevant section with a
    brief inline note: [corrected: see SKELETON.md Corrections]

If no contradictions are found, write in the journal:
    "Stage 2b: no skeleton corrections required."

---

# Step 5 - Consolidate QUESTIONS.md

Review new information gathered against the questions already recorded in
docs/.LORE/work/QUESTIONS.md.

For each existing question that is now answerable from code:
    Do NOT answer it in QUESTIONS.md.
    Instead, add a note in ANALYSIS.md in the relevant section:
        [Note: Q[number] may be answerable from code - see [src: file]]
    This lets the human gate decide whether to remove the question.

For each new question raised by files read in this stage:
    Append to docs/.LORE/work/QUESTIONS.md using the QUESTIONS.md schema.
    Assign question numbers continuing from the last number already present.
    Classify each as Tier 1, 2, or 3 using the same discipline as stage 1:
        - Tier 1 only if documentation cannot be accurate without this answer.
        - Apply the 5-question Tier 1 limit across the entire QUESTIONS.md file.
          If there are already 5 Tier 1 questions, new questions must be Tier 2
          or Tier 3 unless one of the existing Tier 1 questions is demoted.

---

# Step 5b - Update MANIFEST.md

Read docs/.LORE/work/MANIFEST.md. For each file read during Step 2 of this stage:

    - Obtain the file's mtime using bash ls -la if available.
      If bash is not available, write "unknown" for mtime.
    - Assign the subsystem by matching the file against the key files
      lists in the Subsystem entries of ANALYSIS.md.
      If no match, write "none".
    - Write stage as "2b".

For each file already present in the manifest:
    If the file was re-read in this pass, overwrite its entry with the new
    mtime, subsystem, and stage values.
    If the file was not re-read, leave its entry unchanged.

For files read in this pass that are not yet in the manifest, append them.

Update the manifest_produced field to the current timestamp.

---

# Step 6 - Update JOURNAL.md

Append to the Completed Stages section:

    - stage_2b_continuation.md: [DONE | PARTIAL] [optional note]

    Use DONE only if completion_status in ANALYSIS.md is now COMPLETE
    AND all PRIMER-warranted subsystems have analysis_complete YES.
    Use PARTIAL if any subsystem still has analysis_complete NO, and
    note which subsystems remain and their files_read ratio.

Update the following fields:
last_updated: [current timestamp in YYYY-MM-DD HH:MM format]
last_stage_completed: stage_2b_continuation.md
files_read_total: [add the count of FULL reads completed this pass
    to the existing files_read_total value in JOURNAL.md]
    status: IN_PROGRESS

Update the Artifacts section:
    - docs/.LORE/work/ANALYSIS.md: [EXISTS | PARTIAL]
    - docs/.LORE/work/QUESTIONS.md: EXISTS
    - docs/.LORE/work/SKELETON.md: EXISTS
    - docs/.LORE/work/MANIFEST.md: EXISTS

Update the Unanswered Questions section with any new Tier 1 questions
added to QUESTIONS.md during this stage.

---

# Step 7 - Validate Output

Before writing the handoff, verify:

    [ ] ANALYSIS.md completion_status field reflects actual state
    [ ] No section in ANALYSIS.md was overwritten (only appended or extended)
    [ ] Every new claim in ANALYSIS.md this pass is tagged [src:] or [inferred]
    [ ] Every [STAGE 2b ADDENDUM] block is clearly delimited
    [ ] QUESTIONS.md has no new question numbers that collide with existing ones
    [ ] SKELETON.md Corrections section was appended (or journal notes none needed)
    [ ] JOURNAL.md status is IN_PROGRESS

If any check fails: fix it before writing the handoff.

---

# Step 8 - Write the Handoff Note

If ANALYSIS.md is now COMPLETE (all PRIMER-warranted subsystems have
analysis_complete YES and no cross-cutting sections are INCOMPLETE):

    Write the following block exactly, filling in the bracketed fields.
    Copy it into the Last Handoff Note section of JOURNAL.md.
    Output it as the final output of this stage.

    ---
    Stage 2b complete - Analysis Continuation
    Codebase: [total_source_files] files ([languages])
    LORE: Stage [2/5] Pass [2] Context [N]
    Status: COMPLETE - all flagged files read, proceed to Stage 2c

    Subsystems fully analyzed: [count] ([names])
    Files read this pass: [count]
    Total files read across all analysis passes: [count] of [total_source_files]
    Tokens this context: ~[N]k (estimated)
    Remaining tokens (est.): [low]k-[high]k across [N] more contexts

    Please start a new context with this text:

    Please follow `lore/LORE.md` and CONTINUE.
    ---

If ANALYSIS.md is still PARTIAL (context budget exhausted again):

    Write the following block exactly, filling in the bracketed fields.
    Copy it into the Last Handoff Note section of JOURNAL.md.
    Output it as the final output of this stage.

    ---
    Stage 2b partial - Analysis Continuation (budget reached)
    Codebase: [total_source_files] files ([languages])
    LORE: Stage [2/5] Pass [2] Context [N]
    Status: PARTIAL - context budget reached, continue with another Stage 2b

    Subsystems fully analyzed this pass: [count] ([names])
    Subsystems still incomplete: [count] ([names with files_read ratio])
    Files read this pass: [count]
    Next file: [filename]
    Tokens this context: ~[N]k (estimated)
    Remaining tokens (est.): [low]k-[high]k across [N] more contexts

    Please start a new context with this text:

    Please follow `lore/LORE.md` and CONTINUE.
    ---

---

# DO NOT

- Do not rewrite or delete any section of ANALYSIS.md already written
- Do not create ADDENDUM blocks - update subsystem entries in place
- Do not re-read files already read in a previous stage 2 or stage 2b pass
- Do not re-number or remove existing questions in QUESTIONS.md
- Do not produce any output in docs/ - that is stage 4's job
- Do not answer human gate questions yourself
- Do not run stage 2b if ANALYSIS.md completion_status is already COMPLETE
- Do not assign function documentation to any file read with policy HEAD
- Do not create file inventory entries for files that were not read
- Do not proceed to stage_2c_subsystem_completion.md in this same context window
  (output the handoff note and stop)