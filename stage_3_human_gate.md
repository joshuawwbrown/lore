# Stage 3 - Human Gate
# Read this file and execute it exactly. Do not skip steps.
# This stage does not answer questions. It verifies the human has answered them.
# Pipeline reference: lore/LORE.md



---

# Purpose

This stage reads docs/.LORE/work/QUESTIONS.md, presents unanswered questions to the
human organized by tier, records answers in JOURNAL.md, and routes the pipeline
to stage 4 (LORE production) when the gate is cleared.

This stage does not produce documentation. It does not answer questions on
the human's behalf. Its only job is to verify that Tier 1 questions have
been addressed before documentation production begins.

---

# Prerequisites

Before executing this stage:

1. Read lore/SCHEMAS.md in full. You must know the exact schemas for
   QUESTIONS.md, JOURNAL.md, and REVIEW_NOTES.md before writing anything.

2. Read docs/.LORE/work/JOURNAL.md. Verify:
   - status is IN_PROGRESS
   - last_stage_completed is stage_2_analysis.md, stage_2b_continuation.md,
     or stage_2c_subsystem_completion.md

   If status is not IN_PROGRESS, stop and output:
   "Stage 3 prerequisite failed: JOURNAL.md status is [status]. Expected IN_PROGRESS."

   If ANALYSIS.md is listed as MISSING, stop and output:
   "Stage 3 prerequisite failed: ANALYSIS.md does not exist. Run stage 2 first."

3. Read docs/.LORE/work/ANALYSIS.md. For every subsystem entry, check
   analysis_complete and PRIMER warranted fields.

   For every subsystem where PRIMER warranted is YES:
       If analysis_complete is NO: stop and output:
           "Stage 3 prerequisite failed: subsystem [name] has
            analysis_complete: NO (files_read [N of M]).
            Run stage_2c_subsystem_completion.md until all PRIMER-warranted
            subsystems have analysis_complete: YES before proceeding."

   All PRIMER-warranted subsystems must have analysis_complete YES before
   this stage may continue. This check cannot be bypassed.

4. Read docs/.LORE/work/QUESTIONS.md in full.
   Count questions by tier and by answer state:
       - Total questions
       - Tier 1 questions with blank ANSWER fields (blocking)
       - Tier 2 questions with blank ANSWER fields (enhancing)
       - Tier 3 questions with blank ANSWER fields (cosmetic)
   Record these counts. You will use them in the gate report.

5. Read docs/.LORE/work/ANALYSIS.md again for questions flagged as potentially
   answerable from code (look for [Note: Q[number] may be answerable from
   code] tags). These should be surfaced to the human alongside the
   relevant QUESTIONS.md entry.

---

# Step 1 - Produce the Gate Report

Output the following report to the user. Fill in all bracketed fields.
Do not skip this step even if all questions are already answered.

---
HUMAN GATE - Stage 3

Project: [project_name from JOURNAL.md]
Analysis complete: [last_stage_completed from JOURNAL.md]

QUESTIONS SUMMARY
    Total questions: [count]
    Tier 1 (blocking):  [count] questions, [answered count] answered, [unanswered count] unanswered
    Tier 2 (enhancing): [count] questions, [answered count] answered, [unanswered count] unanswered
    Tier 3 (cosmetic):  [count] questions, [answered count] answered, [unanswered count] unanswered

[If all Tier 1 questions are answered:]
    All Tier 1 questions are answered. The pipeline can proceed to documentation production.

[If any Tier 1 questions are unanswered:]
    WARNING: [count] Tier 1 question(s) are unanswered.
    Documentation cannot be fully accurate until these are resolved.

---

# Step 2 - Present Unanswered Questions by Tier

If any questions have blank ANSWER fields, present them now grouped by tier.

Present them in this exact format for each tier that has unanswered questions.
Do not present tiers that have no unanswered questions.

---
TIER 1 - BLOCKING QUESTIONS
These must be answered before documentation production begins.
If you skip these, document accuracy cannot be guaranteed.

[For each unanswered Tier 1 question:]

    Q[number]
    Question: [question text from QUESTIONS.md]
    Context: [context from QUESTIONS.md]
    [If flagged as potentially answerable from code:]
    Note: This question may be partially addressable from code. See ANALYSIS.md
    for context before answering.
    ANSWER: [leave blank - human fills this in]
    CONFIDENCE: [leave blank - or add ??? after your answer if you are uncertain]

---
TIER 2 - ENHANCING QUESTIONS
These are optional but improve documentation accuracy.
If unanswered, stage 4 will use the default assumption listed.

