# Suru-ai
Ahhajaj
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Suru XAI</title>
<style>
* { box-sizing: border-box; }
body {
  font-family: 'Segoe UI', Arial, sans-serif;
  margin:0;
  min-height:100vh;
  background: linear-gradient(135deg, #ff9a56 0%, #ff6a88 45%, #a86bff 100%);
  color:#1a1a2e;
}
header {
  padding:22px;
  text-align:center;
  font-size:28px;
  font-weight:800;
  color:#fff;
  text-shadow:0 2px 8px rgba(0,0,0,0.15);
}
.container { max-width:680px; margin:auto; padding:10px 18px 30px; }

.card {
  background:rgba(255,255,255,0.85);
  backdrop-filter:blur(14px);
  border-radius:20px;
  padding:18px;
  box-shadow:0 8px 30px rgba(0,0,0,0.15);
  border:1px solid rgba(255,255,255,0.5);
}
.keybox { margin-bottom:16px; }
.keybox label { font-size:12px; font-weight:700; color:#7a4dbf; text-transform:uppercase; letter-spacing:0.5px; }
input, button {
  width:100%;
  padding:13px 14px;
  margin-top:8px;
  border-radius:12px;
  border:none;
  font-size:14px;
}
input {
  background:#fff;
  border:1px solid #e5d9f7;
  color:#1a1a2e;
}
input:focus { outline:2px solid #a86bff; }
button {
  cursor:pointer;
  background:linear-gradient(135deg,#ff6a88,#a86bff);
  color:#fff;
  font-weight:700;
  transition:transform 0.15s, opacity 0.15s;
}
button:hover { transform:translateY(-1px); opacity:0.92; }
button:disabled { background:#ccc; cursor:not-allowed; transform:none; }
#status { font-size:12px; color:#2e9e5b; margin-top:6px; min-height:14px; font-weight:600; }

#chat {
  min-height:320px;
  max-height:440px;
  overflow-y:auto;
  border-radius:16px;
  padding:16px;
  margin-top:16px;
  background:#fdf9ff;
  border:1px solid #eee2fb;
  display:flex;
  flex-direction:column;
  gap:12px;
}
.bubble-row { display:flex; gap:8px; align-items:flex-end; }
.bubble-row.user { flex-direction:row-reverse; }
.avatar {
  width:30px; height:30px; border-radius:50%;
  display:flex; align-items:center; justify-content:center;
  font-size:15px; flex-shrink:0;
  box-shadow:0 2px 6px rgba(0,0,0,0.15);
}
.avatar.bot { background:linear-gradient(135deg,#ff9a56,#ff6a88); }
.avatar.user { background:linear-gradient(135deg,#6b8cff,#a86bff); }
.msg {
  padding:11px 15px;
  border-radius:16px;
  line-height:1.5;
  white-space:pre-wrap;
  max-width:78%;
  font-size:14.5px;
}
.user .msg { background:linear-gradient(135deg,#6b8cff,#a86bff); color:#fff; border-bottom-right-radius:4px; }
.bot .msg { background:#fff; border:1px solid #eee2fb; border-bottom-left-radius:4px; color:#1a1a2e; }

.typing { display:flex; gap:4px; padding:4px 0; }
.typing span {
  width:6px; height:6px; border-radius:50%; background:#c39be8;
  animation: blink 1.2s infinite ease-in-out;
}
.typing span:nth-child(2) { animation-delay:0.2s; }
.typing span:nth-child(3) { animation-delay:0.4s; }
@keyframes blink { 0%,80%,100%{opacity:0.25;} 40%{opacity:1;} }

.row { display:flex; gap:10px; margin-top:14px; }
.row input { margin-top:0; }
.row button { width:auto; padding:13px 24px; margin-top:0; white-space:nowrap; }
</style>
</head>
<body>
<header>🌞 Suru XAI</header>
<div class="container">
<div class="card">

<div class="keybox">
  <label for="apiKey">Gemini API Key</label>
  <input type="password" id="apiKey" placeholder="Paste your Gemini API Key here">
  <button onclick="saveKey()">Save API Key</button>
  <div id="status"></div>
</div>

<div id="chat">
  <div class="bubble-row bot">
    <div class="avatar bot">🌞</div>
    <div class="msg">Hello! I'm Suru XAI, your personal AI companion. Save your Gemini API key above to start chatting.</div>
  </div>
</div>

<div class="row">
  <input type="text" id="userInput" placeholder="Type your message..." onkeydown="if(event.key==='Enter') sendMessage()">
  <button id="sendBtn" onclick="sendMessage()">Send</button>
</div>

</div>
</div>

<script>
let apiKey = '';
let history = [];

function saveKey(){
    const key = document.getElementById('apiKey').value.trim();
    if(!key){ alert('Please enter a valid API key'); return; }
    apiKey = key;
    document.getElementById('status').innerText = '✅ API key saved for this session';
    document.getElementById('apiKey').value = '';
    addMessage('bot', "Perfect, I'm ready! Ask me anything.");
}

function addMessage(role, text){
    const chat = document.getElementById('chat');
    const row = document.createElement('div');
    row.className = 'bubble-row ' + role;
    row.innerHTML = `
      <div class="avatar ${role}">${role === 'user' ? '🧑' : '🌞'}</div>
      <div class="msg"></div>
    `;
    row.querySelector('.msg').innerText = text;
    chat.appendChild(row);
    chat.scrollTop = chat.scrollHeight;
    return row;
}

function addTyping(){
    const chat = document.getElementById('chat');
    const row = document.createElement('div');
    row.className = 'bubble-row bot';
    row.id = 'typingRow';
    row.innerHTML = `
      <div class="avatar bot">🌞</div>
      <div class="msg"><div class="typing"><span></span><span></span><span></span></div></div>
    `;
    chat.appendChild(row);
    chat.scrollTop = chat.scrollHeight;
}

function removeTyping(){
    const row = document.getElementById('typingRow');
    if(row) row.remove();
}

async function sendMessage(){
    const input = document.getElementById('userInput');
    const text = input.value.trim();
    if(!text) return;
    if(!apiKey){ alert('Please save your Gemini API key first'); return; }

    addMessage('user', text);
    history.push({ role: 'user', parts: [{ text }] });
    input.value = '';

    const sendBtn = document.getElementById('sendBtn');
    sendBtn.disabled = true;
    addTyping();

    try {
        const res = await fetch(
            `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`,
            {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ contents: history })
            }
        );
        const data = await res.json();
        removeTyping();

        if(data.error){
            addMessage('bot', '⚠️ Error: ' + data.error.message);
        } else {
            const reply = data.candidates?.[0]?.content?.parts?.[0]?.text || 'No response received.';
            history.push({ role: 'model', parts: [{ text: reply }] });
            addMessage('bot', reply);
        }
    } catch (err) {
        removeTyping();
        addMessage('bot', '⚠️ Network error: ' + err.message);
    } finally {
        sendBtn.disabled = false;
    }
}
</script>
</body>
</html>

