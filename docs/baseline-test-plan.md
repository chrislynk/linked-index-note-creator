Linked-Index Note Creator -- Baseline Test Plan

1. Purpose

This document defines the baseline test process for the Linked-Index Note Creator before configuration, generalized note types, and type-specific template support are introduced.

The baseline establishes a known-working reference for the current single-script implementation.

The objectives are to:

* Verify the current workflow behaves as expected.
* Verify behavior independently on desktop and iPad.
* Document the expected behavior before refactoring.
* Identify existing defects separately from future regressions.
* Provide repeatable tests for future releases.
* Establish the behavior represented by the v0.1.0-baseline release.

No major refactoring should begin until the required baseline tests have been completed and failures have been resolved or classified.

⸻

2. Scope

The baseline tests cover:

* Root-index creation
* Primary-index creation
* Primary-index selection
* Linked-index creation
* Linked-index navigation
* Breadcrumbs
* Back navigation
* Maximum index depth
* Add Title Here
* Document creation
* Blog creation
* Task creation
* Tracker creation
* Date validation
* Filename generation
* Duplicate protection
* Target-index linking
* Alphabetical link sorting
* Mixed index and note links
* Preservation of non-link index content
* Desktop compatibility
* iPad compatibility

The baseline tests do not cover planned functionality that is not yet implemented.

Out of scope for the baseline:

* Root-index YAML configuration
* User-defined note types
* Type-specific templates
* Template bootstrapping
* Configurable title formats
* Multiple target indexes for one note
* Automated testing

⸻

3. Baseline Version

Record the exact version under test before beginning.

Repository:
linked-index-note-creator
Branch:
main
Baseline tag:
v0.1.0-baseline
Commit:
[commit SHA]
Script:
[script filename]
Test date:
[YYYY-MM-DD]
Tester:
[name or GitHub username]

If the baseline tag has not yet been created, record the commit SHA being tested and create the tag only after the required baseline tests pass.

⸻

4. Supported Test Platforms

The baseline should be tested independently on each supported platform.

Desktop

Device:
[device]
Operating system:
[OS and version]
Obsidian version:
[version]
Templater version:
[version]
Test date:
[YYYY-MM-DD]
Tester:
[name]

iPad

Device:
[iPad model]
iPadOS version:
[version]
Obsidian version:
[version]
Templater version:
[version]
Test date:
[YYYY-MM-DD]
Tester:
[name]

A test passing on one platform does not automatically count as passing on the other.

⸻

5. Test Vault

Use a dedicated Obsidian vault.

Recommended name:

Linked Index Test Vault

Do not perform baseline testing against a personal or production vault.

Required starting structure

Linked Index Test Vault/
├── Templates/
│   └── Linked-Index Note Creator.md
└── Test Notes/

The exact baseline script from the repository must be copied into:

Templates/Linked-Index Note Creator.md

The script should not be modified in the test vault.

⸻

6. Templater Configuration

Templater must be installed and enabled.

Configure the Templater template folder as:

Templates

If the production workflow uses Templater Folder Templates, reproduce that configuration in the test vault.

Recommended mapping:

Test Notes
    →
Linked-Index Note Creator

Record the actual configuration used:

Template folder:
Templates
Target folder:
Test Notes
Invocation method:
[Folder Template / manual template invocation / other]
Additional Templater settings:
[settings]

⸻

7. Test Reset Procedure

Tests should begin from a known state unless the test explicitly requires data created by an earlier test.

To perform a full reset:

1. Delete all generated files from Test Notes/.
2. Leave the Templates/ folder unchanged.
3. Do not modify the baseline script.
4. Confirm Test Notes/ is empty.
5. Confirm Templater remains configured correctly.

Starting state:

Linked Index Test Vault/
├── Templates/
│   └── Linked-Index Note Creator.md
└── Test Notes/

Do not delete the Templater configuration between tests.

⸻

8. Result Definitions

Use the following results.

PASS
Expected behavior occurred.
FAIL
Behavior differs from the documented expectation.
BLOCKED
The test cannot be completed because another defect or environmental problem prevents it.
NOT TESTED
The test has not yet been executed.
NOT APPLICABLE
The test does not apply to the tested platform or configuration.

For failures, record:

* Actual behavior
* Error message
* Reproduction steps
* Platform
* Obsidian version
* Templater version
* Related GitHub issue

⸻

9. Smoke Test

Run this before the full baseline suite.

BT-001 -- Basic Document Creation

Purpose

Verify that the test environment can execute the baseline workflow.

