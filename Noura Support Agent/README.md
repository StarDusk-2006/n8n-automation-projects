# Noura Beauty: AI Customer Support Agent (n8n)

A Telegram support bot for a skincare e-commerce shop. It answers customer questions in English, Arabic, or mixed Arabic/English using the shop's real product sheet and a knowledge document, and escalates to a human by email when it can't answer reliably.

> Demo: `[add 30-second video or GIF here]`
> Workflow canvas: `[add screenshot here]`

## The problem

Small online shops get the same questions all day: "How much is the serum?", "Is it available?", "How long is shipping?" Answering by hand is slow, and a chatbot that guesses prices or stock does more harm than good.

## What it does

- Replies to customers on Telegram in the language they write in, including Egyptian Arabic.
- Looks up current prices and product info from a Google Sheet instead of guessing.
- Answers policy questions (shipping, returns, usage) from a Google Doc using RAG.
- Asks a short clarifying question when a product description is ambiguous ("the pink serum").
- Sends an escalation email to human support, with a summary and the customer's Telegram chat ID, when it can't confirm an answer.
- Handles non-text messages with a polite "please send your question as text" reply, and shows a friendly error message if the agent fails.

## How it works

```
Telegram message
   └─ has text? ── no ──> "Please send your question as text"
        │ yes
        ▼
     AI Agent (system prompt: accuracy first, never invent prices/stock)
        ├─ Memory (conversation context)
        ├─ Tool: product price sheet (Google Sheets)
        ├─ Tool: knowledge base (vector store, Gemini embeddings)
        └─ Tool: escalation email (Gmail)
        │
        ├─ success ──> reply on Telegram
        └─ failure ──> "Something went wrong, please try again"

Separate manual flow (run once or when the doc changes):
   Google Doc → text splitter → Gemini embeddings → vector store
```

## Stack

n8n · Telegram Bot API · OpenRouter (free model) · Google Gemini embeddings · Google Sheets · Google Docs · Gmail

## Setup

1. Import `noura-support-agent.json` into n8n.
2. Create credentials: Telegram, OpenRouter, Google Gemini, Google Sheets, Google Docs, Gmail.
3. Point the Sheets tool at your own products sheet and the Docs node at your own knowledge document.
4. Set the escalation email address in the Gmail tool.
5. Run the manual flow once to load the knowledge document into the vector store.
6. Activate the workflow and message your bot.

## Limitations

- The vector store is in-memory, so the knowledge base must be re-loaded after n8n restarts. A persistent store (Supabase, Pinecone, Qdrant) would fix this.
- No order lookup yet. The bot can't check order status.
- Uses a free model, so response quality and speed can vary.

## Next steps

- Persistent vector store
- Order-status lookup
- Log conversations and escalations to a sheet to track common questions
