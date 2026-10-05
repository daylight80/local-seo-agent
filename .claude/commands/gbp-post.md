---
description: GBP posts - a month generated to spec (or a single post), queued, and scheduled in Metricool at 2-3/week
argument-hint: [one | offer | event | topic, optional]
---

## STEP 0. Metricool, connected to the profile. Nothing happens before this.

Posts publish through **Metricool**, which schedules them and publishes them to the Google Business Profile at the set time, with no laptop open and no GitHub Action. Claude talks to it through the Metricool connector (the `createScheduledPost`, `getScheduledPosts`, `getBrandSettings` and `getBestTimeToPostByNetwork` tools).

Read CLAUDE.md "## My setup" for `METRICOOL_BLOG_ID`, `METRICOOL_TIMEZONE` and `DEFAULT_CTA_URL`. **If any is missing, run these checks and make the first failing one your FIRST message, and nothing else.** Do not read the spec, do not read context, do not draft a post.

1. **The connector.** If the Metricool tools are not available in this session, ask:
   > "Before I write anything: connect Metricool so posts can go live on their own. In Claude: Settings → Connectors → Metricool → Connect. Free plan works for one brand."
2. **The profile.** Call `getBrandSettings`. If there is more than one brand, ask which one is this business. If the brand has no Google Business Profile connected (no `gmb` network), ask:
   > "Metricool is connected, but your Google Business Profile isn't linked to it yet. In Metricool: your brand → Connections → Google Business Profile → Connect, and pick this location. About a minute."
3. **The link.** `DEFAULT_CTA_URL` is the home page. Take it from `context/business.md` if it is there; ask only if it is not.

Save the brand's `blogId`, its timezone (from `getBrandSettings`) and the home page to "## My setup" so this is never asked twice.

**⛔ Never ask for the GBP account or location ID.** The location is chosen once, inside Metricool, when the profile is connected to the brand. It is not skill config and it is not in the payload.

Then **schedule one real test post a few minutes out** and confirm BOTH signals before generating anything else: `createScheduledPost` returns a `plannerUrl` (Metricool accepted it), and after the scheduled time the post is actually on the profile. The first run needs both; after that, the `plannerUrl` is the per-post receipt, and `getScheduledPosts` shows anything Metricool failed to publish. A connection nobody has proven is a connection that fails on the whole month.

**Only if I reply "skip it":** generate the batch paste-ready for manual posting and say plainly that automatic posting is off.

### What the Metricool route can and cannot post

Metricool's Google Business Profile post is a **"publication"**: an update with text (up to 1,500 characters) and one photo. It has **no title field, no button, and no Offer or Event type**. So on this route:

- **Every post goes out as an update.** The `title` becomes the post's opening line, followed by a blank line and the summary.
- **Offers are written as updates that carry the offer:** what it is, the code, the dates and how to redeem ("mention this post when you call"), all in the text. They do not get Google's yellow "Offer" badge.
- **Links in the text are not clickable on Google.** Write the action as a sentence ("Call us", "Book on our website") and leave the URL out of the body.
- **If I want a real Offer or Event post with the badge and button**, say so in one line in the action list: post that one by hand in the Google Business Profile editor, or switch on the optional Make.com route described in `references/gbp-posts.md`. Never stop the batch over it.

This is the one question asked BEFORE any work happens, and it is mandatory. Later steps do stop for approval - the unsent queue, the angle bank, a missing Pexels key, and the batch itself - but nothing is generated until Metricool is confirmed.

## STEP 0.2. ⛔ THE EXISTING QUEUE EATS FIRST. THIS IS YOUR SECOND MESSAGE, BEFORE ANY RESEARCH.

**Read `gbp-posts-queue.md` before anything else.** Not after the voice file, not after the angle bank - immediately after Metricool is confirmed. Generating a fresh month while approved posts sit unsent is the single most wasteful thing this command can do, and it happens because this step is easy to scroll past.

**If the queue holds posts that are approved but never went out** - Pending or Scheduled, no `sent_at`, usually written back when automatic posting was off - stop and ask:

> **"You've got 8 approved posts in the queue that never went out - automatic posting was off when they were written, and now Metricool is connected. Want me to schedule those first? They'd go out Mon/Wed/Fri from [date]. Or I can generate fresh ones and leave these."**

