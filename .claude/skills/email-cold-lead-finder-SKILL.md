---
name: email-cold-lead-finder
description: >
  Finds independent music producers who are plausible BeatPilot prospects (active on
  YouTube/Instagram/SoundCloud in target niches) and collects their PUBLIC business/collab
  contact email into a clean prospect list in Supabase `cold_prospects` — ready for cold
  outreach through a separate tool (never through Klaviyo or BeatPilot's main sending
  domain). Trigger on "find cold email leads," "build a prospect list," "run the cold lead
  finder," or requests to find people who might want BeatPilot. Never scrapes private data,
  never collects personal (non-business) emails, never sends anything itself.
---

# Cold Lead Finder Skill

## 1. Purpose

Build a list of real, plausible BeatPilot prospects — independent producers active in
niches BeatPilot already tracks — using only contact information they've made public
themselves for business/collab purposes. This mirrors the existing `client-finder` skill's
approach (public contact emails for outreach), applied to individual producers instead of
brands.

**This skill never sends anything.** Output is a Supabase table. Actual outreach happens
through a separate cold-email tool (see section 6), never through Klaviyo.

## 2. Why this can't use Klaviyo or the main sending domain

Klaviyo (like most ESPs) is built for permission-based marketing to people who opted in.
Sending unsolicited cold email through it risks the account being flagged or suspended —
which would take down your real subscriber emails along with it. Cold outreach needs its
own domain and tool, warmed up independently. This skill does not attempt to wire up
sending; it produces the list only.

## 3. Sourcing — target niches from real data, not guesswork

Pull current high-opportunity niches from the same tables the Ideation skill uses:

```sql
select niche_phrase, parent_genre, opportunity_score, growth_potential
from {source_table_niche_opportunities}
where opportunity_score is not null
order by opportunity_score desc
limit 15;
```

For each niche, search for producers/channels actively publishing in that space (YouTube
"[niche] type beat" channels with recent upload activity, Instagram/SoundCloud accounts
producing in that genre). Prioritize accounts that look active (recent posts) and
plausibly independent (not major-label-affiliated, not already a known BeatPilot customer
— cross-check email against Klaviyo before adding, see section 5).

## 4. What counts as fair game to collect — hard boundary

**Collect only:**
- An email explicitly published for business/collab/booking inquiries (YouTube "About"
  page business email, Instagram bio, Linktree-style link pages, video descriptions)

**Never collect:**
- Personal emails not published for business purposes
- Emails obtained by any method other than reading what the person already made public
  for outreach (no guessing common patterns like `firstname@gmail.com`, no purchased
  data, no scraping login-gated content)
- Anyone who has an explicit "no unsolicited business inquiries" or similar note

If a channel/account doesn't publish a business contact email, skip it — do not fall back
to a personal-looking email found elsewhere.

## 5. Deduplication and compliance checks before adding

Before inserting a row:
1. Check the email isn't already an existing BeatPilot user or subscriber — cross-
   reference against Klaviyo lists (including `BeatPilotTrial`,
   `email_os_config.klaviyo_list_beatpilot_trial_id`) and any other known list. Don't
   cold-email someone already in the funnel.
2. Check `cold_prospects` for an existing row with the same `profile_url` (unique
   constraint enforces this, but check first to avoid a failed insert).
3. Make a best-effort, non-authoritative `region_guess` from any public location/language
   signal on the profile — this is for later filtering, not a legal determination.

## 6. Output

Insert into `cold_prospects`: `channel_or_handle`, `platform`, `profile_url`,
`contact_email`, `niche`, `source_context` (exactly where/how the email was found),
`region_guess`, `status = 'new'`.

Present a summary batch to the user (count found, by niche/platform) rather than
individual approval per lead — this is prospecting, not sending, so the stakes are lower
than the campaign pipeline. The user reviews the list before it ever goes anywhere.

## 7. Compliance notes — real, not optional

- Any actual cold email sent from this list must include a real physical mailing address
  and a working, honored opt-out mechanism (CAN-SPAM requirements apply to unsolicited
  commercial email in the US regardless of how the address was obtained).
- Recipients in the EU/UK generally require consent for direct marketing under GDPR —
  scraping and cold-emailing an EU-based individual without consent is a real legal
  exposure, not a formality. Recommend restricting active outreach to US-based prospects
  (using `region_guess` as a rough filter) until/unless a compliant approach for other
  regions is worked out.
- These rules apply to whatever tool eventually sends the email, not to this skill — but
  this skill should still tag `region_guess` so that filtering is possible later.

## 8. Hard rules

- Never send an email. This skill has no sending capability and should not attempt one.
- Never collect an email not explicitly published for business/collab contact.
- Never scrape login-gated, private, or ToS-restricted content to obtain contact info.
- Never add someone already in Klaviyo (existing user/subscriber) to this cold list.
- Never fabricate or guess an email address.
