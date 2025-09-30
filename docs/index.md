<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <title>r1 TODO Creator</title>
  <meta name="viewport" content="width=360, initial-scale=1.0">
  <style>
    body { font-family: sans-serif; background: #fafafa; margin: 0; padding: 1em; max-width: 360px; }
    h1 { font-size: 1.4em; margin-bottom: 0.5em; }
    .input-area { margin-bottom: 1em; }
    .todos { margin-top: 1em; }
    li { background: #fff; margin-bottom: 4px; padding: 6px 10px; border-radius: 5px; box-shadow: 1px 1px 3px #eee; }
    button { padding: 8px; font-size: 1em; border-radius: 5px; border: none; background: #eee; }
    button:active { background: #ddd; }
  </style>
</head>
<body>
  <h1>Sprach-Todos für Rabbit r1</h1>
  <div class="input-area">
    <textarea id="textInput" rows="2" placeholder="Stichworte diktieren oder eintippen..."></textarea><br>
    <button id="micBtn">🎤 Spracheingabe</button>
    <button id="sendBtn">✅ Aufgabe analysieren</button>
  </div>
  <ul id="todos" class="todos"></ul>

  <script>
    // Sprachaufnahme
    const micBtn = document.getElementById('micBtn');
    const textInput = document.getElementById('textInput');
    micBtn.onclick = () => {
      if (!('webkitSpeechRecognition' in window)) {
        alert('Speech Recognition wird nicht unterstützt.');
        return;
      }
      const recognition = new webkitSpeechRecognition();
      recognition.lang = 'de-DE';
      recognition.onresult = (event) => {
        textInput.value = event.results[0][0].transcript;
      };
      recognition.start();
      micBtn.innerText = '🎤 Aufnahme läuft...';
      recognition.onend = () => { micBtn.innerText = '🎤 Spracheingabe'; };
    };

    // Todo-Liste
    const todosUl = document.getElementById('todos');
    function renderTodos(items) {
      todosUl.innerHTML = '';
      items.forEach((todo, idx) => {
        const li = document.createElement('li');
        li.textContent = todo;
        todosUl.appendChild(li);
      });
    }

    // Senden an LLM-API & Todos anzeigen
    document.getElementById('sendBtn').onclick = async () => {
      const input = textInput.value.trim();
      if (!input) return;
      // Prompt für LLM: Extrahiere Aufgaben als JSON-Liste!
      const prompt = `Extrahiere aus folgendem Text klar trennbare Aufgaben/Todos als JSON-Liste. Rückgabe nur die Liste: ${input}`;
      // Beispiel: OpenAI API (ersetzen durch eigenen Key/Endpoint!)
      const llmResponse = await fetch("https://YOUR_LLM_API_ENDPOINT", {
        method: "POST",
        headers: { "Content-Type": "application/json", "Authorization": "Bearer YOUR_API_KEY" },
        body: JSON.stringify({ prompt: prompt, max_tokens: 512 })
      });
      const data = await llmResponse.json();
      // Nimm nur das Array (hier ein Beispiel-Parser)
      let todos = [];
      try {
        todos = JSON.parse(data.choices[0].text);
      } catch { todos = ["Fehler beim Parsen."]; }
      renderTodos(todos);
      textInput.value = '';
    };
  </script>
</body>
</html>
