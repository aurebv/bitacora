<%*
const content = await tp.file.content;
// if the note comes with content, then we do nothing
if (content.trim().length > 0){
	return;
}

//const mainFolder = "4. tasks"; 
const currentFolder = tp.file.folder(true);
const objectivesFolder = "3. objectives";
const month = tp.date.now("YYYY.MM"); 
const day = tp.date.now("MM.DD");     

// retrieve existing objectives 
const files = app.vault.getMarkdownFiles().filter(f => f.path.startsWith(objectivesFolder)); 
const objectiveNames = files.map(f => f.basename);
const options = ["❌ No objective", ...objectiveNames];

//2. show selector 
let selectedObjective = await tp.system.suggester(options, options, false, "Select an objective (Esc to skip)");
if (selectedObjective === "❌ No objective" || !selectedObjective) { 
	selectedObjective = ""; 
} 

// path 
//const monthPath = `${mainFolder}/${month}`;
const monthPath = `${currentFolder}/${month}`
if (!app.vault.getAbstractFileByPath(monthPath)) {
    await app.vault.createFolder(monthPath);
}

// task name 
let taskName = await tp.system.prompt("task name");
const fileName = `T${day}-${taskName}`

// move and rename file 
try { 
	if (!taskName || taskName.trim() === "") throw new Error("empty file name"); 
	await tp.file.move(`${monthPath}/${fileName}`);
} catch (err) { 
	await tp.file.move(`${monthPath}/T${day}-untitled`); 
}

let taskMetadata = `---
status: work in progress
description:
objective: "[[${selectedObjective}]]"
---
`;

if (selectedObjective== ""){
	taskMetadata = `---
status: work in progress
description:
objective: 
---
`;
}
// write metadata
const tFile = tp.file.find_tfile(fileName); 
await app.vault.modify(tFile, taskMetadata); 
%>


