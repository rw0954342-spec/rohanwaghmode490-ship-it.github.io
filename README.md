<!DOCTYPE html>
<html lang="hi">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <title>Hindi AI Voice Studio</title>

  <style>
    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: linear-gradient(135deg, #111827, #1e3a8a);
      color: white;
      min-height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
      padding: 20px;
    }

    .container {
      width: 100%;
      max-width: 700px;
      background: rgba(255,255,255,0.08);
      border: 1px solid rgba(255,255,255,0.15);
      border-radius: 24px;
      padding: 25px;
      backdrop-filter: blur(15px);
      box-shadow: 0 20px 50px rgba(0,0,0,0.35);
    }

    .logo {
      text-align: center;
      font-size: 42px;
    }

    h1 {
      text-align: center;
      margin-bottom: 5px;
    }

    .subtitle {
      text-align: center;
      color: #cbd5e1;
      margin-bottom: 25px;
    }

    textarea {
      width: 100%;
      min-height: 180px;
      resize: vertical;
      border: none;
      outline: none;
      border-radius: 15px;
      padding: 18px;
      font-size: 17px;
      background: #f8fafc;
      color: #111827;
    }

    .controls {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 15px;
      margin-top: 18px;
    }

    .control {
      background: rgba(255,255,255,0.08);
      padding: 12px;
      border-radius: 14px;
    }

    label {
      display: block;
      margin-bottom: 8px;
      font-size: 14px;
      color: #cbd5e1;
    }

    select,
    input {
      width: 100%;
      padding: 10px;
      border-radius: 10px;
      border: none;
    }

    .buttons {
      display: flex;
      gap: 12px;
      margin-top: 20px;
    }

    button {
      flex: 1;
      padding: 14px;
      border: none;
      border-radius: 12px;
      font-size: 16px;
      font-weight: bold;
      cursor: pointer;
    }

    #speakBtn {
      background: #22c55e;
      color: white;
    }

    #stopBtn {
      background: #ef4444;
      color: white;
    }

    button:active {
      transform: scale(0.98);
    }

    .status {
      text-align: center;
      margin-top: 18px;
      color: #86efac;
      min-height: 22px;
    }

    .footer {
      text-align: center;
      margin-top: 25px;
      color: #94a3b8;
      font-size: 13px;
    }

    @media(max-width:600px) {
      .controls {
        grid-template-columns: 1fr;
      }

      .buttons {
        flex-direction: column;
      }
    }
  </style>
</head>

<body>

  <div class="container">

    <div class="logo">🎙️</div>

    <h1>Hindi AI Voice Studio</h1>

    <div class="subtitle">
      Hindi Text → Natural Voice
    </div>

    <textarea
      id="text"
      placeholder="यहाँ अपनी Hindi script लिखें..."
    >अगर तुम अमीर बनना चाहते हो, तो सिर्फ पैसे कमाने के बारे में मत सोचो। हर दिन कोई नई skill सीखो और अपने पैसे को समझदारी से इस्तेमाल करो।</textarea>

    <div class="controls">

      <div class="control">
        <label>Hindi Voice</label>

        <select id="voice">
          <option value="">Hindi voice automatically select करें</option>
        </select>
      </div>

      <div class="control">
        <label>Speed</label>

        <input
          id="rate"
          type="range"
          min="0.5"
          max="2"
          value="0.95"
          step="0.05"
        >

        <div id="rateValue">0.95x</div>
      </div>

      <div class="control">
        <label>Pitch</label>

        <input
          id="pitch"
          type="range"
          min="0.5"
          max="2"
          value="1"
          step="0.05"
        >

        <div id="pitchValue">1.00</div>
      </div>

    </div>

    <div class="buttons">

      <button id="speakBtn">
        ▶️ Generate Voice
      </button>

      <button id="stopBtn">
        ⏹️ Stop
      </button>

    </div>

    <div class="status" id="status">
      Ready
    </div>

    <div class="footer">
      Hindi AI Voice Studio • Free Browser Prototype
    </div>

  </div>

<script>

  const textBox = document.getElementById("text");
  const voiceSelect = document.getElementById("voice");

  const rateSlider = document.getElementById("rate");
  const pitchSlider = document.getElementById("pitch");

  const rateValue = document.getElementById("rateValue");
  const pitchValue = document.getElementById("pitchValue");

  const speakBtn = document.getElementById("speakBtn");
  const stopBtn = document.getElementById("stopBtn");

  const status = document.getElementById("status");

  let voices = [];

  function loadVoices() {

    voices = speechSynthesis.getVoices();

    voiceSelect.innerHTML = "";

    const hindiVoices = voices.filter(voice =>
      voice.lang.toLowerCase().startsWith("hi")
    );

    if (hindiVoices.length === 0) {

      const option = document.createElement("option");

      option.textContent =
        "Hindi voice device में उपलब्ध नहीं है";

      option.value = "";

      voiceSelect.appendChild(option);

      return;
    }

    hindiVoices.forEach((voice, index) => {

      const option = document.createElement("option");

      option.value = voices.indexOf(voice);

      option.textContent =
        voice.name + " — " + voice.lang;

      voiceSelect.appendChild(option);

    });

    status.textContent =
      hindiVoices.length + " Hindi voice मिली";
  }

  speechSynthesis.onvoiceschanged = loadVoices;

  loadVoices();


  rateSlider.addEventListener("input", () => {

    rateValue.textContent =
      rateSlider.value + "x";

  });


  pitchSlider.addEventListener("input", () => {

    pitchValue.textContent =
      Number(pitchSlider.value).toFixed(2);

  });


  speakBtn.addEventListener("click", () => {

    const text = textBox.value.trim();

    if (!text) {

      status.textContent =
        "पहले Hindi text लिखिए।";

      return;
    }

    speechSynthesis.cancel();

    const speech =
      new SpeechSynthesisUtterance(text);

    speech.lang = "hi-IN";

    speech.rate =
      Number(rateSlider.value);

    speech.pitch =
      Number(pitchSlider.value);

    const selectedIndex =
      Number(voiceSelect.value);

    if (
      !isNaN(selectedIndex) &&
      voices[selectedIndex]
    ) {

      speech.voice =
        voices[selectedIndex];

    }

    speech.onstart = () => {

      status.textContent =
        "🔊 Hindi voice चल रही है...";

    };

    speech.onend = () => {

      status.textContent =
        "✅ Voice complete";

    };

    speech.onerror = () => {

      status.textContent =
        "❌ Voice generate नहीं हो सकी।";

    };

    speechSynthesis.speak(speech);

  });


  stopBtn.addEventListener("click", () => {

    speechSynthesis.cancel();

    status.textContent =
      "⏹️ Voice stopped";

  });

</script>

</body>
</html>
