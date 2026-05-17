<%*
const fileName = tp.file.title;
const month = tp.date.now("YYYY.MM"); 
const day = tp.date.now("MM.DD");

// define task metadata 
const taskMetadata = `---
status: work in progress
description:
objective:
---
`;

// rules 
// meetings 
if (fileName.startsWith(`M${day}`)){
	// path 
	const monthPath = `2. meetings/${month}`;
	if (!app.vault.getAbstractFileByPath(monthPath)) {
		await app.vault.createFolder(monthPath);
	}
	// move (and rename)
	await tp.file.move(`${monthPath}/${fileName}`)
}
// tasks
else if (fileName.startsWith(`T${day}`)){
	// path 
	const monthPath = `4. tasks/${month}`;
	if (!app.vault.getAbstractFileByPath(monthPath)) {
		await app.vault.createFolder(monthPath);
	}
	// move (and rename)
	await tp.file.move(`${monthPath}/${fileName}`)
	// write metadata, only if file is empty
	const content = await tp.file.content; 
	if (content.trim() === "") { 
		const tFile = tp.file.find_tfile(fileName); 
		await app.vault.modify(tFile, taskMetadata); 
	}
}
//objectives
else if (fileName.startsWith(`O-`)){
	// move (and rename)
	await tp.file.move(`3. objectives/${fileName}`);
	// retrieving data view code from content note
	const dvCode = await tp.file.include("[[objective dataview code]]"); 
	// write it down
	tR += dvCode;
}
%>


