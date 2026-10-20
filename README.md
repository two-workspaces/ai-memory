# Deploy and Host AI Memory Server on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/new/template/ai-memory-server?referralCode=sLjDXb&utm_medium=integration&utm_source=button&utm_campaign=ai-memory-server)

[ai-memory](https://github.com/akitaonrails/ai-memory) is a long-term memory server for coding agents. Claude Code, Codex, OpenCode, Gemini CLI, Cursor and others send their sessions to it over MCP and lifecycle hooks. The next session, on any machine and with any agent, starts with a handoff of where you left off and can search everything you did before. It's one Rust binary with SQLite full-text search and a git-versioned markdown wiki.

## About Hosting AI Memory Server

It's one service and a Railway volume.

- **Locked from the first boot.** A random bearer token is generated at deploy time. Every MCP, hook and handoff request must carry it; anything else gets a 401.
- **One server for all your machines.** Your laptop, desktop, cloud agent boxes and teammates point at the same HTTPS address, so memory follows you instead of living in one home directory.
- **Survives redeploys.** The SQLite database and the markdown wiki (with its git history) live on the volume.
- **Small.** The image is about 50 MB and the server idles at a few dozen MB of RAM.

## Common Use Cases

- Picking up a Claude Code session on another computer, or in Codex, without re-explaining the project
- A shared memory for a team's agents: decisions, gotchas and procedures one person's agent learned are found by everyone else's
- Handing work between agent vendors (Claude Code, Codex, OpenCode, Gemini CLI, Kimi, Copilot CLI and more) through one MCP server

## Dependencies for AI Memory Server Hosting

- Nothing else: the database is SQLite on the service's volume
- Optional: an LLM key (Anthropic, OpenAI, Gemini, OpenRouter or any OpenAI-compatible endpoint) for better session summaries and semantic search

### Deployment Dependencies

- [ai-memory on GitHub](https://github.com/akitaonrails/ai-memory)
- [Installation guide (every agent)](https://github.com/akitaonrails/ai-memory/blob/main/docs/install.md)
- [Template source on GitHub](https://github.com/two-workspaces/ai-memory)

### Implementation Details

**Connect an agent.** Install the ai-memory CLI on your machine (see the installation guide: Homebrew, release tarball or the Docker wrapper), then point it at your Railway domain with the `AI_MEMORY_AUTH_TOKEN` value from the Variables tab:

```bash
ai-memory install-mcp   --client claude-code --apply \
    --server-url "https://<your-app>.up.railway.app/mcp" --auth-token "<AI_MEMORY_AUTH_TOKEN>"
ai-memory install-hooks --agent  claude-code --apply \
    --server-url "https://<your-app>.up.railway.app" --auth-token "<AI_MEMORY_AUTH_TOKEN>"
```

Repeat with `--client codex` / `--agent codex`, `opencode`, `gemini-cli`, `cursor` and so on for each agent. Any other MCP client can use the streamable HTTP endpoint `https://<your-app>.up.railway.app/mcp` with an `Authorization: Bearer <token>` header.

**Teammates.** The generated token is the root credential. For a team, give each person their own key with `ai-memory user` / `ai-memory api-key` (see `docs/users.md` upstream) so writes are attributed and keys can be revoked one at a time.

**Better summaries and search.** Without an LLM, ai-memory uses full-text search and rule-based session summaries, which already work. To add LLM consolidation, set for example `AI_MEMORY_LLM_PROVIDER=anthropic` and `ANTHROPIC_API_KEY`, or `AI_MEMORY_LLM_PROVIDER=openai-compat` with `AI_MEMORY_LLM_BASE_URL=https://openrouter.ai/api/v1`, `AI_MEMORY_LLM_MODEL` and `LLM_API_KEY`. For hybrid vector search add `AI_MEMORY_EMBEDDING_PROVIDER=openai` and `OPENAI_API_KEY`.

**Custom domain.** Add its hostname to `AI_MEMORY_ALLOWED_HOSTS` (comma separated). The server refuses requests for hostnames not on that list, which protects it from DNS-rebinding attacks.

**Private network.** Agents running in the same Railway project can use `http://${{AI Memory.RAILWAY_PRIVATE_DOMAIN}}:49374` with the same token. That traffic isn't billed as egress.

**Backups.** Everything is in the volume at `/data`: `wiki/` is a git repository you can push to a private remote, and `db/` holds the SQLite indexes (see `docs/backup.md` upstream).

## Why Deploy AI Memory Server on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying AI Memory Server on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.
