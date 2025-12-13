<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>kansaii GPT</title>

<style>
* { -webkit-text-size-adjust: 100%; }
body { margin:0; background:#e5ddd5; font-family:-apple-system,BlinkMacSystemFont,sans-serif; }
header { background:#075e54; color:#fff; padding:12px; text-align:center; font-weight:bold; }
#chat { padding:10px; height:calc(100svh - 170px); overflow-y:auto; }

.message { display:flex; margin:6px 0; align-items:flex-end; }
.message.user { flex-direction:row-reverse; }

.icon { font-size:26px; margin:0 6px; }
.bubble { max-width:75%; padding:10px; border-radius:12px; white-space:pre-wrap; }

.user .bubble { background:#dcf8c6; }
.ai .bubble { background:#fff; }

#controls { padding:8px; background:#f0f0f0; }
select, textarea, button { width:100%; font-size:16px; margin-bottom:6px; }
button { padding:10px; font-weight:bold; }
button:disabled { opacity:.5; }
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

<textarea id="q" rows="2" placeholder="Enterで送信 / Shift+Enterで改行"></textarea>
<button id="sendBtn" onclick="send()">送信</button>
<button onclick="resetPersona()">このキャラをリセット</button>
</div>

<script>
/* ===== Enter送信 ===== */
q.addEventListener("keydown", e=>{
  if(e.key==="Enter" && !e.shiftKey){ e.preventDefault(); send(); }
});

/* ===== APIキー ===== */
function getKey(){
  let k=localStorage.getItem("OPENAI_KEY");
  if(!k){ k=prompt("初回だけAPIキー入れてな"); if(k) localStorage.setItem("OPENAI_KEY",k); }
  return k;
}

/* ===== キャラ定義 ===== */
const PERSONA = {
  gyaru: { icon:"💅", name:"ギャルGPT",
    prompt:`関西弁ギャル。ノリ良しテンポ良し。ツッコミ多め。距離感近い。`
  },
  man: { icon:"😎", name:"兄ちゃんGPT",
    prompt:`落ち着いた関西弁の兄ちゃん。冷静で分かりやすい。`
  }
};

/* ===== キャラ別保存 ===== */
function key(name){ return PERSONA_SEL+"_"+name; }
let PERSONA_SEL = localStorage.getItem("CURRENT_PERSONA") || "gyaru";

persona.value = PERSONA_SEL;

let history = JSON.parse(localStorage.getItem(key("HISTORY")) || "[]");
let memory  = localStorage.getItem(key("MEMORY")) || "";

/* ===== UI ===== */
const chat=document.getElementById("chat");
const sendBtn=document.getElementById("sendBtn");

function addMsg(text,who){
  const d=document.createElement("div");
  d.className="message "+who;
  d.innerHTML=`<div class="icon">${who==="user"?"🧑":PERSONA[PERSONA_SEL].icon}</div>
               <div class="bubble">${text}</div>`;
  chat.appendChild(d); chat.scrollTop=chat.scrollHeight;
  return d.querySelector(".bubble");
}

/* ===== キャラ切替 ===== */
persona.onchange=()=>{
  localStorage.setItem("CURRENT_PERSONA",persona.value);
  location.reload();
};

/* ===== システム ===== */
function systemPrompt(){
  return `
あなたは統合AI「kansaii GPT」。
【GPT】論理【Gemini】整理【Grok】本質

キャラ設定:
${PERSONA[PERSONA_SEL].prompt}

【記憶】
${memory}
`;
}

/* ===== 送信 ===== */
async function send(){
  const keyApi=getKey(); if(!keyApi) return;
  const qText=q.value.trim(); if(!qText) return;
  q.value="";

  history.push({role:"user",content:qText});
  addMsg(qText,"user");

  sendBtn.disabled=true; sendBtn.textContent="送信中…";
  const thinking=addMsg("考え中……🧠","ai");

  const res=await fetch("https://api.openai.com/v1/chat/completions",{
    method:"POST",
    headers:{ "Content-Type":"application/json","Authorization":"Bearer "+keyApi },
    body:JSON.stringify({
      model:"gpt-4o-mini",
      messages:[{role:"system",content:systemPrompt()},...history]
    })
  });

  const data=await res.json();
  const ans=data.choices?.[0]?.message?.content || "エラーやで";

  thinking.textContent=ans;
  history.push({role:"assistant",content:ans});
  localStorage.setItem(key("HISTORY"),JSON.stringify(history));

  learn(qText,ans);
  sendBtn.disabled=false; sendBtn.textContent="送信";
}

/* ===== 学習 ===== */
function learn(u,a){
  memory+=`\n・${u.slice(0,30)} → ${a.slice(0,30)}`;
  memory=memory.split("\n").slice(-20).join("\n");
  localStorage.setItem(key("MEMORY"),memory);
}

/* ===== リセット ===== */
function resetPersona(){
  if(confirm("このキャラの記憶消すで？")){
    localStorage.removeItem(key("HISTORY"));
    localStorage.removeItem(key("MEMORY"));
    location.reload();
  }
}
</script>
</body>
</html>