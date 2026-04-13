# Stage 1 - Reconnaissance
# Read this file and execute it exactly. Do not skip steps.
# Do not begin writing output until Step 4.
# Pipeline reference: lore/LORE.md


---

# Purpose

This stage reads the project structure without reading implementation files.
It produces docs/.LORE/work/SKELETON.md and initializes docs/.LORE/work/JOURNAL.md.

The goal is a compressed structural map of the project that the next context
window can load instead of re-reading the codebase.

Time and token discipline matter here. Read broadly and shallowly.
Do not get pulled into implementation details.

---

# Prerequisites

Before executing this stage:

1. Read lore/SCHEMAS.md in full. You must know the exact schema for
   JOURNAL.md, SKELETON.md, and QUESTIONS.md before writing anything.

2. Confirm the project root. It was established in the lore/LORE.md menu.
   If it was not established (you arrived here from a fresh context),
   assume the project root is the parent directory of lore/.

---

# Step 1 - Timestamp

Use the current date and time from your context window (most AI assistants
have access to the current datetime). Record it as YYYY-MM-DD HH:MM.

If no current datetime is available, use the string "unknown".
Store this timestamp for use when writing JOURNAL.md.

---

# Step 2 - Read the Project Root

Read the directory listing of the project root (one level only).
Do not recurse yet. Record all top-level files and folders.

For each top-level item, make an initial annotation:
    - files: note the extension and any recognizable name pattern
    - folders: make an inference about purpose based on name alone

Tag all annotations [inferred] at this stage.

---

# Step 3 - Identify the Project Manifest

Look for a project manifest file. Check for these in order, stopping at
the first match:

    package.json        (Node.js)
    Cargo.toml          (Rust)
    go.mod              (Go)
    requirements.txt    (Python - basic)
    pyproject.toml      (Python - modern)
    composer.json       (PHP)
    build.gradle        (JVM)
    pom.xml             (Maven/JVM)
    *.csproj            (.NET)

Read the first manifest found. Extract:
    - project name
    - version (if present)
    - declared dependencies (names and versions)
    - scripts or build targets (if present)
    - any description field

If no manifest is found, note this. Derive the project name from the root
directory name and flag the absence in the journal.

---

# Step 4 - Read Existing Documentation

