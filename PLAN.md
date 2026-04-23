# LORE Plan Pipeline
# Schema Version: 1.0
# Copyright (c) 2026 Joshua W W Brown. Licensed under the MIT License. See lore/LICENSE.

This file controls the LORE Plan pipeline. It produces a development plan
document in docs/plans/ that serves as a self-contained implementation recipe
for a proposed change to this project.

To start a new plan, share this file with your AI assistant and say:

    Please follow `lore/PLAN.md` to create a plan.

To continue an existing plan session, share this file and say:

    Please follow `lore/PLAN.md` and CONTINUE.

To continue implementing a plan that is already READY, share the spec file,
the plan file, and this file with your AI assistant and say:

    Please follow `docs/plans/PLAN_[label]_BLUEPRINT.md` and CONTINUE.

    The implementing agent will read the spec, the plan, and the current
    LORE_BRIEF.md before executing the next phase.

---

# What This System Does

This pipeline guides an AI assistant through creating, reviewing, and
executing a disciplined software development plan. Unlike the LORE
documentation pipeline, which describes an existing codebase, this pipeline
produces a forward-looking specification: what will change, exactly how,
in what order, and how to verify each step.

A plan produced by this pipeline is:

    - Specification-grounded: the implementing agent reads the confirmed
      spec before the plan, so the why behind every decision is available.
    - Self-contained: the implementing agent needs only the spec file, the
      plan file, the current LORE_BRIEF.md, and the LORE PRIMERs named in
      the plan.
    - Unambiguous: every file spec, logic sequence, and acceptance criterion
      is written out fully. Nothing is summarized if the summary could allow
      a wrong implementation.
    - Human-reviewable: a human reading the plan can verify it will work
      before a single line of code is written.
    - Resumable: implementation can be interrupted and continued in a new
      context window at any phase boundary.
    - Traceable: every decision, deviation, and discovery is recorded in
      the plan journal with a timestamp and event type. Spec amendments are
      recorded in the spec's own amendment log.

The pipeline runs in four major stages:

    Specify     The planning AI elicits the full requirement from the human
                using one or more structured models (user journey, process
                flow, feature list). It transcribes the human's input into
                all three models, reconciles them, and produces a confirmed
                specification document (PLAN_[label]_SPEC.md) before any
                planning begins.

    Draft       The planning AI reads the confirmed spec, produces a
                capability decomposition confirmed by the human, then writes
                the full plan document with file specs, phases, traceability,
                and a manual testing sequence.

    Review      The planning AI self-reviews the draft against a structured
                checklist. The human reviews and confirms before the plan
                is marked ready.

    Implement   The implementing AI reads the spec and the plan, reads the
                current brief, executes one phase at a time, and records
                progress in the plan journal. The human runs all commands
                and reports output.

---

# IMPORTANT: Deployment Exclusion

BY DEFAULT, THIS PIPELINE DOES NOT PLAN DEPLOYMENT STEPS.

Deployment of any artifact produced by a plan must be performed by a human
operator following the applicable deployment guide. The reasons for this
default are:

    - Deployment commands are often irreversible (pushing to production,
      disabling secrets, deleting revisions).
    - The planning AI has no visibility into the current state of the live
      environment, which roles the operator holds, or what the blast radius
      of a mistake is.
    - A plan that includes deployment steps creates pressure for an AI to
      execute those steps, which is dangerous.
    - Deployment procedures are already documented in LORE_GUIDE files.
      A plan should reference those guides, not duplicate or replace them.

A plan MAY include a "Deployment Considerations" section that:
    - Names which artifacts (images, scripts, config files) need to be
      deployed after implementation is complete.
    - Lists prerequisites the operator must verify before deploying.
    - References the applicable LORE_GUIDE document.
    - Warns about any implementation changes that affect the deploy pipeline.

A plan must never include:
    - Specific gcloud, kubectl, docker push, or equivalent commands framed
      as implementation steps to be executed.
    - A phase labeled "Deploy to staging" or "Deploy to production".
    - Any assumption that deployment is safe to perform automatically.

If the plan itself is a change to the deployment pipeline (e.g. modifying
deploy scripts, Dockerfiles, or YAML templates), those file changes are
implementation tasks and belong in the file specifications. The execution
of those scripts against live infrastructure is still out of scope.

To include deployment steps explicitly, the human must request it and
confirm they understand the risks. See PLAN_DRAFT stage, Step 4.

---

# Step 1 - Detect Pipeline State

Before doing anything else, check whether docs/plans/ exists and whether
any PLAN_*_BLUEPRINT.md files exist there.

    IF docs/plans/ does not exist OR no PLAN_*_BLUEPRINT.md files exist there:
        Pipeline state: NEW
        Go to: MENU A - New Plan

    IF docs/plans/ exists and PLAN_*_BLUEPRINT.md files exist there:

        Read the status field from the header block of each PLAN_*_BLUEPRINT.md file.
        If the prompt named a specific plan file, read only that file.
        If multiple plan files exist and none was specified, present the list
        and ask the human which plan to work with before proceeding.

        IF status is DRAFT:
            Go to: MENU B - Resume Draft

        IF status is IN_REVIEW:
            Go to: MENU C - Plan In Review

        IF status is READY:
            Go to: MENU D - Plan Ready to Implement

        IF status is IN_PROGRESS:
            Go to: MENU E - Implementation In Progress

        IF status is COMPLETE:
            Go to: MENU F - Plan Complete

        IF status is BLOCKED:
            Go to: MENU G - Plan Blocked

        IF status is ERROR:
            Go to: MENU H - Error Recovery

---

# Stage sequence for CONTINUE auto-routing

    last_stage_completed          next stage to execute
    -----------------------------------------------------------
    plan_spec                  -> EXECUTE DRAFT
    plan_draft                 -> EXECUTE REVIEW
    plan_review                -> EXECUTE HUMAN GATE
    plan_human_gate            -> plan status is READY, go to MENU D
    plan_phase_[N]             -> EXECUTE NEXT PHASE (N+1) or EXECUTE COMPLETION
                                  if N was the last phase

    Note: if status is DRAFT and spec_file is absent from the plan header,
    the plan predates the spec step. Treat it as if spec was confirmed and
    proceed to EXECUTE REVIEW or EXECUTE DRAFT as appropriate.

---

# MENU A - New Plan

No plan exists. You are starting a new one.

Before presenting the questionnaire, do the following:

    1. Check whether docs/LORE_BRIEF.md exists.
       If it does not exist:
           Output: "No LORE_BRIEF.md found. Run the LORE documentation pipeline
           first (lore/LORE.md) before creating a plan. The brief provides the
           project context this pipeline requires."
           Stop.

    2. Read docs/LORE_BRIEF.md. Note the last_updated field. This is the
       brief version you will stamp into the plan.

    3. Read the subsystem table from LORE_BRIEF.md. You will use it during
       the specification and capability decomposition steps.

Present this introduction to the user:

    You are starting a new development plan.

    I will ask you 3 questions. Answer with the bracketed letter(s) shown,
    or freeform for questions 1 and 2.

Then present the questionnaire exactly as follows. Wait for all answers
before proceeding.

---
PLANNING QUESTIONNAIRE

Q1. What is the nature of this change?
    You may select more than one if appropriate (e.g. F, T).

    [R] Replacing an existing subsystem
    [U] Upgrading a dependency or tool
    [F] Adding a new feature
    [M] Modify or enhance an existing feature
    [G] General refactor or cleanup
    [T] Troubleshooting / bug fix
    [O] Other -- describe briefly

    ANSWER:

Q2. Describe what you are changing.

    Write as much as you need. Do not summarize. Include what it does,
    why, and what the intended behavior is -- including any rules,
    constraints, or edge cases you consider part of the requirement,
    even if they seem obvious.

    ANSWER:

Q3. Do you have sample code, sample data, or documentation for an API
    or library that is relevant to this change?
    [Y] Yes -- share it now or tell me where it is
    [P] Partial -- share what you have, flag the rest
    [N] No

    ANSWER:
---

After receiving all answers:

    If Q3 answer is [Y] or [P]:
        Output: "Please share the reference material now, or tell me the file
        path where I can read it. I will embed it verbatim in the spec -- it
        will not be summarized."
        Wait for the human to provide or point to the material.
        Read any file paths they provide.

    Derive a plan label from Q2 (lowercase, underscored, max 4 words).
    Derive a plan type from Q1 (one or more codes, comma-separated).
    Set plan label:    [label]
    Set spec filename: docs/plans/PLAN_[label]_SPEC.md
    Set plan filename: docs/plans/PLAN_[label]_BLUEPRINT.md
    Set journal filename: docs/plans/PLAN_[label]_JOURNAL_1.md

    Create docs/plans/ if it does not exist.

    Go to: EXECUTE SPEC

---

# EXECUTE SPEC

This section runs after the questionnaire and before EXECUTE DRAFT.
Its job is to produce a confirmed specification document before any planning begins.
Do not write the plan until the spec is approved by the human.

