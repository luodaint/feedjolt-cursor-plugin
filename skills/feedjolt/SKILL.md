---
name: feedjolt
description: >-
  Use when reading or triaging Feedjolt feedback, the roadmap, or the
  changelog. Prefer these tools over scraping the dashboard.
---

# Feedjolt

Live MCP. Two connectors: `feedjolt-reader` and `feedjolt-writer`. Do not invent posts, votes, comments, or changelog text.

## Data shape

Organize around these types, not the raw JSON dump:

- `Post = { id, title, body_markdown, board_slug, status, tags, votes, comments, featured, is_internal, owner }`
- `Board = { slug, title, description, visibility, votes_enabled, archived, post_count }`
- `Status = { id, name, color, show_on_roadmap, sort_order }`
- `Tag = { id, name, color, post_count }`
- `Comment = { id, body, is_internal, is_pinned }`
- `ChangelogEntry = { id, title, body_markdown, published, post_ids }`

IDs are UUIDs. Boards are addressed by `slug`. Status filters on list/search use the status **name**; mutations use `status_id`.

## When

- Reads (list/search/get, roadmap, changelog): `feedjolt-reader`
- Mutations (status, tags, comments, merges, create/delete, publish): `feedjolt-writer`
- list/search/get first. Then mutate.

The live writer includes create/delete for posts, boards, tags, statuses, comments, and changelog entries. The marketing page still says it does not. Trust the tools.

Do not configure the combined `/mcp/` URL alongside these two connectors.

## Rules

- File a post only from real feedback a human handed over. Never seed fake posts.
- `create_changelog_entry` writes a draft. `publish_changelog_entry` emails subscribers. Do not publish without a human.
- Prefer `merge_posts` or `change_post_status` over `delete_post`. Prefer `update_board` with `archived=true` over `delete_board` (delete only works on an empty board).
- `add_tag_to_post` does not create tags. `list_tags` first.
- Comments on `get_post` cap at 200. If `has_more_comments`, call `list_comments`.

## Do not

- Invent API data.
- Put API keys in chat, git, or the skill.
