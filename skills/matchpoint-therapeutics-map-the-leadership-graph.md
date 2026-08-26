---
name: matchpoint-therapeutics-map-the-leadership-graph
description: >-
  Pull the full Matchpoint Therapeutics people graph — executives, board of directors, board
  observers and scientific founders — from the team custom post type, and reconstruct the role
  grouping the API does not expose.
api: matchpoint-therapeutics:matchpoint-therapeutics-team-api
generated: '2026-08-25'
method: generated
source: openapi/matchpoint-therapeutics-team-openapi.yml
operations:
- listTeamMembers
- getTeamMember
- listTeamTypes
- getMediaItem
---

# Map the Matchpoint Therapeutics leadership graph

Matchpoint publishes 19 named people as a `team` custom post type. This is the richest structured
data on the deployment, and the only place the company's affiliations (Atlas Venture, Access
Biotechnology, Sanofi Ventures, Vertex Ventures HC, Stanford, Dana-Farber, Harvard Medical School)
are stated in a machine-retrievable way.

Base URL: `https://matchpointtx.com/wp-json`. No authentication. All GET.

## Steps

1. **Check the taxonomy first, and expect it to be empty.**
   `GET /wp/v2/team_types` (`listTeamTypes`). It returns `[]` with `X-WP-Total: 0`. The taxonomy is
   registered and hierarchical, so it *looks* like the grouping is available — it is not. Do not
   build a role filter on it, and do not report a role you got from it.

2. **List the roster in one call.**
   `GET /wp/v2/team?per_page=100&_fields=id,slug,title,content,featured_media,link`
   (`listTeamMembers`). `per_page` maxes at 100 and there are 19 objects, so one page is the whole
   roster. Read `X-WP-Total` to confirm the count has not moved from 19 since 2026-08-25.

3. **Recover the role from the biography, not from a field.**
   Each member's role sits in `content.rendered` as HTML — e.g. "President & Chief Executive
   Officer", "Chairman; Partner, Atlas Venture", "Observer; Associate Professor, Dana-Farber Cancer
   Institute and Harvard Medical School". Strip tags and split on `;` to separate the Matchpoint
   role from the outside affiliation. Record the extraction as derived, because it is: the API
   asserts no role.

4. **Handle the duplicates.** At harvest time the roster carried two objects for Andre Turenne
   (ids 81 and 188) and two for Edward Chouchani (ids 187 and 202) — one under the executive
   presentation block and one under the board block. De-duplicate on normalised `title.rendered`,
   and treat the pair as evidence of a person holding two roles rather than as an error.

5. **Resolve headshots only if you need them.**
   `featured_media` is an integer. Either add `&_embed` to step 2 and read
   `_embedded['wp:featuredmedia'][0].source_url`, or call
   `GET /wp/v2/media/{featured_media}` (`getMediaItem`) per person. Prefer `_embed` — 19 extra
   round trips for images is not worth it.

## Conventions and failure modes

- Pagination: `page` / `per_page` (max 100), with `X-WP-Total` and `X-WP-TotalPages`. A `per_page`
  above 100 returns `400 rest_invalid_param`, it does not clamp.
- `author` on every member is an integer that cannot be resolved: `/wp/v2/users` returns
  `401 rest_user_cannot_view`. Ignore it.
- An unknown id returns `404 rest_post_invalid_id`. Switch on the `code` string, not the status —
  several distinct failures share 401. See `errors/matchpoint-therapeutics-problem-types.yml`.
- Responses are cached for ten minutes (`cache-control: max-age=600`) behind WP Engine and
  Cloudflare. There is no rate-limit header of any family; there is also no published limit, so be
  conservative.
- Read-only. There is no write, and therefore nothing to undo — see the `reversibility` block in
  `conventions/matchpoint-therapeutics-conventions.yml`.
