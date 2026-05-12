# AI-Board 初期実装（Vanilla JavaScript版）

## index.html

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>AI-Board</title>

  <link rel="stylesheet" href="style.css" />

  <script src="https://cdn.jsdelivr.net/npm/sortablejs@1.15.0/Sortable.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/papaparse@5.4.1/papaparse.min.js"></script>
</head>
<body>

<header>
  <h1>AI-Board</h1>

  <div class="top-right">
    <div id="currentTime"></div>

    <button id="menuBtn">☰</button>
  </div>
</header>

<div id="hamburgerMenu" class="hidden">
  <button onclick="showPage('settings')">設定</button>
</div>

<main>

<section id="home" class="page active">
  <h2>本日の授業予定</h2>

  <div id="todayLessons"></div>
</section>

<section id="lessonPlans" class="page">

  <h2>授業計画</h2>

  <input id="lessonTitle" placeholder="授業名" />

  <input id="className" placeholder="クラス名（中1Aなど）" />

  <input type="date" id="lessonDate" />

  <select id="lessonPeriod"></select>

  <textarea
    id="criteriaInput"
    placeholder="作成した評価基準を貼り付けてください"
  ></textarea>

  <button id="generateBtn">AIで観察ポイント生成</button>

  <div id="keywordArea"></div>

  <input id="customKeyword" placeholder="追加キーワード" />

  <button id="regenerateBtn">再生成</button>

  <div id="observationPoints"></div>

  <select id="seatMapSelect"></select>

  <button id="saveLessonBtn">この計画を保存</button>

</section>

<section id="seatEditor" class="page">

  <h2>座席表</h2>

  <input id="seatClassName" placeholder="クラス名" />

  <div class="seat-settings">
    <label>縦</label>
    <input type="number" id="rows" value="5" />

    <label>横</label>
    <input type="number" id="cols" value="6" />
  </div>

  <div id="seatCount"></div>

  <textarea
    id="studentNames"
    placeholder="生徒名を改行で入力"
  ></textarea>

  <button id="createSeatBtn">座席表作成</button>

  <div id="seatPreview"></div>

  <button id="saveSeatBtn">保存</button>

  <div id="savedSeatMaps"></div>

</section>

<section id="lessonStart" class="page">

  <h2>授業開始</h2>

  <div id="lessonStartList"></div>

  <div id="activeLessonArea"></div>

</section>

<section id="library" class="page">

  <h2>保存済み授業</h2>

  <div id="savedLessons"></div>

</section>

<section id="settings" class="page">

  <h2>設定</h2>

  <div class="settings-tabs">
    <button onclick="showSettingTab('time')">時間割</button>
    <button onclick="showSettingTab('actions')">評価アクション</button>
  </div>

  <div id="timeSettings" class="setting-content active-setting">

    <div id="periodSettings"></div>

    <button id="addPeriodBtn">時間追加</button>

  </div>

  <div id="actionSettings" class="setting-content">

    <h3>良い行動</h3>

    <div id="goodActions"></div>

    <input id="newGoodAction" placeholder="追加" />

    <button onclick="addAction('good')">追加</button>

    <h3>悪い行動</h3>

    <div id="badActions"></div>

    <input id="newBadAction" placeholder="追加" />

    <button onclick="addAction('bad')">追加</button>

  </div>

</section>

</main>

<nav>
  <button onclick="showPage('home')">ホーム</button>
  <button onclick="showPage('lessonStart')">授業開始</button>
  <button onclick="showPage('lessonPlans')">授業計画</button>
  <button onclick="showPage('seatEditor')">座席表</button>
  <button onclick="showPage('library')">ライブラリ</button>
</nav>

<script src="app.js"></script>

</body>
</html>
```

---

# style.css

```css
body {
  margin: 0;
  font-family: sans-serif;
  background: #f3fff5;
  color: #333;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  background: #b7e4c7;
}

