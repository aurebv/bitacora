<%*
const folder = "1. bitacora";

// date 
const date = tp.date.now("YYYY.MM.DD");
const month = tp.date.now("YYYY.MM");
const monthPath = `${folder}/${month}`;
const finalFilePath = `${monthPath}/${date}.md`; 

// check if there is already a note for today 
if (app.vault.getAbstractFileByPath(finalFilePath)) { 
	new Notice("⚠️ Today's note already exists", 5000); 
	// open already existing note
	const existingFile = app.vault.getAbstractFileByPath(finalFilePath); app.workspace.getLeaf().openFile(existingFile); 
	return; 
}

// create moth folder 
if (!app.vault.getAbstractFileByPath(monthPath)){
	await app.vault.createFolder(monthPath);
}

// rename and move new file 
await tp.file.move(`${monthPath}/${date}`);
%>