**Check staleness before offering, and say what you find in one line.** A post referencing "next month's deadline" or a feature that has since shipped is not schedulable - flag those individually, offer to refresh just those, and schedule the rest. Never quietly ship a post whose date reference has expired.

**⛔ KEEP THE TITLE TO 58 CHARACTERS AND THE WHOLE POST UNDER 1,500.** On the Metricool route the title is the opening line, so title + blank line + summary must fit Google's 1,500-character cap. Write the title to 58 from the start (it reads like a subject line, and it keeps the Make.com route usable if it is ever switched on) rather than trimming a long headline - a trimmed headline reads like a trimmed headline.

**⛔ VALIDATE BEFORE YOU CLAIM ANYTHING IS QUEUED. `python3 code/check_gbp_payload.py gbp-queue` must exit 0.** It checks the length caps, the photo (public https, JPG or PNG, under 2 MB), the date (not in the past) and the spacing (max 3 a week, at least 2 days apart). Never report posts as scheduled until the validator passes, the `.yml` files exist on disk, and each one carries the `plannerUrl` Metricool returned.

**On a yes:** write each to `gbp-queue/` as a dated YAML with `send_after` spaced per the caps, schedule each in Metricool (see "Ship" below), and mark them Scheduled in `gbp-posts-queue.md`. Then stop - do not also generate a new month unless I ask.

### ⛔ Then make the FIRST one go out soon, as the live proof

A queue nobody has watched work is a queue nobody trusts. So the first post does not wait for its slot:

1. **Schedule it in Metricool for a few minutes from now** and read the response. A `plannerUrl` means Metricool accepted it; an error means it did not, and every post after it would fail the same way. On an error, stop and fix it.
2. **Send me to look at the profile** once the time has passed, and confirm the post is actually there. Metricool accepting it proves nothing about Google publishing it. **Wait for me to confirm** - do not proceed on the `plannerUrl` alone.
3. **Move it to `gbp-queue/sent/` with today's `sent_at`** and mark it Published in `gbp-posts-queue.md`, so it can never be scheduled twice.
4. **Then say what happens next in one line:** the rest are already sitting in Metricool's planner and publish on their own on the dates listed.

**⛔ THEN LEAVE POST 2 ALONE.** Post 1 proves the connection carries a real post. Post 2 is the proof that Metricool publishes unattended, which is the part that has to work every week for a year without anyone watching. Say exactly what to look for:

> **"Post 2 goes out Wednesday and I'm not touching it. Check your profile Wednesday afternoon - and Metricool's planner will show it as published."**

**Never block on it.** Say it, schedule it, move on. If it has not appeared, `getScheduledPosts` shows whether Metricool tried and what Google said.

**This runs once, on the first real batch.** After the connection has carried a live post, later runs schedule everything and push nothing early.

## STEP 0.5. Read the voice file. Then write in it, not near it.

**Read `context/voice.md` Part 1 in full before drafting a single post.** Not skim, not "informed by" - the register section and the devices are the spec, and a batch written first and voiced afterwards always reads flat.

Non-negotiable on every post:

- **Contractions throughout.** It's, don't, can't, you're, doesn't. A post with none reads like a legal notice.
- **One line worth screenshotting.** If it wouldn't make somebody read it out to the person next to them, it isn't the line yet - rewrite it rather than settling.
- **Never joke about being bad at what you sell.** Effort, physical toll, personality, taste are all fair. Competence is not. An agency admitting it forgot its own tracking, on its own profile, next to its own price, costs more than the joke earns.
- **Never write a self-audit.** No section at the bottom listing the good lines, no tagging lines with device names, no telling me it's funny. Show me the posts.
- **No tables.** Bullets and plain lines, per `references/output-format.md`.

Then run the gate before showing me anything:

```
python3 code/check_voice.py gbp-posts-queue.md
```

A failing file gets fixed and re-run. It does not get shown to me with an explanation.

**Repo wiring:**
- Voice: write from `context/voice.md` + `context/business.md`. Proof: every number, review stat, and offer traceable to `context/proof/proof-inventory.md` - never invent a discount or review count.
- Dedupe: read `gbp-posts-queue.md` "Published" before generating - never repeat a recent post. Create the queue file if missing (Pending / Scheduled / Published / Archive).

---

## STEP 1. Find the angles BEFORE you write. Never write from memory.

**⛔ A batch written straight from `context/` will be the same three posts eight times.** That is not a style problem, it is an input problem: the same source produces the same posts. Everything the business already knows about itself is stale by definition - it has not changed since the last batch. Go and get material that did not exist last month.

