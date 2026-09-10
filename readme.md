Linked-Index Note Creator

Linked-Index Note Creator is an Obsidian Templater workflow for creating and navigating linked note indexes while keeping logical organization independent from the vault’s folder structure.

Instead of requiring folders to represent relationships between notes, the script uses ordinary Obsidian notes and wiki links to create navigable indexes. Indexes are notes themselves, allowing them to hold context in addition to organizing other indexes and notes.

The project is currently in baseline development. The existing single-script implementation is being preserved and tested before configuration and template extensibility are introduced.

Features

The current version supports:

* Automatic root-index creation
* Primary-index creation and selection
* Navigation through linked indexes
* Breadcrumb navigation
* Back navigation
* Configurable maximum index depth within the script
* Creating notes at different index levels
* Creating additional linked indexes during navigation
* Automatic hierarchical filename construction
* Blog, Document, Task, and Tracker note types
* Date validation for date-dependent note types
* Duplicate filename protection
* Automatic linking of new notes to their selected index
* Alphabetical sorting of index links
* Mixed note and index links within the same index
* Filtering navigation so ordinary notes do not appear as index choices

Requirements

The current implementation requires:

* Obsidian
* Templater
* A configured Templater template folder

The script is implemented as a Templater template and relies on the Templater API and Obsidian vault API.

Platform Support

The current workflow is intended to support:

* Obsidian Desktop
* Obsidian on iPadOS

iPad compatibility is an explicit project requirement. Changes should not depend on Templater functionality that is unavailable on mobile.

Formal baseline testing for both platforms is planned before major refactoring begins.

Installation

Installation instructions will be finalized as part of baseline testing.

At a high level:

1. Install Obsidian.
2. Install and enable the Templater community plugin.
3. Configure a Templater template folder.
4. Copy the Linked-Index Note Creator .md file into the configured template folder.
5. Configure the intended folder/template relationship in Templater if using Folder Templates.
6. Create a new note in the target folder.
7. Run the Linked-Index Note Creator template.

The script determines its working root from the folder containing the target note:

const ROOT = tp.file.folder(false);

As a result, the same template can operate against different target folders.

How It Works

The workflow uses specially named Markdown files as indexes.

Index files are identified by a trailing:

 _

For example:

Engineering _.md
Engineering - CI-CD _.md

Ordinary notes do not use the trailing index suffix:

Engineering - CI-CD - Pipeline Testing.md

Root Index

Each target folder has a root index derived from the folder name.

For a folder named:

Projects

the root index is:

_Projects _.md

The root index links to the primary indexes available within that context.

Linked Indexes

Indexes can link to additional indexes.

For example:

_Projects _.md
└── Engineering _.md
    └── Engineering - CI-CD _.md
        └── Engineering - CI-CD - Testing _.md

The accumulated index name is carried into deeper index filenames.

This provides context directly in filenames while allowing all of the files to remain in the same physical folder.

Notes

A note can be created at an index before reaching the maximum index depth.

For example:

Engineering _.md
├── Engineering - Architecture.md
│
└── Engineering - CI-CD _.md
    ├── Engineering - CI-CD - Pipeline Design.md
    │
    └── Engineering - CI-CD - Testing _.md
        └── Engineering - CI-CD - Testing - Integration.md

This allows an index to act as both an organizational structure and a contextual note.

Navigation

The script begins at the root index and allows the user to navigate through linked indexes.

A typical workflow is:

1. Select or create a primary index.
2. Select an existing linked index, create another linked index, or create the note at the current index.
3. Continue navigating until the desired context is reached.
4. Enter the note title and type.
5. Allow the script to rename and link the note.

Breadcrumbs

While navigating, the current path is represented as a breadcrumb.

For example:

Engineering › CI-CD › Testing

The breadcrumb represents navigation context without requiring the user to interpret the full accumulated index filename.

Back Navigation

When navigation has not reached the configured maximum depth, the user can select:

👈 Back

to return to the previous index.

The navigation state is preserved while moving backward.

Add Title Here

At a non-root index, the user can select:

✏ Add Title Here

This stops index navigation and selects the current index as the destination for the new note.

Link New Index

The user can select:

📌 Link New Index

to create another index.

The new index is:

1. Named using the current index as its prefix.
2. Created as an Obsidian Markdown file.
3. Linked from the current index.
4. Entered as the next navigation context.

For example, creating Testing from:

Engineering - CI-CD _.md

creates:

Engineering - CI-CD - Testing _.md

Maximum Index Depth

The current maximum depth is configured directly in the script:

const MAX_INDEX_LEVELS = 4;

The root index is level 0, and the primary index is level 1.

When the configured maximum depth is reached, the current index automatically becomes the target for the note rather than offering another level of navigation.

Root-index configuration of this value is planned but is not implemented in the baseline version.

Note Types

The current version supports four hard-coded note types.

Document

Format:

{index} - {title}.md

Example:

Engineering - CI-CD - Pipeline Architecture.md

Task

Task requires a date.

Format:

{index} - ({date}) {title}.md

Example:

Engineering - CI-CD - (2026-09-09) Fix Pipeline.md

Tracker

Tracker requires a date.

Format:

{index} - ({date}) {title} Tracker.md

Example:

Engineering - CI-CD - (2026-09-09) Deployment Tracker.md

Blog

Format:

{index} - {title} ({date}).md

The current implementation uses the current date when constructing Blog filenames but does not prompt the user for a date.

Example:

Engineering - CI-CD - Pipeline Design (2026-09-09).md

Index Contents

Index files can contain both links to other indexes and links to ordinary notes.

