<p align="center"><a href="https://apify.com/DevilScrapes/lemmy-community-scraper"><img src="assets/banner.svg" alt="Lemmy Scraper — Posts, Comments &amp; Community Data — Python example" width="100%"></a></p>

# Lemmy Scraper in Python: community posts and comment threads for pandas

<p align="center">
<a href="https://apify.com/DevilScrapes/lemmy-community-scraper"><img alt="Run on Apify" src="https://img.shields.io/badge/Run%20on-Apify-DC2626?logo=apify&logoColor=white"></a>
<img alt="Python" src="https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-0A0606"></a>
<a href="https://github.com/devilscrapes/lemmy-scraper-python/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/devilscrapes/lemmy-scraper-python/actions/workflows/ci.yml/badge.svg"></a>
</p>

Here you will find a short Python script, covered by tests, that runs the **[Lemmy Community Scraper](https://apify.com/DevilScrapes/lemmy-community-scraper)** on Apify. Name a Lemmy server and one of its public communities, and you receive that community's posts, sorted the way you ask, with titles, bodies or links, vote counts and publish times; switch on comments and the replies arrive in the same table with their thread position. Lemmy is federated, so the same community can be read through different servers; this client reads it through whichever one you name. The API calls happen on Apify, and your machine only collects `results.json` and `results.csv`, ready for a DataFrame, a notebook or a corpus-building job.

> **Actor:** [Lemmy Scraper — Posts, Comments & Community Data](https://apify.com/DevilScrapes/lemmy-community-scraper) · **Publisher:** [Devil Scrapes](https://apify.com/DevilScrapes) · **Also in JavaScript:** [lemmy-scraper-javascript](https://github.com/devilscrapes/lemmy-scraper-javascript)

## 🔥 What you get

- **Any instance, any public community**: point `instanceUrl` at lemmy.world, lemmy.ml or a niche server, and name the community either locally (`asklemmy`) or with its home server (`memes@lemmy.world`).
- **Posts and comments share one table**: `row_type` separates them, and every comment row repeats its community and post columns, so a CSV opened on its own still says where each reply lives.
- **Thread structure kept**: Lemmy's dotted `comment_path` is stored as is, and `comment_parent_id` is derived from it, so a reply can be attached to its parent without another request.
- **Sixteen sort orders**: from `Hot` and `New` to time-boxed tops such as `TopDay` or `TopYear`. The old bare `Top` value is refused up front, since current Lemmy servers reject it.
- **Two caps you control**: `maxPosts` limits post rows and `maxCommentsPerPost` limits replies per post, with separate per-row prices for each kind.
- **Errors that say what went wrong**: a misspelled community fails the run with the server's own error, a run that yields nothing exits as failed, and busy-server responses are retried with backoff first.

## 🎯 Use cases

### Measure how a community engages week by week

Pull `TopWeek` every Monday from cron, append the rows and chart score, upvote ratio and comment volume over time. Because subscriber and post counts ride along on each row, growth of the community itself is in the same file.

### Assemble a threaded text corpus

Comment rows carry the text, score and position in the tree. Researchers working on discourse, toxicity or summarisation can rebuild conversations from parent IDs and keep the post title as context for every reply.

### Compare one topic across instances

Run the same federated name, such as `technology@lemmy.world`, against two servers and diff the results. Different instances may have fetched different slices of a remote community, and the per-row `instance_url` keeps the sources apart.

### Study the Reddit-alternative migration

Lemmy grew largely from people leaving Reddit, which makes its communities a natural sample. Post dates, author account creation dates and bot flags make it possible to separate early adopters from later arrivals in a notebook.

## ⚡ Quick start

You need an Apify account and an API token from [Console → Settings → Integrations](https://console.apify.com/settings/integrations). Apify's free plan includes monthly usage credit; this Actor's event prices are listed under Pricing below.

```bash
git clone https://github.com/devilscrapes/lemmy-scraper-python.git
cd lemmy-scraper-python
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
export APIFY_TOKEN=your_token_here
python main.py                      # uses input.json
python main.py my-input.json --out data --max-charge 2
```

The script starts the [Lemmy Scraper — Posts, Comments & Community Data](https://apify.com/DevilScrapes/lemmy-community-scraper), waits for it to finish, prints a link to the run in Apify Console and saves every result to `results.json` and `results.csv`. It exits with code 1 and the run's status message if the run fails.

## 🧩 The core call

Reduced to essentials, the integration looks like this. The official `apify-client` package starts the run, waits on it and then streams the dataset back item by item.

```python
from apify_client import ApifyClient

client = ApifyClient("YOUR_APIFY_TOKEN")
run = client.actor("DevilScrapes/lemmy-community-scraper").call(run_input={'instanceUrl': 'https://lemmy.world', 'communityName': 'asklemmy', 'sort': 'TopWeek', 'maxPosts': 3, 'includeComments': True, 'maxCommentsPerPost': 2})
for item in client.dataset(run.default_dataset_id).iterate_items():
    print(item)
```

## 📥 Input

`main.py` passes `input.json` through untouched. The sample reads the top three `asklemmy` posts of the past week on lemmy.world, plus up to two comments on each, so at most nine rows are billed. Raise `maxPosts` for a real study; turn `includeComments` off when titles and scores are all you need.

```json
{
  "instanceUrl": "https://lemmy.world",
  "communityName": "asklemmy",
  "sort": "TopWeek",
  "maxPosts": 3,
  "includeComments": true,
  "maxCommentsPerPost": 2
}
```

| Field | Type | Required | Default | Allowed values |
|---|---|---|---|---|
| `instanceUrl` | string | yes | `"https://lemmy.world"` | — |
| `communityName` | string | yes | `"asklemmy"` | — |
| `sort` | string | no | `"Hot"` | `Active`, `Controversial`, `Hot`, `MostComments`, `New`, `NewComments`, `Old`, `Scaled`, `TopAll`, `TopDay`, `TopHour`, `TopMonth`, `TopSixHour`, `TopTwelveHour`, `TopWeek`, `TopYear` |
| `maxPosts` | integer | no | `100` | — |
| `includeComments` | boolean | no | `false` | — |
| `maxCommentsPerPost` | integer | no | `50` | — |
| `useProxy` | boolean | no | `false` | — |

## 📤 Output

Rows come in two kinds, told apart by `row_type`. On post rows the `comment_*` columns are `null`; on comment rows the `author_*` columns describe the commenter, not the post author. Timestamps are ISO 8601 in UTC.

| Field | Type | Always present |
|---|---|---|
| `row_type` | string | yes |
| `instance_url` | string | yes |
| `community_actor_id` | string | yes |
| `community_name` | string | yes |
| `community_title` | string | yes |
| `community_description` | string or null | no |
| `community_subscribers` | integer | yes |
| `community_posts_count` | integer | yes |
| `community_local` | boolean | yes |
| `post_id` | integer | yes |
| `post_ap_id` | string | yes |
| `post_url` | string | yes |
| `post_title` | string | yes |
| `post_body` | string or null | no |
| `post_external_url` | string or null | no |
| `post_score` | integer | yes |
| `post_upvotes` | integer | yes |
| `post_downvotes` | integer | yes |
| `post_comments_count` | integer | yes |
| `post_published` | string | yes |
| `post_updated` | string or null | no |
| `post_nsfw` | boolean | yes |
| `post_featured_community` | boolean | yes |
| `post_locked` | boolean | yes |
| `comment_id` | integer or null | no |
| `comment_ap_id` | string or null | no |
| `comment_path` | string or null | no |
| `comment_content` | string or null | no |
| `comment_score` | integer or null | no |
| `comment_published` | string or null | no |
| `comment_parent_id` | integer or null | no |
| `author_actor_id` | string | yes |
| `author_name` | string | yes |
| `author_display_name` | string or null | no |
| `author_bot_account` | boolean | yes |
| `author_published` | string | yes |
| `scraped_at` | string | yes |

## 💰 Pricing

| Event | Price (USD) |
|---|---|
| Actor start | $0.2 |
| Post row emitted | $0.002 |
| Comment row emitted | $0.001 |

Prices as configured on 2026-08-20. The [Actor page](https://apify.com/DevilScrapes/lemmy-community-scraper) always shows the current price. Runs are billed to the Apify account that owns the token. Use `--max-charge` to cap what a single run can cost.

## 🛠️ Working with the data in Python

This pandas sketch (`pip install pandas`) splits posts from comments, adds an upvote ratio and an age in hours to each post, and measures how deep the sampled comment threads go. Depth comes from counting the dots in `comment_path`, so 1 means a top-level reply.

```python
import json

import pandas as pd

rows = pd.DataFrame(json.load(open("results.json", encoding="utf-8")))
posts = rows[rows["row_type"] == "post"].copy()
comments = rows[rows["row_type"] == "comment"].copy()

votes = (posts["post_upvotes"] + posts["post_downvotes"]).clip(lower=1)
posts["upvote_ratio"] = posts["post_upvotes"] / votes
published = pd.to_datetime(posts["post_published"], utc=True, format="ISO8601")
posts["age_hours"] = (pd.Timestamp.now(tz="UTC") - published).dt.total_seconds() / 3600

comments["depth"] = comments["comment_path"].str.count(r"\.")
threads = comments.groupby("post_id").agg(
    sampled=("comment_id", "count"),
    deepest=("depth", "max"),
    replies=("comment_parent_id", lambda s: s.notna().sum()),
)
report = posts.set_index("post_id").join(threads)
cols = ["post_title", "post_score", "upvote_ratio", "post_comments_count", "sampled", "deepest", "age_hours"]
print(report[cols].sort_values("post_score", ascending=False).round(2).to_string())
```

## ❓ FAQ

### Do I need a Lemmy account or an API key?

No. Public communities are readable without logging in, and the Actor sends no credentials. The only secret in this repository is your Apify token.

### Which instance should I use for a remote community?

The community's home server is the safest choice, so for `memes@lemmy.world` use `https://lemmy.world`. Reading through another instance works only if that server has already federated the community, and it may hold fewer posts.

### Can one run cover several communities?

Not inside the Actor; it reads one community per run. From Python, loop over a list of names, call the Actor for each and concatenate the frames; `community_actor_id` keeps them distinguishable.

### Why do some comment counts not match `post_comments_count`?

`post_comments_count` is Lemmy's total for the post, while comment rows stop at `maxCommentsPerPost`. Treat the comment rows as a sample of the discussion unless the cap is above the total.

## 🧯 Troubleshooting

- **`Set APIFY_TOKEN first`**: export the token in the same shell, or put it in a local `.env` file your shell loads. Never commit it.
- **Run finished with `FAILED` or `ABORTED`**: open the printed Console link; the run log says why. Input validation errors name the field.
- **Fewer rows than expected**: a `--max-charge` cap also limits how many results can be charged. Raise the cap or lower the limits in your input.
- **Something looks wrong in the data**: open an issue on the [Actor page](https://apify.com/DevilScrapes/lemmy-community-scraper) so the maintainers see it.

## 🔗 More from Devil Scrapes

- [Lemmy Scraper — Posts, Comments & Community Data on Apify Store](https://apify.com/DevilScrapes/lemmy-community-scraper)
- [JavaScript version of this example](https://github.com/devilscrapes/lemmy-scraper-javascript)
- [All Devil Scrapes Actors](https://apify.com/DevilScrapes)
- [Apify web scraping cookbook](https://github.com/devilscrapes/apify-web-scraping-cookbook)

Related Actors:

- [Mastodon Posts Scraper — Hashtags & Accounts](https://apify.com/DevilScrapes/mastodon-posts-scraper) — the microblogging side of the fediverse, by hashtag or account.
- [Bluesky Scraper — Feed Posts](https://apify.com/DevilScrapes/bluesky-feed-posts) — posts from public Bluesky custom feeds in a similar flat shape.
- [Hacker News Scraper](https://apify.com/DevilScrapes/hacker-news-scraper) — threaded tech discussion from a single, centralised site.

---

MIT licensed example code. The Actor itself runs on [Apify](https://apify.com/DevilScrapes/lemmy-community-scraper) and is maintained by [Devil Scrapes](https://apify.com/DevilScrapes). Not affiliated with the Lemmy project or any Lemmy instance.
