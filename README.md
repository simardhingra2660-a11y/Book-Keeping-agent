# Book-Keeping-agent

**The problem**- As a fintech PM I know the gap well: users get dozens of transaction alerts a week, but turning them into a budget means either manual entry or handing a third-party app their bank credentials. I wanted the insight without giving up the data.


**Working & Architecture:**


<img width="505" height="278" alt="image" src="https://github.com/user-attachments/assets/1743fda3-ee4a-48bc-aded-9ae50596d412" />









**Product decisions worth calling out**

1.Privacy first. Sensitive numbers are masked before any model sees the text, and the models run locally with Ollama, so no financial data goes to a third-party LLM API.

2.Human in the loop where it matters. The model returns a confidence score; only uncertain transactions ask the user for a tap, which keeps the ledger accurate without creating busywork.

3.Moved from regex to LLM-first extraction. The first version used regex per bank template and kept breaking on small format changes (extra spaces, decimals, new layouts). A single few-shot prompt to a 1.5B model handled all formats, filtered out promotional and OTP emails, and pulled clean merchant names from UPI strings. The iteration log is at the bottom of the notebook.

4.Right-sized models. A 1.5B model for extraction (fast on a laptop CPU) and text-to-SQL grounded with schema, rules and "golden" example queries to keep answers reliable.

5.Production habits. Idempotent inserts with a SHA-256 transaction fingerprint (no duplicates), connection pooling with reconnect for a serverless database, stale-button protection on Telegram approvals, and a log of failed queries to improve the prompts.