main {
  padding: 20px;
  padding-bottom: 90px;
}

.page {
  display: none;
}

.active {
  display: block;
}

nav {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  display: flex;
  background: #95d5b2;
}

nav button {
  flex: 1;
  border: none;
  padding: 15px;
  background: transparent;
}

textarea {
  width: 100%;
  min-height: 160px;
  margin-top: 10px;
}

input,
select,
button {
  width: 100%;
  padding: 10px;
  margin-top: 10px;
  border-radius: 10px;
  border: 1px solid #ccc;
  box-sizing: border-box;
}

button {
  background: #95d5b2;
}

#seatPreview {
  display: grid;
  gap: 10px;
  margin-top: 20px;
}

.seat {
  background: white;
  padding: 15px;
  border-radius: 12px;
  text-align: center;
  cursor: move;
}

.lesson-card {
  background: white;
  padding: 15px;
  margin-top: 10px;
  border-radius: 15px;
}

.action-btn {
  margin: 5px;
}

.hidden {
  display: none;
}

#hamburgerMenu {
  position: absolute;
  right: 10px;
  top: 70px;
  background: white;
  padding: 10px;
  border-radius: 10px;
}

@media (min-width: 900px) {

  main {
    max-width: 1200px;
    margin: auto;
  }

}
```

---

# app.js

```javascript
const store = {

  lessons: [],
  seatMaps: [],
  evaluations: [],

  settings: {

    periods: [
      {
        name: '1時間目',
        start: '08:30',
        end: '09:20'
      }
    ],

    actions: {
      good: ['発言', '教え合い'],
      bad: ['私語', '離席']
    }
  }
};

function saveStore() {
  localStorage.setItem('ai-board', JSON.stringify(store));
}

function loadStore() {

  const data = localStorage.getItem('ai-board');

  if (!data) return;

  const parsed = JSON.parse(data);

  Object.assign(store, parsed);
}

loadStore();

function showPage(id) {

  document.querySelectorAll('.page').forEach(page => {
    page.classList.remove('active');
  });

  document.getElementById(id).classList.add('active');

  renderAll();
}

function renderAll() {

  renderPeriods();
  renderSeatMapOptions();
  renderSavedLessons();
  renderSeatMaps();
  renderLessonStart();
  renderTodayLessons();
}

function updateClock() {

  const now = new Date();

  document.getElementById('currentTime').innerText =
    now.toLocaleTimeString('ja-JP');
}

setInterval(updateClock, 1000);
updateClock();

const menuBtn = document.getElementById('menuBtn');

menuBtn.onclick = () => {

  document
    .getElementById('hamburgerMenu')
    .classList.toggle('hidden');
};

function renderPeriods() {

  const select = document.getElementById('lessonPeriod');

  select.innerHTML = '';

  store.settings.periods.forEach(period => {

    const option = document.createElement('option');

    option.value = period.name;
    option.innerText = period.name;

    select.appendChild(option);
  });

  const settings = document.getElementById('periodSettings');

  settings.innerHTML = '';

  store.settings.periods.forEach((period, index) => {

    const div = document.createElement('div');

    div.innerHTML = `
      <input value="${period.name}" onchange="changePeriodName(${index}, this.value)">
      <input type="time" value="${period.start}" onchange="changePeriodStart(${index}, this.value)">
      <input type="time" value="${period.end}" onchange="changePeriodEnd(${index}, this.value)">
    `;

    settings.appendChild(div);
  });
}

function changePeriodName(index, value) {
  store.settings.periods[index].name = value;
  saveStore();
}

function changePeriodStart(index, value) {
  store.settings.periods[index].start = value;
  saveStore();
}

function changePeriodEnd(index, value) {
  store.settings.periods[index].end = value;
  saveStore();
}

document.getElementById('addPeriodBtn').onclick = () => {

  store.settings.periods.push({
    name: '追加時間',
    start: '09:30',
    end: '10:20'
  });

  saveStore();
  renderPeriods();
};

