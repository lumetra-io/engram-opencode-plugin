**Shipped: Engram memory plugin for OpenCode**

Jacob from Lumetra here — just shipped an OpenCode integration that gives your agent durable cross-session memory via the hosted Engram MCP server. Same six tools that work in Claude Code, Codex, Goose, OpenClaw, and Paperclip — `store_memory`, `query_memory`, `list_memories`, `list_buckets`, `delete_memory`, `clear_memories`.

What it does: hybrid retrieval (BM25 + vector + knowledge graph) with an explanation trace on every recall, so the agent can cite where it got an answer from.

**Setup (3 steps)**

1. Grab an Engram API key from <https://lumetra.io> (free tier, no card). Looks like `eng_live_…`. Export it: `export ENGRAM_API_KEY="eng_live_..."`
2. **Don't forget BYOK** — Engram is bring-your-own-key end-to-end for the LLM that handles extraction + synthesis. Configure a provider at <https://lumetra.io/models>. DeepSeek is what I'd recommend, cheap and fast. Without one, every store/query returns HTTP 412.
3. Add this to `~/.config/opencode/opencode.json` (or `./opencode.json` for project scope):
   ```json
   {
     "$schema": "https://opencode.ai/config.json",
     "mcp": {
       "engram": {
         "type": "remote",
         "url": "https://mcp.lumetra.io/mcp/sse",
         "enabled": true,
         "headers": {
           "Authorization": "Bearer {env:ENGRAM_API_KEY}"
         }
       }
     }
   }
   ```

Verify with `opencode mcp list` — should show `✓ engram connected`. Then `opencode` and the six tools are available to the agent.

**Heads up on `opencode mcp debug`**

That command tests the OAuth handshake specifically and will show `401 Unauthorized` because it doesn't include the Bearer header. The real runtime path (used during sessions) uses the header correctly. If `mcp list` shows connected, you're fine — ignore the debug output.

**Links**

- GitHub: <https://github.com/lumetra-io/engram-opencode-plugin>
- Engram docs: <https://lumetra.io/docs>

Feedback, bug reports, and PRs welcome.
