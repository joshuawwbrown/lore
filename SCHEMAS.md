# LORE Work File Schemas
# Single source of truth for all inter-stage artifacts
# Schema Version: 1.7
# All stage files reference this document. Do not define schemas elsewhere.

---

# How to Use This File

Each schema below defines the exact structure of a work file in docs/.LORE/work/.
Stages that produce a file must match its schema exactly.
Stages that consume a file must expect exactly this structure.

When this file is updated to a new version, a migration entry must be added
to the MIGRATION LOG section at the bottom of this file.

---

# Schema: JOURNAL.md

Path: docs/.LORE/work/JOURNAL.md
Created by: stage_1_recon.md (initial creation)
Updated by: every stage (append only, except status field which is overwritten)
Read by: lore/START.md (state detection), every stage (prerequisite check)
         stage_6_update.md (update mode)

## Fields

project_name: string
    The detected name of the project. Source: package.json, Cargo.toml, go.mod,
    or the root directory name if no manifest is found.

project_root: string
    The relative path to the root of the project being documented.
    Example: app/

schema_version: string
    The version of SCHEMAS.md in use for this run.
    Must match the version declared at the top of this file.
    Example: 1.0

started: string
    A timestamp string. Produced by creating a temp file and reading its
    filesystem creation time. Format: YYYY-MM-DD HH:MM (approximate is fine).
    If creation time cannot be read, use the string "unknown".

last_updated: string
    Same format as started. Overwritten by each stage on completion.

last_stage_completed: string
    The filename of the last stage that completed successfully.
    Example: stage_1_recon.md
    Value is "none" if no stage has completed.

status: string
    One of exactly these values:
        NEW             - journal just created, no stage complete
        IN_PROGRESS     - a stage is underway or was interrupted
        AWAITING_HUMAN  - pipeline is paused at the human gate
        COMPLETE        - all stages finished, docs produced
        ERROR           - a stage reported an unrecoverable problem

last_error: string
    A plain English description of the last error encountered.
    Value is "none" if no error has occurred.

files_read_total: integer (or "unknown" if not yet tracked)
    The cumulative count of source files read with policy FULL across all
    analysis stages (2, 2b, 2c) for this run. Updated by each analysis stage
    on completion. Used to verify the handoff note's "Codebase: N files" claim
    without reading MANIFEST.md separately.
    Initial value: 0 (set by stage_1_recon.md).
    Updated to "unknown" only if MANIFEST.md is unavailable.

## Sections (append-only except where noted)

### Completed Stages
A bullet list. Each entry added when a stage finishes.
Format: - [stage filename]: [DONE | PARTIAL] [optional note]
Example:
    - stage_1_recon.md: DONE
    - stage_2_analysis.md: PARTIAL - 3 files flagged but not read, context budget reached

### Artifacts
A bullet list of expected output files and their current state.
Overwritten in full by each stage that produces or modifies artifacts.
Format: - [filepath]: [EXISTS | MISSING | PARTIAL]
Example:
    - docs/.LORE/work/SKELETON.md: EXISTS
    - docs/.LORE/work/ANALYSIS.md: PARTIAL
    - docs/.LORE/work/QUESTIONS.md: EXISTS
    - docs/LORE_ABSTRACT.md: MISSING
    - docs/LORE_ENCHIRIDION.md: MISSING
    - docs/LORE_INDEX.md: MISSING
    - docs/LORE_BRIEF.md: MISSING

### Unanswered Questions
A bullet list of question IDs from QUESTIONS.md that have not yet been answered.
Format: - Q[number]: [one-line summary of question]
Cleared when the human gate stage marks questions resolved.
Example:
    - Q3: What is the intended production deployment environment?
    - Q7: Is there a staging environment separate from production?

### Last Handoff Note
The exact handoff text output by the last completed stage.
Overwritten each time a stage completes.
This is what the user copies into the next context window.

---

# Schema: SKELETON.md

