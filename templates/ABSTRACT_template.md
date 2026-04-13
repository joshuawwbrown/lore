# LORE_ABSTRACT
# Template for docs/LORE_ABSTRACT.md
# Consumed by: stage_4_lore.md
# Replace every [PLACEHOLDER] line with real content.
# Remove all lines that begin with # (template comments) before writing the final file.
# Do not leave any [PLACEHOLDER] strings in the output.

---

# [project_name] - Project Abstract

## what is this project

[Write 2-4 sentences describing what this project is and what problem it solves.
 Audience: a new developer who has never seen this codebase.
 Source: ANALYSIS.md Theory of Operation, SKELETON.md Project Identity.
 Source tags are required in the output file: tag every factual sentence
 [src: filename] or [inferred]. Do not omit tags.]

## quick facts

	Language:        [language from SKELETON.md Project Identity]
	Runtime:         [runtime from SKELETON.md Project Identity]
	Framework:       [framework from SKELETON.md Project Identity, or "none detected"]
	Package manager: [package_manager from SKELETON.md Project Identity]
	Test framework:  [test_framework from SKELETON.md Project Identity, or "none detected"]
	Entry point(s):  [entry_points from SKELETON.md Project Identity]

---

## how it works

[Write a narrative of the system's operation end to end. 300-600 words.
 Cover: what triggers the system, how data flows through it, what it produces or serves.
 Do not list files. Describe behavior and purpose.
 Source: ANALYSIS.md Theory of Operation.
 Tag every factual sentence [src: filename] or [inferred].]

---

## architecture overview

[Write 1-2 paragraphs summarizing the subsystem layout. Cover how the major
 parts relate to each other. Do not go deep - the ENCHIRIDION covers depth.
 Source: ANALYSIS.md Subsystems section.]

### subsystems at a glance

[For each subsystem in ANALYSIS.md Subsystems, one row:]

	| Subsystem | Purpose | PRIMER available |
	|-----------|---------|-----------------|
	| [name]    | [one-line purpose from ANALYSIS.md] | [YES - see docs/LORE_[name]_PRIMER.md | NO] |

[If no subsystems were identified:]
	No distinct subsystems were identified. The codebase appears to be a single-layer application. [inferred]

---

## data layer

[Write 1-2 paragraphs on how data is stored and moved.
 Cover: database type, schema approach, data model shape, flow between layers.
 Source: ANALYSIS.md Data Paradigm section.
 If ANALYSIS.md Data Paradigm is INCOMPLETE, write:
     [INCOMPLETE - data layer analysis was not completed. See ANALYSIS.md for details.]]

---

## infrastructure and deployment

[Write 1-2 paragraphs on how the application runs in production.
 Cover: runtime environment, how it is started, containerization if any,
 cloud services if any, exposed ports if any.
 Source: ANALYSIS.md IT Paradigm section.
 If any Tier 1 question about deployment is unanswered, write the marker:
     [AWAITING ANSWER - Q[number]: question summary]
 where the unanswered information would go.]

---

## getting started

### prerequisites

[List software, tools, and environment requirements needed before setup.
 Source: SKELETON.md Dependency Summary, Entry Point Summaries, any observed README.
 Format each as a bullet: - [tool name] [version if known] - [why it is needed]]

### setup

[List the steps to get the project running locally.
 Number each step.
 Source: any setup instructions observed in the project README or entry point files.
 If no setup instructions were observed, write:
     No setup instructions were found in the project. [inferred]
     Typical steps for a [language/framework] project would apply. [inferred]]

### running the application

[Describe how to start the application.
 Source: SKELETON.md Entry Point Summaries, package.json scripts or equivalent.
 Include the exact command if it was observed. Tag [src: filename] if quoting.
 If the start command is unknown, write:
     [AWAITING ANSWER - Q[number]: how is this application started in development?]
     or use the inferred default if no Tier 1 question covers this.]

### running tests

[Describe how to run the test suite.
 Source: SKELETON.md Project Identity test_framework field, any observed test scripts.
 If no test framework was detected, write:
     No test framework was detected in this project. [src: SKELETON.md]]

---

## key dependencies

[List the most important external dependencies - not all of them, just the ones
 a developer needs to know about to understand or contribute to the project.
 Limit to 10 entries. Source: SKELETON.md Dependency Summary.
 Format:]

	| Package | Version | Purpose |
	|---------|---------|---------|
	| [name]  | [version or "unversioned"] | [one-line purpose from SKELETON.md] |

[If no dependencies were detected:]
	No external dependencies were detected. [src: SKELETON.md]

---

## known issues and risks

[List any bugs or risks flagged in ANALYSIS.md Observed Bugs and Risks.
 Use this format for each:]

	[N]. [severity: HIGH | MEDIUM | LOW] - [description]
	     Location: [src: file]

[If none were observed:]
	No bugs or risks were identified during analysis. [src: ANALYSIS.md]

[If ANALYSIS.md Observed Bugs and Risks section is INCOMPLETE:]
	[INCOMPLETE - risk analysis was not completed for all subsystems.]

---

## open questions

[List any Tier 3 questions from QUESTIONS.md that were not answered.
 These are cosmetic but surfaced here so the team can address them over time.
 Format: - Q[number]: [question text]
 If no Tier 3 questions are open, omit this section entirely.]

---

## authors

[Include this section only if git_available is YES in SKELETON.md Git Metadata.
 If git_available is NO, omit this section entirely.]

[If git_available is YES:]

	Overall contributors (by commit count): [src: git]
	[For each contributor from SKELETON.md Git Metadata overall_contributors:]
	- [author name] ([commit count] commits)

	Primary author by subsystem: [src: git]
	[For each directory in SKELETON.md Git Metadata directory_primary_author:]
	- [directory/]: [author name] (last commit: [date from directory_last_commit])

[If no git metadata was available:]
	[Omit this section - do not write it at all if git_available is NO]

---

## further reading

[List the other documents available for this project. Adjust paths as needed.]

	- docs/LORE_BRIEF.md        - AI session entry point and reading sequence
	- docs/LORE_ENCHIRIDION.md  - full technical reference for developers and AI agents
	- docs/LORE_INDEX.md        - guide to all documents in this docs/ directory
	[For each PRIMER produced:]
	- docs/LORE_PRIMER_[name].md - deep reference for the [name] subsystem