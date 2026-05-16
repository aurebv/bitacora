<%*
// 1. CONFIGURATION & PATHS
const mainFolder = "4. tasks";
const activeFile = app.workspace.getActiveFile();
const month = tp.date.now("YYYY.MM");
const day = tp.date.now("MM.DD");

// 2. DETECT OBJECTIVE AUTOMATICALLY
// Since we run this FROM the objective note, we take its name
const selectedObjective = activeFile ? activeFile.basename : "";

// 3. PREPARE FOLDER (YYYY.MM)
const monthPath = `${mainFolder}/${month}`;
if (!app.vault.getAbstractFileByPath(monthPath)) {
    await app.vault.createFolder(monthPath);
}

// 4. TASK NAME PROMPT
let taskName = await tp.system.prompt("Task name");
const fileName = `T${day}-${taskName || "untitled"}`;
const filePath = `${monthPath}/${fileName}.md`;

// 5. BUILD YAML (Using your style)
let taskMetadata = `---
status: work in progress
description: 
objective: "[[${selectedObjective}]]"
---
`;

// 6. CREATE FILE 
try {
    await app.vault.create(filePath, taskMetadata);
    new Notice(`✅ Task created in ${month}: ${fileName}`);
} catch (err) {
    new Notice("❌ Error: Task might already exist");
    console.error(err);
}
%>