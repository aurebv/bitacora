<%*
const folder = "1. bitacora";

// date 
const date = tp.date.now("YYYY.MM.DD");
const month = tp.date.now("YYYY.MM");
const monthPath = `${folder}/${month}`;
const finalFilePath = `${monthPath}/${date}.md`; // Añadimos .md para la comprobación real 
// 1. Verificar si la nota de hoy YA existe antes de hacer nada 
if (app.vault.getAbstractFileByPath(finalFilePath)) { 
	// Avisar al usuario con un mensaje flotante negro 
	new Notice("⚠️ There already exists a note for today.", 5000); 
	// Opcional: Si quieres que además te abra la nota existente en lugar de dejar la "Untitled" vacía 
	const existingFile = app.vault.getAbstractFileByPath(finalFilePath); app.workspace.getLeaf().openFile(existingFile); 
	return; // Detiene la ejecución por completo 
}

// create moth folder 
if (!app.vault.getAbstractFileByPath(monthPath)){
	await app.vault.createFolder(monthPath);
}

// rename and move new file 
await tp.file.move(`${monthPath}/${date}`);
%>