Check for these files and read any that exist:

    [project root]/README.md
    [project root]/docs/README.md
    [project root]/docs/*.md  (directory listing only, do not read contents yet)
    [project root]/CHANGELOG.md
    [project root]/.env.example or .env.sample
    [project root]/docker-compose.yml or docker-compose.yaml
    [project root]/Dockerfile  (first 40 lines only)

For each doc file read in full: note key claims about the project's purpose,
architecture, and operation. Tag all notes [src: filename].

For the Dockerfile (if present): note the base image, exposed ports, and
any CMD or ENTRYPOINT instructions only.

Do NOT read all docs files in full at this stage. The directory listing
of docs/ is sufficient for now.

## Existing Documentation Inventory

After reading, record:

    a) Whether a docs/ directory exists at all.
    b) A list of every file found in docs/ (names only at this stage).
    c) For each file in docs/ that appears to be LORE canonical output
       (LORE_ABSTRACT.md, LORE_ENCHIRIDION.md, LORE_INDEX.md, LORE_BRIEF.md, LORE_*_PRIMER.md):
           - Note it as a prior LORE artifact.
           - Note the file's approximate size (rough line count or "empty").
           - Flag it: stage 4 will overwrite this file.
    d) For each file in docs/ that does NOT match a canonical LORE output name:
           - Note it as a user-created file.
           - Note its name and approximate size.
           - Flag it: stage 4 must preserve this file and not overwrite it.
    e) If any existing docs file was read in full (README.md, etc.):
           - Extract any claims about architecture, deployment, or data that
             could contradict or supplement what stage 2 discovers in code.
           - Record these claims tagged [src: filename].
           - Record them in SKELETON.md Existing Documentation Claims section
             (see Step 11). Stage 2 will verify them against actual code.

If docs/ does not exist, note: "no docs/ directory found [inferred]".
This is not an error - stage 4 will create it.

---

# Step 4b - Git Reconnaissance (optional)

This step collects authorship and change history from git. It is optional.
If git is unavailable or the user declines, set git_available to NO and skip
all sub-steps. The pipeline continues normally without this data.

Check whether a .git/ directory exists in the project root.

    If .git/ is NOT present:
        Set git_available: NO
        Record in SKELETON.md Project Identity: git_available: NO
        Skip the rest of this step.

    If .git/ IS present:
        Ask the user:
            "Git repository detected. I can read git history to add authorship
             information to the documentation. This requires running read-only
             git commands (git shortlog, git log). OK? [yes/no]"

        If the user says no:
            Set git_available: NO
            Record in SKELETON.md Project Identity: git_available: NO
            Skip the rest of this step.

        If the user says yes:
            Set git_available: YES
            Record in SKELETON.md Project Identity: git_available: YES

            Run the following read-only git commands and record results
            in a "Git Metadata" section of SKELETON.md:

            a) Overall contributors:
                   git shortlog -sn --no-merges HEAD
               Record: each author name and commit count.
               If the output is empty or the command fails, note "unavailable".

            b) Most recent commit date per top-level directory:
                   git log -1 --format="%ad" --date=short -- [directory]/
               Run this for each top-level source directory identified in Step 2.
               Record: directory name and last-commit date.
               If a directory has no commits, note "no commits found".

            c) Primary author per top-level directory (most commits):
                   git shortlog -sn --no-merges HEAD -- [directory]/
               Run this for each top-level source directory.
               Record: the top author name (first line of output) for each directory.
               If output is empty, note "unknown".

            Tag all git-derived data [src: git].
            Do not run any git command that modifies the repository
            (no checkout, reset, add, commit, push, pull).

---

# Step 5 - Identify Entry Points

Look for application entry point files. Check for these patterns in order:

    [project root]/index.js, index.ts, index.mjs
    [project root]/app.js, app.ts, app.mjs
    [project root]/server.js, server.ts, server.mjs
    [project root]/main.js, main.ts, main.mjs, main.go, main.rs, main.py
    [project root]/src/index.*, src/main.*, src/app.*
    [project root]/cmd/main.go  (Go convention)
    Any file named in the manifest's "main" or "scripts.start" field

For each entry point found: read the first 60 lines only.
Note: imports, framework initialization, server setup, route registration,
and any top-level configuration loading.
Tag all notes [src: filename:lines].

---

# Step 6 - Recurse One Level into Key Directories

For each top-level directory that is NOT one of these, read its contents
(one level only, no file reading):

    node_modules/
    .git/
    .vscode/
    .idea/
    dist/
    build/
    target/
    __pycache__/
    .cache/
    vendor/   (exception: read vendor/ listing if no manifest was found)
    lore/    (you are already here)

For each directory you do recurse into:
    - List its contents
    - Annotate each subdirectory with inferred purpose [inferred]
    - Note any files with recognizable names (config files, entry points, etc.)

Do not read file contents during this step. List only.

---

# Step 7 - Identify Configuration Files

Scan the project root and one level of subdirectories for configuration files.
Configuration files include:

    *.config.js, *.config.ts, *.config.mjs
    *.json (excluding package-lock.json, yarn.lock, node_modules)
    *.yaml, *.yml
    *.toml, *.ini, *.env.example
    .env.example, .env.sample, .env.defaults
    nginx.conf or any nginx config
    pm2.config.*, ecosystem.config.*

For each config file found: note the filename and infer its purpose.
Do NOT read the contents of config files at this stage unless one of these
conditions is met:
    a) The file is 10 lines or fewer.
    b) The file is manifest-adjacent: it defines the project's data model,
       schema, feature flags, billing SKUs, or global constants. Examples:
       globals.js, dbConfig.js, schema.js, constants.js, features.js.
       These files are structurally critical and reading them in stage 1
       materially improves subsystem scoring. Read them in full.

When reading any config file that may contain secrets (passwords, API keys,
tokens, private keys): read the file for structure and key names only.
Do not record the values of any field whose name suggests a credential
(patterns: password, secret, key, token, apiKey, privateKey, credentials,
auth, pass, pwd, connectionString, dsn). Record the key name and type only.
Tag all inferences [inferred].

---

# File Read Policy

Every file in the project falls into one of three read policy categories.
Assign a policy to each file when flagging it for deep read in Step 10,
and record the policy in the Files Flagged for Deep Read section of SKELETON.md.

    FULL  - Read the entire file.
            Applies to: all source code files (.js, .mjs, .ts, .py, .go,
            .sh, .bash), config files, Dockerfiles, markdown docs within
            a subsystem, YAML/JSON config files under approximately 100 lines.

    HEAD  - Read the first 20 lines only.
            Applies to: data files, large lookup tables, large JSON arrays,
            CSV files, any JSON or YAML file that is clearly a data dump
            rather than configuration (e.g. carriers.json, a fixtures file,
            a compiled translation file). The 20-line HEAD is enough to
            confirm the file's purpose and record it in the inventory.
            HEAD files appear in the file inventory as:
                [data file - HEAD only: [what the first line revealed]]
            HEAD files are NEVER documented with function details.

    SKIP  - Do not read.
            Applies to: package-lock.json, yarn.lock, any lock file,
            compiled output, minified JS, binary files, image files,
            anything in node_modules/, .git/, dist/, build/, target/.
            SKIP files do not appear in any file inventory.

No documentation is ever produced for a file that has not been fully read.
A file listed in a PRIMER's file directory must have been read with policy FULL.
A HEAD file may be acknowledged in the file inventory but receives no
function documentation.

---

# Step 8 - Assess Subsystem Candidates

Based on the directory structure and file patterns observed so far,
identify potential subsystems. Score each candidate using this rubric:

    +1 if 5 or more files are dedicated to it
    +1 if it appears to be referenced by multiple other parts of the project
        (infer from directory names and file names alone at this stage)
    +1 if it has a non-obvious pattern (not just CRUD, not just config)
    +1 if it has its own config files or external service dependencies
    +1 if it is a standalone operational tool with its own configuration
        and more than 5 non-trivial files, even if no other part of the
        application imports it (examples: deploy/, tools/, scripts/, cli/)

A score of 2 or higher means a PRIMER is likely warranted.
Record your assessment for each candidate. Be conservative - it is better
to under-score here and let stage 2 upgrade scores than to over-commit.

Note: standalone operator tools (deploy systems, CLI toolkits, admin scripts)
will naturally score 0 on the "referenced by multiple other parts" criterion
because they run independently. The fifth criterion exists specifically to
prevent these from being under-scored.

---

# Step 9 - Generate Human Questions

Review everything observed. Identify questions that:
    a) cannot be answered by reading more code
    b) would materially affect the accuracy of the documentation

Classify each question:

    TIER 1 (blocking): documentation cannot be accurate without this answer
        Examples: production hosting environment, intended user base,
        whether there is a staging environment, SLA or uptime expectations

    TIER 2 (enhancing): documentation would be more accurate with this answer,
        but a reasonable default assumption can be made
        Examples: planned future features, known technical debt decisions,
        rationale for architectural choices

    TIER 3 (cosmetic): would add polish, no accuracy impact
        Examples: preferred terminology, project history, team preferences

Apply this discipline: if you are tempted to ask more than 5 Tier 1 questions,
force yourself to reconsider. Tier 1 questions must be truly blocking.
If in doubt, move a question to Tier 2.

---

# Step 9b - Identify Domain Jargon Candidates

Before flagging files for deep read, scan everything observed so far
(directory names, file names, manifest description, README, entry point
summaries, config key names) for terms that appear to be project-specific
or domain-specific jargon.

A jargon candidate is any term that a developer from outside this project's
industry would not recognize. The test is: "would a competent backend
developer, hired today, be confused by this term when reading the code?"

A jargon candidate IS:
    - an industry abbreviation or acronym (DIP, LRN, NPA-NXX, NANP, OCN)
    - a domain term specific to the project's vertical (carrier, ported,
      deactivation record, litigator, test-drive in telecom context)
    - a project-internal name for a non-obvious concept (grocer, deac)

A jargon candidate is NOT:
    - an internal module or library name (zero, fmt, biz) - these belong
      in the PRIMER file directory, not the jargon list
    - a common programming term used normally (handler, router, schema)
    - an acronym that is universally understood (API, HTTP, JSON, DB)

Record each candidate in a section of SKELETON.md titled "Domain Jargon
Candidates". For each entry, record:
    - the term as it appears in the codebase
    - where you saw it (filename or directory name)
    - your best inference of its meaning, tagged [inferred]

Do not fabricate definitions. If you cannot infer a meaning, write
"meaning unclear [inferred]".

This list is consumed by stage 5 QA check C7.

---

# Step 10 - Flag Files for Deep Read

Produce a prioritized list of files for stage_2_analysis.md to read.
This list covers the first context window of analysis only. Stage 2b and
stage 2c will continue reading until all PRIMER-warranted subsystems are
fully analyzed.

Order by priority:
    1. Files belonging to the highest-PRIMER-score subsystems first.
       Within a subsystem, list all its files before moving to the next.
       This ensures subsystems are analyzed completely in PRIMER-score order.
    2. Cross-cutting files (config, schema, entry points) that inform
       multiple subsystems.
    3. Files belonging to lower-priority subsystems.

A file warrants deep reading if it is:
    - an entry point or primary application file
    - a configuration file that reveals infrastructure or data paradigm
    - a file whose name appears frequently in other files (infer from listing)
    - a file explicitly referenced in existing documentation
    - a schema, model, or type definition file
    - a routing or dispatch file
    - a handler file for an authentication subsystem (if one is identified)
    - a handler file for a billing or payment subsystem (if one is identified)

For PRIMER-warranted subsystems (score 2+): every file in the subsystem must
eventually be read. The goal for stage 2's first pass is not to maximize
files read - it is to read the highest-priority files and produce a useful
partial ANALYSIS.md that stage 2b can extend. Flag enough files to fill
approximately one context window of reading. The remainder are picked up
by stage 2b and stage 2c automatically.

For standalone operator tool directories (operator_facing subsystems):
flag at least 3 representative files even if the directory contains many more.
These give stage 2 enough context to produce the subsystem entry and GUIDE
scaffolding. All remaining files are read in stage 2c.

For pipeline subsystems with sequential stages (downloader/conditioner/
digester, clean/verify, etc.): flag all stage files, not just the first.

Assign a read policy to each flagged file (FULL | HEAD | SKIP) using the
rules defined in the File Read Policy section above.

Size this list to fit approximately one context window of analysis reading:
    - Default guidance: 15-20 FULL files
    - Adjust down for large files (200+ lines average): use 10-15
    - Adjust up for small files (under 50 lines average): use 20-25
    - Hard maximum: 30 files regardless of file size

If more files exist than fit in this list, the remainder go into stage 2b
and stage 2c automatically.

For each file, write one sentence explaining why it was flagged, and record
the assigned read policy.

---

# Step 10b - Produce Resource Estimate

After completing the flagged file list and subsystem candidates, produce a
resource estimate for the full documentation run.

Count the following:
    - total_source_files: count all non-SKIP files across all identified
      subsystems. For each subsystem, use the directory listing to count
      files with source code extensions. Exclude lock files, data files,
      and generated output from this count.
    - files_flagged_stage2: count of entries in Step 10's flagged list.
    - files_remaining: total_source_files minus files_flagged_stage2.

Estimate context windows:
    - Assume approximately 15-20 FULL file reads per context window.
    - Divide total_source_files by 17 (midpoint) to get estimated windows
      for analysis alone.
    - Add 1 window for stage 3 (human gate).
    - Add 2-3 windows for stage 4 (documentation production; more if
      many PRIMERs and GUIDEs are warranted).
    - Add 1 window for stage 5 QA.
    - Sum these for estimated_context_windows range (low = optimistic,
      high = pessimistic with 2b/2c overflow passes).

Estimate tokens:
    - Assume approximately 30k-50k tokens per context window of analysis.
    - Multiply estimated_context_windows by this range.
    - Express as [low]k-[high]k.

Note on PRIMER count: the estimate records a preliminary PRIMER-warranted
subsystem count from stage 1 scoring. Stage 2 regularly revises scores
upward after reading actual files. The estimate should note this explicitly:
"at least N subsystems warrant PRIMERs; expect stage 2 to revise upward."

Record this estimate in the Resource Estimate section of SKELETON.md.
Output it prominently in the handoff note so the human sees it before
committing to the next stage.

---

# Step 11 - Write Output Files


DO NOT begin writing until all previous steps are complete.

Write the following files. Match the schemas in lore/SCHEMAS.md exactly.

## Write docs/.LORE/work/SKELETON.md

Sections to include (in this order):
    1. Project Identity (include git_available field)
    2. Dependency Summary
    3. Directory Map
    4. Entry Point Summaries
    5. Files Flagged for Deep Read (with read policy per file)
    6. Resource Estimate
    7. Subsystem Candidates
    8. Existing Documentation Inventory
    9. Domain Jargon Candidates
    10. Git Metadata (if git_available is YES; omit section entirely if NO)
    11. Human Questions (Tier 1)
    12. Human Questions (Tier 2)
    13. Human Questions (Tier 3)

### Existing Documentation Inventory section format

    ### Existing Documentation Inventory

    docs/ directory: [EXISTS | NOT FOUND]

    Prior LORE artifacts (will be overwritten by stage 4):
    [For each canonical LORE file found:]
        - [filepath]: [approximate line count or "empty"]
    [If none:]
        none found

    User-created files (will be preserved by stage 4):
    [For each non-canonical file found:]
        - [filepath]: [approximate line count or "empty"]
    [If none:]
        none found

    Claims extracted from existing documentation:
    [For each claim noted in Step 4e:]
        - [claim text] [src: filename]
    [If none:]
        none extracted - no readable docs found

    Reconciliation note for stage 2:
    [If claims were extracted:]
        Stage 2 must verify these claims against actual code and flag any
        contradictions in the SKELETON.md Corrections section.
    [If no claims:]
        No existing documentation claims to reconcile.

Every factual claim must be tagged [src: filename] or [inferred].
Do not include claims you cannot support with either a source or a
reasonable inference from structure alone.

## Write docs/.LORE/work/QUESTIONS.md

Include all questions from all tiers. Use the QUESTIONS.md schema from
lore/SCHEMAS.md. Leave all ANSWER fields blank.

## Write docs/.LORE/work/JOURNAL.md

Use the JOURNAL.md schema from lore/SCHEMAS.md. Fill in:
    project_name: from manifest or directory name
    project_root: the path established in prerequisites
    schema_version: from lore/SCHEMAS.md
    started: the timestamp from Step 1
    last_updated: same as started
    last_stage_completed: stage_1_recon.md
    status: IN_PROGRESS
    last_error: none
    files_read_total: 0

Completed Stages section: add one entry:
    - stage_1_recon.md: DONE

Artifacts section: list all three files just written (EXISTS) and all
expected final output docs (MISSING). The MISSING list must include:
    - docs/LORE_ABSTRACT.md: MISSING
    - docs/LORE_ENCHIRIDION.md: MISSING
    - docs/LORE_INDEX.md: MISSING
    - docs/LORE_BRIEF.md: MISSING
    (plus any PRIMER and GUIDE files identified as warranted)

Unanswered Questions section: list all Tier 1 questions by ID and summary.

Last Handoff Note section: leave blank for now (filled in Step 12).

---

# Step 12 - Validate Output

Before writing the handoff, verify:

    [ ] docs/.LORE/work/SKELETON.md exists and contains all required sections
    [ ] SKELETON.md Project Identity section contains git_available field
    [ ] docs/.LORE/work/QUESTIONS.md exists with at least one question (any tier)
    [ ] docs/.LORE/work/JOURNAL.md exists with status IN_PROGRESS
    [ ] Every claim in SKELETON.md is tagged [src:] or [inferred]
    [ ] Files Flagged for Deep Read contains between 1 and 20 entries
    [ ] No file in node_modules/, .git/, or dist/ was read or listed in detail

If any check fails: fix it before proceeding.
If a check cannot be fixed (e.g. no manifest exists), note the exception
in the journal under last_error, set status to IN_PROGRESS (not ERROR),
and continue. Only set status to ERROR if the output is fundamentally
unusable.

---

# Step 13 - Write the Handoff Note

Write the following block exactly, filling in the bracketed fields.
Then copy it into the Last Handoff Note section of JOURNAL.md.
Then output it as the final output of this stage.

---
Stage 1 complete - Reconnaissance
Codebase: [total_source_files] files ([comma-separated list of main languages/types])
LORE: Stage [1/5] Pass [1] Context [1]
Status: COMPLETE - proceed to Stage 2

Resource estimate for full run:
    Source files to read:      [total_source_files]
    PRIMER-warranted systems:  at least [count] ([names]) - expect stage 2 to revise upward
    Estimated context windows: [low]-[high] total
    Estimated tokens:          [low]k-[high]k total
    Tokens this context:       ~[N]k (estimated for stage 1)

Please start a new context with this text:

Please follow `lore/LORE.md` and CONTINUE.
---

---

# DO NOT

- Do not read any file in node_modules/, .git/, dist/, build/, or target/
- Do not read implementation files (only entry points, manifests, configs, docs)
- Do not produce any output in docs/ - that is stage 4's job
- Do not answer the human gate questions yourself
- Do not write the Enchiridion, Abstract, Brief, or any final documentation
- Do not assign read policy FULL to data files, lock files, or compiled output
- Do not assign read policy HEAD or SKIP to source code files
- Do not run git commands that modify the repository (no checkout, reset, add, commit)
- Do not proceed to stage 2 in this same context window
  (output the handoff note and stop)