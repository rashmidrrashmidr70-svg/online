# online
An Online Project Management System is a cloud-based platform that helps teams plan, track, and collaborate on work in real time. It centralizes task assignments, file sharing, deadlines, and progress dashboards into one workspace, boosting accountability, streamlining communication, and ensuring projects are delivered on schedule from 
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>TaskFlow - Online Project Management</title>
  <style>
    * { box-sizing: border-box; font-family: system-ui, -apple-system, sans-serif; }
    body { margin: 0; background: #f0f2f5; color: #1e293b; }
    header { background: #2563eb; color: white; padding: 1rem 2rem; display: flex; justify-content: space-between; align-items: center; }
    .controls { max-width: 1200px; margin: 1.5rem auto; padding: 0 1rem; display: flex; flex-wrap: wrap; gap: 0.75rem; background: white; padding: 1rem; border-radius: 10px; box-shadow: 0 2px 6px rgba(0,0,0,0.06); }
    input, select, button { padding: 0.6rem 0.9rem; border: 1px solid #cbd5e1; border-radius: 6px; font-size: 0.95rem; }
    input { flex: 2; min-width: 200px; }
    select { flex: 1; min-width: 130px; }
    button.add-btn { background: #2563eb; color: white; border: none; font-weight: 600; cursor: pointer; }
    button.add-btn:hover { background: #1d4ed8; }
    .board { display: grid; grid-template-columns: repeat(3, 1fr); gap: 1.5rem; max-width: 1200px; margin: 0 auto 2rem; padding: 0 1rem; }
    .column { background: #e2e8f0; border-radius: 10px; padding: 1rem; min-height: 450px; }
    .column h3 { margin: 0 0 1rem; display: flex; justify-content: space-between; font-size: 1.05rem; color: #334155; }
    .badge-count { background: #cbd5e1; padding: 0.1rem 0.6rem; border-radius: 999px; font-size: 0.85rem; }
    .task-card { background: white; padding: 1rem; border-radius: 8px; margin-bottom: 0.8rem; box-shadow: 0 1px 4px rgba(0,0,0,0.08); cursor: grab; }
    .task-card h4 { margin: 0.4rem 0; font-size: 1rem; }
    .meta { display: flex; justify-content: space-between; font-size: 0.8rem; color: #64748b; margin-top: 0.6rem; }
    .priority { font-size: 0.75rem; font-weight: 700; padding: 0.15rem 0.5rem; border-radius: 4px; text-transform: uppercase; }
    .High { background: #fee2e2; color: #dc2626; }
    .Medium { background: #fef3c7; color: #d97706; }
    .Low { background: #dcfce7; color: #16a34a; }
    .actions { display: flex; gap: 0.4rem; margin-top: 0.8rem; }
    .actions button { flex: 1; padding: 0.3rem; font-size: 0.8rem; cursor: pointer; background: #f8fafc; }
    .actions button.del { color: #dc2626; border-color: #fecaca; }
    @media (max-width: 850px) { .board { grid-template-columns: 1fr; } }
  </style>
</head>
<body>

<header>
  <h2>📋 TaskFlow Project Manager</h2>
  <span id="progress-summary">Progress: 0% Complete</span>
</header>

<!-- Task Creation Bar -->
<div class="controls">
  <input type="text" id="task-title" placeholder="Enter new task title (e.g., Design Login UI)...">
  <input type="text" id="task-assignee" placeholder="Assignee (e.g., Rahul)" style="flex: 1;">
  <select id="task-priority">
    <option value="High">High Priority</option>
    <option value="Medium" selected>Medium Priority</option>
    <option value="Low">Low Priority</option>
  </select>
  <input type="date" id="task-deadline" style="flex: 1;">
  <button class="add-btn" onclick="addTask()">+ Add Task</button>
</div>

<!-- Kanban Board Columns -->
<div class="board">
  <div class="column" ondragover="allowDrop(event)" ondrop="dropTask(event, 'todo')">
    <h3>📝 To Do <span class="badge-count" id="count-todo">0</span></h3>
    <div id="col-todo"></div>
  </div>

  <div class="column" ondragover="allowDrop(event)" ondrop="dropTask(event, 'inprogress')">
    <h3>⚙️ In Progress <span class="badge-count" id="count-inprogress">0</span></h3>
    <div id="col-inprogress"></div>
  </div>

  <div class="column" ondragover="allowDrop(event)" ondrop="dropTask(event, 'done')">
    <h3>✅ Completed <span class="badge-count" id="count-done">0</span></h3>
    <div id="col-done"></div>
  </div>
</div>

<script>
  let tasks = JSON.parse(localStorage.getItem("projectTasks")) || [
    { id: 1, title: "Setup Database Schema", assignee: "Ananya", priority: "High", deadline: "2026-11-25", status: "done" },
    { id: 2, title: "Build REST API Endpoints", assignee: "Karthik", priority: "High", deadline: "2026-11-28", status: "inprogress" },
    { id: 3, title: "Create User Dashboard UI", assignee: "Sneha", priority: "Medium", deadline: "2026-12-02", status: "todo" }
  ];

  function saveAndRender() {
    localStorage.setItem("projectTasks", JSON.stringify(tasks));
    renderBoard();
  }

  function addTask() {
    const title = document.getElementById("task-title").value.trim();
    const assignee = document.getElementById("task-assignee").value.trim() || "Unassigned";
    const priority = document.getElementById("task-priority").value;
    const deadline = document.getElementById("task-deadline").value || "No due date";

    if (!title) return alert("Please enter a task title!");

    tasks.push({
      id: Date.now(),
      title,
      assignee,
      priority,
      deadline,
      status: "todo"
    });

    document.getElementById("task-title").value = "";
    document.getElementById("task-assignee").value = "";
    saveAndRender();
  }

  function moveTask(id, newStatus) {
    const task = tasks.find(t => t.id === id);
    if (task) {
      task.status = newStatus;
      saveAndRender();
    }
  }

  function deleteTask(id) {
    tasks = tasks.filter(t => t.id !== id);
    saveAndRender();
  }

  // Drag and Drop Handlers
  function dragTask(event, id) {
    event.dataTransfer.setData("text/plain", id);
  }

  function allowDrop(event) {
    event.preventDefault();
  }

  function dropTask(event, targetStatus) {
    event.preventDefault();
    const id = Number(event.dataTransfer.getData("text/plain"));
    moveTask(id, targetStatus);
  }

  function renderBoard() {
    const cols = { todo: "", inprogress: "", done: "" };
    const counts = { todo: 0, inprogress: 0, done: 0 };

    tasks.forEach(task => {
      counts[task.status]++;
      cols[task.status] += `
        <div class="task-card" draggable="true" ondragstart="dragTask(event, ${task.id})">
          <span class="priority ${task.priority}">${task.priority}</span>
          <h4>${task.title}</h4>
          <div class="meta">
            <span>👤 ${task.assignee}</span>
            <span>📅 ${task.deadline}</span>
          </div>
          <div class="actions">
            ${task.status !== "todo" ? `<button onclick="moveTask(${task.id}, 'todo')">← To Do</button>` : ""}
            ${task.status !== "inprogress" ? `<button onclick="moveTask(${task.id}, 'inprogress')">In Progress</button>` : ""}
            ${task.status !== "done" ? `<button onclick="moveTask(${task.id}, 'done')">Done ✓</button>` : ""}
            <button class="del" onclick="deleteTask(${task.id})">✕</button>
          </div>
        </div>
      `;
    });

    document.getElementById("col-todo").innerHTML = cols.todo;
    document.getElementById("col-inprogress").innerHTML = cols.inprogress;
    document.getElementById("col-done").innerHTML = cols.done;

    document.getElementById("count-todo").innerText = counts.todo;
    document.getElementById("count-inprogress").innerText = counts.inprogress;
    document.getElementById("count-done").innerText = counts.done;

    const percent = tasks.length ? Math.round((counts.done / tasks.length) * 100) : 0;
    document.getElementById("progress-summary").innerText = `Progress: ${percent}% Complete (${counts.done}/${tasks.length})`;
  }

  renderBoard();
</script>

</body>
</html>
