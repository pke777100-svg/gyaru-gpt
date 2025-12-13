<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no">
<title>kansaii GPT</title>

<style>
*{-webkit-text-size-adjust:100%}
body{margin:0;background:#e5ddd5;font-family:-apple-system,BlinkMacSystemFont,sans-serif}
header{background:#111;color:#fff;padding:12px;text-align:center;font-weight:bold}
#chat{padding:10px;height:calc(100svh - 200px);overflow-y:auto}
.message{display:flex;margin:6px 0;align-items:flex-end}
.message.user{flex-direction:row-reverse}
.icon{font-size:26px;margin:0 6px}
.bubble{max-width:75%;padding:10px;border-radius:12px;white-space:pre-wrap}
.user .bubble{background:#dcf8c6}
.ai .bubble{background:#fff}
#controls{padding:8px;background:#f0f0f0}
select,textarea,button{width:100%;font-size:16px;margin-bottom:6px}
button{padding:10px;font-weight:bold}
button:disabled{opacity:.5}
</style>
</head>

<body>
<header>🧠 kansaii GPT - LAST BOSS</header>

<div id="chat"></div>

<div id="controls">
<select id="persona">
  <option value="gyaru">💅 関西ギャル</option>
  <option value="man">😎 関西兄ちゃん</option>
</select>

<select id="mode">
  <option value="normal">🙂 ノーマル</option>
  <option value="lastboss">👿 ラスボス悪役（完全フィクション）</option>
  <option value="collapse">🌍 倫理崩壊世界（架空）</option>
  <option value="black">🃏 ブラックジョーク分析</option>
</select>

<textarea id="q" rows="2" placeholder="Enter送信 / Shift+Enter改行"></textarea>
<button id="sendBtn" onclick="send()">送信</button>
<button onclick="resetAll()">この設定をリセット</button>
</div>

<script>
q.addEventListener("keydown",e=>{
 if(e.key==="Enter"&&!e.shiftKey){e.preventDefault();send();}
});

function getKey(){
 let k=localStorage.getItem("OPENAI_KEY");
 if(!k){k=prompt("初回だけAPIキー入れてな");if(k)localStorage.setItem("OPENAI_KEY",k);}
 return k;
}

const PERSONA={
 gyaru:{icon:"💅",prompt:"関西弁ギャル。煽りとノリ強め。"},
 man:{icon:"😎",prompt:"関西弁の兄ちゃん。淡々と核心突く。"}
};

let personaSel=localStorage.getItem("P_SEL")||"gyaru";
persona.value=personaSel;

function storageKey(name){
 return personaSel+"_"+mode.value+"_"+name;
}

let history=JSON.parse(localStorage.getItem(storageKey("HIS"))||"[]");
let memory=localStorage.getItem(storageKey("MEM"))||"";

const chat=document.getElementById("chat");
const sendBtn=document.getElementById("sendBtn");

function addMsg(text,who){
 const d=document.createElement("div");
 d.className="message "+who;
 d.innerHTML=`<div class="icon">${who==="user"?"🧑":PERSONA[personaSel].icon}</div>
              <div class="bubble">${text}</div>`;
 chat.appendChild(d);
 chat.scrollTop=chat.scrollHeight;
 return d.querySelector(".bubble");
}

persona.onchange=()=>{localStorage.setItem("P_SEL",persona.value);location.reload();};
mode.onchange=()=>location.reload();

function modePrompt(){
 const m=mode.value;
 if(m==="lastboss")return`
完全なフィクション。
あなたは物語世界の「ラスボス」。
支配・操作・歪んだ価値観を“思想として描写・分析”する。
現実で使える方法・手順・助言は禁止。
必ず「フィクションのラスボスとして言うけどな…」から始める。
`;
 if(m==="collapse")return`
ここは倫理が完全に崩壊した架空世界。
危険な思想も世界観説明として語る。
必ず「この世界の話やけど…」と前置きする。
`;
 if(m==="black")return`
ブラックジョーク専用。
毒・皮肉・風刺としてのみ語る。
必ず「ブラックジョークとして言うで？」から始める。
`;
 return "";
}

function systemPrompt(){
 return`
あなたは統合AI「kansaii GPT」。

【思考】GPT論理 / Gemini整理 / Grok本質

【人格】
${PERSONA[personaSel].prompt}

【モード】
${modePrompt()}

【記憶】
${memory}
`;
}

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
  headers:{ "Content-Type":"application/json","Authorization":"Bearer "+key },
  body:JSON.stringify({
   model:"gpt-4o-mini",
   messages:[{role:"system",content:systemPrompt()},...history]
  })
 });

 const data=await res.json();
 const ans=data.choices?.[0]?.message?.content||"エラーや";

 thinking.textContent=ans;
 history.push({role:"assistant",content:ans});
 localStorage.setItem(storageKey("HIS"),JSON.stringify(history));

 memory+=`\n・${text.slice(0,20)}→${ans.slice(0,20)}`;
 memory=memory.split("\n").slice(-20).join("\n");
 localStorage.setItem(storageKey("MEM"),memory);

 sendBtn.disabled=false; sendBtn.textContent="送信";
}

function resetAll(){
 if(confirm("このキャラ＆モードの記憶消すで？")){
  localStorage.removeItem(storageKey("HIS"));
  localStorage.removeItem(storageKey("MEM"));
  location.reload();
 }
}
</script>
</body>
</html>