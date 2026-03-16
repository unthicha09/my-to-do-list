<script>
  import { onMount } from "svelte";
  import { fade, slide, scale } from "svelte/transition";
  import { Plus, Pencil, Trash2, Calendar, CheckCircle } from "lucide-svelte";

  let todos = [];
  let newTask = "";
  let dueDate = "";
  let editIndex = -1;
  let toast = "";

  onMount(() => {
    const saved = localStorage.getItem("todos");
    if (saved) {
      todos = JSON.parse(saved);
    }
    checkNotifications();
  });

  // แยกงานที่ทำแล้วยังไม่ทำ
  $: notDone = todos.filter((t) => !t.done);
  $: done = todos.filter((t) => t.done);

  function saveTodos() {
    localStorage.setItem("todos", JSON.stringify(todos));
  }

  function showToast(msg) {
    toast = msg;
    setTimeout(() => (toast = ""), 2500);
  }

  function addTask() {
    if (!newTask || !dueDate) return;
    const newItem = {
      task: newTask,
      due: dueDate,
      done: false,
      category: getCategory(newTask),
    };
    if (editIndex !== -1) {
      todos[editIndex] = newItem;
      todos = [...todos];
      showToast("✏️ แก้ไขงานแล้ว");
      editIndex = -1;
    } else {
      todos = [...todos, newItem]; // <- ใช้แบบนี้เพื่อให้ reactive
      showToast("✅ เพิ่มงานใหม่!");
    }
    newTask = "";
    dueDate = "";
    saveTodos();
  }

  function toggleDone(index) {
    todos[index].done = !todos[index].done;
    todos = [...todos];
    saveTodos();
  }

  function deleteTask(index) {
    todos = todos.filter((_, i) => i !== index);
    saveTodos();
    showToast("🗑️ ลบงานแล้ว");
  }

  function editTask(index) {
    newTask = todos[index].task;
    dueDate = todos[index].due;
    editIndex = index;
  }
  function getCategory(task) {
    const text = task.toLowerCase();

    if (
      text.includes("อ่าน") ||
      text.includes("การบ้าน") ||
      text.includes("สอบ") ||
      text.includes("เรียน")
    ) {
      return "📚 Study";
    }

    if (
      text.includes("โค้ด") ||
      text.includes("โปรเจก") ||
      text.includes("portfolio") ||
      text.includes("coding")
    ) {
      return "💻 Project";
    }

    if (
      text.includes("วิ่ง") ||
      text.includes("ฟิตเนส") ||
      text.includes("ออกกำลังกาย")
    ) {
      return "💪 Health";
    }

    if (
      text.includes("ล้าง") ||
      text.includes("ซัก") ||
      text.includes("ทำความสะอาด")
    ) {
      return "🧹 Chores";
    }

    if (
      text.includes("ซื้อ") ||
      text.includes("ธุระ") ||
      text.includes("นัด")
    ) {
      return "🏠 Personal";
    }

    return "📌 Other";
  }
  function checkNotifications() {
    if (Notification.permission !== "granted") {
      Notification.requestPermission();
    }
    const now = new Date();
    todos.forEach((t) => {
      const due = new Date(t.due);
      const diffDays = (due - now) / (1000 * 60 * 60 * 24);
      if (diffDays < 1 && diffDays > 0 && !t.done) {
        new Notification("📌 งานใกล้ครบกำหนด!", {
          body: `${t.task} กำหนด: ${t.due}`,
        });
      }
    });
  }
  function getCategoryEmoji(category) {
    if (!category) return "📌";

    if (category.includes("Study")) return "📚";
    if (category.includes("Project")) return "💻";
    if (category.includes("Health")) return "💪";
    if (category.includes("Chores")) return "🧹";
    if (category.includes("Personal")) return "🏠";

    return "📌";
  }
</script>

