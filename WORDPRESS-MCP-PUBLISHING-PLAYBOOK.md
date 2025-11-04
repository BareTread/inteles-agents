# Inteles.ro MCP Publishing Playbook (External Agents Edition)

Purpose: a single, self-contained reference you can hand to AI agents that are NOT connected to our MCP stack so they understand how the full, production workflow functions. It includes strict roles, tool usage, token-efficiency rules, duplicate protection, media handling, monetization policy, error recovery, and example payloads.

This is authoritative as of 2025-11-04.

---

## TL;DR

- Orchestrator drives the pipeline: fetch existing content by slug, hand off to writer → image curator → monetizer → publisher, then publish create/update safely.
- Writer (content-quickfire) NEVER calls MCP. It produces exactly one line: the absolute path to the saved `.md` file.
- Image Curator and Monetizer MAY call their own MCP tools (e.g., Pexels, filesystem, whitelist reads). Keep assets WebP, absolute paths, Romanian alt text.
- Duplicate detection is exact-slug with `per_page=1`. Never use `status:"any"`. Avoid broad queries.
- Publisher uses inteles-wordpress MCP: `create_post` / `update_post` plus `create_media` (prefer file:// abs paths). Supports `publish_as_is` for complete HTML.
- Degrade gracefully: if images or products fail/empty, publish without them. Abort on no-op updates.

---

## Agent Roles (Boundaries That Matter)

- General Orchestrator (CLAUDE.md)
  - Owns MCP calls for discovery/duplicate checks via inteles-wordpress.
  - Hands off the final payload to the Publisher.
  - Enforces token discipline (exact slug + per_page=1).

- Writer (content-quickfire)
  - Single responsibility: produce the article content and save to disk.
  - Output: exactly one line with the absolute `.md` path.
  - Prohibited: ALL MCP calls, image search/download, monetization, publishing.

- Image Curator (inteles-image-curator)
  - May use Pexels MCP or local assets; outputs 4–6 WebP images max, abs paths, verified exist, Romanian ALT 50–125 chars.
  - Validates paths before returning; returns JSON only.

- Monetizer (romanian-affiliate-product-finder)
  - Reads whitelist from `04-Monetization/affiliates.csv`; returns ONLY approved merchant product URLs (0–4).
  - Empty return is OK. No generic marketplaces. No homepages unless allowed.

- WordPress Publisher (wordpress-publisher)
  - Publishes or updates via MCP `create_post` / `update_post`.
  - Uploads media via `create_media` (prefer `file://` absolute paths), reuses via `list_media`.
  - Supports `publish_as_is` to publish a complete HTML file exactly as-is.

---

## MCP Tooling Map (WordPress)

- Discovery & duplicate checks (orchestrator):
  - `find_content_by_url(url)`
  - `get_content_by_slug(slug, content_type)` (use `"post"`, fallback to `"page"` if needed)
  - `list_content(content_type, search, per_page, status)` with `per_page=1` and exact slug

- Publish/update (publisher):
  - `create_post(title, content, status, excerpt, author, categories, tags, featured_media, format, slug)`
  - `update_post(id, title, content, status, excerpt, author, categories, tags, featured_media, format, slug)`

- Media (publisher):
  - `list_media(page, per_page, search)` (use filename stem, per_page ≤ 3)
  - `create_media(title, alt_text, caption, description, source_url)` (prefer `file://` abs paths; https allowed)
  - `get_media(id)`, `edit_media(...)`

Never use `status:"any"`. Never run broad keyword searches. Always exact slug + `per_page=1`.

---

## Start-to-Finish Workflow

1) Input handling
   - url → extract strict slug (lowercase, keep hyphens). Try MCP lookup in order:
     1) `find_content_by_url(url)`
     2) `get_content_by_slug(slug, "post")`, else `"page"`
     3) `list_content("post", search=slug, per_page=1, status: publish → draft → pending → private)`
     4) If still none → WebFetch page and clean to plaintext (last resort).
   - text/keyword → clean; keep Romanian diacritics.

2) Writer handoff (content-quickfire)
   - Give cleaned text/keyword + title hint.
   - Receives one line: absolute `.md` path.
   - Writer never touches MCP/images/products/WordPress.

3) Parallel enrich (after writer returns path)
   - Image Curator with `{file_path}` → ≤6 WebP images, abs paths, ALT 50–125 chars, verified `test -f`. If fails, continue without images.
   - Monetizer with `{file_path}` → URLs from whitelist only (0–4). If none, continue without products.

4) Pre-publish duplicate check (orchestrator)
   - Compute title + slug (ASCII kebab; keep diacritics in body).
   - Re-run exact slug lookup with `per_page=1`.
   - Decide `operation=create|update`; capture `post_id` when found.

5) Publish (wordpress-publisher)
   - Input: `operation`, optional `post_id`, `article_path` (.md or .html), images JSON, product URLs.
   - If `article_path` is a complete HTML or `publish_as_is=true` → publish EXACTLY AS-IS.
   - Use `create_post` / `update_post`. Upload via `create_media` (`file://` abs paths) or reuse via `list_media` (≤3). Set hero as `featured_media`.
   - Place products sparsely (max 4), never adjacent to an image in the same paragraph; add `rel="nofollow sponsored noopener"`.

6) Degrade gracefully
   - Image curator fail → publish without images.
   - Monetizer empty/non-approved → publish without products.
   - Update where content hash unchanged → abort to avoid no-op duplicate.

7) Final output
   - `{operation, article_file, post_url, notes}`

---

## Strict Rules & Token Budget

