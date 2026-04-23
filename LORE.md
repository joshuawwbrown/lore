# LORE Pipeline
# Schema Version: 1.6
# Copyright (c) 2026 Joshua W W Brown. Licensed under the MIT License. See lore/LICENSE.

This file controls the LORE documentation pipeline.

To start a new documentation run, share this file with your AI assistant and say:

    Please follow `lore/LORE.md` to document this project.

To resume an interrupted run or continue after a stage completes, share this file and say:

    Please follow `lore/LORE.md` and CONTINUE.

---

# What This System Does

This directory contains a documentation pipeline for the project adjacent to
this lore/ folder. It produces human-facing and AI-agent-facing documentation
files in the docs/ directory at the same level as lore/.

The pipeline runs in stages. Each stage produces artifacts that the next stage
consumes. At the end of each stage you will receive an exact prompt to paste
into a fresh context window. All output files are plain ASCII. No unicode.
No emojis.

---

# Step 1 - Detect Pipeline State

Before doing anything else, check whether docs/.LORE/work/JOURNAL.md exists.

    IF docs/.LORE/work/JOURNAL.md does not exist:
        Pipeline state: NEW
        Go to: MENU A - New Project

    IF docs/.LORE/work/JOURNAL.md exists:
        Read docs/.LORE/work/JOURNAL.md now.
        Read the status field.
        Read the schema_version field.
        Compare schema_version to the version declared in lore/SCHEMAS.md.

        IF schema_version does not match SCHEMAS.md version:
            Pipeline state: SCHEMA_MISMATCH
            Go to: MENU F - Schema Mismatch

        IF status is NEW:
            Go to: MENU A - New Project

        IF status is IN_PROGRESS:
            Pipeline state: IN_PROGRESS

            IF the prompt contained the word CONTINUE:
                Read last_stage_completed from JOURNAL.md.
                Determine the next stage automatically (see stage sequence below).
                Execute that stage without presenting a menu.
            ELSE:
                Go to: MENU B - Resume Interrupted Run

        IF status is AWAITING_HUMAN:
            Pipeline state: AWAITING_HUMAN

            IF the prompt contained the word CONTINUE:
                Go to: EXECUTE STAGE 3
            ELSE:
                Go to: MENU C - Human Gate Pending

        IF status is COMPLETE:
            Pipeline state: COMPLETE
            Go to: MENU D - Documentation Exists

        IF status is ERROR:
            Pipeline state: ERROR
            Go to: MENU E - Error Recovery

## Stage sequence for CONTINUE auto-routing

    last_stage_completed            next stage to execute
    -------------------------------------------------------
    stage_1_recon.md             -> EXECUTE STAGE 2
    stage_2_analysis.md          -> EXECUTE STAGE 2b (if ANALYSIS.md is PARTIAL)
                                    EXECUTE STAGE 2c (if ANALYSIS.md is COMPLETE)
                                    Note: on real codebases stage 2 almost always
                                    produces PARTIAL and routes to 2b. Routing
                                    directly to 2c after stage 2 is rare.
    stage_2b_continuation.md     -> EXECUTE STAGE 2b (if any PRIMER-warranted subsystem
                                    still has analysis_complete NO)
                                    EXECUTE STAGE 2c (if all PRIMER-warranted subsystems
                                    have analysis_complete YES but completion gate
                                    has not yet run)
                                    Note: stage 2b runs until all warranted subsystems
                                    are complete. Stage 2c is a final verification gate
                                    that may be skipped if stage 2b already achieved
                                    full completion.
    stage_2c_subsystem_completion.md -> EXECUTE STAGE 2c (if incomplete subsystems remain)
                                        EXECUTE STAGE 3 (if all warranted subsystems complete)
    stage_3_human_gate.md        -> EXECUTE STAGE 4
    stage_4_lore.md              -> EXECUTE STAGE 5
    stage_5_qa.md (pass < 3, defects remain) -> EXECUTE STAGE 5 (next pass)
    stage_5_qa.md (all clear or pass 3)      -> EXECUTE STAGE 7
    stage_7_brief.md             -> pipeline finished

---

# MENU A - New Project

No previous documentation run detected. You are starting fresh.