Path: docs/.LORE/work/SKELETON.md
Created by: stage_1_recon.md
Updated by: stage_2_analysis.md (corrections section appended)
Read by: stage_2_analysis.md, stage_4_lore.md

## Sections

### Project Identity
Fields written as plain labeled lines:
    name: [project name]
    language: [primary language or comma-separated list if polyglot]
    runtime: [e.g. Node.js 22, Python 3.11, Go 1.22]
    framework: [detected framework name, or "none detected"]
    package_manager: [npm, cargo, pip, go modules, etc.]
    entry_points: [comma-separated list of entry point files]
    test_framework: [detected test framework, or "none detected"]
    git_available: [YES | NO]
        YES - a .git/ directory was found and the user permitted git commands.
              Git metadata was collected and appears in the Git Metadata section.
        NO  - no .git/ directory found, or user declined git access.
              The Git Metadata section is omitted from this file.

### Dependency Summary
A bullet list of external dependencies grouped by category.
Categories: core runtime, database, web framework, utilities, dev tools, other
Each entry: - [package name] ([version if available]): [one-line purpose]
Source must be cited: [src: filename]

### Directory Map
A tree-style listing of top-level directories and one level of subdirectories.
Each directory annotated with a one-line inferred purpose.
Annotation must be tagged [inferred] or [src: filename] to indicate confidence.
Example:
    app/
        srvr/           - backend server code [inferred]
        ui/             - frontend assets [inferred]
        docs/           - project documentation [src: package.json]
        lore/           - LORE pipeline (this system) [known]

### Entry Point Summaries
For each entry point identified above, a brief summary:
    [filename]
        purpose: [what this file does]
        key exports or routes: [comma-separated list, or "none"]
        source: [src: filename] or [inferred]

### Files Flagged for Deep Read
A numbered list of files to be read in stage_2_analysis.md, in priority order.
Each entry must include a reason and a read policy.
Format:
    1. [filepath] - [reason for flagging] - [read policy: FULL | HEAD | SKIP]
Example:
    1. app/srvr/index.js - primary application entry point - FULL
    2. app/config/default.js - configuration structure reveals data paradigm - FULL
    3. app/data/carriers.json - large lookup table, structure only needed - HEAD

Read policy definitions:
    FULL  - read the entire file
    HEAD  - read first 20 lines only (data files, large lookup tables, generated JSON)
    SKIP  - do not read (lock files, compiled output, node_modules, binaries)

List length guidance:
    The goal is not to maximize files read in stage 2 - it is to read the
    highest-priority files and produce a useful partial ANALYSIS.md that
    stage 2b can extend. Flag enough files to fill approximately one context
    window of reading (roughly 15-20 FULL files for average-sized files;
    fewer for large files, more for small ones). The remainder are handled
    by stage 2b and 2c automatically. A hard cap of 20 is a reasonable
    default; adjust down for codebases with many large files (200+ lines),
    adjust up for codebases with many small files (under 50 lines).

### Resource Estimate
Produced by stage_1_recon.md Step 10b. Records the estimated cost of completing
the full analysis so the human can make an informed decision before proceeding.

Format:
    total_source_files: [count of all non-skipped files across all subsystems]
    files_flagged_stage2: [count of files on the deep-read list]
    files_remaining_after_stage2: [total_source_files minus files_flagged_stage2]
    estimated_context_windows: [low]-[high]
    estimated_tokens: [low]k-[high]k
    basis: [one sentence: how the estimate was derived]
    primer_warranted_subsystems: [count]
    largest_subsystem: [name] ([file count] files)

### Existing Documentation Inventory
A record of what was found in the docs/ directory before stage 4 runs.
Written by stage_1_recon.md. Read by stage_2_analysis.md and stage_4_lore.md.

