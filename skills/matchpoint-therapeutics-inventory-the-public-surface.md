---
name: matchpoint-therapeutics-inventory-the-public-surface
description: >-
  Enumerate everything Matchpoint Therapeutics exposes anonymously — every route, content type and
  media asset — and establish what is genuinely absent, before concluding anything about the
  company's API posture.
api: matchpoint-therapeutics:matchpoint-therapeutics-discovery-api
generated: '2026-08-25'
method: generated
source: openapi/matchpoint-therapeutics-discovery-openapi.yml
operations:
- getRouteIndex
- listTypes
- getType
- listTaxonomies
- getTaxonomy
- listStatuses
- searchContent
- listMedia
---

# Inventory the Matchpoint Therapeutics public surface

Matchpoint runs no developer program, so there is no reference to read. The route index *is* the
reference. This skill is how you establish what exists without guessing, and — just as importantly
— how you establish an absence you can defend.

Base URL: `https://matchpointtx.com/wp-json`. No authentication. All GET.

## Steps

1. **Start at the route index.**
   `GET /` (`getRouteIndex`). At harvest time: 222 routes across 13 namespaces, `authentication: []`
   (nothing advertised), site name "Matchpoint Therapeutics". Every route carries an `args` object
   that is the real parameter contract — this is what the nine specs in `openapi/` were derived
   from. Narrow with `?namespace=wp/v2` (`getWpV2Index`) for the 118 core routes.

2. **Find the company-specific content types.**
   `GET /wp/v2/types` (`listTypes`) then `GET /wp/v2/types/team` (`getType`). Twelve post types are
   registered; eleven are WordPress infrastructure (post, page, attachment, nav_menu_item, wp_block,
   templates, global styles, navigation, fonts). Exactly one — `team` — is Matchpoint's own. Read
   `rest_base` and `rest_namespace` off the descriptor to build the collection URL rather than
   assuming it.

3. **Check every taxonomy for emptiness before trusting it.**
   `GET /wp/v2/taxonomies` (`listTaxonomies`), then hit each `rest_base` and read `X-WP-Total`. Here
   `category` has 1 term, `post_tag` has 0, `team_types` has 0. A registered taxonomy is not a
   populated one, and the difference decides whether a filter you build will ever return anything.

4. **Confirm the anonymous scope.**
   `GET /wp/v2/statuses` (`listStatuses`) returns only `publish` to an anonymous caller. Everything
   in `context=edit` requires a WordPress user you will not be issued.

5. **Take one census pass.**
   `GET /wp/v2/search?search=<term>&per_page=100` (`searchContent`) spans posts, pages and team in
   one call — 27 objects total. `GET /wp/v2/media?per_page=100&_fields=id,mime_type,source_url`
   (`listMedia`) gives the 43 image assets. Together with the counts from step 3 that is the entire
   public content inventory.

6. **Record the absences as findings, with their status codes.**
   Verified 2026-08-25: `/openapi.json`, `/swagger.json`, `/api-docs`, `/graphql`, `/api`,
   `/developers`, `/llms.txt` and every `/.well-known/` path (security.txt, openid-configuration,
   oauth-authorization-server, api-catalog, ai-plugin.json, agent-card.json, agent.json) all
   return 404. `wp-abilities/v1` — the surface a WordPress MCP bridge would use — is registered but
   returns `401 rest_forbidden`. State those as probed absences with codes; do not state them as
   "no API".

## Conventions and failure modes

- Two soft-404 bodies exist and they tell you which layer answered: a 146-byte nginx body for paths
  outside the WordPress rewrite, and a ~23KB theme body for paths inside it. Neither is a 200, so
  neither can be mistaken for a served document.
- Plugin routes do not all follow core conventions: `/acf/v3/options/{name}` returns a bare
  `forbidden` code at 403, and `/customgf/v2/forms/1` returned HTTP 500 with no JSON envelope at
  all. Handle a non-envelope failure.
- Do not exercise `POST /customgf/v2/forms/{id}/submissions`. It is the deployment's one
  unauthenticated write route, it delivers a contact-form submission to a real human inbox, and
  Matchpoint publishes no retraction path.
