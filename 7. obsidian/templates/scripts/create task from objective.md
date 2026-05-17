<%*
const mainFolder = "4. tasks";
const activeFile = app.workspace.getActiveFile();
const month = tp.date.now("YYYY.MM");
const day = tp.date.now("MM.DD");

// detect objective automaticaly
// Since we run this FROM the objective note, we take its name
const selectedObjective = activeFile ? activeFile.basename : "";

// folder 
const monthPath = `${mainFolder}/${month}`;
if (!app.vault.getAbstractFileByPath(monthPath)) {
    await app.vault.createFolder(monthPath);
}

// task name
let taskName = await tp.system.prompt("Task name");
const fileName = `T${day}-${taskName || "untitled"}`;
const filePath = `${monthPath}/${fileName}.md`;

let taskMetadata = `---
status: work in progress
description: 
objective: "[[${selectedObjective}]]"
---
`;

// create file 
try {
    await app.vault.create(filePath, taskMetadata);
    new Notice(`✅ Task created in ${month}: ${fileName}`);
} catch (err) {
    new Notice("❌ Error: Task might already exist");
    console.error(err);
}
%>