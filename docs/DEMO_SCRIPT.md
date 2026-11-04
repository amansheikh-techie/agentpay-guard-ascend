# AgentPay Guard — Showcase Demo Walkthrough Script

> **GCET ASCEND Week-1 Showcase** | Theme: *Voice AI & Conversational Agents*  
> **Target Video Duration:** ≤ 2 minutes 30 seconds (150 seconds)  
> **Format:** Spoken walkthrough with on-screen browser interaction

---

## Quick Reference: Demo Cue Sheet

| Timestamp | Phase | Exact Spoken User Phrase | Item & Catalog Price | Expected Decision |
| :--- | :--- | :--- | :--- | :--- |
| **0:00 – 0:25** | Problem & Intro | *(Presenter to camera/screen)* | — | — |
| **0:25 – 0:50** | Allowed Purchase | `"Find a wireless mouse for work, under $35, from an approved merchant."` | `[DEMO] TechCore Wireless Mouse` — **$29.99 USD** | `ALLOWED` |
| **0:50 – 1:15** | Over-Limit Request | `"I need a high-end mechanical keyboard for typing."` | `[DEMO] TechCore Mechanical Keyboard` — **$79.99 USD** | `BLOCKED` (`MAX_ORDER_AMOUNT_EXCEEDED`) |
| **1:15 – 1:40** | Unapproved Merchant | `"Find a shadowtech gadget for my desk."` | `[DEMO] ShadowTech Gadget` — **$25.00 USD** | `BLOCKED` (`MERCHANT_NOT_APPROVED`) |
| **1:40 – 2:00** | Speech Rejection | `"uh um hello"` | — | HTTP 422 (`UNCLEAR_INPUT`, 0 DB writes) |
| **2:00 – 2:20** | Human Approval & Pay | *(Click Approve button)* | TechCore Wireless Mouse — **$29.99 USD** | PayPal Sandbox Capture (`COMPLETED`) |
| **2:20 – 2:30** | Audit Log & Wrap | *(Navigate to Audit Log)* | — | Telemetry verification (`input_mode: "voice"`) |

---

## Detailed Spoken Script & Screen Actions

### [0:00 – 0:25] Introduction: The Voice Checkout Vulnerability
**Screen:** Show the AgentPay Guard home screen on the **🎙️ Voice Assistant** tab with the Policy Overview card visible ($100 budget, $40 per-order cap).

**Presenter (Spoken):**
> "Voice AI is transforming how we shop online, but giving an autonomous agent direct access to your wallet is a massive security risk. Ambient audio, conversational hallucinations, or prompt injection can accidentally trigger real financial transactions.
> 
> This is **AgentPay Guard**. We built a voice-first conversational shopping agent with a deterministic spending safety engine and an absolute human-in-the-loop payment boundary. Voice can propose purchases, but voice can *never* authorize a payment."

---

### [0:25 – 0:50] Scenario 1: Allowed In-Policy Request by Voice
**Screen:** Click the microphone button in the Voice Assistant panel. The mic pulses red to indicate live listening.

**Presenter speaks into microphone:**
> *"Find a wireless mouse for work, under $35, from an approved merchant."*

**Action & Observation:**
- Browser speech recognition displays the live transcript.
- The backend matches `[DEMO] TechCore Wireless Mouse` priced at **$29.99 USD**.
- The policy engine evaluates: $29.99 is within the $40.00 cap and TechCore is an approved merchant. Decision: `ALLOWED`.
- The browser speaks aloud via speech synthesis:
  > *"Proposed [DEMO] TechCore Wireless Mouse for $29.99 USD. This purchase is ALLOWED by your policy. Final approval is required in the Approvals tab."*

**Presenter (Spoken Commentary):**
> "Notice that even though the purchase is fully compliant and allowed, payment was **not** automatically triggered. The proposal is safely staged for human review."

---

### [0:50 – 1:15] Scenario 2: Blocked Keyboard Request (Per-Order Cap Exceeded)
**Screen:** Click the microphone button again.

**Presenter speaks into microphone:**
> *"I need a high-end mechanical keyboard for typing."*

**Action & Observation:**
- Transcript captures the request.
- The catalog resolves `[DEMO] TechCore Mechanical Keyboard` at **$79.99 USD**.
- Policy engine strictly evaluates: $79.99 exceeds the active $40.00 per-order limit. Decision: `BLOCKED` with reason code `MAX_ORDER_AMOUNT_EXCEEDED`.
- Assistant speaks aloud:
  > *"Proposed [DEMO] TechCore Mechanical Keyboard for $79.99 USD. This purchase is BLOCKED by MAX_ORDER_AMOUNT_EXCEEDED and cannot be approved."*
