<%*
// Constants for injecting Dataview code
const bt = "```";
const nl = "\n";
%>


<%- bt %>dataviewjs
/**
 * TASK MANAGER SIDEBAR (ATLAS) - HIGH PERFORMANCE VERSION
 */

const FOLDER_OBJECTIVES = '"3. objectives"'; 
const FOLDER_TASKS = '"4. tasks"';
// Optimization: Limit the scope to specific folders to avoid scanning the entire vault
const TODO_SOURCES = '"" or "4. tasks" or "1. bitacora" or "2. meetings"'; 

// 1. FETCH DATA
const objectives = dv.pages(FOLDER_OBJECTIVES);
const rawTasks = dv.pages(TODO_SOURCES).file.tasks.where(t => !t.completed);
const allTaskFiles = dv.pages(FOLDER_TASKS);

// 2. OPTIMIZATION: Categorize TODOs in a single pass (O(n) instead of O(n*2))
const todos = [];
const backlog = [];

for (let task of rawTasks) {
    if (task.text.includes("#backlog")) {
        backlog.push(task);
    } else {
        todos.push(task);
    }
}

// 3. OPTIMIZATION: Pre-group tasks by objective (Indexing)
// This avoids running .where() inside the objectives loop
const tasksByObjective = {};

allTaskFiles.forEach(p => {
    if (p.objective) {
        // We extract the name clearly. 
        // If p.objective is a link, .path or .fileName usually works better than String()
        let objName = p.objective;
        
        // If it's a Dataview Link object, get the display name or path
        if (typeof objName === 'object' && objName.path) {
            // Extracts "My Objective" from "Folder/My Objective.md"
            objName = objName.path.split('/').pop().replace('.md', '');
        } else {
            objName = String(objName);
        }

        if (!tasksByObjective[objName]) tasksByObjective[objName] = [];
        tasksByObjective[objName].push(p);
    }
});

// --- RENDERING FUNCTIONS ---

const renderObjectives = () => {
    dv.el("h3", "Objectives", { attr: { style: "color: #7b59ad; margin-top: 20px; border-bottom: 1px solid #7b59ad40;" } });
    
    if (objectives.length === 0) {
        dv.paragraph("*No objectives in /3. objectives*");
        return;
    }
    
    let renderedCount = 0;

    objectives.forEach(obj => {
        // Optimization: Quick lookup in our pre-built dictionary
        const linkedTasks = tasksByObjective[obj.file.name] || [];
        
        const total = linkedTasks.length;
        const completed = linkedTasks.filter(t => t.status === "done").length;
        const percent = total > 0 ? Math.round((completed / total) * 100): 0;
        
	    if (percent === 100) return; 
	    
	    renderedCount++;
	    
        dv.el("div", `${obj.file.link} <span style="color: gray; font-size: 0.85em;">**${percent}%** (${completed}/${total})</span>`, 
            { attr: { style: "margin-bottom: 8px; font-size: 0.95em;" } }
        );
    });
    
    if (renderedCount === 0) {
	    dv.paragraph("*All objectives completed! :)*")
    }
}

const renderSection = (title, taskList, color) => {
	dv.el("h3", title, { attr: { style: `color: ${color}; margin-top: 20px; border-bottom: 1px solid ${color}40; padding-bottom: 5px;` } });
	if (taskList.length === 0) {
		dv.paragraph("No pending tasks");
	} else {
		dv.taskList(taskList, false);
	}
}

// --- EXECUTION ---
renderObjectives();
renderSection("TODOs", todos, "#4a6fa5");
renderSection("Backlog", backlog, "#888888");
<%- nl + bt %>