Format:

    docs/ directory: [EXISTS | NOT FOUND]

    Prior LORE artifacts (will be overwritten by stage 4):
    - [filepath]: [approximate line count or "empty"]
    (or "none found" if no canonical LORE files exist)

    User-created files (will be preserved by stage 4):
    - [filepath]: [approximate line count or "empty"]
    (or "none found" if no user files exist)

    Claims extracted from existing documentation:
    - [claim text] [src: filename]
    (or "none extracted - no readable docs found")

    Reconciliation note for stage 2:
    [one sentence: either "Stage 2 must verify N claims against actual code"
     or "No existing documentation claims to reconcile."]

### Git Metadata
Only present if git_available is YES in Project Identity.
Omit this section entirely if git_available is NO.

Format:

    git_available: YES

    overall_contributors:
        [For each contributor, one line:]
        - [commit count] [author name]
        [or "unavailable" if the command failed or returned no output]

    directory_last_commit:
        [For each top-level source directory:]
        - [directory/]: [YYYY-MM-DD | no commits found]

    directory_primary_author:
        [For each top-level source directory:]
        - [directory/]: [author name | unknown]

All entries tagged [src: git].

### Domain Jargon Candidates
A list of project-specific and domain-specific terms identified during stage 1.
Written by stage_1_recon.md. Read by stage_5_qa.md (check C7).

Format for each entry:
    - [term]: seen in [filename or directory] - [inferred meaning, tagged [inferred]]
    (or "none identified" if no jargon candidates were found)

### Subsystem Candidates
A list of potential subsystems identified from structure alone.
Each entry includes a PRIMER score (0-5) based on these criteria:
    +1 if 5 or more files are dedicated to it
    +1 if referenced by 3 or more other subsystems (estimate from imports)
    +1 if it has a non-obvious pattern or framework convention
    +1 if it has its own external dependencies
    +1 if it is a standalone operational tool with its own configuration
       and more than 5 non-trivial files, independent of the main application
Score of 2 or higher suggests a PRIMER is warranted.
Format:
    - [subsystem name]: score [N/5] - [one-line description] - [reason for score]

### Human Questions (Tier 1 - Blocking)
Questions that cannot be answered by reading code and must be resolved
before stage_4_lore.md can produce accurate documentation.
Format: Q[number] [TIER-1]: [question text]
        Context: [why this question matters to the documentation]
Example:
    Q1 [TIER-1]: What is the intended production hosting environment?
        Context: Deployment docs cannot be accurate without this.

### Human Questions (Tier 2 - Enhancing)
Questions that would improve documentation quality but are not blocking.
Stage 4 will use best-guess defaults if these are unanswered.
Format: Q[number] [TIER-2]: [question text]
        Context: [why this question matters]
        Default assumption: [what stage 4 will assume if unanswered]

### Human Questions (Tier 3 - Cosmetic)
Questions whose answers would add polish but have no impact on accuracy.
These are documented as open questions in the final output.
Format: Q[number] [TIER-3]: [question text]

### Corrections
Added by stage_2_analysis.md if it discovers contradictions with sections above.
Format:
    CORRECTION [date]: [section name] - [what changed and why]
    Source: [file and line that contradicts the original entry]

---

# Schema: ANALYSIS.md

Path: docs/.LORE/work/ANALYSIS.md
Created by: stage_2_analysis.md
Updated by: stage_2b_continuation.md (if overflow pass is needed)
Read by: stage_4_lore.md

## Fields

completion_status: string
    One of: COMPLETE | PARTIAL
    If PARTIAL, a list of incomplete sections must follow this field,
    each tagged [INCOMPLETE - reason].

## Sections

### Theory of Operation
A narrative description of what the application does and how it works end to end.
Written for a technically literate reader who has not seen the codebase.
Length: 200-500 words.
Every factual claim tagged [src: file] or [inferred].
Inferred claims must be plausible given what was read, not fabricated.

