# Stage 7 - Project Brief Generation
# Read this file and execute it exactly. Do not skip steps.
# Do not begin writing output until Step 3.
# Pipeline reference: lore/LORE.md



---

# Purpose

This stage produces docs/LORE_BRIEF.md - the AI session entry-point document.

LORE_BRIEF.md is shared with an AI coding assistant at the start of a development
session. It gives the AI collaboration guidelines, a concise project orientation,
and a directed reading sequence covering the other LORE documents. The AI reads
the brief, follows the reading sequence, and acknowledges completion before
beginning work.

This stage may be run:
    - Automatically at the end of a full pipeline run (after stage 5 completes)
    - Standalone via MENU D option [B] on an existing doc set
    - After a scoped update (stage 6) if LORE_BRIEF.md needs refreshing

---

# Prerequisites

Before executing this stage:

1. Read lore/SCHEMAS.md in full. Know all schemas before writing anything.

2. Read docs/.LORE/work/JOURNAL.md. Verify:
   - status is IN_PROGRESS or COMPLETE
   - LORE_ENCHIRIDION.md is listed in Artifacts as EXISTS
   - LORE_ABSTRACT.md is listed in Artifacts as EXISTS

   If either document is MISSING, output:
       "Stage 7 prerequisite failed: LORE_ABSTRACT.md and LORE_ENCHIRIDION.md
        must exist before generating the project brief. Run stages 4 and 5 first."
   Stop.

3. Read docs/.LORE/work/ANALYSIS.md in full. You will use:
   - Code Style and Organization section (for project-specific collaboration rules)
   - Subsystems section (for the reading sequence and subsystem list)
   - Observed Bugs and Risks (for known issues to flag in the brief)

4. Read docs/.LORE/work/SKELETON.md. You will use:
   - Project Identity section (name, language, framework, git_available)
   - Git Metadata section (if git_available is YES)

5. Read lore/templates/BRIEF_template.md as your scaffold.

---

# Step 1 - Derive Collaboration Guidelines

The collaboration guidelines section of LORE_BRIEF.md has two parts:
    a) Generic base rules (from the template - do not modify these)
    b) Project-specific rules (derived from ANALYSIS.md Code Style section)

For the project-specific rules, read ANALYSIS.md Code Style and Organization.
Extract the following categories if observable:

    Language and type patterns:
        What language is this project written in?
        Are there explicit type conventions observed in the code?
        Are there any anti-patterns explicitly avoided (e.g. use of "any" in TS,
        untyped function signatures, implicit returns)?
        Tag each observation [src: filename] or [inferred].

    Module and structure patterns:
        How is code organized? (classes, modules, functions, services?)
        Is there a dominant pattern for grouping related logic?
        Tag each observation [src: filename] or [inferred].

    Framework conventions:
        If a framework is detected, what are the non-obvious conventions?
        What does "correct" use of the framework look like in this codebase?
        Tag each observation [src: filename] or [inferred].

    Size and complexity conventions:
        Are there any observed norms for file length, function length,
        or module scope? If not observed, omit this sub-section.
        Tag each observation [src: filename] or [inferred].

    Error handling patterns:
        What is the observed error handling convention?
        (exceptions vs result types, logging patterns, etc.)
        Tag each observation [src: filename] or [inferred].

    Async patterns:
        What is the observed async convention? (async/await, promises, callbacks?)
        Tag each observation [src: filename] or [inferred].

Record these as a structured list. Each item will become a project-specific
rule in the collaboration guidelines. Do not fabricate rules - only write
rules that are directly supported by ANALYSIS.md observations.

If ANALYSIS.md Code Style section is INCOMPLETE or sparse, note this.
Only include rules for patterns actually observed. Omit any category where
no clear pattern was found.

---

# Step 2 - Build the Reading Sequence

The reading sequence tells the AI which documents to read before beginning
work. It must be ordered: higher-level orientation first, then technical
depth.

Build the sequence as follows:

    1. docs/LORE_ABSTRACT.md
       Always first. Gives the overall project context.

    2. docs/LORE_ENCHIRIDION.md
       Always second. Full subsystem catalog and technical reference.

    3. For each PRIMER listed in JOURNAL.md Artifacts as EXISTS:
       Add one entry per PRIMER in order of PRIMER score (highest first).
       If scores are not available from this context, use alphabetical order.
       Format: docs/LORE_PRIMER_[name].md

    4. For each GUIDE listed in JOURNAL.md Artifacts as EXISTS:
       Add one entry per GUIDE, alphabetical order.
       Format: docs/LORE_GUIDE_[name].md