Run these lanes **in parallel sub-agents**, then write. Do not write a single line until the angle bank exists.

**Lane 1 · News in the space, last 90 days.** WebSearch for what actually changed: platform and algorithm changes, AI features landing in the tools this buyer uses, pricing moves, new regulation, a competitor's public stumble. **Anything older than 90 days is not news, it is background** - check the publication date before using it. One post that reacts to something from this month beats four evergreen ones, because it is the only kind of post nobody else in the map pack can have written in advance.

Two date traps that have shipped in real batches:
- **Every dated claim carries its YEAR in your head before it ships.** "As of 31 July" with no year is how a 2024 event gets dressed as this week's news - a reader who knows the real date reads the whole profile as fake. Old events are allowed as BACKGROUND, phrased as background ("Google killed the chat button back in 2024"), never as the headline.
- **A feature is only "live" where THIS business's customers are.** Check the rollout geography before writing "rolling out now" - a US-only launch is not news in Vancouver, it is a forecast. State the local timing honestly ("live in the US, Canada expected this year"), which usually makes a BETTER hook anyway: real urgency with a real deadline instead of a claim a reader can falsify with one search.

**Lane 2 · The calendar, next 6 weeks.** What is about to happen to this buyer: the season for their trade, the quarter they are planning, the weather, the local event, the deadline. Seasonal beats evergreen because it has a reason to exist NOW - it earns urgency without faking it. Search the actual dates rather than assuming, and check the arithmetic. *(A post dated 12 September opening "Q4 is eight weeks out" shipped in a real batch. Q4 was under three weeks away.)*

**Lane 2b · The GBP candidates queue.** Read the `## GBP candidates` section of `gbp-posts-queue.md` - blog posts the batch scheduler flagged since the last run. **These are suggestions, not entries.** Each one competes with every other angle on merit; a candidate that does not beat the alternatives does not ship, and saying so is the filter working. A blog that DOES win becomes a post pointing at it, because sending profile traffic to a genuinely useful page is better than another offer. Clear the ones you use from the section and leave the rest for next month.

**Lane 3 · The proof file, mined for what has NOT been used.** Read `context/proof/proof-inventory.md` in full and cross it against the "claims used" log in `gbp-posts-queue.md`. **List the proof that has never carried a post.** The best angle is nearly always a real story sitting unused while the same two client results get recycled. If two results are structurally identical (client, we automated it, saved $X), only one of them ships this month.

**Lane 4 · Real customer language, and Reddit is the best of it.** Recent call transcripts, reviews, community questions, sales objections - what did somebody actually say out loud? A post built on a sentence a real customer said outperforms one built on what the business wishes they said.

**Scrape Reddit with the Apify actor `trudax/reddit-scraper-lite`** (posts + comments, search term, no login) - and actually RUN the actor via the Apify API, one call per search term:

```bash
source .env && curl -s -X POST \
  "https://api.apify.com/v2/acts/trudax~reddit-scraper-lite/run-sync-get-dataset-items?token=$APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"searches": ["[service] cost"], "maxItems": 40, "maxPostCount": 20, "maxComments": 10}'
```

⛔ **Google search snippets that quote Reddit are NOT a substitute.** Snippets truncate the thread, hide the comments (where the real objections live), and strip the upvote counts. If `APIFY_TOKEN` is missing from `.env`, stop and ask for it (apify.com, free tier, 30 seconds) - never silently fall back to searching. This lane runs on the scraper or it waits.

Reviews are written in public by people being polite; Reddit is buyers talking to each other with nothing at stake, so the objection appears in full. Search `"[service] cost"`, `"how much did you pay for [service]"`, `"is [service] worth it"`, `"[service] scam"`, and `r/[their city]` plus the service. **A post that answers a question 40 people upvoted is a post with demand already proven.**

**Keep every phrase verbatim, with the date.** The exact wording is the whole value - paraphrasing it into marketing language destroys the reason it was worth collecting. And Reddit is anonymous: it is a **language and fear source, never proof**. Never quote a Reddit number as evidence, and check the date before treating a price thread as current.

**Lane 5 · The local competitors' own GBP posts.** Search the map pack for this service in this city and read what the top 3 are posting. **This lane exists to tell you what NOT to write.** If everyone is posting "Happy Friday from our team", the entire field is open.

