# friendly-fiesta
Agentic RAG: Router-Retriever System with PDF and Web Search Tools
# Agentic RAG System

A minimal but production-shaped multi-agent Retrieval-Augmented Generation
system with three role-based agents:

- **Router Agent** (`agents/router_agent.py`) — classifies each query as
  `pdf`, `web`, or `none` (no retrieval needed).
- **Retriever Agent** (`agents/retriever_agent.py`) — executes the chosen
  strategy: FAISS similarity search over your ingested PDFs, or a live
  Tavily web search.
- **Synthesizer Agent** (`agents/synthesizer_agent.py`) — writes the final
  answer, citing sources by number, and admits when the retrieved context
  isn't enough.

`orchestrator.py` wires the three agents together for a single query.

## Setup (VS Code)

1. Open this folder in VS Code.
2. Create a virtual environment and install dependencies:

   ```bash
   python -m venv .venv
   source .venv/bin/activate      # Windows: .venv\Scripts\activate
   pip install -r requirements.txt
   ```

3. Copy `.env.example` to `.env`, then fill in your keys:

   ```bash
   cp .env.example .env
   ```

   **If you're using Azure OpenAI** (default, `LLM_PROVIDER=azure`), set:
   - `AZURE_OPENAI_API_KEY` — from your Azure resource
   - `AZURE_OPENAI_ENDPOINT` — e.g. `https://your-resource-name.openai.azure.com`
   - `AZURE_CHAT_DEPLOYMENT` / `AZURE_EMBED_DEPLOYMENT` — the **deployment
     names** you created in Azure AI Studio (not the underlying model
     names — e.g. you might have deployed `gpt-4o-mini` under a deployment
     called `my-chat-deployment`)

   **If you're using standard OpenAI**, set `LLM_PROVIDER=openai` and fill
   in `OPENAI_API_KEY`.

   Either way, also set `TAVILY_API_KEY` (free at [tavily.com](https://tavily.com))
   for the web search route.

4. Drop one or more PDFs into `data/pdfs/`.

5. Build the vector index (run again whenever your PDFs change):

   ```bash
   python ingest.py
   ```

6. Run the system:

   ```bash
   python main.py
   # or single-shot:
   python main.py "What does the manual say about warranty terms?"
   ```

## How routing works

The Router Agent asks the LLM to classify the question, expecting strict
JSON like `{"route": "pdf", "reasoning": "..."}`. If that call fails or
returns something unparsable, a keyword heuristic (e.g. "today", "latest",
"current price") kicks in as a fallback so the system degrades gracefully
instead of crashing.

## Swapping components

- **LLM provider**: toggle `LLM_PROVIDER` between `azure` and `openai` in
  `.env` — `agents/llm_client.py` and `agents/embeddings_client.py` handle
  the rest. Everything else talks to the LLM through `.chat(system_prompt,
  user_prompt)`, so no other file needs to change.
- **Web search provider**: swap out `_load_tavily_client`/`retrieve_web` in
  `agents/retriever_agent.py` for another provider (e.g. Bing, Serper) if
  you'd rather not use Tavily.
- **Vector store**: swap `FAISS` in `retriever_agent.py`/`ingest.py` for
  Chroma, Pinecone, etc. — the rest of the pipeline is agnostic to this.

## Extending

Common next steps:
- Add a **Critique/Verifier Agent** that checks the synthesized answer
  against the sources before returning it.
- Add a **hybrid** route that queries both PDF and web and merges results.
- Persist conversation history for multi-turn follow-ups.
