# hot.angels.kz
Rebuild SPA → Nuxt SSR for SEO + self-service CMS. No visual redesign.

## Phase
Discovery

## Stakeholders
| Name | Role |
|------|------|
| Саша | Client / Decision Maker (company owner) |
| Олег | Client-side SME (advertising & PM background) |
| Женя | Backend Architect (CakePHP) |
| Слава | Frontend Architect (Vue.js) |

## Stack
- Nuxt (SSR — confirmed, required for SEO)
- CakePHP REST API
- MariaDB/MySQL
- VPS hosting — cheapest tier, provisioned by our team (include in budget)

## Decisions
- Rebuild required — current SPA has no SSR support
- Design preserved — no visual redesign
- Mobile-first — desktop deferred to Phase 2
- Fixed-price commercial model
- Our team creates: GTM account, GA4 property, Google Search Console property
- TikTok Pixel + Facebook Pixel via GTM
- Admin CMS access — available (client has it)
- Team interest is experimental (AI tooling); compensation modest but present

## Site Structure

### Public pages
| URL | Page |
|-----|------|
| `/` | Home — hero, social links, featured products, CTA |
| `/sauces` | Sauces — product grid |
| `/sauces/:slug` | Sauce detail — photos, description, SEO text |
| `/peppers` | Peppers — product grid |
| `/peppers/:slug` | Pepper detail — photos, description, SEO text |
| `/where-to-buy` | Where to buy — store addresses, filter by city |
| `/faq` | FAQ — with FAQPage Schema markup |

### Mobile bottom nav
Главная · Соусы · Перцы · Где купить · Позвонить · WhatsApp

### Admin (private)
| URL | Manages |
|-----|---------|
| `/admin/sauces` | Sauce catalog |
| `/admin/peppers` | Pepper catalog |
| `/admin/faq` | FAQ entries |
| `/admin/stores` | Store addresses |
| `/admin/seo` | SEO globals (title, description, footer text) |

## Scope — Phase 1
- Nuxt SSR rebuild (existing design)
- Admin panel: SEO globals, FAQ page, store locations, product catalog
- Technical SEO: sitemap, robots, meta/OG tags, Schema markup (audit first)
- AI indexation: FAQ structured for LLM search engines
- Analytics setup: GTM + GA4 + Search Console + TikTok + FB pixels
- Social links carry-over
- Baseline "before" snapshot before any code changes (keyword positions, PageSpeed mobile, organic traffic)

## Out of Scope — Phase 1
- Desktop layout
- Shopping cart (process after cart undefined — see open questions)
- Payment gateway
- Email/WhatsApp notification funnel
- Promotions system

## Open Questions
1. Cart flow — what happens after customer fills cart? (WhatsApp order / email / payment / other)
2. Store data — will Sasha provide store addresses or do we build placeholders?
3. Budget range for Phase 1
4. Target keywords — which queries should we track for baseline?

## Next Steps
1. Set up GA4 + Search Console on current site; record keyword positions + PageSpeed baseline
2. Discovery call with Саша/Олег — resolve open questions
3. Schema/SEO audit of current site
4. Produce Phase 1 acceptance criteria + fixed-price proposal
