
```dataviewjs
/**
 * TASK MANAGER SIDEBAR (ATLAS)
 * Clean version: No bars, just text and links.
 */

// 1. Fetch data
const objectives = dv.pages('"objectives"');
const allTasks = dv.pages().file.tasks.where(t => !t.completed);
const allTaskFiles = dv.pages('"3. tasks"');

// 2. Filter logic for standard tasks
const isBacklog = (t) => t.text.includes("#backlog");
const isNormalTodo = (t) => !t.text.includes("#backlog") && !t.text.includes("#objective");

// --- RENDERING FUNCTIONS ---

const renderObjectives = () => {
    dv.el("h3", "🎯 Objectives", { attr: { style: "color: #7b59ad; margin-top: 20px; border-bottom: 1px solid #7b59ad40;" } });
    
    if (objectives.length === 0) {
        dv.paragraph("*No active objectives*");
        return;
    }

    objectives.forEach(obj => {
        // Find tasks linked to this specific objective
        const linkedTasks = allTaskFiles.where(p => 
            p.objective && String(p.objective).includes(obj.file.name)
        );
        
        const total = linkedTasks.length;
        const completed = linkedTasks.where(t => t.status === "done").length;
        const percent = total > 0 ? Math.round((completed / total) * 100) : 0;
        
        // Render: [80%] Objective Name (4/5)
        // Uses a subtle gray for the task count to keep focus on the name
        dv.el("div", `**${percent}%** ${obj.file.link} <span style="color: gray; font-size: 0.85em;">(${completed}/${total})</span>`, 
            { attr: { style: "margin-bottom: 8px; font-size: 0.95em;" } }
        );
    });
}

const renderSection = (title, taskList, color) => {
    dv.el("h3", title, { attr: { style: `color: ${color}; margin-top: 20px; border-bottom: 1px solid ${color}40; padding-bottom: 5px;` } });
    if (taskList.length === 0) {
        dv.paragraph("*No pending tasks*");
    } else {
        dv.taskList(taskList, false);
    }
}

// --- EXECUTION ---

renderObjectives();
renderSection("🔵 TODOs", allTasks.filter(isNormalTodo), "#4a6fa5");
renderSection("📁 Backlog", allTasks.filter(isBacklog), "#888888");
```



