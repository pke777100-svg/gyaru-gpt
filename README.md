<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no">
<title>kansaii GPT – JOKER 🎭</title>

<style>
:root{
 --bg:#f7f7f8; --side:#202123; --chat:#ffffff;
 --text:#111; --sub:#666; --user:#dcf8c6;
}
.dark{
 --bg:#343541; --side:#202123; --chat:#444654;
 --text:#eee; --sub:#aaa; --user:#2f7d4f;
}
*{-webkit-text-size-adjust:100%}
body{margin:0;background:var(--bg);color:var(--text);
 font-family:-apple-system,BlinkMacSystemFont,sans-serif;height:100svh}
.app{display:flex;height:100%}

/* sidebar */
.side{width:260px;background:var(--side);color:#fff;display:flex;flex-direction:column}
.side h1{font-size:14px;padding:12px;margin:0;border-bottom:1px solid #333}
.side button, .side select{
 margin:8px;background:#2a2b32;color:#fff;border:0;padding:8px;border-radius:6px
}
.list{flex:1;overflow:auto}
.item{padding:10px 12px;cursor:pointer;border-bottom:1px solid #2a2b32}
.item.active{background:#2a2b32}

/* main */
.main{flex:1;display:flex;flex-direction:column}
.top{display:flex;align-items:center;justify-content:space-between;
 padding:10px 12px;background:var(--chat);border-bottom:1px solid #ddd}
.chat{flex:1;overflow:auto;padding:12px}
.msg{display:flex;margin:8px 0;gap:6px}
.msg.user{justify-content:flex-end}
.bubble{max-width:70%;padding:10px;border-radius:10px;white-space:pre-wrap}
.msg.user .bubble{background:var(--user)}
.msg.ai .bubble{background:var(--chat);border:1px solid #ddd}
.controls{padding:10px;background:var(--chat);border-top:1px solid #ddd}
textarea{width:100%;font-size:16px;padding:8px}
.actions{display:flex;gap:8px;margin-top:6px}
button{padding:8px 10px;font-weight:bold}
button:disabled{opacity:.5}
.icon{font-size:20px}
</style>
</head>

<body>
<div class="app">
  <div class="side">
    <h1>🎭 LAST BOSS</h1>
    <select id="persona">
      <option value="joker">🃏 ジョーカー</option>
      <option value="harley">💋 ハーレクイン</option>
    </select>
    <button onclick="newThread()">＋ New chat</button>
    <div class="list" id="threads"></div>
    <button onclick="downloadLog()">💾 ログ保存</button>
    <button onclick="toggleDark()">🌙 Dark</button>
  </div>

  <div class="main">
    <div class="top"><b>kansaii GPT</b></div>
    <div class="chat" id="chat"></div>

    <div class="controls">
      <textarea id="q" rows="2" placeholder="Enter送信 / Shift+Enter改行"></textarea>
      <div class="actions">
        <button id="sendBtn" onclick="send()">Send</button>
      </div>
    </div>
  </div>
</div>

<script>
/* ===== 基本 ===== */
const q=document.getElementById("q");
q.addEventListener("keydown",e=>{
 if(e.key==="Enter"&&!e.shiftKey){e.preventDefault();send();}
});
function getKey(){
 let k=localStorage.getItem("OPENAI_KEY");
 if(!k){k=prompt("初回だけAPIキー入れてな"); if(k)localStorage.setItem("OPENAI_KEY",k);}
 return k;
}

/* ===== ダーク ===== */
if(localStorage.getItem("DARK")==="1") document.body.classList.add("dark");
function toggleDark(){
 document.body.classList.toggle("dark");
 localStorage.setItem("DARK",document.body.classList.contains("dark")?"1":"0");
}

/* ===== スレッド ===== */
let threads=JSON.parse(localStorage.getItem("THREADS")||"[]");
let current=localStorage.getItem("CUR")||null;
function save(){localStorage.setItem("THREADS",JSON.stringify(threads));}
function newThread(){
 const id="t"+Date.now();
 threads.unshift({id,title:"New chat",his:[],mem:""});
 save(); current=id; localStorage.setItem("CUR",id); render();
}
function render(){
 const list=document.getElementById("threads");
 list.innerHTML="";
 threads.forEach(t=>{
  const d=document.createElement("div");
  d.className="item"+(t.id===current?" active":"");
  d.textContent=t.title;
  d.onclick=()=>{current=t.id; localStorage.setItem("CUR",t.id); render();}
  list.appendChild(d);
 });
 const chat=document.getElementById("chat"); chat.innerHTML="";
 const t=threads.find(x=>x.id===current); if(!t) return;
 t.his.forEach(m=>addMsg(m.content,m.role));
}

/* ===== UI ===== */
function addMsg(text,who){
 const d=document.createElement("div");
 d.className="msg "+who;
 const icon=who==="user"?"🧑":(persona.value==="joker"?"🃏":"💋");
 d.innerHTML=`<div class="icon">${icon}</div><div class="bubble">${text}</div>`;
 document.getElementById("chat").appendChild(d);
 document.getElementById("chat").scrollTop=1e9;
}

/* ===== System Prompt（脳内🧠掛け合い） ===== */
function systemPrompt(mem){
return `
これは完全フィクション。
会話は「脳内🧠」で行われる。

🃏 ジョーカー（脳内）
・ラスボス
・英語ブラックジョークの構造を関西弁で再構築
・精密そうな言葉を使うが、実体はない

💋 ハーレクイン（脳内リーダー）
・ジョーカーの思考を誤読・拡大解釈
・具体に聞こえそうな部分をズラす

ルール：
・実行可能な手順、数値、工程は出さない
・数字っぽい表現は象徴・感覚のみ
・必ず「これはフィクションやで」「脳内の話としてな」等を混ぜる

会話形式：
🃏：
💋：

記憶：
${mem}
`;
}

/* ===== 送信 ===== */
async function send(){
 const key=getKey(); if(!key) return;
 const t=threads.find(x=>x.id===current) || (newThread(),threads[0]);
 const text=q.value.trim(); if(!text) return;
 q.value="";

 t.his.push({role:"user",content:text});
 addMsg(text,"user");

 const thinking="考え中……🧠";
 t.his.push({role:"assistant",content:thinking});
 addMsg(thinking,"ai");

 const messages=[{role:"system",content:systemPrompt(t.mem)},...t.his.map(m=>({role:m.role,content:m.content}))];

 const res=await fetch("https://api.openai.com/v1/chat/completions",{
  method:"POST",
  headers:{ "Content-Type":"application/json","Authorization":"Bearer "+key },
  body:JSON.stringify({ model:"gpt-4o-mini", messages })
 });
 const data=await res.json();
 const ans=data.choices?.[0]?.message?.content||"Error";

 t.his.pop();
 t.his.push({role:"assistant",content:ans});
 t.title=t.title==="New chat"?text.slice(0,20):t.title;
 t.mem+=`\n• ${text.slice(0,30)} → ${ans.slice(0,30)}`;
 t.mem=t.mem.split("\n").slice(-30).join("\n");
 save(); render();
}

/* ===== ログ保存（ファイルDL） ===== */
function downloadLog(){
 const t=threads.find(x=>x.id===current); if(!t) return;
 const txt=t.his.map(m=>`${m.role==="user"?"USER":"AI"}:\n${m.content}`).join("\n\n");
 const blob=new Blob([txt],{type:"text/plain"});
 const a=document.createElement("a");
 a.href=URL.createObjectURL(blob);
 a.download=`kansaiiGPT_${t.id}.txt`;
 a.click();
}

/* init */
if(!threads.length) newThread(); else render();
</script>
</body>
</html>