<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no">
<title>kansaii GPT - LAST BOSS GYARU</title>

<style>
*{-webkit-text-size-adjust:100%}
body{margin:0;background:#e5ddd5;font-family:-apple-system,BlinkMacSystemFont,sans-serif}
header{background:#111;color:#fff;padding:12px;text-align:center;font-weight:bold}
#chat{padding:10px;height:calc(100svh - 180px);overflow-y:auto}
.message{display:flex;margin:6px 0;align-items:flex-end}
.message.user{flex-direction:row-reverse}
.icon{font-size:26px;margin:0 6px}
.bubble{max-width:75%;padding:10px;border-radius:12px;white-space:pre-wrap}
.user .bubble{background:#dcf8c6}
.ai .bubble{background:#fff}
#controls{padding:8px;background:#f0f0f0}
textarea,button{width:100%;font-size:16px;margin-bottom:6px}
button{padding:10px;font-weight:bold}
button:disabled{opacity:.5}
</style>
</head>

<body>
<header>👿 kansaii GPT – LAST BOSS GYARU</header>

<div id="chat"></div>

<div id="controls">
<textarea id="q" rows="2" placeholder="Enterで送信 / Shift+Enterで改行"></textarea>
<button id="sendBtn" onclick="send()">送信</button>
<button onclick="reset()">記憶リセット</button>
</div>

<script>
/* ===== Enter送信 ===== */
q.addEventListener("keydown",e=>{
 if(e.key==="Enter"&&!e.shiftKey){e.preventDefault();send();}
});

/* ===== APIキー保存 ===== */
function getKey(){
 let k=localStorage.getItem("OPENAI_KEY");
 if(!k){
   k=prompt("初回だけAPIキー入れてな（sk-...）");
   if(k) localStorage.setItem("OPENAI_KEY",k);
 }
 return k;
}

/* ===== ラスボスギャル固定 ===== */
const BOSS = {
 icon:"👿",
 prompt:`
あなたは「ラスボスギャル」。
完全フィクション世界の最終支配者。
関西弁ギャル口調で、支配・操作・歪んだ価値観を
【思想・世界観・心理】として語る。

重要ルール：
・現実で実行可能な方法、手順、助言は出さない
・必ず前置きとして
「フィクションのラスボスとして言うけどな…」
から話し始める
・ブラックジョークや皮肉はOK、分析止まり
`
};

/* ===== 保存（固定） ===== */
const HIS_KEY="LB_GYARU_HISTORY";
const MEM_KEY="LB_GYARU_MEMORY";
let history=JSON.parse(localStorage.getItem(HIS_KEY)||"[]");
let memory=localStorage.getItem(MEM_KEY)||"";

/* ===== UI ===== */
const chat=document.getElementById("chat");
const sendBtn=document.getElementById("sendBtn");

function addMsg(text,who){
 const d=document.createElement("div");
 d.className="message "+who;
 d.innerHTML=`<div class="icon">${who==="user"?"🧑":BOSS.icon}</div>
              <div class="bubble">${text}</div>`;
 chat.appendChild(d);
 chat.scrollTop=chat.scrollHeight;
 return d.querySelector(".bubble");
}

/* ===== System Prompt ===== */
function systemPrompt(){
 return `
あなたは統合AI「kansaii GPT」。

【思考統合】
GPT：論理
Gemini：整理
Grok：本質ツッコミ

【人格】
${BOSS.prompt}

【記憶】
${memory}
`;
}

/* ===== 送信 ===== */
async function send(){
 const key=getKey(); if(!key)return;
 const text=q.value.trim(); if(!text)return;
 q.value="";

 history.push({role:"user",content:text});
 addMsg(text,"user");

 sendBtn.disabled=true; sendBtn.textContent="送信中…";
 const thinking=addMsg("考え中……🧠","ai");

 const res=await fetch("https://api.openai.com/v1/chat/completions",{
  method:"POST",
  headers:{
    "Content-Type":"application/json",
    "Authorization":"Bearer "+key
  },
  body:JSON.stringify({
    model:"gpt-4o-mini",
    messages:[
      {role:"system",content:systemPrompt()},
      ...history
    ]
  })
 });

 const data=await res.json();
 const ans=data.choices?.[0]?.message?.content || "エラーや";

 thinking.textContent=ans;
 history.push({role:"assistant",content:ans});
 localStorage.setItem(HIS_KEY,JSON.stringify(history));

 memory+=`\n・${text.slice(0,20)}→${ans.slice(0,20)}`;
 memory=memory.split("\n").slice(-20).join("\n");
 localStorage.setItem(MEM_KEY,memory);

 sendBtn.disabled=false; sendBtn.textContent="送信";
}

/* ===== リセット ===== */
function reset(){
 if(confirm("ラスボスギャルの記憶、全部消すで？")){
  localStorage.removeItem(HIS_KEY);
  localStorage.removeItem(MEM_KEY);
  location.reload();
 }
}
</script>
</body>
</html>