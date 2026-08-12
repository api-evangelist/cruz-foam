---
name: Read the Cruz Foam news archive
description: >-
  Search and retrieve Cruz Foam's published articles on compostable packaging, chitin materials and
  cold-chain sustainability from the public read-only content API. No credentials required.
api: openapi/cruz-foam-posts-api-openapi.yml
operations: [search, listPosts, getPost, listTags, listCategories, getSeoHead]
---

# Read the Cruz Foam news archive

139 published articles on compostable packaging, expanded-polystyrene replacement, chitin-based
materials and cold-chain shipping, available anonymously as JSON.

Base URL: `https://cruzfoam.com/wp-json`

## Authentication

None. Anonymous HTTPS. The collection answers `Allow: GET`.

## Steps

1. **Search across every content type** — `search`
   `GET /wp/v2/search?search=<terms>&per_page=20`
   Returns lightweight `{id, title, url, type, subtype}` records across posts, pages and the
   customer showcase. Branch on `subtype`: `post` for an article, `customers` for a showcase entry,
   `page` for a marketing page. Each result's `_links.self` gives the correctly typed URL to fetch.

2. **Or walk the archive by date** — `listPosts`
   `GET /wp/v2/posts?per_page=100&orderby=date&order=desc&_fields=id,date,slug,title,link,tags`
   `X-WP-Total` was 139 at capture across 70 pages at the default page size. Narrow with
   `after=<ISO8601>` / `before=<ISO8601>`, or `modified_after` to pick up edits.

3. **Retrieve the article body** — `getPost`
   `GET /wp/v2/posts/{id}?_fields=id,date,modified,title,content,excerpt,link,tags`
   `content.rendered` is HTML, not markdown or plain text. `content.protected` tells you whether
   the body was withheld behind a post password.

4. **Get structured metadata instead of scraping** — `getSeoHead`
   `GET /yoast/v1/get_head?url=<article url>`
   Returns the parsed schema.org `@graph` for the article — headline, dates, author, images,
   breadcrumbs — as JSON-LD. Prefer this over parsing `content.rendered`.

5. **Taxonomy is thin — do not rely on it** — `listTags`, `listCategories`
   Only 9 tags and 1 category cover all 139 posts, and one tag ("Compostable Packagign", id 42) is
   a typo with zero posts attached. Full-text `search` is the reliable filter here.

## Conventions that will bite you

- **The archive is stale**: the most recent post at capture is dated 2025-04-04. Check `date`
  before presenting anything as current company news.
- **`per_page` max is 100**; over it returns HTTP 400 `rest_invalid_param`.
- **Edge caching**: responses carry `Cache-Control: s-maxage=2592000`. A freshly edited article can
  be served stale for up to 30 days. There is no conditional-request support to work around it.
- **`X-Robots-Tag: noindex`** is set on every response. The data is public and readable but the
  provider signals it should not be indexed as content — respect that in anything you publish.
- **No rate-limit headers and no support channel.** Self-throttle; there is nobody to ask.