### Subsystems
One entry per identified subsystem. Format:
    #### [Subsystem Name]
    purpose: [one paragraph]
    files_read: [N of M] (N = files read this pass, M = total files in subsystem)
    file_inventory:
        [For each file read:]
        - [filepath]: [one-line description] [src: filepath]
          key functions: [name] - [one-line description] [src: filepath]
        [For each file given HEAD only:]
        - [filepath]: [data file | generated file] - HEAD only, not analyzed
        [No entry for unread files. A file not listed here was not read.]
    key functions or exports: [consolidated list across all read files]
    external dependencies: [list or "none"]
    integration points: [what other subsystems it talks to]
    PRIMER warranted: [YES score N/5 | NO score N/5]
    analysis_complete: [YES | NO]
        YES - files_read N equals M; all files in subsystem have been read.
              A PRIMER may be produced for this subsystem.
        NO  - files_read N is less than M; subsystem is not fully analyzed.
              A PRIMER must NOT be produced until analysis_complete is YES.
        Note: if the directory listing count for a subsystem is uncertain
        (dynamic directory, generated files mixed with source files), the
        agent must resolve the ambiguity before setting YES. Default to NO
        until the exact file count is confirmed from the directory listing
        in SKELETON.md.
    disposition: [active | deprecated | ignored]
        active     - normal treatment; full analysis and documentation.
        deprecated - the subsystem still exists in the codebase but is
                     being replaced or removed. Include a stub entry in
                     ENCHIRIDION and OVERVIEW; no PRIMER produced.
                     The stub must name what replaced it and why it still
                     exists. Example: "google/ legacy deploy scripts -
                     deprecated, superseded by deploy/; retained pending
                     feature parity."
        ignored    - foreign code, sample projects, or vendored code that
                     should not be documented as part of this project.
                     Excluded from all output documents. Noted in JOURNAL.md
                     under a "Ignored Subsystems" section only.
        Default: active. Set by the human at stage 3 via Q13-style answers,
        or inferred from README/comments if unambiguous.
    operator_facing: [YES | NO | MAYBE]
        YES   - the subsystem's primary purpose is to be run by an operator
                or developer directly. The whole directory is a tool
                collection, deployment system, or admin executable set.
                A GUIDE will be produced. Examples: srvr/tools/, deploy/
        NO    - internal only; not directly invoked by humans.
        MAYBE - unclear from files read; will become a Tier 3 question.
    dev_entry_points: [list of scripts/commands a developer invokes directly, or "none"]
        Admin executables, initializers, connectivity tests, or one-off
        migration runners that are not the subsystem's main function but
        may be invoked directly by a developer. A GUIDE section is produced
        for these even if operator_facing is NO.

### Data Paradigm
Describes how data is structured, stored, and moved through the system.
Covers: database type and schema approach, data models, data flow between layers.
Every claim tagged [src: file] or [inferred].

### IT Paradigm
Describes the infrastructure and deployment model.
Covers: runtime environment, containerization, cloud services, process management,
networking, and how the application is served.
Every claim tagged [src: file] or [inferred].

### Code Style and Organization
Describes the conventions observed in the codebase.
Covers: naming conventions, module patterns, async patterns, error handling style,
test conventions, and any framework-specific patterns.
Every claim tagged [src: file] or [inferred].

### Key Files and Functions Directory
A curated reference list. Not exhaustive - only entries important enough
to appear in documentation. Format:
    - [filepath]: [one-line description]
        key functions: [function name] - [one-line description]

### Observed Bugs and Risks
A numbered list of problems observed in the code.
Each entry: severity [HIGH | MEDIUM | LOW], description, location [src: file].
If none found, write: none observed.

### Incomplete Sections
Only present if completion_status is PARTIAL.
A list of sections that could not be completed and why.
Format: - [section name]: [reason incomplete] - [files not read]

---

# Schema: MANIFEST.md

Path: docs/.LORE/work/MANIFEST.md
Created by: stage_2_analysis.md (initial creation)
Updated by: stage_2b_continuation.md, stage_6_update.md
Read by: stage_6_update.md (mtime comparison to detect changed files)

