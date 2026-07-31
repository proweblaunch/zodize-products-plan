# Marketing Website Template

Every Zodize product ships a public marketing website before it ships a
product. This document is the scaffold specification for that site. A product
team customizes copy, imagery, and pricing tiers; it does not customize
information architecture, SEO plumbing, or the consent/analytics baseline
without an ADR (see [`../decisions/adr-template.md`](../decisions/adr-template.md)).

## Directory structure

```
marketing/
  content/
    home.md
    pricing.md
    features/
      <feature-slug>.md
    about.md
    contact.md
    legal/
      privacy-policy.md
      terms-of-service.md
      cookie-policy.md
  resources/
    seo/
      sitemap.xml
      robots.txt
    og/
      <page-slug>.png
  resources/js/
    pages/
      Home.vue
      Pricing.vue
      Features/Index.vue
      Features/Show.vue
      About.vue
      Contact.vue
      Legal/Privacy.vue
      Legal/Terms.vue
      Legal/Cookies.vue
    components/
      Nav.vue
      Footer.vue
      ConsentBanner.vue
      PricingTable.vue
  routes/
    marketing.php
```

## Required pages

Every product's marketing site MUST ship the following routes at minimum:

| Route | Purpose |
|---|---|
| `/` | Home — value proposition, primary CTA, social proof. |
| `/pricing` | Pricing tiers, feature comparison, FAQ on billing. |
| `/features` and `/features/{slug}` | Feature index and per-feature detail pages. |
| `/about` | Company/product narrative, team (if disclosed). |
| `/contact` | Contact form and support channels. |
| `/docs` | MUST link out to the product's documentation site (see [documentation-template.md](./documentation-template.md)); it MAY be a subdomain, not a section of the marketing site. |
| `/legal/privacy-policy` | Privacy policy — content owned by legal/compliance, not engineering. |
| `/legal/terms-of-service` | Terms of service. |
| `/status` | MUST link to the product's external status page (e.g. a hosted status provider); MUST NOT be self-hosted on infrastructure that the status page is meant to report on. |

A product MAY add pages (case studies, blog, integrations directory) but MUST
NOT omit any page in this table before GA. Absence of any of these pages is a
blocking item on [`../checklists/production-readiness-checklist.md`](../checklists/production-readiness-checklist.md).

## SEO requirements

Every page rendered by this template MUST include:

- A unique `<title>` (50–60 characters) and `<meta name="description">`
  (140–160 characters) per page — no two pages share a title or description.
- Open Graph tags (`og:title`, `og:description`, `og:image`, `og:url`,
  `og:type`) and Twitter Card tags on every page, with a dedicated
  `resources/og/<page-slug>.png` (1200×630) per page — the OG image MUST NOT
  default to a single site-wide fallback for indexable pages.
- A canonical `<link rel="canonical">` tag on every page.
- `resources/seo/sitemap.xml`, generated at build time (not hand-maintained),
  covering every indexable route, and referenced from `robots.txt`.
- `resources/seo/robots.txt` that allows crawling of all marketing routes and
  explicitly disallows any authenticated application routes.
- JSON-LD structured data: `Organization` schema site-wide, `Product` schema
  on `/pricing`, `FAQPage` schema on any page with an FAQ block, and
  `BreadcrumbList` on any page nested more than one level deep.
- Semantic heading order (exactly one `<h1>` per page) and descriptive `alt`
  text on every content image.

## Performance budget

The marketing site is the first thing a prospective customer's browser
renders and MUST meet the performance budget defined in
[`../quality/performance-standards.md`](../quality/performance-standards.md).
It MUST NOT ship a JavaScript framework runtime heavier than what is required
to hydrate interactive islands (nav, consent banner, pricing toggle); static
content MUST be served pre-rendered, not client-rendered on first paint.

## Content approach

Marketing content is **markdown-driven**, stored under `marketing/content/`
and rendered at build time. A product MUST NOT introduce a headless CMS
dependency for marketing content without an ADR — the markdown-driven
approach keeps marketing content in the same repository, review process, and
deploy pipeline as the product itself, which is a deliberate simplicity
decision. Each markdown file MUST carry frontmatter with, at minimum:
`title`, `description`, `og_image`, and `updated_at`.

## Analytics and consent

- The site MUST NOT load any analytics, advertising, or third-party tracking
  script before the visitor has made a consent choice via `ConsentBanner.vue`.
- Consent state MUST be stored client-side and MUST be re-requested if the
  consent policy version changes.
- Analytics events fired from the marketing site MUST NOT include personally
  identifiable information in the event payload; only anonymized/aggregate
  identifiers are permitted pre-consent.
- The consent banner MUST offer a functional "reject non-essential" path that
  is as easy to select as "accept" — no dark patterns.

## What ZodiCore provides vs. what a product customizes

ZodiCore provides: the `ConsentBanner.vue` component, the sitemap generator
build step, the shared `Nav.vue`/`Footer.vue` shell components, and the OG
image generation pipeline.

A product customizes: all content under `marketing/content/`, the pricing
tiers and their feature-comparison matrix, hero imagery, and any
product-specific feature pages. A product MUST NOT fork the SEO or consent
infrastructure — extend it via configuration, not by copying and modifying
the underlying component.
