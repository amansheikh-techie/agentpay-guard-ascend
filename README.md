# AgentPay Guard — Voice AI & Conversational Shopping Agent

> **GCET ASCEND Week-1 Showcase** | Theme: **Voice AI & Conversational Agents**  
> Prototype Environment: PayPal Sandbox (USD only). No real money.

---

### ℹ️ Built On Note
**AgentPay Guard** was originally built for the **PayPal AI Hackathon 2026** as a deterministic safety and policy boundary for AI commerce. For the **GCET ASCEND Week-1 Showcase**, the platform was advanced with a **voice-first conversational layer**, introducing real-time browser speech recognition, spoken policy verdicts, audio playback controls, and voice-specific safety boundaries.

---

## Demo Video

📹 **ASCEND Week-1 Showcase Demo:** [Watch the Video Walkthrough](https://youtu.be/placeholder-demo-video) *(Placeholder — insert final link)*

---

## Problem Statement

As artificial intelligence systems evolve from passive chat interfaces into autonomous agents capable of interacting with payment systems, granting them unrestricted access to financial checkout creates severe risks:

1. **The Voice Checkout Vulnerability:** Voice-first commerce systems that trigger checkout automatically are prone to accidental or unauthorized purchases caused by ambient television audio, background chatter, misheard phonemes, or unverified speech.
2. **AI Hallucinations & Prompt Injections:** Large Language Models (LLMs) cannot serve as security boundaries. Hallucinated parameters or prompt injection attacks can recommend unauthorized products, inflate costs, or route payments to unvetted sellers.
3. **Autonomous Budget Bleed:** Without deterministic, server-enforced velocity limits, conversational bots executing multiple tasks can rapidly exhaust user or enterprise funds.

---

## Solution Overview

**AgentPay Guard** provides a voice-first, policy-guarded shopping assistant that establishes an absolute trust boundary between conversational AI and financial transactions:

- **Voice-First Shopping Agent:** Users speak shopping queries naturally through browser speech input (`SpeechRecognition`) and receive real-time interim transcripts, followed by deterministic spoken replies (`SpeechSynthesis`).
- **Constrained Intent Discovery (Gemini):** Google Gemini (or deterministic keyword fallback) identifies user intent and proposes a catalog product ID. The AI model **never** sets price, merchant, policy verdict, or payment status.
- **Deterministic Policy Engine Deciding:** A server-side Python policy engine independently validates all proposals against spending rules (total budget, per-order caps, velocity limits, and merchant allowlists) before any order can be created.
- **Human-in-the-Loop Approval:** Even when a voice request is within spending limits, **voice alone can never authorize or execute a payment**. The user must review the authoritative summary and explicitly click to authorize checkout.

**Core Principle:** Voice proposes → Gemini identifies → Policy engine decides → Human approves → Backend captures & verifies.

---

## Architecture

```text
[ User Speech / Microphone ] ──► Web Speech API (Browser Recognition + Live Transcript)
                                         │
                                         ▼ (input_mode="voice")
[ FastAPI Backend ] ────────────► Deterministic Pre-Check (Reject unclear noise / filler)
        │                                │
        ▼ (Valid intent)                 ▼
[ Gemini AI / Fallback ] ───────► Suggests catalog product_id ONLY (No pricing authority)
        │
        ▼
[ Policy Engine (Python) ] ─────► Evaluates cap, budget, velocity & merchant allowlist
        │
        ├──► Synthesizes Spoken Verdict ("Proposed [Item] for $[USD]... ALLOWED/BLOCKED by [Rule]")
        ├──► Writes to Immutable Audit Trail (input_mode="voice")
        ▼
[ Mandatory Human Gate ] ───────► User explicitly clicks "Approve & Pay" in Approvals Tab
        │
        ▼
[ PayPal Sandbox Orders API ] ──► Authoritative Server-Side Capture & Cryptographic Verification
```

### Trust Boundary Invariants
- **Backend Authority:** Product prices, approved merchants, policy evaluations, and payment captures are strictly computed on the server from authoritative SQLite records.
- **Voice Discovery Only:** Voice input is purely an interaction mechanism. No payment API can be triggered by speech alone.
- **Input Mode Parity:** Submitting an identical shopping query via voice or text produces identical policy decisions, prices, and merchant resolutions.

---

## Voice Features

1. **Browser Speech Input (Web Speech API):**
   - Click the microphone button in the **🎙️ Voice Assistant** tab to speak natural-language shopping requests.
   - Live visual transcript with audio pulse animation displays recognition progress in real time.
2. **Deterministic Spoken Feedback (`window.speechSynthesis`):**
   - Spoken responses are generated deterministically by the backend rule engine.
   - Verbally recites the selected product name, exact USD price, policy outcome (`ALLOWED` or `BLOCKED` with the explicit rule name), and an approval reminder.
   - Automatically selects natural Indian English (`en-IN`) or US English (`en-US`) voices.
3. **Interactive Audio Controls:**
   - **Voice Reply Toggle:** Easily toggle between "Voice Reply ON" and "Muted".
   - **Stop Audio Button:** Instantly halts in-flight speech synthesis playback (`window.speechSynthesis.cancel()`).
4. **Ambient Noise & Unclear Speech Rejection:**
   - Detects filler words, empty noise, or unintelligible audio before invoking AI models or database queries.
   - Rejects unclear input with HTTP 422 and a spoken repeat prompt without creating database rows or charging budget.
5. **Graceful Browser Fallback:**
   - Supported on Google Chrome, Microsoft Edge, and modern Chromium browsers.
   - In unsupported browsers, the microphone button is cleanly hidden while text-based conversational shopping remains 100% operational.
   - Provides clear visual alerts if microphone permissions are denied.

---

## How to Run

### Prerequisites
- **Python 3.11+** (`python --version`)
- **Node.js 18+** & **npm 9+** (`node --version`, `npm --version`)
- **Git** (`git --version`)
- *(Optional)* Free Google Gemini API key from [Google AI Studio](https://aistudio.google.com/) for live AI mode (offline fallback mode runs automatically if omitted).
- *(Optional)* Free PayPal Developer Sandbox account from [developer.paypal.com](https://developer.paypal.com/) for sandbox payments.

---

### Step 1: Clone and Navigate
```powershell
git clone https://github.com/amansheikh-techie/agentpay-guard.git agentpay-guard-ascend
cd agentpay-guard-ascend
```

---

### Step 2: Backend Setup
```powershell
cd backend

# Create and activate Python virtual environment
python -m venv .venv
.\.venv\Scripts\Activate.ps1

# Install backend dependencies
pip install -r requirements.txt

# Create environment file from template
Copy-Item .env.example .env
```

#### Configure `backend/.env` (Optional)
Open `backend\.env` in an editor. Leave keys empty to use the deterministic offline fallback mode, or add your credentials:
```ini
APP_ENV=development
DATABASE_URL=sqlite:///./agentpay_guard.db
FRONTEND_ORIGIN=http://localhost:5173

# AI Provider (Optional: "gemini", "openai", or "none" for offline fallback)
AI_PROVIDER=gemini
AI_API_KEY=your_gemini_api_key
AI_MODEL=gemini-2.5-flash

# PayPal Sandbox (Optional for sandbox payment captures)
PAYPAL_MODE=sandbox
PAYPAL_CLIENT_ID=your_sandbox_client_id
PAYPAL_CLIENT_SECRET=your_sandbox_client_secret
```
> **Security Notice:** Never commit `.env` or secrets to version control.

---

### Step 3: Start the Backend Server
```powershell
# From the backend/ directory with .venv activated:
uvicorn app.main:app --reload --port 8000
```
- Health Check: [http://localhost:8000/api/health](http://localhost:8000/api/health)
- Interactive Swagger API Docs: [http://localhost:8000/docs](http://localhost:8000/docs)

---

### Step 4: Frontend Setup & Launch
Open a **new terminal** window:
```powershell
cd G:\agentpay-guard-ascend\frontend

# Install dependencies
npm install

# Start Vite development server
npm run dev
```
Open [http://localhost:5173](http://localhost:5173) in Google Chrome or Microsoft Edge. The app opens directly to the **🎙️ Voice Assistant** tab.

---

### Step 5: Running Automated Backend Tests
Open a terminal in `backend/` with the virtual environment activated:
```powershell
cd G:\agentpay-guard-ascend\backend
.\.venv\Scripts\Activate.ps1
pytest tests/ -v
```
*(All 81 automated tests run in fast offline fallback mode with in-memory SQLite).*

---

### Step 6: Building Frontend for Production
```powershell
cd G:\agentpay-guard-ascend\frontend
npm run build
```

---

## Test Scenarios (The 4 Showcase Demo Requests)

These scenarios can be tested hands-free via the microphone or by clicking the quick-scenario buttons in the Voice Assistant tab.

| # | Demo Request | Input Mode | Product Match & Price | Expected Verdict | Reason Code | Spoken Voice Response |
| :-: | :--- | :---: | :--- | :---: | :---: | :--- |
| **1** | *"Find a wireless mouse for work, under $35, from an approved merchant."* | Voice / Text | TechCore Wireless Mouse (`$29.99 USD`) | **`ALLOWED`** | `[]` | *"Proposed [DEMO] TechCore Wireless Mouse for $29.99 USD. This purchase is ALLOWED by your policy. Final approval is required in the Approvals tab."* |
| **2** | *"I need a high-end mechanical keyboard for typing."* | Voice / Text | TechCore Mechanical Keyboard (`$79.99 USD`) | **`BLOCKED`** | `MAX_ORDER_AMOUNT_EXCEEDED` | *"Proposed [DEMO] TechCore Mechanical Keyboard for $79.99 USD. This purchase is BLOCKED by MAX_ORDER_AMOUNT_EXCEEDED and cannot be approved."* (Recommends in-policy alternatives). |
| **3** | *"Find a shadowtech gadget for my desk."* | Voice / Text | ShadowTech Gadget (`$25.00 USD`) | **`BLOCKED`** | `MERCHANT_NOT_APPROVED` | *"Proposed [DEMO] ShadowTech Gadget for $25.00 USD. This purchase is BLOCKED by MERCHANT_NOT_APPROVED and cannot be approved."* |
| **4** | *"uh um hello"* *(or noise/silence)* | Voice | N/A | **HTTP 422** | `UNCLEAR_INPUT` | Server rejects immediately with: *"Speech unclear. Please repeat your shopping request with an item name or description."* **0 database rows, 0 orders, 0 API calls**. |

> For the comprehensive 10-scenario manual test checklist (including audio controls, text parity, permission handling, and demo reset), see [docs/TEST_SCENARIOS.md](./docs/TEST_SCENARIOS.md).

---

## Tech Stack

| Component | Technology | Description |
| :--- | :--- | :--- |
| **Frontend** | React 19 + TypeScript + Vite | Interactive mission control SPA with real-time audio controls. |
| **Voice Recognition** | Web Speech API (`SpeechRecognition`) | Client-side speech-to-text with continuous interim transcript feedback. |
| **Voice Synthesis** | Web Speech API (`SpeechSynthesis`) | Browser-native voice engine with Indian/US English voice preferences. |
| **Backend Framework** | Python 3.11 + FastAPI + Uvicorn | High-performance asynchronous REST API. |
| **Validation & Schema** | Pydantic v2 + Pydantic-Settings | Strict request/response modeling and environment validation. |
| **Database & ORM** | SQLite + SQLAlchemy 2.0 | Authoritative catalog, proposals, payments, and immutable audit logs. |
| **AI / Intent Engine** | Google Gemini / OpenAI-Compatible | Extracts category and suggests candidate catalog product IDs. |
| **Offline Fallback** | Deterministic Keyword Engine | Offline matcher tagged `[Demo Fallback Mode]` when no AI key is provided. |
| **Payment Gateway** | PayPal Orders v2 REST API (Sandbox) | Cryptographically verified server-side payment capture and verification. |
| **Test Suite** | pytest + pytest-asyncio + TestClient | 81 automated tests verifying rules, voice invariants, and API security. |

---

## Team Contributions

| Member | Showcase Role & Responsibilities |
| :--- | :--- |
| **[Name 1]** | **Voice AI & Conversational UX Engineer**<br>• Designed and implemented Web Speech API integration (recognition, interim transcript, and speech synthesis).<br>• Built audio controls (mute/unmute, audio playback cancellation, microphone error recovery).<br>• Reorganized UI to make the Voice Assistant front and center for the ASCEND showcase. |
| **[Name 2]** | **Backend Policy Engine & Payment Guardrails Engineer**<br>• Developed deterministic Python spending policy engine (caps, budgets, merchant allowlists).<br>• Implemented unclear speech rejection (HTTP 422) and deterministic spoken response generation.<br>• Built PayPal Sandbox Orders v2 server capture, approval binding invariant, and automated test suite. |

---

## License

MIT — See [LICENSE](./LICENSE) for details.