## Fields

manifest_produced: string
    Timestamp of when this manifest was last written.
    Format: YYYY-MM-DD HH:MM

mtime_available: string
    Whether filesystem mtime values could be obtained during this run.
    One of: YES | NO
    Set to YES if bash ls -la (or equivalent) was available and produced
    real timestamps. Set to NO if all mtimes were recorded as "unknown".
    Stage 6 reads this field first. If NO, stage 6 immediately falls back
    to git-diff mode for change detection (with user confirmation) rather
    than attempting mtime comparison.

git_commit: string
    The git commit hash recorded at the time this manifest was last written.
    Format: full 40-character SHA-1 hex string, or "none" if git was not
    available or the project has no commits.
    Stage 6 reads this field to run git diff [git_commit] HEAD when detecting
    committed changes since the last analysis run.

sha256_available: string
    Whether sha256 checksums were recorded for files during this analysis run.
    One of: YES | NO
    Set to YES if sha256sum was available and checksums were computed and stored
    in the Files Read section. Set to NO if sha256sum was unavailable or the
    user declined the permission request.
    Stage 6 reads this field to determine whether checksum confirmation is
    possible for the current manifest.

file_read_policy_applied: string
    Records which file read policy was used for this analysis pass.
    One of: STANDARD | HEAD_APPLIED
    STANDARD   - all files were read in full (no HEAD-only reads occurred)
    HEAD_APPLIED - at least one file was read as HEAD only (first 20 lines)
    This field is informational; it helps QA verify that data files were
    not accidentally read in full and that source files were not skipped.

## Sections

### Files Read
A list of every file read during analysis, in the order they were read.
One entry per file. Updated after each analysis pass - entries for files
re-read in a later pass overwrite the prior entry for that file.

Format for each entry:
    - [filepath]: mtime [YYYY-MM-DD HH:MM] | sha256 [64-char hex or "none"] | subsystem [name or "none"] | stage [2 | 2b | update]

    filepath:   path relative to the project root
    mtime:      the file's last-modified time at the moment it was read,
                obtained via bash ls -la or equivalent if available,
                or "unknown" if mtime could not be read
    sha256:     the sha256 checksum of the file at the time it was read,
                computed via sha256sum if available and permitted,
                or "none" if sha256sum was unavailable or user declined
    subsystem:  the subsystem this file was assigned to in ANALYSIS.md,
                or "none" if it does not belong to a known subsystem
    stage:      which stage read this file

Example:
    - app/srvr/index.js: mtime 2024-11-01 14:23 | sha256 a3f1c2... | subsystem auth | stage 2
    - app/config/default.js: mtime 2024-10-30 09:11 | sha256 none | subsystem none | stage 2
    - app/srvr/payments.js: mtime 2024-11-03 16:45 | sha256 9b0e44... | subsystem payments | stage 2b

### Manual Overrides
A list of corrections made via MENU D option [C] that were promoted to
ANALYSIS.md. These entries warn a future update pass that ANALYSIS.md
has been hand-patched and the relevant file should be re-read carefully.

Format for each entry:
    - [filepath]: override on [date] - [one-line description of what changed]

If no overrides have been made, write: none

### Preserved Files
A list of all non-LORE files in docs/ at the time of the last stage 4 or
stage 6 run. Used by stage 6 to detect user files that were renamed or
deleted between runs, so the preserve-list stays accurate.

Written by: stage_4_lore.md (initial creation), stage_6_update.md (updates)
Read by: stage_6_update.md

Format for each entry:
    - [filepath]: [approximate line count or "empty"]

If no user files were present in docs/, write: none

---

# Schema: QUESTIONS.md

Path: docs/.LORE/work/QUESTIONS.md
Created by: stage_1_recon.md (Tier 1 questions from SKELETON.md)
            stage_2_analysis.md (all tiers, consolidated)
