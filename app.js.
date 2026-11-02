let SUPABASE_URL = "";
let API_KEY = "";
let TABLE = "tasks";
let tasks = [];
let editingId = null;

const form = document.getElementById("task-form");
const titleInput = document.getElementById("task-title");
const descInput = document.getElementById("task-description");
const submitBtn = document.getElementById("task-submit");
const cancelBtn = document.getElementById("task-cancel");
const listEl = document.getElementById("task-list");
const statusEl = document.getElementById("status-message");

function showStatus(message) {
  statusEl.textContent = message;
}

async function loadConfig() {
  const res = await fetch("config.json");
  if (!res.ok) throw new Error("Could not load config.json");
  const cfg = await res.json();
  SUPABASE_URL = cfg.supabaseUrl;
  API_KEY = cfg.supabasePublishableKey;
  TABLE = cfg.tableName || "tasks";
}

// Generic helper for every REST call to Supabase
async function api(query, options = {}) {
  const res = await fetch(`${SUPABASE_URL}/rest/v1/${TABLE}${query}`, {
    ...options,
    headers: {
      apikey: API_KEY,
      Authorization: `Bearer ${API_KEY}`,
      "Content-Type": "application/json",
      ...(options.headers || {}),
    },
  });
  const text = await res.text();
  if (!res.ok) throw new Error(`${res.status}: ${text}`);
  return text ? JSON.parse(text) : null;
}

// READ: GET /tasks?select=*
async function loadTasks() {
  try {
    tasks = await api("?select=*&order=id.asc");
    renderTasks();
    showStatus("");
  } catch (err) {
    showStatus("Could not load tasks. " + err.message);
  }
}

function renderTasks() {
  listEl.innerHTML = "";
  if (tasks.length === 0) {
    listEl.textContent = "No tasks yet. Add your first task above.";
    return;
  }
  tasks.forEach((task) => {
    const item = document.createElement("div");
    item.className = "task-item" + (task.completed ? " done" : "");
    item.dataset.id = task.id;

    const title = document.createElement("div");
    title.className = "task-title";
    title.textContent = task.title;

    const desc = document.createElement("div");
    desc.className = "task-description";
    desc.textContent = task.description || "";

    const status = document.createElement("div");
    status.className = "task-status";
    status.textContent = task.completed ? "Completed" : "Not completed";

    const toggle = document.createElement("button");
    toggle.className = "toggle-task";
    toggle.textContent = task.completed ? "Mark incomplete" : "Mark complete";

    const edit = document.createElement("button");
    edit.className = "edit-task";
    edit.textContent = "Edit";

    const del = document.createElement("button");
    del.className = "delete-task";
    del.textContent = "Delete";

    item.append(title, desc, status, toggle, edit, del);
    listEl.appendChild(item);
  });
}

function resetForm() {
  editingId = null;
  form.reset();
  submitBtn.textContent = "Add task";
  cancelBtn.hidden = true;
}

// CREATE (POST) or UPDATE (PATCH)
form.addEventListener("submit", async (e) => {
  e.preventDefault();
  const title = titleInput.value.trim();
  const description = descInput.value.trim();
  if (!title) {
    showStatus("Task title is required.");
    return;
  }
  try {
    if (editingId === null) {
      await api("", {
        method: "POST",
        body: JSON.stringify({ title, description, completed: false }),
      });
    } else {
      await api(`?id=eq.${editingId}`, {
        method: "PATCH",
        body: JSON.stringify({ title, description }),
      });
    }
    resetForm();
    await loadTasks();
  } catch (err) {
    showStatus("Could not save the task. " + err.message);
  }
});

cancelBtn.addEventListener("click", resetForm);

// Buttons inside the list (edit, delete, toggle)
listEl.addEventListener("click", async (e) => {
  const item = e.target.closest(".task-item");
  if (!item) return;
  const id = Number(item.dataset.id);
  const task = tasks.find((t) => t.id === id);
  if (!task) return;

  try {
    if (e.target.classList.contains("delete-task")) {
      // DELETE /tasks?id=eq.ID
      await api(`?id=eq.${id}`, { method: "DELETE" });
      if (editingId === id) resetForm();
      await loadTasks();
    } else if (e.target.classList.contains("toggle-task")) {
      // PATCH completed flag
      await api(`?id=eq.${id}`, {
        method: "PATCH",
        body: JSON.stringify({ completed: !task.completed }),
      });
      await loadTasks();
    } else if (e.target.classList.contains("edit-task")) {
      editingId = id;
      titleInput.value = task.title;
      descInput.value = task.description || "";
      submitBtn.textContent = "Save changes";
      cancelBtn.hidden = false;
      titleInput.focus();
    }
  } catch (err) {
    showStatus("Action failed. " + err.message);
  }
});

(async function init() {
  try {
    await loadConfig();
    await loadTasks();
  } catch (err) {
    showStatus("Setup error. " + err.message);
  }
})();
