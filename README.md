# Local SEO Agent

Nine Claude Code commands that take a local business from an empty Google Business Profile to a live website, ranked pages, and social posts that publish themselves.

Free. This is the whole thing from the video.

---

## Get started

```bash
git clone https://github.com/youtube-jono/local-seo-agent
cd local-seo-agent
claude
```

Then run:

```
/gbp-build
```

Everything else assumes the profile exists.

---

## The commands

**The profile**

| Command | What it does |
|---|---|
| `/gbp-build` | Fills the whole profile: 10 categories, 50 services ranked by search volume, 20 products, the 750-character description, hours, attributes and service area |
| `/gbp-build citations` | Just the citation campaign: 33 directories, tiered |
| `/reviews` | The review request sequence, the reply templates, and the automation so every finished job asks |
| `/gbp-post` | A month of profile posts, queued and dripped 2 to 3 a week |

**The website**

| Command | What it does |
|---|---|
| `/keyword-research` | The keyword map: every term, filtered, clustered, routed to a page type |
| `/service-page` | One money page per service and area pair |
| `/blog-post` | One local blog post, written to rank and to link down to a money page |
| `/build-website` | A full Next.js site if there is no website yet |
| `/publish` | Deploys: GitHub, Vercel, robots, sitemap, and the Search Console walkthrough |

**The distribution**

| Command | What it does |
|---|---|
| `/blotato` | Turns each published blog post into native social posts and schedules them |

---

## What you will need

Nothing to start. Each command asks for what it needs the first time it runs and records the answer.

As you go:

- **Semrush** for real search volumes. Without it, volumes are marked as estimates rather than invented.
- **Metricool** so `/gbp-post` can publish. Connect the Metricool connector in Claude and link your Google Business Profile to your Metricool brand; posts are scheduled straight into Metricool's planner. (A Make.com webhook is still supported as an optional route for real Offer and Event posts.)
- **A Pexels API key** for photos. Free, no card.
- **GitHub and Vercel** for `/publish`. One-time setup, then every future change is one push.
- **Blotato** for `/blotato`. Paid-only, so skip it unless you are scheduling social posts.

Copy `.env.example` to `.env` and fill them in as you are asked.

---

## Three things worth knowing

**The city never goes in a service name.** Services get ranked by `[service] [city]` search volume because that is the only way to measure local demand, but the city is a measuring device. "Emergency Plumber", never "Emergency Plumber Toronto". Putting it in the name is a suspension risk, not a ranking one.

**Local search volumes are small and that is fine.** A local term at 50 searches a month with buying intent beats a national one at 500 without it. The filters run at a lower floor for anything carrying a city.

**Local blog posts are not national guides with a city bolted on.** The demand sits one step earlier in the buying journey: the venue, the neighbourhood, the vendors your customer hires alongside you. "Best wedding venues in Toronto" outranks "catering Toronto" and reaches the same buyer weeks earlier.

---

## Licence

MIT. Use it, change it, ship it.