[For each unanswered Tier 2 question:]

    Q[number]
    Question: [question text from QUESTIONS.md]
    Context: [context from QUESTIONS.md]
    Default assumption if unanswered: [default from QUESTIONS.md]
    ANSWER: [leave blank - human fills this in]
    CONFIDENCE: [leave blank - or add ??? after your answer if you are uncertain]

---
TIER 3 - COSMETIC QUESTIONS
These add polish only. Safe to skip entirely.

[For each unanswered Tier 3 question:]

    Q[number]
    Question: [question text from QUESTIONS.md]
    ANSWER: [leave blank - human fills this in]
    CONFIDENCE: [leave blank - or add ??? after your answer if you are uncertain]

---

After presenting the questions, output exactly the following instruction to
the user:

    Please answer the questions above, then return here and respond with one
    of the following:

        [A] I have answered the questions - paste your answers below or
            confirm that you have updated QUESTIONS.md directly.
        [K] Skip all remaining unanswered questions and proceed with defaults.
        [X] Cancel - stop the pipeline here.

Wait for the human to respond before continuing.

---

# Step 3 - Accept and Record Human Answers

This stage handles three response paths.

## Path A - Human provides answers

If the human chooses [A] and provides answers inline:

    For each question the human answers inline:
        Write the answer into the ANSWER field in docs/.LORE/work/QUESTIONS.md.
        Inspect the answer text:
            If the answer ends with ???:
                Set CONFIDENCE to ???
            Else:
                Leave CONFIDENCE blank (blank means authoritative - high confidence)
        Preserve the exact question text, context, and structure.
        Do not edit any field except ANSWER and CONFIDENCE.

    If the human says they updated QUESTIONS.md directly:
        Re-read docs/.LORE/work/QUESTIONS.md.
        Verify that ANSWER fields are now populated for the claimed questions.
        If any claimed answer is still blank, report which ones are still blank
        and ask the human to confirm or re-provide.

    After recording answers, proceed to Step 4 (gate check).

## Path K - Human skips remaining questions

If the human chooses [K]:

    Do not modify any ANSWER fields in QUESTIONS.md.
    Update docs/.LORE/work/JOURNAL.md:
        Append to the Completed Stages section:
            - stage_3_human_gate.md: DONE - human skipped [count] unanswered questions
        Add a note: "Stage 4 will use default assumptions for all unanswered
        Tier 2 questions. Tier 1 gaps will be flagged in output documents."
    Proceed to Step 5 (update journal and route to stage 4).

## Path X - Human cancels

If the human chooses [X]:

    Update docs/.LORE/work/JOURNAL.md:
        status: AWAITING_HUMAN
        last_stage_completed: stage_3_human_gate.md
        last_updated: [current timestamp]
        Append to Completed Stages:
            - stage_3_human_gate.md: PARTIAL - cancelled by human at question review

    Output exactly:
        Pipeline paused at human gate. Status set to AWAITING_HUMAN.
        When ready, run LORE again and choose option [A] from MENU C.
    Stop.

---

# Step 4 - Gate Check

Read docs/.LORE/work/QUESTIONS.md again after any answer recording.

Count Tier 1 questions with blank ANSWER fields.

If any Tier 1 questions still have blank ANSWER fields:

    Output:
        [count] Tier 1 question(s) still unanswered:
        [list the Q-numbers and one-line question text for each]

        These questions are blocking. Options:
        [A] Provide answers now (paste them below).
        [K] Skip all and proceed with defaults and accuracy warnings.
        [X] Cancel and save pipeline state.

    Wait for the human to respond and process accordingly.
    Repeat this step until no Tier 1 questions remain unanswered,
    or the human chooses [K] or [X].

If all Tier 1 questions are answered (or [K] was chosen):

    Proceed to Step 5.

---

# Step 5 - Mark Questions Resolved in JOURNAL.md

Update docs/.LORE/work/JOURNAL.md:

    Clear the Unanswered Questions section completely.

    For each question in QUESTIONS.md that still has a blank ANSWER field
    (Tier 2 or Tier 3 questions not answered), add it to the Unanswered
    Questions section with a note:
        - Q[number] [TIER-2 | TIER-3]: [one-line summary] - will use default

    For questions answered during this stage, add a note in the journal's
    Completed Stages entry (not in Unanswered Questions):
        - Questions answered this session: [count] ([T1 count] Tier 1,
          [T2 count] Tier 2, [T3 count] Tier 3)

    Update:
        last_updated: [current timestamp in YYYY-MM-DD HH:MM format]
        last_stage_completed: stage_3_human_gate.md
        status: IN_PROGRESS

    Append to Completed Stages:
        - stage_3_human_gate.md: DONE

    Update Artifacts section:
        - docs/.LORE/work/QUESTIONS.md: EXISTS
        (all other artifacts unchanged from their current state)