function showSettingTab(type) {

  document
    .querySelectorAll('.setting-content')
    .forEach(el => el.classList.remove('active-setting'));

  if (type === 'time') {
    document
      .getElementById('timeSettings')
      .classList.add('active-setting');
  }

  if (type === 'actions') {
    document
      .getElementById('actionSettings')
      .classList.add('active-setting');
  }
}

function addAction(type) {

  const input = document.getElementById(
    type === 'good'
      ? 'newGoodAction'
      : 'newBadAction'
  );

  if (!input.value) return;

  store.settings.actions[type].push(input.value);

  input.value = '';

  saveStore();

  renderActions();
}

function renderActions() {

  const good = document.getElementById('goodActions');
  const bad = document.getElementById('badActions');

  good.innerHTML = '';
  bad.innerHTML = '';

  store.settings.actions.good.forEach(action => {

    const div = document.createElement('div');
    div.innerText = action;

    good.appendChild(div);
  });

  store.settings.actions.bad.forEach(action => {

    const div = document.createElement('div');
    div.innerText = action;

    bad.appendChild(div);
  });
}

renderActions();

const createSeatBtn = document.getElementById('createSeatBtn');

createSeatBtn.onclick = () => {

  const rows = Number(document.getElementById('rows').value);
  const cols = Number(document.getElementById('cols').value);

  const students = document
    .getElementById('studentNames')
    .value
    .split('\n')
    .filter(Boolean);

  document.getElementById('seatCount').innerText =
    `総人数 ${rows * cols}`;

  const preview = document.getElementById('seatPreview');

  preview.innerHTML = '';

  preview.style.gridTemplateColumns = `repeat(${cols}, 1fr)`;

  students.forEach(name => {

    const div = document.createElement('div');

    div.className = 'seat';
    div.innerText = name;

    preview.appendChild(div);
  });

  new Sortable(preview, {
    animation: 150
  });
};

document.getElementById('saveSeatBtn').onclick = () => {

  const className = document.getElementById('seatClassName').value;

  const seats = [...document.querySelectorAll('.seat')]
    .map(el => el.innerText);

  store.seatMaps.push({
    className,
    seats
  });

  saveStore();

  renderSeatMaps();
  renderSeatMapOptions();
};

function renderSeatMaps() {

  const area = document.getElementById('savedSeatMaps');

  area.innerHTML = '';

  store.seatMaps.forEach(map => {

    const div = document.createElement('div');

    div.className = 'lesson-card';

    div.innerHTML = `
      <h3>${map.className}</h3>
      <button>編集</button>
    `;

    area.appendChild(div);
  });
}

function renderSeatMapOptions() {

  const select = document.getElementById('seatMapSelect');

  select.innerHTML = '';

  store.seatMaps.forEach((map, index) => {

    const option = document.createElement('option');

    option.value = index;
    option.innerText = map.className;

    select.appendChild(option);
  });
}

async function generateObservationPoints(text) {

  const API_KEY = 'YOUR_GEMINI_API_KEY';

  const prompt = `
以下の評価基準から、
授業中に観察できる行動を3つ作成。

条件:
・20文字以内
・「〜している」
・解説禁止

評価基準:
${text}
  `;

  const response = await fetch(
    `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${API_KEY}`,
    {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        contents: [{
          parts: [{ text: prompt }]
        }]
      })
    }
  );

  const data = await response.json();

  return data
    .candidates[0]
    .content
    .parts[0]
    .text;
}

document.getElementById('generateBtn').onclick = async () => {

  const text = document.getElementById('criteriaInput').value;

  const result = await generateObservationPoints(text);

  document.getElementById('observationPoints').innerText = result;
};

