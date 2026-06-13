
```dataviewjs
/**
 * TASK MANAGER SIDEBAR (ATLAS) - HIGH PERFORMANCE VERSION
 */

const FOLDER_OBJECTIVES = '"3. objectives"'; 
const FOLDER_TASKS = '"4. tasks"';
// Optimization: Limit the scope to specific folders to avoid scanning the entire vault
const TODO_SOURCES = '"" or "4. tasks" or "1. bitacora" or "2. meetings"'; 

// 1. FETCH DATA
const objectives = dv.pages(FOLDER_OBJECTIVES).sort(o => o.file.ctime, "desc");
const allTaskFiles = dv.pages(FOLDER_TASKS);

// order the TODOs after priority and cronological order 
const rawTODOs = Array.from(dv.pages(TODO_SOURCES).file.tasks.where(t => !t.completed)); //JavaScript array

// priority is defined with the number of ! at the end of the TODO
const getPriority = (todo) => { 
	const match = todo.text.match(/(!+)$/); // look for ! only at the end of the text 
	return match ? match[0].length : 0; 
};

rawTODOs.sort((a, b) => {
	// JavaScript sort: if return < 0 than a before b, if return > 0 than b before a 
	if (!a || !b || !a.path || !b.path) return 0;
	
	const priorityA = getPriority(a); 
	const priorityB = getPriority(b);
	
	// put the TODOs with more priority first
	if (priorityA !== priorityB) {
        return priorityB - priorityA; 
    }
    
    // If TODOs are from diferent files, put the newest first 
    if (a.path !== b.path) {
        const fileA = dv.page(a.path);
        const fileB = dv.page(b.path);
        if (!fileA || !fileB) return 0;
        return fileB.file.ctime - fileA.file.ctime; 
    }
    
    // if the TODOs are from the same file, the ones under appear first 
    return b.line - a.line; // 'line' represent the line number in the md file
});

const todos = [];
const backlog = [];

for (let todo of rawTODOs) {
	todo.text = todo.text.replace(/\s*(!+)$/, ""); // do not render ! priority signs 
    if (todo.text.includes("#backlog")) {
        backlog.push(todo);
    } else {
        todos.push(todo);
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

const renderSection = (title, TODOList, color) => {
	dv.el("h3", title, { attr: { style: `color: ${color}; margin-top: 20px; border-bottom: 1px solid ${color}40; padding-bottom: 5px;` } });
	if (TODOList.length === 0) {
		dv.paragraph("No pending tasks");
	} else {
		dv.taskList(TODOList, false);
	}
}

// --- EXECUTION ---
renderObjectives();
renderSection("TODOs", todos, "#4a6fa5");
renderSection("Backlog", backlog, "#888888");
```