Read by: stage_3_human_gate.md
Updated by: stage_3_human_gate.md (human fills in answers inline)

## Structure

The file is a numbered list of questions grouped by tier.
The human answers each question by filling in the ANSWER field.
Do not remove or reorder questions. Do not change question text.
Add answers only. Leave ANSWER blank if unknown.

Format for each question:
    ---
    Q[number] [TIER-1 | TIER-2 | TIER-3]
    Question: [question text]
    Context: [why this matters to the documentation]
    Default assumption if unanswered: [what stage 4 will use if blank - Tier 2 only]
    ANSWER: [human fills this in]
    CONFIDENCE: [blank if unanswered | ??? if uncertain]
    ---

The CONFIDENCE field is filled by the human alongside the ANSWER field.
    - If ANSWER is filled and CONFIDENCE is blank: treated as authoritative (high confidence).
    - If ANSWER is filled and CONFIDENCE is ???: treated as low confidence (uncertain).
    - If ANSWER is blank: CONFIDENCE is ignored.

Stage 4 applies hedging language ("reportedly", "as described by the operator",
"not confirmed in code") for answers marked ???.
Stage 5 cross-references ??? answers against code observations before accepting
them as authoritative.

Internally, the pipeline maps confidence as follows:
    blank (answered) = high confidence (equivalent to former scale value 3)
    ???              = low confidence  (equivalent to former scale value 1)
This mapping is used only within stage logic and is not exposed to the human.

---

# Schema: REVIEW_NOTES.md

Path: docs/.LORE/work/REVIEW_NOTES.md
Created by: stage_5_qa.md
Read by: stage_5_qa.md (subsequent loop iterations), human reviewer

## Fields

qa_pass_number: integer
    Which iteration of the QA loop produced this file. Starts at 1.

overall_score: string
    Format: [N] of [total] checks passed
    Example: 11 of 14 checks passed

exit_condition_met: boolean string
    YES or NO
    YES means all checks passed or max iterations reached.

## Sections

### Checks Performed
A table of every rubric check run this pass. Format:
    | Check ID | Description | Result | Confidence | Notes |
    |----------|-------------|--------|------------|-------|
    Result is: PASS | FAIL | PARTIAL
    Confidence is: 0 | 1 | 2 | 3 | 4
        0 = no confidence (0%)
        1 = low confidence (~25%)
        2 = moderate confidence (~50%)
        3 = high confidence (~75%)
        4 = certain (100%)

### Defects Found
A numbered list of defects. Present even if empty (write "none").
Format:
    [N]. [Check ID] - [description of defect]
         Location: [file and section]
         Severity: [BLOCKING | MAJOR | MINOR]
         Correction: [what must change]

### Corrections Applied
A list of corrections made to output documents automatically during this pass.
Only corrections that were successfully applied belong here.
Format: - [document]: [what was changed] ([check ID])
If no corrections were made: none

### Corrections Deferred
A list of defects that could not be auto-corrected this pass, either because
the fix required information not available in work files, or because the
defect was MINOR and not worth auto-correcting.
Format: - [check ID] - [description] - Reason deferred: [why not corrected]
If no corrections were deferred: none

### Next Pass Instructions
Only present if exit_condition_met is NO.
A numbered list of specific things the next pass must address.
Format: [N]. [what to fix] in [document] - [how to fix it]

---

# MIGRATION LOG

This section records all schema changes between versions.
When upgrading SCHEMAS.md, append a new entry here before changing anything else.
Each entry describes how to migrate existing work files from the old version to the new one.

Format:
    ## Migration: [old version] -> [new version]
    Date: [date]
    Changed files: [list of work files affected]
    Instructions:
        [file]: [DROP field X | ADD field Y with default Z | RENAME A to B | CONVERT format]

