---
name: matchpoint-therapeutics-track-the-news-archive
description: >-
  Monitor Matchpoint Therapeutics announcements — financings, partnerships and coverage — from the
  posts collection, and tell company-issued releases apart from reprinted third-party articles.
api: matchpoint-therapeutics:matchpoint-therapeutics-posts-api
generated: '2026-08-25'
method: generated
source: openapi/matchpoint-therapeutics-posts-openapi.yml
operations:
- listPosts
- getPost
- listCategories
---

# Track the Matchpoint Therapeutics news archive

Five posts at harvest time, spanning the October 2022 launch with $100 million through the July
2025 exclusive option and licence agreement with Novartis. Small, but it is the company's only
dated public record and the only place deal terms appear.

Base URL: `https://matchpointtx.com/wp-json`. No authentication. All GET.

## Steps

1. **Poll for new items by date, not by scanning.**
   `GET /wp/v2/posts?after=<last-seen ISO 8601>&orderby=date&order=asc&_fields=id,date,slug,title,link`
   (`listPosts`). `after` is server-side, so an empty array is a definitive "nothing new". Keep the
   `date` of the newest item you have seen as the cursor.

2. **Do not rely on categories to classify.**
   `GET /wp/v2/categories` (`listCategories`) returns exactly one term, `uncategorized` (id 1),
   carrying all five posts. `post_tag` is empty. The archive is taxonomically flat.

3. **Classify by title prefix instead.**
   Company-issued releases begin with "Matchpoint Therapeutics …". Reprinted third-party coverage
   is prefixed with the outlet — "Timmerman Report:", "BioSpace:", "BioCentury:". Treat the outlet
   prefix as a citation, and follow the outbound link inside `content.rendered` to the original
   rather than quoting the reprint as a company statement.

4. **Pull the body when a headline matters.**
   `GET /wp/v2/posts/{id}` (`getPost`). `content.rendered` is HTML and carries the substance —
   dateline city (which is how you learn the headquarters moved from Cambridge to Watertown between
   2022 and 2025), financial terms, and named quotes. Strip tags before summarising.

5. **Use the RSS feed if you only need a change signal.**
   `https://matchpointtx.com/feed/` is a plain RSS 2.0 mirror of the same five items and is cheaper
   to poll than the REST collection. There is no `/news/` index page — it returns 404.

## Conventions and failure modes

- Date filters: `after`, `before`, `modified_after`, `modified_before`, all ISO 8601. Full text via
  `search`, narrowable with `search_columns=post_title,post_content,post_excerpt`.
- `X-WP-Total` and `X-WP-TotalPages` on every collection response; `Link` carries rel="next".
- The archive has not moved since 2025-07-24. A long quiet period is normal here and is not a
  fetch failure.
- Responses are cached for ten minutes. No rate-limit headers and no published limit — poll at
  human cadence, not machine cadence.
