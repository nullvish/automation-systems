# n8n Workflows

A collection of importable [n8n](https://n8n.io) workflows for AI agents and automation. Credentials, instance IDs, and test data have been stripped from every export, so you will need to attach your own credentials after importing.

## Workflows

| Workflow | Type | Description |
|---|---|---|
| [Save Memory to Supabase](#save-memory-to-supabase) | Sub-workflow (agent tool) | Embeds a piece of text and stores it in a Supabase vector table, tagged with a session ID |

---

## Save Memory to Supabase

**File:** [`workflows/save-memory-to-supabase.json`](workflows/save-memory-to-supabase.json)

A small sub-workflow meant to be used as a **tool** by an AI Agent. When the agent decides something is worth remembering, it calls this workflow with the text and a session ID. The workflow embeds the text and inserts it into a Supabase (pgvector) table for later semantic retrieval.

### How it works

```
Execute Workflow Trigger  ──►  Supabase Vector Store (Insert)
   (memory, session_id)            ▲              ▲
                                   │              │
                     Default Data Loader     OpenAI Embeddings
                     (adds metadata)
```

1. **Save Memory Input** (Execute Workflow Trigger) receives two inputs: `memory` (the text to store) and `session_id`.
2. **Memory Document** (Default Data Loader) wraps `memory` as the document content and attaches metadata: `session_id` and `stored_at` (ISO timestamp).
3. **Embeddings OpenAI** generates the vector embedding for the document.
4. **Store Memory in Supabase** (Supabase Vector Store, *insert* mode) writes the content, metadata, and embedding to the `documents` table.

### Inputs

| Field | Type | Description |
|---|---|---|
| `memory` | string | The text to remember |
| `session_id` | string | Identifier for the conversation or user session; stored in metadata so retrieval can be filtered per session |

### Requirements

- n8n with the LangChain / AI nodes available (recent 1.x release)
- A [Supabase](https://supabase.com) project with the `pgvector` extension
- An OpenAI API key (used for embeddings)

### Setup

**1. Create the table and match function in Supabase** (SQL Editor):

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

> The `1536` dimension matches OpenAI's `text-embedding-3-small` / `ada-002`. If you change the embedding model, change the dimension to match. The same embedding model must be used when you later query the table.

**2. Import the workflow**

In n8n: **Workflows → Import from File** and select `workflows/save-memory-to-supabase.json`.

**3. Attach credentials**

- **Store Memory in Supabase**: create a *Supabase API* credential (project URL and service role key).
- **Embeddings OpenAI**: create an *OpenAI API* credential.

**4. Check the table name**

The Supabase node is set to the `documents` table. Change it if you named yours differently.

### Using it as an agent tool

1. Save and activate this workflow.
2. In your AI Agent workflow, add a **Call n8n Workflow Tool** node and select this workflow.
3. Give the tool a clear description, e.g. *"Saves an important fact or piece of context to long-term memory. Use when the user shares something worth remembering."*
4. Map the `memory` and `session_id` inputs. `session_id` is typically the same value you use for the agent's chat memory.

### Notes and limitations

- This workflow only **writes** memory. You need a separate retrieval path (a Supabase Vector Store node in *retrieve-as-tool* mode, or an equivalent) to read it back.
- There is no deduplication: saving the same text twice creates two rows.
- Every call makes one embedding API request, so very frequent saves will add up in cost.
- Memory is only as good as what the agent chooses to store; tune the tool description to control this.

---

## Repo structure

```
.
├── README.md
└── workflows/
    └── save-memory-to-supabase.json
```

## Security

Exports in this repo contain no API keys, credential IDs, or instance identifiers. If you fork this repo and add your own workflows, check exports for hardcoded secrets in HTTP Request and Code nodes before committing.

## License

MIT