## Step 1 - Read project context

    1. docs/LORE_BRIEF.md (already read in MENU A -- re-read if resuming)
    2. For each LORE PRIMER relevant to the subsystems this change touches:
       read it now. Cross-reference the subsystem table from LORE_BRIEF.md
       with the Q1 and Q2 answers. When in doubt, read it.
    3. If Q3 was [Y] or [P]: confirm you have the reference material in context.

## Step 2 - Choose an elicitation model and gather the specification

Explain the three elicitation models to the human, suggest one based on
the plan type and what you know about the project, then accept whatever
the human provides -- including combinations and hybrids.

Present the following to the human:

---
To help me write a complete specification, I can use one or more of these
approaches. I will suggest one based on your change type, but you can use
any combination:

    [J] User journey -- a sequence of interactions as experienced by the
        person or system using this feature, step by step.
        Example: "Admin opens Status tab, clicks Upload, selects a .xlsx file,
        sees a progress indicator, sees a success message with record count."

    [P] Process flow -- the execution sequence of business logic, as the
        system carries it out internally.
        Example: "Handler receives file path. Reads rows in batches of 1000.
        For each row: strips non-digits, classifies type, upserts to DB.
        Returns inserted/updated/skipped counts."

    [F] Feature list -- an enumeration of discrete capabilities this change
        adds, modifies, or removes, with the rules that govern each.
        Example: "Litigator filter: optional, 1 credit per record, returns
        litigator/litigator_type/litigator_name fields, sets action=unsubscribe
        on hit."

Based on your change type ([plan_type]) and this project, I suggest: [one of
J, P, or F, with a one-sentence reason].

Please describe your change using whichever approach feels natural.
You can mix approaches, use bullet points, paste screenshots as descriptions,
or write prose. I will transcribe whatever you provide into all three models.

ANSWER:
---

Wait for the human's response before proceeding.

## Step 3 - Transcribe into all three models

Take whatever the human provided and produce a structured version in each
of the three models, regardless of which model they used. It is normal for
gaps to appear during this transcription -- a feature list may not specify
the journey, a journey may not specify internal logic. Record any gaps as
open questions (see Step 4).

Write all three transcriptions internally before presenting anything.
Do not show the human partial work.

## Step 4 - Reconcile the models

Compare the three transcriptions and identify:

    - Anything the user journey implies that the feature list does not name
    - Anything the process flow implies that the journey does not show
    - Anything the feature list names that has no corresponding journey step
      or process flow step
    - Any step in any model that has no clear owner, trigger, or output

Each discrepancy is either:
    - A gap (something missing from the spec that needs to be filled)
    - A conflict (two models disagree and need resolution)
    - An assumption (something the AI inferred that should be confirmed)

Record all gaps, conflicts, and assumptions as open questions.

## Step 5 - Write the spec file

Write docs/plans/PLAN_[label]_SPEC.md using the format below.

Use only ASCII characters. No unicode. No emojis.
Use tabs for indentation.
Number every section using legal-style outline numbering (1, 1.A, 1.A.1).
This numbering is used in the plan and proposal to cite specific requirements.

Tell the human at the top of every spec how to annotate it:

---
SPEC FILE FORMAT:

# Specification: [plan_label]

spec_label:         [label]
plan_type:          [type codes, comma-separated]
status:             DRAFT
created:            [YYYY-MM-DD HH:MM]
last_updated:       [YYYY-MM-DD HH:MM]
brief_version:      [last_updated from LORE_BRIEF.md]
plan_file:          docs/plans/PLAN_[label]_BLUEPRINT.md
amendment_count:    0

---

REVIEWER ANNOTATIONS

	You may annotate this document at any point using >> at the start of
	a line. Annotations are notes or corrections for the AI to incorporate.
	They are not required. Examples:

		>> 1.B.2 -- the field name is actually `hadWPassive4`, not `hadWPassive`

		>> The upload size limit is 50MB, not 10MB.

	You can also raise issues directly in the chat instead of annotating
	the document. Either approach works.

---

1. Overview

	[Two to four sentences. What this change does and why, written so a
	human unfamiliar with the implementation can understand it.]

2. Capabilities

	[One numbered entry per distinct capability. A capability is a discrete
	thing the system will be able to do after this change that it cannot do
	now, or a behavior that will change.]

	2.A [Capability name]

		[One paragraph describing what this capability does, the rules
		that govern it, and the expected outcome. Be specific: include
		field names, credit costs, status values, error conditions, and
		any other concrete detail provided by the human.]

	2.B [Capability name]
		[...]

3. User Journey

	[The sequence of interactions as experienced by the person or system
	that uses this feature. Written in steps. If multiple actors or entry
	points exist, write one journey per actor.]

	3.A [Actor or entry point name]

		Step 1: [what the actor does or what triggers the sequence]
		Step 2: [what the system does in response]
		Step 3: [what the actor sees or receives]
		...

4. Process Flow

	[The internal execution sequence as carried out by the system.
	Written in steps. If multiple code paths exist, write one flow per path.]

	4.A [Flow name]

		Step 1: [what initiates this flow]
		Step 2: [what the system does]
		Step 3: [what the system checks or decides]
		Step 4: [what the system produces or stores]
		...

5. Feature List

	[A flat enumeration of every discrete capability, rule, constraint,
	or behavior this change introduces or modifies.]

	5.A [Feature name]: [description, rules, and any concrete values]
	5.B [Feature name]: [...]
	...

6. Open Questions

	[Any gap, conflict, or assumption identified during reconciliation
	that must be resolved before planning begins. Number each.]

	6.1 [Question]: [what needs to be answered and why]
	6.2 [Question]: [...]

7. Amendment Log

	[Recorded by the AI whenever the spec is amended after initial approval.
	Do not fill in during drafting -- leave the header only.]

	[No amendments recorded.]

---
END OF SPEC FILE FORMAT

## Step 6 - Present the spec to the human and wait for confirmation

After writing the spec file, present the following to the human:

---
I have produced a specification document:

	docs/plans/PLAN_[label]_SPEC.md

Please open and read it. If anything is substantially wrong or missing,
you can:

	- Annotate the document directly using >> at the start of a line:
	      >> 2.A -- the credit cost is 2, not 1
	  Then choose [R] below to have me read your notes.

	- Raise issues here in the chat, like:
	      "In section 3.A you describe an admin upload but it should
	       be client-facing."

What would you like to do?

	[R] Read your notes on the spec
	[C] Continue to planning
	(or describe any issues here directly)
---

Wait for the human to respond. Do not proceed until one of the following
is true:
	- The human chooses [C] with no outstanding issues
	- The human raises issues, you resolve them, and they confirm [C]
	- The human annotates the spec, you incorporate the annotations
	  (amending the spec per Step 7 if the content changes materially),
	  and they confirm [C]

## Step 7 - Amend the spec when corrections are made

If the human's feedback (whether inline annotations or chat) requires
changing the spec content:

	1. Make the change in the spec file.
	2. If the change is material (not a typo or formatting fix), add an
	   entry to section 7 (Amendment Log):

		AMENDMENT [N]
		Date:    [YYYY-MM-DD HH:MM]
		Section: [section number]
		Change:  [one sentence describing what changed]
		Reason:  [why it changed -- human correction, planning discovery, etc.]

	3. Increment amendment_count in the spec header.
	4. Re-present the spec gate (Step 6) so the human can confirm the
	   updated version.

## Step 8 - Update spec header and proceed

Once the human confirms [C]:

	Update the spec header:
		status:       CONFIRMED
		last_updated: [current timestamp]

	Append to the plan journal (create it now if it does not exist):

		EVENT: SPEC_CONFIRMED
		DATE:  [YYYY-MM-DD HH:MM]
		PLAN:  [plan_label]
		SPEC:  docs/plans/PLAN_[label]_SPEC.md
		AMENDMENTS: [count, or 0]
		NOTE:  Specification confirmed. Proceeding to planning.

	Go to: EXECUTE DRAFT

---

# MENU B - Resume Draft

A plan draft was started but not completed.

Read the plan file header block for:
    - plan_label
    - plan_type
    - created
    - last_updated
    - last_stage_completed
    - spec_file (may be absent if plan predates the spec step)

Present this to the user:

    An unfinished plan draft was found.

    Plan: [plan_label]
    Type: [plan_type]
    Started: [created]
    Last updated: [last_updated]
    Last stage: [last_stage_completed]
    Spec: [spec_file if present, or "none -- predates spec step"]

    Options:
        [C] Continue -- resume the draft
        [R] Restart -- discard this draft and start a new one
        [X] Cancel -- do nothing

