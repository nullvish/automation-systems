# n8n Workflows

Importable [n8n](https://n8n.io) workflows for AI agents with long-term memory. Credentials, instance IDs, and webhook IDs have been stripped from every export, so you attach your own after importing.

## Workflows

| Workflow | Type | Description |
|---|---|---|
| [Julius Memory Agent](#julius-memory-agent) | Main workflow | Chat agent with a reply gatekeeper, per-session chat history, and long-term vector memory |
| [Save Memory to Supabase](#save-memory-to-supabase) | Sub-workflow (agent tool) | Embeds text and stores it in a Supabase vector table, tagged with a session ID |

The two work together: the agent calls **Save Memory to Supabase** as its `save_memory` tool and reads the same table through its `search_memory` tool.

---

## Julius Memory Agent

**File:** [`workflows/julius-memory-agent.json`](workflows/julius-memory-agent.json)

An assistant named Julius that only speaks when it should, and remembers what matters across conversations.

### How it works

```
Chat Message (trigger)
      │
      ▼
Should Julius Reply  ◄── Decision Model (LLM) + Decision Parser (JSON)
      │
      ▼
   Respond? ──── false ──►  Stay Silent
      │ true
      ▼
 Memory Agent  ◄── OpenAI Chat Model
      ├── Postgres Chat Memory   (short-term, per session)
      ├── search_memory          (Supabase vector store, retrieve as tool)
      │        └── Embeddings OpenAI
      └── save_memory            (calls the Save Memory sub-workflow)
```

1. **Chat Message** receives the incoming message (`chatInput`) and a `sessionId`.
2. **Should Julius Reply** is an LLM gatekeeper that returns `{"respond": true|false, "reason": "..."}`. Rules, in priority order: always respond to a direct mention, respond to direct replies, use judgment on relevant conversation, stay quiet during normal human chatter, never force participation, and default to silence when ambiguous.
3. **Respond?** routes on `output.respond`. If false, **Stay Silent** emits `(Julius stayed silent: <reason>)` and nothing is sent to the user.
4. **Memory Agent** answers. It has three kinds of memory:
   - **Postgres Chat Memory**: recent turns for the current session.
   - **search_memory**: semantic search (top 5) over the Supabase `documents` table, used when the user asks about something from earlier.
   - **save_memory**: stores durable facts, preferences, or useful conclusions. The system prompt tells the agent not to save small talk or plain questions.

### Requirements

- n8n 1.x with the AI / LangChain nodes
- OpenAI API key (chat model `gpt-5-mini` and embeddings)
- Supabase project with the vector table from the [setup SQL below](#setup-for-the-memory-table)
- A Postgres database for chat history (the Postgres Chat Memory node creates its table automatically)
- The [Save Memory to Supabase](#save-memory-to-supabase) workflow imported first

### Setup

1. Import **Save Memory to Supabase** first and note its workflow ID (it is in the URL: `/workflow/<id>`).
2. Import `julius-memory-agent.json`.
3. Attach credentials to: **Decision Model**, **OpenAI Chat Model**, **Embeddings OpenAI** (OpenAI), **search_memory** (Supabase), and **Postgres Chat Memory** (Postgres).
4. Open the **save_memory** node and replace `YOUR_SAVE_MEMORY_WORKFLOW_ID` with the ID from step 1.
5. Use the same embedding model here and in the Save Memory workflow. Mixing models makes search results meaningless.
6. Activate the workflow and send a test message.

### Swapping the input source

The workflow ships with n8n's Chat Trigger, but nothing about the logic depends on it. You can feed it from Discord, Telegram, Slack, a plain webhook, or anything else. The only contract is two values: the **message text** and a **session ID**.

**Where the input variables are used**

| Node | Expression | What it needs |
|---|---|---|
| Should Julius Reply | `{{ $json.chatInput }}` | message text |
| Memory Agent | `{{ $('Chat Message').item.json.chatInput }}` | message text |
| save_memory | `{{ $('Chat Message').item.json.sessionId }}` | session ID, stored in memory metadata |
| Postgres Chat Memory | session key (defaults to the Chat Trigger's `sessionId`) | session ID for chat history |

**Recommended method: normalize once, then point three places at it**

1. Replace **Chat Message** with your trigger (Webhook, Telegram Trigger, Slack Trigger, etc.).
2. Add an **Edit Fields (Set)** node right after it named `Normalize Input` that outputs exactly two fields: `chatInput` and `sessionId`.
3. Update the references:
   - **Should Julius Reply**: no change, because it reads `$json.chatInput` from the previous node.
   - **Memory Agent** text: `{{ $('Normalize Input').item.json.chatInput }}`
   - **save_memory** `session_id`: `{{ $('Normalize Input').item.json.sessionId }}`
   - **Postgres Chat Memory**: set *Session ID* to *Define below* and use `{{ $('Normalize Input').item.json.sessionId }}`.
4. Add a send node (Discord, Telegram, Slack, HTTP Request) after **Memory Agent** that posts `{{ $json.output }}` back. Leave the **Stay Silent** branch unconnected so nothing is sent.

**Example mappings** (verify field paths against your own execution data, since payloads vary by node version and relay setup):

| Source | `chatInput` | `sessionId` |
|---|---|---|
| Chat Trigger (default) | `chatInput` | `sessionId` |
| Webhook (custom relay) | `{{ $json.body.content }}` | `{{ $json.body.channel_id }}` |
| Telegram Trigger | `{{ $json.message.text }}` | `{{ $json.message.chat.id }}` |
| Slack Trigger | `{{ $json.text }}` | `{{ $json.channel }}` |

**Choosing the session ID.** This decides how memory is scoped. Channel ID gives a shared conversation per channel. User ID gives a private history per person. Whatever you choose, use the same value everywhere.

### Other things you can swap

- **Models**: **Decision Model** and **OpenAI Chat Model** can be any chat model node (Groq, Anthropic, Ollama). A cheap, fast model suits the gatekeeper.
- **Persona**: the name "Julius" appears in the gatekeeper prompt and the agent system message. Change both together.
- **Vector store**: replace the Supabase nodes with Postgres/pgvector, Pinecone, or similar, in both the agent and the sub-workflow.

### Setup for the memory table

Run in the Supabase SQL Editor:

```sql
create extension if not exists vector;

create table documents (
  id bigserial primary key,
  content text,
  metadata jsonb,
  embedding vector(1536)
);

create function match_documents (
  query_embedding vector(1536),
  match_count int default null,
  filter jsonb default '{}'
) returns table (
  id bigint,
  content text,
  metadata jsonb,
  similarity float
)
language plpgsql
as $$
#variable_conflict use_column
begin
  return query
  select
    id,
    content,
    metadata,
    1 - (documents.embedding <=> query_embedding) as similarity
  from documents
  where metadata @> filter
  order by documents.embedding <=> query_embedding
  limit match_count;
end;
$$;
```

The `1536` dimension matches OpenAI's `text-embedding-3-small`. Change it if you use a different embedding model.

### Limitations

- **The gatekeeper sees one message at a time.** The prompt mentions recent conversation history, but no memory is attached to the gatekeeper, so it cannot tell whether a message is a reply to Julius unless the text says so. Attach a chat memory node to **Should Julius Reply** if you want that.
- **Long-term memory is shared across sessions.** `session_id` is saved in metadata, but `search_memory` does not filter on it, so one session can recall what another stored. Add a metadata filter in the node's options if you need isolation.
- The Stay Silent output is a plain string. With the Chat Trigger in `lastNode` mode it shows up in the chat window, so connect a send node only to the true branch when using other sources.
- No deduplication in the memory store; saving the same fact twice creates two rows.
- Each message costs a gatekeeper call plus an agent call, and each memory save or search adds an embedding request.

---

## Save Memory to Supabase

**File:** [`workflows/save-memory-to-supabase.json`](workflows/save-memory-to-supabase.json)

A small sub-workflow used as a **tool** by the agent above. When the agent decides something is worth remembering, it calls this workflow with the text and a session ID. The workflow embeds the text and inserts it into a Supabase (pgvector) table.

### How it works

```
Execute Workflow Trigger  ──►  Supabase Vector Store (Insert)
   (memory, session_id)            ▲              ▲
                                   │              │
                     Default Data Loader     OpenAI Embeddings
                     (adds metadata)
```

1. **Save Memory Input** (Execute Workflow Trigger) receives `memory` (the text) and `session_id`.
2. **Memory Document** (Default Data Loader) wraps `memory` as the document content and attaches metadata: `session_id` and `stored_at` (ISO timestamp).
3. **Embeddings OpenAI** generates the vector embedding.
4. **Store Memory in Supabase** (insert mode) writes content, metadata, and embedding to the `documents` table.

### Inputs

| Field | Type | Description |
|---|---|---|
| `memory` | string | The text to remember |
| `session_id` | string | Conversation or user identifier, stored in metadata |

### Setup

1. Create the table using the [SQL above](#setup-for-the-memory-table).
2. Import `workflows/save-memory-to-supabase.json`.
3. Attach a *Supabase API* credential to **Store Memory in Supabase** and an *OpenAI API* credential to **Embeddings OpenAI**.
4. Confirm the table name is `documents`.
5. Save the workflow and copy its ID for the agent's **save_memory** node.

### Notes

- This workflow only writes. Reading happens in the agent's `search_memory` node.
- No deduplication, and one embedding request per call.





## Security

Exports in this repo contain no API keys, credential IDs, webhook IDs, or instance identifiers. If you add your own workflows, check exports for hardcoded secrets in HTTP Request and Code nodes before committing.

## License

MIT