<div class="todo-container">
  <h1>📝 My To-Do List</h1>

  <div class="input-group">
    <input bind:value={newTask} placeholder="Task name..." />

    <div class="date-input">
      <Calendar size="18" />
      <input type="date" bind:value={dueDate} />
    </div>

    <button on:click={addTask} class:edit-mode={editIndex !== -1}>
      {#if editIndex !== -1}
        <Pencil size="18" /> Edit Task
      {:else}
        <Plus size="18" /> Add Task
      {/if}
    </button>
  </div>

  <ul>
    {#each notDone as t, i (i)}
      <li in:slide={{ duration: 250 }} class="task-card">
        <input
          type="checkbox"
          checked={t.done}
          on:change={() => toggleDone(todos.indexOf(t))}
        />

        <div class="task-text-container">
          <span class:done={t.done} class="task-text">
            {t.task}
          </span>

          <span class="due-date">
            <Calendar size="14" />
            {t.due}
          </span>

          <span class="category">
            {t.category}
          </span>
        </div>

        <div class="buttons">
          <button
            class="icon-btn edit"
            on:click={() => editTask(todos.indexOf(t))}
          >
            <Pencil size="16" />
          </button>

          <button
            class="icon-btn delete"
            on:click={() => deleteTask(todos.indexOf(t))}
          >
            <Trash2 size="16" />
          </button>
        </div>
      </li>
    {/each}
  </ul>

  {#if done.length > 0}
    <h3 style="color:#888; margin-top: 24px;">✅ งานที่ทำแล้ว</h3>
    <ul>
      {#each done as t, i (i)}
        <li in:fade={{ duration: 300 }}>
          <input
            type="checkbox"
            checked={t.done}
            on:change={() => toggleDone(todos.indexOf(t))}
          />
          <div class="task-text-container">
            <span class:done={t.done} class="task-text">
              🌟 {t.task}
            </span>
            <span class="due-date">📅 {t.due}</span>
            <span class="category">{t.category}</span>
          </div>
          <div class="buttons">
            <button
              class="edit-button"
              on:click={() => editTask(todos.indexOf(t))}>แก้ไข</button
            >
            <button
              class="delete-button"
              on:click={() => deleteTask(todos.indexOf(t))}>ลบ</button
            >
          </div>
        </li>
      {/each}
    </ul>
  {/if}
</div>

{#if toast}
  <div class="toast" transition:fade>{toast}</div>
{/if}

<style>
  :global(body) {
    font-family: "Noto Sans Thai", sans-serif;
    background: linear-gradient(135deg, #eef2ff, #f8fafc);
    margin: 0;
    padding: 0;
  }

  .todo-container {
    width: 100%;
    max-width: 1000px; /* หรือจะลบออกเลยก็ได้ */
    margin: auto;
    padding: 40px 20px;
  }
  h1 {
    text-align: center;
    color: #2563eb;
    font-weight: 700;
    margin-bottom: 30px;
  }

  /* input */

  .input-group {
    display: flex;
    flex-direction: column;
    gap: 12px;
    margin-bottom: 30px;
  }

  .input-group input,
  .input-group button {
    padding: 12px;
    font-size: 15px;
    border-radius: 10px;
    border: 1px solid #ddd;
    width: 100%;
    box-sizing: border-box;
    transition: 0.2s;
  }

  .input-group input:focus {
    outline: none;
    border-color: #3b82f6;
    box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.2);
  }

  .input-group button {
    background: #2563eb;
    color: white;
    border: none;
    font-weight: 500;
    cursor: pointer;
  }

  .input-group button:hover {
    transform: translateY(-2px);
    background: #1d4ed8;
    box-shadow: 0 5px 12px rgba(0, 0, 0, 0.1);
  }

  .input-group button.edit-mode {
    background-color: #facc15;
    color: #333;
  }

  /* list */

  ul {
    list-style: none;
    padding: 0;
    width: 100%;
  }

  li {
    display: flex;
    align-items: flex-start;
    background-color: white;
    border-radius: 14px;
    padding: 14px;
    margin-bottom: 12px;
    box-shadow: 0 3px 10px rgba(0, 0, 0, 0.05);
    transition: 0.2s;
  }

  li:hover {
    transform: translateY(-3px);
    box-shadow: 0 8px 18px rgba(0, 0, 0, 0.08);
  }

  /* task */

  .task-text-container {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    margin-left: 10px;
    flex: 1;
  }

  .task-text {
    font-size: 16px;
    font-weight: 500;
  }

  .task-text.done {
    text-decoration: line-through;
    color: #9ca3af;
  }

  .due-date {
    font-size: 13px;
    color: #6b7280;
    margin-top: 4px;
  }

  /* buttons */

  .buttons {
    display: flex;
    gap: 6px;
  }

  .buttons button {
    padding: 6px 10px;
    border: none;
    border-radius: 8px;
    font-size: 12px;
    cursor: pointer;
    transition: 0.2s;
  }

  .edit-button {
    background-color: #3b82f6;
    color: white;
  }

  .delete-button {
    background-color: #ef4444;
    color: white;
  }

  .edit-button:hover {
    transform: scale(1.05);
    background-color: #2563eb;
  }

  .delete-button:hover {
    transform: scale(1.05);
    background-color: #dc2626;
  }

  /* toast */

  .toast {
    position: fixed;
    bottom: 25px;
    left: 50%;
    transform: translateX(-50%);
    background: #111827;
    color: white;
    padding: 12px 22px;
    border-radius: 999px;
    font-size: 14px;
    opacity: 0.95;
    box-shadow: 0 6px 16px rgba(0, 0, 0, 0.2);
    animation: toastPop 0.3s ease;
  }

  @keyframes toastPop {
    from {
      transform: translate(-50%, 20px);
      opacity: 0;
    }
    to {
      transform: translate(-50%, 0);
      opacity: 1;
    }
  }

  /* responsive */

  @media (max-width: 600px) {
    .todo-container {
      margin: 20px;
    }

    li {
      flex-direction: column;
      align-items: flex-start;
    }

    .buttons {
      margin-top: 10px;
    }
  }

  .category {
    font-size: 12px;
    background: #e0e7ff;
    color: #3730a3;
    padding: 3px 8px;
    border-radius: 6px;
    margin-top: 4px;
    display: inline-block;
  }
</style>
