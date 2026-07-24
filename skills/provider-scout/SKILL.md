---
name: provider-scout
description: Find and vet local service providers (plumbers, electricians, HVAC, roofers, GCs, landscapers, handymen, and other home-improvement trades). Use when the user wants recommendations for a contractor or service company "near me", wants a company checked out before hiring, or asks whether a provider is reputable, locally owned, or a private-equity rollup. Triggers include "find me a plumber", "who should I call for X", "is <company> legit", "vet this contractor", "recommend an electrician near me".
---

# Provider Scout

Find local service providers worth hiring — and screen out the ones that aren't. Three disqualifiers drive the vetting: **bad reviews** (read for patterns, not star averages), **institutional ownership** (PE rollups and franchise conversions), and **manipulative tactics** (pressure sales, review gaming, lead-gen middlemen).

This is a research skill: use WebSearch and WebFetch throughout. Never recommend a company you haven't vetted through the full pipeline below.

## 0. Location

Check `state.json` next to this SKILL.md for a saved default location (gitignored; see `state.example.json`). If missing, ask the user for their city/zip and offer to save it so future searches skip this step.

## 1. Intake

Establish before searching:
- **Trade/job**: what actually needs doing (a "water heater replacement" search beats a generic "plumber" search — specialists and honest generalists surface differently)
- **Urgency**: emergency jobs still deserve a fast vet (steps 3–5 compressed, not skipped) — pressure works on people with burst pipes, which is exactly when rollups profit
- **Scale**: repair vs. replacement vs. renovation. For large jobs (>$5k), do the full pipeline for every finalist and suggest 3 quotes.

## 2. Discovery — cast wide, from independent sources

Run several searches in parallel; each channel has different blind spots:

- **Maps/directories**: `<trade> <city>` on Google Maps results via search; note companies appearing organically (not "Sponsored")
- **Word of mouth**: `site:reddit.com <city OR metro> <trade> recommendation` — local subreddits are the least gameable source; also try Nextdoor mentions via search
- **License rolls**: state contractor license board (search `<state> contractor license lookup`) — sometimes the best small operators barely have a web presence
- **Trade-specific**: for niche work, supplier/manufacturer installer lists (e.g. certified installers) over generic directories

Build a candidate list of 6–10 names before filtering. Record where each came from — a company found only via ads and lead-gen sites is already a yellow flag.

**Skip entirely**: Angi/HomeAdvisor/Networx/Thumbtack "matches" — these are lead-resale middlemen, not recommendations. A company *listed* there is fine; a company you can only reach *through* them is not.

## 3. Review screen — patterns, not averages

For each candidate, read actual review text (Google, Yelp, BBB), especially 1–3 star reviews and the newest 10. Star averages are the most gamed number on the internet.

**Disqualify on recurring patterns** (one-off angry reviews are noise; three reviews describing the same behavior are data):
- Quoted price ballooning after work started
- Diagnosed a catastrophe during a "free inspection" that a second opinion contradicted
- No-shows, unreachable after payment, warranty not honored
- Technician spent the visit selling (memberships, replacements, financing) instead of fixing

**Disqualify on review manipulation**:
- Bursts of 5-star reviews in a short window, especially generic ones ("Great service! Highly recommend!") from accounts with 1 review
- Reviews that read like they were written on-site while the tech waits ("Bob was here today and did great")
- Owner responses to negative reviews that are hostile or accuse the customer of lying
- A large gap between Google rating and Yelp/BBB rating (Yelp's filter is harder to game)

**Good signs**: negative reviews that describe minor issues resolved reasonably; reviewers naming the same tech/owner across years; detailed reviews mentioning specific jobs and prices.

## 4. Ownership screen — find the rollups

PE-owned trades companies keep the local brand name after acquisition. The vet:

1. Search `"<company>" acquired`, `"<company>" acquisition`, `"<company>" private equity`, `"<company>" parent company`
2. Fetch the company's About page — vague language ("proudly serving since 1987") with no named owner, or a recent site redesign matching a corporate template, warrants digging
3. Search the company on LinkedIn — employees listing a different parent company is a giveaway
4. Check against known rollup platforms and franchise umbrellas (non-exhaustive; the roster grows, so also trust the acquisition search): **Apex Service Partners, Wrench Group, TurnPoint Services, Redwood Services, Leap Partners, Sila Services, Heartland Home Services, Southern Home Services, Authority Brands** (Benjamin Franklin Plumbing, Mister Sparky, One Hour Heating), **Neighborly** (Mr. Rooter, Mr. Electric, Aire Serv), **Ace Hardware Home Services**
5. Behavioral tells even without a paper trail: recent reviews complaining prices jumped or "it's not the same company anymore"; heavy push of membership/"club" plans; techs on commission (reviews mentioning quotas or upselling); slick financing offers (GreenSky/Service Finance) front and center

Franchises of national brands count as institutional for this filter. **Prefer**: owner's name on the website or trucks, owner responding to reviews personally, licensed under an individual's name, in business >10 years under the same ownership.

## 5. Tactics screen

Disqualify companies showing sales-machine behavior:
- "Today only" / "sign now" discount structures (reported in reviews or on their site)
- Free inspection → urgent catastrophic finding as a pattern (step 3 overlap)
- Door-to-door or post-storm solicitation (search `"<company>" door to door OR soliciting` — big roofing red flag)
- Review gating (only happy customers get the review link — visible as a wall of 5-stars with near-zero middle ratings)
- Won't give ballpark pricing on the phone but insists on an in-home "consultation" with all decision-makers present (classic one-call-close setup)
- Fine print requiring binding arbitration or waiving lien rights on estimates

## 6. Verify the finalists

For the 2–4 survivors:
- License status active on the state board; matches the business name
- Insurance/bond claims on their site (advise the user to request a COI for big jobs)
- BBB complaint history — pattern of complaints matters more than the letter grade (accreditation is paid)
- Years in business under current ownership (state business registry via search if unclear)

## 7. Report

Deliver a shortlist, best first. For each finalist: name, contact, why they made the cut (evidence, not vibes — cite specific reviews/sources), ownership status, and anything to watch for. Then a brief disqualified list with the reason each was cut — the user will likely encounter these names in ads and should know why to skip them.

Close with job-specific advice where warranted (e.g. "for a roof, get 3 quotes and don't sign anything the same day").
