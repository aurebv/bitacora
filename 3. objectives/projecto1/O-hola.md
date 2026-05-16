


## 📊 Progress
```dataviewjs
/**
 * OBJECTIVE TRACKER
 * Automatically tracks tasks linked to this specific objective.
 * looks only in the YAML property of the task
 */

// 1. Configuration: Use the name we gave the file
const objectiveTitle = dv.current().file.name;

// 2. Fetch linked tasks from the "4. tasks" folder
const linkedTasks = dv.pages('"4. tasks"').where(p => {
	if (!p.objective) return false; 
	// Compara el texto de la propiedad con el título del objetivo 
	return String(p.objective).includes(objectiveTitle); 
});

// 3. Calculation logic
const total = linkedTasks.length;
const completed = linkedTasks.where(t => t.status === "done").length;
const progress = total > 0 ? Math.round((completed / total) * 100) : 0;

// 4. UI: Progress Bar
const barColor = progress === 100 ? "#44ff44" : "#7b59ad";
const containerStyle = "width: 100%; background-color: #333; border-radius: 8px; height: 12px; margin: 10px 0; border: 1px solid #444; overflow: hidden;";
const progressStyle = "width: " + progress + "%; background-color: " + barColor + "; height: 100%; transition: width 0.4s ease-in-out;";

dv.el("div", '<div style="' + containerStyle + '"><div style="' + progressStyle + '"></div></div>');
dv.paragraph("**" + progress + "%** complete — " + completed + " of " + total + " tasks finished");

dv.el("hr", "");

// 5. Rendering: Table of Commits
dv.header(3, "📑 Linked Tasks");

if (total === 0) {
    dv.paragraph("No tasks linked to this objective yet. Add `objective: [[" + objectiveTitle + "]]` to your task notes.");
} else {
    dv.table(["Task", "Status", "Description"], 
        linkedTasks
            .sort(t => t.file.ctime, "desc")
            .map(t => [
                t.file.link, 
                "**" + (t.status || "todo") + "**", 
                t.description || "—"
            ])
    );
}
```


*Do not include content in this note. Once it is archived it will be deleted. 