Note: LORE_INDEX.md is not included in the reading sequence. It is navigation,
not content, and the brief already provides the reading sequence directly.

Record the full ordered list. Count the entries. The reading sequence
acknowledgement in the brief will reference this count.

---

# Step 3 - Produce LORE_BRIEF.md

Load lore/templates/BRIEF_template.md as your scaffold.
Replace every placeholder with real content.
Do not preserve any template scaffolding markers in the output.
The output is a finished document.

## Content rules

    - Audience: an AI coding assistant opening a fresh session.
    - Tone: direct and imperative. No narrative prose in instruction sections.
      The AI should be able to scan this document quickly.
    - The generic base rules section must be reproduced from the template
      verbatim. Do not edit, shorten, or rephrase the base rules.
    - The project-specific rules section is derived from Step 1.
      Only include rules backed by ANALYSIS.md observations.
      Keep each rule to one or two sentences. Cite the source.
    - The reading sequence must exactly match the ordered list from Step 2.
      No additions, no omissions.
    - The acknowledgement markers (ingestion token and completion token) must
      be reproduced exactly from the template. Do not rephrase them.
    - Use only ASCII characters.
    - No unicode. No emojis.

Write the file to docs/LORE_BRIEF.md.
Validate it exists before continuing.

---

# Step 4 - Update JOURNAL.md

Update docs/.LORE/work/JOURNAL.md:

    last_updated: current timestamp (YYYY-MM-DD HH:MM)
    last_stage_completed: stage_7_brief.md
    status: COMPLETE

    Completed Stages: append one of:
        - stage_7_brief.md: DONE
        - stage_7_brief.md: PARTIAL - [brief reason]

    Artifacts section: overwrite in full.
        Add or update: docs/LORE_BRIEF.md: EXISTS
        All other artifacts retain their current state.

---

# Step 5 - Validate Output

Before writing the handoff note, verify:

    [ ] docs/LORE_BRIEF.md exists and is not empty
    [ ] LORE_BRIEF.md begins with the ingestion token prompt (not the template header)
    [ ] LORE_BRIEF.md contains the generic base rules section verbatim
    [ ] LORE_BRIEF.md contains the project-specific rules section
        (or a note that no project-specific rules were derivable)
    [ ] LORE_BRIEF.md contains the reading sequence from Step 2 in full
    [ ] LORE_BRIEF.md ends with the completion acknowledgement token
    [ ] No template scaffolding markers remain in the output
        (check for: [PLACEHOLDER], [INSERT, {{, }}, [TEMPLATE INSTRUCTIONS])
    [ ] No unicode characters or emojis in the output
    [ ] JOURNAL.md status is COMPLETE
    [ ] JOURNAL.md last_stage_completed is stage_7_brief.md
    [ ] docs/LORE_BRIEF.md is listed in JOURNAL.md Artifacts as EXISTS

If any check fails: fix it before writing the handoff note.

---

# Step 6 - Write the Handoff Note

Write the following block exactly, filling in the bracketed fields.
Copy it into the Last Handoff Note section of JOURNAL.md.
Output it as the final output of this stage.

---
Stage 7 complete - Project Brief
Project: [project_name]
LORE: Stage [7/7] Context [N]
Status: COMPLETE - pipeline finished

Documents delivered:
    - docs/LORE_ABSTRACT.md:     EXISTS
    - docs/LORE_ENCHIRIDION.md:  EXISTS
    - docs/LORE_INDEX.md:        EXISTS
    [for each PRIMER:]
    - docs/LORE_PRIMER_[name].md: EXISTS
    [for each GUIDE:]
    - docs/LORE_GUIDE_[name].md: EXISTS
    - docs/LORE_BRIEF.md:        EXISTS

Reading sequence in LORE_BRIEF.md: [count] documents
Project-specific collaboration rules derived: [count or "none"]
Tokens this context: ~[N]k (estimated)

To begin an AI development session, share docs/LORE_BRIEF.md with your
AI assistant. It will read the listed documents and confirm when ready.

The pipeline is complete. Documentation is ready for use.
---

---

# DO NOT

- Do not invent collaboration rules not supported by ANALYSIS.md observations
- Do not modify the generic base rules from the template
- Do not include LORE_INDEX.md in the reading sequence
- Do not produce any other output files during this stage
- Do not modify any file in lore/ other than docs/.LORE/work/JOURNAL.md
- Do not run this stage if LORE_ABSTRACT.md or LORE_ENCHIRIDION.md do not exist
- Do not use unicode characters or emojis in any output
- Do not omit the ingestion token or the completion token from the output file