# 🤖 AI Chat Robot for Children

> A child-friendly AI voice assistant built with **ESP32-S3** and **Python** — designed to safely interact with children, detect their emotions, and respond in a fun, educational way.

🎥 **Video Demo**: [Watch on YouTube](https://www.youtube.com/watch?v=6qxjWUGYwhE)

---

## ✨ What It Does

| Feature | Description |
|---|---|
| 🎙️ **Voice Input** | Child holds button to speak |
| 🧠 **Speech Recognition** | Converts audio to text using Whisper (via Groq) |
| 💬 **AI Response** | Generates child-friendly responses using Llama LLM |
| 😊 **Emotion Detection** | Detects emotions (happy, sad, angry, scared, etc.) |
| 🔊 **Text-to-Speech** | Speaks the response using Edge TTS |
| 👁️ **Visual Feedback** | OLED eyes display matching emotions |
| 🛡️ **Safety Detection** | ML-powered toxic/hate speech classifier blocks harmful content |

<img width="883" height="558" alt="image" src="https://github.com/user-attachments/assets/1a5cf196-ea79-43cc-a6e5-77a8f4860a81" />

---

## 🛡️ Safety Detection System

This project includes a **dual-layer child safety system**:

### Layer 1 — Keyword Filter (Real-time)
- Blocks dangerous words, profanity, and inappropriate content instantly
- Built into `server.py` as `check_response_safety()`
- Zero latency, works offline

### Layer 2 — ML Safety Model (Advanced)
An ML-powered toxic/hate speech classifier trained on two real-world datasets:

| Dataset | Size | Labels |
|---|---|---|
| **Jigsaw Toxic Comment** (`train.csv`) | 159,571 entries | toxic, severe_toxic, obscene, threat, insult, identity_hate |
| **Hate Speech** (`labeled_data.csv`) | 24,783 tweets | hate_speech, offensive_language, neither |

**Model Architecture — Stacking Classifier:**
```
TF-IDF Vectorizer
       ↓
┌──────────────────────────────┐
│  Base Classifiers (Level 1)  │
│  • Logistic Regression       │
│  • LinearSVC                 │
│  • Multinomial Naive Bayes   │
│  • SGD Classifier            │
└──────────────────────────────┘
       ↓
┌──────────────────────────────┐
│  Meta Classifier (Level 2)   │
│  • Logistic Regression       │
└──────────────────────────────┘
       ↓
  safe / unsafe prediction
```

**To train and use the safety model:**
```bash
# 1. Download datasets (links in notebook)
#    - train.csv (Jigsaw Toxic Comments)
#    - labeled_data.csv (Hate Speech Dataset)

# 2. Run the notebook
jupyter notebook safety_detection_model.ipynb

# 3. The trained model will be saved as a .pkl file
#    and can be integrated into server.py
```

> See [`safety_detection_model.ipynb`](safety_detection_model.ipynb) for the full training pipeline with EDA, preprocessing, model evaluation, ROC curves, and confusion matrices.

---

## 🔩 Hardware Required

### Electronics
| Part | Quantity | Notes |
|------|----------|-------|
| ESP32-S3 (XIAO) | 1 | Or any ESP32-S3 board |
| Microphone (I2S) | 1 | e.g., INMP441 or SPH0645 |
| Audio Amplifier | 1 | I2S amplifier (MAX98357A) |
| Speaker | 1 | 4Ω 3W speaker |
| OLED Display | 1 | 128x64 SSD1306, I2C |
| Push Button | 1 | Any momentary button |
| LED | 1 | Any color (optional) |
| Wires | ~20 | Dupont connectors |

### Tools
- Computer with USB
- Soldering iron (optional)
- 3D printed case (optional)

---

## 🔌 Wiring Diagram

```
ESP32-S3 → Microphone (INMP441)
┌─────────────────────────────┐
│  MIC_VDD   → 3.3V           │
│  MIC_GND   → GND            │
│  MIC_SCK   → GPIO5 (SCK)    │
│  MIC_WS    → GPIO4 (WS)     │
│  MIC_SD    → GPIO6 (SD)     │
└─────────────────────────────┘

ESP32-S3 → OLED Display
┌─────────────────────────────┐
│  OLED_SDA   → GPIO3 (SDA)   │
│  OLED_SCL   → GPIO44 (SCL)  │
│  OLED_VCC   → 3.3V          │
│  OLED_GND   → GND           │
└─────────────────────────────┘

ESP32-S3 → I2S Amplifier (MAX98357A)
┌─────────────────────────────┐
│  AMP_LRC   → GPIO7          │
│  AMP_BCLK  → GPIO8          │
│  AMP_DIN   → GPIO43         │
│  AMP_VCC   → 5V             │
│  AMP_GND   → GND            │
│  AMP_OUT+  → Speaker +      │
│  AMP_OUT-  → Speaker -      │
└─────────────────────────────┘

ESP32-S3 → Button & LED
┌─────────────────────────────┐
│  BUTTON   → GPIO2 (with 10K pullup)│
│  LED      → GPIO21 (via 330Ω)     │
└─────────────────────────────┘
```

![Wiring Photo](https://github.com/user-attachments/assets/f6334f45-a84b-4a64-add3-623945d13686)

---

## 💻 Software Setup

### 1. Get Groq API Key (Free)

1. Go to [groq.com](https://groq.com)
2. Sign up for a free account
3. Copy your API key from the dashboard

### 2. Install Python & Dependencies

```bash
# Create virtual environment (recommended)
python -m venv venv

# Activate on Windows
venv\Scripts\activate

# Activate on Mac/Linux
source venv/bin/activate

# Install packages
pip install flask flask-cors joblib requests groq python-dotenv edge-tts scikit-learn
```

### 3. Install FFmpeg (Required for TTS)

**Windows:**
```bash
choco install ffmpeg
```
Or download from https://ffmpeg.org/download.html

**Mac:**
```bash
brew install ffmpeg
```

**Linux:**
```bash
sudo apt install ffmpeg
```

---

## ⚙️ Configuration

### 1. Create `.env` File

```env
GROQ_API_KEY=your_groq_api_key_here
```

### 2. Update ESP32 IP Address

In `esp32_robot/esp32_robot.ino`, update:

```cpp
String server_url = "http://YOUR_PC_IP:5050";
```

**Find your IP:**
- **Windows**: Run `ipconfig` in cmd
- **Mac/Linux**: Run `hostname -I` in terminal

---

## ▶️ Running the Server

### Option 1: Batch File (Windows)
```bash
run_server.bat
```

### Option 2: Manual Start
```bash
# Terminal 1 — Start TTS server first
python tts_server.py

# Terminal 2 — Start main server
python server.py
```

**Verify it's working:**
- Main server: http://localhost:5050
- TTS server: http://localhost:8000/tts

---

## 📡 Upload ESP32 Code

### 1. Install Arduino ESP32 Board

1. Open Arduino IDE
2. Go to **File → Preferences**
3. Add to "Additional Boards Manager URLs":
   ```
   https://espressif.github.io/arduino-esp32/package_esp32_index.json
   ```
4. **Tools → Board → Board Manager**
5. Search "ESP32" and install

### 2. Install Required Libraries

**Sketch → Include Library → Manage Libraries:**
- `WiFiManager` by tzapu
- `ArduinoJson` by bblanchon
- `Adafruit_GFX_Library` by adafruit
- `Adafruit_SSD1306` by adafruit

### 3. Upload & Initial Setup

1. Open `esp32_robot/esp32_robot.ino`
2. Select board: **XIAO ESP32S3** or **ESP32S3 Dev Module**
3. Select the correct USB port
4. Click **Upload**
5. Power on → robot creates WiFi AP **"XIAO-Robot-Setup"**
6. Connect with phone/laptop → enter your WiFi credentials
7. Robot restarts and connects to your network

---

## 🕹️ Using the Robot

1. Start both servers on your computer
2. Hold the button on the robot
3. Speak your message (up to 3 seconds)
4. Release the button
5. Robot processes audio → generates response → speaks back
6. OLED eyes display the detected emotion

---

## 🌐 Web Interface

| Page | URL | Purpose |
|------|-----|---------|
| Conversations | http://localhost:5050 | View full chat history |
| Report | http://localhost:5050/report | Child development report |

---

## 🔗 API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/upload` | POST | Send audio, receive AI response |
| `/audio/<filename>` | GET | Retrieve response audio |
| `/conversations` | GET | Get all conversations |
| `/conversations/<id>` | DELETE | Delete a conversation |
| `/conversations/clear` | POST | Clear all history |
| `/conversations/recompute` | POST | Re-detect emotions |
| `/api/report` | GET | Generate child development report |

---

## 📁 Project Structure

```
ai-robot/
├── server.py                    # Main Flask server + safety filter
├── tts_server.py                # Text-to-speech server
├── safety_detection_model.ipynb # ML safety classifier (our addition)
├── .env                         # API key (your file, not committed)
├── .gitignore                   # Git ignore rules
├── README.md                    # This file
├── run_server.bat               # Windows startup script
├── conversations.db             # Chat history (auto-created)
├── esp32_robot/
│   ├── esp32_robot.ino          # Robot firmware
│   └── RoboEyesSimple.h         # OLED eye animation library
└── templates/
    ├── conversations.html       # Chat history web page
    └── report.html              # Development report web page
```

---

## 🔧 Troubleshooting

| Problem | Solution |
|---------|----------|
| "No WiFi" in Serial Monitor | Check WiFi credentials, check ESP32 range |
| "Connection Failed" Error | Verify server is running, check IP in `.ino`, allow port 5050 in firewall |
| Audio Not Playing | Check speaker connection, verify TTS server on port 8000 |
| "API Key" Error | Check `.env` file exists with correct key |

---

## 🏫 College Mini-Project

This project was developed as a college mini-project exploring the intersection of:
- **Hardware**: ESP32-S3 microcontroller + I2S audio + OLED display
- **NLP**: Speech recognition, emotion detection, child-safe language modeling
- **ML Safety**: Toxic/hate speech detection using ensemble classifiers
- **AI**: LLM-powered child-friendly conversation

---

## 🙏 Credits

- [Groq](https://groq.com) — LLM & Whisper API
- [Edge TTS](https://github.com/rany2/edge-tts) — Free text-to-speech
- [WiFiManager](https://github.com/tzapu/WiFiManager) — ESP32 WiFi configuration
- [Jigsaw Toxic Comment Dataset](https://www.kaggle.com/c/jigsaw-toxic-comment-classification-challenge) — Safety model training data
- [Hate Speech Dataset](https://github.com/t-davidson/hate-speech-and-offensive-language) — Safety model training data
