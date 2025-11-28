<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>StudyTech — Local ChatGPT-like AI (Single File)</title>
  <style>
    :root{--bg:#0b1020;--panel:#0f1724;--muted:#9aa4b2;--accent:#6ee7b7;--glass: rgba(255,255,255,0.03)}
    html,body{height:100%;margin:0;font-family:Inter,ui-sans-serif,system-ui,-apple-system,"Segoe UI",Roboto,"Helvetica Neue",Arial}
    body{background:linear-gradient(180deg,#020617 0%, #071026 100%);color:#e6eef6}
    .app{display:grid;grid-template-columns:420px 1fr;gap:18px;height:100vh;padding:18px}

    /* Sidebar */
    .sidebar{background:var(--panel);border-radius:14px;padding:14px;display:flex;flex-direction:column;gap:10px;box-shadow:0 6px 30px rgba(0,0,0,0.6)}
    .brand{display:flex;gap:10px;align-items:center}
    .logo{width:44px;height:44px;border-radius:10px;background:linear-gradient(135deg,#6ee7b7,#3b82f6);display:flex;align-items:center;justify-content:center;font-weight:700;color:#042;}
    .controls{display:flex;flex-direction:column;gap:8px}
    input,select,button{padding:10px;border-radius:10px;border:1px solid rgba(255,255,255,0.04);background:var(--glass);color:inherit}
    .small{font-size:13px;color:var(--muted)}

    /* Big chat panel */
    .chat-panel{background:linear-gradient(180deg, rgba(255,255,255,0.02), transparent);border-radius:14px;padding:18px;display:flex;flex-direction:column}
    .topbar{display:flex;justify-content:space-between;align-items:center;margin-bottom:10px}
    .screen{flex:1;overflow:auto;padding:12px;border-radius:12px;background:linear-gradient(180deg, rgba(255,255,255,0.01), rgba(0,0,0,0.03));border:1px solid rgba(255,255,255,0.02)}
    .msg{display:flex;gap:12px;margin-bottom:12px;align-items:flex-start}
    .avatar{width:40px;height:40px;border-radius:10px;flex:0 0 40px;background:#0b1220;display:flex;align-items:center;justify-content:center;color:var(--muted)}
    .bubble{background:linear-gradient(180deg, rgba(255,255,255,0.02), transparent);padding:12px;border-radius:12px;max-width:78%;white-space:pre-wrap}
    .user .bubble{background:#0b6bff;border-radius:14px;color:white;margin-left:auto}
    .controls-row{display:flex;gap:8px;margin-top:12px}
    .input-area{display:flex;gap:8px}
    textarea{flex:1;min-height:56px;resize:vertical;padding:12px;border-radius:10px;background:transparent;border:1px solid rgba(255,255,255,0.04);color:inherit}

    .meta{font-size:12px;color:var(--muted)}
    .muted{color:var(--muted)}

    .chip{display:inline-block;padding:6px 8px;border-radius:999px;background:rgba(255,255,255,0.02);font-size:13px}

    footer{font-size:12px;color:var(--muted);text-align:center;margin-top:8px}

    /* responsive */
    @media (max-width:900px){.app{grid-template-columns:1fr;grid-auto-rows:auto}.sidebar{order:2}}
  </style>
</head>
<body>
  <div class="app">
    <aside class="sidebar">
      <div class="brand">
        <div class="logo">ST</div>
        <div>
          <div style="font-weight:700">StudyTech AI</div>
          <div class="small">Local ChatGPT-style interface</div>
        </div>
      </div>

      <div class="controls">
        <label class="small">API Key (optional)</label>
        <input id="apiKey" placeholder="Paste your OpenAI API key here (sk-...)" />

        <label class="small">Model</label>
        <select id="modelSelect">
          <option value="gpt-4o-mini">gpt-4o-mini</option>
          <option value="gpt-4">gpt-4</option>
          <option value="gpt-3.5-turbo">gpt-3.5-turbo</option>
        </select>

        <label class="small">Temperature <span id="tempDisplay" style="float:right">0.2</span></label>
        <input id="temperature" type="range" min="0" max="1" step="0.05" value="0.2" />

        <label class="small">System prompt (assistant persona)</label>
        <textarea id="systemPrompt" style="min-height:80px;">You are a helpful, concise AI assistant that explains concepts clearly and provides examples when helpful.</textarea>

        <div style="display:flex;gap:8px">
          <button id="saveKey">Save Key</button>
          <button id="clearStorage">Clear</button>
        </div>

        <div class="meta">Advanced features: streaming responses, regenerate, stop, copy.</div>
      </div>

      <div style="margin-top:auto" class="small muted">Tip: If you don't provide an API key, the interface will return simulated example replies to test the UI.</div>
    </aside>

    <main class="chat-panel">
      <div class="topbar">
        <div>
          <div style="font-size:18px;font-weight:700">Big Screen AI Chat</div>
          <div class="small muted">A simple single-file chat UI — connect your OpenAI key to use the real models.</div>
        </div>
        <div class="chip">Status: <span id="status">Ready</span></div>
      </div>

      <div id="screen" class="screen" aria-live="polite"></div>

      <div class="input-area">
        <textarea id="input" placeholder="Type your message — press Enter to send (Shift+Enter for newline)"></textarea>
        <div style="display:flex;flex-direction:column;gap:8px">
          <button id="send" style="min-width:110px">Send</button>
          <button id="stop">Stop</button>
          <button id="regenerate">Regenerate</button>
        </div>
      </div>

      <div class="controls-row">
        <div class="meta">Model response preview shown as streaming tokens. Use the system prompt to change tone.</div>
      </div>

      <footer>StudyTech • Local demo — modify freely for your project</footer>
    </main>
  </div>

<script>
// Helper utilities
const screen = document.getElementById('screen');
const input = document.getElementById('input');
const sendBtn = document.getElementById('send');
const stopBtn = document.getElementById('stop');
const regenBtn = document.getElementById('regenerate');
const apiKeyInput = document.getElementById('apiKey');
const saveKeyBtn = document.getElementById('saveKey');
const clearStorageBtn = document.getElementById('clearStorage');
const statusEl = document.getElementById('status');
const modelSelect = document.getElementById('modelSelect');
const temperature = document.getElementById('temperature');
const tempDisplay = document.getElementById('tempDisplay');
const systemPrompt = document.getElementById('systemPrompt');

let abortController = null;
let lastConversation = [];
let lastUserMessage = null;

// load saved key
apiKeyInput.value = localStorage.getItem('st_api_key') || '';
modelSelect.value = localStorage.getItem('st_model') || modelSelect.value;
temperature.value = localStorage.getItem('st_temp') || temperature.value;
systemPrompt.value = localStorage.getItem('st_system') || systemPrompt.value;
tempDisplay.innerText = temperature.value;

temperature.addEventListener('input', ()=> tempDisplay.innerText = temperature.value);

saveKeyBtn.onclick = ()=>{
  localStorage.setItem('st_api_key', apiKeyInput.value.trim());
  localStorage.setItem('st_model', modelSelect.value);
  localStorage.setItem('st_temp', temperature.value);
  localStorage.setItem('st_system', systemPrompt.value);
  status('Saved settings to localStorage');
}
clearStorageBtn.onclick = ()=>{
  localStorage.removeItem('st_api_key');
  localStorage.removeItem('st_model');
  localStorage.removeItem('st_temp');
  localStorage.removeItem('st_system');
  apiKeyInput.value='';
  status('Cleared saved settings');
}

function status(t){ statusEl.innerText = t; }

function addMessage(role,text,streaming=false){
  const container = document.createElement('div');
  container.className = 'msg ' + (role==='user' ? 'user' : 'assistant');
  const avatar = document.createElement('div'); avatar.className='avatar'; avatar.innerText = role==='user' ? 'U' : 'AI';
  const bubble = document.createElement('div'); bubble.className='bubble';
  bubble.textContent = text;
  container.appendChild(avatar);
  container.appendChild(bubble);
  screen.appendChild(container);
  screen.scrollTop = screen.scrollHeight;
  return bubble;
}

function replaceLastAssistantNode(text){
  // find last assistant bubble
  const msgs = Array.from(screen.querySelectorAll('.msg'));
  for(let i=msgs.length-1;i>=0;i--){
    if(!msgs[i].classList.contains('user')){
      const bubble = msgs[i].querySelector('.bubble');
      bubble.textContent = text;
      screen.scrollTop = screen.scrollHeight;
      return bubble;
    }
  }
  return null;
}

function simulateReply(prompt){
  // Simple offline fallback — returns a canned reply after a short delay, then streams words
  const full = "Ye ek simulated reply hai jo aapke input par based hai: " + prompt + "\n\nFeatures:\n- Local demo mode\n- Replace with real API key to enable streaming from OpenAI.";
  addMessage('assistant','');
  let idx=0;
  const bubble = replaceLastAssistantNode('');
  const words = full.split(/(\s+)/);
  const iv = setInterval(()=>{
    idx++;
    bubble.textContent += words[idx-1];
    screen.scrollTop = screen.scrollHeight;
    if(idx>=words.length){ clearInterval(iv); status('Ready'); }
  }, 35);
}

async function sendMessage(){
  const txt = input.value.trim();
  if(!txt) return;
  addMessage('user', txt);
  lastUserMessage = txt;
  input.value='';
  status('Thinking...');

  const key = apiKeyInput.value.trim() || localStorage.getItem('st_api_key');
  const model = modelSelect.value;
  const temp = parseFloat(temperature.value);
  const sys = systemPrompt.value;

  // push to conversation history
  lastConversation.push({role:'user',content:txt});

  // If no key provided — simulate
  if(!key){
    simulateReply(txt);
    return;
  }

  // prepare API call (OpenAI Chat Completions with streaming)
  abortController = new AbortController();
  try{
    addMessage('assistant','');
    replaceLastAssistantNode('');

    // Build messages object: include system prompt + last conversation (simple)
    const messages = [{role:'system',content:sys}, ...lastConversation.slice(-12)];

    const resp = await fetch('https://api.openai.com/v1/chat/completions', {
      method:'POST',
      headers: {
        'Content-Type':'application/json',
        'Authorization':'Bearer ' + key
      },
      body: JSON.stringify({
        model,
        messages,
        temperature: temp,
        stream: true
      }),
      signal: abortController.signal
    });

    if(!resp.ok){
      const txt = await resp.text();
      replaceLastAssistantNode('Error: ' + resp.status + ' — ' + txt);
      status('Error');
      return;
    }

    // stream parse (SSE-ish) — this reads event-stream style where each chunk starts with "data: "
    const reader = resp.body.getReader();
    const decoder = new TextDecoder('utf-8');
    let done=false;
    let assistantText='';
    const bubble = replaceLastAssistantNode('');

    while(!done){
      const {value,done:doneReading} = await reader.read();
      done = doneReading;
      if(value){
        const chunk = decoder.decode(value);
        // the OpenAI stream sends lines like: data: {json}\n\n and final: data: [DONE]
        const parts = chunk.split(/\n/).filter(Boolean);
        for(const part of parts){
          if(part.trim() === 'data: [DONE]'){
            done = true; break;
          }
          if(part.startsWith('data: ')){
            const jsonStr = part.replace(/^data: /, '');
            try{
              const parsed = JSON.parse(jsonStr);
              const delta = parsed.choices?.[0]?.delta;
              if(delta?.content){
                assistantText += delta.content;
                bubble.textContent = assistantText;
                screen.scrollTop = screen.scrollHeight;
              }
            }catch(e){
              // sometimes chunk boundaries split JSON -> ignore parse errors
            }
          }
        }
      }
    }

    status('Ready');
    // add final assistant message to conversation history
    lastConversation.push({role:'assistant',content:assistantText});

  }catch(err){
    if(err.name === 'AbortError'){
      status('Aborted');
      replaceLastAssistantNode('<<Stopped by user>>');
    }else{
      status('Error');
      replaceLastAssistantNode('Error: ' + (err.message || String(err)));
    }
  } finally {
    abortController = null;
  }
}

sendBtn.addEventListener('click', sendMessage);
input.addEventListener('keydown', (e)=>{
  if(e.key === 'Enter' && !e.shiftKey){ e.preventDefault(); sendMessage(); }
});

stopBtn.addEventListener('click', ()=>{
  if(abortController) abortController.abort();
});

regenBtn.addEventListener('click', async ()=>{
  // remove last assistant message, then re-send last user message
  // simple strategy: drop last assistant from history and re-send
  for(let i=lastConversation.length-1;i>=0;i--){ if(lastConversation[i].role==='assistant'){ lastConversation.splice(i,1); break; } }
  if(lastUserMessage){ input.value = lastUserMessage; sendMessage(); }
});

// small demo welcome
addMessage('assistant', 'Hello! This is a local ChatGPT-like interface. Type a message and press Send.');

</script>
</body>
</html>

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Advanced AI Chat v2</title>
  <style>
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: #0d1117;
      color: #ffffff;
      display: flex;
      height: 100vh;
      overflow: hidden;
    }

    .sidebar {
      width: 260px;
      background: #161b22;
      padding: 20px;
      box-sizing: border-box;
      border-right: 1px solid #30363d;
      display: flex;
      flex-direction: column;
    }

    .sidebar h2 {
      margin: 0 0 15px;
      font-size: 20px;
    }

    .sidebar label {
      font-size: 13px;
      margin-top: 10px;
      color: #9ba3b4;
    }

    .sidebar input,
    .sidebar select,
    .sidebar textarea,
    .sidebar button {
      width: 100%;
      margin-top: 5px;
      padding: 10px;
      border-radius: 8px;
      background: #0d1117;
      border: 1px solid #30363d;
      color: white;
      outline: none;
    }

    .sidebar button {
      background: #238636;
      border: none;
      cursor: pointer;
      margin-top: 12px;
      font-weight: bold;
    }

    .chat-area {
      flex: 1;
      display: flex;
      flex-direction: column;
    }

    .chat-header {
      padding: 15px;
      background: #161b22;
      border-bottom: 1px solid #30363d;
      font-size: 18px;
      font-weight: bold;
    }

    .messages {
      flex: 1;
      overflow-y: auto;
      padding: 20px;
    }

    .msg {
      margin-bottom: 18px;
      max-width: 75%;
      padding: 12px;
      border-radius: 12px;
      white-space: pre-wrap;
      line-height: 1.5;
    }

    .user {
      background: #238636;
      margin-left: auto;
    }

    .ai {
      background: #30363d;
    }

    .input-box {
      padding: 15px;
      background: #161b22;
      border-top: 1px solid #30363d;
      display: flex;
      gap: 10px;
    }

    .input-box textarea {
      flex: 1;
      height: 60px;
      resize: none;
      border-radius: 8px;
      padding: 10px;
      background: #0d1117;
      border: 1px solid #30363d;
      color: white;
    }

    .send-btn {
      width: 120px;
      background: #1f6feb;
      border: none;
      cursor: pointer;
      border-radius: 8px;
      font-weight: bold;
    }
  </style>
</head>
<body>

  <div class="sidebar">
    <h2>AI Settings</h2>

    <label>API Key</label>
    <input id="apiKey" placeholder="sk-..." />

    <label>Model</label>
    <select id="model">
      <option value="gpt-4o-mini">gpt-4o-mini</option>
      <option value="gpt-4">gpt-4</option>
      <option value="gpt-3.5-turbo">gpt-3.5-turbo</option>
    </select>

    <label>Temperature</label>
    <input type="range" id="temp" min="0" max="1" step="0.05" value="0.2" />

    <label>System Prompt</label>
    <textarea id="system" rows="4">You are a smart, advanced AI like ChatGPT.</textarea>

    <button onclick="saveSettings()">Save Settings</button>
  </div>

  <div class="chat-area">
    <div class="chat-header">Advanced AI Chat</div>

    <div class="messages" id="messages"></div>

    <div class="input-box">
      <textarea id="userInput" placeholder="Type your message..."></textarea>
      <button class="send-btn" onclick="sendMessage()">Send</button>
    </div>
  </div>

<script>
const msgBox = document.getElementById("messages");

function addMessage(text, role) {
  const div = document.createElement("div");
  div.className = "msg " + role;
  div.textContent = text;
  msgBox.appendChild(div);
  msgBox.scrollTop = msgBox.scrollHeight;
}

function saveSettings() {
  localStorage.setItem("ai_key", document.getElementById("apiKey").value);
  localStorage.setItem("ai_model", document.getElementById("model").value);
  localStorage.setItem("ai_temp", document.getElementById("temp").value);
  localStorage.setItem("ai_system", document.getElementById("system").value);
  alert("Settings Saved ✔");
}

async function sendMessage() {
  const input = document.getElementById("userInput").value.trim();
  if (!input) return;

  addMessage(input, "user");
  document.getElementById("userInput").value = "";

  const key = localStorage.getItem("ai_key");
  const model = localStorage.getItem("ai_model") || "gpt-4o-mini";
  const temp = parseFloat(localStorage.getItem("ai_temp") || 0.2);
  const system = localStorage.getItem("ai_system") || "You are a helpful AI";

  if (!key) {
    addMessage("❗ API key missing. Go to sidebar and enter your key.", "ai");
    return;
  }

  addMessage("Thinking...", "ai");
  const thinkingNode = msgBox.lastChild;

  const body = {
    model: model,
    temperature: temp,
    messages: [
      { role: "system", content: system },
      { role: "user", content: input }
    ],
    stream: true
  };

  const response = await fetch("https://api.openai.com/v1/chat/completions", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: "Bearer " + key
    },
    body: JSON.stringify(body)
  });

  const reader = response.body.getReader();
  const decoder = new TextDecoder();

  let full = "";

  while (true) {
    const { value, done } = await reader.read();
    if (done) break;

    const chunk = decoder.decode(value);
    const lines = chunk.split("\n");

    for (const line of lines) {
      if (line.startsWith("data: ")) {
        const json = line.replace("data: ", "").trim();
        if (json === "[DONE]") break;

        try {
          const parsed = JSON.parse(json);
          const delta = parsed.choices?.[0]?.delta?.content;
          if (delta) {
            full += delta;
            thinkingNode.textContent = full;
          }
        } catch {}
      }
    }
  }
}
</script>

</body>
</html>






