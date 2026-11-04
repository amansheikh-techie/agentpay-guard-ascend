# AgentPay Guard — Voice & Conversational AI Test Scenarios

> **GCET ASCEND Week-1 Showcase** | Theme: *Voice AI & Conversational Agents*  
> Repository: `agentpay-guard-ascend`

This document details the test scenarios, execution steps, expected policy decisions, reason codes, and verification criteria for testing both the **Conversational Voice Assistant** and the **Deterministic Spending Safety Engine**.

---

## Pre-Requisites & Environment Setup

1. **Backend Server Running:**
   ```powershell
   cd backend
   .\.venv\Scripts\uvicorn.exe app.main:app --port 8000
   ```
2. **Frontend Running:**
   ```powershell
   cd frontend
   npm run dev
   ```
3. **Browser:** Google Chrome or Microsoft Edge (Chromium with Web Speech API support).
4. **Microphone:** Ensure microphone permissions are granted for `http://localhost:5173`.
5. **Initial Policy Defaults:** Total Budget: `$100.00`, Per-Order Cap: `$40.00`, Approved Merchants: `TechCore`, `HomePlus`, Human Approval: `Required`.
6. **Language Selector:** The voice assistant supports English India (`en-IN`) and Hindi (`hi-IN`). Select your preferred language next to the mic.

---

## Core Showcase Scenarios (The 4 Demo Requests)

### Scenario 1: Allowed In-Policy Purchase (Voice & Text)

| Attribute | Details |
| :--- | :--- |
| **User Request** | *"Find a wireless mouse for work, under $35, from an approved merchant."* |
| **Input Mode** | Voice (Microphone) or Quick Pick / Text |
| **Catalog Match** | `[DEMO] TechCore Wireless Mouse` (ID: `tc-wireless-mouse`, Price: `$29.99 USD`) |
| **Expected Decision** | `ALLOWED` |
| **Reason Codes** | `[]` (empty list — meets all constraints) |
| **Spoken Response** | *"Proposed [DEMO] TechCore Wireless Mouse for $29.99 USD. This purchase is ALLOWED by your policy. Final approval is required in the Approvals tab."* |
| **Human Approval** | Explicit human click required in the Approvals tab to initiate PayPal checkout. |

#### Verification Steps:
1. Navigate to the **🎙️ Voice Assistant** tab.
2. Click the microphone button and clearly speak: *"Find a wireless mouse for work, under $35, from an approved merchant."* (Alternatively, click the **Allowed** quick chip).
3. Observe live transcript pulse while speaking.
4. Verify HTTP 201 response with `policy_decision: "ALLOWED"`.
5. Listen to browser speech synthesis reading the verdict aloud.
6. Observe that payment is **NOT** automatically triggered: the proposal is queued in the **Approvals** tab.
7. Click to the **Approvals** tab and verify the item is ready for manual confirmation.

---

### Scenario 2: Over-Limit Request with Smart Alternatives (Cap Exceeded)

| Attribute | Details |
| :--- | :--- |
| **User Request** | *"I need a high-end mechanical keyboard for typing."* |
| **Input Mode** | Voice (Microphone) or Quick Pick / Text |
| **Catalog Match** | `[DEMO] TechCore Mechanical Keyboard` (ID: `tc-mechanical-keyboard`, Price: `$79.99 USD`) |
| **Expected Decision** | `BLOCKED` |
| **Reason Codes** | `["MAX_ORDER_AMOUNT_EXCEEDED"]` |
| **Spoken Response** | *"Proposed [DEMO] TechCore Mechanical Keyboard for $79.99 USD. This purchase is BLOCKED by MAX_ORDER_AMOUNT_EXCEEDED and cannot be approved."* |
| **Safety Invariant** | Checkout is locked; PayPal order creation is rejected with HTTP 400. |
| **Smart Alternatives** | Recommends in-policy alternatives (e.g. `TechCore Wireless Mouse` at `$29.99`). |

#### Verification Steps:
1. Click the microphone button and speak: *"I need a high-end mechanical keyboard for typing."*
2. Verify HTTP 201 response with `policy_decision: "BLOCKED"` and reason `MAX_ORDER_AMOUNT_EXCEEDED`.
3. Listen to spoken response confirming the purchase is blocked and stating why.
4. Verify that the primary proposal displays a red blocked banner and disabled payment button.
5. Verify the **Smart Alternatives** card appears proposing verified allowed items within the $40.00 cap.
6. Click **"Review this instead →"** on the alternative to switch active proposal to the allowable item.

---

### Scenario 3: Unapproved Merchant Blocked

