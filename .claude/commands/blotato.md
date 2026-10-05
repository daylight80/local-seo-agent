---
description: Turn a published blog post into native social posts and schedule them through Blotato - one source, one post per platform, each rewritten for its own feed
argument-hint: [/blog/slug or a URL, optional - defaults to the newest published post]
---

## STEP 0. The Blotato key. Nothing happens before this.

Read CLAUDE.md "## My setup" and `.env` for `BLOTATO_API_KEY`. **If it is missing, empty, or still a placeholder, your FIRST message is this question and nothing else.** Do not read the blog post, do not draft a caption, do not pick platforms.

> "Before I write anything: paste your Blotato API key so these can actually schedule. In Blotato go to **Settings → API → Generate API Key** (my.blotato.com/settings). One warning first: generating a key ends a free trial and starts a paid Starter subscription, because the API is paid-only. About 30 seconds."

**⛔ Copy the key whole, including any trailing `=`.** Blotato keys are base64 and often end in one or two `=`. Stripping, trimming or URL-encoding them is the single most common auth failure, and it looks identical to a wrong key.

Save it to `.env` as `BLOTATO_API_KEY` (never CLAUDE.md, never a chat) and record in "## My setup" that Blotato is connected, so this is never asked twice.

**The whole API, in the four facts that matter:**

- Base URL is `https://backend.blotato.com/v2`. **`api.blotato.com` does not exist** - it is the name an AI guesses, and it fails.
- Auth is a custom header: `blotato-api-key: $BLOTATO_API_KEY`. Not `Authorization`, not `Bearer`. A bad key returns 401.
- Rate limits: 30 requests/min for creating posts and for `POST /v2/media`, 60/min for reads. A 429 tells you how many seconds to wait, so read it rather than retrying blind.
- **A 201 means accepted, not published.** The real state comes from polling.

**Only if I reply "skip it":** produce the posts as a paste-ready file and say plainly that scheduling is off.

---

## STEP 0.2. ⛔ LIST THE ACCOUNTS BEFORE DRAFTING A WORD.

Every publish payload references an account by id, and the ids are bare numeric strings the owner cannot guess.

```bash
source .env && curl -s https://backend.blotato.com/v2/users/me/accounts \
  -H "blotato-api-key: $BLOTATO_API_KEY"
```

Returns `{"items":[{"id":"98432","platform":"twitter","fullname":"Jane Smith","username":"janesmith"}]}`. Use `items[].id`. Show the list back as a plain line and let me choose:

> "Connected in Blotato: LinkedIn (Your Name) · X (@yourhandle) · Instagram (Your Business) · Facebook page (Your Business). Not connected: TikTok, Threads. Want all four, or a subset?"

**Three platforms need a second call before they can be posted to at all.** Make it now, not at publish time:

- **Facebook** - `pageId` is REQUIRED. `GET /v2/users/me/accounts/{accountId}/subaccounts`
- **LinkedIn** - `pageId` only for a company page; omit it entirely to post as the personal profile. `GET /v2/integrations/oauth2/linkedin/pages?accountId=X`
- **Pinterest** - `boardId` is REQUIRED. `GET /v2/social/pinterest/boards?accountId=X`

**⛔ Never invent an account id, a platform key or a payload field.** If a call fails or the shape does not match this file, STOP and say exactly what came back. Blotato restructured its docs once already, so check `https://help.blotato.com/llms-full.txt` (the full text export, and the best source) or `https://backend.blotato.com/openapi.json` rather than writing to a schema you inferred. A guessed field name is how a batch reports success and schedules nothing.

**⛔ Reddit and Google Business Profile are not supported.** They are absent from the API entirely. If I ask for either, say so plainly - GBP posts are `/gbp-post`, which owns that surface through Metricool.

---

## STEP 1. Pick the source post. Never ask.

The ladder, top to bottom, no stopping:

1. The slug or URL I passed as an argument
2. No argument = the newest **Published** post in `blog-posts-log.md`
3. Log empty of published posts = the newest route under `website/app/blog/` that is not `_example-post`

State the pick in one line and continue. The ONLY stop: **the post is not published yet.** A social post pointing at a 404 is worse than no post, so if the source is a Draft, say so and offer `/publish` first.

**⛔ Read the actual page, not the keyword map row.** Open the built page and read the whole thing - the quick answer, the tables, the numbers, the FAQ. The map row is a plan; the page is what shipped, and they drift. Every claim in a social post has to exist on the page it links to.

**⛔ Check it has not already been posted.** Read `blotato-queue.md` before drafting. A source already listed under `## Posted to social` does not get a second run unless I say so - repeat announcements of one blog read as a bot, and that is the fastest way to get a feed muted.

---

## STEP 2. One source, one post per platform. Never one caption pasted five times.

