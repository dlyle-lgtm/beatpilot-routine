# Research Mode

Use this mode to find, score, and log new blog topic candidates. This mode NEVER approves a topic — it only ever writes `Idea` or `Research` to the Status column.

## 1. Source monitoring

Check these sources for new topic material, in this order:

1. **`02 Competitor Research / RSS Research`** — any RSS/feed exports already saved here. If a feed hasn't been checked recently, pull current headlines from producer-facing publications and YouTube-creator-economy sources.
2. **`02 Competitor Research / SERP Research`** — existing search-results snapshots for target keywords.
3. **`02 Competitor Research / Social Research`** — trending discussion threads (FutureProducers forum, producer subreddits via aggregator sites, since direct Reddit fetches are frequently blocked — gummysearch.com/r/[subreddit]/ is a working pattern for subreddit-level aggregate data).
4. **Live web search** for anything not already captured in the folders above — competitor blog posts, YouTube creator-economy news, FL Studio/Ableton community discussion.

Save any new raw research snapshot (a SERP export, an RSS pull, a subreddit summary) into the matching subfolder above so future runs don't repeat the same research from scratch.

## 2. Competitive research

For each candidate topic, identify:
- Which competitors (or top-ranking pages) currently own this topic.
- What their article actually covers — structure, angle, depth.
- Where they're thin: missing data, no BeatPilot-style tool tie-in, generic advice with no product-backed proof.

Never copy structure or phrasing directly — the point is to find the gap BeatPilot can fill, grounded in a real BeatPilot feature (check `BeatPilot Product Information` for what's actually live before proposing an angle).

## 3. Content-gap analysis

A topic is a genuine gap if:
- No existing top-ranking content offers a data-backed, tool-verified answer (most competitor content is opinion/anecdote).
- BeatPilot has a real, live feature that maps directly to the topic (check Product Information — don't propose an angle for a feature that's still "in development," like Mobile Alerts).
- The topic doesn't already exist, or exist in a stale/underperforming form, elsewhere in the pipeline or in `01 Topic Pipeline / Published Topic Log`.

## 4. Keyword clustering

For every candidate, identify:
- **Primary Keyword** — the single term the article should rank for.
- **Supporting Keywords** — secondary terms naturally covered by the same article.
- **Primary Cluster / Supporting Cluster** — which broader content pillar this belongs to (e.g. "Thumbnails & CTR," "Upload Timing," "Trend Discovery," "Producer Business," "AI Workflow" — match existing cluster values already in the Sheet rather than inventing new cluster names ad hoc).

Group candidate topics by cluster before scoring — this makes cannibalization (step 6) much easier to catch.

## 5. Topic scoring

Score every candidate against the scoring columns already defined in the `BeatPilot Blog Content Pipeline` sheet:
- Audience Relevance Score
- Search Opportunity Score
- Buyer Intent Score
- Product Alignment Score
- Conversion Potential Score
- Competitive Weakness Score
- Original Data Potential Score (can this post use real BeatPilot data no one else has?)
- Visual Potential Score
- Social Potential Score
- LLM Potential (would this content likely get cited/surfaced by AI answer engines?)
- Opportunity Score (composite — use the same rough logic as the other scored columns rather than inventing a new formula)

Use a consistent 1–10 (or 0–100, matching whatever scale already appears in existing rows) scale across all candidates in a single run so scores are comparable.

## 6. Duplicate checking

Before adding any row:
1. Search the `BeatPilot Blog Content Pipeline` sheet for the same or a near-duplicate Primary Keyword, Proposed Title, or Original Topic.
2. Check `01 Topic Pipeline / Published Topic Log` and `01 Topic Pipeline / Rejected Topics` — don't resurface something already published or already rejected without a genuinely new angle.
3. Check for keyword cannibalization — two different rows targeting the same Primary Keyword. If found, either merge the angles into one row or clearly differentiate the Primary Keyword/Search Intent between them, and flag the conflict to the user rather than silently picking one.

## 7. Writing ideas to Google Sheets

For every qualified candidate, append a new row to `BeatPilot Blog Content Pipeline` with at minimum:
- Topic ID (next sequential ID in whatever format the sheet already uses)
- Proposed Title
- Original Topic
- Source Type (RSS / SERP / Social / Manual / Competitor — match what actually generated the idea)
- Source URL / Competitor Source, if applicable
- Primary Keyword, Supporting Keywords
- Primary Cluster, Supporting Cluster
- Search Intent, Buyer Stage
- The scoring columns from step 5
- **Status: `Idea` or `Research` — never `Approved`.**
- Date Added

Leave SEO/production columns (Recommended Word Count, Visual Opportunities, etc.) blank if not yet researched deeply — those get filled in during Article Mode's research brief step, or by a human reviewing the queue.

Report back to the user: how many candidates were evaluated, how many were added (with a one-line reason each), and how many were rejected as duplicates/cannibalizing/not a real gap.
