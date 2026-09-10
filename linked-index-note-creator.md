<%*
/** ============================================================
 * Linked-index Note Creation for Obsidian - Templater 
 * ============================================================
 * About: A Templater workflow for creating and navigating linked note indexes while keeping logical organization independent from folders. Navigation and creation in one workflow. This script creates navigable note relationships without requiring the vault’s folder structure to encode those relationships.  Essentially, automation for constructing an Obsidian-native information structure, rather than creating a proprietary database. It automatically creates index’s linking notes to concepts. Indexes are notes themselves allowing for context not merely organization. Notes can be added at any level for carried context.
 * Index files use trailing " _" for identification/sort position.
 *
 * Expected structure:
 *
 * templateRoot/
 * ├── _rootFolderName _.md
 * ├── Primary \_.md
 * ├── Primary - Linked \_.md
 * ├── Primary - Linked - Linked \_.md
 * └── Primary - Linked - Title.md
 *
 * Index files may contain:
 * - links to other indexes
 * - links to notes
 *
 * Only index files appear during index navigation.
 */

// ============================================================
// CONFIG
// ============================================================

const ROOT = tp.file.folder(false);

const ROOT_NAME =
    ROOT.split("/").filter(Boolean).pop();

const ROOT_INDEX_NAME =
    `_${ROOT_NAME} _`;

const TEMPLATE_FOLDER =
    app.plugins.plugins["templater-obsidian"]
        .settings.templates_folder;

const INDEX_SUFFIX = " _";

// Root is level 0.
// Primary index is level 1.
const MAX_INDEX_LEVELS = 4;

const ADD_NEW_INDEX = "📌 Link New Index";
const ADD_TITLE_HERE = "✏ Add Title Here";
const GO_BACK = "👈 Back";

const TYPES = [
    "Blog",
    "Document",
    "Task",
    "Tracker"
];


// ============================================================
// INPUT
// ============================================================