For example:

- [[Engineering - Architecture]]
- [[Engineering - CI-CD _]]
- [[Engineering - Planning]]
- [[Engineering - Security _]]

When a new link is added, the script sorts simple wiki-link list entries alphabetically.

Existing non-link content in an index is preserved.

This means an index can eventually contain additional context such as headings, descriptions, metadata, or other Markdown content while still functioning as a navigational index.

Index Navigation vs. Note Links

Only index files appear in navigation selectors.

The script identifies an index by its trailing:

 _

Therefore an index containing:

- [[Engineering - Architecture]]
- [[Engineering - CI-CD _]]
- [[Engineering - Planning]]
- [[Engineering - Security _]]

presents only the following as navigation choices:

CI-CD
Security

Ordinary notes remain linked from the index but are not treated as navigational indexes.

Duplicate Protection

Before renaming the target note, the script checks whether another file already exists with the generated filename.

If a duplicate exists:

* The existing note is not overwritten.
* The current note is not renamed to the duplicate filename.
* The user receives an Obsidian notice.

File Creation

New index files are initially staged through the configured Templater template folder before being moved into the target root folder.

This prevents newly created index files from unintentionally retriggering the same folder-template workflow.

Current Limitations

The baseline implementation intentionally has several limitations that are planned for later development.

Configuration is hard-coded

Values such as maximum index depth, labels, and note types currently live directly in the JavaScript.

Root-index configuration is planned but is not yet implemented.

Note types are hard-coded

The available types are currently:

Blog
Document
Task
Tracker

Users cannot yet define new types without editing the script.

Title rules are hard-coded

Filename construction is currently implemented directly in the script.

Future versions are planned to support configurable title formats.

Type-specific templates are not implemented

Selecting Blog, Document, Task, or Tracker currently affects filename behavior but does not apply a corresponding user-editable Templater template.

Future development is planned to allow each note type to reference an editable template.

Template bootstrapping is not implemented

The project does not currently create missing type templates automatically.

This is planned for a later milestone.

A note has one target index during creation

The current workflow selects one target index for the newly created note.

Supporting additional contextual index relationships may be considered in future development.

Testing is currently manual

A repeatable baseline test suite is being developed.

Desktop and iPad behavior will be recorded separately.

Baseline Development

The current working implementation is being preserved before configuration and template functionality are introduced.

The baseline test suite will cover:

* Root-index creation
* Primary-index creation
* Primary-index selection
* Linked-index creation
* Linked-index navigation
* Breadcrumbs
* Back navigation
* Maximum index depth
* Add Title Here
* Blog creation
* Document creation
* Task creation
* Tracker creation
* Date validation and naming
* Duplicate protection
* Alphabetical link sorting
* Mixed index and note links
* Desktop operation
* iPad operation

A verified baseline release will be tagged:

v0.1.0-baseline

Development Roadmap

Development is planned in six major stages.

1. Baseline and Project Setup

Establish the GitHub project, documentation, repeatable testing process, and verified baseline.

2. Configuration Foundation

Introduce default configuration and optional root-index YAML overrides.

3. Generalized Note Types

Convert note types into configurable definitions containing behavior such as date requirements and filename formats.

4. Template Infrastructure

Associate note types with editable Templater templates and bootstrap missing templates without overwriting user changes.

5. Template Application and User-Defined Types

Apply selected templates and allow root-index configuration to define new note types without modifying JavaScript.

6. Regression and Release

Verify baseline compatibility on desktop and iPad, update documentation, and prepare the generalized release.

Design Principles

Development of Linked-Index Note Creator follows several principles.

Obsidian-native structure

The resulting organization should remain understandable without the script.

Indexes and relationships use ordinary Markdown files and Obsidian wiki links rather than a proprietary database.

Logical organization independent of folders

Folders determine where files are physically stored.

Linked indexes determine the logical context in which notes belong.

Indexes are notes

An index should be capable of containing context, metadata, descriptions, or other content rather than functioning only as a container.

User-owned templates

Future generated type templates should become ordinary editable Templater files.

Once created, the script should not overwrite user modifications.

Mobile compatibility

iPad compatibility is a project requirement.

Features that work only through desktop-specific Templater functionality should not become required parts of the core workflow.

Backward compatibility

Configuration and template extensibility should preserve the behavior of the verified baseline unless a behavior change is deliberate and documented.

Troubleshooting

Templater does not run

Verify that:

* Templater is installed and enabled.
* The template is inside the configured Templater template folder.
* Any Folder Template configuration points to the intended template.

Root index is not created

Verify that:

* The target note is in the intended folder.
* Templater has permission to create files.
* The configured template folder exists.

A linked index does not appear

Verify that:

* The file is actually linked from the current index.
* Its filename ends with  _.
* The wiki link resolves to an existing Obsidian file.

A note does not appear in index navigation

This is expected.

Ordinary notes remain linked in indexes but are intentionally excluded from index-navigation selectors.

A duplicate note is rejected

This is intentional duplicate protection.

Choose another title or resolve the existing note before running the workflow again.

Desktop and iPad behavior differ

Record the:

* Obsidian version
* Templater version
* Operating system
* Device
* Steps to reproduce
* Error message

Platform-specific problems should be reported as project issues once issue tracking is established.

Contributing

The project is currently establishing its baseline architecture and testing process.

Bug reports, platform compatibility findings, documentation improvements, and design feedback are useful during this stage.

Contribution guidelines may be expanded as the project matures.

License

A project license has not yet been finalized.

The selected license will be documented here and included in the repository before the first formal release.