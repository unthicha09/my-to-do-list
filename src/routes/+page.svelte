<script>
  import { onMount } from "svelte";
  import { fade } from "svelte/transition";

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
    const newItem = { task: newTask, due: dueDate, done: false };
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
</script>

<div class="todo-container">
  <h1>📝 My To-Do List</h1>

  <div class="input-group">
    <input bind:value={newTask} placeholder="✏️ ชื่องาน" />
    <input type="date" bind:value={dueDate} />
    <button on:click={addTask} class:edit-mode={editIndex !== -1}>
      {editIndex !== -1 ? "📝 แก้ไขงาน" : "➕ เพิ่มงาน"}
    </button>
  </div>

  <ul>
    {#each notDone as t, i (i)}
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
    background: linear-gradient(to right, #fdfbfb, #ebedee);
    margin: 0;
    padding: 0;
  }

  h1 {
    text-align: center;
    color: #3b82f6;
    margin-bottom: 16px;
  }

  .todo-container {
    max-width: 100vw;
    height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: center;
    padding: 24px;
    box-sizing: border-box;
    background-color: white;
    overflow-y: auto;
    scrollbar-width: none; /* สำหรับ Firefox */
    -ms-overflow-style: none; /* สำหรับ IE และ Edge */
  }

  .todo-container::-webkit-scrollbar {
    display: none; /* สำหรับ Chrome, Safari และ Opera */
  }

  .input-group {
    display: flex;
    flex-direction: column;
    gap: 12px;
    align-items: center;
    margin-bottom: 20px;
    width: 100%;
  }

  .input-group input,
  .input-group button {
    padding: 12px;
    font-size: 16px;
    border-radius: 12px;
    border: 1px solid #ddd;
    width: 100%;
    max-width: 400px;
    box-sizing: border-box;
    transition: all 0.2s ease;
  }

  .input-group input:focus {
    outline: none;
    border-color: #3b82f6;
    box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.2);
  }

  .input-group button {
    background: #3b82f6;
    color: white;
    border: none;
    font-weight: 500;
    cursor: pointer;
  }

  .input-group button:hover {
    background: #2563eb;
    transform: scale(1.02);
  }

  .input-group button.edit-mode {
    background-color: #facc15;
    color: #333;
  }

  .input-group button.edit-mode:hover {
    background-color: #fbbf24;
  }

  ul {
    list-style: none;
    padding: 0;
    width: 100%;
    max-width: 600px;
  }

  li {
    display: flex;
    align-items: flex-start;
    background-color: #e0f7fa;
    border-radius: 12px;
    padding: 14px;
    margin-bottom: 12px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  }

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
    color: #aaa;
  }

  .due-date {
    font-size: 14px;
    color: #666;
    margin-top: 4px;
  }

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
    background-color: #60a5fa;
    color: white;
  }

  .delete-button {
    background-color: #f87171;
    color: white;
  }

  .edit-button:hover {
    background-color: #3b82f6;
  }

  .delete-button:hover {
    background-color: #ef4444;
  }

  .toast {
    position: fixed;
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
    background: #333;
    color: white;
    padding: 12px 24px;
    border-radius: 999px;
    font-size: 14px;
    opacity: 0.9;
    z-index: 1000;
  }

  @media (max-width: 600px) {
    .todo-container {
      margin: 20px;
    }

    .input-group input,
    .input-group button {
      max-width: 100%;
    }

    li {
      flex-direction: column;
      align-items: flex-start;
    }

    .buttons {
      margin-top: 10px;
    }
  }
</style>
