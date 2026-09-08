# E-commerce Support Ticket Triage & Escalation Workflow (Dify)

> A Dify-based automation workflow for e-commerce customer support tickets: **~25 nodes / 33 edges**, auto-routing across 5 issue categories, with a dual-channel escalation alert (Feishu Webhook + local qwen2.5:7b inference).

**🌐 [简体中文](./README.md) | English**

## Overview

The workflow takes **customer message + order ID + customer tier** as input, then uses a local LLM to perform **intent recognition / sentiment analysis / urgency scoring**, and automatically:

- **Intent classification**: 4 top-level categories — pre-sales / after-sales / complaint / other
- **Issue sub-typing**: 5 sub-categories — logistics / product quality / return & exchange / pricing & promotion / account & payment
- **Escalation alert**: when sentiment is "angry" or **urgency >= 4**, a supervisor alert is fired through a **Feishu Webhook**
- **Reply generation**: the model produces a handling plan (responsible team + SLA)
- **Fallback branch**: "other" issues get a polite redirect instead of running the main chain pointlessly

## Tech Stack

| Category | Tool / Model |
|---|---|
| Workflow orchestration | Dify (visual editor) |
| LLM inference | Ollama, locally hosted qwen2.5:7b |
| Notification | Feishu custom bot Webhook |
| Code nodes | Python (builds the Feishu `msg_type` payload) |
| Retry policy | HTTP node with 3 retries and backoff |

## Dual-Channel Escalation

To prevent losing "just a moment, please" customers and to contain angry ones before they escalate, the workflow uses **two channels**:

1. **Supervisor Feishu channel**: a Python node builds the Feishu message body (`msg_type` protocol) -> an HTTP node calls the bot webhook -> 3 retries configured.
2. **Agent ticket channel**: auto-generates ticket details including order ID, customer tier, message summary, suggested owner team, and SLA.

> Fires only when **sentiment == angry** or **urgency >= 4**. Routine issues stay quiet.

## Files

- **`dify-customer-service-workflow.yml`**: the Dify workflow DSL. Import it via *Dify console -> Import App -> Custom*.

## Reproducing It

1. Prepare a machine with qwen2.5:7b deployed (pull the model via Ollama).
2. Prepare a Feishu bot Webhook URL (any group works).
3. Import `dify-customer-service-workflow.yml` into Dify: *Console -> Workflow -> Import DSL -> pick this file*.
4. Configure the **order ID / customer tier / message** inputs on the Start node.
5. In the HTTP Request node (Feishu Webhook), replace the URL with your own webhook.
6. Enable the workflow and test it from the UI or the API.

## Suggested Test Samples

- **Logistics**: "My parcel hasn't moved in 3 days, order #xxx"
- **Return & exchange**: "The quality is bad, I want a refund"
- **Pricing & promotion**: "No price protection on Singles' Day, can I get the difference back?"
- **Account & payment**: "I topped up 100 and it never arrived"
- **Escalation**: "This service is a disgrace, I'm reporting you to 12315!!!" <- urgency 5, triggers the supervisor alert

> *12315 is China's official consumer complaint hotline — a strong escalation signal in Chinese-language support contexts.*

## License

MIT © 2026 XDC-666
