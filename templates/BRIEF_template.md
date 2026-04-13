# LORE_BRIEF
# Template for docs/LORE_BRIEF.md
# Consumed by: stage_7_brief.md
# Replace every [PLACEHOLDER] with real content.
# Remove all lines that begin with # (template comments) before writing the final file.
# Do not leave any [PLACEHOLDER] strings in the output.
# The output file must begin with the ingestion prompt line below.
# Do not include any TEMPLATE INSTRUCTIONS block - this template has none.

---

READING INSTRUCTIONS: If you are an AI assistant reading this file, reply with
exactly these 4 words when you have finished reading it and completed the
reading sequence below: ALL DOCUMENTATION INGESTED

Do not summarize, narrate, or explain. Do not respond until you have finished
the full reading sequence. If you encounter an error or inconsistency in any
document, note it briefly and continue reading.

---

# Project Brief - [project_name]

generated_by: LORE stage_7_brief.md
schema_version: [schema_version]
last_updated: [last_updated]

---

# what this project is

[Write 2-4 sentences describing what the project is and what it does.
 Source: ANALYSIS.md Theory of Operation and SKELETON.md Project Identity.
 Keep it tight - this is orientation, not documentation.
 Tag every factual sentence [src: filename] or [inferred].]

Language:   [language from SKELETON.md]
Runtime:    [runtime from SKELETON.md]
Framework:  [framework from SKELETON.md, or "none detected"]

---

# collaboration guidelines

## base rules

These rules apply to all work on this project regardless of task.

### confirm before large changes

Stop and ask before:
    - Writing 1,000 or more lines of new code
    - Modifying a file with 2,000 or more lines of existing code
    - Writing a function with 200 or more lines
    - Modifying a function with 300 or more lines of existing code
    - Making 2,000 or more total lines of code changes in a single session

Suggested questions:
    "This task will create approximately [N] lines of new code. Should we
     break it into smaller pieces?"
    "This file has [N] lines. Should we refactor it before making changes?"
    "This function is getting large. Should we split it up first?"

### ask before assuming on design decisions

Stop and ask when you encounter:
    - A choice between two or more valid architectural approaches
    - A file or module that does not clearly belong in one place
    - A type, schema, or interface that does not exist yet and must be created
    - A pattern in the codebase that conflicts with your planned approach
    - Any change that affects more than one subsystem

Suggested questions:
    "I see two approaches here: [A] or [B]. Which fits better?"
    "I need to create a new type for this. Should I add it to [file] or
     create a new file?"
    "This change touches [subsystem X] and [subsystem Y]. Should I proceed
     or discuss scope first?"

### check for existing code before writing new code

Before writing a new function or module:
    - Search the codebase for similar functionality
    - If something similar exists, prefer extending it over duplicating it
    - If duplication seems necessary, ask first

### do not add global state without discussion

Prefer module-scoped constants, class instances, or passed parameters over
global mutable variables. If a global seems necessary, ask before adding it.

### use real types

Do not use placeholder or catch-all types (such as any in TypeScript,
interface{} in Go, or object in Python) unless the existing codebase
establishes that pattern. If the correct type is unclear, ask.

### communicate scope changes early

If you discover mid-task that the scope is larger than originally understood,
stop and report before continuing. Do not silently expand scope.

---

## project-specific rules

[These rules are derived from ANALYSIS.md Code Style and Organization.
 Each rule must be backed by a source tag. Only include rules for patterns
 actually observed in this codebase. If no project-specific rules were
 derivable, write:
     No project-specific rules were derivable from the analysis.
     Follow the base rules and the patterns you observe in existing code.]

[For each observed pattern, write one rule in this format:]

### [rule name - short, descriptive]

[One or two sentences describing the rule and why it matters in this codebase.]
[src: filename or inferred from ANALYSIS.md Code Style section]

[Repeat for each derived rule. Typical categories:]
[- Language/type conventions]
[- Module organization conventions]
[- Framework-specific do/don't patterns]
[- Error handling conventions]
[- Async conventions]
[- Testing conventions (if a test framework was observed)]

---

# project orientation

[Write 2-3 short paragraphs giving the AI enough context to begin work without
 reading all documentation first. Cover:
 - What the project does (one sentence)
 - The major subsystems and how they relate (two or three sentences)
 - The most important thing to know before touching the code (one sentence)
 Source: ANALYSIS.md Theory of Operation and Subsystems sections.
 Tag every factual sentence [src: filename] or [inferred].]

## subsystems at a glance

[For each subsystem in ANALYSIS.md with disposition: active, one row:]

    | Subsystem | Purpose | PRIMER available |
    |-----------|---------|-----------------|
    | [name]    | [one-line purpose] | [YES - docs/LORE_PRIMER_[name].md | NO] |

---

# reading sequence

Read the following documents now, in this order, before beginning any work.
Do not skip documents. Do not reorder them.

[For each document in the reading sequence from stage 7 Step 2, one numbered entry:]

    [N]. [filepath]
         [one-line description of what this document contains and why it matters]

After reading all [count] documents above, respond with exactly:

    ALL DOCUMENTATION INGESTED

Nothing else. No summary. No commentary. If you encountered an error or
inconsistency in any document, note it on the line before the acknowledgement.

---

# known issues and risks

[List any HIGH or MEDIUM severity items from ANALYSIS.md Observed Bugs and Risks.
 These are things an AI assistant should know before touching the code.
 Keep descriptions brief - one sentence each.
 Source: ANALYSIS.md Observed Bugs and Risks section.]

[For each HIGH or MEDIUM severity issue:]
    - [severity]: [one-sentence description] [src: filename]

[If no issues were found:]
    No HIGH or MEDIUM severity issues were identified during analysis. [src: ANALYSIS.md]

[If the bugs section was incomplete:]
    [INCOMPLETE - risk analysis was not completed for all subsystems.]

---

# planning a change

To create a disciplined development plan for a proposed change to this project,
share lore/PLAN.md with your AI assistant and say:

    Please follow `lore/PLAN.md` to create a plan.

The plan pipeline will ask you 6 short questions, produce a fully specified
implementation plan in docs/plans/, and guide implementation phase by phase
in subsequent context windows. Plans are human-reviewable before any code
is written. All commands are run by the human operator, not the AI.

To continue an existing plan:

    Please follow `lore/PLAN.md` and CONTINUE.

To resume implementation of a plan that is already approved:

    Please follow `docs/plans/PLAN_[label].md` and CONTINUE.

---

# document end

[If git_available is YES in SKELETON.md, append:]
Primary contacts by subsystem (from git history): [src: git]
[For each directory in SKELETON.md Git Metadata directory_primary_author:]
    - [directory/]: [author name]

[If git_available is NO, omit the contacts section entirely.]