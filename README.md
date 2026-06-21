# Lead Response Auto-Pilot

An n8n workflow that captures inbound leads from a contact form, scores them with AI, sends a personalized reply within seconds, logs everything to a Google Sheets CRM, and fires a Telegram alert for hot leads so you can call them before a competitor does.

## How It Works

1. **Capture** — An n8n-hosted form collects Name, Email, Phone, and message from the lead.
2. **Analyze** — Claude Sonnet reads the submission and returns four structured fields:
   - `lead_score`: `Hot`, `Warm`, or `Cold` (based on buying intent, urgency, and specificity)
   - `category`: e.g. `Pricing enquiry`, `Support`, `Partnership`, `Spam`
   - `summary`: one-line digest the owner can read in 2 seconds
   - `personalized_reply`: a warm, ≤120-word reply that references what the lead actually asked
3. **Log** — All fields are appended to a Google Sheets CRM row (Timestamp, Name, Email, Phone, Message, Score, Category, Summary, Reply Sent, Status).
4. **Route by score**
   - **Hot or Warm** → send the personalized reply to the lead via Gmail + notify the owner by email with the full score breakdown + fire a Telegram push alert for Hot leads
   - **Cold** → logged to CRM only, no reply sent
5. **Telegram alert** — Hot leads trigger an instant phone notification so you can call while interest is highest.

## Tech Stack

| Tool | Role |
|------|------|
| n8n | Workflow orchestration |
| n8n Form Trigger | Hosted contact form (no extra frontend needed) |
| Claude Sonnet 4.6 | Lead scoring & personalized reply generation |
| Google Sheets | CRM / lead log |
| Gmail OAuth2 | Lead reply + owner notification |
| Telegram Bot | Instant hot-lead phone alert |

## Setup

**Step 1 — Claude credential**
- Connect an Anthropic API credential to the `Claude` node.

**Step 2 — Google Sheets CRM**
- Create a Google Sheet named `Leads CRM` with a sheet tab named `Leads`.
- Add columns: `Timestamp`, `Name`, `Email`, `Phone`, `Message`, `Score`, `Category`, `Summary`, `Reply Sent`, `Status`.
- Connect a Google Sheets OAuth2 credential and update the `Log to CRM Sheet` node with the correct spreadsheet ID.

**Step 3 — Gmail**
- Connect a Gmail OAuth2 credential to `Send Personalized Reply` and `Notify Me with Score`.
- Update the owner notification email address in `Notify Me with Score`.

**Step 4 — Telegram (Hot leads only)**
- Create a Telegram bot via @BotFather and get the API token.
- Find your Telegram Chat ID.
- Connect the Telegram credential and update the `chat_id` in `Phone Alert - Hot Lead`.

**Step 5 — Form URL**
- Activate the workflow. n8n generates a public form URL — embed it as your website contact form's `action` or share the link directly.

## Lead Scoring Logic

| Score | Criteria |
|-------|----------|
| Hot | Ready to buy, asking about price or start date, high urgency |
| Warm | Interested but still exploring options |
| Cold | Vague, spammy, or just browsing |

Only Hot and Warm leads receive an automated reply and owner notification. Cold leads are logged silently.

## Workflow Nodes

```
New Lead Form → Analyze & Draft Reply (Claude)
                         ↓
                 Log to CRM Sheet (Google Sheets)
                         ↓
                  Hot or Warm?
                   ↓         ↓ (Cold — stop)
        Send Personalized Reply (Gmail)
                   ↓
        Notify Me with Score (Gmail)

        Hot or Warm? ──→ Phone Alert - Hot Lead (Telegram)  [parallel branch]
```
