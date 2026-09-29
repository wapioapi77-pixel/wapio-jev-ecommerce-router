# Wapio + Jev AI: WhatsApp E-Commerce Triage & Return Classifier

An autonomous, sub-300ms WhatsApp triage and returns management engine for e-commerce, Shopify, and WooCommerce brands built with **n8n**, **Wapio WhatsApp API**, and **Jev AI (TypeSafe AI)**.

---

## ⚡ Overview

Handling e-commerce customer support on WhatsApp with traditional chat models is slow, unpredictable, and risks hallucinated return policies. This workflow uses **Jev AI** to make deterministic, typed classifications in under 300 milliseconds and executes native WhatsApp actions through **Wapio**:

- **⚡ Sub-300ms Classification:** Classifies incoming customer inquiries into four typed categories: `damaged_or_return`, `size_exchange`, `track_order`, and `general_inquiry`.
- **🛡️ Zero Bot Loops:** Built-in filter eliminates infinite reply loops by ignoring bot-generated messages.
- **👕 Native Interactive WhatsApp Polls:** Dispatches 1-tap interactive WhatsApp voting buttons (`sendPoll`) with replacement sizes (S, M, L, XL) for instant customer exchanges.
- **📦 Dynamic Order Tracking:** Intelligently checks for order numbers (e.g. `#1234`) or emails; prompts customers if missing, and acknowledges live courier tracking when provided.
- **🚨 Intelligent Manager Escalation:** Protects store managers from notification fatigue. Only when an unboxing photo is uploaded or an order number is provided does the bot fire an instant priority WhatsApp alert directly to the store manager.

```
WhatsApp Customer ──> Wapio Gateway ──> Wapio Trigger (n8n)
                                              │
                                  Filter Outgoing (fromMe == false)
                                              │
                                  ⚙️ Store & Admin Settings
                                              │
                                  ⚡ Jev AI Decision Engine (<300ms)
                                              │
                                  Router (Category Switch)
                   ┌──────────────┬───────────┴──────────┬──────────────┐
                   ▼              ▼                      ▼              ▼
              📦 Track      👕 Size Poll            🛡️ Damaged     💬 Store FAQ
               Order         (sendPoll)              Return           Menu
                   │              │                      │
                   └──────────────┼──────────────────────┘
                                  ▼
                        Should Notify Admin?
                                  │ (true)
                                  ▼
                    🚨 Alert Store Manager (WhatsApp)
```

---

## 📦 Files in this Repository

| File | Description |
| :--- | :--- |
| `wapio_jev_ecommerce_router.json` | The complete, sanitized n8n workflow JSON ready for direct canvas import. |
| `README.md` | Step-by-step setup, credential configuration, and troubleshooting guide. |

---

## 🛠️ Prerequisites