const Input = {

    async name(message) {
        while (true) {
            const value =
                await tp.system.prompt(message);

            if (value === null) {
                return null;
            }

            const cleaned =
                value.trim();

            if (!cleaned) {
                new Notice(
                    "Name cannot be empty."
                );

                continue;
            }

            if (/[\\/:*?"<>|]/.test(cleaned)) {
                new Notice(
                    'Name cannot contain: \\ / : * ? " < > |'
                );

                continue;
            }

            return cleaned;
        }
    },


    async date(defaultDate) {
        while (true) {
            const value =
                await tp.system.prompt(
                    "Enter date (YYYY-MM-DD):",
                    defaultDate
                );

            if (value === null) {
                return null;
            }

            const cleaned =
                value.trim();

            let valid = false;

            if (/^\d{4}-\d{2}-\d{2}$/.test(cleaned)) {
                const [year, month, day] =
                    cleaned
                        .split("-")
                        .map(Number);

                const date =
                    new Date(
                        year,
                        month - 1,
                        day
                    );

                valid =
                    date.getFullYear() === year &&
                    date.getMonth() === month - 1 &&
                    date.getDate() === day;
            }

            if (!valid) {
                new Notice(
                    "Invalid date. Use YYYY-MM-DD."
                );

                continue;
            }

            return cleaned;
        }
    }
};


// ============================================================
// INDEX MODULE
// ============================================================

const Index = (() => {

    // --------------------------------------------------------
    // Internal implementation
    // --------------------------------------------------------

    function extractWikiLinks(content) {
        const links = [];

        const regex =
            /\[\[([^\]|#]+)(?:#[^\]|]*)?(?:\|[^\]]+)?\]\]/g;

        let match;

        while (
            (match = regex.exec(content)) !== null
        ) {
            const path =
                match[1].trim();

            if (path) {
                links.push(path);
            }
        }

        return links;
    }


    // --------------------------------------------------------
    // Public: locate an Obsidian file
    // --------------------------------------------------------

    function getFile(pathWithoutExtension) {
        const path =
            pathWithoutExtension.endsWith(".md")
                ? pathWithoutExtension
                : `${pathWithoutExtension}.md`;

        const file =
            tp.app.vault
                .getAbstractFileByPath(path);

        return file instanceof tp.obsidian.TFile
            ? file
            : null;
    }


    // --------------------------------------------------------
    // Public: logical index name without trailing " _"
    // --------------------------------------------------------

    function getName(file) {
        return file.basename
            .replace(/\s+_$/, "")
            .trim();
    }


    // --------------------------------------------------------
    // Public: linked index files only
    //
    // Ordinary note links are intentionally ignored.
    // Link order is preserved from the index file.
    // --------------------------------------------------------

    async function getLinkedIndexes(indexFile) {
        const content =
            await tp.app.vault.cachedRead(
                indexFile
            );

        const paths =
            extractWikiLinks(content);

        const files = [];
        const seenPaths = new Set();

        for (const path of paths) {
            const file =
                tp.app.metadataCache
                    .getFirstLinkpathDest(
                        path,
                        indexFile.path
                    );

            if (
                !(file instanceof tp.obsidian.TFile)
            ) {
                continue;
            }

            if (
                !file.basename.endsWith(
                    INDEX_SUFFIX
                )
            ) {
                continue;
            }

            if (seenPaths.has(file.path)) {
                continue;
            }

            seenPaths.add(file.path);
            files.push(file);
        }

        return files;
    }


    // --------------------------------------------------------
    // Public: add a sorted link to an index
    //
    // Existing non-link content is preserved.
    // All simple "- [[...]]" link lines are kept together
    // and sorted alphabetically.
    // --------------------------------------------------------

    async function addLink(
        indexFile,
        linkedFile
    ) {
        await tp.app.vault.process(
            indexFile,
            content => {

                // --------------------------------------------
                // Do not add an existing link twice.
                // --------------------------------------------

                const existingLinks =
                    extractWikiLinks(content);

                const linkedPath =
                    linkedFile.path
                        .replace(/\.md$/, "");

                const alreadyLinked =
                    existingLinks.some(link => {
                        const cleaned =
                            link
                                .replace(/\.md$/, "")
                                .trim();

                        return (
                            cleaned === linkedFile.basename ||
                            cleaned === linkedPath
                        );
                    });

                if (alreadyLinked) {
                    return content;
                }


                // --------------------------------------------
                // Separate link-list lines from other content.
                // --------------------------------------------

                const newLink =
                    `- [[${linkedFile.basename}]]`;

                const linkLines = [];
                const otherLines = [];

                let firstLinkPosition = null;

                for (
                    const line of content.split("\n")
                ) {
                    const isLinkLine =
                        /^\s*-\s+\[\[[^\]]+\]\]\s*$/
                            .test(line);

                    if (isLinkLine) {
                        if (
                            firstLinkPosition === null
                        ) {
                            firstLinkPosition =
                                otherLines.length;
                        }

                        linkLines.push(line);
                    } else {
                        otherLines.push(line);
                    }
                }


                // --------------------------------------------
                // Add and sort links.
                // --------------------------------------------

                linkLines.push(newLink);

                linkLines.sort((a, b) => {
                    const aPath =
                        extractWikiLinks(a)[0] ?? a;

                    const bPath =
                        extractWikiLinks(b)[0] ?? b;

                    const aName =
                        (aPath.split("/").pop() ?? aPath)
                            .toLowerCase();

                    const bName =
                        (bPath.split("/").pop() ?? bPath)
                            .toLowerCase();

                    return aName.localeCompare(
                        bName,
                        undefined,
                        {
                            sensitivity: "base",
                            numeric: true
                        }
                    );
                });


                // --------------------------------------------
                // Put sorted links back into the document.
                // --------------------------------------------

                if (firstLinkPosition === null) {

                    // Remove trailing blank lines.
                    while (
                        otherLines.length &&
                        !otherLines[
                            otherLines.length - 1
                        ].trim()
                    ) {
                        otherLines.pop();
                    }

                    // Separate existing content from links
                    // with one blank line.
                    if (
                        otherLines.some(
                            line => line.trim()
                        )
                    ) {
                        otherLines.push("");
                    }

                    otherLines.push(
                        ...linkLines
                    );

                } else {

                    otherLines.splice(
                        firstLinkPosition,
                        0,
                        ...linkLines
                    );
                }


                return (
                    otherLines
                        .join("\n")
                        .replace(/\n*$/, "") +
                    "\n"
                );
            }
        );
    }


    // --------------------------------------------------------
    // Public: physically create an index file
    // --------------------------------------------------------

    async function createFile(filename) {
        const destinationPath =
            `${ROOT}/${filename}`;

        const existing =
            getFile(destinationPath);

        if (existing) {
            return existing;
        }

        try {
            // Stage creation in the template folder
            // to prevent template re-triggering.
            const createdFile =
                await tp.file.create_new(
                    "",
                    filename,
                    false,
                    TEMPLATE_FOLDER
                );

            if (!createdFile) {
                new Notice(
                    `Failed to create: ${filename}`
                );

                return null;
            }

            await tp.file.move(
                destinationPath,
                createdFile
            );

            const movedFile =
                getFile(destinationPath);

            if (!movedFile) {
                new Notice(
                    `Created "${filename}", but could not locate it after moving.`
                );

                return null;
            }

            return movedFile;

        } catch (error) {
            console.error(
                `Failed to create index "${filename}":`,
                error
            );

            new Notice(
                `Could not create index: ${filename}`
            );

            return null;
        }
    }


    // --------------------------------------------------------
    // Public: create an index and link it from another index
    // --------------------------------------------------------

    async function createEntry({
        sourceIndexFile,
        prompt,
        filenamePrefix = ""
    }) {
        if (
            !(
                sourceIndexFile instanceof
                tp.obsidian.TFile
            )
        ) {
            throw new Error(
                "createEntry requires a valid sourceIndexFile."
            );
        }

        const entryName =
            await Input.name(prompt);

        if (!entryName) {
            return null;
        }

        const filename =
            filenamePrefix
                ? `${filenamePrefix} - ${entryName}${INDEX_SUFFIX}`
                : `${entryName}${INDEX_SUFFIX}`;

        const newFile =
            await createFile(filename);

        if (!newFile) {
            return null;
        }

        await addLink(
            sourceIndexFile,
            newFile
        );

        return newFile;
    }


    // --------------------------------------------------------
    // Public API
    // --------------------------------------------------------

    return {
        getFile,
        getName,
        getLinkedIndexes,
        addLink,
        createFile,
        createEntry
    };

})();


/// ============================================================
// INDEX NAVIGATION
// ============================================================

async function selectTargetIndex(path) {

    // --------------------------------------------------------
    // Current navigation state
    // --------------------------------------------------------

    const current =
        path[path.length - 1];

    const currentIndexFile =
        current.file;

    const currentLevel =
        path.length - 1;

    const isRoot =
        currentLevel === 0;


    // Maximum configured depth reached.
    // Current index automatically becomes the target.
    if (
        !isRoot &&
        currentLevel >= MAX_INDEX_LEVELS
    ) {
        return currentIndexFile;
    }


    const currentIndexName =
        Index.getName(currentIndexFile);

    const linkedIndexes =
        await Index.getLinkedIndexes(
            currentIndexFile
        );


    // --------------------------------------------------------
    // Breadcrumb
    //
    // Root is excluded because it is the system index,
    // rather than part of the user's selected context.
    // --------------------------------------------------------

    const breadcrumb =
        path
            .slice(1)
            .map(item => item.label)
            .join(" › ");


    // --------------------------------------------------------
    // Build selector options
    // --------------------------------------------------------

    const options = [];
    
    for (const file of linkedIndexes) {
        const fullName =
            Index.getName(file);

        let label =
            fullName;

        if (!isRoot) {
            const prefix =
                `${currentIndexName} - `;

            if (
                fullName.startsWith(prefix)
            ) {
                label =
                    fullName
                        .substring(prefix.length)
                        .trim();
            }
        }

        options.push({
            action: "select",
            file,
            label
        });
    }
	if (!isRoot) {
		options.push({
			action: "back",
			label: GO_BACK
		});
		options.push({
			action: "title",
			label: ADD_TITLE_HERE
		});
	}

	options.push({
		action: "create",
		label: ADD_NEW_INDEX
	});


    // --------------------------------------------------------
    // Prompt
    // --------------------------------------------------------

    let prompt;

    if (isRoot) {

        prompt =
            "Select or create primary index:";

    } else if (linkedIndexes.length === 0) {

        prompt =
            `${breadcrumb} — No linked indexes:`;

    } else {

        prompt =
            `${breadcrumb} — Select linked index:`;
    }


    const selected =
        await tp.system.suggester(
            option => option.label,
            options,
            false,
            prompt
        );


    if (!selected) {
        return null;
    }


    // --------------------------------------------------------
    // Navigate back one index
    // --------------------------------------------------------

    if (selected.action === "back") {
        return await selectTargetIndex(
            path.slice(0, -1)
        );
    }


    // --------------------------------------------------------
    // Stop navigation and use current index.
    // --------------------------------------------------------

    if (selected.action === "title") {
        return currentIndexFile;
    }


    // --------------------------------------------------------
    // Create and navigate into a new linked index.
    // --------------------------------------------------------

    if (selected.action === "create") {
        const linkedIndex =
            await Index.createEntry({
                sourceIndexFile:
                    currentIndexFile,

                prompt: isRoot
                    ? "New primary index:"
                    : `New index for ${currentIndexName}:`,

                filenamePrefix:
                    isRoot
                        ? ""
                        : currentIndexName
            });


        if (!linkedIndex) {
            return null;
        }


        const fullName =
            Index.getName(linkedIndex);

        let label =
            fullName;


        // Use only the newly-added portion
        // for the breadcrumb below root.
        if (!isRoot) {
            const prefix =
                `${currentIndexName} - `;

            if (
                fullName.startsWith(prefix)
            ) {
                label =
                    fullName
                        .substring(prefix.length)
                        .trim();
            }
        }


        return await selectTargetIndex([
            ...path,
            {
                file: linkedIndex,
                label
            }
        ]);
    }


    // --------------------------------------------------------
    // Navigate into an existing linked index.
    // --------------------------------------------------------

    return await selectTargetIndex([
        ...path,
        {
            file: selected.file,
            label: selected.label
        }
    ]);
}


// ============================================================
// MAIN
// ============================================================


// ------------------------------------------------------------
// 1. Find or create root index
// ------------------------------------------------------------

let rootIndexFile =
    Index.getFile(
        `${ROOT}/${ROOT_INDEX_NAME}`
    );

if (!rootIndexFile) {
    rootIndexFile =
        await Index.createFile(
            ROOT_INDEX_NAME
        );

    if (!rootIndexFile) {
        new Notice(
            `Could not create root index: ${ROOT_INDEX_NAME}`
        );

        return;
    }
}


// ------------------------------------------------------------
// 2. Select target index
// ------------------------------------------------------------

const targetIndex = await selectTargetIndex([
	{
		file: rootIndexFile,
		label: ROOT_NAME
	}
]);

if (!targetIndex) {
    return;
}

const indexName =
    Index.getName(targetIndex);


// ------------------------------------------------------------
// 3. Gather note information
// ------------------------------------------------------------

const rawTitle =
    await Input.name(
        "Enter the note title:"
    );

if (!rawTitle) {
    return;
}


const type =
    await tp.system.suggester(
        TYPES,
        TYPES,
        false,
        "Select Type"
    );

if (!type) {
    return;
}


let due =
    tp.date.now("YYYY-MM-DD");

if (
    type === "Task" ||
    type === "Tracker"
) {
    due =
        await Input.date(due);

    if (!due) {
        return;
    }
}


// ------------------------------------------------------------
// 4. Build final note title
// ------------------------------------------------------------

const prefix =
    `${indexName} - `;

let title;

switch (type) {
    case "Task":
        title =
            `${prefix}(${due}) ${rawTitle}`;
        break;

    case "Tracker":
        title =
            `${prefix}(${due}) ${rawTitle} Tracker`;
        break;

    case "Blog":
        title =
            `${prefix}${rawTitle} (${due})`;
        break;

    case "Document":
    default:
        title =
            `${prefix}${rawTitle}`;
        break;
}


// ------------------------------------------------------------
// 5. Validate and rename current note
// ------------------------------------------------------------

const currentFile =
    tp.config.target_file;

if (!currentFile) {
    new Notice(
        "Could not determine current note."
    );

    return;
}


const folderPath =
    currentFile.parent?.path ?? "";

const newPath =
    folderPath
        ? `${folderPath}/${title}.md`
        : `${title}.md`;


const existingFile =
    tp.app.vault
        .getAbstractFileByPath(newPath);

if (
    existingFile instanceof tp.obsidian.TFile &&
    existingFile.path !== currentFile.path
) {
    new Notice(
        `A note named "${title}" already exists.`
    );

    return;
}


await tp.app.fileManager.renameFile(
    currentFile,
    newPath
);


// ------------------------------------------------------------
// 6. Link renamed note to target index
// ------------------------------------------------------------

await Index.addLink(
    targetIndex,
    currentFile
);

new Notice(
    `Added "${currentFile.basename}" to "${targetIndex.basename}".`
);

_%>