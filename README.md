<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8" />
<title>kansaii GPT</title>
<style>
body {
  font-family: -apple-system, BlinkMacSystemFont, sans-serif;
  margin: 0;
  background: #e5ddd5;
}
header {
  background: #075e54;
  color: #fff;
  padding: 12px;
  text-align: center;
  font-weight: bold;
}
#chat {
  padding: 10px;
  height: calc(100vh - 160px);
  overflow-y: auto;
}
.bubble {
  max-width: 80%;
  padding: 10px;
  margin: 6px 0;
  border-radius: 10px;
  line-height: 1.4;
  white-space: pre-wrap;
}
.user {
  background: #dcf8c6;
  margin-left: auto;
}
.ai {
  background: #fff;
  margin-right: auto;
}
#controls {
  padding: 8px;
  background: #f0f0f0;
}
textarea, select, button {
  width: 100%;
  font-size: 16px;
  margin-bottom: 6px;
}
button {
  padding: 10px;
  font-weight: bold;
}
</style>
</head>

<body>
<header>🗣 kansaii GPT</header>

<div id="chat"></div>

<div id="controls">
  <select id="persona">
    <option value="gyaru">💅 関西ギャル</option>
    <option value="man">😎 関西兄ちゃん</option>
  </select>

  <textarea id="q" rows="2" placeholder="メッセージ入力"></textarea>
  <button onclick="send()">送信</button>
  <button onclick="clearAll()">履歴リセット</button>
</div>

<script>
/* ===== APIキー ===== */
function getKey() {
  let k = localStorage.getItem("OPENAI_KEY");
  if (!k) {
    k = prompt("初回だけAPIキー入れてな（sk-...）");
    if (k) localStorage.setItem("OPENAI_KEY", k);
  }
  return k;
}

/* ===== 保存 ===== */
let history = JSON.parse(localStorage.getItem("CHAT_HISTORY") || "[]");
let memory = localStorage.getItem("KANSAl_MEMORY") || "";

/* ===== 表示 ===== */
const chat = document.getElementById("chat");

function addBubble(text, cls) {
  const div = document.createElement("div");
  div.className = "bubble " + cls;
  div.textContent = text;
  chat.appendChild(div);
  chat.scrollTop = chat.scrollHeight;
}

/* ===== 人格 ===== */
function personaPrompt(type) {
  return type === "man"
    ? "あなたは落ち着いた関西弁の兄ちゃんAI。的確で分かりやすい。"
    : "あなたはノリ良し関西ギャルAI。テンポ良くツッコむ。";
}

/* ===== 複合思考 ===== */
function systemPrompt() {
  return `
あなたは統合司令塔AI「kansaii GPT」。

【GPT視点】論理と正確性
【Gemini視点】整理と分かりやすさ
【Grok視点】本質ツッコミ

${personaPrompt(document.getElementById("persona").value)}

【記憶】
${memory}
`;
}

/* ===== 送信 ===== */
async function send() {
  const key = getKey();
  if (!key) return;

  const q = document.getElementById("q").value;
  document.getElementById("q").value = "";

  history.push({ role: "user", content: q });
  addBubble(q, "user");

  // 考え中表示
  const thinking = document.createElement("div");
  thinking.className = "bubble ai";
  thinking.textContent = "考え中……🧠";
  chat.appendChild(thinking);
  chat.scrollTop = chat.scrollHeight;

  const res = await fetch("https://api.openai.com/v1/chat/completions", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": "Bearer " + key
    },
    body: JSON.stringify({
      model: "gpt-4o-mini",
      messages: [
        { role: "system", content: systemPrompt() },
        ...history
      ]
    })
  });

  const data = await res.json();
  const ans = data.choices?.[0]?.message?.content || "エラーやで";

  chat.removeChild(thinking);
  addBubble(ans, "ai");

  history.push({ role: "assistant", content: ans });
  localStorage.setItem("CHAT_HISTORY", JSON.stringify(history));

  learn(q, ans);
}

/* ===== 疑似学習 ===== */
function learn(u, a) {
  memory += `\n・${u.slice(0,30)} → ${a.slice(0,30)}`;
  memory = memory.split("\n").slice(-20).join("\n");
  localStorage.setItem("KANSAl_MEMORY", memory);
}

/* ===== リセット ===== */
function clearAll() {
  if (confirm("履歴全部消すで？")) {
    localStorage.clear();
    chat.innerHTML = "";
    history = [];
    memory = "";
  }
}
</script>
</body>
</html>