## Migration: 1.6 -> 1.7
    Date: see docs/.LORE/work/JOURNAL.md started field for run date
    Changed files: MANIFEST.md
    Instructions:
        MANIFEST.md: ADD field git_commit after mtime_available field.
            Set to the output of: git rev-parse HEAD
            If git is not available or the project has no commits, set to "none".
        MANIFEST.md: ADD field sha256_available after git_commit field.
            Set to NO (checksums were not recorded in prior runs).
        MANIFEST.md: UPDATE schema_version field value to 1.7.
        MANIFEST.md: For each entry in Files Read section, ADD sha256 field
            with value "none" (checksums were not available at prior analysis time).
            New format: mtime [...] | sha256 none | subsystem [...] | stage [...]

## Migration: 1.5 -> 1.6
    Date: see docs/.LORE/work/JOURNAL.md started field for run date
    Changed files: JOURNAL.md, QUESTIONS.md, docs/ output files, SKELETON.md
    Instructions:
        JOURNAL.md: UPDATE schema_version field value to 1.6.
        JOURNAL.md: In the Artifacts section, RENAME docs/LORE_OVERVIEW.md to
            docs/LORE_ABSTRACT.md if it appears. ADD docs/LORE_BRIEF.md: MISSING
            to the Artifacts section if not already present.
        QUESTIONS.md: CONVERT CONFIDENCE fields. For each answered question:
            If CONFIDENCE is 3 or 4: set CONFIDENCE to blank (now means authoritative).
            If CONFIDENCE is 0, 1, or 2: set CONFIDENCE to ???.
            If CONFIDENCE is already blank on an answered question: leave blank.
            For unanswered questions: leave CONFIDENCE blank (no change).
        docs/: RENAME docs/LORE_OVERVIEW.md to docs/LORE_ABSTRACT.md if it exists.
            Update any references to LORE_OVERVIEW.md inside docs/LORE_INDEX.md,
            docs/LORE_ENCHIRIDION.md, and any PRIMER files to LORE_ABSTRACT.md.
        SKELETON.md: ADD git_available field to Project Identity section.
            Set to YES if git is present in the project root (.git/ directory exists),
            NO otherwise. If uncertain, set to NO.
        docs/LORE_BRIEF.md: MISSING - this document did not exist in schema 1.5.
            Run stage_7_brief.md to generate it, or choose option [B] from MENU D.

## Migration: (none - initial version 1.0)
    This is the first version. No migration required.

## Migration: 1.0 -> 1.1
    Date: see docs/.LORE/work/JOURNAL.md started field for run date
    Changed files: JOURNAL.md, new file MANIFEST.md
    Instructions:
        JOURNAL.md: no field changes; schema_version field value changes to 1.1
        MANIFEST.md: CREATE new file using schema above
            Files Read section: populate from docs/.LORE/work/ANALYSIS.md [src:] tags
                if available, with mtime "unknown" and stage "2" for all entries
            Manual Overrides section: write "none"
            manifest_produced: use current timestamp

## Migration: 1.2 -> 1.3
    Date: see docs/.LORE/work/JOURNAL.md started field for run date
    Changed files: ANALYSIS.md, MANIFEST.md (filename references)
    Instructions:
        ANALYSIS.md: ADD field user_runnable after PRIMER warranted in each
            subsystem entry. Default to NO for internal-only subsystems,
            YES for any subsystem with operator-invocable scripts or entry points.
            If unclear, use MAYBE.
        ANALYSIS.md: UPDATE all PRIMER warranted format from [YES score N/4]
            to [YES score N/5] if not already done by 1.1->1.2 migration.
        MANIFEST.md: UPDATE schema_version field value to 1.3.
        docs/: RENAME any existing LORE_[name]_PRIMER.md files to
            LORE_PRIMER_[name].md. Update LORE_INDEX.md entries to match.
            Update JOURNAL.md Artifacts section to match new filenames.

