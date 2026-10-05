# n8n-arabic-store-agent

An Arabic AI sales assistant for an online store, built as n8n workflows.

It answers customers in the store's web chat around the clock, recommends products from a live catalogue, explains the product domain from a curated knowledge base, and admits when it does not know instead of guessing.

## Background

I built this for a Saudi oud and fragrance retailer where one person handled marketing, and customer questions in the store chat were eating the working day. The same questions came back daily, and the right answers needed real product expertise, not a generic bot reply.

This repository is the sanitised version of the production workflows. Store name, accounts, sheet IDs, endpoints and contact details are replaced with placeholders. The production system prompt and the knowledge base are not included.

In production it has answered 320+ customer questions.

## How it works

```
Store web chat
      |
     n8n  --  AI Agent node
      |
      |-- Product search       Qdrant collection, built from a Google Sheet
      |-- Domain knowledge     Qdrant collection, built from a Markdown file
      |-- Store info           the store platform's MCP server (live data)
      |-- Log unknown question Google Sheets
      `-- Notify team          Gmail
      |
  Postgres chat memory (per session)
```

The agent picks a tool per question:

| Question | Tool |
| --- | --- |
| A product, its price, availability or link | Product search |
| How the product category works, origins, terminology | Domain knowledge |
| Shipping, policies, live store details | Store info (MCP) |
| Anything it cannot ground in a tool result | Log it, notify the team, say so |

## Design decisions

**It only speaks from tool results.** The system prompt forbids answering from the model's general knowledge. The worst acceptable answer is "I don't have a reliable answer for that", never a confident wrong one attributed to the store.

**Unknown questions become a backlog.** When the agent cannot answer, it writes the customer's exact question to a sheet and emails the team. That sheet is how the knowledge base grows.

**Search with the customer's own words.** The product tool is told to pass the product name as the customer typed it, not a paraphrase. Rewriting "is X available?" into a general question was the most common cause of wrong answers during testing.

**Two collections, two jobs.** Product facts and domain knowledge live in separate Qdrant collections with separate tool descriptions, so a question that merely contains a product-category word is not routed to the encyclopaedia.

**No middle layer.** The first version ran the agent in Flowise behind n8n. It worked, and I removed it: the n8n AI Agent node does the same job with one less service and one less database to operate.

## Operations

The parts that keep it honest after launch:

- **Daily heartbeat.** A scheduled run sends a test message to the chat endpoint every morning and emails an alert if the reply is missing or empty.
- **Failure alerts.** An error trigger emails the failed workflow, node and error message for any execution that breaks.
- **Guarded catalogue refresh** (`workflows/catalog-daily-refresh.json`). Rebuilds the product collection from the sheet every night. It refuses to delete the old collection if the sheet returns fewer rows than a minimum, and verifies the point count after rebuilding.
- **Conversation log.** Every exchange is appended to a sheet with the tools the agent used. Test sessions are filtered out.

## Files

| Path | What it is |
| --- | --- |
| `workflows/store-agent.json` | The agent, its tools, memory, logging, heartbeat and failure alerts |
| `workflows/catalog-daily-refresh.json` | Nightly catalogue rebuild with safety checks |
| `prompts/system-message.example.md` | The structure of the system prompt, as an outline |

## Setup

1. Run n8n and Qdrant on the same Docker network so Qdrant is reachable at `http://qdrant:6333`. A Postgres database is needed for chat memory.
2. Import both workflow files into n8n.
3. Create these credentials and attach them where the nodes ask for `YOUR_CREDENTIAL_ID`: OpenAI, Qdrant, Postgres, Google Sheets, Gmail.
4. Replace the placeholders:

| Placeholder | Where |
| --- | --- |
| `YOUR_SHEET_ID` | Google Sheets nodes (product catalogue, unknown questions, conversation log) |
| `https://YOUR_STORE_MCP_ENDPOINT` | The store-info MCP client tool |
| `https://your-n8n.example.com/webhook/YOUR_WEBHOOK_ID/chat` | The heartbeat HTTP request |
| `you@example.com` | Gmail alert recipients |
| System message | The AI Agent node, see `prompts/system-message.example.md` |

5. The product sheet needs these columns, which the data loader joins into one searchable text per product: `الاسم`, `الوصف`, `الدرجة`, `الاستخدام`, `طابع الرائحة`, `خيارات الأحجام`, `فئة الميزانية`, `الرابط`.
6. Put your knowledge file in the path the "Read Knowledge File" node points to, then run the two manual triggers once to build both collections.

## Stack

n8n (AI Agent, LangChain nodes), OpenAI (chat and embeddings), Qdrant, Postgres, Google Sheets, Gmail, MCP.

## License

MIT
