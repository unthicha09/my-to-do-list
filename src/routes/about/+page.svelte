<script>
  import { onMount } from "svelte";
  import { Calendar } from "@fullcalendar/core";
  import dayGridPlugin from "@fullcalendar/daygrid";
  import interactionPlugin from "@fullcalendar/interaction";

  let calendar;

  // ดึงข้อมูล Todos ที่บันทึกใน LocalStorage
  let todos = [];
  onMount(() => {
    const saved = localStorage.getItem("todos");
    if (saved) {
      todos = JSON.parse(saved);
    }
    initializeCalendar();
  });

  function initializeCalendar() {
    const calendarEl = document.getElementById("calendar");

    calendar = new Calendar(calendarEl, {
      plugins: [dayGridPlugin, interactionPlugin],
      initialView: "dayGridMonth",
      events: todos.map((todo) => ({
        title: todo.task,
        date: todo.due,
        backgroundColor: "#7EC8E3", // สีฟ้าอ่อนสำหรับกิจกรรม
        borderColor: "#7EC8E3", // กรอบสีเดียวกับพื้นหลัง
        textColor: "#333", // สีตัวอักษร
      })),
      dateClick: (info) => {
        alert("You clicked on " + info.dateStr);
      },
    });

    calendar.render();
  }
</script>

<div class="about-container">
  <h1>📅 ปฏิทินกิจกรรม</h1>
  <div id="calendar"></div>
</div>

<style>
  .about-container {
    padding: 24px;
    text-align: center;
  }

  #calendar {
    margin-top: 20px;
    max-width: 100%;
    max-height: 80vh;
    margin: 0 auto;
  }
</style>
