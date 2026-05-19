# engram-opencode-plugin

[OpenCode](https://opencode.ai) integration for [Engram](https://lumetra.io) — durable, explainable memory for the open-source terminal AI coding agent.

Adds six MCP tools to your OpenCode agent — `store_memory`, `query_memory`, `list_buckets`, `list_memories`, `delete_memory`, `clear_memories` — backed by the hosted Engram MCP server. Same surface as the [Claude Code plugin](https://github.com/lumetra-io/engram-claude-plugin), the [Codex plugin](https://github.com/lumetra-io/engram-codex-plugin), the [OpenClaw skill](https://github.com/lumetra-io/engram-openclaw-skill), the [Goose extension](https://github.com/lumetra-io/engram-goose-extension), the [Paperclip plugin](https://github.com/lumetra-io/engram-paperclip-plugin), and the official [TypeScript](https://github.com/lumetra-io/engram-js), [Python](https://github.com/lumetra-io/engram-py), and [Go](https://github.com/lumetra-io/engram-go) SDKs.

## Setup

You need three things in place: an Engram API key, a BYOK provider key, and an MCP block in your OpenCode config.

### 1. Get an Engram API key

Sign up at <https://lumetra.io> — free tier, no card. You'll see an `eng_live_…` token in your dashboard.

```bash
export ENGRAM_API_KEY="eng_live_..."
```

OpenCode resolves `{env:ENGRAM_API_KEY}` at runtime, so export it in your shell (or in your `~/.zshrc` / `~/.bashrc`) before launching OpenCode.

### 2. Configure a BYOK provider key

Engram is bring-your-own-key end-to-end for the LLM that handles extraction and synthesis. Configure one provider at <https://lumetra.io/models>. DeepSeek is what we recommend — cheap and fast. Without a provider key, every `store_memory` / `query_memory` returns HTTP 412.

### 3. Add Engram to your OpenCode config

Edit `~/.config/opencode/opencode.json` (or `./opencode.json` in your project root for project-scoped config) and add the `mcp.engram` block:

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

That's it. Verify:

```bash
opencode mcp list
```

You should see:

```
●  ✓ engram   connected
│      https://mcp.lumetra.io/mcp/sse
└  1 server(s)
```

Open a session (`opencode`) and the six Engram tools appear alongside the rest.

## Tools exposed

| Tool | What it does |
|---|---|
| `store_memory(content, bucket?)` | Save a fact to a bucket (defaults to `"default"`). Loose-dedup by default. |
| `query_memory(question, bucket?)` | Hybrid retrieval + synthesized answer with citations. |
| `list_memories(bucket, limit?)` | Newest-first list of memories in a bucket (limit 1–100, default 20). |
| `list_buckets(limit?, offset?)` | Paginated list of all buckets in your tenant. |
| `delete_memory(memory_id, bucket)` | Remove a single memory. |
| `clear_memories(bucket)` | Empty a bucket. Destructive. |

## A note on `mcp debug`

If you run `opencode mcp debug engram` to troubleshoot, you'll see `401 Unauthorized` and an OAuth flow message. That command specifically tests the OAuth handshake — it doesn't include the `Authorization: Bearer …` header from your config. The real runtime path (visible in `mcp list` as `✓ connected`) uses the Bearer token correctly. If `mcp list` shows connected, you're fine.

## Manual verification (skip OpenCode entirely)

If something is misbehaving and you want to confirm Engram itself is reachable with your key:

```bash
curl -s https://api.lumetra.io/v1/buckets \
  -H "authorization: Bearer $ENGRAM_API_KEY" | head -c 300
```

You should see a JSON bucket list. If that 401s, the API key is wrong. If that 200s but OpenCode can't connect, the config-file syntax or env-var interpolation is wrong.

## License

MIT — Lumetra