Preconditions

Test Notes/ is empty.

Steps

1. Create a new note in Test Notes/.
2. Trigger the Linked-Index Note Creator.
3. Create a primary index named:

Alpha

4. Select:

✏ Add Title Here

5. Enter the note title:

First Test

6. Select:

Document

Expected result

The following files exist:

Test Notes/
├── _Test Notes _.md
├── Alpha _.md
└── Alpha - First Test.md

_Test Notes _.md contains:

- [[Alpha _]]

Alpha _.md contains:

- [[Alpha - First Test]]

No unexpected error or duplicate file is produced.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

10. Root Index Tests

BT-010 -- Root Index Creation

Purpose

Verify automatic creation of the root index.

Preconditions

Test Notes/ contains no root index.

Steps

1. Create a new note in Test Notes/.
2. Trigger the script.

Expected result

The script creates:

_Test Notes _.md

inside:

Test Notes/

The workflow continues to primary-index selection.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

BT-011 -- Existing Root Index Reuse

Purpose

Verify that an existing root index is reused.

Preconditions

The following already exists:

Test Notes/_Test Notes _.md

Steps

1. Create another new note.
2. Trigger the script.

Expected result

* No duplicate root index is created.
* Existing root-index content is preserved.
* Workflow proceeds normally.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

11. Primary Index Tests

BT-020 -- Create Primary Index

Steps

1. Start from the root selector.
2. Select:

📌 Link New Index

3. Enter:

Alpha

Expected result

The script creates:

Alpha _.md

The root index contains:

- [[Alpha _]]

The new primary index becomes the current navigation context.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

BT-021 -- Select Existing Primary Index

Preconditions

Create:

Alpha _.md
Beta _.md
Gamma _.md

and ensure all three are linked from the root index.

Steps

1. Start the script.
2. Select Beta.

Expected result

* Alpha, Beta, and Gamma appear as primary-index choices.
* Selecting Beta enters the Beta index.
* No files or links are modified merely by selecting it.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

12. Linked Index Tests

BT-030 -- Create Linked Index

Preconditions

Current index:

Alpha _.md

Steps

1. Select:

📌 Link New Index

2. Enter:

Testing

Expected result

The script creates:

Alpha - Testing _.md

Alpha _.md contains:

- [[Alpha - Testing _]]

The newly created index becomes the current navigation context.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

BT-031 -- Create Multiple Linked Levels

Steps

Create the following navigation path:

Alpha
→ Testing
→ Integration

Expected result

Files are created as:

Alpha _.md
Alpha - Testing _.md
Alpha - Testing - Integration _.md

Each index links only to the index created directly from it.

Expected relationships:

_Test Notes _
    ↓
Alpha _
    ↓
Alpha - Testing _
    ↓
Alpha - Testing - Integration _

The root index should not directly link Testing or Integration.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

BT-032 -- Ordinary Notes Excluded From Navigation

Preconditions

An index contains:

- [[Alpha - Architecture]]
- [[Alpha - Testing _]]
- [[Alpha - Planning]]
- [[Alpha - Security _]]

Steps

Navigate into Alpha.

Expected result

Navigation choices include:

Testing
Security

Ordinary notes such as:

Architecture
Planning

do not appear as linked-index choices.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

13. Breadcrumb Tests

BT-040 -- Breadcrumb Displays Navigation Path

Steps

Navigate:

Alpha
→ Testing
→ Integration

Expected result

The selector displays a breadcrumb equivalent to:

Alpha › Testing › Integration

The root index name is not included as user context.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

BT-041 -- Breadcrumb Updates After Navigation

Steps

1. Navigate to Alpha.
2. Navigate to Testing.
3. Navigate to Integration.

Expected result

The breadcrumb updates at each stage:

Alpha
Alpha › Testing
Alpha › Testing › Integration

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

14. Back Navigation Tests

BT-050 -- Back One Level

Preconditions

Current path:

Alpha › Testing › Integration

Steps

Select:

👈 Back

Expected result

The user returns to:

Alpha › Testing

No files or links are changed.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

BT-051 -- Back to Primary Selection

Steps

1. Navigate into Alpha.
2. Select Back.

Expected result

The script returns to the primary-index selector.

Existing primary indexes are displayed.

No index or note is created by navigating backward.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

15. Maximum Depth Tests

BT-060 -- Maximum Index Depth

Preconditions

Record the baseline value of:

MAX_INDEX_LEVELS

Expected baseline:

4

Steps

Navigate/create indexes until the maximum depth is reached.

