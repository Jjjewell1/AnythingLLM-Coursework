# AnythingLLM — Coursework RAG Assistant

Self-hosted RAG (Retrieval-Augmented Generation) assistant for college coursework.
Uses [AnythingLLM](https://github.com/Mintplex-Labs/anything-llm) with Ollama as the LLM and embedding backend.

## Status: ✅ LIVE

| Piece | Value |
|---|---|
| **Deployed via** | Coolify (GitHub-based build) |
| **URL** | `https://study.jewellcore.com` (HTTP 200, API ping `{"online":true}`) |
| **GitHub repo** | `Jjjewell1/AnythingLLM-Coursework` (branch `main`) |
| **Coolify app** | `qug6cml5i8fumqus82pmb101` |
| **Container port** | 3001 (internal only, Traefik routes externally) |
| **Cloudflare** | Tunnel ingress `study.jewellcore.com → http://localhost:80` + proxied DNS CNAME on tunnel `88da47d7` |
| **LLM** | Ollama on Windows PC (`192.168.1.199:11434`, model `qwen2.5:7b-instruct`) |
| **Embeddings** | Ollama (`nomic-embed-text:latest`) |
| **Vector DB** | LanceDB (built-in, stored in named volume) |

## Infrastructure

| Piece | Value |
|---|---|
| **Deployed via** | Coolify (GitHub-based build) |
| **Domain** | `study.jewellcore.com` |
| **Container port** | 3001 (internal only, Traefik routes externally) |
| **LLM** | Ollama on Windows PC (`qwen2.5:7b-instruct`) |
| **Embeddings** | Ollama (`nomic-embed-text:latest`) |
| **Vector DB** | LanceDB (built-in, stored in volume) |

## Prerequisites

1. **Ollama on your Windows PC** must be reachable from Unraid over LAN.
   Set this environment variable on the Windows PC before starting Ollama:
   ```
   set OLLAMA_HOST=0.0.0.0
   ```
   This binds Ollama to all interfaces so Unraid can connect over LAN (default is `127.0.0.1` only).

2. **Models pulled in Ollama** on the Windows PC:
   ```
   ollama pull qwen2.5:7b-instruct
   ollama pull nomic-embed-text
   ```

## Deploy to Coolify

### 1. Push to GitHub

```bash
git init
git add .
git commit -m "Initial AnythingLLM setup"
git remote add origin https://github.com/Jjjewell1/AnythingLLM-Coursework.git
git branch -M main
git push -u origin main
```

### 2. Create the Coolify Application

In Coolify UI or via API:
- **Build pack:** Docker Compose
- **Repository:** `https://github.com/Jjjewell1/AnythingLLM-Coursework.git`
- **Branch:** `main`

### 3. Set Environment Variables in Coolify

Go to the application's **Environment Variables** tab and set these:

| Variable | Value | Notes |
|---|---|---|
| `OLLAMA_BASE_URL` | `http://192.168.1.199:11434` | Windows PC LAN IP — enables remote server binding (`192.168.1.199`) |
| `JWT_SECRET` | `<RANDOM_32_CHAR_STRING>` | Generate with: `openssl rand -hex 32` |

> All other env vars (LLM provider, model names, embedding config, vector DB) are already set in the compose file.
> The `OLLAMA_BASE_URL` is interpolated into both `OLLAMA_BASE_PATH` and `EMBEDDING_BASE_PATH`.
> Ollama on the PC must be started with `OLLAMA_HOST=0.0.0.0` (see Prerequisites) or it will only listen on localhost and AnythingLLM won't connect.

### 4. Set the Domain (after first deploy)

After the first successful deploy, set the domain in Coolify:
- Domain: `http://study.jewellcore.com` (use `http://`, NOT `https://` — TLS terminates at Cloudflare)

### 5. Cloudflare Tunnel (manual)

Add a tunnel ingress rule in the Cloudflare dashboard:
```
study.jewellcore.com → http://localhost:80
```
Then create a proxied CNAME DNS record:
```
study → <tunnel-id>.cfargotunnel.com
```

### 6. First Login

After deployment, open `https://study.jewellcore.com`. The AnythingLLM setup wizard will walk you through:
- Confirming the Ollama connection
- Selecting `qwen2.5:7b-instruct` as the chat model
- Selecting `nomic-embed-text:latest` as the embedding model
- Creating your first workspace

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Ollama shows "not available" in AnythingLLM | Wrong URL or Ollama not bound to `0.0.0.0` | Check `OLLAMA_BASE_URL` in Coolify env vars. On Windows PC, ensure `OLLAMA_HOST=0.0.0.0` is set before starting Ollama. |
| Document upload succeeds but queries return nothing | Embedding model not configured | Re-check embedding provider in AnythingLLM Settings → Embedding. Ensure `nomic-embed-text` is pulled in Ollama. Re-upload documents after fixing. |
| 404 from Cloudflare | Missing tunnel ingress rule | Add `study.jewellcore.com` to the tunnel ingress config |
| Traefik 404 whitepage | Container not on `coolify` network or wrong port | Verify the compose file has `networks: [coolify]` and port is `3001` |
| Container won't start | `SYS_ADMIN` cap missing | Ensure `cap_add: [SYS_ADMIN]` is present in compose (required for document parsing) |
