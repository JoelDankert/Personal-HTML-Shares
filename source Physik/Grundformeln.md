```dataviewjs
// dv.paragraph("DEBUG: Script execution starting..."); // Uncomment for basic execution check

const targetFolders = [
    "Commitments/Schule Oberstufe/Fächer/Physik (LK)/1. Elektrisches Feld",
    "Commitments/Schule Oberstufe/Fächer/Physik (LK)/2. Magnetisches Feld",
    "Commitments/Schule Oberstufe/Fächer/Physik (LK)/3. Induktion",
    "Commitments/Schule Oberstufe/Fächer/Physik (LK)/4. Mechanische Schwingungen und Wellen",
    "Commitments/Schule Oberstufe/Fächer/Physik (LK)/5. Elektromagnetische Schwingungen und Wellen",
   "Commitments/Schule Oberstufe/Fächer/Physik (LK)/6. Atom- und Quantenphysik"
];

const targetExpression = "\\quad\\textcolor{red}{\\text{GF}}";
let itemsByFolder = {}; // Key: folderName, Value: array of extracted items
let outputGenerated = false; // Flag to track if any dv.paragraph was called by our print()

// Wrapper for dv.paragraph to track output
function print(message) {
    dv.paragraph(message);
    outputGenerated = true;
}

// Function to print folder headings using Markdown via dv.paragraph
function printFolderHeader(folderName) {
    // Using the print() function which calls dv.paragraph()
    // dv.paragraph() can render Markdown, so this will create an H3 heading
    print(`### ${folderName}`);
}

async function run() {
    // print("DEBUG: Async function 'run' started.");

    for (const targetFolder of targetFolders) {
        const folderName = targetFolder.split('/').pop() || targetFolder;
        // print(`DEBUG: Processing folder: ${folderName}`);

        const notesInFolder = dv.pages(`"${targetFolder}"`)
            .filter(p => p.file && typeof p.file.path === 'string' && p.file.path.endsWith(".md"));

        let linesFoundInThisFolderCount = 0;
        let notesProcessedInThisFolderCount = 0;
        let folderHadNotes = notesInFolder.length > 0;

        if (!folderHadNotes) {
            print(`Ordner "${folderName}": Keine Notizen gefunden.`);
        } else {
            if (!itemsByFolder[folderName]) {
                itemsByFolder[folderName] = [];
            }

            for (let note of notesInFolder) {
                notesProcessedInThisFolderCount++;
                const filePath = note.file.path;

                try {
                    const file = app.vault.getAbstractFileByPath(filePath);

                    if (file) {
                        const fileContent = await app.vault.read(file);
                        const lines = fileContent.split("\n");

                        for (let line of lines) {
                            if (line.includes(targetExpression)) {
                                let beforeExpression = line.split(targetExpression)[0].trim();
                                beforeExpression = beforeExpression
                                    .replace(/\$/g, "")
                                    .replace(/&/g, "")
                                    .replace(/\\huge/g, "");

                                if (beforeExpression) {
                                    const formattedLine = `$\\huge ${beforeExpression}$`;
                                    itemsByFolder[folderName].push({
                                        filename: note.file.name,
                                        originalPath: filePath,
                                        extractedLine: formattedLine
                                    });
                                    linesFoundInThisFolderCount++;
                                }
                            }
                        }
                    } else {
                        print(`Skipping invalid file: ${filePath}`);
                    }
                } catch (error) {
                    print(`Error reading file: "${note.file?.name || filePath}" (Ordner: ${folderName}). ${error.message}`);
                }
            }

            if (folderHadNotes && linesFoundInThisFolderCount === 0 && notesProcessedInThisFolderCount > 0) {
                print(`Ordner "${folderName}": Keine Zeilen mit dem gesuchten Ausdruck gefunden (in ${notesProcessedInThisFolderCount} Notiz(en) durchsucht).`);
            }
        }
    }

    let actualFormulasDisplayed = false;
    for (const targetFolderPath of targetFolders) {
        const folderName = targetFolderPath.split('/').pop() || targetFolderPath;
        const itemsInThisFolder = itemsByFolder[folderName];

        if (itemsInThisFolder && itemsInThisFolder.length > 0) {
            printFolderHeader(folderName); // This will now use print("### " + folderName)
            
            itemsInThisFolder.sort((a, b) => {
                const filenameCompare = a.filename.localeCompare(b.filename);
                if (filenameCompare !== 0) return filenameCompare;
                return a.originalPath.localeCompare(b.originalPath);
            });
            
            itemsInThisFolder.forEach(item => {
                print(item.extractedLine);
            });
            actualFormulasDisplayed = true;
        }
    }
    
    if (targetFolders.length === 0 && !outputGenerated) {
        print("Keine Ordner zum Durchsuchen konfiguriert. Bitte 'targetFolders' im Skript anpassen.");
    } else if (targetFolders.length > 0 && !outputGenerated && !actualFormulasDisplayed) {
        print("Keine Inhalte gefunden oder Ausgaben generiert. Überprüfen Sie die Ordnerpfade, Dateiinhalte und die Entwicklerkonsole (Strg+Shift+I) auf mögliche Fehler.");
    }
    
    // print("DEBUG: Script execution finished.");
}

run().catch(e => {
    const errorMessage = "Ein schwerwiegender Fehler ist im Skript aufgetreten: " + e.message;
    if (typeof dv !== 'undefined' && dv.paragraph) {
        dv.paragraph(errorMessage);
    }
    console.error("Fatal script error:", e);
    if (typeof outputGenerated !== 'undefined') outputGenerated = true;
});

```