document.getElementById('regenerateBtn').onclick = async () => {

  const text = document.getElementById('criteriaInput').value;

  const custom = document.getElementById('customKeyword').value;

  const result = await generateObservationPoints(
    text + '\n注目ポイント:' + custom
  );

  document.getElementById('observationPoints').innerText = result;
};

document.getElementById('saveLessonBtn').onclick = () => {

  const lesson = {

    title: document.getElementById('lessonTitle').value,

    className: document.getElementById('className').value,

    date: document.getElementById('lessonDate').value,

    period: document.getElementById('lessonPeriod').value,

    criteria: document.getElementById('criteriaInput').value,

    observations:
      document.getElementById('observationPoints').innerText,

    seatMap:
      document.getElementById('seatMapSelect').value
  };

  store.lessons.push(lesson);

  saveStore();

  renderSavedLessons();
  renderLessonStart();

  showPage('library');
};

function renderSavedLessons() {

  const area = document.getElementById('savedLessons');

  area.innerHTML = '';

  store.lessons.forEach((lesson, index) => {

    const div = document.createElement('div');

    div.className = 'lesson-card';

    div.innerHTML = `
      <h3>${lesson.title}</h3>
      <p>${lesson.className}</p>
      <p>${lesson.date}</p>
      <button onclick="downloadLessonCSV(${index})">CSV</button>
    `;

    area.appendChild(div);
  });
}

function renderLessonStart() {

  const area = document.getElementById('lessonStartList');

  area.innerHTML = '';

  store.lessons.forEach((lesson, index) => {

    const div = document.createElement('div');

    div.className = 'lesson-card';

    div.innerHTML = `
      <h3>${lesson.title}</h3>
      <button onclick="startLesson(${index})">開始</button>
    `;

    area.appendChild(div);
  });
}

function startLesson(index) {

  const lesson = store.lessons[index];

  const seatMap = store.seatMaps[lesson.seatMap];

  const area = document.getElementById('activeLessonArea');

  area.innerHTML = `
    <h3>${lesson.title}</h3>
    <p>${lesson.observations}</p>
    <div id="lessonSeats"></div>
  `;

  const seatArea = document.getElementById('lessonSeats');

  seatArea.style.display = 'grid';
  seatArea.style.gridTemplateColumns = 'repeat(6, 1fr)';
  seatArea.style.gap = '10px';

  seatMap.seats.forEach(student => {

    const div = document.createElement('div');

    div.className = 'seat';

    const buttons = [
      ...store.settings.actions.good,
      ...store.settings.actions.bad
    ]
    .map(action => {

      return `
        <button
          class="action-btn"
          onclick="recordEvaluation('${student}','${action}','${lesson.title}')"
        >
          ${action}
        </button>
      `;
    })
    .join('');

    div.innerHTML = `
      <strong>${student}</strong>
      ${buttons}
    `;

    seatArea.appendChild(div);
  });
}

function recordEvaluation(student, action, lesson) {

  store.evaluations.push({
    student,
    action,
    lesson,
    timestamp: new Date().toISOString()
  });

  saveStore();
}

function renderTodayLessons() {

  const area = document.getElementById('todayLessons');

  area.innerHTML = '';

  const today = new Date()
    .toISOString()
    .split('T')[0];

  store.lessons
    .filter(lesson => lesson.date === today)
    .forEach(lesson => {

      const div = document.createElement('div');

      div.className = 'lesson-card';

      div.innerHTML = `
        <h3>${lesson.title}</h3>
        <p>${lesson.period}</p>
      `;

      area.appendChild(div);
    });
}

function downloadLessonCSV(index) {

  const lesson = store.lessons[index];

  const data = store.evaluations
    .filter(e => e.lesson === lesson.title);

  const csv = Papa.unparse(data);

  const blob = new Blob([csv], {
    type: 'text/csv'
  });

  const a = document.createElement('a');

  a.href = URL.createObjectURL(blob);

  a.download = `${lesson.title}.csv`;

  a.click();
}

renderAll();
```

---
