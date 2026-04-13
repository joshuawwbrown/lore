# LORE Documentation Pipeline
# Schema Version: 1.6

A continuous documentation factory. For details, see `lore/CONCEPTS.md`.

---

## Quick Start

Copy `lore` to the top level of your project.

Open a new AI session and say:

    Please follow `lore/LORE.md` to document this project.

---

## Installing Lore into a Project

To install lore into a project from the public repository, open a new AI
session in your project and say:

    Clone https://github.com/joshuawwbrown/lore.git to a temporary location,
    copy the contents into a new lore/ directory at the root of this project,
    then delete the temporary clone. Report what was copied.

---

## Updating from a Pre-Repository Version of Lore

If lore is already installed in your project but predates the public repository
(there is no `lore/VERSION` file), open a new AI session in your project and say:

    Clone https://github.com/joshuawwbrown/lore.git to a temporary location,
    copy the contents into the existing lore/ directory at the root of this project,
    overwriting any files that already exist, then delete the temporary clone.
    Report what was added or changed.

---

## Updating Lore

If lore is already installed in your project, share this file with your AI
assistant and say:

    Please follow `lore/LORE_UPDATE.md` to update lore.

A human with a terminal can also follow `lore/LORE_UPDATE.md` directly.

---

## What it does

LORE reads every source file in the project and produces human-facing and
AI-agent-facing documentation in the `docs` directory at the same level as
the `lore` directory. It will create `docs` if it does not exist. It will
not overwrite files it did not create.

A resource estimate (file count, context windows, tokens) is produced at
the end of stage 1 so you can decide whether to proceed before any analysis
begins.

---

## Files in this directory

    lore/README.md                        - this file (human-facing)
    lore/VERSION                          - current lore version string
    lore/LORE.md                          - documentation pipeline reference
    lore/LORE_UPDATE.md                   - instructions for updating lore to latest version
    lore/PLAN.md                          - development plan pipeline reference
    lore/CONCEPTS.md                      - glossary and terminology for LORE
    lore/SCHEMAS.md                       - work file schemas (single source of truth)
    lore/stage_1_recon.md                 - stage 1: reconnaissance and resource estimate
    lore/stage_2_analysis.md              - stage 2: deep analysis
    lore/stage_2b_continuation.md         - stage 2b: overflow continuation pass
    lore/stage_2c_subsystem_completion.md - stage 2c: subsystem completion gate
    lore/stage_3_human_gate.md            - stage 3: human question review
    lore/stage_4_lore.md                  - stage 4: documentation production
    lore/stage_5_qa.md                    - stage 5: quality control
    lore/stage_6_update.md                - stage 6: scoped update pass
    lore/stage_7_brief.md                 - stage 7: AI session brief generation
    lore/templates/                       - output document scaffolds
    docs/.LORE/work/                      - pipeline work files (created at runtime)
    docs/plans/                           - plan documents (created at runtime)

---

## Output documents

All output files are prefixed with `LORE_` and written to `docs/`.

    docs/LORE_ABSTRACT.md              - human-facing project overview
    docs/LORE_BRIEF.md                 - AI session entry point and reading sequence
    docs/LORE_ENCHIRIDION.md           - AI-agent-facing technical reference
    docs/LORE_INDEX.md                 - index of all docs
    docs/LORE_PRIMER_[name].md         - deep reference per subsystem (if warranted)
    docs/LORE_GUIDE_[name].md          - operator guide per runnable subsystem

---

## Continuing after a stage completes

Each stage outputs the exact prompt to use for the next context window.
The pipeline runs 7 stages (plus overflow stages 2b and 2c):

    1  recon -> 2  analysis -> 2b/2c completion -> 3  human gate
    -> 4  docs -> 5  QA -> 6  update (on changes) -> 7  brief


    Please follow `lore/LORE.md` and CONTINUE.


The word CONTINUE causes the pipeline to skip the menu and execute the next
stage automatically.

---

## Planning a change to the project

To create a disciplined development plan for a new feature, refactor,
dependency upgrade, or any other change, open a new AI session and say:

    Please follow `lore/PLAN.md` to create a plan.

The plan pipeline will ask you 6 questions about the change, read the
relevant project documentation, and produce a set of documents in docs/plans/:

    PLAN_[label].md           -- full specification (file specs, phases, checklist)
    PLAN_[label]_PROPOSAL.md  -- human-facing approval document (verbatim key
                                 sections, inline reference material, phase summary)
    PLAN_[label]_journal_1.md -- running log of all events, decisions, deviations
    PLAN_[label]_DEPLOY.md    -- human operator deployment runbook (written at
                                 the end of implementation, only if [D] was
                                 chosen at approval time -- not during planning)

The PROPOSAL is what you read and approve before any code is written. It
contains everything needed to catch mistakes: goal, files changed, constraints,
reference material, known risks, and phase summary -- all inline, no hunting
through the full plan.

To continue an existing plan:

    Please follow `lore/PLAN.md` and CONTINUE.

To resume implementation of a plan that is already approved:

    Please follow `docs/plans/PLAN_[label].md` and CONTINUE.

See lore/PLAN.md for full pipeline documentation.

---

## Starting an AI development session

After the pipeline has run, share `docs/LORE_BRIEF.md` with your AI assistant
at the start of each development session. The AI will read the listed documents
and confirm with `ALL DOCUMENTATION INGESTED` when ready to begin work.

---

## Updating docs after code changes (including after a plan completes)

Open a new AI session and say:

    Please follow `lore/LORE.md` to update the project documentation.

## Re-running quality control only

Open a new AI session and say:

    Please follow `lore/LORE.md` to re-run quality control on the project documentation.