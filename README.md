<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Digital Stress Diary</title>

  <!-- Google Fonts -->
  <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap" rel="stylesheet" />
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

  <style>
    :root {
      --bg-gradient: linear-gradient(135deg, #a1c4fd 0%, #c2e9fb 100%);
      --card-bg: #ffffffcc;
      --text-dark: #333;
      --accent: #6a11cb;
      --accent2: #2575fc;
      --button-gradient: linear-gradient(135deg, #6a11cb, #2575fc);
      --shadow: 0 8px 20px rgba(0, 0, 0, 0.1);
      --radius: 16px;
    }

    body {
      margin: 0;
      font-family: "Poppins", sans-serif;
      background: var(--bg-gradient);
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: flex-start;
      min-height: 100vh;
      color: var(--text-dark);
    }

    h1 {
      margin-top: 30px;
      font-weight: 600;
      background: var(--button-gradient);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
    }

    .container {
      background: var(--card-bg);
      backdrop-filter: blur(12px);
      box-shadow: var(--shadow);
      border-radius: var(--radius);
      padding: 30px;
      margin: 30px;
      width: 90%;
      max-width: 600px;
      text-align: center;
    }

    select, textarea, button {
      width: 90%;
      padding: 12px;
      border: none;
      border-radius: var(--radius);
      margin: 10px 0;
      font-family: inherit;
      font-size: 1rem;
      box-shadow: var(--shadow);
    }

    select, textarea {
      background: #f8f9fa;
      color: var(--text-dark);
    }

    button {
      background: var(--button-gradient);
      color: white;
      cursor: pointer;
      transition: transform 0.2s ease, box-shadow 0.3s ease;
    }

    button:hover {
      transform: scale(1.05);
      box-shadow: 0 4px 20px rgba(106, 17, 203, 0.3);
    }

    canvas {
      margin-top: 20px;
    }

    /* AI Chat Button */
    .ai-btn {
      position: fixed;
      bottom: 25px;
      right: 25px;
      background: var(--button-gradient);
      color: white;
      border: none;
      border-radius: 50%;
      width: 60px;
      height: 60px;
      font-size: 26px;
      box-shadow: var(--shadow);
      cursor: pointer;
      transition: transform 0.2s ease;
    }

    .ai-btn:hover {
      transform: rotate(10deg) scale(1.1);
    }

    /* AI Chat Window */
    .chat-popup {
      display: none;
      position: fixed;
      bottom: 90px;
      right: 30px;
      background: #fff;
      border-radius: var(--radius);
      box-shadow: var(--shadow);
      width: 320px;
      max-height: 400px;
      overflow: hidden;
      flex-direction: column;
    }

    .chat-header {
      background: var(--button-gradient);
      color: white;
      padding: 12px;
      font-weight: 600;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .chat-body {
      padding: 10px;
      overflow-y: auto;
      height: 300px;
      font-size: 0.9rem;
    }

    .chat-input {
      display: flex;
      border-top: 1px solid #eee;
    }

    .chat-input input {
      flex: 1;
      border: none;
      padding: 10px;
      font-family: inherit;
    }

    .chat-input button {
      background: var(--button-gradient);
      border: none;
      color: white;
      padding: 10px 14px;
      cursor: pointer;
      border-radius: 0 var(--radius) var(--radius) 0;
    }

    .user-msg, .bot-msg {
      margin: 8px 0;
      padding: 8px 12px;
      border-radius: 12px;
      max-width: 80%;
      word-wrap: break-word;
    }

    .user-msg {
      background: #e1f5fe;
      align-self: flex-end;
    }

    .bot-msg {
      background: #f3e5f5;
      align-self: flex-start;
    }
  </style>
</head>

<body>
  <h1>🌤️ Digital Stress Diary</h1>

  <div class="container">
    <h2>Log Your Mood</h2>
    <select id="mood">
      <option value="">Select your mood</option>
      <option value="10">🤩 Ecstatic</option>
      <option value="9">😄 Very Happy</option>
      <option value="8">😊 Cheerful</option>
      <option value="7">🙂 Content</option>
      <option value="6">😌 Relaxed</option>
      <option value="5">😐 Neutral</option>
      <option value="4">😕 Anxious</option>
      <option value="3">😟 Stressed</option>
      <option value="2">😢 Sad</option>
      <option value="1">😵‍💫 Overwhelmed</option>
    </select>
    <textarea id="notes" rows="3" placeholder="Add notes..."></textarea>
    <button onclick="saveEntry()">Save Entry</button>

    <canvas id="moodChart"></canvas>
  </div>

  <!-- AI Assistant Popup -->
  <button class="ai-btn" onclick="toggleChat()">💬</button>

  <div class="chat-popup" id="chatPopup">
    <div class="chat-header">
      AI Assistant
      <span style="cursor:pointer;" onclick="toggleChat()">✖️</span>
    </div>
    <div class="chat-body" id="chatBody">
      <div class="bot-msg">Hi! I'm your stress diary assistant. How are you feeling today?</div>
    </div>
    <div class="chat-input">
      <input type="text" id="chatInput" placeholder="Type a message..." onkeypress="if(event.key==='Enter') sendMessage()" />
      <button onclick="sendMessage()">➤</button>
    </div>
  </div>

  <script>
    // Mood Chart
    const ctx = document.getElementById("moodChart").getContext("2d");
    const moodData = JSON.parse(localStorage.getItem("moodEntries")) || [];

    const chart = new Chart(ctx, {
      type: "line",
      data: {
        labels: moodData.map(e => e.date),
        datasets: [{
          label: "Mood Level",
          data: moodData.map(e => e.mood),
          borderColor: "#6a11cb",
          backgroundColor: "#2575fc33",
          tension: 0.4,
          fill: true
        }]
      },
      options: {
        scales: { y: { min: 0, max: 10 } },
        plugins: { legend: { display: false } }
      }
    });

    function saveEntry() {
      const mood = document.getElementById("mood").value;
      const notes = document.getElementById("notes").value;
      if (!mood) return alert("Please select your mood!");

      const entry = {
        mood: parseInt(mood),
        notes,
        date: new Date().toLocaleDateString()
      };

      moodData.push(entry);
      localStorage.setItem("moodEntries", JSON.stringify(moodData));
      chart.data.labels.push(entry.date);
      chart.data.datasets[0].data.push(entry.mood);
      chart.update();

      alert("Entry saved successfully!");
      document.getElementById("mood").value = "";
      document.getElementById("notes").value = "";
    }

    // AI Chat Logic
    function toggleChat() {
      const chat = document.getElementById("chatPopup");
      chat.style.display = chat.style.display === "flex" ? "none" : "flex";
      chat.scrollTop = chat.scrollHeight;
    }

    function sendMessage() {
      const input = document.getElementById("chatInput");
      const text = input.value.trim();
      if (!text) return;
      addMessage("user", text);
      input.value = "";

      setTimeout(() => {
        const reply = getBotReply(text.toLowerCase());
        addMessage("bot", reply);
      }, 600);
    }

    function addMessage(sender, text) {
      const div = document.createElement("div");
      div.className = sender === "user" ? "user-msg" : "bot-msg";
      div.textContent = text;
      document.getElementById("chatBody").appendChild(div);
      const chatBody = document.getElementById("chatBody");
      chatBody.scrollTop = chatBody.scrollHeight;
    }

    function getBotReply(msg) {
      if (msg.includes("stressed") || msg.includes("anxious")) return "Take a deep breath 🌿. Try journaling or a short walk.";
      if (msg.includes("sad")) return "It’s okay to feel sad 😢. Maybe talk to a friend or do something relaxing.";
      if (msg.includes("happy") || msg.includes("great") || msg.includes("good")) return "That’s awesome 😄! Keep that positive energy going!";
      if (msg.includes("angry") || msg.includes("upset")) return "Try a quick break — deep breathing or stretching might help 💨.";
      if (msg.includes("tired")) return "Make sure to rest 😴. A power nap or hydration can work wonders.";
      return "I'm here to listen 💬. Tell me more about your day.";
    }
  </script>
</body>
</html>
