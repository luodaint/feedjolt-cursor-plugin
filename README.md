# Feedjolt Cursor plugin

Cursor plugin that wraps Feedjolt's live Streamable HTTP MCP so the agent can read and triage feedback, the roadmap, and the changelog without leaving the editor.

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

Two MCP connectors:

- `feedjolt-reader` → `https://api.feedjolt.com/mcp/reader/`
- `feedjolt-writer` → `https://api.feedjolt.com/mcp/writer/`

Plus a `feedjolt` skill that says when to call them.

Cursor auth is native OAuth: Install → Connect → browser consent on feedjolt.com → token. This plugin does not ask for an API key. This repo has no secrets.

API keys (`fjk_…`) still work as a second credential for Claude, mcp-remote, and curl. They are the non-Cursor fallback, not this plugin.

The combined endpoint `https://api.feedjolt.com/mcp/` is legacy. Do not add it next to the split servers.

The live writer includes create/delete (posts, boards, tags, statuses, comments, changelog). Older marketing copy says it cannot. Trust the tools.

## Install

From the Cursor Marketplace (once listed): install `feedjolt` → Connect → complete browser consent on feedjolt.com → token. Cursor may show two Connect buttons (reader and writer). That is expected.

Locally: copy this directory to `~/.cursor/plugins/local/feedjolt` as a real directory (not a symlink whose target is outside that folder). Connect the same way, then reload.

## Agent rules

- File posts only from real feedback. Never seed fake posts.
- Do not publish a changelog without a human. `create_changelog_entry` is a draft; `publish_changelog_entry` emails subscribers.
- Prefer `merge_posts` or `change_post_status` over `delete_post`. Prefer `update_board` with `archived=true` over `delete_board`.
