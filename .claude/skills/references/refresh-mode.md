# Content Refresh Mode

Use this mode to audit already-published articles and recommend (not silently make) updates, merges, or redirects.

## 1. Ranking declines

If an analytics/search-console data source is connected in this session, pull real ranking/traffic trend data per published URL. If no such source is connected, say so plainly rather than guessing at performance — do not invent a ranking trend. (As of the last confirmed setup, Plausible Analytics is installed on beatpilot.io for traffic; search-ranking data specifically may require a separate Search Console connection that may not be available — check what's actually connected before claiming a decline or improvement.)

## 2. Outdated pricing

Check every published article that mentions a specific price, plan name, generation limit, or thumbnail limit against the current `BeatPilot Pricing and Trial Information` doc. Flag any mismatch — pricing has changed more than once historically, so don't assume an older post is still accurate just because it was correct when published.

## 3. Outdated screenshots

Check every published article's images against `BeatPilot Product Information`'s current screenshot filenames and the actual current product UI. The product ships new sections fairly often (e.g. the July 2026 Data Freshness badges were added across several pages) — a screenshot that was accurate at publish time can go stale within weeks.

## 4. Outdated feature descriptions

Beyond screenshots, check whether the article's description of *how a feature works* still matches `BeatPilot Product Information`. Features get renamed, gated differently by plan, or gain new capabilities (e.g. Mobile Alerts moving from "not built" to "in development" to eventually live) — flag any published claim that no longer matches current documentation.

## 5. Broken links

Check that every internal link in a published article still resolves to a live page (not a 404, not a redirect chain), and that every external source link still exists. Flag broken links rather than silently removing them — a human may want to replace with an updated source rather than just deleting the citation.

## 6. Content merging and redirect recommendations

If two published articles now cover very similar ground (common after enough Research Mode runs across a growing pipeline), recommend either:
- Merging into one stronger article with a 301 redirect from the weaker URL, or
- Differentiating them more clearly (retitling, refocusing one on a distinct sub-angle) if both are pulling meaningful independent traffic.

Redirect and technical SEO changes require developer/Replit involvement — this mode produces a recommendation with reasoning, not a live change. Save refresh audit findings to `09 Analytics and Reports / Content Refresh Reports`, and list clear, specific next actions (not just "this seems stale") for each flagged article.

## Reporting format

For every audited article, report: URL, publish date, what's stale (pricing / screenshot / feature description / broken link / cannibalization), and the specific recommended fix. Don't bundle vague "needs a refresh" findings — each flag should be actionable on its own.
