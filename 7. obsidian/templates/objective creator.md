<%*
// 1. Prompt for the Objective name
const objectiveName = await tp.system.prompt("Objective Name");

if (objectiveName) {
    // Rename the file
    await tp.file.rename(`O-${objectiveName}`);
}

%>

<% tp.file.include("[[objective dataview]]") %>