**This is the whole point of the command, and it is the thing most likely to be done lazily.** The blog post is raw material. Each platform gets a post written FOR that platform that happens to share a source. If two drafts differ only by hashtag count, they are one post and the pass is not done.

Read `context/voice.md` Part 1 before drafting a line, then write to the dial per platform:

- **LinkedIn** - the longest. A hook line, a line break, then the story or the argument, then the takeaway. The first two lines are all anyone sees before "see more", so the hook carries the post. One good beat, not a standup set. Three hashtags maximum, at the end.
- **X (`twitter`)** - one post or a thread, decided deliberately. A single sharp claim beats a weak thread. If it threads, every tweet stands alone.
- **Instagram** - written to be read under a picture. First line is the hook because the rest truncates. Line breaks, not paragraphs.
- **Facebook** - plainer and warmer than LinkedIn, shorter than the blog. This is where a local business's actual customers are, so the local detail earns its place here most.
- **Threads / Bluesky** - conversational, short, no marketing cadence. A post that reads like an ad dies here.
- **TikTok / YouTube** - only if I asked, and only with a video. These are video platforms: TikTok requires seven separate flags in `target` and YouTube requires a title and privacy status. A caption with no video is not a post - say so rather than shipping one.

Non-negotiable on every platform:

- **Contractions throughout.** A post with none reads like a legal notice.
- **One line worth screenshotting.** If it would not make someone read it out to the person next to them, it is not the line yet.
- **Never joke about being bad at what you sell.** Effort, toll, personality, taste are fair. Competence is not.
- **Every number traces to `context/proof/proof-inventory.md` or to the blog post itself.** Never round a real number because it scans better, and never invent a stat the page does not contain.
- **No self-audit.** No section naming the devices used, no telling me it is funny. Show me the posts.

**⛔ EVERY POST DRIVES TRAFFIC TO THE ARTICLE. That is the entire job.** A post that gets a laugh and ends is a failed post, however good the joke. Three parts, non-negotiable on every platform:

1. **The link**, which is the published blog URL, verified to resolve. Not the home page, not a guess at the route. On Instagram, where a caption is not clickable, write "link in bio" and flag that the bio needs updating rather than pasting a dead-looking URL.
2. **The image.** Text alone does not stop a scroll. Pull the hero from the post itself so the social image and the article match.
3. **The ask**, as the last line, and it is **a full sentence that names the reader's situation**, not a fragment. A label reads like a section header and nobody clicks a header.
   - Wrong: "How many you actually need, by event type:" - that is a caption, and it assumes the reader has already decided they want it.
   - Right: "Planning a party and can't work out how much to order? Here's the guide:" - it names the person, names their problem, then hands over the link.
   - The shape: **[the situation they are in]? [what the thing is]:** Ask the question they are already asking themselves, then answer it with the link. "Read more" and "check it out" are not asks, they are throat-clearing.

**⛔ NEVER GIVE AWAY THE ANSWER IN THE POST.** This is the failure that looks like a good post. A draft that told the whole joke AND stated the number ("guests eat 6 pieces in hour one, 3 every hour after") has no reason to be clicked, because the reader already has the thing the article was written to give them. **Tease the number, deliver the joke, and put the payoff behind the link.** The story is the free part; the answer is the paid part.

The test on every draft, before it goes anywhere: **if a reader finished this post and never clicked, did they get everything they needed?** If yes, rewrite it. The post exists to move somebody to the page, not to be a small, self-contained piece of content on a platform that will never send you a customer.

Then run the gate before showing me anything:

```
python3 code/check_voice.py blotato-queue.md
```

A failing file gets fixed and re-run. It does not get shown to me with an explanation.

---

## STEP 3. The image. Every post ships with one.

The ladder, in order: a real photo from `context/proof/images/` → an image already on the blog post → stock via `python3 code/fetch_stock_photos.py "[query]" --count 8`.

**Look at every photo before it goes on a post.** Overfetch and pick - a pool of eight for four slots, never the same photo on two platforms, delete the rejects.

**⛔ Faces go where faces belong.** A team photo goes on a post about the team. A testimonial shot goes on the post telling that customer's story. Neither gets slapped on a generic tip because it was the nearest real photo.

**Media is just a public URL.** Blotato fetches it, so `content.mediaUrls` takes the live https URL directly and no upload call is needed. A local path like `website/public/images/...` queues the post and silently drops the image - publish the site first (`/publish`) and use the live URL.

Two optional endpoints, only when you need them:

- **Rehost on Blotato:** `POST /v2/media` with `{"url":"https://..."}` returns a `database.blotato.io` URL. Known flake: it intermittently returns "Failed to read media metadata", so wrap it in **two attempts with a 3 second gap** before treating it as broken.
- **A local file with no public URL:** `POST /v2/media/uploads` with `{"filename":"photo.jpg"}` returns `presignedUrl` and `publicUrl`. Then `PUT` the **raw bytes** to `presignedUrl` - not JSON, not multipart - and use `publicUrl`.

