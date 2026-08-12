# Source: https://support.originhq.com/docs/chat-interface

The **Chat Interface** lets you ask natural-language questions about your endpoints, users, agents, and activity — without writing a query or building a filter. Origin answers by querying the analytics graph, so anything visible in the dashboard is fair game to ask about.

## 

Opening the chat

The chat can be launched two ways:

- Click the **chat icon** (speech bubble) in the top bar of the left panel, next to the date range picker
- Press **Ctrl+K** (Windows/Linux) or **Cmd+K** (Mac) from anywhere in the console

![](https://files.readme.io/8b3d56afda198d2712517e6143e77d04a6107de4814bba399203248c6d94f015-origin-chat-june1.png)

## 

Start screen

A new chat opens with a prompt and a set of example questions to help you get started. The agent can query all data you have access to across your tenant.

![](https://files.readme.io/757e90bfdbc20b385ee0a84b2a45d58acd27c976223c8cb831e186a4131df6d3-origin-chat-start.png)

Click any example to use it as your first message, or type your own question in the **Ask the analytics graph** box. Use **Shift+Enter** for a newline, and `/` to see available commands.

## 

Model selection

Click the **MODEL** indicator in the status bar at the bottom of the chat window to open the model configuration panel.

![](https://files.readme.io/672e8cdb85252c687e558515c5964316fc7b8671261133c16e0b308bc44c6303-origin-set-model.png)

The **Model** tab lists available models grouped by status. Each entry shows:

- **Context window** — the maximum token capacity (e.g. 1.1M, 2M)
- **Price** — cost per million input / output tokens
- **RSN** — indicates the model supports reasoning/thinking mode
- **IMG** — indicates the model supports image input

The **TASK MODEL** shown at the bottom (e.g. Anthropic: Claude Haiku 4.5 AUTO) is a smaller model used automatically for internal background tasks such as judging and extraction — it is separate from the main conversation model.

Click any model row to apply it to the current and future conversations.

## 

Provider & API key

The **Provider & API key** tab configures which AI provider and credentials back the chat.

![](https://files.readme.io/8c3bf547fbd10e5d7d9c60d54428c8050733db8f408ded34779226863bc6256d-origin-provider-apikey.png)

Select a **Provider** from the dropdown (e.g. Anthropic, OpenAI) and enter your **API key**. The key is stored on the Origin backend and shared across your tenant — the browser only ever sees whether one is configured, never the key itself. Click **Apply** to save.

## 

MCP

The **MCP** tab connects remote MCP (Model Context Protocol) servers, exposing their tools to the chat agent.

![](https://files.readme.io/5165a577df137e0c972be5edc14ed2ac969df4cdce827222c1dc85be63caa123-origin-mcp.png)

Available integrations include:

| Integration | Auth | Description |
| --- | --- | --- |
| **Linear** | OAuth | Issues, projects, comments, and search |
| **Notion** | OAuth | Pages, databases, and search across your Notion workspace |
| **Zapier** | OAuth | Run any of your Zapier-connected app actions as tools |
| **Firecrawl** | API key | Scrape, crawl, search, and map the web — clean markdown returned for LLM context |
| **Tavily** | API key | Search the web with answer-grade snippets purpose-built for LLM agents |
| **Browserbase** | API key | Drive a real headless browser — navigate, click, fill forms, extract content from JS-rendered pages |

OAuth tokens and API keys for MCP servers are stored in the browser only — they are not sent to the Origin backend. Connected integrations show a **CONNECTED** badge and a **Disconnect** button. Tools from connected servers are available to the agent on the next chat message.

## 

Chat interface

### 

Asking a question

Type your question in the **Ask the analytics graph** box and press **Enter** to submit. The agent reasons over your question and runs one or more tools (e.g. `analytics_query`) against the live dataset. Results are returned as structured tables and/or clickable endpoint chips that link directly to the relevant endpoints in the dashboard.

### 

Response actions

Each agent response includes:

- **↓ PNG** — save the response as an image
- **FORK** — branch the conversation from that point to explore a different follow-up

### 

Conversation actions

The top-right toolbar provides:

- **+** — start a new conversation
- **↓ PDF** — export the full conversation as a PDF
- **⤢** — expand the chat to full screen

### 

Status bar

- **READY** — the agent is idle and ready for the next question
- **MODEL** — the LLM currently powering the chat; click to change it
- **THINK** — toggle extended reasoning on or off for more complex queries

## 

Example questions

- _Give me a list of endpoints and associated users who are actively using Claude Code in the past week_
- _Show me the top 10 tools used across the tenant in the last 7 days, with each tool's share of total tool calls_
- _Plot daily prompt volume for the last 30 days with a 7-day rolling average overlay_
- _Find endpoints whose prompt volume today is ≥1.5× their 7-day baseline mean_
- _Group the last 28 days of prompts by weekday and show me the Mon–Sun pattern_
- _Build me a canvas with the model usage breakdown, latency by provider, and a daily token trend_

 

Copy Page