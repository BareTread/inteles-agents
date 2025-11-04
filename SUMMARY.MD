





## Inteles.ro System Overview — What We Do and How We Do It

### Mission
We publish trustworthy Romanian content about dreams, psychology, and personal wellbeing while integrating affiliate monetization responsibly. Every workflow is engineered so human writers and AI sub-agents can operate interchangeably without sacrificing quality, reader trust, or compliance.

---

### Operating Principles
1. **Reader First:** Each article delivers a quick, authoritative answer in the opening paragraphs, then expands with genuine psychological insight, practical scenarios, and cultural context.
2. **Holistic Value:** We pair expert-level writing with hand-picked resources (books, supplements, journals) that genuinely help readers.
3. **Sustainable Monetization:** One contextual affiliate link per article is the rule. We rely on clean merchant homepages and the 2Performant Link2 plugin to apply tracking server-side.
4. **Quality Gatekeeping:** Agents self-score against a strict rubric—if any dimension falls below 7/10 (intent coverage, depth, structure, word count, mobile layout, sourcing, FAQs, imagery, monetization, language), the piece is revised before publishing.
5. **Automation with Oversight:** Scripts gather partner data, verify links, and generate curated lists. Writers and agents still review outputs for accuracy.

6. **MCP Discipline (2025 Update):**
   - Duplicate checks via inteles-wordpress MCP use exact slug + `per_page=1` (publish → draft → pending → private). Never use `status:"any"`.
   - The writer (content-quickfire) never touches MCP; it only returns a single absolute `.md` path.
   - Other agents (image curator, monetizer, publisher) may use their own MCP tools within their scope.
   - Publisher uses `create_post` / `update_post`, and `create_media` with `file://` absolute paths (reuse via `list_media`); supports a publish-as-is mode for complete HTML.

---

### Content Creation Workflow
1. **Task Intake:** A brief describes the topic, article type, and target depth (default 1200–2000 words).
2. **Template Selection:** Agents load the specific Markdown template matching the article genre (dream interpretation, “ce înseamnă…”, general psychology). Templates enforce section order, heading cadence, FAQ structure, and monetization placement.
3. **Merchant Matching:** Before writing, the agent picks the best-aligned merchant homepage from a small, vetted roster:
   - Libris.ro for any dream/psychology article (default).
   - Librex.ro for journals and note-taking themes.
   - Springfarma.com for sleep/anxiety remedies.
   - Manukashop.ro for vitality or morning-energy angles.
  - .... ETC
   Clean homepages are stored as canonical URLs that never expire.
4. **Draft Execution:** The agent writes to the template, ensuring:
   - Short paragraphs (2–3 sentences) for mobile readability.
   - H2/H3 roughly every 300–400 words.
   - Jung/Freud framing where relevant, with citations in Romanian.
   - 6–8 FAQ entries in a Q/A format ready for schema markup.
   - Romanian diacritics and a professional, empathetic tone.
5. **Resource Box Integration:** Exactly one affiliate resource box is inserted near the end of the article, using a pre-approved HTML block with the chosen merchant URL and disclosure text.
ALSO A COUPLE OF LINKS ARE PEPPERED THROUGHOUT THE ARTICLE AIMING TO MAXIMIZE REVENUE
6. **Quality Self-Check:** The writing rubric is applied. If any score is <7, the agent revises. Only compliant drafts advance.
7. **Media Enhancement (optional):** For long-form pieces, the image pipeline selects 2–3 Pexels photos, processes them into WebP, and outputs metadata with alt text and captions. These align with WordPress media requirements.

8. **Publish Workflow (MCP):**
   - Orchestrator computes slug from title and rechecks duplicates (exact slug + `per_page=1`).
   - Publisher receives: `operation=create|update`, optional `post_id`, `article_path` (.md or .html), images JSON, product URLs.
   - If `publish_as_is` or a complete HTML path is supplied, publish the HTML exactly as-is (no enrichment).
   - Upload/reuse media: `create_media` with `file://` ABS paths (or `https`), reuse via `list_media` (≤3), set hero as `featured_media`.
   - Insert products sparsely with `rel="nofollow sponsored noopener"`; never adjacent to an image in the same paragraph.
   - Degrade gracefully: publish without images/products if they fail or are empty; abort on no-op updates.

---

### Monetization & Data Integrity
- **Link Handling:** We utilize the 2Performant Link2 plugin so writers paste only clean merchant URLs. Tracking parameters are injected automatically on click, minimizing human error and keeping article HTML clean.
- **Affiliate Whitelists:** A curated CSV (approved partner domains, categories) governs which programs are eligible. 
  - Monetizer returns 0–4 links from approved domains only; the publisher double-checks and drops non-whitelisted URLs.

---



---

### Governance & Compliance
- **Trust Safeguards:** Maximum of THREE affiliate link keeps articles informational, not salesy. Disclosure text accompanies every resource box.
-
- **Change Discipline:** Small, reviewable edits; avoid mass Markdown rewrites. Automation outputs are clear about their provenance so stale data can be detected quickly.

### WordPress MCP Orchestration — Quick Reference
- Exact-slug discovery with `per_page=1` (never `status:"any"`).
- Writer returns only a path; no MCP/image/product/WordPress work.
- Image curator outputs WebP with abs paths + Romanian ALTs (50–125 chars); verifies files exist.
- Monetizer reads `04-Monetization/affiliates.csv`; outputs only approved merchant product URLs (0–4).
- Publisher uses `create_post` / `update_post`; uploads via `create_media` (`file://` ABS paths preferred), reuses via `list_media` (≤3); supports publish-as-is when given complete HTML.
- Degrade gracefully; abort duplicate no-op updates.

See the full guide: 03-MCP-Operations/WORDPRESS-MCP-PUBLISHING-PLAYBOOK.md

---

### Why This Works
By unifying templates, monetization strategy, and automation under tight guardrails, the system yields:
- Predictable, high-quality articles fit for human or AI authors.
- Monetization that doesn’t erode reader trust.
- Reusable artifacts for future agents without exposing them to outdated or conflicting instructions.
- A clean handoff between content teams and monetization specialists, ensuring every published piece is compliant, useful, and profitable.
