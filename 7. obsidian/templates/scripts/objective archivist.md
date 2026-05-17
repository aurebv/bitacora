<%*
// 1. DATA FETCHING & VALIDATION
const activeFile = app.workspace.getActiveFile();
if (!activeFile) { new Notice("Error: No active file found"); return; }

const objName = activeFile.basename;
if (!objName.startsWith("O-")){ new Notice("Error: This is not an objective file"); return; }

const archiveObjPath = "8. archive/objectives";
const archiveTasksRoot = "8. archive/tasks";

// 2. CONFIRMATION
const confirm = await tp.system.suggester(["Yes, archive everything", "No, cancel"], [true, false], false, `Archive: ${objName}?`);
if (!confirm) return;

// 3. DATAVIEW API
const dv = app.plugins.plugins.dataview.api;
const linkedTasks = dv.pages('"4. tasks"').where(p => 
    p.objective && String(p.objective).includes(objName)
);

// 4. PREPARE NEW CONTENT (List of links)
let newContent = `## Tasks linked to this objective:\n`;
linkedTasks.forEach(t => {
    newContent += `- [[${t.file.name}]]\n`;
});
newContent += `\n**Archived on:** ${tp.date.now("YYYY-MM-DD")}`

// 5. PROCESS TASKS (Organize by Folder Date)
for (let taskPage of linkedTasks) {
    const taskFile = app.vault.getAbstractFileByPath(taskPage.file.path);
    if (!taskFile) continue;

    // Deduce YYYY.MM from current parent folder (e.g., "4. tasks/2024.05/task.md")
    // If not found, it defaults to current month
    const parentFolder = taskFile.parent.name;
    const folderDate = /^\d{4}\.\d{2}$/.test(parentFolder) ? parentFolder : tp.date.now("YYYY.MM");
    
    const destinationFolder = `${archiveTasksRoot}/${folderDate}`;

    // Create month folder if missing
    if (!app.vault.getAbstractFileByPath(destinationFolder)) {
        await app.vault.createFolder(destinationFolder);
    }

    // Move task
    try {
        await app.fileManager.renameFile(taskFile, `${destinationFolder}/${taskFile.name}`);
    } catch (e) { console.log("Task already moved or error:", e); }
}

// 6. MODIFY OBJECTIVE CONTENT
await app.vault.modify(activeFile, newContent);

// 7. RENAME AND MOVE OBJECTIVE
const newObjName = `${objName} (archived)`;
const finalObjPath = `${archiveObjPath}/${newObjName}.md`;

try {
    await app.fileManager.renameFile(activeFile, finalObjPath);
    new Notice(`✅ ${objName} archived successfully.`);
} catch (e) {
    new Notice("Error moving objective. Check if folder exists.");
}
%>