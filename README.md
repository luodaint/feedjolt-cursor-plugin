# Feedjolt Cursor plugin

Cursor plugin that wraps Feedjolt's live Streamable HTTP MCP so the agent can read and triage feedback, the roadmap, and the changelog without leaving the editor.

**Auth in Cursor is OAuth 2.1** (DCR + PKCE S256). Install, Connect, approve in the browser on feedjolt.com, done. No API key in this plugin.

Docs: https://www.feedjolt.com/en/feedback-mcp-server

## Data shape

Organize around these types, not the raw JSON dump:

- `Post = { id, title, body_markdown, board_slug, status, tags, votes, comments, featured, is_internal, owner }`
- `Board = { slug, title, description, visibility, votes_enabled, archived, post_count }`
- `Status = { id, name, color, show_on_roadmap, sort_order }`
- `Tag = { id, name, color, post_count }`
- `Comment = { id, body, is_internal, is_pinned }`
- `ChangelogEntry = { id, title, body_markdown, published, post_ids }`

IDs are UUIDs. Boards are addressed by `slug`. List/search filters use status **name**; mutations use `status_id`.

## What it ships

Two MCP connectors (URL-only — Cursor discovers OAuth itself):

- `feedjolt-reader` → `https://api.feedjolt.com/mcp/reader/`
- `feedjolt-writer` → `https://api.feedjolt.com/mcp/writer/`

Plus a `feedjolt` skill that says when to call them.

Feedjolt is its own OAuth 2.1 authorization server (`/.well-known/oauth-authorization-server`, DCR at `/oauth/register`, PKCE S256). Protected-resource metadata is `/.well-known/oauth-protected-resource/mcp`. JWT `aud` is `https://api.feedjolt.com/mcp`. Reader and writer are aliases of that resource. Cursor owns discovery, DCR, browser consent, and token storage. Do not pre-register a Cursor `CLIENT_ID`.

API keys (`fjk_…`) still work as a **second credential** for Claude, mcp-remote, and curl. They do not belong in this Cursor plugin while OAuth discovery is on (Cursor may ignore Bearer headers once RFC 9728/8414 return 200).

The combined endpoint `https://api.feedjolt.com/mcp/` is the JWT audience and a legacy combined MCP. Do not add it next to the split servers.

The live writer includes create/delete (posts, boards, tags, statuses, comments, changelog). Older marketing copy says it cannot. Trust the tools.

## Install (Cursor)

1. Install `feedjolt` from the Cursor Marketplace (or copy this directory to `~/.cursor/plugins/local/feedjolt` as a real directory).
2. Connect each server when Cursor prompts. Approve in the browser on feedjolt.com.
3. Cursor may show two Connect buttons (reader and writer). Same `aud` can cover both; two prompts are fine.

No plugin variable to set. No `fjk_` key for this path.

## Agent rules

- File posts only from real feedback. Never seed fake posts.
- Do not publish a changelog without a human. `create_changelog_entry` is a draft; `publish_changelog_entry` emails subscribers.
- Prefer `merge_posts` or `change_post_status` over `delete_post`. Prefer `update_board` with `archived=true` over `delete_board`.