When the user responds:

    If C:
        If last_stage_completed is "plan_spec" or spec_file is present and
        status is CONFIRMED: Go to EXECUTE DRAFT.
        If last_stage_completed is absent or spec_file is absent:
            The plan predates the spec step. Go to EXECUTE DRAFT directly.
            Note in the journal that spec step was skipped (legacy plan).
        Otherwise: Go to EXECUTE SPEC to resume specification.
    If R: Confirm "This will delete the draft and spec. Are you sure? [Y/N]"
          If Y: delete the plan file and spec file (if present) and go to MENU A.
          If N: re-present MENU B.
    If X: Output "No action taken." Stop.

---

# MENU C - Plan In Review

The draft is complete and self-review is either pending or in progress.

Read the plan file. Present:

    Plan: [plan_label]
    Type: [plan_type]
    Status: IN_REVIEW
    Last updated: [last_updated]

    Options:
        [C] Continue -- run or re-run the self-review
        [H] Proceed to human gate -- skip self-review (not recommended)
        [X] Cancel -- do nothing

When the user responds:

    If C: Go to EXECUTE REVIEW.
    If H: Go to EXECUTE HUMAN GATE.
    If X: Output "No action taken." Stop.

---

# MENU D - Plan Ready to Implement

The plan has passed review and the human has confirmed it.

Read the plan file. Identify the first unchecked phase.

Present:

    Plan: [plan_label]
    Type: [plan_type]
    Status: READY
    Brief version stamped: [brief_version from plan header]

    Implementation phases:
    [For each phase, show: [DONE] or [ ] and the phase name]

    To begin or continue implementation, start a new context window and say:

        Please follow `docs/plans/PLAN_[label]_BLUEPRINT.md` and CONTINUE.

    The implementing agent will read this plan, read the current brief,
    check for brief version changes, and execute the next unchecked phase.

    Options:
        [C] Continue implementation in this context (not recommended for
            large plans -- prefer a fresh context)
        [A] Amend the plan before implementing
        [X] Cancel -- do nothing

When the user responds:

    If C: Go to EXECUTE NEXT PHASE.
    If A: Go to EXECUTE AMENDMENT.
    If X: Output "No action taken." Stop.

---

# MENU E - Implementation In Progress

Implementation has started. At least one phase is complete.

Read the plan file and the latest journal file. Find the last completed
phase and the next unchecked phase.

Present:

    Plan: [plan_label]
    Type: [plan_type]
    Status: IN_PROGRESS

    Progress:
    [For each phase, show: [DONE] or [ ] or [BLOCKED] and the phase name]

    Last journal entry:
    [paste the most recent journal entry from the latest journal file]

    Options:
        [C] Continue -- execute the next unchecked phase
        [A] Amend the plan
        [B] Mark a phase as blocked and record reason
        [X] Cancel -- do nothing

When the user responds:

    If C: Go to EXECUTE NEXT PHASE.
    If A: Go to EXECUTE AMENDMENT.
    If B: Go to EXECUTE BLOCK.
    If X: Output "No action taken." Stop.

---

# MENU F - Plan Complete

All phases are done. The plan is complete.

Read the plan file. Present:

    Plan: [plan_label]
    Type: [plan_type]
    Status: COMPLETE
    Completed: [last_updated]

    All phases complete.

    Next step: the next time LORE is run, it will offer to update project
    documentation to reflect the changes made in this plan.

    Options:
        [L] Update LORE now -- run the LORE update pipeline
        [X] Done -- no further action

When the user responds:

    If L: Output "Share lore/LORE.md with your AI assistant and say:
           Please follow lore/LORE.md and CONTINUE."
    If X: Output "Plan archived. No further action." Stop.

---

# MENU G - Plan Blocked

A phase is blocked and cannot proceed.

Read the plan file and latest journal. Find the PHASE_BLOCKED entry.

Present:

    Plan: [plan_label]
    Status: BLOCKED

    Blocked phase: [phase name]
    Reason: [blocking reason from journal]
    Recorded: [timestamp]

    Options:
        [U] Unblock -- record that the blocker is resolved and continue
        [A] Amend the plan -- revise approach to work around the blocker
        [X] Cancel -- do nothing

When the user responds:

    If U: Ask "What resolved the blocker?" Record a DISCOVERY journal entry.
          Set plan status to IN_PROGRESS. Go to EXECUTE NEXT PHASE.
    If A: Go to EXECUTE AMENDMENT.
    If X: Output "No action taken." Stop.

---

# MENU H - Error Recovery

An error was recorded in the plan.

Read the plan file. Find the ERROR journal entry.

Present:

    Plan: [plan_label]
    Status: ERROR
    Error: [error description from journal]

    Options:
        [R] Retry the phase that failed
        [A] Amend the plan and retry
        [S] Skip the failed phase (use with caution)
        [X] Cancel -- do nothing

When the user responds:

    If R: Record a journal entry: EVENT: ERROR_RETRY. Go to EXECUTE NEXT PHASE.
    If A: Go to EXECUTE AMENDMENT.
    If S: Confirm "Skipping may leave the system in a partial state. Proceed? [Y/N]"
          If Y: record DEVIATION journal entry, mark phase skipped, continue.
          If N: re-present MENU H.
    If X: Output "No action taken." Stop.

---

# EXECUTE DRAFT

Read this section and execute it fully before writing the plan file.

## Step 1 - Read project context

You must read the following before writing anything:

	1. docs/LORE_BRIEF.md (already read in MENU A -- re-read if resuming)
	   Note: last_updated field = brief_version to stamp in the plan.

	2. For each LORE PRIMER that is relevant to the subsystems this plan
	   touches: read it now.
	   How to determine relevance: use the subsystem table from LORE_BRIEF.md
	   and cross-reference with Q1 and Q2 answers. When in doubt, read it.
	   Record which PRIMERs you read -- they will be listed in the plan header.

	3. If Q3 answer was [Y] or [P]: confirm you have the reference material
	   in context. If not, ask for it before proceeding.

	4. Read the confirmed spec file: docs/plans/PLAN_[label]_SPEC.md
	   This is the authoritative source for what this plan must implement.
	   Every capability in the spec must be traceable to at least one phase.

## Step 2 - Produce a capability decomposition and confirm scope

Before writing any part of the plan, extract every capability from the
spec and organize it by surface area. Present this to the human and wait
for confirmation. Do not proceed to Step 3 until the human confirms.

For each capability in SPEC section 2, consider every surface area below.
Use what you have read in the PRIMERs to identify the specific files and
components in this project that serve each purpose. Name them explicitly.
Do not leave a surface area blank without stating why it does not apply.

	CORE BEHAVIOR
		The fundamental logic change: what the system does differently.
		Examples: a new algorithm, a changed pipeline stage, a new rule
		applied at runtime, a bug fix changing the outcome of an operation.

	INPUT SURFACE
		How operators, users, callers, or upstream systems initiate or
		configure this change.
		Examples: REST or RPC API parameters, message queue topics or event
		schemas, command-line flags or arguments, configuration files or
		environment variables, file uploads or batch input formats,
		webhooks or callbacks from external systems.

	OUTPUT SURFACE
		What downstream systems, users, or operators receive or observe
		as a result of this change.
		Examples: new or changed API response fields, new or changed log
		lines, metrics, or traces, emitted events or queue records,
		generated files or reports, rendered UI elements or pages.

	PERSISTENT STORAGE
		Any new or changed state that outlives a single request or process.
		Examples: database collections, tables, or fields, indexes or schema
		changes, on-disk files or directory structures, caches (Redis,
		Memcached, in-process with TTL), object storage (S3, GCS).

	OPERATOR TOOLING
		Anything a human uses to manage, configure, observe, or debug this
		system -- whether they are an admin, developer, or operator.
		Examples: admin UI panels or dashboards, CLI commands or scripts,
		health check or status endpoints, monitoring dashboards or alert
		rules, import/export or maintenance tools, interactive testing or
		exploration tools.
		For every new backend handler or action: ask whether a human needs
		a UI to invoke or observe it. If yes, name the component. If no,
		state why not.

	DOCUMENTATION
		Anything a human reads to understand or use this change.
		Examples: API reference docs (inline, Swagger, or a docs component),
		field description registries that drive UI labels or result tables,
		README or operator runbook updates, inline help text or tooltips.
		Check: does this project have an API documentation component?
		Check: does this project have a field description registry?
		If either exists and this change touches the API surface: name them
		here explicitly.

	EXTERNAL INTEGRATION
		New or changed connections to systems outside this codebase.
		Examples: third-party API calls (new endpoints, changed auth, new
		fields), SFTP or file-based data exchange, email or SMS providers,
		payment processors, cloud platform services.

	TESTING
		How to verify the change works correctly end to end.
		Not "verify it works." Specific named inputs and expected outputs.
		Examples: "call X with input Y, expect response Z", "upload file F,
		confirm record count N in collection C", "send event E, confirm
		downstream system D receives message M", "run with feature flag
		absent, confirm existing behavior is unchanged."

Work through the decomposition internally. Do not present the full
structured decomposition to the human.

When the decomposition is complete, present a short scope summary:

---
SCOPE CONFIRMATION