1. **n8n Instance:** n8n Cloud or self-hosted n8n (v1.0.0+) with community nodes enabled.
2. **Wapio Account & Session:** An active WhatsApp session connected in your [Wapio Dashboard](https://wapio.io/account/whatsapp).
3. **Wapio Personal Access Token (PAT):** Generated from [Wapio API Keys](https://wapio.io/account/api-keys) (starts with `bps_pat_...`).
4. **Webhook Signing Secret:** A private string ($\ge 16$ characters) used for HMAC SHA-256 webhook signature verification.
5. **Jev AI API Key:** An active API key from [Jev AI](https://jev.ai/).

---

## 🚀 Quick Setup (3 Steps)

### Step 1: Install the Wapio Community Node in n8n

1. In your n8n workspace, click **Settings** (gear icon in the bottom-left sidebar).
2. Go to **Community nodes** $\rightarrow$ Click **Install a community node**.
3. In the package name field, enter:
   ```text
   n8n-nodes-wapio
   ```
4. Check the acknowledgment box and click **Install**.

### Step 2: Import the Workflow Template

1. Open a new workflow canvas in n8n.
2. Open `wapio_jev_ecommerce_router.json` from this folder.
3. Select and copy all JSON content, then press <kbd>Ctrl</kbd> + <kbd>V</kbd> (or <kbd>Cmd</kbd> + <kbd>V</kbd> on macOS) on the n8n canvas.

### Step 3: Configure Credentials & Settings

1. **Wapio API Credential:**
   - Double-click the **Wapio Trigger** node.
   - Select or create a new **Wapio API** credential.
   - Enter your **Personal Access Token** (`bps_pat_...`) and your **Webhook Signing Secret** ($\ge 16$ chars).
   - Select your connected **WhatsApp Session** from the dropdown.
2. **Store & Admin Settings:**
   - Open the **⚙️ Store & Admin Settings** node.
   - Set `storeName` to your brand name (e.g. `TrendWear Boutique`).
   - Set `managerPhone` to your WhatsApp phone number with country code (e.g. `15551234567`).
3. **Jev AI Decision Engine:**
   - Open the **⚡ Jev AI Decision Engine** node.
   - Under Headers, replace `YOUR_JEV_API_KEY` with your active Jev API key.
4. **Wapio Action Nodes:**
   - Ensure the remaining Wapio nodes (`Track Order Handler`, `Size Exchange`, `Customer Return Reassurance`, `Alert Store Manager`, and `General Store FAQ`) have your connected session selected.
5. **Activate:**
   - Toggle the workflow switch to **Active** in the top-right corner.

---

## 🧠 Key Architecture Concepts

### 1. Deterministic TypeSafe AI vs Conversational LLMs
Traditional conversational LLMs are slow (2–5 seconds) and frequently hallucinate store policies or offer unauthorized refunds. **Jev AI** uses TypeSafe schema classification to return strictly typed JSON fields (`category`, `has_order_id`, `is_photo_received`, `should_notify_manager`, `churn_risk_score`) in under 300ms.

### 2. Native Interactive WhatsApp Polls (`sendPoll`)
Rather than forcing customers to type numbers or words back and forth, the **Size Exchange** branch leverages Wapio's native `sendPoll` operation. Customers tap their desired replacement size with a single touch directly on their phone screen.

### 3. Intelligent Manager Escalation Gate
To prevent store owners from being bombarded with alert spam:
- When a customer says *"where is my order?"* or *"my shirt is damaged"*, the bot handles them automatically, keeping the manager's phone silent.
- Only when an **unboxing photo is received** or an **order number is provided** does `should_notify_manager` evaluate to `true`, dispatching an immediate priority alert with customer details.

### 4. Bot Loop Prevention (`fromMe`)
Wapio webhooks broadcast all session messages. The `Filter Bot Outgoing (fromMe == false)` If node filters out messages where `fromMe === true`, preventing self-reply infinite loops.

---

## 🔧 Troubleshooting

| Symptom | Likely Cause | Solution |
| :--- | :--- | :--- |
| **"Your request is invalid" on trigger** | Webhook secret is shorter than 16 characters or using a Session Key. | Verify Webhook Signing Secret is $\ge 16$ characters and token is a Personal Access Token (`bps_pat_...`). |
| **No events received on local n8n** | `localhost:5678` is not publicly reachable. | Start a tunnel (`ngrok http 5678`) and set `WEBHOOK_URL="https://your-domain.ngrok-free.dev"`. |
| **Empty session list in dropdown** | Credential configured with Session Key instead of PAT. | Use a Personal Access Token (`bps_pat_...`) for automatic account-level session discovery. |
| **Store manager receives no alerts** | Customer has not uploaded a photo or provided an order number yet. | Expected behavior. Test by sending a photo proof or typing an order number like `#1234`. |
| **Store name appears blank in FAQ** | `storeName` key mismatch in settings node. | Ensure both `storeName` and `StoreName` are defined in the `⚙️ Store & Admin Settings` node. |

---

## 📄 License

MIT License. Built with ❤️ for the [Wapio](https://wapio.io) developer community.
