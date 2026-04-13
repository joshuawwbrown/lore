# LORE_INDEX
# Template for docs/LORE_INDEX.md
# Consumed by: stage_4_lore.md
# Replace every [PLACEHOLDER] line with real content.
# Remove all lines that begin with # (template comments) before writing the final file.
# Do not leave any [PLACEHOLDER] strings in the output.

---

# documentation - [project_name]

This directory contains all generated documentation for the [project_name] project.
Documentation was produced by the LORE pipeline. To regenerate or update it,
share lore/LORE.md with your AI assistant and say "Please follow lore/LORE.md
to update the project documentation."

Last updated: [last_updated]

---

## documents in this directory

### lore_abstract.md

	File: docs/LORE_ABSTRACT.md
	Audience: new developers, project stakeholders
	Contents: what the project does, how it works at a high level, architecture
	          overview, setup instructions, key dependencies, known risks, authors

	Start here if you are new to this project.

---

### lore_brief.md

	File: docs/LORE_BRIEF.md
	Audience: AI coding assistants beginning a development session
	Contents: collaboration guidelines, project orientation, directed reading
	          sequence, ingestion acknowledgement protocol

	Share this file with your AI assistant at the start of each development
	session. It will read the listed documents and confirm when ready.

---

### lore_enchiridion.md

	File: docs/LORE_ENCHIRIDION.md
	Audience: developers working on the codebase, AI coding assistants
	Contents: full subsystem catalog, key files and functions directory,
	          data paradigm, infrastructure model, code style reference,
	          observed bugs and risks

	Use this as a lookup reference when reading or modifying code.
	Every factual claim in this document carries a source tag.

---

### lore_index.md (this file)

	File: docs/LORE_INDEX.md
	Audience: anyone arriving at this docs/ directory
	Contents: index of all documents and what each one contains

---

[# PRIMER BLOCK - repeat one entry per PRIMER file produced]
[# Include this block only if at least one PRIMER was produced]
[# If no PRIMERs were produced, remove this block entirely]

### lore_primer_[subsystem_name].md

	File: docs/LORE_PRIMER_[subsystem_name_lowercase].md
	Audience: developers working on or integrating with the [subsystem_name] subsystem
	Contents: subsystem purpose, key files, key functions and exports,
	          external dependencies, integration points, configuration reference,
	          usage notes

	Read this after the LORE_ENCHIRIDION if you are working directly on [subsystem_name].

[# END PRIMER BLOCK]

---

[# OTHER FILES BLOCK]
[# If any user-created files exist in docs/ that are not LORE output,]
[# list each one here with a one-line description.]
[# If no such files exist, remove this section entirely.]

## other files in this directory

[For each pre-existing non-LORE file in docs/:]

	File: docs/[filename]
	Note: this file was not produced by LORE and will not be modified by pipeline re-runs.
	Contents: [one-line description of what the file contains]

[# END OTHER FILES BLOCK]

---

## regenerating this documentation

To update these documents after code changes:

	1. Open a new context window with your AI assistant.
	2. Share lore/LORE.md and say "Please follow lore/LORE.md to update
	   the project documentation."
	3. Choose option [U] (Update docs after code changes) from the menu.

To re-run quality control only:

	1. Open a new context window with your AI assistant.
	2. Share lore/LORE.md and say "Please follow lore/LORE.md to update
	   the project documentation."
	3. Choose option [Q] (Re-run quality control only) from the menu.

To generate or regenerate the AI session brief:

	1. Open a new context window with your AI assistant.
	2. Share lore/LORE.md and say "Please follow lore/LORE.md to update
	   the project documentation."
	3. Choose option [B] (Generate project brief) from the menu.

Pipeline work files are stored in docs/.LORE/work/.
Do not edit work files manually unless recovering from a pipeline error.

---

## about LORE

LORE is a staged documentation pipeline that runs entirely within an AI
assistant context window. It produces this documentation from source code
analysis without requiring any build tools or external services.

Pipeline stages:
	1. stage_1_recon.md          - project structure reconnaissance
	2. stage_2_analysis.md       - deep file analysis
	2b. stage_2b_continuation.md - overflow pass for large codebases
	2c. stage_2c_subsystem_completion.md - subsystem completion gate
	3. stage_3_human_gate.md     - human question review
	4. stage_4_lore.md           - documentation production (produced this file)
	5. stage_5_qa.md             - adversarial quality control
	6. stage_6_update.md         - scoped update pass
	7. stage_7_brief.md          - AI session brief generation

Source: lore/LORE.md