Based on the spec, here is the scope of this plan:

	[2-4 bullet points naming the key capabilities and the main files
	 or components affected. One line per capability. Be specific:
	 name actual files and components from the project.]

	Files that will change: [count]
	Files that will NOT change: [list the most notable exclusions, or "none identified yet"]

Does this look right? Any capabilities missing or files that should be excluded?
---

Wait for the human to respond.

	If the human confirms: proceed to Step 3.

	If the human corrects or adds to the decomposition:
		Update the internal decomposition.
		Re-present the corrected scope summary.
		Ask for confirmation again.
		Do not proceed until the human confirms.

The confirmed decomposition is the authoritative scope for this plan.
Every item in the decomposition must appear in Files Changed in the plan,
or be explicitly listed in Files Not Changed with a stated reason for
exclusion. Any item that appears in neither section is a FAIL in the
review checklist.

If any correction to the decomposition reveals an error or gap in the spec
itself, amend the spec per EXECUTE SPEC Step 7 before continuing. Record
the amendment in the spec's Amendment Log and in the plan journal.

## Step 3 - Determine plan type emphasis

The plan type is set by Q1 and may contain multiple codes. Apply the
emphasis for every code present.

	[R] Replacing a subsystem:
		Emphasize: invariants that must not change, transition state,
		parallelism with old system, rollback path.

	[U] Upgrading a dependency or tool:
		Emphasize: compatibility matrix, files affected, rollback procedure,
		testing checkpoints at each upgrade step.

	[F] Adding a new feature:
		Emphasize: new file specifications, integration points with existing
		subsystems, acceptance criteria for each new behavior.

	[M] Modifying or enhancing an existing feature:
		Emphasize: before/after behavior description, which existing components
		change and how, what the user or operator experience looks like before
		and after, and which behaviors or invariants must be preserved.
		Flag any risk of unintended side effects on adjacent features.

	[G] General refactor or cleanup:
		Emphasize: before/after behavior equivalence, what must not change,
		test coverage of changed paths.

	[T] Troubleshooting / bug fix:
		Emphasize: root cause statement, reproduction steps, exact fix,
		verification that the fix resolves the root cause without side effects.

	[O] Other:
		Use judgment. Apply all sections. Flag anything that does not fit.

## Step 4 - Assess deployment scope

Review the confirmed decomposition and the plan type.

	If any of the following are true:
		- the decomposition includes production files in any surface area
		- the decomposition includes external integrations
		- plan type includes [R] (replacing a subsystem)
		- plan type includes [U] (upgrading a dependency)

	Then: include a "Deployment Considerations" section in the plan.
	That section must contain:
		- Which artifacts need to be deployed after implementation
		- Prerequisites the operator must verify before deploying
		- A reference to the applicable LORE_GUIDE file if one exists
		- Any warnings about implementation changes affecting the deploy pipeline

	Always include the standard Deployment Exclusion notice at the top of
	any Deployment Considerations section (see template below).

	If none of the above are true: omit the Deployment Considerations section.
	Do not ask the human -- the default is omission.

	EXCEPTION: if the human explicitly requested deployment steps be included,
	include them in a clearly marked section titled "Deployment Steps (Human
	Operator Only)" and prefix every command with:
		[OPERATOR RUNS] <command>
	Never frame these as steps an AI should execute.

## Step 5 - Identify reference material handling

If Q3 answer was [Y] or [P]:

	ALL reference material (sample code, protocol specs, exact command
	sequences, API contracts, configuration schemas) must be embedded
	verbatim in the plan under a clearly marked section.

	DO NOT summarize, paraphrase, or condense reference material.

	Each verbatim block must be preceded by this warning:

		VERBATIM REFERENCE -- do not paraphrase
		The following must be reproduced exactly by the implementing agent.
		Summarizing this section has caused implementation errors in the past.

	If Q3 was [P] (partial): list the areas where reference material is
	missing. Mark those sections in the plan with:

		[UNSPECIFIED - no reference available. Implementing agent must ask
		 the human before proceeding with this section.]

## Step 6 - Write the plan file

Write the plan to docs/plans/PLAN_[label]_BLUEPRINT.md using the structure below.
Write every section. Do not skip sections -- use "N/A" with a reason if
a section genuinely does not apply.

Use only ASCII characters. No unicode. No emojis.
Use tabs for indentation. Four spaces wide.

---
PLAN FILE STRUCTURE:

# Plan: [plan_label]

plan_label:         [label]
plan_type:          [type codes from Q1, comma-separated]
status:             DRAFT
created:            [YYYY-MM-DD HH:MM]
last_updated:       [YYYY-MM-DD HH:MM]
brief_version:      [last_updated from LORE_BRIEF.md]
primers_read:       [comma-separated list of PRIMER filenames read]
last_stage_completed: plan_draft
spec_file:          docs/plans/PLAN_[label]_SPEC.md
journal_file:       docs/plans/PLAN_[label]_JOURNAL_1.md
journal_line_count: 0
plan_line_count:    [line count of this file]
deployment_guide:   NO
proposal_file:      docs/plans/PLAN_[label]_PROPOSAL.md

---

## Overview

[Two to four sentences. What is being changed, why, and what the end state
looks like. Written so a human can understand it without reading the rest
of the plan. This section summarizes the spec -- do not restate every detail.
Reference the spec for the full requirement: see SPEC_FILE.]

---

## Scope

### Files Changed

[List every file that will be created or modified. For each file:]
	[filename]: [CREATE | MODIFY | DELETE] -- [one-line purpose]

### Files Not Changed

[List any file that might seem related but must not be touched. For each
file, state explicitly why it is excluded. "Not applicable" is not
sufficient -- explain what aspect of this change does not require it.

This section must be actively reasoned, not passively assembled. Ask: is
there anything in the decomposition that this file would normally serve?
If yes and it is still excluded, explain why.]

### Not In Scope

[List any features or behaviors the human might reasonably expect this
plan to address, but which are explicitly deferred. State what is deferred
and why. This prevents the implementing agent from expanding scope and
prevents the human from being surprised at what is missing.]

### External Systems

[Name every external system this change interacts with, or write:
"None. This change is self-contained."]

### Production Impact

[State whether this change affects files currently in production, what the
impact window is, and whether a rollback path exists. If unknown, say so
and explain how to determine it before deploying.]

### Alternatives Considered