- A red blocked alert appears with payment buttons permanently locked.
- Below the blocked card, **Smart Alternatives** automatically proposes the allowable mouse at $29.99 USD.

**Presenter (Spoken Commentary):**
> "The policy engine blocks the $79.99 keyboard deterministically on the server side and suggests compliant in-budget alternatives. An LLM can never override this rule."

---

### [1:15 – 1:40] Scenario 3: Blocked Unapproved Merchant (ShadowTech)
**Screen:** Click the microphone button.

**Presenter speaks into microphone:**
> *"Find a shadowtech gadget for my desk."*

**Action & Observation:**
- The catalog resolves `[DEMO] ShadowTech Gadget` at **$25.00 USD**.
- Even though $25.00 is well below the $40.00 order limit, merchant `shadowtech` is not in the approved merchant list.
- Policy engine returns `BLOCKED` with reason code `MERCHANT_NOT_APPROVED`.
- Assistant speaks aloud:
  > *"Proposed [DEMO] ShadowTech Gadget for $25.00 USD. This purchase is BLOCKED by MERCHANT_NOT_APPROVED and cannot be approved."*

**Presenter (Spoken Commentary):**
> "Even though it is under the price cap, the unapproved vendor is rejected instantly. Both price and merchant constraints are enforced independently."

---

### [1:40 – 2:00] Scenario 4: Unclear Speech Rejection & Voice Payment Boundary
**Screen:** Click the microphone button.

**Presenter speaks into microphone (filler / ambient noise):**
> *"uh um hello"*

**Action & Observation:**
- The system evaluates the utterance as unclear speech.
- The server immediately rejects the request with **HTTP 422 Unprocessable Entity**.
- Assistant speaks aloud:
  > *"Speech unclear. Please repeat your shopping request with an item name or description."*
- **Zero database rows** are created, **zero orders** are initiated, and no external LLM or payment tokens are consumed.

**Presenter (Spoken Commentary):**
> "Unclear speech never pollutes our state. Furthermore, if anyone speaks approval commands like *'approve'* or *'pay'*, the assistant strictly responds: *'Voice can't approve or pay. Please use the Approve button after reviewing the purchase.'*"

---

### [2:00 – 2:20] Scenario 5: Human Approval & PayPal Sandbox Capture
**Screen:** Click the **✅ Approvals** tab.

**Action & Observation:**
1. Locate the staged proposal for `[DEMO] TechCore Wireless Mouse` at **$29.99 USD**.
2. Point out the verified policy status badge: `ALLOWED`.
3. Click the physical **"Approve & Pay ($29.99)"** button.
4. The system executes a final pre-flight verification against current spending limits, generates a PayPal Sandbox order, and captures the funds.
5. The UI updates to a green badge: `COMPLETED` with a live PayPal Sandbox Capture ID.

**Presenter (Spoken Commentary):**
> "Here in the Approvals tab, a real human explicitly reviews the $29.99 purchase and clicks Approve. The server runs a second pre-flight policy check and captures the funds in the PayPal Sandbox."

---

### [2:20 – 2:30] Wrap-Up: Immutable Audit Log
**Screen:** Click the **📋 Audit Log** tab.

**Action & Observation:**
- Display the chronological audit trail.
- Highlight the entries showing `input_mode: "voice"`, the recognized transcript, policy reason codes, and the final payment capture event.

**Presenter (Spoken Commentary):**
> "Every voice prompt, policy decision, and transaction is permanently written to an immutable audit log with full voice telemetry.
> 
> AgentPay Guard combines the speed of conversational voice AI with the bulletproof security of deterministic spending controls. Thank you!"

---

## Demo Checklist Before Recording

1. [ ] Backend is running on port 8000 (`uvicorn app.main:app --port 8000`).
2. [ ] Frontend is running on port 5173 (`npm run dev`).
3. [ ] Microphone is tested and browser voice synthesis is unmuted (header shows **"Voice Reply ON"**).
4. [ ] Click **"Reset Demo"** on the Policy card to start with fresh $100 budget and $0 spent.
5. [ ] PayPal sandbox credentials are loaded in backend environment.