- Exact slug lookups only; `per_page=1` for content, ≤3 for `list_media`.
- Never use `status:"any"`.
- No generic searches (“TVA”)—always exact slug.
- Writer never touches MCP (costly agent). Other agents may use their own MCP tools.

---

## Publisher: Input Contracts

Minimal JSON handoff (markdown path):

```
{
  "operation": "create|update",
  "post_id": 123,                  // only for update
  "article_path": "/abs/file.md", // or .html
  "images": {
    "images": [
      {"role": "hero", "file": "x.webp", "path": "/abs/x.webp", "alt": "…", "caption": null, "priority": 1}
    ]
  },
  "products": ["https://partner.ro/produs-1"],
  "publish_as_is": false
}
```

HTML-direct (publish as-is):

```
{
  "operation": "create",
  "article_path": "/abs/ready.html",
  "publish_as_is": true,
  "title": "Romanian title",
  "slug": "romanian-seo-slug",
  "excerpt": "25–45 word Romanian summary"
}
```

---

## Images: Requirements & Reuse

- WebP only, absolute paths, file exists pre-check.
- Romanian ALT 50–125 chars; one image per major section; include a hero.
- Reuse path: `list_media(search=filename_stem, per_page=3)` and match filename; else upload with `create_media` using `file://`.
- If running MCP-WP in Docker, bind-mount the asset directory and use the container path in `file://` URLs.

Example Docker (host→container):

```
docker run --rm -it \
  -e WP_URL="https://inteles.ro" \
  -e WP_USER="alin@example.com" \
  -e WP_APP_PASSWORD="xxxx xxxx xxxx xxxx" \
  -v /home/alin/DATA/OBSIDIAN/inteles-vault/10-Assets/pexels/processed:/assets:ro \
  ghcr.io/instawp/mcp-wp:latest

// Then use: source_url: "file:///assets/your-image.webp"
```

Remote `https` source_url is supported but `file://` is preferred for idempotency and headers.

---

## Monetizer: Whitelist

- Read `04-Monetization/affiliates.csv` to derive approved merchant domains.
- Output: one product URL per line from approved domains (0–4). No placeholders if none found.
- Publisher double-checks and drops non-whitelisted URLs.

---

## Duplicate-Safe Publishing

- Compute slug from title (ASCII kebab). Do not alter body diacritics.
- Before publish: re-check exact slug (per_page=1). If found → update; else → create.
- On update, abort if content hash unchanged to avoid no-op publishes.

---

## Common Errors & Fixes

- Token overflow on `list_posts` → Never use it. Use `list_content(..., per_page=1)` with exact slug.
- `status:"any"` invalid → Use a concrete status.
- `Nicio dispoziție de conținut furnizată` on media upload → Missing/invalid `source_url`. Use `file://` abs path and ensure container can read it (bind mount if Docker).
- Unknown skill: X → The publisher must not call subagents. Only it uses its MCP tools; writer does not use MCP at all.

---

## Sanity Tests (90 seconds)

1) Tool presence: list MCP tools; confirm `find_content_by_url`, `get_content_by_slug`, `list_content`, `create_post`, `update_post`, `create_media`.
2) Slug probe: `list_content("post", search=slug, per_page=1, status="publish")`, fallback statuses with `per_page=1`.
3) Media dry run: `create_media(source_url="file:///abs/test.webp", title="sanity-test", alt_text="…")` → expect ID + URL.

---

## Offline/External-Agent Guidance (no MCP)

- Generate high-quality `.md` and optionally a ready-to-publish `.html` (Kadence blocks compatible).
- Return:
  - Article file path
  - Image plan (WebP filenames + Romanian ALTs + desired placements)
  - Product URLs (only from approved domains, if known)
- The connected orchestrator/publisher will:
  - Resolve duplicate (exact slug)
  - Upload images (convert or fetch) and publish
  - Insert products with correct `rel` attributes

If you can’t verify whitelists or MCP media, leave those fields empty rather than guessing.

---

## Quick Payload Examples

Minimal create/update handoff to publisher:

```
{
  "operation": "update",
  "post_id": 21600,
  "article_path": "/home/alin/DATA/OBSIDIAN/inteles-vault/articles/tva-2025.md",
  "images": {
    "images": [
      {"role": "hero", "file": "tva-hero.webp", "path": "/home/alin/DATA/OBSIDIAN/inteles-vault/10-Assets/pexels/processed/tva-hero.webp", "alt": "…", "caption": null, "priority": 1}
    ]
  },
  "products": ["https://libris.ro/..."],
  "publish_as_is": false
}
```

HTML as-is publish:

```
{
  "operation": "create",
  "article_path": "/home/alin/DATA/OBSIDIAN/inteles-vault/articles/ready.html",
  "publish_as_is": true,
  "title": "TVA în România – Ghid Complet 2025",
  "slug": "tva-in-romania-ghid-complet-2025",
  "excerpt": "Ghid complet TVA România 2025…"
}
```

---

## Security & Configuration

- Do not commit secrets; use environment variables (application password).
- Keep MCP server HTTPS if exposed.
- Redact affiliate codes in shared outputs unless intended for publication.

---

## Final Checklist (Orchestrator)

- Input normalized (url/text/keyword) and cleaned.
- Writer returned `.md` path (one line only).
- Image Curator done (optional images JSON; abs paths verified).
- Monetizer done (0–4 whitelisted URLs).
- Duplicate check complete; operation decided.
- Publisher invoked with correct payload; `create_post`/`update_post` succeeded; featured image set; post URL returned.