Expected result

At maximum depth:

* The current index automatically becomes the target index.
* No additional linked-index selector is shown.
* The workflow proceeds to note-title entry.
* No index is created beyond the configured maximum.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

16. Add Title Here Tests

BT-070 -- Add Note at Primary Index

Steps

1. Select Alpha.
2. Select:

✏ Add Title Here

3. Create a Document titled:

Architecture

Expected result

The note is named:

Alpha - Architecture.md

and linked from:

Alpha _.md

No additional linked index is required.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

BT-071 -- Add Note at Deeper Index

Preconditions

Path:

Alpha → Testing

Steps

At Testing, select:

✏ Add Title Here

Create a Document titled:

Test Strategy

Expected result

The note is:

Alpha - Testing - Test Strategy.md

and is linked from:

Alpha - Testing _.md

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

17. Note Type Tests

BT-080 -- Document Naming

Inputs

Index:
Alpha - Testing
Title:
Architecture
Type:
Document

Expected filename

Alpha - Testing - Architecture.md

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

BT-081 -- Task Naming

Inputs

Index:
Alpha - Testing
Title:
Fix Pipeline
Type:
Task
Date:
2026-09-09

Expected filename

Alpha - Testing - (2026-09-09) Fix Pipeline.md

Expected behavior

A date prompt appears.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

BT-082 -- Tracker Naming

Inputs

Index:
Alpha - Testing
Title:
Deployment
Type:
Tracker
Date:
2026-09-09

Expected filename

Alpha - Testing - (2026-09-09) Deployment Tracker.md

Expected behavior

A date prompt appears.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

BT-083 -- Blog Naming

Inputs

Index:
Alpha - Testing
Title:
Pipeline Design
Type:
Blog

Expected filename

Using the current date as {date}:

Alpha - Testing - Pipeline Design ({date}).md

Expected behavior

The baseline implementation does not prompt for a Blog date. It uses the current date already assigned by the script.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

18. Date Validation Tests

BT-090 -- Valid Date

Input

2026-09-09

Expected result

Date is accepted.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

BT-091 -- Invalid Date Format

Test values such as:

09/09/2026
2026-9-9
September 9 2026

Expected result

Each value is rejected.

The user is prompted again.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

BT-092 -- Impossible Date

Test:

2026-02-31

Expected result

The date is rejected even though it matches YYYY-MM-DD.

The user is prompted again.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

19. Name Validation Tests

BT-100 -- Empty Name

Submit an empty or whitespace-only name.

Expected result

The script rejects it and prompts again.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

BT-101 -- Invalid Filename Characters

Test names containing:

\ / : * ? " < > |

Expected result

The script rejects the name and displays an explanatory notice.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

20. Duplicate Protection

BT-110 -- Duplicate Note Filename

Preconditions

Create:

Alpha - Architecture.md

Steps

Attempt to create another Document at Alpha with the title:

Architecture

Expected result

* Existing note is not overwritten.
* Duplicate note is not created.
* User receives a notice identifying the duplicate filename.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

21. Link Sorting Tests

BT-120 -- Alphabetical Link Sorting

Steps

Add links in this order:

Zulu
Alpha
Charlie
Bravo

Expected result

The index stores them as:

- [[Alpha]]
- [[Bravo]]
- [[Charlie]]
- [[Zulu]]

Sorting should be case-insensitive according to baseline behavior.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

BT-121 -- Numeric Link Sorting

Where practical, test:

Item 10
Item 2
Item 1

Expected result

Because the baseline uses numeric-aware localeCompare, expected ordering is:

Item 1
Item 2
Item 10

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

22. Mixed Index and Note Links

BT-130 -- Mixed Link List

Create an index containing links to both ordinary notes and linked indexes.

Example:

- [[Alpha - Architecture]]
- [[Alpha - Security _]]
- [[Alpha - Testing _]]
- [[Alpha - Weekly Review]]

Expected result

* All links remain in the file.
* Links remain alphabetically sorted.
* Only index files appear in navigation.
* Ordinary note links are preserved.
* No duplicate links are introduced.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

23. Existing Index Content Preservation

BT-140 -- Preserve Non-Link Content

Preconditions

Add content to an index:

# Alpha
This index contains Alpha-related work.
## Links
- [[Alpha - Existing Note]]

Steps

Use the script to add another note or linked index.

Expected result

* Heading remains.
* Description remains.
* Existing non-link content remains.
* New link is added.
* Link-list entries are alphabetically sorted.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

24. Cancellation Tests