Video over 120 seconds is not auto-converted and must already meet the platform's specs.

---

## STEP 4. Show me the batch. One yes covers it.

Show the drafts as a legible list: platform, the hook line, the image, the link. Not a table - `references/output-format.md` applies here like every other file.

**Read the drafts in a row before you show me.** If they sound like one voice at one volume, that is the failure this command exists to prevent. Fix it before I see it.

---

## STEP 5. Schedule it. Spaced, never simultaneous.

**⛔ Never fire every platform at the same minute.** Same text, same link, same second, across five feeds is the signature of an automation tool, and at least two platforms downrank it. Space them 20 to 45 minutes apart, one platform per slot, starting at the next sensible hour.

The payload. `POST /v2/posts`:

```json
{
  "post": {
    "accountId": "98432",
    "content": {
      "text": "the post body",
      "mediaUrls": ["https://yoursite.com/images/hero.jpg"],
      "platform": "twitter"
    },
    "target": { "targetType": "twitter" }
  },
  "scheduledTime": "2026-10-02T15:00:00Z"
}
```

Four rules that decide whether this works:

- **`content.platform` and `target.targetType` must be the same string.** A mismatch is rejected.
- **`mediaUrls` is required.** Text-only posts send `[]`, never omit the key.
- **⛔ `scheduledTime` is a ROOT-LEVEL sibling of `post`.** Nested inside `post` or an `options` object it is *silently ignored* and the post goes out immediately. This is the one mistake that turns a scheduled month into an instant burst, and nothing in the response tells you it happened.
- **Omit `scheduledTime` entirely to publish now.** `useNextFreeSlot: true` uses the next calendar slot instead, and is ignored if `scheduledTime` is set.
- **⛔ THERE IS NO DRAFT STATE FOR X, LINKEDIN, FACEBOOK OR INSTAGRAM. `isDraft` EXISTS ON TIKTOK ONLY.** Verified against `https://backend.blotato.com/openapi.json` on 29 September 2026: `isDraft` appears on exactly one of the ten `target` variants, the TikTok one. Send `isDraft: true` inside a `twitter` target and Blotato **does not reject it** - it drops the unrecognised field and publishes immediately. That happened on a live account: the request returned 201, and the poll came back `published` with a public X URL. A post cannot be un-published, and there is no `DELETE /v2/posts/{id}`.

  **So "save it as a draft" is not a thing this command can do on the platforms that matter.** When somebody asks for a draft, give them a SCHEDULED post dated a week out instead, and say plainly that Blotato has no draft state for their platform. A schedule is reversible - `GET`, `PATCH` and `DELETE /v2/schedules/{id}` all exist - which makes it the safe equivalent, and the only one.

  **The general rule this cost us:** an unrecognised field in a Blotato payload is silently ignored, never rejected. So an untested flag is never tried first against a live account. Test it with `scheduledTime` set far in the future, confirm the behaviour on the schedule endpoint, and only then send anything that could publish.

Threads and multi-post: `content.additionalPosts` is an array of `{text, mediaUrls}`, and it works on **X, Bluesky and Threads only**. A LinkedIn carousel is different - pass 2 to 10 image URLs in `mediaUrls` and Blotato builds the PDF carousel itself. Instagram Stories cannot carry a caption, so `content.text` is an empty string there.

**⛔ A 201 is a receipt for acceptance, not for publication.** It returns `{"postSubmissionId": "..."}`. The post is only really out when `GET /v2/posts/{postSubmissionId}` reports `published`. Record the submission id in `blotato-queue.md` against its row, and poll it before ever calling a post live. A `failed` status is the thing worth catching, and it arrives minutes after the 201.

Write each approved post to `blotato-queue.md` with its platform, account id, scheduled time, body, media URL, link and submission id. Anything that is not a 2xx leaves the row Pending, gets ONE retry, then lands in a failure list with its status code and response body. Report queued/failed per platform honestly, and never mark a post queued on the strength of having fired the request.

**⛔ Verify the first batch by eye.** After the first successful schedule, send me to Blotato to confirm the post is really there with the right text, image and time. The API answering 201 proves Blotato received it; only the dashboard proves it is scheduled correctly. Wait for my confirmation on the first run - after that, the status poll is the per-post confirmation. `https://my.blotato.com/api-dashboard` shows the full payload and response per request, which is where to look first when something queues empty.

Managing what is already scheduled: `GET`, `PATCH` and `DELETE /v2/schedules/{id}`.

**Now, instead of scheduled** - only if I explicitly say "post them now". Say what it costs before doing it: five platforms at once is the exact pattern the spacing rule exists to avoid.

Finally, append the source post to a `## Posted to social` section in `blotato-queue.md` so Step 1's duplicate check can see it, and add one line to `blog-posts-log.md` under that post. No new top-level files, and the three states this repo already uses: **Pending · Queued · Published**.