**Then build the angle bank.** 15 to 20 candidate angles, each one line, each tagged with its lane and the specific fact behind it. Show me the bank before the posts if I asked for the full month - it is a 30-second read and it is the cheapest place to redirect the batch.

**Pick 8 that are maximally DIFFERENT, not the 8 best.** Two brilliant angles from the same lane, resting on the same claim, are one post. Spread across lanes deliberately.

---

## STEP 2. The diversity gate. Run it before you show me anything.

Eight posts that pass individually can still be one post written eight times. Check the batch as a batch:

- **No claim carries two posts.** One appearance per claim per month, logged in "the claims used this month".
- **No two posts share a shape.** Assign each a different one and name it in your head: story · contrarian claim · a number that surprises · the question a customer actually asked · a confession about our own business · a teardown of standard industry practice · a short numbered list · a comparison · a reaction to this month's news. If two posts open the same way, one gets rewritten.
- **At least 3 lanes represented**, and at least one post that could only have been written this month.
- **Read the eight headlines in a row.** If they sound like one voice at one volume, that is the failure. Fix it before I see it.

**⛔ THE LOCAL REFERENCE IS A REAL DETAIL, NOT A KEYWORD.** Every post needs one, and it has to be something true about the place: a street, a neighbourhood, the weather, a local event, a landmark, the actual office. **Never a phrase bolted onto a generic sentence.** "The audit costs nothing, Vancouver or anywhere else" and "here in Vancouver and everywhere else" are keyword stuffing wearing a sentence, and a reader clocks it instantly. One local reference per post, maximum, and it earns its place or it comes out.

**⛔ Never argue against local on a local surface.** "Same build whether you are in Vancouver or three time zones away" is a fine line on a website and a self-inflicted wound on a Google Business Profile, whose entire job is proximity. If the proof is from another city, use it - just do not point out that geography is irrelevant on the one surface where it is the ranking factor.

---

**⛔ VOICE: read `context/voice.md` BEFORE the first sentence.** The dial table in Part 1 sets GBP posts at **one good line each** - not zero. A batch with no joke in it has failed the spec, not played it safe. Part 1 is the house style and it outranks anything in Part 2 that reads as more cautious; where they conflict, Part 1 wins. Specificity is the device that is never switched off: the concrete weird detail IS the joke. The straight zones still apply - prices, timelines, proof and the last line before the ask are written flat, always.

**Which of the four joke shapes fit a GBP post:** the **absurd prop** and the **universal tangent** - both land in one sentence, which is all a post has ("Your water heater doesn't break on a Tuesday afternoon. It waits."). The escalating confession needs three beats of runway a GBP post doesn't have - rarely. The stacked rule-of-three eats half the word count - never. The one line per post aims at the swipe-file bar (voice.md's top entries); everything else in the post is straight, because the reader is minutes from calling and the joke earns the read while the straight sentence earns the call.

**⛔ PEXELS KEY - get it before you need a photo, not after.** Check `.env` for `PEXELS_API_KEY`. Missing or empty? Ask inline, right then:

> "Paste your Pexels API key so I can pull real photos: pexels.com/api → 'Get Started' → copy the key. Free, no card, about 30 seconds."

Save it to `.env` so nothing asks twice. Then pull with `python3 code/fetch_stock_photos.py "[query]" --count 8` - a pool to pick from, not a quota to use: look at all of them, ship the best one per post, never the same photo on two posts, delete the rejects. Photos are DOWNLOADED, never hotlinked. Without a key it falls back to Openverse, which needs no signup but returns amateur photo-library results (a search for "hvac service van" came back with a refurbished tram). Usable in a pinch, never shipped unlooked-at. **Look at every photo before it goes on a page.**

