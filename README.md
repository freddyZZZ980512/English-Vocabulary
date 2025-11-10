# English-Vocabulary
<!-- --- 以下為可直接貼上的完整範例（HTML + Script） --- -->
<div>
  <h2>新增單字</h2>
  <input id="enInput" placeholder="英文單字 (apple)">
  <input id="zhInput" placeholder="中文意思 (蘋果)">
  <input id="setInput" placeholder="單字集 (例如: 水果)">
  <button id="addBtn">新增</button>
</div>

<div style="margin-top:20px;">
  <p>今天待背單字：<span id="todayCount">0</span></p>
  <button id="startBtn">開始今天單字</button>
</div>

<div id="reviewArea" style="display:none; margin-top:16px;">
  <p id="progressText"></p>
  <h3 id="reviewWord">單字</h3>
  <p id="reviewZh">中文</p>
  <button id="correctBtn">✔ 會</button>
  <button id="wrongBtn">✖ 不會</button>
</div>

<ul id="wordList" style="margin-top:24px;"></ul>

<script>
/* utility: date helpers */
function toDateStr(d){
  // return YYYY-MM-DD
  const y = d.getFullYear();
  const m = String(d.getMonth()+1).padStart(2,'0');
  const dd = String(d.getDate()).padStart(2,'0');
  return `${y}-${m}-${dd}`;
}
function addDaysStr(dateStr, days){
  const d = new Date(dateStr + "T00:00:00");
  d.setDate(d.getDate() + days);
  return toDateStr(d);
}
function todayStr(){ return toDateStr(new Date()); }

/* storage */
const STORAGE_KEY = "myWords_v1";
let words = JSON.parse(localStorage.getItem(STORAGE_KEY)) || [];

function saveWords(){
  localStorage.setItem(STORAGE_KEY, JSON.stringify(words));
}

/* create schedule: today, +3, +7, +20 */
function createSchedule(){
  const t = todayStr();
  return [ t, addDaysStr(t,3), addDaysStr(t,7), addDaysStr(t,20) ];
}

/* add a new word */
function addWord(en, zh, setName){
  const id = Date.now().toString();
  const schedule = createSchedule();
  const w = {
    id,
    en,
    zh,
    set: setName,
    schedule,
    currentStage: 0,
    nextReview: schedule[0] // 預設今天
  };
  words.push(w);
  saveWords();
  renderWordList();
  renderTodayCount();
}

/* load today's words */
function getTodayWords(){
  const t = todayStr();
  return words.filter(w => w.nextReview === t);
}

/* render functions */
function renderTodayCount(){
  document.getElementById("todayCount").textContent = getTodayWords().length;
}
function renderWordList(){
  const list = document.getElementById("wordList");
  list.innerHTML = "";
  words.forEach((w, idx) => {
    const li = document.createElement("li");
    li.innerHTML = `${w.en} - ${w.zh} （集:${w.set}） 下次: ${w.nextReview}
      <button onclick="deleteWord('${w.id}')">刪除</button>`;
    list.appendChild(li);
  });
}

/* delete */
function deleteWord(id){
  if(!confirm("確定要刪除？")) return;
  words = words.filter(w => w.id !== id);
  saveWords();
  renderWordList();
  renderTodayCount();
}

/* Review flow */
let reviewQueue = [];
let reviewIndex = 0;

function startTodayReview(){
  reviewQueue = getTodayWords();
  reviewIndex = 0;
  if(reviewQueue.length === 0){
    alert("今天沒有需要複習的單字！");
    return;
  }
  document.getElementById("reviewArea").style.display = "block";
  showReviewCard();
  updateProgressText();
}

function showReviewCard(){
  const w = reviewQueue[reviewIndex];
  document.getElementById("reviewWord").textContent = w.en;
  document.getElementById("reviewZh").textContent = w.zh;
  updateProgressText();
}

function updateProgressText(){
  document.getElementById("progressText").textContent =
    `第 ${reviewIndex+1} / ${reviewQueue.length}`;
}

function handleCorrect(){
  const w = reviewQueue[reviewIndex];
  // 找到全局 words 裡的物件並更新
  const globalW = words.find(x => x.id === w.id);
  if(!globalW) return;
  // 進入下一階段
  globalW.currentStage = Math.min(globalW.currentStage + 1, globalW.schedule.length);
  if(globalW.currentStage < globalW.schedule.length){
    globalW.nextReview = globalW.schedule[globalW.currentStage];
  } else {
    // 全部階段完成 -> 設為已完成 (不再加入今日列表)
    globalW.nextReview = null;
  }
  saveWords();
  moveToNextReview();
}

function handleWrong(){
  const w = reviewQueue[reviewIndex];
  const globalW = words.find(x => x.id === w.id);
  if(!globalW) return;
  // 若不會，立刻安排為明天再出現
  globalW.nextReview = addDaysStr(todayStr(), 1);
  // 注意：currentStage 不變，等明天再給一次機會
  saveWords();
  moveToNextReview();
}

function moveToNextReview(){
  reviewIndex++;
  renderTodayCount();
  if(reviewIndex >= reviewQueue.length){
    // 完成今天所有題目
    document.getElementById("reviewArea").style.display = "none";
    alert("恭喜！今天的單字已練習完畢。");
    renderWordList();
  } else {
    showReviewCard();
  }
}

/* wire up UI */
document.getElementById("addBtn").addEventListener("click", () => {
  const en = document.getElementById("enInput").value.trim();
  const zh = document.getElementById("zhInput").value.trim();
  const setName = document.getElementById("setInput").value.trim();
  if(!en || !zh || !setName) return alert("請輸入完整欄位");
  addWord(en, zh, setName);
  // clear inputs
  document.getElementById("enInput").value = "";
  document.getElementById("zhInput").value = "";
  document.getElementById("setInput").value = "";
});

document.getElementById("startBtn").addEventListener("click", startTodayReview);
document.getElementById("correctBtn").addEventListener("click", handleCorrect);
document.getElementById("wrongBtn").addEventListener("click", handleWrong);

/* on load */
renderWordList();
renderTodayCount();

</script>
