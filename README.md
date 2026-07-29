# Agent Swarm — Multi-Tool AI Assistant for n8n
 
## Overview
Agent Swarm is a production-ready, modular n8n workflow that transforms Telegram into a unified interface for intelligent personal assistance. It leverages LangChain agents to dynamically route user requests—whether text or voice—to purpose-built tool agents (Email, Calendar, Contacts, Web Research, YouTube) backed by real APIs. The system includes memory, logging, error handling, and full observability.

## How It Works
1. **Input Capture**: A Telegram message (text or voice) triggers the workflow.
2. **Voice Handling**: If voice, it’s downloaded and transcribed via OpenAI’s Whisper.
3. **Input Normalization**: Text (or transcribed speech) is routed to the `Input` node and passed to the central `AI Agent`.
4. **Orchestration**: The `AI Agent` uses a custom LLM prompt and routing logic to decide which specialized agent to invoke—e.g., `Email Agent`, `Calendar Agent`, etc.
5. **Tool Execution**: Each agent delegates to its own set of tools (e.g., Gmail API, Google Calendar API, Tavily, Perplexity, Airtable) and uses dedicated LLMs (OpenRouter GPT-4.1-mini variants) for domain-specific reasoning.
6. **Memory & Context**: Conversation history is preserved per chat ID using `Simple Memory` (BufferWindow).
7. **Output & Logging**: Final response is sent back to Telegram; all actions, intermediate steps, and token usage are cleaned up and logged to a Google Sheet.
8. **Error Recovery**: Failed executions trigger an `Error Message` node to notify the user.

## Nodes & Tools Used
- **Triggers & I/O**: `telegramTrigger`, `telegram`
- **Core AI & Orchestration**: `@n8n/n8n-nodes-langchain.agent`, `@n8n/n8n-nodes-langchain.agentTool`, `@n8n/n8n-nodes-langchain.toolThink`, `@n8n/n8n-nodes-langchain.memoryBufferWindow`
- **LLMs**: `@n8n/n8n-nodes-langchain.lmChatOpenRouter` (6 instances), `@n8n/n8n-nodes-langchain.openAi` (transcription)
- **Email Automation**: `gmailTool` (7 operations: send, reply, draft, label, mark unread, get emails/labels)
- **Calendar Management**: `googleCalendarTool` (5 operations: create, get, delete, update, with/without attendees)
- **Contacts**: `airtableTool` (search, upsert)
- **Web & Research**: `@tavily/n8n-nodes-tavily.tavilyTool`, `n8n-nodes-base.perplexityTool`, `n8n-nodes-base.openWeatherMapTool`, `n8n-nodes-base.httpRequestTool` (YouTube scraper)
- **YouTube Ideation**: `googleSheetsTool` (get/add ideas), `httpRequestTool` (Apify YouTube scraper)
- **Utilities**: `n8n-nodes-base.set`, `n8n-nodes-base.switch`, `n8n-nodes-base.code`, `n8n-nodes-base.stickyNote`

## Prerequisites
- An n8n instance (cloud or self-hosted) with LangChain, OpenRouter, and community node support enabled.
- Required API credentials:
  - Telegram Bot Token
  - OpenRouter API Key
  - Gmail OAuth2
  - Google Calendar OAuth2
  - Google Sheets OAuth2
  - Airtable API Token
  - Tavily API Key
  - Perplexity API Key
  - OpenWeatherMap API Key
  - Apify API Token (for YouTube scraping)
- Pre-configured Google Sheets:
  - Agent Action Log ([template](https://docs.google.com/spreadsheets/d/1PlRVi9Iv2n11SPYdshjhPGWccRCpKLHn3PjUkJEjgAQ/edit?usp=sharing))
  - YouTube Ideas Sheet ([template](https://docs.google.com/spreadsheets/d/1Jazczp5HtPwcJvu6bmJQrjkqt31S7adVqIFVCqXZwGk/edit?usp=sharing))
- Optional: Airtable base for contacts (pre-configured in workflow).

## Setup & Usage
1. **Import Workflow**: In n8n, go to *Workflows > Import*, paste the JSON, and confirm.
2. **Configure Credentials**: Replace all `REPLACE_WITH_YOUR_CREDENTIAL_*` placeholders in nodes with valid credential IDs/names.
3. **Update Hardcoded Values**: Adjust Airtable base/table IDs, Google Sheet document IDs, calendar IDs, and Apify API key in respective nodes.
4. **Enable Telegram Webhook**: Ensure your Telegram bot is authorized and the webhook is active (test via `/start`).
5. **Activate Workflow**: Toggle “Active” on and test with commands like *“Send an email to john@example.com asking about the meeting”* or *“What’s the weather in Berlin?”*
6. **Monitor Logs**: All actions appear in your Google Sheet log in real time.

## Use Cases
- **Knowledge Workers & Executives**: Automate daily coordination—email drafting, calendar scheduling, contact lookup—without leaving Telegram.
- **Content Creators**: Research trending topics, fetch high-performing YouTube videos, and log new video ideas instantly.
- **Developers & Automators**: A reference architecture for building scalable, maintainable multi-agent systems in n8n using LangChain composability.
- **Teams Adopting AI Assistants**: Customize agents per department (e.g., HR agent for leave requests, Sales agent for CRM sync) using this modular blueprint.