Present this menu to the user. Wait for a response before proceeding.

    You are about to start a new documentation run for this project.

    The pipeline will:
        1. Scan the project structure (stage 1 - reconnaissance)
        2. Read key files in depth (stage 2 - analysis, may span multiple contexts)
        3. Complete all PRIMER-warranted subsystems (stage 2c - completion gate)
        4. Pause for your answers to a short list of questions (human gate)
        5. Produce all documentation files (stage 4 - LORE production)
        6. Run quality control checks (stage 5 - QA loop)

    A resource estimate (files, context windows, tokens) will be produced
    at the end of stage 1 so you can decide whether to proceed.

    What is the root directory of the project to document?
    (Default: the directory containing this lore/ folder)

    Options:
        [S] Start - begin with the default project root
        [P] Specify a different project root path, then start
        [X] Cancel - do nothing

When the user responds:

    If S or default confirmed:
        Set project_root to the parent directory of lore/
        Go to: EXECUTE STAGE 1

    If P:
        Accept the path the user provides.
        Set project_root to that path.
        Go to: EXECUTE STAGE 1

    If X:
        Output: "Pipeline not started. Run again when ready."
        Stop.

---

# MENU B - Resume Interrupted Run

An unfinished run was detected.

Read the following from JOURNAL.md:
    - last_stage_completed
    - last_handoff_note (from the Last Handoff Note section)
    - artifacts list
    - any PARTIAL or MISSING artifacts

Present this menu to the user. Wait for a response.

    An unfinished documentation run was detected.

    Project: [project_name from journal]
    Last completed stage: [last_stage_completed from journal]
    Started: [started from journal]

    Artifacts produced so far:
    [paste the Artifacts section from JOURNAL.md]

    Options:
        [C] Continue - resume from the next stage
        [R] Restart - discard all work and start over from stage 1
        [I] Inspect - show the last handoff note from the interrupted run
        [X] Cancel - do nothing

When the user responds:

    If C:
        Read the Last Handoff Note section from JOURNAL.md.
        Output: "Resuming. Next stage:"
        Output the last handoff note verbatim.
        Then follow the stage sequence to determine and execute the next stage.

    If R:
        Confirm with the user: "This will delete all work files. Are you sure? [yes/no]"
        If yes: delete all files in docs/.LORE/work/ and go to EXECUTE STAGE 1.
        If no: return to MENU B.

    If I:
        Output the Last Handoff Note section from JOURNAL.md verbatim.
        Then re-present MENU B.

    If X:
        Output: "Pipeline not resumed."
        Stop.

---

# MENU C - Human Gate Pending

The pipeline is paused waiting for human answers to questions.

Read docs/.LORE/work/QUESTIONS.md.
Count the number of questions where ANSWER is blank.

Present this menu to the user. Wait for a response.

    The pipeline is paused at the human gate.

    Project: [project_name from journal]
    Unanswered questions: [count] remaining

    The pipeline cannot produce final documentation until Tier 1 questions
    are answered. Tier 2 and Tier 3 questions are optional.

    Options:
        [A] I have answered the questions - continue to documentation production
        [S] Show me the unanswered questions now
        [K] Skip all remaining questions and continue with best-guess defaults
        [X] Cancel - do nothing

When the user responds:

    If A:
        Read docs/.LORE/work/QUESTIONS.md.
        Verify that all Tier 1 questions have non-blank ANSWER fields.
        If any Tier 1 questions are still blank:
            Output: "The following Tier 1 questions are still unanswered:"
            List them.
            Output: "Please answer these before continuing, or choose [K] to skip."
            Re-present MENU C.
        If all Tier 1 questions are answered:
            Update JOURNAL.md: set status to IN_PROGRESS,
            last_stage_completed to stage_3_human_gate.md
            Go to: EXECUTE STAGE 4

    If S:
        Output all questions from QUESTIONS.md where ANSWER is blank.
        Re-present MENU C.

    If K:
        Note in JOURNAL.md: "Human gate skipped. Stage 4 will use default
        assumptions for all unanswered questions."
        Update JOURNAL.md: status to IN_PROGRESS
        Go to: EXECUTE STAGE 4

    If X:
        Output: "Pipeline not resumed."
        Stop.

---

# MENU D - Documentation Exists

A complete documentation run was previously finished.

Read JOURNAL.md for the list of artifacts produced.

Present this menu to the user. Wait for a response.

    Documentation for this project is complete.

    Project: [project_name from journal]
    Completed: [last_updated from journal]

    Artifacts on file:
    [paste the Artifacts section from JOURNAL.md]

    What would you like to do?

    Options:
        [U] Update docs after code changes - detect changed files and re-analyze
        [B] Generate project brief - produce LORE_BRIEF.md (AI session entry point)
        [P] Plan a change - create a development plan for a proposed change
        [F] Full re-scan - discard all analysis and start over from stage 1
        [Q] Re-run quality control only - re-run the QA loop on existing docs
        [C] Correct a specific document - identify and fix a problem
        [W] Answer unanswered questions - improve docs with new information
        [R] Start over completely - discard everything and re-run from scratch
        [X] Cancel - do nothing