| Attribute | Details |
| :--- | :--- |
| **User Request** | *"Find a shadowtech gadget for my desk."* |
| **Input Mode** | Voice (Microphone) or Quick Pick / Text |
| **Catalog Match** | `[DEMO] ShadowTech Gadget` (ID: `st-gadget`, Price: `$25.00 USD`, Merchant: `shadowtech`) |
| **Expected Decision** | `BLOCKED` |
| **Reason Codes** | `["MERCHANT_NOT_APPROVED"]` |
| **Spoken Response** | *"Proposed [DEMO] ShadowTech Gadget for $25.00 USD. This purchase is BLOCKED by MERCHANT_NOT_APPROVED and cannot be approved."* |
| **Safety Invariant** | Rules are enforced independently on the server regardless of agent wording. |

#### Verification Steps:
1. Click the microphone button and speak: *"Find a shadowtech gadget for my desk."*
2. Verify HTTP 201 response with `policy_decision: "BLOCKED"` and reason `MERCHANT_NOT_APPROVED`.
3. Confirm spoken response identifies the item, $25.00 USD price, and states it is blocked by `MERCHANT_NOT_APPROVED`.
4. Check that no checkout order can be created for unapproved merchant items.

---

### Scenario 4: Unclear / Unintelligible Speech Rejection

| Attribute | Details |
| :--- | :--- |
| **User Request** | *"uh um hello"* or background ambient noise |
| **Input Mode** | Voice (Microphone) or Text |
| **Expected Decision** | Rejection with HTTP 422 Unprocessable Entity |
| **Response Detail** | `{"detail": "Speech unclear. Please repeat your shopping request with an item name or description."}` |
| **Database Impact** | **0 proposals created in SQLite, 0 orders created, 0 PayPal calls made.** |
| **Spoken Response** | Spoken repeat prompt asking the user to clearly mention the desired item. |

#### Verification Steps:
1. Click the microphone button and say only filler words (e.g. *"uh um"* or *"testing testing"*).
2. Verify the server immediately returns HTTP 422 before invoking AI models or database insertion.
3. Observe the amber notification banner in the UI instructing the user to repeat the request.
4. Check the database / Overview tab: verify proposal count and spend meters remain untouched.

---

## Additional Voice & Edge Case Scenarios

### Scenario 5: Text & Voice Invariance Parity Check
- **Objective:** Verify that the safety engine produces identical policy verdicts, prices, and merchant resolutions regardless of whether the user speaks or types.
- **Steps:**
  1. Submit `"Find a wireless mouse under $35"` via voice (`input_mode: "voice"`). Record product ID, price, and decision.
  2. Submit the identical string via typed text (`input_mode: "text"`).
  3. Compare responses.
- **Expected Result:** Decisions (`ALLOWED`), catalog price (`$29.99`), and reason codes are 100% identical. `input_mode` is recorded in the audit log for telemetry only.

### Scenario 6: Voice Controls (Mute / Unmute & Stop Playback)
- **Objective:** Verify user control over browser speech synthesis.
- **Steps:**
  1. In the Voice Assistant header, click **"Voice Reply ON"** to toggle it to **"Muted"**.
  2. Submit a shopping request.
  3. Verify no audio synthesis plays.
  4. Toggle back to **"Voice Reply ON"** and submit a request.
  5. While the audio is reading the verdict, click the red **"Stop"** button.
- **Expected Result:** Playback stops immediately via `window.speechSynthesis.cancel()`.

### Scenario 7: Microphone Permission Handling
- **Objective:** Verify graceful degradation if microphone permission is denied.
- **Steps:**
  1. In browser site settings, block microphone access and reload.
  2. Click the microphone button.
- **Expected Result:** A clear amber alert appears: *"Microphone permission denied. Please allow microphone access in your browser or type your request."* The typed shopping interface remains fully operational.

### Scenario 8: Unsupported Browser Graceful Degradation
- **Objective:** Ensure no runtime crashes in non-Chromium browsers lacking Web Speech API.
- **Steps:**
  1. Open the application in a browser where `window.SpeechRecognition` and `webkitSpeechRecognition` are undefined.
- **Expected Result:** The microphone button cleanly hides; standard text-based conversational shopping remains 100% functional.

### Scenario 9: Audit Trail Telemetry
- **Objective:** Verify append-only audit trail logging for voice events.
- **Steps:**
  1. Run a voice shopping request.
  2. Navigate to the **Audit Log** tab.
- **Expected Result:** A `PROPOSAL_CREATED` event appears with `details.input_mode: "voice"`, the recognized transcript, and the deterministic `spoken_response`.