BT-150 -- Cancel Name Prompt

Cancel while entering a note or index name.

Expected result

The workflow terminates without creating an invalid file.

Previously completed intentional index creation may remain if cancellation occurs later in the workflow.

Record actual baseline behavior.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

BT-151 -- Cancel Selector

Cancel an index or note-type selector.

Expected result

The workflow terminates without an unhandled error.

Record any files already created before cancellation.

Results

Desktop:
[NOT TESTED]
iPad:
[NOT TESTED]
Notes:

⸻

25. Platform Regression Summary

After executing the suite, summarize results.

Desktop

Total tests:
Passed:
Failed:
Blocked:
Not tested:
Overall status:
[PASS / FAIL / BLOCKED]
Known issues:
- 

iPad

Total tests:
Passed:
Failed:
Blocked:
Not tested:
Overall status:
[PASS / FAIL / BLOCKED]
Known issues:
- 

⸻

26. Baseline Failure Handling

Every failed baseline test must be classified before the baseline is finalized.

Use one of:

Must fix before baseline
Known limitation
Platform-specific limitation
Environment/configuration issue
Test-plan issue
Future enhancement
Out of scope

For a defect, create a GitHub issue containing:

Test ID:
Platform:
Obsidian version:
Templater version:
Baseline commit:
Expected:
Actual:
Steps to reproduce:
Error:
Classification:

Link the issue from this document where appropriate.

⸻

27. Baseline Completion Criteria

The baseline is considered verified when:

* Required desktop tests have been executed.
* Required iPad tests have been executed.
* Core note-creation workflow passes on supported platforms.
* Root-index behavior is verified.
* Linked-index navigation is verified.
* Breadcrumbs are verified.
* Back navigation is verified.
* Maximum-depth behavior is verified.
* All four default note types are verified.
* Duplicate protection is verified.
* Link sorting is verified.
* Mixed index/note links are verified.
* All failures are resolved or classified.
* README behavior descriptions match observed behavior.
* Exact tested commit is recorded.
* GitHub issues exist for known defects.
* No undocumented baseline behavior remains.

After these criteria are satisfied, create the baseline tag:

v0.1.0-baseline

and use that commit as the reference point for configuration and template extensibility work.

⸻

28. Baseline Checklist

Environment

* [ ]	Baseline commit recorded
* [ ]	Desktop environment recorded
* [ ]	iPad environment recorded
* [ ]	Test vault created
* [ ]	Baseline script copied unchanged
* [ ]	Templater configured
* [ ]	Reset procedure verified

Core

* [ ]	BT-001 Basic Document Creation
* [ ]	BT-010 Root Index Creation
* [ ]	BT-011 Existing Root Index Reuse
* [ ]	BT-020 Create Primary Index
* [ ]	BT-021 Select Existing Primary Index
* [ ]	BT-030 Create Linked Index
* [ ]	BT-031 Create Multiple Linked Levels
* [ ]	BT-032 Ordinary Notes Excluded From Navigation

Navigation

* [ ]	BT-040 Breadcrumb Displays Navigation Path
* [ ]	BT-041 Breadcrumb Updates After Navigation
* [ ]	BT-050 Back One Level
* [ ]	BT-051 Back to Primary Selection
* [ ]	BT-060 Maximum Index Depth
* [ ]	BT-070 Add Note at Primary Index
* [ ]	BT-071 Add Note at Deeper Index

Types and Naming

* [ ]	BT-080 Document Naming
* [ ]	BT-081 Task Naming
* [ ]	BT-082 Tracker Naming
* [ ]	BT-083 Blog Naming
* [ ]	BT-090 Valid Date
* [ ]	BT-091 Invalid Date Format
* [ ]	BT-092 Impossible Date
* [ ]	BT-100 Empty Name
* [ ]	BT-101 Invalid Filename Characters
* [ ]	BT-110 Duplicate Note Filename

Links and Content

* [ ]	BT-120 Alphabetical Link Sorting
* [ ]	BT-121 Numeric Link Sorting
* [ ]	BT-130 Mixed Link List
* [ ]	BT-140 Preserve Non-Link Content

Cancellation

* [ ]	BT-150 Cancel Name Prompt
* [ ]	BT-151 Cancel Selector

Completion

* [ ]	Desktop suite completed
* [ ]	iPad suite completed
* [ ]	Failures documented
* [ ]	Failures resolved or classified
* [ ]	README verified against observed behavior
* [ ]	Baseline tag created
* [ ]	Baseline GitHub release created