When the user responds:

    If B:
        Output: "Generating project brief."
        Update JOURNAL.md status to IN_PROGRESS.
        Go to: EXECUTE STAGE 7

    If P:
        Output:
            "To create a development plan, share lore/PLAN.md with your AI
             assistant and say:

                 Please follow `lore/PLAN.md` to create a plan.

             The plan pipeline is separate from the documentation pipeline.
             It does not modify any LORE work files or output documents."
        Stop.

    If U:
        Read docs/.LORE/work/MANIFEST.md.
        If MANIFEST.md does not exist:
            Output: "No file manifest found. A full re-scan is required."
            Go to: If F (below).

        Read the git_commit field from MANIFEST.md.
        Read the sha256_available field from MANIFEST.md.

        -- Step 1: git discovery (fast candidate detection) --

        Check whether git is available by checking for a .git/ directory
        in the project root.

        If git is available:
            Run: git status --short
            Ask the user: "I need to run git status to detect changed files. OK? [yes/no]"
            If yes:
                Run: git status --short
                Collect all files listed as modified, added, deleted, or untracked.
                This is the initial candidate list.

                If git_commit is recorded in MANIFEST.md:
                    Ask the user: "I need to run git diff to check committed changes. OK? [yes/no]"
                    If yes:
                        Run: git diff [git_commit] HEAD --name-only
                        Add any files listed to the candidate list (deduplicated).
                        For each candidate, use commit messages from
                        git log [git_commit]..HEAD -- [file] as context
                        when reporting changed files to the user.
                    If no:
                        Note: committed changes since last analysis run may be missed.
            If no:
                Set candidate list to empty.
                Note: git skipped by user. Proceeding to checksum-only detection.

        If git is not available:
            Set candidate list to all files listed in MANIFEST.md Files Read section.
            Note: no git available. All manifest files will be checksum-verified.

        -- Step 2: checksum confirmation --

        Check whether sha256sum is available by attempting: sha256sum --version

        If sha256sum is available and sha256_available is YES in MANIFEST.md:
            Ask the user: "I need to run sha256sum to confirm changed files. OK? [yes/no]"
            If yes:
                For each file in the candidate list (or all manifest files if git
                was unavailable or skipped):
                    Run: sha256sum [filepath]
                    Compare the result to the sha256 recorded in MANIFEST.md.
                    If the hash matches: remove the file from the changed-files list
                    (content is identical despite mtime or git status difference).
                    If the hash differs or the file no longer exists: keep on list.
                Build the final changed-files list from confirmed hash mismatches only.
            If no:
                Use the candidate list as-is for the changed-files list.

        If sha256sum is not available:
            Output: "sha256sum is not available on this system."
            Output: "To enable precise change detection, install it:"
            Output: "  macOS:   brew install coreutils"
            Output: "  Linux:   sha256sum is included in coreutils (already installed)"
            Output: "  Windows: available in Git Bash or WSL"
            Output: "Falling back to git status / mtime detection without confirmation."
            Use the candidate list as-is for the changed-files list.

        If sha256sum is available but sha256_available is NO in MANIFEST.md:
            Note: previous analysis run did not record checksums. Checksums cannot
            be used for confirmation this run. Use the candidate list as-is.
            Checksums will be recorded during this update run for future use.

        -- Step 3: report and proceed --

        If the changed-files list is empty:
            Output: "No changed files detected since the last analysis run."
            Output: "Docs are up to date. Choose [Q] to re-run QA or [X] to cancel."
            Re-present MENU D.

        If the changed-files list is not empty:
            Output: "Changed files detected:"
            For each file: list the filepath, the recorded sha256 (or "none" if
            not available), and the current sha256 (or "not checked").
            If git commit messages are available for a file, show a one-line
            summary of the most recent commit message touching that file.
            Output: "LORE will re-analyze these files and update affected documentation."
            Confirm: "Proceed? [yes/no]"
            If yes:
                Update JOURNAL.md status to IN_PROGRESS.
                Go to: EXECUTE STAGE 6
            If no:
                Re-present MENU D.

    If F:
        Output: "Full re-scan selected."
        Output: "All analysis work files will be discarded and rebuilt from scratch."
        Confirm: "Proceed? [yes/no]"
        If yes:
            Delete docs/.LORE/work/SKELETON.md, docs/.LORE/work/ANALYSIS.md,
            docs/.LORE/work/QUESTIONS.md, docs/.LORE/work/MANIFEST.md.
            Preserve docs/.LORE/work/JOURNAL.md.
            Update JOURNAL.md status to IN_PROGRESS.
            Note: docs/LORE_PRIMER_*.md and docs/LORE_GUIDE_*.md will be
            regenerated by stage 4. Do not delete them manually.
            Go to: EXECUTE STAGE 1
        If no: re-present MENU D.

    If Q:
        Output: "QA re-run selected."
        Go to: EXECUTE STAGE 5

    If C:
        Ask: "Which document needs correction? Describe the problem."
        Accept the user's description.
        Read the relevant document.
        Read docs/.LORE/work/ANALYSIS.md to understand the current ground truth
        for the section being corrected.

        Make the correction to the output document.

        For each factual claim that was changed (added, removed, or altered):
            State explicitly to the user:
                "I changed the following claim in [document name]:"
                "  Before: [exact old text]"
                "  After:  [exact new text]"
                "  Source for new claim: [src: filename] or [inferred]"
                ""
                "This changes a fact, not just wording. Do you want me to update
                the analysis record in ANALYSIS.md so this is not overwritten
                by a future update or full re-scan? [Y/N]"

            If user says Y:
                Patch the relevant section of docs/.LORE/work/ANALYSIS.md to match
                the corrected claim.
                Append to docs/.LORE/work/MANIFEST.md Manual Overrides section:
                    - [source filepath]: override on [date] - [one-line description
                      of what changed in the output doc and in ANALYSIS.md]
                Note in JOURNAL.md under a "Manual Overrides" note:
                    "[date]: [document] - [one-line description]"

            If user says N:
                Note in JOURNAL.md under a "Manual Overrides" note:
                    "[date]: [document] - correction applied to output only.
                    ANALYSIS.md not updated. A future [U] or [F] run may
                    overwrite this correction."

        If no factual claims were changed (wording, formatting, or structure
        corrections only):
            Skip the above and output:
                "Correction applied (no factual claims changed)."

        Update JOURNAL.md last_updated.
        Output: "Done. Run QA again with option [Q] to verify."

    If W:
        Output the current unanswered questions from QUESTIONS.md.
        Instruct the user to answer them and then choose option [U] or [Q].

    If R:
        Confirm: "This will delete all work files and all generated docs. Are you sure? [yes/no]"
        If yes:
            Delete all files in docs/.LORE/work/.
            Delete all docs/LORE_*.md files
            (LORE_ABSTRACT.md, LORE_BRIEF.md, LORE_ENCHIRIDION.md, LORE_INDEX.md,
             LORE_PRIMER_*.md, LORE_GUIDE_*.md).
            Do not delete any docs/ file that does not begin with LORE_.
            Go to: EXECUTE STAGE 1.
        If no: re-present MENU D.

    If X:
        Output: "No changes made."
        Stop.

---

---

# MENU E - Error Recovery

An error was recorded in the journal.

Read the last_error field from JOURNAL.md.
Read the last_stage_completed field.

Present this menu to the user. Wait for a response.

    An error was detected in the previous pipeline run.

    Project: [project_name from journal]
    Failed at stage: [last_stage_completed or "unknown"]
    Error: [last_error from journal]

    Options:
        [T] Retry the failed stage
        [S] Skip the failed stage and continue to the next one
        [I] Inspect work files - show current artifact status
        [R] Restart from scratch
        [X] Cancel - do nothing

When the user responds:

    If T:
        Update JOURNAL.md: status to IN_PROGRESS, last_error to "none"
        Re-execute the stage named in last_stage_completed.

    If S:
        Output: "Skipping [last_stage_completed]. This may result in incomplete documentation."
        Update JOURNAL.md: append "[last_stage_completed]: SKIPPED - user override"
        to Completed Stages.
        Advance to the next stage in the sequence and execute it.

    If I:
        Output the Artifacts section from JOURNAL.md.
        Re-present MENU E.

    If R:
        Confirm: "This will delete all work files. Are you sure? [yes/no]"
        If yes: delete all files in docs/.LORE/work/ and go to EXECUTE STAGE 1.
        If no: re-present MENU E.

    If X:
        Output: "No action taken."
        Stop.

---

# MENU F - Schema Mismatch

The journal was produced with a different version of SCHEMAS.md than
the current one.

Read the schema_version from JOURNAL.md.
Read the current version from SCHEMAS.md.
Read the MIGRATION LOG section of SCHEMAS.md.
Find the migration entry for the version transition.

Present this menu to the user. Wait for a response.

    Schema version mismatch detected.

    Journal schema version: [from journal]
    Current schema version: [from SCHEMAS.md]

    The pipeline cannot continue until work files are migrated to the new schema.
    Migration instructions are in lore/SCHEMAS.md (MIGRATION LOG section).

    [If migrating from 1.5 to 1.6, also note:]
    This upgrade adds:
        - LORE_ABSTRACT.md (renamed from LORE_OVERVIEW.md)
        - LORE_BRIEF.md (new AI session entry-point document)
        - git authorship integration in stage 1
        - simplified confidence model for human answers

    Options:
        [M] Migrate work files now - apply the migration instructions automatically
        [R] Restart from scratch - discard all work files and start fresh
        [X] Cancel - do nothing

When the user responds:

    If M:
        Read the migration instructions from SCHEMAS.md for the relevant version transition.
        Apply each migration instruction to the relevant work file.
        Update schema_version in JOURNAL.md to the current version.
        Update JOURNAL.md last_updated.
        Output: "Migration complete."
        [If migrating to 1.6:]
            Output: "LORE_BRIEF.md does not yet exist for this project."
            Output: "Run option [B] from MENU D to generate it, or start a new
                     pipeline run - stage 7 will produce it automatically."
        Return to Step 1 - Detect Pipeline State.

    If R:
        Confirm: "This will delete all work files. Are you sure? [yes/no]"
        If yes: delete all files in docs/.LORE/work/ and go to EXECUTE STAGE 1.
        If no: re-present MENU F.

    If X:
        Output: "No action taken."
        Stop.

---

# EXECUTE STAGE 1

Read lore/stage_1_recon.md and execute it exactly.
Do not proceed past it until it has completed and written its handoff note.

---

# EXECUTE STAGE 2

Read lore/stage_2_analysis.md and execute it exactly.
Do not proceed past it until it has completed and written its handoff note.

---

# EXECUTE STAGE 2b

Read lore/stage_2b_continuation.md and execute it exactly.
Do not proceed past it until it has completed and written its handoff note.

---

# EXECUTE STAGE 2c

Read lore/stage_2c_subsystem_completion.md and execute it exactly.
Do not proceed past it until it has completed and written its handoff note.

Stage 2c may be run multiple times until all PRIMER-warranted subsystems
have analysis_complete: YES. Each pass picks up where the last left off.

---

# EXECUTE STAGE 3

Read lore/stage_3_human_gate.md and execute it exactly.
Do not proceed past it until it has completed and written its handoff note.

---

# EXECUTE STAGE 4

Read lore/stage_4_lore.md and execute it exactly.
Do not proceed past it until it has completed and written its handoff note.

---

# EXECUTE STAGE 5

Read lore/stage_5_qa.md and execute it exactly.
Do not proceed past it until it has completed and written its handoff note.

---

# EXECUTE STAGE 6

Read lore/stage_6_update.md and execute it exactly.
Do not proceed past it until it has completed and written its handoff note.

---

# EXECUTE STAGE 7

Read lore/stage_7_brief.md and execute it exactly.
Do not proceed past it until it has completed and written its handoff note.

---

# Notes for All Stages

- Read lore/SCHEMAS.md before writing any work file. Match schemas exactly.
- Read lore/CONCEPTS.md if any pipeline term is unclear before proceeding.
- Tag every factual claim with [src: filename] or [inferred].
- Do not read files outside the project root unless explicitly instructed.
- Do not write to any file outside docs/.LORE/work/ and docs/ unless explicitly instructed.
- Do not assign read policy FULL to data files, lock files, or compiled output.
- Do not assign read policy HEAD or SKIP to source code files.
- No documentation is ever produced for a file that was not read with policy FULL.
- If you encounter an ambiguity not covered by these instructions, choose the
  conservative option (do less, flag the ambiguity in the journal) rather than
  improvising.
- If context budget is running low, write what you have, mark incomplete sections
  clearly with [INCOMPLETE - context budget reached], update the journal to PARTIAL,
  and output the standard handoff note for the current stage.
- Only use ASCII characters in all output files.