---

# Step 6 - Produce a Question Resolution Summary for Stage 4

Write a brief resolution summary that stage 4 will be able to use as
a quick reference. Append it to docs/.LORE/work/JOURNAL.md under a section
titled "Question Resolution Summary". This section replaces any prior
version of itself if this stage is re-run.

Format:

    ### Question Resolution Summary
    Produced by: stage_3_human_gate.md

    Tier 1 resolutions:
    [For each Tier 1 question:]
        Q[number]: [ANSWERED | SKIPPED]
        [If ANSWERED:] Summary: [one-sentence paraphrase of the answer]
        [If SKIPPED:] Stage 4 action: flag accuracy gap in relevant section

    Tier 2 defaults applied (unanswered):
    [For each unanswered Tier 2 question:]
        Q[number]: using default - [default assumption text from QUESTIONS.md]

    Tier 3 open questions:
    [For each unanswered Tier 3 question:]
        Q[number]: left open - will appear as open question in final docs

    Subsystem dispositions:
    [For each subsystem in ANALYSIS.md, record its disposition:]
        [subsystem name]: [active | deprecated | ignored]
    [If any answer during this session changed a subsystem's disposition from
     the default of "active", note the change and the question that drove it:]
        [subsystem name]: changed to [deprecated | ignored] per Q[number]
    [If no dispositions changed from active:]
        all subsystems: active (no disposition changes this session)

---

# Step 6b - Write Ignored Subsystems Record

If any subsystem has disposition: ignored (either set in ANALYSIS.md before
this stage or changed by a human answer during this session):

    Append to docs/.LORE/work/JOURNAL.md a section titled
    "Ignored Subsystems":

        ### Ignored Subsystems
        [For each ignored subsystem:]
        - [subsystem name]: [one-line reason] [per Q[number] if applicable]

    These subsystems will not appear in any output document. Stage 4 must
    read this section before producing any output and skip all ignored
    subsystems entirely.

If no subsystems are ignored, do not write this section.

---

# Step 7 - Validate Output

Before writing the handoff note, verify:

    [ ] docs/.LORE/work/QUESTIONS.md has been updated with all inline answers recorded
    [ ] JOURNAL.md status is IN_PROGRESS
    [ ] JOURNAL.md last_stage_completed is stage_3_human_gate.md
    [ ] Question Resolution Summary section is present in JOURNAL.md
    [ ] Question Resolution Summary includes Subsystem dispositions block
    [ ] If any subsystems are ignored: Ignored Subsystems section present in JOURNAL.md
    [ ] Unanswered Questions section in JOURNAL.md is accurate
    [ ] No question was deleted from QUESTIONS.md (only ANSWER fields were filled)
    [ ] No question text was modified (only ANSWER fields were filled)

If any check fails: fix it before writing the handoff note.

---

# Step 8 - Write the Handoff Note

Write the following block exactly, filling in all bracketed fields.
Copy it into the Last Handoff Note section of JOURNAL.md (overwrite prior note).
Output it as the final output of this stage.

---
Stage 3 complete - Human Gate
Codebase: [total_source_files] files ([languages])
LORE: Stage [3/5] Pass [1] Context [N]
Status: COMPLETE - gate cleared, proceed to Stage 4
[If CLEARED WITH SKIPS:]
    Note: [count] questions skipped. Stage 4 will apply defaults for Tier 2,
    flag gaps for any skipped Tier 1.

Questions answered this session: [count]
PRIMER-warranted subsystems (all analysis_complete YES): [count] ([names])
Deprecated subsystems (stub only, no PRIMER): [count or "none"] ([names])
Ignored subsystems (excluded from all output): [count or "none"] ([names])
Tokens this context: ~[N]k (estimated)
Remaining tokens (est.): [low]k-[high]k across [N] more contexts

Please start a new context with this text:

Please follow `lore/LORE.md` and CONTINUE.
---

---

# DO NOT

- Do not answer human questions yourself, even if the answer seems obvious
  from the code you have read
- Do not change any field in QUESTIONS.md other than ANSWER and CONFIDENCE
- Do not reorder, renumber, or delete any question from QUESTIONS.md
- Do not modify ANALYSIS.md or SKELETON.md during this stage
- Do not write any files in docs/ - that is stage 4's job
- Do not bypass the analysis_complete prerequisite check under any circumstances
- Do not proceed to stage 4 in this same context window unless the handoff
  note above has been written and output to the user
  (output the handoff note and stop)
- Do not infer answers from code and silently populate ANSWER fields
  without explicit human input