# gyaru-gpt
gyaru-gpt
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>ギャルGPT</title>
</head>
<body>
  <h1>💅 ギャルGPT 💅</h1>
  <p>ここにAPIキー入れてね👇</p>
  <input id="key" type="password" placeholder="sk-..." />
  <br><br>
  <textarea id="q" rows="4" cols="40" placeholder="聞きたいこと書いてね♡"></textarea>
  <br><br>
  <button onclick="send()">送信</button>
  <pre id="a"></pre>

<script>
async function send() {
  const key = document.getElementById("key").value;
  const q = document.getElementById("q").value;
  const res = await fetch("https://api.openai.com/v1/chat/completions", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": "Bearer " + key
    },
    body: JSON.stringify({
      model: "gpt-4o-mini",
      messages: [
        {role: "system", content: "あなたは日本語で話すノリのいいギャルAI、ギャルGPTです"},
        {role: "user", content: q}
      ]
    })
  });
  const data = await res.json();
  document.getElementById("a").textContent =
    data.choices?.[0]?.message?.content || "エラーだよ🥺";
}
</script>
</body>
</html>