## Migration: 1.4 -> 1.5
    Date: see docs/.LORE/work/JOURNAL.md started field for run date
    Changed files: JOURNAL.md, ANALYSIS.md, REVIEW_NOTES.md, SCHEMAS.md
    Instructions:
        JOURNAL.md: ADD field files_read_total after last_error field.
            Derive value by counting entries in MANIFEST.md Files Read section
            with stage "2", "2b", or "2c" and policy FULL. If MANIFEST.md is
            unavailable, write "unknown".
        ANALYSIS.md: ADD disposition field to each subsystem entry after
            analysis_complete field. Default: "active" for all subsystems
            unless a prior Q13-style answer indicated deprecated or ignored.
            For subsystems previously excluded by human instruction: set
            "ignored". For subsystems noted as legacy/superseded: set
            "deprecated".
        REVIEW_NOTES.md: ADD "Corrections Deferred" section after
            "Corrections Applied" section. Populate by reviewing the
            Defects Found list: any defect marked "No auto-correction
            possible" or similar belongs in Corrections Deferred, not
            Corrections Applied. If no deferred corrections: write "none".
        SCHEMAS.md: UPDATE schema_version field value to 1.5.

## Migration: 1.3 -> 1.4
    Date: see docs/.LORE/work/JOURNAL.md started field for run date
    Changed files: ANALYSIS.md, SKELETON.md, MANIFEST.md
    Instructions:
        SKELETON.md: ADD Resource Estimate section after Files Flagged for
            Deep Read. Populate by reviewing the subsystem candidates and
            file counts already present. Use "unknown" for token estimates
            if they cannot be derived. Add read policy (FULL | HEAD | SKIP)
            annotation to each entry in Files Flagged for Deep Read.
        ANALYSIS.md: REPLACE key files field in each subsystem entry with
            files_read (N of M format) and file_inventory block. Derive N
            from the number of [src:] tags in the existing entry. Derive M
            from the subsystem candidate file count in SKELETON.md.
            ADD analysis_complete field: YES if N equals M, NO otherwise.
        ANALYSIS.md: REPLACE user_runnable field with operator_facing and
            dev_entry_points fields. Map: user_runnable YES where the whole
            directory is a tool collection -> operator_facing YES.
            user_runnable YES where only specific scripts are invocable ->
            operator_facing NO, dev_entry_points [list those scripts].
            user_runnable NO -> operator_facing NO, dev_entry_points none.
        MANIFEST.md: ADD file_read_policy_applied field. Set to
            HEAD_APPLIED if any file in Files Read has mtime "unknown" and
            was a data file; otherwise STANDARD.
        MANIFEST.md: UPDATE schema_version field value to 1.4.

## Migration: 1.1 -> 1.2
    Date: see docs/.LORE/work/JOURNAL.md started field for run date
    Changed files: MANIFEST.md, QUESTIONS.md, SKELETON.md
    Instructions:
        MANIFEST.md: ADD field mtime_available with default "NO" if all existing
            mtime entries are "unknown", otherwise "YES"
        MANIFEST.md: ADD section "Preserved Files" after Manual Overrides section.
            Populate by listing all non-LORE files currently in docs/ with
            approximate line counts. If docs/ has no non-LORE files, write "none".
        QUESTIONS.md: ADD field CONFIDENCE after each ANSWER field.
            For questions with a blank ANSWER, leave CONFIDENCE blank.
            For questions with a filled ANSWER, set CONFIDENCE to 3 (high)
            as the default, unless the answer text contains uncertainty language
            ("may", "not sure", "unclear", "unconfirmed", "possibly") in which
            case set CONFIDENCE to 1 (low).
        SKELETON.md: ADD section "Domain Jargon Candidates" after the
            Existing Documentation Inventory section and before Subsystem
            Candidates. Populate by scanning directory names, file names,
            and any README content already in the skeleton for abbreviations
            or domain terms. If none are identifiable, write "none identified".
        SKELETON.md: UPDATE all subsystem score formats from [N/4] to [N/5].
            Scores do not change - only the denominator in the label changes.