# 713-ctrl.github.jo

<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Diary</title>

<style>
body {
  background-color: #f7f3ec;
  font-family: "Hiragino Maru Gothic ProN", sans-serif;
  display: flex;
  justify-content: center;
}

.container {
  background: #fffdf8;
  width: 340px;
  padding: 20px;
  margin-top: 20px;
  border-radius: 15px;
}

/* カレンダー */
.calendar {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 5px;
  margin-bottom: 10px;
}

.day {
  padding: 8px;
  text-align: center;
  border-radius: 8px;
  background: #eee;
  cursor: pointer;
  font-size: 12px;
}

.week {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  text-align: center;
  font-size: 12px;
  margin-bottom: 5px;
}

.empty {
  background: transparent;
  cursor: default;
}

.active {
  background: #d8cfc4;
}

.hasData {
  background: #ffe4e1; /* 書いた日 */
}

textarea {
  width: 100%;
  height: 100px;
  border-radius: 10px;
}

</style>
</head>

<body>

<div class="container">

<div style="display:flex; justify-content:space-between; align-items:center;">
  <button onclick="prevMonth()">◀︎</button>
  <h3 id="monthTitle"></h3>
  <button onclick="nextMonth()">▶︎</button>
</div>


<div class="week">
  <div>Mon</div><div>Tue</div><div>Wed</div>
  <div>Thu</div><div>Fri</div><div>Sat</div><div>Sun</div>
</div>
<div class="calendar" id="calendar"></div>


<h3 id="dateTitle"></h3>

<textarea id="diary"></textarea>

<input type="file" id="imageInput" accept="image/*">
<div id="imagePreview"></div>

<br>

<h3>✔ ToDo</h3>
<input type="text" id="todoInput" placeholder="やることを書く">
<button onclick="addTodo()">追加</button>
<ul id="todoList"></ul>

<button onclick="saveData()">保存</button>

</div>

<script>
let currentDate = "";
let year, month;

// カレンダー作成
function createCalendar() {
  const calendar = document.getElementById("calendar");
  calendar.innerHTML = "";

  const firstDay = new Date(year, month, 1).getDay();
  const lastDate = new Date(year, month + 1, 0).getDate();

  const monthNames = [
  "January","February","March","April","May","June",
  "July","August","September","October","November","December"
];

document.getElementById("monthTitle").innerText =
  monthNames[month] + " " + year;

  // 空白
  for (let i = 0; i < firstDay; i++) {
    const empty = document.createElement("div");
    empty.className = "empty";
    calendar.appendChild(empty);
  }

  // 日付
  for (let i = 1; i <= lastDate; i++) {
    const dateStr = year + "-" + (month+1) + "-" + i;

    const div = document.createElement("div");
    div.className = "day";
    div.innerText = i;

    // データがある日だけ色つけ
    if (localStorage.getItem(dateStr + "_diary")) {
      div.classList.add("hasData");
    }

    div.onclick = () => loadData(dateStr, div);

    calendar.appendChild(div);
  }
}


function prevMonth() {
  month--;
  if (month < 0) {
    month = 11;
    year--;
  }
  createCalendar();
}
function nextMonth() {
  month++;
  if (month > 11) {
    month = 0;
    year++;
  }
  createCalendar();
}



let todos = [];
// ToDo追加
function addTodo() {
  const input = document.getElementById("todoInput");
  const text = input.value;

  if (text === "") return;

  todos.push({ text: text, done: false });
  input.value = "";

  renderTodos();
}
// 表示更新
function renderTodos() {
  const list = document.getElementById("todoList");
  list.innerHTML = "";

  todos.forEach((todo, index) => {
    const li = document.createElement("li");

    li.innerHTML = `
      <input type="checkbox" ${todo.done ? "checked" : ""} onchange="toggleTodo(${index})">
      ${todo.text}
      <button onclick="deleteTodo(${index})" style="margin-left:10px;">🗑</button>
    `;

    list.appendChild(li);
  });
}

function deleteTodo(index) {
  todos.splice(index, 1); // 配列から削除
  renderTodos(); // 表示更新
}

// チェック切り替え
function toggleTodo(index) {
  todos[index].done = !todos[index].done;
}



let imageData = "";
// 画像選択
document.getElementById("imageInput").addEventListener("change", function(e) {
  const file = e.target.files[0];
  const reader = new FileReader();

  reader.onload = function(event) {
    imageData = event.target.result;
    document.getElementById("imagePreview").innerHTML =
      `<img src="${imageData}" style="width:100%; border-radius:10px;">`;
  };

  if (file) {
    reader.readAsDataURL(file);
  }
});



// データ読み込み
function loadData(date, element) {
  currentDate = date;

  document.getElementById("dateTitle").innerText = "📔 " + date;

  document.getElementById("diary").value =
    localStorage.getItem(date + "_diary") || "";

  document.getElementById("todo1").checked =
    localStorage.getItem(date + "_todo1") === "true";

  document.getElementById("todo2").checked =
    localStorage.getItem(date + "_todo2") === "true";

  document.querySelectorAll(".day").forEach(d => d.classList.remove("active"));
  element.classList.add("active");

　imageData = localStorage.getItem(date + "_image") || "";

if (imageData) {
  document.getElementById("imagePreview").innerHTML =
    `<img src="${imageData}" style="width:100%; border-radius:10px;">`;
} else {
  document.getElementById("imagePreview").innerHTML = "";
}

  todos = JSON.parse(localStorage.getItem(date + "_todos")) || [];
renderTodos();
}

// 保存
function saveData() {
  localStorage.setItem(currentDate + "_diary", document.getElementById("diary").value);

  localStorage.setItem(currentDate + "_todos", JSON.stringify(todos));

　localStorage.setItem(currentDate + "_image", imageData);

  alert("保存完了");
  createCalendar();
}

// 初期化
window.onload = function() {
  const today = new Date();
  year = today.getFullYear();
  month = today.getMonth();

  createCalendar();

  const todayStr = year + "-" + (month+1) + "-" + today.getDate();
  const firstDay = document.querySelector(".day");

  loadData(todayStr, firstDay);
}
</script>

</body>
</html>
