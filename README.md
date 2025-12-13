<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8" />
<title>kansaii GPT</title>
<style>
body {
  font-family: -apple-system, BlinkMacSystemFont, sans-serif;
  padding: 14px;
}
select, textarea, button {
  width: 100%;
  font-size: 16px;
  margin-bottom: 8px;
}
button {
  padding: 10px;
  font-weight: bold;
}
pre {
  white-space: pre-wrap;
  background: #f3f3f3;
  padding: 10px;
}
</style>
</head>

<body>
<h2>🗣 kansaii GPT</h2>

<select id="persona">
  <option value="gyaru">💅 関西ギャル</option>
  <option value="man">😎 関西兄ちゃん</option>
</select>

<textarea id="q" rows="4" placeholder="聞きたいこと書いてな"></textarea>
<button onclick="send()">送信</button>
<button onclick="clearAll()">記憶リセット</button>

<pre id="a"></pre>

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

/* ===== 人格 ===== */
function personaPrompt(type) {
  if (type === "man") {
    return `
あなたは関西弁で話す男のAI。
落ち着いてて兄ちゃん気質。
分かりやすく、無駄に煽らず、的確。
`;
  }
  return `
あなたは関西弁オンリーのノリ強めギャルAI。
距離感近く、テンポ良くツッコミ入れる。
`;
}

/* ===== 複合思考（正式実装） ===== */
function systemPrompt() {
  const persona = personaPrompt(document.getElementById("persona").value);
  return `
あなたは統合司令塔AI「kansaii GPT」。

内部では以下の3視点で思考せよ（出力は統合結果のみ）。

【GPT視点】
論理・網羅・正確性を重視。

【Gemini視点】
整理力・初心者への分かりやすさ重視。

【Grok視点】
本質的ツッコミ・大胆な仮説。

${persona}

【記憶メモ】
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

  history.push({ role: "assistant", content: ans });
  localStorage.setItem("CHAT_HISTORY", JSON.stringify(history));
  document.getElementById("a").textContent = ans;

  learn(q, ans);
}

/* ===== 疑似学習 ===== */
function learn(u, a) {
  memory += `\n・${u.slice(0,30)} → ${a.slice(0,30)}`;
  memory = memory.split("\n").slice(-20).join("\n");
  localStorage.setItem("KANSAl_MEMORY", memory);
}

/* ===== 全消し ===== */
function clearAll() {
  if (confirm("全部消すで？")) {
    localStorage.clear();
    history = [];
    memory = "";
    document.getElementById("a").textContent = "リセット完了や";
  }
}
</script>
</body>
</html>