<%*
//const mainFolder = "2. meetings"; 
const currentFolder = tp.file.folder(true);
const month = tp.date.now("YYYY.MM"); 
const day = tp.date.now("MM.DD");     

// path 
//const monthPath = `${mainFolder}/${month}`;
const monthPath = `${currentFolder}/${month}`;
if (!app.vault.getAbstractFileByPath(monthPath)) {
    await app.vault.createFolder(monthPath);
}

// user input 
let meetingName = await tp.system.prompt("meeting name");


// move and rename file 
try { 
	if (!meetingName || meetingName.trim() === "") throw new Error("empty file name"); 
	await tp.file.move(`${monthPath}/M${day}-${meetingName}`);
} catch (err) { 
	await tp.file.move(`${monthPath}/M${day}-untitled`); 
}
%>