**The batch (per the spec):**
- **Every post traces to an angle from Step 1.** A post you cannot point back to a lane and a specific fact is a post written from memory - cut it and take the next angle off the bank.
- Cadence 2-3/week (the cap enforced by `check_gbp_payload.py`) → generate a month (4-8 posts) in the spec's monthly mix: ~50% offer posts, ~30% updates (seasonal tips, customer stories from my real jobs), an event only if one exists, 1 product/service spotlight. On the Metricool route all of them go out as updates (see Step 0); the mix is about what each post says
- Every post: the 3 justification-bait layers (specific service variant + ONE real local detail + a long-tail phrase a customer would actually say), first-100-chars hook, image_query for the stock pull (real photos from `context/proof/images/` beat stock - use them first), CTA always. **Baited means woven in so it reads as writing. If you can see the layer, it failed.**
- **⛔ Every post ships with an image. No exceptions.** The ladder: a real photo from `context/proof/images/` → stock via `code/fetch_stock_photos.py` → if the first query returns nothing usable, rewrite the query and pull again until one does. A post with no image is a failed post, not a shipped one - it never goes to Metricool and never lands in the queue as ready.
- **⛔ Team faces and testimonial shots are not decoration.** A team photo goes on a post ABOUT the team (a hire, a milestone, behind-the-scenes); a testimonial image goes on the post telling THAT customer's story. Neither gets slapped on a seasonal tip because it was the nearest "real" photo - a face on an unrelated post reads as filler, and it spends the credibility of the person in it. When the proof folder has nothing that matches the post's actual subject, that's what the stock rung of the ladder is for.
- **⛔ The link is the home page. Never ask which page a post points at.** `cta_url` (and `redeem_online_url`) default to `DEFAULT_CTA_URL` on every post, silently. On the Metricool route the link is not sent (there is no button), but keep the field: it is what the owner sees in the queue and what the Make.com route would use. A deeper page happens only if I named one, or the post covers one service that already has a live page in `website-index.md` - and that page gets verified before it is used. Eight posts is eight silent home-page links, not eight questions. See "The link on every post" in the spec.
- Offer posts: ALL the offer facts generated per the spec's rules (confirmed coupon code, ISO dates, 7-day window, terms default), and on the Metricool route the code, dates and how to redeem are written into the text itself
- Lengths per the spec's real-world table - short beats the guides' claims
- **⛔ Photos go to Metricool as public https URLs, never local paths.** Metricool and Google download the image themselves, so `website/public/images/...` fails silently. Download the photo, put it in `website/public/images/`, publish it, send the live URL. Site not published? That is a `/publish` job - say so and offer to run it, never file it as "photos cannot attach automatically".

**Ship:** show me the month as a legible list (title + type + week + the hook line) and get ONE yes on the batch. Then:

- **⛔ Scheduled - THE DEFAULT. One approval covers the whole month.** Write each approved post as a dated YAML file in `gbp-queue/` (payload fields per the spec, plus `send_after: YYYY-MM-DD` spaced Mon/Wed/Fri across the month), run the validator, then schedule each one with `createScheduledPost`:
  - `blogId`: `METRICOOL_BLOG_ID` from "## My setup"
  - `date` and `info.publicationDate`: the `send_after` day at 09:00 in `METRICOOL_TIMEZONE` (or the slot `getBestTimeToPostByNetwork` gives for `gmb`, if I asked for best times)
  - `info.providers`: `[{"network": "gmb"}]`, `info.gmbData`: `{"type": "publication"}`, `info.autoPublish`: true, `info.draft`: false
  - `info.text`: the title, a blank line, then the summary, plain text
  - `info.media`: `[media_items]` (the public https photo URL)

  Write the returned `plannerUrl` into the post's YAML as `metricool_planner_url` and move the file to `gbp-queue/scheduled/`. **A file in `scheduled/` or `sent/` is never scheduled again**, so a re-run cannot double-post. An error from Metricool leaves the file in `gbp-queue/`, gets ONE retry, then lands in the failure list with the error text as Metricool returned it.

  **Never ask which lane I want.** Approving the batch means schedule it. A month of posts is written to be read over a month, and firing eight at once buries seven of them - the profile shows the newest and the rest scroll away the same afternoon. Report the dates each post will go out on, with its Metricool link, then stop.

- **Now - only if I explicitly say "post them all now" or "publish immediately".** Schedule each a few minutes apart starting a few minutes from now, read each response, and report scheduled/failed per post. **Say what it costs before doing it:** eight posts in one afternoon means seven are buried by the eighth.

- **Optional: the Make.com route.** Only if I ask for real Offer or Event posts with Google's badge and button. The original webhook setup is kept in `references/gbp-posts.md` ("Optional: the Make.com route") and `code/publish_due_gbp_posts.py`. Mark those posts `route: make` in their YAML and the validator checks them against Make's schema instead.

Page-announcement posts arrive through this same queue and obey the same caps, so a month of content and a batch of new-page announcements cannot collide into a burst.
