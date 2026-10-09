# Book-Keeping-agent

**The problem**- As a fintech PM I know the gap well: users get dozens of transaction alerts a week, but turning them into a budget means either manual entry or handing a third-party app their bank credentials. I wanted the insight without giving up the data.


**Working & Architecture:**
Bank alert emails ──► Postgres (raw_emails)
        │
        ▼
 Mask PII (card / account numbers, CVV, OTP)          ← privacy by design
        │
        ▼
 Local LLM extractor (Qwen 2.5 1.5B, JSON mode)
   is_transaction? amount, vendor, category, date, confidence
        │
        ├── confidence ≥ 0.9 ──► auto-approve ──► expenses table
        └── confidence < 0.9 ──► Telegram: [Approve] [Reject] ──► expenses table
                                                     
 Telegram chat: "How much did I spend on Swiggy last week?"
        └──► Vanna text-to-SQL (trained on schema + golden queries) ──► answer + the SQL it ran



**Product decisions worth calling out**

Privacy first. Sensitive numbers are masked before any model sees the text, and the models run locally with Ollama, so no financial data goes to a third-party LLM API.
Human in the loop where it matters. The model returns a confidence score; only uncertain transactions ask the user for a tap, which keeps the ledger accurate without creating busywork.
Moved from regex to LLM-first extraction. The first version used regex per bank template and kept breaking on small format changes (extra spaces, decimals, new layouts). A single few-shot prompt to a 1.5B model handled all formats, filtered out promotional and OTP emails, and pulled clean merchant names from UPI strings. The iteration log is at the bottom of the notebook.
Right-sized models. A 1.5B model for extraction (fast on a laptop CPU) and text-to-SQL grounded with schema, rules and "golden" example queries to keep answers reliable.
Production habits. Idempotent inserts with a SHA-256 transaction fingerprint (no duplicates), connection pooling with reconnect for a serverless database, stale-button protection on Telegram approvals, and a log of failed queries to improve the prompts.


