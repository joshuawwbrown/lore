# MODIFYING LORE
# How to safely make local modifications to the lore/ pipeline files.
# Read this file before changing any file in lore/.

---

## Overview

The lore/ directory is a self-contained documentation pipeline. Its files
are versioned and may be updated from the public repository at any time.
Local modifications must be made carefully so that:

    - The pipeline continues to function correctly
    - Version tracking remains accurate
    - AI assistants reading lore/ files know what version they are working with
    - Future updates from the public repository do not silently overwrite
      local changes (LORE_UPDATE.md flags conflicts before writing)

---

## Two Version Numbers

Lore maintains two separate version numbers. Know which one applies to
your change before you touch anything.

### lore/VERSION

    Location:   lore/VERSION
    Tracks:     The lore pipeline files themselves (stages, templates,
                SCHEMAS.md, CONCEPTS.md, LORE.md, PLAN.md, etc.)
    Format:     MAJOR.MINOR.PATCH  (example: 1.6.1)
    Who uses it: LORE_UPDATE.md update procedure compares this value
                against the public repository to detect whether an update
                is needed.

Bump lore/VERSION whenever you modify ANY file in lore/.
Use a patch increment (1.6 -> 1.6.1) for small changes.
Use a minor increment (1.6 -> 1.7) for significant additions or restructuring.

### Schema version in lore/SCHEMAS.md

    Location:   lore/SCHEMAS.md, header line "# Schema Version: X.Y"
                Also stamped into docs/.LORE/work/JOURNAL.md at pipeline start.
    Tracks:     The structure of the work files produced by the pipeline
                (JOURNAL.md, ANALYSIS.md, MANIFEST.md, SKELETON.md, etc.)
    Who uses it: The pipeline checks JOURNAL.md schema_version against
                SCHEMAS.md on every run. A mismatch triggers MENU F
                (schema migration) in lore/LORE.md.

The schema version is NOT the same as lore/VERSION. Most changes to pipeline
files do not require a schema bump. See "When to bump the schema version" below.

---

## Rules for Every Modification

1.  Read this file before making any change.

2.  Make your change.

3.  Always bump lore/VERSION after any change to any file in lore/.
    No exceptions. This is what allows the update procedure in
    LORE_UPDATE.md to detect that local modifications have been made.

4.  If your change affects the structure of any work file schema
    (see "When to bump the schema version" below), also bump the schema
    version in lore/SCHEMAS.md and add a migration entry.

5.  If your change affects instructions that AI assistants must follow
    verbatim (acknowledgement tokens, stage output formats, handoff note
    format), verify that all references to the changed text are updated
    consistently across all stage files and templates.

---

## When to Bump the Schema Version

Bump the schema version in lore/SCHEMAS.md only when a change affects the
structure of a work file in docs/.LORE/work/. Specifically:

    Bump schema version if you:
        - Add, remove, or rename a field in any work file schema
        - Add, remove, or rename a section in any work file schema
        - Change the format of an existing field (e.g. value set, date format)
        - Add a new work file that stages must produce or consume

    Do NOT bump schema version if you:
        - Reword instructions in a stage file
        - Change a label or phrase in a template output file
        - Add or modify a DO NOT rule
        - Add a new template for an output document (docs/, not work files)
        - Change lore/CONCEPTS.md, lore/README.md, or lore/LORE_UPDATE.md
        - Fix a typo

When in doubt, ask: "Does this change require existing docs/.LORE/work/
files to be edited before the pipeline can run correctly?" If yes, bump
the schema version. If no, do not.

---

## How to Bump the Schema Version

1.  Open lore/SCHEMAS.md.

2.  Before changing anything else, add a new migration entry at the top
    of the MIGRATION LOG section (after the section header and format
    comment, before the existing entries). Use this format exactly:

        ## Migration: [old version] -> [new version]
            Date: [YYYY-MM-DD]
            Changed files: [list of work files affected]
            Instructions:
                [file]: [DROP field X | ADD field Y with default Z | RENAME A to B | CONVERT format]

3.  Update the schema version string in the SCHEMAS.md header:

        # Schema Version: [new version]

4.  Update the schema version string in lore/LORE.md header to match:

        # Schema Version: [new version]

5.  Update the schema version string in lore/CONCEPTS.md:
        - Header line
        - "Current version: X.Y" line in the "schema version" section

6.  Bump lore/VERSION as required by the general rules above.

Existing projects running an older schema version will be detected
automatically by the pipeline on next run and routed to MENU F for migration.

---

## Files You Are Most Likely to Modify

    lore/templates/BRIEF_template.md      - scaffold for LORE_BRIEF.md output
    lore/templates/ABSTRACT_template.md   - scaffold for LORE_ABSTRACT.md output
    lore/templates/ENCHIRIDION_template.md - scaffold for LORE_ENCHIRIDION.md output
    lore/templates/PRIMER_template.md     - scaffold for LORE_PRIMER_[name].md output
    lore/templates/GUIDE_template.md      - scaffold for LORE_GUIDE_[name].md output
    lore/stage_7_brief.md                 - instructions for brief generation stage
    lore/SCHEMAS.md                       - work file schemas and migration log
    lore/CONCEPTS.md                      - glossary

    Changes to any of these require a VERSION bump.
    Changes to SCHEMAS.md field or section definitions require a schema version bump.

---

## Local Modifications and the Update Procedure

lore/LORE_UPDATE.md describes how to pull updates from the public repository.
That procedure compares file contents and reports conflicts before writing.

If you have modified a lore file locally and the public repository has also
changed that file, LORE_UPDATE.md instructs the AI to flag it as a conflict
and ask you how to proceed. Your local changes will not be silently overwritten.

The update procedure always performs a full content diff regardless of whether
lore/VERSION matches. The version comparison is informational only. Bumping
lore/VERSION after every local change is still required - it signals to
collaborators and AI agents that the local copy has diverged from the published
release, and it will appear in the diff report as a changed file.