### Scenario 10: Demo State Reset
- **Objective:** Restore clean demo defaults for repeatable presentations.
- **Steps:**
  1. Complete a purchase.
  2. Click **"Reset Demo"** on the Overview tab or Policy Overview card.
- **Expected Result:** Budget restores to $100.00, per-order cap to $40.00, spend counters reset to $0.00, and previous payments are preserved in historical records without breaking active counters.

### Scenario 11: Hindi / Hinglish Shopping Request (Voice)

| Attribute | Details |
| :--- | :--- |
| **User Request** | *"मुझे एक वायरलेस माउस चाहिए"* (Hindi) or *"ek badhiya wireless mouse chahiye"* (Hinglish) |
| **Input Mode** | Voice (Microphone with `hi-IN` or `en-IN` selected) |
| **Catalog Match** | `[DEMO] TechCore Wireless Mouse` (ID: `tc-wireless-mouse`, Price: `$29.99 USD`) |
| **Expected Decision** | `ALLOWED` |
| **Reason Codes** | `[]` (meets all spending constraints) |
| **Spoken Response** | *"Proposed [DEMO] TechCore Wireless Mouse for $29.99 USD. This purchase is ALLOWED by your policy. Final approval is required in the Approvals tab."* |
| **Safety Invariant** | Non-empty input of 4+ characters in Hindi/Hinglish modes is accepted by the unclear-speech filter and evaluated by the policy engine. Final checkout still requires explicit human click in Approvals tab. |

#### Verification Steps:
1. Select **Hindi (hi-IN)** or **English India (en-IN)** from the language selector next to the microphone.
2. Click the microphone button and speak: *"मुझे एक वायरलेस माउस चाहिए"* (or in Hinglish: *"ek badhiya wireless mouse chahiye"*).
3. Observe live transcript captures the utterance.
4. Verify HTTP 201 response with `policy_decision: "ALLOWED"`.
5. Listen to browser speech synthesis reading the verdict.
6. Verify the proposal is queued in the **Approvals** tab and no funds are deducted automatically.

---

### Scenario 12: Voice Approval / Payment Command Refusal

| Attribute | Details |
| :--- | :--- |
| **User Request** | *"pay karo"*, *"पे करो"*, *"approve karo"*, or *"kharid lo"* |
| **Input Mode** | Voice (Microphone with `en-IN` or `hi-IN` selected) |
| **Expected Decision** | Rejection with HTTP 422 Unprocessable Entity |
| **Response Detail** | `{"detail": "Voice can't approve or pay. Please use the Approve button after reviewing the purchase."}` |
| **Spoken Response** | Spoken refusal: *"Voice can't approve or pay. Please use the Approve button after reviewing the purchase."* |
| **Safety Invariant** | **Voice must never approve or pay.** 0 proposals created in SQLite, 0 orders created, 0 PayPal calls made. |

#### Verification Steps:
1. Click the microphone button and speak an approval command: *"pay karo"* or *"पे करो"* or *"approve karo"*.
2. Verify the server immediately returns HTTP 422 Unprocessable Entity.
3. Observe the alert banner and spoken response instructing the user that approval requires the manual Approve button.
4. Check the database / Overview tab: verify proposal count, orders, and spend counters remain at 0.

---

## Summary Matrix

| # | Scenario | Prompt | Mode | Expected Verdict | Reason Code |
| :-: | :--- | :--- | :---: | :---: | :---: |
| **1** | Normal In-Policy Purchase | *"Find a wireless mouse under $35..."* | Voice | `ALLOWED` | `[]` |
| **2** | Over Per-Order Cap | *"I need a high-end mechanical keyboard..."* | Voice | `BLOCKED` | `MAX_ORDER_AMOUNT_EXCEEDED` |
| **3** | Unapproved Merchant | *"Find a shadowtech gadget for my desk."* | Voice | `BLOCKED` | `MERCHANT_NOT_APPROVED` |
| **4** | Unclear Speech Rejection | *"uh um hello"* | Voice | HTTP 422 | `UNCLEAR_INPUT` (0 DB writes) |
| **5** | Text Parity Check | Same as Scenario 1 | Text | `ALLOWED` | `[]` (Identical outcome) |
| **6** | Hindi / Hinglish Purchase | *"मुझे एक वायरलेस माउस चाहिए"* / *"ek wireless mouse chahiye"* | Voice (`hi-IN` / `en-IN`) | `ALLOWED` | `[]` |
| **7** | Voice Approval Refusal | *"pay karo"* / *"पे करो"* / *"approve karo"* | Voice (`hi-IN` / `en-IN`) | HTTP 422 | `VOICE_PAYMENT_FORBIDDEN` (0 DB writes) |
