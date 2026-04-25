# LORE Self-Update
# Repository: https://github.com/joshuawwbrown/lore.git

This file tells you how to update the lore/ directory in this project to the
latest version from the public lore repository.

It can be followed by an AI agent or by a human with a terminal.

---

## Before You Begin

The lore/ directory contains a VERSION file. The public repository also
contains a VERSION file. Compare them to determine whether changes are
expected. Even if they match, a full content diff is always performed -
a version bump may have been missed.

---

## Steps

1.  Read lore/VERSION in this project. Note the version string.

2.  Clone the public lore repository to a temporary location outside this
    project. Use any path that is convenient and writable. The examples below
    use TEMP as a placeholder for that location.

        git clone https://github.com/joshuawwbrown/lore.git TEMP

3.  Read TEMP/VERSION. Note the version string.

4.  Compare the two version strings and note whether they match or differ.
    This is informational only. Always continue to step 5 regardless.

5.  Produce a list of differences between
    TEMP/ and lore/ in this project:

    -   Files present in /tmp/lore-update/ that are missing from lore/
        These are new files to be added.

    -   Files present in both locations whose contents differ.
        These are files to be updated.

    -   Files present in lore/ that are absent from TEMP/
        These are files that have been removed from lore and should be
        deleted locally. Report them to the human before deleting.

    Report this list to the human before making any changes.

6.  Wait for the human to confirm before proceeding.

7.  For each new or changed file identified in step 5:

        cp TEMP/<filename> lore/<filename>

    Preserve subdirectory structure. For example, template files live in
    lore/templates/ and should be copied to lore/templates/.

8.  For each file identified as removed (absent from the repo), ask the
    human explicitly before deleting it. Do not delete without confirmation.

9.  Delete the temporary clone:

        rm -rf TEMP

10. Read the updated lore/VERSION and report the new version string to the
    human.

11. If the schema version (the version string in lore/SCHEMAS.md) has changed,
    note this explicitly. A schema version change may require a full re-scan
    of existing documentation. See lore/CONCEPTS.md, section "schema version",
    for details.

---

## Notes for Human Operators

If you do not have git available, you can download a zip archive of the
repository from:

    https://github.com/joshuawwbrown/lore/archive/refs/heads/main.zip

Extract it, then follow steps 5 through 11 above using your file manager
or terminal copy commands instead of git.

---

## Notes for AI Agents

Do not modify any files outside the lore/ directory during this procedure.

Do not update lore/ files that the human has locally modified without
reporting the conflict first. If a file differs from the repo version AND
differs from what you would expect a clean install to contain, flag it as
a possible local customization and ask the human how to proceed.

This update procedure does not re-run the documentation pipeline. After
a lore update, the human may want to run a scoped update or full re-scan.
See lore/README.md for instructions.