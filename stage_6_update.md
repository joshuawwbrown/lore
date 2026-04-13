# Stage 6 - Scoped Update Pass
# Canonical output file naming: LORE_PRIMER_[name].md and LORE_GUIDE_[name].md
# Read this file and execute it exactly. Do not skip steps.
# Pipeline reference: lore/LORE.md
# Do not begin writing output until Step 5.

---

# Purpose

This stage performs a targeted re-analysis of files that have changed since
the last analysis run. It patches docs/.LORE/work/ANALYSIS.md in place and
regenerates only the documentation sections affected by the changes.

It does not re-read files that have not changed. It does not rewrite
sections of ANALYSIS.md that are not affected. It does not regenerate
output documents that are not affected.

If the scope of changes is too broad to handle incrementally, this stage
will say so and recommend option [F] (full re-scan) from MENU D in START.md.

---

# Prerequisites

Before executing this stage:

1. Read lore/SCHEMAS.md in full. Know all schemas before writing anything.

2. Read docs/.LORE/work/JOURNAL.md. Verify:
   - status is IN_PROGRESS
   - MANIFEST.md is listed in Artifacts as EXISTS
   If MANIFEST.md is MISSING, output:
       "Stage 6 prerequisite failed: MANIFEST.md does not exist.
        A full re-scan is required. Return to START.md and choose [F]."
   Stop.

3. Read docs/.LORE/work/MANIFEST.md in full. Note:
   - The mtime_available field at the top of the file
   - Every file in the Files Read section with its recorded mtime
   - Every entry in the Manual Overrides section
   - Every entry in the Preserved Files section

   If mtime_available is NO:
       Output:
           "MANIFEST.md was produced without filesystem mtime data.
            mtime-based change detection is not available.
            Falling back to git-diff mode."
       Check whether git is available by attempting: git --version
       If git is available:
           Ask the user: "I need to run git diff to detect changed files. OK? [yes/no]"
           If yes: proceed using git diff output as the changed-files list.
           If no:
               Output: "Cannot detect changes without mtime or git.
                        Choose [F] from MENU D for a full re-scan."
               Stop.
       If git is not available:
           Output: "Neither mtime nor git is available. Cannot detect changed files.
                    Choose [F] from MENU D for a full re-scan."
           Stop.

4. Read docs/.LORE/work/ANALYSIS.md in full. This is your ground truth.
   Note which subsystems exist and which files each subsystem owns
   (from the key files field of each subsystem entry).

5. Read docs/.LORE/work/SKELETON.md in full.

---

# Step 1 - Confirm Changed File List

The changed-files list was built by MENU D in START.md before routing here.
Restate it explicitly before proceeding:

    Changed files detected: [count]
    [For each file:]
        - [filepath]: recorded mtime [old] | current mtime [new | no longer exists]

If the list is empty, output:
    "No changed files. Nothing to do. Stage 6 is not needed."
    Update JOURNAL.md: status to COMPLETE (it was already complete).
    Stop.

---

# Step 2 - Assess Scope

Before reading any file, assess whether the changes are amenable to a
scoped update or whether a full re-scan is more appropriate.

For each changed file, classify it:

    KNOWN - the file appears in MANIFEST.md and maps to a known subsystem
            in ANALYSIS.md. A targeted patch is possible.

    KNOWN-UNOWNED - the file appears in MANIFEST.md but its subsystem
            field is "none". It was read before but not assigned to a
            subsystem. Patching is possible but broader sections of
            ANALYSIS.md (Theory of Operation, Code Style, Data Paradigm,
            IT Paradigm) may be affected.

    NEW - the file does not appear in MANIFEST.md at all. It is a file
            that was not read in the original analysis. Treat it as a
            newly important file. Read it and assess whether it reveals
            a new subsystem or significantly changes existing understanding.

    DELETED - the file no longer exists on disk. Its contributions to
            ANALYSIS.md may need to be retracted or flagged.

Tally the classifications. Apply this scope judgment:

    If more than 8 files are changed in total:
        Output:
            "Warning: [count] files changed. This is a large update.
             A full re-scan (option [F] from MENU D) may produce better
             results than a scoped update. Proceed with scoped update
             anyway? [yes/no]"
        If no: output the handoff note with recommendation to use [F]. Stop.
        If yes: continue.

    If any NEW files are detected:
        Note them. They will be read and assessed in Step 3.
        If more than 5 NEW files are detected:
            Output:
                "Warning: [count] new files found that were not in the
                 original analysis. This may indicate a significant
                 structural change. A full re-scan may be more appropriate.
                 Proceed with scoped update anyway? [yes/no]"
            If no: output the handoff note with recommendation to use [F]. Stop.
            If yes: continue.

    If any DELETED files are detected:
        Note them. Their contributions will be retracted in Step 5.

---

# Step 3 - Read Changed and New Files

Read each KNOWN, KNOWN-UNOWNED, and NEW file, in this order:
    1. KNOWN files, grouped by subsystem (highest-PRIMER-score subsystem first)
    2. KNOWN-UNOWNED files
    3. NEW files

For each file read, extract the same categories as stage 2 Step 2:
    a) What does this file do? Has its purpose changed?
    b) What subsystem does it belong to? (re-assess for NEW files)
    c) Key exported functions, classes, routes, or constants - what changed?
    d) Notable imports - did dependencies change?
    e) Any data structures, schemas, or type definitions - what changed?
    f) Any external service calls - what changed?
    g) Any configuration values - what changed?
    h) Any hardcoded values, magic numbers, or new technical debt
    i) Any error handling patterns - what changed?
    j) Anything that contradicts a claim in ANALYSIS.md or SKELETON.md

For git-available environments:
    Before reading each file in full, attempt:
        git diff HEAD -- [filepath]
    If the diff is available and under 200 lines, read the diff first.
    This gives precise visibility into what changed without reading
    the entire file. Then read the full file to confirm context.
    If the diff is over 200 lines, skip it and read the file in full.

Tag every note [src: filename] or [src: filename:line].
Monitor context budget continuously. If CAUTION threshold (60%) is reached,
finish the current file and stop reading further files. Note all unread
changed files explicitly - they must be handled in a subsequent update pass
or a full re-scan.

---

# Step 4 - Assess Impact

After reading all changed files you can fit, determine what in ANALYSIS.md
needs to change.

For each changed file, map its impact:

    Subsystem entries affected:
        List each subsystem whose entry in ANALYSIS.md needs patching.
        For each: describe specifically what needs to change (purpose,
        key files, key functions, dependencies, integration points).

    Cross-cutting sections affected:
        Theory of Operation: YES | NO | MAYBE
        Data Paradigm:       YES | NO | MAYBE
        IT Paradigm:         YES | NO | MAYBE
        Code Style:          YES | NO | MAYBE
        Observed Bugs:       YES | NO | MAYBE
            (MAYBE means a change was observed but its documentation
             impact is unclear - note it and proceed conservatively)

    Output documents affected:
        For each subsystem entry changed: its PRIMER (if one exists) is affected.
        If Theory of Operation changed: LORE_ABSTRACT.md and LORE_ENCHIRIDION.md are affected.
        If Data Paradigm changed: LORE_ABSTRACT.md and LORE_ENCHIRIDION.md are affected.
        If IT Paradigm changed: LORE_ABSTRACT.md and LORE_ENCHIRIDION.md are affected.
        If Code Style changed: LORE_ENCHIRIDION.md is affected.
        LORE_INDEX.md is never affected by content changes (only by new/removed PRIMERs).

    New subsystems detected (from NEW files):
        If a NEW file reveals a subsystem not in ANALYSIS.md: flag for addition.
        A new subsystem addition affects LORE_ABSTRACT.md, LORE_ENCHIRIDION.md,
        and LORE_INDEX.md (new PRIMER entry needed).

    DELETED files:
        For each deleted file: identify which ANALYSIS.md claims sourced from it.
        Mark those claims for retraction or replacement with [inferred].

Record this impact map explicitly before writing anything.

---

# Step 5 - Patch ANALYSIS.md

DO NOT rewrite ANALYSIS.md from scratch.
DO NOT remove or alter sections not in your impact map.
DO NOT re-state facts that have not changed.

For each subsystem entry in your impact map:

    Navigate to the subsystem's entry in ANALYSIS.md.
    Apply targeted edits:
        - Update the purpose paragraph if the file's role changed.
        - Update key files if files were added, removed, or renamed.
        - Update key functions or exports if signatures or behavior changed.
        - Update external dependencies if imports changed.
        - Update integration points if service calls changed.
        - Update PRIMER warranted score if the change affects the criteria.
    Label each changed block with an inline note:
        [updated: stage_6_update, date]
    Do not remove the prior content wholesale - if a function was removed,
    note it as "removed as of [date] [src: filename]" rather than silently
    deleting the entry. This preserves auditability.

For each cross-cutting section in your impact map marked YES or MAYBE:

    Navigate to the section in ANALYSIS.md.
    Append a [STAGE 6 UPDATE] block below the existing content:
        [STAGE 6 UPDATE - date]
        [new or corrected information with src tags]
    Do not alter the existing text above the update block.
    If a claim in the existing text is now directly contradicted, add
    an inline retraction immediately after the contradicted sentence:
        [retracted: stage_6_update, date - see update block below]

For each deleted file:

    Find every [src: filepath] tag in ANALYSIS.md that references it.
    For each such claim, append inline:
        [source deleted: stage_6_update, date - claim unverified]

For new subsystems detected from NEW files:

    Append a new subsystem entry at the end of the Subsystems section.
    Use the same format as existing entries.
    Label it: [new subsystem - added by stage_6_update, date]

---

# Step 6 - Update MANIFEST.md

For each file read during Step 3:

    If the file already has an entry in MANIFEST.md:
        Overwrite its entry with the new mtime, subsystem, and stage "update".

    If the file is new (not previously in the manifest):
        Append a new entry with its mtime, assigned subsystem (or "none"),
        and stage "update".

For each DELETED file:
    Update its manifest entry:
        - [filepath]: mtime [original] | subsystem [name] | stage [original] | DELETED [date]

Update the manifest_produced field to the current timestamp.

---

# Step 7 - Regenerate Affected Output Documents

Regenerate only the output documents identified in your Step 4 impact map.

For each affected output document:

    Read docs/.LORE/work/ANALYSIS.md (now patched).
    Read the relevant template from lore/templates/.
    Regenerate only the affected sections of the document, not the entire file.

    Specifically:
        For a subsystem entry change:
            Update that subsystem's entry in LORE_ENCHIRIDION.md.
            If a LORE_PRIMER_[name].md exists for that subsystem, regenerate
            it in full (PRIMERs are short enough that targeted patching is
            not worth the risk).
            If a LORE_GUIDE_[name].md exists for that subsystem, regenerate
            it in full as well (operator-facing content must stay in sync with
            any subsystem entry changes).

        For Theory of Operation change:
            Update the "how it works" section of LORE_ABSTRACT.md.
            Update the Theory of Operation section of LORE_ENCHIRIDION.md.

        For Data Paradigm change:
            Update the "data layer" section of LORE_ABSTRACT.md.
            Update the data paradigm section of LORE_ENCHIRIDION.md.

        For IT Paradigm change:
            Update the "infrastructure and deployment" section of LORE_ABSTRACT.md.
            Update the IT paradigm section of LORE_ENCHIRIDION.md.

        For Code Style change:
            Update the code style section of LORE_ENCHIRIDION.md.

        For new subsystem:
            Add the subsystem row to the "subsystems at a glance" table
            in LORE_ABSTRACT.md.
            Add the subsystem entry to LORE_ENCHIRIDION.md subsystem catalog.
            If PRIMER warranted (score 2+): produce LORE_PRIMER_[name].md
            using lore/templates/PRIMER_template.md.
            If operator_facing is YES or dev_entry_points is non-empty:
            produce LORE_GUIDE_[name].md using
            lore/templates/GUIDE_template.md.
            Add any new PRIMER and GUIDE files to LORE_INDEX.md.

    Label each regenerated section with a comment on the last line of
    the section (before the next --- separator):
        [updated by stage_6_update on date]

For Manual Override entries in MANIFEST.md:
    Before regenerating any section, check whether that section's content
    was hand-patched via MENU D option [C].
    If yes: output a warning:
        "Warning: [document] [section] has a manual override recorded on [date].
         Regenerating this section will incorporate the patched ANALYSIS.md
         content, but verify the result matches your intent."
    Proceed with regeneration (ANALYSIS.md was already patched in Step 5 to
    reflect manual overrides, so the output should be correct).

If no output documents are affected (impact map shows no changes to
documented sections): skip this step and note it in the journal.

---

# Step 8 - Update JOURNAL.md

Update docs/.LORE/work/JOURNAL.md:

    last_updated: current timestamp (YYYY-MM-DD HH:MM)
    last_stage_completed: stage_6_update.md
    status: COMPLETE

    Append to Completed Stages:
        - stage_6_update.md: DONE - [count] files re-analyzed, [count] sections
          patched, [count] output documents updated

    Artifacts section: overwrite in full.
        List all work files as EXISTS.
        List all output documents - updated ones as EXISTS, unchanged ones
        as EXISTS (unchanged). If a new PRIMER was added, list it as EXISTS (new).

    Append a section titled "Update Pass Summary - [date]":
        Files re-analyzed:          [count] ([list filenames])
        Files deleted:              [count] ([list filenames, or "none"])
        New files incorporated:     [count] ([list filenames, or "none"])
        ANALYSIS.md sections patched: [list section names, or "none"]
        Output documents updated:   [list filenames, or "none"]
        Manual overrides preserved: [count, or "none"]
        Unread changed files:       [list filenames, or "none - all changes processed"]
        [If unread files exist:]
        Note: run stage_6_update.md again or use option [F] from MENU D
        to process remaining changes.

---

# Step 9 - Validate Output

Before writing the handoff note, verify:

    [ ] docs/.LORE/work/ANALYSIS.md has [updated: stage_6_update] labels on changed sections
    [ ] docs/.LORE/work/MANIFEST.md manifest_produced timestamp is current
    [ ] docs/.LORE/work/MANIFEST.md mtime_available field is present and set correctly
    [ ] Every changed file from the changed-files list has an updated entry in MANIFEST.md
    [ ] Every affected output document has been regenerated
    [ ] No unregenerated output document contains stale [updated by stage_6_update]
        a prior pass that conflicts with the current one
    [ ] LORE_INDEX.md lists any newly added PRIMER or GUIDE files
    [ ] All PRIMER files follow the LORE_PRIMER_[name].md naming pattern
    [ ] All GUIDE files follow the LORE_GUIDE_[name].md naming pattern
    [ ] No file uses the old LORE_[name]_PRIMER.md naming pattern
    [ ] JOURNAL.md status is COMPLETE (or IN_PROGRESS if unread files remain)
    [ ] No section of ANALYSIS.md was deleted - only patched or appended

## Mini-validity pass on regenerated sections

For each output document section that was regenerated in Step 7, run these
checks before writing the handoff note:

    [ ] No template markers remain in the regenerated section.
        Check for: [PLACEHOLDER], [INSERT, {{, }}, [TEMPLATE INSTRUCTIONS]
        FAIL: fix inline before writing the handoff note.

    [ ] Every factual claim in a regenerated ENCHIRIDION section is tagged
        [src: filename] or [inferred]. Untagged claims in regenerated content
        are a common error because the regeneration is done without the full
        stage 4 source map in memory.
        FAIL: add missing tags or change the claim to [inferred] if the
        source cannot be confirmed from ANALYSIS.md.

    [ ] Every factual claim in a regenerated OVERVIEW section is tagged
        [src: filename] or [inferred].
        FAIL: same correction as above.

    [ ] If a manual override was recorded for this section (MANIFEST.md
        Manual Overrides), verify that the regenerated section reflects the
        patched content from ANALYSIS.md, not the pre-patch content.
        FAIL: re-read the relevant ANALYSIS.md section and regenerate.

    [ ] No [STAGE 6 UPDATE] block was accidentally included in an output
        document. These blocks belong in ANALYSIS.md only.
        FAIL: remove any such blocks from output documents.

If any mini-validity check fails and cannot be fixed inline, record it in
JOURNAL.md last_error and note the affected document. Do not set status
to ERROR for a mini-validity failure unless it is a template marker or
missing source tag that could mislead a reader - set status to COMPLETE
and flag the issue for human review.

If unread changed files remain (context budget was reached in Step 3):
    Set JOURNAL.md status to IN_PROGRESS instead of COMPLETE.
    Note the unread files in the journal Update Pass Summary.

---

# Step 10 - Write the Handoff Note

If all changed files were processed:

    ---
    STAGE 6 COMPLETE

    Project: [project_name]
    Root: [project_root]
    Schema version: [schema_version]

    Files re-analyzed: [count]
    ANALYSIS.md sections patched: [list or "none"]
    Output documents updated: [list or "none"]
        (include both LORE_PRIMER_*.md and LORE_GUIDE_*.md files updated)
    Manual overrides preserved: [count or "none"]
    Remaining tokens (est.): [low]k-[high]k across [N] more contexts

    Pipeline status: COMPLETE
    Documentation is up to date.

    Please start a new context with this text:

    Please follow `lore/LORE.md` and CONTINUE.
    ---

If unread changed files remain:

    ---
    STAGE 6 PARTIAL - additional pass required

    Files re-analyzed this pass: [count]
    Files still unread: [count]
        [list unread filenames]
    ANALYSIS.md sections patched this pass: [list or "none"]
    Output documents updated this pass: [list or "none"]
    Remaining tokens (est.): [low]k-[high]k across [N] more contexts

    Please start a new context with this text:

    Please follow `lore/LORE.md` and CONTINUE.
    ---

Copy the handoff note into the Last Handoff Note section of JOURNAL.md.
Output it as the final output of this stage.

---

# DO NOT

- Do not rewrite ANALYSIS.md from scratch - only patch affected sections
- Do not regenerate output documents that are not in the impact map
- Do not delete entries from MANIFEST.md - only update or append them
- Do not remove prior content from ANALYSIS.md - retract it with inline notes
- Do not run git commands that modify the repository (no checkout, reset, add, commit)
- Do not read files outside the project root
- Do not write to any file outside docs/.LORE/work/ and docs/
- Do not proceed past this stage without outputting the handoff note