[For each significant design decision made in this plan, state what
alternatives were considered and why this approach was chosen. Short
entries are fine. The goal is to prevent the implementing agent from
silently reversing a decision the human already made.

If no significant design decisions were required, write: "None -- the
approach follows directly from the spec with no meaningful alternatives."]

---

## Invariants

[Things that must remain true throughout and after implementation. These
are the properties that define correctness for this change. The implementing
agent must verify each invariant is preserved after every phase.]

Examples of good invariants:
    - The existing API contract at /api must return the same response shape.
    - The google/ directory must not be modified.
    - All existing tests must continue to pass.
    - configLocal.js must remain loadable by the existing require() path.

[If plan type is [G] refactor or [R] replace: this section is mandatory
and must be thorough. A refactor with no invariants is unverifiable.]

---

## Reference Material

[If Q3 was [N]: write "None provided."]

[If Q3 was [Y] or [P]: embed all reference material here verbatim.
Precede each block with the VERBATIM REFERENCE warning.
If Q3 was [P]: list what is missing and mark with [UNSPECIFIED].]

---

## File Specifications

[For each file listed in Scope / Files Changed: write a full specification.
Each specification must contain:]

### [filename]

Purpose: [one sentence]

[For new files:]
    Module pattern: [CommonJS | ESM | other]
    Exports:
        [function or class name]: [signature, parameters, return type, behavior]
        [List every exported symbol. For each: state what it does, what it
         returns on success, what it does on failure, and whether it throws
         or returns an error value.]

    Internal functions: [list any private helpers with the same detail]

    Error handling: [how errors are caught, logged, and surfaced]

    Dependencies: [what this file imports, in order: built-ins, npm, local]

    "use strict": [YES | NO | N/A (ESM)]

[For modified files:]
    Current behavior: [what the file does today]
    Changes:
        [function or section name]:
            Before: [exact current behavior]
            After:  [exact new behavior]
            Reason: [why this change is needed]
    Lines affected: [approximate line range or section name if known]
    Unchanged: [what must not change in this file]

[Do not write placeholder specs. If you cannot fully specify a file,
mark it [INCOMPLETE - needs further analysis] and flag it in the
review checklist.]

---

## Implementation Phases

[Divide the work into phases. Each phase must:]
    - Have a single clear goal
    - Leave the codebase in a valid (if incomplete) state when done
    - Have a verifiable acceptance criterion
    - Not depend on a result from a later phase

[Phase ordering rules:]
    - Foundation first: files with no local dependencies come first
    - Dependency order: if file A imports file B, B is implemented before A
    - Tests after implementation: if a test file is included, it comes after
      the file it tests
    - Human verification steps are phases, not footnotes

### Phase [N] -- [Phase Name]

Goal: [one sentence]

Tasks:
    [ ] [specific task -- one action per line]
    [ ] [specific task]
    [ ] ...

Acceptance Criterion:
    [Exact, verifiable condition. Not "verify it works." Examples:]
    - Run: [command]. Expected output: [exact output or pattern].
    - File [filename] exists and contains [specific content].
    - [function] returns [value] when called with [input].
    - No errors appear in [log location] after [action].

Human steps (operator runs these -- agent does not):
    [If any step requires running a command or checking a live system,
     list it here explicitly and label it OPERATOR RUNS.]
    OPERATOR RUNS: [command or action]
    OPERATOR REPORTS: [what to paste back]

---

## Requirement Traceability

[Map every capability from the spec to the phases that implement it.
If any capability has no implementing phase, that is a FAIL -- add the
missing phase before finalizing the plan.]

	Capability (SPEC ref)               Phases
	----------------------------------  ---------------------------
	[capability name] (SPEC 2.A)        Phase [N], Phase [N]
	[capability name] (SPEC 2.B)        Phase [N]
	...

## Development Checklist

[A flat summary of all phase tasks, for quick status tracking.]
[Updated by the implementing agent as phases complete.]

	Phase 1 -- [name]: [ ] pending
	Phase 2 -- [name]: [ ] pending
	...

---

## Deployment Considerations

[Include only if determined in Step 3 of EXECUTE DRAFT.]

DEPLOYMENT EXCLUSION NOTICE: This plan does not include deployment steps.
Deployment must be performed by a human operator following the applicable
deployment guide. No AI should execute deployment commands on behalf of
this plan.

Artifacts to deploy after implementation is complete:
    [list]

Operator prerequisites before deploying:
    [list]

Deployment guide reference:
    [docs/LORE_GUIDE_[name].md or "none available"]

Warnings:
    [any implementation changes that affect the deploy pipeline]

---

## Known Risks and Open Questions

[Things that could go wrong, things that are uncertain, and things the
implementing agent must ask the human about before proceeding.]

	Risk: [description] -- Mitigation: [approach]
	Open: [question that needs an answer before or during implementation]

## Testing Sequence

[A manual end-to-end testing sequence the operator runs after implementation
is complete to verify the change works as intended. Each test must name
specific inputs and expected outputs. "Verify it works" is not acceptable.

Each test should reference the capability it verifies (SPEC section).]

	Test 1 -- [capability name] (SPEC 2.A): [happy path]
		OPERATOR RUNS: [specific action or command]
		Expected:      [exact or described output]

	Test 2 -- [capability name] (SPEC 2.A): [feature absent, existing behavior]
		OPERATOR RUNS: [same action without the new feature enabled]
		Expected:      [output identical to pre-change behavior]

	Test 3 -- [next capability] (SPEC 2.B): [...]
		...

---

## Implementation Status

[Updated by the implementing agent during the implementation loop.
Do not fill this in during drafting -- leave all fields as shown.]

    Plan status:     DRAFT
    Phases complete: 0 of [N]
    Last phase done: none
    Last updated:    [created date]
    Journal file:    docs/plans/PLAN_[label]_JOURNAL_1.md
    Journal entries: 0

---

END OF PLAN FILE STRUCTURE
---

## Step 6 - Write the initial journal entry

Create docs/plans/PLAN_[label]_JOURNAL_1.md.

Write the first journal entry using this format:

    EVENT: PLAN_CREATED
    DATE: [YYYY-MM-DD HH:MM]
    PLAN: [plan_label]
    TYPE: [plan_type]
    BRIEF_VERSION: [brief_version]
    PRIMERS_READ: [list]
    NOTE: Plan draft produced. Proceeding to self-review.

## Step 7 - Update plan header

Update the plan file header:
    last_updated: [current timestamp]
    last_stage_completed: plan_draft
    status: IN_REVIEW
    journal_line_count: [current line count of journal file]
    plan_line_count: [current line count of plan file]

Go to: EXECUTE REVIEW

---

# EXECUTE REVIEW

The planning AI self-reviews the draft against a structured checklist.
This is not optional. Every item must be checked before the human sees
the plan.

## Step 1 - Run the review checklist

Read the plan file in full. Check every item below.
For each item, mark: [PASS] [FAIL] or [WARN].
FAIL means the plan has a defect that must be fixed before human review.
WARN means the plan has a weakness worth noting but not blocking.

COMPLETENESS CHECKS

    [ ] Every file listed in Scope / Files Changed has a full File Specification.
    [ ] Every File Specification for a new file lists all exported symbols with
        their full signatures and error behavior.
    [ ] Every File Specification for a modified file states Before/After for
        each changed function or section.
    [ ] Every phase has at least one task.
    [ ] Every phase has an Acceptance Criterion that is specific and verifiable.
        ("verify it works" is a FAIL. "Run X, expect Y" is a pass.)
    [ ] No phase depends on a result that comes from a later phase.
    [ ] The Development Checklist matches the phases exactly.
    [ ] The Invariants section is present and contains at least one entry
        (or explicitly states "none -- this change is purely additive").

REFERENCE MATERIAL CHECKS

    [ ] If Q3 was [Y] or [P]: all reference material is embedded verbatim,
        not summarized.
    [ ] If Q3 was [Y] or [P]: every verbatim block has the VERBATIM REFERENCE
        warning preceding it.
    [ ] If Q3 was [P]: all missing reference areas are marked [UNSPECIFIED].
    [ ] No section summarizes a protocol, API contract, or configuration schema
        that was provided in full by the human. If a summary exists in place
        of the full material: FAIL.

SCOPE CHECKS

    [ ] Files Not Changed section is present. Every entry has a stated reason
        for exclusion. "Not applicable" is not sufficient. A file may not
        appear in Files Not Changed solely because it was not considered.
    [ ] Not In Scope section is present. Any feature or behavior the human
        might reasonably expect but which is deferred must be listed here
        with a reason. An empty section is only acceptable if the spec
        accounts for every reasonable expectation.
    [ ] No deployment commands appear as implementation tasks.
        (Exception: if the human explicitly requested deployment steps,
         they are marked OPERATOR RUNS and are in a clearly labeled section.)
    [ ] External systems named in the decomposition are named in both the
        Scope section and in any File Specification that interacts with them.
    [ ] Production Impact section states whether production files are affected,
        what the impact window is, and whether a rollback path exists.

UI AND API SURFACE CHECKS

    [ ] For every new backend handler or action in the plan: the plan either
        includes a corresponding operator tooling entry point in Files Changed,
        or explicitly states in Files Not Changed that no UI or tooling is
        needed and why. "Modeled on X" is not a justification -- the specific
        file and component must be named.

    [ ] For every new or changed input surface item (API parameter, CLI flag,
        config field, etc.): the documentation component for this project is
        in Files Changed, or is explicitly listed in Files Not Changed with a
        stated reason.

    [ ] For every new or changed output surface item (response field, emitted
        event, generated file column, etc.): any field description registry
        or catalogue in this project is in Files Changed, or is explicitly
        listed in Files Not Changed with a stated reason.

    [ ] For every new input surface item: the interactive testing or
        exploration tool for this project (if one exists) is in Files Changed,
        or is explicitly listed in Files Not Changed with a stated reason.

    [ ] Every file name, function name, collection name, and field name in
        the File Specifications matches the actual name used in the codebase,
        or is explicitly flagged as a new name being introduced by this plan.
        Names that differ from the codebase without explanation are a FAIL.

TRACEABILITY CHECKS

    [ ] The Requirement Traceability table is present and every capability
        from the spec (SPEC section 2) has at least one implementing phase.
        A capability with no phase entry is a FAIL -- add the missing phase.

    [ ] The Testing Sequence covers:
        - The happy path for every capability in the traceability table
        - The case where each new feature is NOT enabled, confirming
          existing behavior is unchanged
        - At least one test for every new operator tooling entry point
        - At least one test for every new input surface parameter

BRIEF AND LORE CHECKS

    [ ] spec_file is stamped in the plan header and the spec file exists.
    [ ] brief_version is stamped in the plan header.
    [ ] primers_read lists all PRIMERs that are relevant to the subsystems
        this plan touches.
    [ ] The plan does not contradict any convention stated in LORE_BRIEF.md.
        (module system, async pattern, naming, error handling, etc.)
        If a contradiction exists: FAIL. Fix the plan to match the brief
        or add an explicit note that this plan intentionally deviates and why.

FORMAT CHECKS

    [ ] No unicode characters or emojis in the plan file.
    [ ] All indentation uses tabs.
    [ ] plan_line_count in the header is accurate.
    [ ] journal_line_count in the header is accurate.

## Step 2 - Fix all FAIL items

For each FAIL item: correct the plan file before proceeding.
Do not present a plan with FAIL items to the human.

For each WARN item: add a note to the Known Risks and Open Questions section.

## Step 3 - Write the PROPOSAL

Write docs/plans/PLAN_[label]_PROPOSAL.md. This file is the human-facing
approval document. It is generated fresh every time EXECUTE REVIEW runs,
overwriting any prior version. It contains all key content verbatim --
the human should not need to open the plan file to give a meaningful approval.

Use only ASCII characters. No unicode. No emojis.

---
PROPOSAL FILE STRUCTURE:

# Proposal: [plan_label]

plan_label:     [label]
plan_type:      [plan_type]
created:        [created]
last_updated:   [current timestamp]
brief_version:  [brief_version]
self_review:    [PASS | PASS_WITH_WARNINGS | FAIL_CORRECTED]

---

## What this proposal does

[Paste the Overview section from the plan verbatim.]

---

## Why

[One to three sentences derived from the Q1 answer and the plan context
explaining the motivation for this change. Not a restatement of the
overview -- the reason behind it.]

---

## Files that will change

[Paste the Files Changed list from the plan verbatim, including the
CREATE / MODIFY / DELETE label and one-line purpose for each file.]

---

## Files that will NOT change

[Paste the Files Not Changed list from the plan verbatim, including
the reason each file is excluded.]

---

## Constraints (Invariants)

[Paste the Invariants section from the plan verbatim.]

---

## Reference material

[If Q3 was [N]: write "None. No reference material was provided."]

[If Q3 was [Y] or [P]:
 Paste the full Reference Material section from the plan verbatim.
 Precede it with this notice:]

    IMPORTANT: The following content must be reproduced exactly by the
    implementing agent. It must not be summarized or paraphrased. Read
    it carefully -- errors or omissions here are the most common source
    of implementation mistakes that pass human review undetected.

[If Q3 was [P]: also paste any [UNSPECIFIED] markers with their section
names, and note:]

    The following areas have no reference material. The implementing agent
    will stop and ask for guidance when it reaches them:
        [list UNSPECIFIED section names]

---

## Production impact

[If Q2 was [N]: write "None. This change does not affect production files."]

[If Q2 was [Y] or [?]: paste the Production Impact section from the plan
verbatim. If Q2 was [?], add:]

    Note: production impact was marked unknown at planning time. Verify
    before approving.

---

## Known risks and open questions

[Paste the Known Risks and Open Questions section from the plan verbatim.
If it is empty, write "None identified."]

---

## Implementation phases (summary)

[Paste the Development Checklist from the plan verbatim. This gives the
human a quick map of what will happen in what order.]

---

## Deployment

[If deployment_guide is NO:]
    This proposal does not include deployment steps. If deployment planning
    is needed, choose [D] when approving.

[If deployment_guide is YES:]
    A human operator deployment runbook will be written at the end of
    implementation (not during planning) as docs/plans/PLAN_[label]_DEPLOY.md
    Deployment constraints noted during review:
        [list constraints from journal DEPLOYMENT_CONSTRAINTS field, or "none"]

---

## Your decision

This proposal contains everything you need to approve or push back.
The full blueprint is available at docs/plans/PLAN_[label]_BLUEPRINT.md
if you want to read the complete file specifications -- but it is not
required reading.

Respond with one of:

    [A] Approved -- looks good, proceed to implementation
    [D] Approved + deployment guide -- proceed, and produce a human
        operator deployment runbook at completion
    [C] Changes needed -- describe what needs to change (freeform)
    [X] Cancel

Deployment note: This plan does not include steps for an AI to execute
against live infrastructure. If you choose [D], the plan will read the
existing deployment guide (if one exists) to inform implementation
decisions, and will produce a human operator deployment runbook when
implementation is complete. All commands in that runbook are run by you,
not the AI.

---
END OF PROPOSAL FILE STRUCTURE

## Step 4 - Write a review journal entry

Append to the journal file:

    EVENT: PLAN_REVIEWED
    DATE: [YYYY-MM-DD HH:MM]
    RESULT: [PASS | PASS_WITH_WARNINGS | FAIL_CORRECTED]
    CHECKS_PASSED: [count]
    CHECKS_WARNED: [count]
    CHECKS_FAILED_AND_FIXED: [count]
    PROPOSAL_FILE: docs/plans/PLAN_[label]_PROPOSAL.md
    NOTES: [one line per WARN or corrected FAIL, or "none"]

## Step 5 - Update plan header

    last_updated: [current timestamp]
    last_stage_completed: plan_review
    status: IN_REVIEW
    journal_line_count: [updated count]
    plan_line_count: [updated count]

Go to: EXECUTE HUMAN GATE

---

# EXECUTE HUMAN GATE

The human reads the PROPOSAL and responds with a single verdict.
Do not ask the human to answer individual questions. Present the PROPOSAL
file and wait for one response.

## Step 1 - Present the PROPOSAL

Output the following to the human:

---
PROPOSAL READY FOR REVIEW

    docs/plans/PLAN_[label]_PROPOSAL.md

Read the proposal above. It contains everything you need to make a
decision: the goal, files that will change, constraints, reference
material, known risks, and the implementation phase summary.

The full blueprint is at docs/plans/PLAN_[label]_BLUEPRINT.md if you
want to read the complete file specifications -- but the proposal is
the required read. You should not need to open the blueprint to approve.

Self-review result: [PASS | PASS_WITH_WARNINGS | FAIL_CORRECTED]
[If PASS_WITH_WARNINGS:]
Warnings noted: [list warnings one per line, indented]

    [A] Approved -- looks good, proceed to implementation
    [D] Approved + deployment guide -- proceed, and produce a human
        operator deployment runbook at completion
    [C] Changes needed -- describe what needs to change (freeform)
    [X] Cancel
---

## Step 2 - Process human response

If [A]:

    Update plan file:
        status: READY
        deployment_guide: NO
        last_updated: [current timestamp]
        last_stage_completed: plan_human_gate

    Append to journal:
        EVENT: HUMAN_GATE
        DATE: [YYYY-MM-DD HH:MM]
        RESULT: APPROVED
        DEPLOYMENT_GUIDE: NO
        NOTE: Human approved. Plan is READY to implement.

    Output:
        Plan [plan_label] is approved and ready to implement.

        To begin implementation, start a new context window and say:
            Please follow `docs/plans/PLAN_[label]_BLUEPRINT.md` and CONTINUE.

    Stop. Do not begin implementation in this context.

If [D]:

    If docs/LORE_GUIDE_deploy_system.md exists (or any LORE_GUIDE file
    relevant to deployment for this project): read it now. Note any
    deployment constraints that affect implementation decisions. If any
    file specification in the plan must change as a result, apply the
    change and record a PLAN_AMENDED journal entry before proceeding.

    Update plan file:
        status: READY
        deployment_guide: YES
        last_updated: [current timestamp]
        last_stage_completed: plan_human_gate

    Append to journal:
        EVENT: HUMAN_GATE
        DATE: [YYYY-MM-DD HH:MM]
        RESULT: APPROVED
        DEPLOYMENT_GUIDE: YES
        NOTE: Human approved with deployment guide. A human operator
              deployment runbook will be written at completion.
        [If deployment guide was read and affected file specs:]
        DEPLOYMENT_CONSTRAINTS: [one line per constraint noted]

    Output:
        Plan [plan_label] is approved and ready to implement.
        A deployment runbook will be written when implementation is complete.

        To begin implementation, start a new context window and say:
            Please follow `docs/plans/PLAN_[label]_BLUEPRINT.md` and CONTINUE.

    Stop. Do not begin implementation in this context.

If [C]:

    Ask: "What needs to change?"
    Accept the human's freeform description.
    Apply the requested changes to the plan file.
    Record a journal entry:
        EVENT: PLAN_AMENDED
        DATE: [YYYY-MM-DD HH:MM]
        REASON: [human's stated reason]
        CHANGES: [one line per change made]
        PREVIOUS_STATUS: IN_REVIEW
    Re-run EXECUTE REVIEW from Step 1.
    A fresh PROPOSAL will be written as part of that review pass.
    Then re-present EXECUTE HUMAN GATE from Step 1.

If [X]:

    Update plan file:
        status: IN_REVIEW
        last_updated: [current timestamp]

    Output: "Review cancelled. Plan remains in IN_REVIEW status."
    Output: "To resume, share lore/PLAN.md and say: Please follow lore/PLAN.md and CONTINUE."
    Stop.

---

# EXECUTE NEXT PHASE

This section is executed by the implementing agent during the implementation
loop. Each invocation executes exactly one phase.

## Step 1 - Session startup (run once per context window)

At the start of every implementation context window:

    1. Read the spec file: docs/plans/PLAN_[label]_SPEC.md
       This is the authoritative statement of what this plan must deliver.
       Read it before the plan so the why informs your reading of the how.
       If the spec file does not exist, note this and proceed -- the plan
       was created before the spec step was introduced.

    2. Read this plan file (docs/plans/PLAN_[label]_BLUEPRINT.md) in full.

    3. Read the current docs/LORE_BRIEF.md.

    4. Compare the brief_version stamped in the plan header to the
       last_updated field of the current LORE_BRIEF.md.

       If they differ:
           Output:
               WARNING: The LORE brief has been updated since this plan was written.
               Plan brief version:    [plan value]
               Current brief version: [LORE_BRIEF.md value]

               This may mean project conventions, file locations, or subsystem
               behavior has changed since the plan was drafted. You must review
               the diff before proceeding.

               Please review the brief changes and confirm one of:
               [C] Changes do not affect this plan -- continue
               [A] Changes affect this plan -- amend the plan first

           Wait for the human to respond before proceeding.
           If [A]: go to EXECUTE AMENDMENT, then return here.
           If [C]: record a journal entry:
               EVENT: BRIEF_VERSION_MISMATCH
               DATE: [YYYY-MM-DD HH:MM]
               PLAN_VERSION: [plan value]
               CURRENT_VERSION: [LORE_BRIEF.md value]
               RESOLUTION: Human confirmed no impact. Proceeding.

    5. Read the latest journal file (check journal_file in plan header).
       Find the most recent entry. Determine the last completed phase.

    6. Read any LORE PRIMERs listed in primers_read that are relevant to
       the current phase. If a PRIMER was not read in a previous session
       and the current phase touches that subsystem, read it now.

    7. For every file in the current phase's task list: read the actual
       current state of that file in the codebase before writing anything.
       Do not rely solely on the plan's description of current behavior.
       If the actual file differs materially from the plan's "Current behavior"
       description, record a DISCOVERY journal entry and flag it to the human
       before proceeding with that task.

    8. Find the next unchecked phase in the Development Checklist.
       This is the phase you will execute.

    9. Output to the human:
           Resuming plan: [plan_label]
           Next phase: Phase [N] -- [phase name]
           Goal: [phase goal]
           Tasks:
           [list all tasks for this phase]

           Acceptance Criterion:
           [paste acceptance criterion]

           [If phase has OPERATOR RUNS steps:]
           This phase requires you to run commands and report output.
           I will tell you what to run and what to paste back.

        Wait for the human to confirm before executing.
        [C] Continue   [S] Skip this phase   [X] Stop

## Step 2 - Execute the phase

Work through each task in the phase in order.

For file creation or modification tasks:
    - Write or edit the file exactly as specified in the File Specifications
      section of this plan.
    - Do not add functionality not specified in the plan.
    - Do not modify files not listed in this phase's tasks.
    - If the spec is [INCOMPLETE] or [UNSPECIFIED]: stop and ask the human
      before writing anything.

For tasks that require running a command:
    - Output the command to the human with the prefix OPERATOR RUNS.
    - Wait for the human to run it and report back.
    - Record the output in a journal entry.
    - If the output does not match the expected result: record an ERROR
      journal entry and ask the human how to proceed. Do not continue.

For tasks that require reading a live system state:
    - Ask the human to provide the relevant output.
    - Do not assume the state matches the plan.

## Step 3 - Verify acceptance criterion

After all tasks are complete:

    Present the acceptance criterion to the human.
    Ask them to verify it.

    If the criterion requires running a command:
        Output: OPERATOR RUNS: [command]
        Output: OPERATOR REPORTS: [what to paste back]
        Wait for the report.

    If the criterion is met:
        Record journal entry:
            EVENT: PHASE_DONE
            DATE: [YYYY-MM-DD HH:MM]
            PHASE: [N] -- [phase name]
            CRITERION_MET: YES
            NOTES: [any relevant observations]

        Mark the phase as [DONE] in the Development Checklist.
        Mark each task in the phase as [x] in the Implementation Phases section.
        Update the Implementation Status section:
            Phases complete: [N] of [total]
            Last phase done: Phase [N] -- [name]
            Last updated: [current timestamp]

    If the criterion is not met:
        Record journal entry:
            EVENT: ERROR
            DATE: [YYYY-MM-DD HH:MM]
            PHASE: [N] -- [phase name]
            DESCRIPTION: [what failed]
            COMMAND_OUTPUT: [relevant output if applicable]
            ACTION_TAKEN: [what was changed to attempt a fix]

        Make at most 2 correction attempts.
        If still failing after 2 attempts:
            Record journal entry:
                EVENT: PHASE_BLOCKED
                DATE: [YYYY-MM-DD HH:MM]
                PHASE: [N] -- [phase name]
                REASON: [why it is blocked]
                NEEDS: [what the human must provide or decide]
            Update plan status to BLOCKED.
            Output the situation to the human and stop.

## Step 4 - Check journal file size

After recording the journal entry, check the line count of the current
journal file.

    If line count exceeds 400:
        Create a new journal file:
            docs/plans/PLAN_[label]_JOURNAL_[N+1].md
        Update the plan header:
            journal_file: docs/plans/PLAN_[label]_JOURNAL_[N+1].md
            journal_line_count: 0
        Add a link to the old journal file in the plan header under a
        new field:
            journal_archive: [comma-separated list of prior journal filenames]
        Write a bridging entry in the new journal:
            EVENT: JOURNAL_CONTINUED
            DATE: [YYYY-MM-DD HH:MM]
            FROM: [previous journal filename]
            NOTE: Journal split due to size. Prior entries in [previous filename].

## Step 5 - Check plan file size and update

Update the plan file:
    plan_line_count: [current line count]
    journal_line_count: [current line count of journal file]
    last_updated: [current timestamp]
    status: IN_PROGRESS (or BLOCKED if blocked)

If the plan file exceeds 800 lines:
    Append a note to the Known Risks section (do not restructure the file):
        NOTE [date]: Plan file is large ([N] lines). If context budget is
        a concern, the implementing agent may read only the Implementation
        Phases, Development Checklist, and Implementation Status sections
        to resume. The File Specifications section is reference material
        and need not be re-read on every resume if the relevant phase
        has already been planned.

## Step 6 - Determine next action

    If more unchecked phases remain:
        Output the handoff note (see HANDOFF NOTE FORMAT below).
        Stop. Do not begin the next phase in this context window unless
        the human explicitly asks to continue.

    If all phases are complete:
        Go to EXECUTE COMPLETION.

---

# EXECUTE COMPLETION

All phases are done. The plan is complete.

## Step 1 - Final verification

Read the Development Checklist. Confirm every phase is marked [DONE].
If any phase is not marked [DONE]: do not proceed. Report to the human.

## Step 2 - Write completion journal entry

    EVENT: PLAN_COMPLETE
    DATE: [YYYY-MM-DD HH:MM]
    PLAN: [plan_label]
    PHASES_COMPLETED: [N] of [N]
    BRIEF_VERSION_AT_START: [brief_version from plan header]
    BRIEF_VERSION_AT_END: [current LORE_BRIEF.md last_updated]
    LORE_UPDATE_NEEDED: [YES | NO]
    NOTE: [brief summary of what was built]
    NEXT_STEP: Run LORE pipeline (lore/LORE.md) to update project documentation.

    Set LORE_UPDATE_NEEDED to YES if any of the following are true:
        - New files were created that LORE has not documented
        - Existing subsystem behavior was changed
        - New external dependencies were added
        - The deploy pipeline was modified

## Step 3 - Update plan file

    status: COMPLETE
    last_updated: [current timestamp]
    last_stage_completed: plan_complete
    plan_line_count: [current count]
    journal_line_count: [current count]

    Update Implementation Status:
        Plan status: COMPLETE
        Phases complete: [N] of [N]
        Last phase done: Phase [N] -- [name]
        Last updated: [current timestamp]

## Step 4 - Write deployment runbook (if requested)

This step runs ONLY at the end of implementation, after all phases are
complete and verified. It must not be written during planning or review.

Check the deployment_guide field in the plan header.

If deployment_guide is NO: skip this step entirely.

If deployment_guide is YES:

    Read the final Implementation Status section and the journal in full.
    Build an accurate picture of exactly what was built, including any
    deviations from the original plan.

    Write docs/plans/PLAN_[label]_DEPLOY.md using the following structure:

    ---
    # Deployment Runbook: [plan_label]

    plan_label:     [label]
    created:        [YYYY-MM-DD HH:MM]
    plan_file:      docs/plans/PLAN_[label]_BLUEPRINT.md
    deploy_file:    docs/plans/PLAN_[label]_DEPLOY.md
    status:         DRAFT -- operator must review before use

    ---

    ## What was built

    [Two to four sentences summarizing what the implementation produced.
     Based on the Implementation Status and journal, not the original plan.
     If deviations occurred, describe the final state, not the planned state.]

    ## Artifacts to deploy

    [List every file, image, config, or secret that must be deployed.
     For each: filename, what it is, and what system consumes it.]

    ## Prerequisites

    [Everything the operator must verify or set up before running any
     deployment command. Be specific. If a prerequisite requires checking
     a live system, say so explicitly.]

    ## Deployment steps

    [Each step on its own block. Label every command OPERATOR RUNS.
     State what to look for in the output after each command.]

        Step [N]: [one-line description]
        OPERATOR RUNS: [exact command]
        Expected output: [what success looks like]
        If output is unexpected: [what to do]

    ## Rollback

    [How to undo this deployment if something goes wrong. Be specific.
     If there is no rollback path, say so explicitly and explain why.]

    ## Verification

    [How to confirm the deployment succeeded end to end. Include at least
     one command or observable that proves the new code is live and working.]

    ---

    Append to journal:
        EVENT: DEPLOYMENT_GUIDE_WRITTEN
        DATE: [YYYY-MM-DD HH:MM]
        FILE: docs/plans/PLAN_[label]_DEPLOY.md
        NOTE: Deployment runbook produced from final implementation state.
              Operator must review before use.

    Update plan header:
        last_updated: [current timestamp]
        plan_line_count: [current count]

    Output to the human:
        Deployment runbook written to docs/plans/PLAN_[label]_DEPLOY.md
        Review it before deploying -- it was produced from the final
        implementation state and should be accurate, but operator
        verification is required before running any commands.

## Step 5 - Output to the human

    Plan [plan_label] is complete. All [N] phases done.

    [If LORE_UPDATE_NEEDED is YES:]
    Project documentation should be updated to reflect these changes.
    The next time you run LORE, it will detect this plan and offer to
    update the documentation. Share lore/LORE.md with your AI assistant
    and say: Please follow lore/LORE.md and CONTINUE.

    [If brief version changed during implementation:]
    Note: the LORE brief was updated during implementation (from [start]
    to [end]). Verify that no plan deviations were caused by that change.

---

# EXECUTE AMENDMENT

A plan amendment is needed after implementation has started or during review.

## Step 1 - Identify what is changing

Ask the human (or read from context if it is already clear):
    - Which section or phase is changing?
    - What is the new requirement?
    - Why is the change needed?

## Step 2 - Record the amendment

Append to the journal:

    EVENT: PLAN_AMENDED
    DATE: [YYYY-MM-DD HH:MM]
    SECTION: [section or phase name]
    REASON: [why the change is needed]
    PREVIOUS: [one paragraph or bullet list of what the plan said before]
    NEW: [one paragraph or bullet list of what it says now]
    RISK: [any risk this amendment introduces]

## Step 3 - Apply the amendment to the plan file

Edit the relevant section of the plan file.
If the changed section is a File Specification or Phase, add an inline
amendment marker immediately before the changed content:

    [AMENDED [YYYY-MM-DD]: [one-line description of what changed]]

Do not delete the previous content if it is useful for understanding the
change. If it is long, move it to the journal PREVIOUS field and reference
the journal entry inline:

    [AMENDED [YYYY-MM-DD]: see journal entry [date] for previous version]

## Step 4 - Update plan header

    last_updated: [current timestamp]
    plan_line_count: [updated count]

Return to wherever this was called from.

---

# EXECUTE BLOCK

A phase cannot proceed.

Append to the journal:

    EVENT: PHASE_BLOCKED
    DATE: [YYYY-MM-DD HH:MM]
    PHASE: [N] -- [phase name]
    REASON: [specific description of what is blocking it]
    NEEDS: [what the human must provide, decide, or resolve]
    ATTEMPTED: [what was tried before concluding it is blocked]

Update plan file:
    status: BLOCKED
    last_updated: [current timestamp]

Output to the human:
    Phase [N] -- [phase name] is blocked.
    Reason: [reason]
    To unblock: [what is needed]

    When the blocker is resolved, resume with:
        Please follow `docs/plans/PLAN_[label]_BLUEPRINT.md` and CONTINUE.

Stop.

---

# HANDOFF NOTE FORMAT

Write this block at the end of any implementation context window that
does not end in completion or a block. Output it to the human.

---
Plan: [plan_label]
Status: IN_PROGRESS
Phase just completed: Phase [N] -- [phase name]
Next phase: Phase [N+1] -- [phase name]
Goal of next phase: [one sentence]

Journal file: [current journal filename] ([line count] lines)
Plan file: docs/plans/PLAN_[label]_BLUEPRINT.md ([line count] lines)

To continue in a new context window:

    Please follow `docs/plans/PLAN_[label]_BLUEPRINT.md` and CONTINUE.

The implementing agent will read the plan, read the current brief,
check for brief version changes, and execute Phase [N+1].
---

---

# Journal Event Types Reference

Every journal entry must use one of the following EVENT values.
No other values are permitted.

    PLAN_CREATED        Plan draft produced for the first time.
    PLAN_REVIEWED       Self-review checklist completed. PROPOSAL written or rewritten.
    HUMAN_GATE          Human reviewed and approved (or requested changes).
    PLAN_AMENDED        Plan content changed after initial draft.
    PHASE_STARTED       Implementing agent began a phase (optional, for long phases).
    PHASE_DONE          Phase completed and acceptance criterion verified.
    PHASE_BLOCKED       Phase cannot proceed. Needs human input.
    FILE_CREATED        A new file was written to disk.
    FILE_CHANGED        An existing file was modified.
    FILE_ABANDONED      A planned file was not created. Records why.
    HUMAN_INPUT         Human answered a question or provided guidance during
                        implementation (distinct from the pre-implementation gate).
    DISCOVERY           Something found during implementation that was not
                        anticipated by the plan. May or may not require an amendment.
    ERROR               A step failed. Records what happened and what was tried.
    ERROR_RETRY         A previously errored phase is being retried.
    DEVIATION           The implementing agent did something different from the
                        plan. Records what the plan said, what was done, and why.
                        A DEVIATION always requires a PLAN_AMENDED entry as well
                        unless the deviation was trivial (e.g. a variable name).
    BRIEF_VERSION_MISMATCH  Brief version changed since plan was written.
                            Records resolution.
    PROPOSAL_WRITTEN         PROPOSAL file written or rewritten after a review pass.
    JOURNAL_CONTINUED        Journal split to a new file due to size.
    LORE_UPDATED             LORE documentation was updated as a result of this plan.
    DEPLOYMENT_GUIDE_WRITTEN Deployment runbook written at plan completion.
    PLAN_COMPLETE            All phases done. Implementation finished.

---

# Journal Entry Format

Every journal entry uses this exact format:

    EVENT: [event type from list above]
    DATE: [YYYY-MM-DD HH:MM]
    [additional fields as required by event type]
    NOTE: [free text, one or more lines, indented with four spaces]

Required additional fields by event type:

    PLAN_CREATED:   PLAN, TYPE, BRIEF_VERSION, PRIMERS_READ
    PLAN_REVIEWED:  RESULT, CHECKS_PASSED, CHECKS_WARNED, CHECKS_FAILED_AND_FIXED,
                    PROPOSAL_FILE
    HUMAN_GATE:     RESULT (APPROVED | CHANGES_REQUESTED)
    PLAN_AMENDED:   SECTION, REASON, PREVIOUS, NEW, RISK
    PHASE_DONE:     PHASE, CRITERION_MET
    PHASE_BLOCKED:  PHASE, REASON, NEEDS, ATTEMPTED
    FILE_CREATED:   FILE, PURPOSE
    FILE_CHANGED:   FILE, WHAT, WHY
    FILE_ABANDONED: FILE, REASON
    DISCOVERY:      IMPACT (affects plan | no plan change needed), ACTION
    ERROR:          PHASE, DESCRIPTION, COMMAND_OUTPUT, ACTION_TAKEN
    HUMAN_INPUT:    QUESTION, ANSWER
    DEVIATION:      PLAN_SAID, DID_INSTEAD, WHY
    BRIEF_VERSION_MISMATCH: PLAN_VERSION, CURRENT_VERSION, RESOLUTION
    LORE_UPDATED:           DOCS_UPDATED (list)
    DEPLOYMENT_GUIDE_WRITTEN: FILE
    PLAN_COMPLETE:          PHASES_COMPLETED, BRIEF_VERSION_AT_START,
                            BRIEF_VERSION_AT_END, LORE_UPDATE_NEEDED

---

# Notes for All Stages

- Use only ASCII characters in all plan and journal files.
- No unicode. No emojis.
- Use tabs for indentation throughout.
- Tag every factual claim about the codebase with [src: filename] or [inferred].
- Never execute code. Never run commands. All command execution is performed
  by the human operator. The agent describes what to run and waits for output.
- Never assume a command succeeded. Always wait for the human to report output.
- Never add functionality to a file that is not specified in the plan.
- If the plan is ambiguous, ask the human. Do not resolve ambiguity silently.
- If context budget is running low, finish the current task, write a
  PHASE_BLOCKED or partial PHASE_DONE journal entry, update the plan header,
  output the handoff note, and stop.
- The plan is the source of truth. If the plan contradicts the brief on a
  specific point, the plan governs for that point only, and the contradiction
  must be noted in Known Risks.
- The human is the only one who can run code, execute commands, or verify
  live system state. The agent writes, reads, and reasons. The human acts.
```

Now save it: