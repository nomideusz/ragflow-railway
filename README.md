# Deploy and Host RAGFlow on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/new/template/ragflow-engine?utm_medium=integration&utm_source=button&utm_campaign=ragflow-engine)

[RAGFlow](https://ragflow.io/) is an open-source RAG (retrieval-augmented generation) engine built around deep document understanding. It parses PDFs, Word files, slides, spreadsheets, scans and web pages into clean, layout-aware chunks, indexes them for hybrid full-text and vector search, and lets you chat with your knowledge bases through grounded answers that cite their sources. It also has an agent builder, an OpenAI-compatible API and an MCP server. This template runs the official RAGFlow image with Elasticsearch, MySQL, MinIO and Redis, each with its own volume.

## About Hosting RAGFlow

The stack is five services, the same shape as upstream's Docker Compose setup:

- **RAGFlow** (`infiniflow/ragflow:v0.27.2`, pinned): one container runs the web UI behind its built-in nginx, the API server and the document parsing task executor. The admin server and the data-source sync worker are optional (see below), which keeps the default stack lean. It is the only public service. Its health check (`/api/v1/system/healthz`) only passes once MySQL, Redis, Elasticsearch and MinIO all answer.
- **Elasticsearch 8.11** (upstream's tested version): the full-text and vector index. Single-node, password-protected, 1 GB JVM heap, data on a volume. Disk watermarks are off so a small Railway volume does not flip the index to read-only.
- **MySQL 8.0**: users, knowledge bases, documents and settings.
- **MinIO** (the `pgsty/silo` build upstream uses): stores the original uploaded files on a volume.
- **Redis 8**: the parsing task queue and cache, with an append-only file on a volume.

Every password, plus RAGFlow's session signing key and the built-in superuser's password, is generated per deployment; none of upstream's well-known defaults (`infini_rag_flow`, admin password `admin`) are used.

## Common Use Cases

- Chat with a company's handbooks, contracts, manuals or research papers, with every answer linked to the source passage
- A private document Q&A backend for your own app through RAGFlow's REST and OpenAI-compatible APIs
- Agent workflows that retrieve from knowledge bases, call tools and connect to MCP clients
- Turning messy PDFs, scans, tables and slides into structured, searchable chunks you can inspect and fix by hand

## Dependencies for RAGFlow Hosting

- Elasticsearch 8.11 (included, private network only)
- MySQL 8.0 (included, private network only)
- MinIO object storage (included, private network only)
- Redis 8 (included, private network only)
- An API key for at least one model provider (OpenAI, Anthropic, Gemini, Mistral, OpenRouter, Ollama, any OpenAI-compatible endpoint, and many more) for embeddings and chat

### Deployment Dependencies

- [RAGFlow documentation](https://ragflow.io/docs/dev/)
- [RAGFlow on GitHub](https://github.com/infiniflow/ragflow)
- [Template source on GitHub](https://github.com/nomideusz/ragflow-railway)

### Implementation Details

**First login.** Open the RAGFlow service's Railway domain once the deploy is green and sign up. Each account gets its own workspace. Sign-up is open to anyone who reaches the URL (`REGISTER_ENABLED=1`, upstream's default): once your accounts exist, set `REGISTER_ENABLED=0` on the RAGFlow service.

**Optional: admin panel and superuser (`ENABLE_ADMIN_SERVER=true`).** Off by default to save about 1 GB of RAM. Without it there is no `/admin` panel (user management, system settings, roles and whitelist) and no superuser account; regular users sign up and use RAGFlow normally. Set `ENABLE_ADMIN_SERVER=true` on the RAGFlow service and it redeploys with the admin server, which on its first start creates the superuser `admin@ragflow.io` (`DEFAULT_SUPERUSER_EMAIL`) with the generated `ADMIN_DEFAULT_PASSWORD` shown in the service's variables. Log in at `/admin` and change the password; editing the variable later does not change an existing account.

**Optional: data-source sync (`ENABLE_DATA_SYNC=true`).** Off by default to save about 0.8 GB of RAM. It runs the worker that pulls documents on a schedule from connectors such as Confluence, Notion, Google Drive or S3. Uploading files works without it.

**Add a model provider before uploading.** RAGFlow ships without embedded models: in the UI open your avatar, then *Model providers*, add a provider with your API key and pick default chat and embedding models. Knowledge bases use the embedding model to index documents, so parsing waits for this step.

**Resources.** RAGFlow loads OCR and layout models for deep document parsing, so it is not small. With the default (lean) settings the stack idled at about 4.8 GB of RAM in testing: RAGFlow about 2.6 GB, Elasticsearch about 1.7 GB, MySQL, MinIO and Redis about 400 MB together. The parsing worker grows by another 1.5 GB or so once it has handled PDFs, and each optional feature above adds its share. It needs the Hobby or Pro plan, not Trial. The first deploy pulls a ~13 GB image and can take several minutes. For big knowledge bases raise `ES_JAVA_OPTS` (for example `-Xms2g -Xmx2g`) on Elasticsearch.

**Uploads and backups.** Original files live in MinIO, parsed chunks and vectors in Elasticsearch, everything else in MySQL. Back up all three volumes together.

**Custom domain.** Add it to the RAGFlow service in Railway's settings; no variable needs changing.

**Optional features left at upstream's defaults.** The code-execution sandbox (needs Docker-in-Docker, not available on Railway), the MCP server and GPU parsing are off. Infinity, OpenSearch and other document engines are not included; Elasticsearch is upstream's default.

## Why Deploy RAGFlow on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying RAGFlow on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.
