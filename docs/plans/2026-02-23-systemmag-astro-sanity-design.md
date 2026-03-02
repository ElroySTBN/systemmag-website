# SYSTEMMAG Website — Astro + Sanity Design

**Date:** 2026-02-23
**Status:** Approved

## Context

SYSTEMMAG est un fabricant français de fermetures magnétiques textiles (9 brevets internationaux, agréé US Army 2014, basé Paris 11e). Le site précédent (systemmag-next/ Next.js) a été supprimé. Ce document définit la nouvelle architecture.

## Decision

**Astro + Sanity** — site vitrine B2B statique ultra-performant avec CMS headless.

Astro choisi car :
- Site vitrine B2B = peu d'interactivité client-side → 0 JS par défaut
- Meilleurs scores Core Web Vitals / SEO (critique pour GEO optimization)
- Sanity intégré officiel via `@sanity/astro`
- Astro Islands pour les animations spécifiques (React islands)

## Architecture

```
systemmag-website/
└── systemmag-astro/
    ├── src/
    │   ├── pages/
    │   │   ├── index.astro
    │   │   ├── technologie.astro
    │   │   ├── secteurs/
    │   │   │   ├── defense.astro
    │   │   │   ├── medical.astro
    │   │   │   └── mode.astro
    │   │   ├── produits/
    │   │   │   └── index.astro       ← Dynamique Sanity
    │   │   ├── articles/
    │   │   │   ├── index.astro       ← Liste articles
    │   │   │   └── [slug].astro      ← Article individuel
    │   │   ├── histoire.astro
    │   │   ├── contact.astro
    │   │   └── studio/               ← Sanity Studio embed
    │   ├── components/
    │   │   ├── Navigation.astro
    │   │   ├── Footer.astro
    │   │   └── AnimatedCounter.tsx   ← React island
    │   ├── layouts/
    │   │   └── BaseLayout.astro
    │   └── lib/
    │       └── sanity.ts
    ├── sanity/
    │   ├── schemaTypes/
    │   │   ├── product.ts
    │   │   └── article.ts
    │   └── sanity.config.ts
    └── astro.config.mjs
```

## Content Managed by Sanity

### Document Types

1. **`product`**
   - `title` (string)
   - `family` (string: BA | ZipC | ZipG | ZipCE | ZipCD | Fourreaux)
   - `sku` (string)
   - `description` (text)
   - `width` (string: V04/V06/V08/V12)
   - `magnetGrade` (string: N38/N40/N45/N48/N52)
   - `sectors` (array: defense/medical/fashion/industry)
   - `pdfDatasheet` (file)
   - `image` (image)

2. **`article`**
   - `title` (string)
   - `slug` (slug)
   - `excerpt` (text)
   - `body` (Portable Text)
   - `sector` (string)
   - `publishedAt` (datetime)
   - `seoTitle`, `seoDescription` (string)

## Design System

- **Background:** #0C0C0F (near black)
- **Primary:** #1A56FF (blue)
- **Accent:** #C9A84C (gold)
- **Text:** #FFFFFF / #8A8A9A (secondary)
- **Typography:** Space Grotesk (display) + Inter (body)
- **Framework CSS:** Tailwind CSS v4

## Pages

| Page | Type | Data Source |
|------|------|-------------|
| `/` | Static | Hardcoded |
| `/technologie` | Static | Hardcoded |
| `/secteurs/defense` | Static | Hardcoded |
| `/secteurs/medical` | Static | Hardcoded |
| `/secteurs/mode` | Static | Hardcoded |
| `/produits` | Dynamic | Sanity |
| `/articles` | Dynamic | Sanity |
| `/articles/[slug]` | Dynamic | Sanity |
| `/histoire` | Static | Hardcoded |
| `/contact` | Static | Hardcoded |
| `/studio` | Studio | Sanity |

## Astro Islands Strategy

Only components needing client-side interactivity use React islands:
- `AnimatedCounter.tsx` — animated stats (patents: 9, years: 28+, washes: 10,000)
- `HeroParallax.tsx` — parallax scroll effect on hero
- Navigation mobile menu (Astro + minimal vanilla JS)

## SEO / GEO

- Schema.org Organization structured data in BaseLayout
- OpenGraph + Twitter cards
- `llms.txt` in public/ for AI discoverability
- Static HTML = all content indexable at parse time
