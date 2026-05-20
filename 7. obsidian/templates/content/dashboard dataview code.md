<%*
// Constants for injecting Dataview code
const bt = "```";
const nl = "\n";
%>


<%- bt %>dataviewjs
/**
 * AUTO-GENERATED DASHBOARD
 * This block finds and renders markdown blocks containing the file-specific tag.
 */

const { MarkdownRenderer } = obsidian;

// 1. Configuration: Normalize tag based on current filename
const currentFileName = dv.current().file.name;
const dashboardTag = "#" + currentFileName.replace(/\s+/g, "");
const tagRegex = new RegExp(dashboardTag, "gi"); // global and intensive 

// 2. Optimization: Only fetch pages that Dataview already indexed with this tag
const taggedPages = dv.pages(dashboardTag) // we look only files with the tag
    .filter(p => p.file.path !== dv.current().file.path) // we dont look in this file 
    .sort(p => p.file.ctime, 'desc'); // show new blocks first

/**
 * Processes a file and extracts blocks containing the specific tag
 */
async function getMatchingBlocks() {
    let matches = [];

    for (const page of taggedPages) {
        const file = app.vault.getAbstractFileByPath(page.file.path);
        if (!file) continue;

        const content = await app.vault.cachedRead(file);
        
        // Split by double newlines to isolate paragraphs/blocks
		// that way we follow the logic of markdown 
		// (show full lists, code blocks, ...)
        const blocks = content.split(/\n\s*\n/).reverse();

        for (const block of blocks) {
            if (tagRegex.test(block)) {
                matches.push({
                    link: page.file.link,
                    path: page.file.path,
                    // Remove the tag from the text before storing
                    text: block.replace(tagRegex, "").trim() 
	                // global intensive regex setting
                });
            }
        }
    }
    return matches;
}

// 3. Execution and Rendering
const results = await getMatchingBlocks();

if (results.length === 0) {
    dv.paragraph("No blocks found with " + dashboardTag);
} else {
    for (const item of results) {
        const container = dv.el("div", ""); // empty html block
        
        // Render the cleaned markdown block
        await MarkdownRenderer.render(
            app, 
            item.text, // text to draw
            container, // where to draw it 
            item.path, // path with original info (for images)
            dv.component // when the note is not open (or in view) the render dies
        );
        
        // Render source link and separator
        dv.el("div", `<sub>↳ ${item.link}</sub>`, { attr: { style: "text-align: right; opacity: 0.8;" } });
        dv.el("hr", "");
    }
}
<%- nl + bt %>
