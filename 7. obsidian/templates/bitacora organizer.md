<%*
const folder = "1. bitacora";

// date 
const date = tp.date.now("YYYY.MM.DD");
const month = tp.date.now("YYYY.MM");

// create moth folder 
const monthPath = `${folder}/${month}`;
if (!app.vault.getAbstractFileByPath(monthPath)){
	await app.vault.createFolder(monthPath);
}

// rename and move new file 
await tp.file.move(`${monthPath}/${date}`);
%>
