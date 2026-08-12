---
name: Browse the Cruz Foam customer showcase
description: >-
  Retrieve the brands that ship in Cruz Foam materials and the market segments they fall into,
  using the public read-only content API behind cruzfoam.com. No credentials required.
api: openapi/cruz-foam-customers-api-openapi.yml
operations: [listPortfolioCategories, listCustomers, getCustomer, getMediaItem]
---

# Browse the Cruz Foam customer showcase

Cruz Foam publishes no developer program. The only machine-readable surface is the WordPress REST
content API behind its marketing site, and the one genuinely company-specific resource on it is the
`customers` custom post type — 20 published brands, each tagged with market segments from the
`portfolio-categories` taxonomy.

Base URL: `https://cruzfoam.com/wp-json`

## Authentication

None. Every operation below is anonymous over HTTPS. Do not send an `Authorization` header — there
is no public credential to send, and the write methods that would use one are staff-only.

## Steps

1. **Get the segment vocabulary first** — `listPortfolioCategories`
   `GET /wp/v2/portfolio-categories?per_page=100&_fields=id,name,slug,count`
   Returns the eight segments (B2B, B2C, Cushioning, Food & Wine, Healthcare & Wellness,
   Industrial & Commercial, Insulation, Retail & Lifestyle) with a `count` of how many showcase
   entries carry each. You need the numeric `id` to filter in step 2. Unescape HTML entities in
   `name` — the API returns `Food &amp; Wine`.

2. **List the showcase** — `listCustomers`
   `GET /wp/v2/customers?per_page=100&_fields=id,slug,title,excerpt,portfolio-categories,link`
   All 20 records fit in one page. Filter to a segment with
   `GET /wp/v2/customers?portfolio-categories=<id>`.
   `title` and `excerpt` are objects shaped `{"rendered": "..."}`, not strings — read `.rendered`.

3. **Retrieve one entry in full** — `getCustomer`
   `GET /wp/v2/customers/{id}`
   The full record is roughly 12KB and includes an `acf` block carrying the showcase detail and a
   `yoast_head_json` block carrying the schema.org metadata. Add `_embed` to inline the featured
   image and terms in the same round trip instead of calling step 4.

4. **Resolve imagery** — `getMediaItem`
   `GET /wp/v2/media/{featured_media}`
   Read `source_url` for the original and `media_details.sizes` for the generated variants.

## Conventions that will bite you

- **Pagination**: `per_page` is capped at 100. Exceeding it returns HTTP 400 `rest_invalid_param`,
  not a clamped result. Read `X-WP-Total` and `X-WP-TotalPages` for the record count.
- **Always send `_fields`**: an unfiltered customer record is ~12KB and a post is ~16KB, mostly
  rendered HTML and SEO metadata you did not ask for.
- **Ids are not typed**: post, page, media and customer ids share one integer sequence. Carry the
  resource type with the id or you will fetch the wrong object.
- **Errors**: match on `code`, never on `message`. `404 rest_post_invalid_id` means "not readable",
  which includes unpublished — it does not prove the record does not exist. Full catalog in
  `errors/cruz-foam-problem-types.yml`.
- **No rate-limit signal**: no `RateLimit-*` or `Retry-After` header is ever returned. Self-throttle.
- **No idempotency, no versioning commitment, no status page.** Treat this surface as unmanaged and
  cache what you read; see `lifecycle/cruz-foam-lifecycle.yml`.
