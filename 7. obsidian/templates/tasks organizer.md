<%*
const mainFolder = "4. tasks"; 
const objectivesFolder = "3. objectives";
const month = tp.date.now("YYYY.MM"); 
const day = tp.date.now("MM.DD");     

// 1. OBTENER OBJETIVOS EXISTENTES 
const files = app.vault.getMarkdownFiles().filter(f => f.path.startsWith(objectivesFolder)); 
const objectiveNames = files.map(f => f.basename);
const options = ["❌ Sin objetivo", ...objectiveNames];

//2. mostarar selector 
let selectedObjective = await tp.system.suggester(options, options, false, "Selecciona el Objetivo (Esc para saltar)");
if (selectedObjective === "❌ Sin objetivo" || !selectedObjective) { 
	selectedObjective = ""; 
} 

// path 
const monthPath = `${mainFolder}/${month}`;
if (!app.vault.getAbstractFileByPath(monthPath)) {
    await app.vault.createFolder(monthPath);
}

// task name 
let taskName = await tp.system.prompt("task name");
const fileName = `T${day}-${taskName}`

// move and rename file 
try { // Si el nombre está vacío, forzamos el error para ir al catch 
	if (!taskName || taskName.trim() === "") throw new Error("empty file name"); 
	await tp.file.move(`${monthPath}/${fileName}`);
} catch (err) { 
	await tp.file.move(`${monthPath}/T${day}-untitled`); 
}
// 5. Construir el YAML manualmente para evitar errores de anidación 
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
// Escribir el YAML y dejar una línea en blanco debajo 
const content = await tp.file.content; 
if (content.trim() === "") { 
	const tFile = tp.file.find_tfile(fileName); 
	await app.vault.modify(tFile, taskMetadata); 
}
%>


