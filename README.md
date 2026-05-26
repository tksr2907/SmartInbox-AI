# 📬 SmartInbox-AI

<div align="center">

> **AI-powered email writing assistant — right in your browser, right when you need it.**

![JavaScript](https://img.shields.io/badge/JavaScript-63.4%25-F7DF1E?style=flat-square&logo=javascript&logoColor=black)
![Java](https://img.shields.io/badge/Java-32.5%25-ED8B00?style=flat-square&logo=openjdk&logoColor=white)
![HTML](https://img.shields.io/badge/HTML-4.1%25-E34F26?style=flat-square&logo=html5&logoColor=white)
![React](https://img.shields.io/badge/React-19.2.6-61DAFB?style=flat-square&logo=react&logoColor=black)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.5.14-6DB33F?style=flat-square&logo=springboot&logoColor=white)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)

[Live Demo](#) · [Report Bug](https://github.com/tksr2907/SmartInbox-AI/issues) · [Request Feature](https://github.com/tksr2907/SmartInbox-AI/issues)

</div>

---

## 📖 About

**SmartInbox-AI** is a full-stack, AI-powered email writing assistant that helps you draft professional, context-aware emails in seconds. It consists of three integrated layers: a **Spring Boot REST backend** that calls the Gemini AI API, a **React + Vite web app** for standalone use, and a **Chrome browser extension** that injects AI assistance directly into Gmail — no copy-pasting needed.

### Why SmartInbox-AI?

- ⚡ **Instant drafts** — generate complete, professional emails from a short prompt
- 🎯 **Tone-aware** — choose Formal, Casual, Friendly, or Assertive styles
- 🧩 **Browser-native** — the extension injects directly into Gmail's compose view
- 🔒 **Self-hosted** — your data stays on your own infrastructure
- 🧱 **Modular** — frontend, backend, and extension are fully decoupled

---

## 🏗️ System Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                              │
│                                                                  │
│  ┌─────────────────────────┐   ┌──────────────────────────────┐  │
│  │   email-writer-react    │   │     email-writer-ext         │  │
│  │   (React 19 + Vite)     │   │   (Chrome Extension MV3)     │  │
│  │                         │   │                              │  │
│  │  ┌──────────────────┐   │   │  ┌────────────────────────┐  │  │
│  │  │   App.jsx        │   │   │  │  content.js            │  │  │
│  │  │   EmailForm.jsx  │   │   │  │  (Injects "AI Reply"   │  │  │
│  │  │   ResultBox.jsx  │   │   │  │   button into Gmail)   │  │  │
│  │  └────────┬─────────┘   │   │  └──────────┬─────────────┘  │  │
│  │           │ Axios POST  │   │             │ fetch() POST   │  │
│  └───────────┼─────────────┘   └─────────────┼────────────────┘  │
│              │                               │                   │
└──────────────┼───────────────────────────────┼───────────────────┘
               │                               │
               └─────────────┬─────────────────┘
                             │ HTTP POST /api/email/generate
                             ▼
┌──────────────────────────────────────────────────────────────────┐
│                       BACKEND LAYER                              │
│              email-writer-sb  (Spring Boot 3.5.14)               │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              EmailGeneratorController                    │   │
│  │          @RestController  @CrossOrigin(*)                │   │
│  │          POST /api/email/generate                        │   │
│  └───────────────────────┬──────────────────────────────────┘   │
│                          │                                       │
│  ┌───────────────────────▼──────────────────────────────────┐   │
│  │               EmailGeneratorService                      │   │
│  │   - Builds Gemini prompt from EmailRequest               │   │
│  │   - Calls Gemini API via WebClient (WebFlux)             │   │
│  │   - Parses & returns generated reply text                │   │
│  └───────────────────────┬──────────────────────────────────┘   │
│                          │                                       │
│  ┌───────────────────────▼──────────────────────────────────┐   │
│  │                   Model / DTOs (Lombok)                  │   │
│  │  EmailRequest  { emailContent: String, tone: String }    │   │
│  │  GeminiResponse { candidates[content[parts[text]]] }     │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
└──────────────────────────────────────────┬───────────────────────┘
                                           │ HTTPS (WebClient)
                                           ▼
┌──────────────────────────────────────────────────────────────────┐
│                    EXTERNAL AI LAYER                             │
│         Google Gemini API (generativelanguage.googleapis.com)    │
│         Model: gemini-pro  |  Auth: API Key via env var          │
└──────────────────────────────────────────────────────────────────┘
```

### Request / Response Flow

```
User types prompt
      │
      ▼
EmailForm.jsx / content.js
  POST /api/email/generate  { emailContent, tone }
      │
      ▼
EmailGeneratorController   ← @RequestBody EmailRequest
      │
      ▼
EmailGeneratorService
  → Builds prompt: "Generate a {tone} email reply for: {emailContent}"
  → WebClient.post() → Gemini API
  → Parses GeminiResponse.candidates[0].content.parts[0].text
      │
      ▼
Returns plain-text email string
      │
      ▼
ResultBox.jsx / Gmail compose box  ← User sees AI draft
```

---

## 📁 Project Structure

```
SmartInbox-AI/
│
├── 📂 email-writer-sb/                           # ── Spring Boot Backend ──
│   ├── pom.xml                                   # Maven config (Java 17, Boot 3.5.14)
│   └── src/
│       └── main/
│           ├── java/com/email/writer/sb/
│           │   ├── EmailWriterSbApplication.java  # @SpringBootApplication entry point
│           │   ├── controller/
│           │   │   └── EmailGeneratorController.java   # POST /api/email/generate
│           │   ├── service/
│           │   │   └── EmailGeneratorService.java      # Gemini API + prompt logic
│           │   └── model/
│           │       ├── EmailRequest.java          # DTO: emailContent + tone
│           │       └── GeminiResponse.java        # Maps Gemini JSON response
│           └── resources/
│               └── application.properties         # API key, URL, CORS config
│
├── 📂 email-writer-react/                         # ── React + Vite Frontend ──
│   ├── package.json                               # React 19.2.6, MUI, Axios, Emotion
│   ├── vite.config.js                             # Dev server + /api proxy to :8080
│   ├── index.html                                 # HTML shell
│   └── src/
│       ├── main.jsx                               # ReactDOM.createRoot entry point
│       ├── App.jsx                                # State: emailContent, tone, reply, loading
│       └── components/
│           ├── EmailForm.jsx                      # Textarea + MUI tone selector
│           └── ResultBox.jsx                      # Output display + copy-to-clipboard
│
├── 📂 email-writer-ext/                           # ── Chrome Extension (Production) ──
│   ├── manifest.json                              # MV3: activeTab + scripting permissions
│   ├── content.js                                 # Injected into Gmail — adds AI button,
│   │                                              #   reads thread, calls backend, fills compose
│   └── background.js                             # Service worker (lifecycle events)
│
├── 📂 hello-world-ext/                            # ── Demo Extension Scaffold ──
│   ├── manifest.json                              # Minimal MV3 example
│   ├── popup.html                                 # Extension popup template
│   └── popup.js                                  # Popup button interaction
│
└── 📂 .idea/                                      # IntelliJ IDEA workspace config
```

---

## 🧩 Module Details

### `email-writer-sb` — Spring Boot Backend

| Class | Annotation | Responsibility |
|---|---|---|
| `EmailWriterSbApplication` | `@SpringBootApplication` | Boots the Spring context |
| `EmailGeneratorController` | `@RestController`, `@CrossOrigin` | Receives `POST /api/email/generate`, delegates to service |
| `EmailGeneratorService` | `@Service` | Builds the Gemini prompt, calls API via `WebClient`, extracts reply text |
| `EmailRequest` | `@Data` (Lombok) | Input DTO — `emailContent: String`, `tone: String` |
| `GeminiResponse` | `@Data` (Lombok) | Output model — deserializes nested Gemini JSON |

**Key `pom.xml` dependencies:**

```xml
spring-boot-starter-web       <!-- REST API -->
spring-boot-starter-webflux   <!-- Non-blocking WebClient for Gemini calls -->
lombok                        <!-- Boilerplate elimination -->
jackson-databind               <!-- JSON serialization / deserialization -->
```

---

### `email-writer-react` — React Frontend

| File | Key Logic |
|---|---|
| `App.jsx` | Owns state (`emailContent`, `tone`, `generatedReply`, `loading`); triggers `axios.post('/api/email/generate', ...)` |
| `EmailForm.jsx` | Controlled `<textarea>` + MUI `<Select>` for tone; calls `onGenerate` prop on submit |
| `ResultBox.jsx` | Renders reply text; `navigator.clipboard.writeText()` for copy button |

**Key `package.json` dependencies:**

```json
"react": "^19.2.6",
"@mui/material": "latest",
"@emotion/react": "latest",
"@emotion/styled": "latest",
"axios": "latest"
```

---

### `email-writer-ext` — Chrome Extension

| File | What it does |
|---|---|
| `manifest.json` | Declares MV3 manifest; `content_scripts` targets `*://mail.google.com/*`; requests `activeTab`, `scripting` |
| `content.js` | Uses `MutationObserver` to detect Gmail's compose window; injects an **"AI Reply"** button; reads email thread text from the DOM; `fetch()`s the Spring Boot backend; writes the response into Gmail's compose `div[contenteditable]` |
| `background.js` | Service worker; handles `chrome.runtime.onInstalled` for first-run setup |

---

### `hello-world-ext` — Demo Scaffold

A stripped-down Chrome extension for developers learning the MV3 extension model. Contains only `manifest.json`, `popup.html`, and `popup.js` — useful as a blank canvas before building `email-writer-ext`.

---

## ⚙️ Quick Start

### Prerequisites

| Tool | Version | Download |
|---|---|---|
| Node.js | ≥ 18.x | [nodejs.org](https://nodejs.org) |
| JDK | 17 | [adoptium.net](https://adoptium.net) |
| Maven | ≥ 3.8 | [maven.apache.org](https://maven.apache.org) |
| Gemini API Key | — | [aistudio.google.com](https://aistudio.google.com) |

### 1. Clone

```bash
git clone https://github.com/tksr2907/SmartInbox-AI.git
cd SmartInbox-AI
```

### 2. Configure the Backend

```bash
cd email-writer-sb
```

Edit `src/main/resources/application.properties`:

```properties
gemini.api.url=https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent
gemini.api.key=YOUR_GEMINI_API_KEY
spring.web.cors.allowed-origins=http://localhost:5173
```

```bash
mvn clean install
mvn spring-boot:run
# Backend runs at http://localhost:8080
```

### 3. Start the Frontend

```bash
cd ../email-writer-react
npm install
npm run dev
# Frontend runs at http://localhost:5173
```

### 4. Load the Extension

1. Go to `chrome://extensions` in Chrome
2. Enable **Developer Mode**
3. Click **Load unpacked** → select `email-writer-ext/`
4. Extension icon appears in the Chrome toolbar ✅

---

## 💻 Usage

### Web App

1. Open `http://localhost:5173`
2. Paste the received email or describe what you want to write
3. Select a tone from the dropdown
4. Click **Generate** — the AI draft appears
5. Click **Copy** to use it in your email client

### Gmail Extension

1. Open [Gmail](https://mail.google.com) in Chrome
2. Open any thread or click **Compose**
3. An **"AI Reply"** button appears in the toolbar
4. Click it — the extension reads the thread and calls the backend automatically
5. The AI-generated draft is written directly into the compose box

---

## 📡 API Reference

**Base URL:** `http://localhost:8080/api`

### `POST /email/generate`

| Field | Type | Required | Description |
|---|---|---|---|
| `emailContent` | `String` | ✅ | The email content or context to reply to |
| `tone` | `String` | ✅ | `professional`, `casual`, `friendly`, `assertive` |

**Example request:**

```bash
curl -X POST http://localhost:8080/api/email/generate \
  -H "Content-Type: application/json" \
  -d '{"emailContent": "Following up on the Q3 proposal.", "tone": "professional"}'
```

**Example response (`200 OK`, `text/plain`):**

```
Dear [Name],

Thank you for your follow-up regarding the Q3 proposal. Our team has reviewed the
submission and we are currently in the final stages of internal evaluation. We will
have a detailed response ready for you by end of week.

Best regards,
[Your Name]
```

---

## 🛠️ Development

### Running Tests

```bash
# Backend
cd email-writer-sb && mvn test

# Frontend
cd email-writer-react && npm run lint && npm run build
```

### Contributing

1. Fork the repository
2. Create a branch: `git checkout -b feature/your-feature`
3. Commit: `git commit -m "feat: your change"` (follow [Conventional Commits](https://www.conventionalcommits.org))
4. Push and open a Pull Request against `main`

---

## 📊 Project Status

**Version:** `v1.0.0-alpha`

**Completed:** Spring Boot + Gemini integration, React UI, Chrome Extension with Gmail injection, CORS support

**In Progress:** Outlook Web extension support, Docker Compose setup

**Roadmap:** Email thread summarisation, saved templates, dark mode, CI/CD pipeline

**Known Limitations:** Extension tested on Chromium only; free-tier backend may have ~50s cold-start delay; Gemini API key required at startup

---

## 📄 License

MIT — see [`LICENSE`](./LICENSE)

---

## 👤 Author

**[tksr2907](https://github.com/tksr2907)** — Built with React, Spring Boot, and Google Gemini.

**[⭐ Star this repo](https://github.com/tksr2907/SmartInbox-AI)** if it helped you!

Issues → [github.com/tksr2907/SmartInbox-AI/issues](https://github.com/tksr2907/SmartInbox-AI/issues)
