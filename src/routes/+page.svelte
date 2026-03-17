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
      id: Date.now(),
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
      text.includes("ทำโปรเจก") ||
      text.includes("portfolio") ||
      text.includes("coding") ||
      text.includes("ทำงาน")
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
      text.includes("ถู") ||
      text.includes("กวาด") ||
      text.includes("เช็ด") ||
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
  $: currentMonth = new Date().getMonth();
  $: currentYear = new Date().getFullYear();

  $: monthlyTodos = todos.filter((t) => {
    const d = new Date(t.due);
    return d.getMonth() === currentMonth && d.getFullYear() === currentYear;
  });
  $: monthlyDone = monthlyTodos.filter((t) => t.done).length;
  $: monthlyTotal = monthlyTodos.length;
  $: categories = [...new Set(monthlyTodos.map((t) => t.category))];
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
    <h3 class="month-title">
      📅 {new Date().toLocaleString("th-TH", {
        month: "long",
        year: "numeric",
      })}
    </h3>

    <p class="progress-text">
      ทำไปแล้ว {monthlyDone}/{monthlyTotal} งาน
    </p>

    <div class="progress">
      <div
        class="bar"
        style="width: {monthlyTotal ? (monthlyDone / monthlyTotal) * 100 : 0}%"
      ></div>
    </div>
  </div>

  <ul>
    {#each categories as cat}
      <h4 class="category-title">{cat}</h4>

      <ul>
        {#each monthlyTodos.filter((t) => !t.done && t.category === cat) as t (t)}
          <li class="task-card {t.done ? 'done' : ''}">
            <label class="custom-checkbox">
              <input
                type="checkbox"
                checked={t.done}
                on:change={() => toggleDone(todos.indexOf(t))}
              />
              <span class="checkmark"></span>
            </label>

            <div class="task-text-container">
              <span class="task-text {t.done ? 'line' : ''}">
                {t.task}
              </span>

              <span class="due-date">
                <Calendar size="14" />
                {t.due}
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
    {/each}
  </ul>

  {#if done.length > 0}
    <h3 style="color:#888; margin-top: 24px;">✅ งานที่ทำแล้ว</h3>
    <ul>
      {#each done as t (t)}
        <li class="task-card done">
          <!-- checkbox วงกลม -->
          <label class="custom-checkbox">
            <input
              type="checkbox"
              checked={t.done}
              on:change={() => toggleDone(todos.indexOf(t))}
            />
            <span class="checkmark"></span>
          </label>

          <!-- text -->
          <div class="task-text-container">
            <span class="task-text line">
              🌟 {t.task}
            </span>

            <span class="due-date">📅 {t.due}</span>
            <span class="category">{t.category}</span>
          </div>

          <!-- ปุ่มไอคอน -->
          <div class="buttons">
            <button
              class="icon-btn edit"
              on:click={() => editTask(todos.indexOf(t))}
            >
              ✏️
            </button>

            <button
              class="icon-btn delete"
              on:click={() => deleteTask(todos.indexOf(t))}
            >
              🗑️
            </button>
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
    background: linear-gradient(-45deg, #ffffff, #f0f9ff, #e0f2fe, #f8fafc);
    background-size: 400% 400%;
    animation: gradientBG 15s ease infinite;
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

  .input-group input {
    padding: 12px;
    font-size: 15px;
    border-radius: 12px;
    border: 1px solid #e5e7eb;
    width: 100%;
    background: #ffffff;
    transition: all 0.2s;
  }
  .input-group input:focus {
    outline: none;
    border-color: #60a5fa;
    box-shadow: 0 0 0 4px rgba(96, 165, 250, 0.2);
  }

  .input-group button {
    background: #60a5fa; /* ฟ้าอ่อน */
    color: white;
    border: none;
    border-radius: 6px; /* 👈 เหลี่ยมขึ้น */
    padding: 10px;
    font-weight: 500;
    cursor: pointer;
    transition: 0.2s;
    width: fit-content; /* 👈 ให้กว้างตามข้อความ */
    align-self: center;
  }
  .input-group button:hover {
    transform: translateY(-2px) scale(1.02);
    box-shadow: 0 8px 20px rgba(59, 130, 246, 0.25);
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
    background: rgba(255, 255, 255, 0.9);
    border-radius: 18px;
    padding: 16px;
    margin-bottom: 14px;
    box-shadow: 0 6px 20px rgba(0, 0, 0, 0.06);
    transition: all 0.25s ease;
    border: 1px solid #eef2ff;
  }

  li:hover {
    transform: translateY(-4px) scale(1.01);
    box-shadow: 0 10px 28px rgba(37, 99, 235, 0.15);
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

  @keyframes gradientBG {
    0% {
      background-position: 0% 50%;
    }
    50% {
      background-position: 100% 50%;
    }
    100% {
      background-position: 0% 50%;
    }
  }

  /* บาร์ */
  .month-title {
    margin-top: 10px;
    color: #2563eb;
    text-align: center;
  }

  .progress-text {
    text-align: center;
    font-size: 14px;
    color: #64748b;
  }

  .progress {
    background: #e2e8f0;
    border-radius: 10px;
    height: 8px;
    margin: 10px auto 20px;
    width: 60%;
  }

  .bar {
    background: #60a5fa;
    height: 100%;
    border-radius: 10px;
    transition: 0.3s;
  }
  .category-title {
    margin-top: 20px;
    color: #3b82f6;
    font-size: 15px;
  }
  /* checkbox วงกลม */
  .custom-checkbox input {
    display: none;
  }

  .checkmark {
    width: 18px;
    height: 18px;
    border: 2px solid #60a5fa;
    border-radius: 50%;
    display: inline-block;
    position: relative;
  }

  .custom-checkbox input:checked + .checkmark {
    background: #60a5fa;
  }

  .custom-checkbox input:checked + .checkmark::after {
    content: "";
    position: absolute;
    top: 3px;
    left: 6px;
    width: 4px;
    height: 8px;
    border: solid white;
    border-width: 0 2px 2px 0;
    transform: rotate(45deg);
  }

  /* ปุ่มไอคอน */
  .icon-btn {
    border: none;
    background: transparent;
    cursor: pointer;
  }

  .icon-btn.edit {
    color: #facc15;
  }

  .icon-btn.delete {
    color: #ef4444;
  }

  /* fix การ์ด */
  .task-card {
    display: flex;
    align-items: center;
    gap: 10px;
  }
  /* ✅ ตอนทำเสร็จแล้วทั้งการ์ด */
  .task-card.done {
    opacity: 0.6;
    background: #f1f5f9;
  }

  /* ขีดฆ่าข้อความ */
  .task-text.line {
    text-decoration: line-through;
    color: #94a3b8;
  }

  /* ปุ่มตอน done แล้ว */
  .task-card.done .icon-btn.edit {
    color: #cbd5f5; /* จางลง */
  }

  .task-card.done .icon-btn.delete {
    color: #fca5a5; /* แดงอ่อน */
  }
</style>
