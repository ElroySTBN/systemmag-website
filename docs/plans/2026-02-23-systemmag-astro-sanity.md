# SYSTEMMAG Website — Astro + Sanity Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build the complete SYSTEMMAG B2B website with Astro v5 (static HTML), Sanity v3 CMS (products + articles), and Sanity Studio embedded at `/studio`.

**Architecture:** Monorepo in `systemmag-astro/` — Astro v5 with App Router, React islands for animations, Sanity as headless CMS for the product catalog and GEO blog articles. All other pages are static Astro components.

**Tech Stack:** Astro v5, Sanity v3 (`@sanity/astro`), React (`@astrojs/react`), Tailwind CSS v4 (Vite plugin), TypeScript, Framer Motion (React island only)

---

## Task 1: Initialize Astro Project

**Files:**
- Create: `systemmag-astro/` (via CLI)

**Step 1: Create the Astro project**

Run from `/Users/elroysitbon/systemmag-website/`:
```bash
npm create astro@latest systemmag-astro -- --template minimal --typescript strict --install --no-git
```

**Step 2: Verify project created**

```bash
ls systemmag-astro/
```
Expected: `src/`, `astro.config.mjs`, `package.json`, `tsconfig.json`

**Step 3: Commit**

```bash
cd systemmag-astro && git init && git add -A && git commit -m "feat: initialize Astro v5 project"
```

---

## Task 2: Install All Dependencies

**Files:**
- Modify: `systemmag-astro/package.json`

**Step 1: Add all integrations and dependencies**

Run from `systemmag-astro/`:
```bash
npx astro add react --yes
npx astro add sanity --yes
npm install tailwindcss @tailwindcss/vite framer-motion lucide-react
npm install @sanity/client @sanity/image-url
```

**Step 2: Verify `astro.config.mjs` now has react + sanity integrations**

Expected content (will need to be updated in Task 3):
```js
import { defineConfig } from 'astro/config'
import react from '@astrojs/react'
import sanity from '@sanity/astro'
```

**Step 3: Commit**

```bash
git add -A && git commit -m "feat: install react, sanity, tailwind, framer-motion dependencies"
```

---

## Task 3: Configure `astro.config.mjs` with Tailwind + Sanity + React

**Files:**
- Modify: `systemmag-astro/astro.config.mjs`
- Create: `systemmag-astro/.env.example`
- Create: `systemmag-astro/.env` (with placeholder values for dev)

**Step 1: Write the full `astro.config.mjs`**

```js
import { defineConfig } from 'astro/config'
import react from '@astrojs/react'
import sanity from '@sanity/astro'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  integrations: [
    react(),
    sanity({
      projectId: import.meta.env.SANITY_PROJECT_ID || 'placeholder',
      dataset: import.meta.env.SANITY_DATASET || 'production',
      useCdn: false,
      studioBasePath: '/studio',
    }),
  ],
  vite: {
    plugins: [tailwindcss()],
  },
  output: 'static',
})
```

**Step 2: Create `.env` with placeholder values**

```env
SANITY_PROJECT_ID=placeholder
SANITY_DATASET=production
SANITY_API_TOKEN=
```

**Step 3: Create `.env.example`**

```env
SANITY_PROJECT_ID=your_project_id_here
SANITY_DATASET=production
SANITY_API_TOKEN=optional_for_draft_mode
```

**Step 4: Create `src/styles/global.css`**

```css
@import "tailwindcss";

:root {
  --black: #0C0C0F;
  --white: #FFFFFF;
  --blue: #1A56FF;
  --gold: #C9A84C;
  --gray: #8A8A9A;
  --gray-dark: #1A1A24;
}

html {
  scroll-behavior: smooth;
  background-color: var(--black);
  color: var(--white);
}

body {
  font-family: 'Inter', sans-serif;
  background-color: var(--black);
}

::-webkit-scrollbar {
  width: 4px;
}
::-webkit-scrollbar-track {
  background: var(--black);
}
::-webkit-scrollbar-thumb {
  background: var(--blue);
  border-radius: 2px;
}
```

**Step 5: Commit**

```bash
git add -A && git commit -m "feat: configure astro with tailwind v4, sanity, react"
```

---

## Task 4: Sanity Schemas

**Files:**
- Create: `systemmag-astro/sanity/schemaTypes/product.ts`
- Create: `systemmag-astro/sanity/schemaTypes/article.ts`
- Create: `systemmag-astro/sanity/schemaTypes/index.ts`
- Create: `systemmag-astro/sanity/sanity.config.ts`
- Create: `systemmag-astro/src/lib/sanity.ts`

**Step 1: Create `sanity/schemaTypes/product.ts`**

```typescript
import { defineType, defineField } from 'sanity'

export const product = defineType({
  name: 'product',
  title: 'Produit',
  type: 'document',
  fields: [
    defineField({
      name: 'title',
      title: 'Nom du produit',
      type: 'string',
      validation: (Rule) => Rule.required(),
    }),
    defineField({
      name: 'family',
      title: 'Famille',
      type: 'string',
      options: {
        list: [
          { title: 'Blocs & Bandes Aimant (BA)', value: 'BA' },
          { title: 'Zip C — Standard centré', value: 'ZipC' },
          { title: 'Zip G — Plat centré', value: 'ZipG' },
          { title: 'Zip CE/GE — Haute puissance', value: 'ZipCE' },
          { title: 'Zip CD/GD — Double aimant', value: 'ZipCD' },
          { title: 'Fourreaux', value: 'Fourreaux' },
        ],
      },
      validation: (Rule) => Rule.required(),
    }),
    defineField({
      name: 'sku',
      title: 'Référence SKU',
      type: 'string',
    }),
    defineField({
      name: 'description',
      title: 'Description',
      type: 'text',
      rows: 3,
    }),
    defineField({
      name: 'width',
      title: 'Largeur',
      type: 'string',
      options: {
        list: [
          { title: 'V04 — 6mm', value: 'V04' },
          { title: 'V06 — 8mm', value: 'V06' },
          { title: 'V08 — 10mm', value: 'V08' },
          { title: 'V12 — 14mm', value: 'V12' },
        ],
      },
    }),
    defineField({
      name: 'magnetGrade',
      title: 'Grade magnétique',
      type: 'string',
      options: {
        list: ['N38', 'N40', 'N45', 'N48', 'N52'].map((g) => ({
          title: g,
          value: g,
        })),
      },
    }),
    defineField({
      name: 'sectors',
      title: 'Secteurs d\'application',
      type: 'array',
      of: [{ type: 'string' }],
      options: {
        list: [
          { title: 'Défense & Militaire', value: 'defense' },
          { title: 'Médical & Santé', value: 'medical' },
          { title: 'Mode & Prêt-à-Porter', value: 'fashion' },
          { title: 'Industrie & OEM', value: 'industry' },
        ],
      },
    }),
    defineField({
      name: 'image',
      title: 'Image',
      type: 'image',
      options: { hotspot: true },
    }),
    defineField({
      name: 'pdfDatasheet',
      title: 'Fiche technique PDF',
      type: 'file',
      options: { accept: '.pdf' },
    }),
    defineField({
      name: 'usageNote',
      title: 'Note d\'utilisation',
      type: 'string',
    }),
  ],
  preview: {
    select: {
      title: 'title',
      subtitle: 'sku',
      media: 'image',
    },
  },
})
```

**Step 2: Create `sanity/schemaTypes/article.ts`**

```typescript
import { defineType, defineField } from 'sanity'

export const article = defineType({
  name: 'article',
  title: 'Article / GEO',
  type: 'document',
  fields: [
    defineField({
      name: 'title',
      title: 'Titre',
      type: 'string',
      validation: (Rule) => Rule.required(),
    }),
    defineField({
      name: 'slug',
      title: 'Slug URL',
      type: 'slug',
      options: { source: 'title', maxLength: 96 },
      validation: (Rule) => Rule.required(),
    }),
    defineField({
      name: 'excerpt',
      title: 'Résumé',
      type: 'text',
      rows: 2,
    }),
    defineField({
      name: 'body',
      title: 'Contenu',
      type: 'array',
      of: [
        { type: 'block' },
        {
          type: 'image',
          options: { hotspot: true },
          fields: [
            defineField({ name: 'alt', title: 'Alt text', type: 'string' }),
          ],
        },
      ],
    }),
    defineField({
      name: 'sector',
      title: 'Secteur',
      type: 'string',
      options: {
        list: [
          { title: 'Défense', value: 'defense' },
          { title: 'Médical', value: 'medical' },
          { title: 'Mode', value: 'fashion' },
          { title: 'Industrie', value: 'industry' },
          { title: 'Technologie', value: 'technology' },
          { title: 'Général', value: 'general' },
        ],
      },
    }),
    defineField({
      name: 'publishedAt',
      title: 'Date de publication',
      type: 'datetime',
    }),
    defineField({
      name: 'seoTitle',
      title: 'Titre SEO',
      type: 'string',
    }),
    defineField({
      name: 'seoDescription',
      title: 'Description SEO',
      type: 'text',
      rows: 2,
    }),
  ],
  preview: {
    select: {
      title: 'title',
      subtitle: 'publishedAt',
    },
  },
})
```

**Step 3: Create `sanity/schemaTypes/index.ts`**

```typescript
import { product } from './product'
import { article } from './article'

export const schemaTypes = [product, article]
```

**Step 4: Create `sanity/sanity.config.ts`**

```typescript
import { defineConfig } from 'sanity'
import { structureTool } from 'sanity/structure'
import { visionTool } from '@sanity/vision'
import { schemaTypes } from './schemaTypes'

export default defineConfig({
  name: 'systemmag',
  title: 'SYSTEMMAG',
  projectId: import.meta.env.SANITY_PROJECT_ID || 'placeholder',
  dataset: import.meta.env.SANITY_DATASET || 'production',
  plugins: [structureTool(), visionTool()],
  schema: {
    types: schemaTypes,
  },
})
```

**Step 5: Create `src/lib/sanity.ts`**

```typescript
import { createClient } from '@sanity/client'

export const sanityClient = createClient({
  projectId: import.meta.env.SANITY_PROJECT_ID || 'placeholder',
  dataset: import.meta.env.SANITY_DATASET || 'production',
  useCdn: true,
  apiVersion: '2024-01-01',
})

export async function getProducts() {
  return sanityClient.fetch(`
    *[_type == "product"] | order(family asc, sku asc) {
      _id,
      title,
      family,
      sku,
      description,
      width,
      magnetGrade,
      sectors,
      usageNote,
      "imageUrl": image.asset->url,
      "pdfUrl": pdfDatasheet.asset->url
    }
  `)
}

export async function getArticles() {
  return sanityClient.fetch(`
    *[_type == "article"] | order(publishedAt desc) {
      _id,
      title,
      "slug": slug.current,
      excerpt,
      sector,
      publishedAt
    }
  `)
}

export async function getArticleBySlug(slug: string) {
  return sanityClient.fetch(`
    *[_type == "article" && slug.current == $slug][0] {
      _id,
      title,
      "slug": slug.current,
      excerpt,
      body,
      sector,
      publishedAt,
      seoTitle,
      seoDescription
    }
  `, { slug })
}
```

**Step 6: Commit**

```bash
git add -A && git commit -m "feat: add sanity schemas (product, article) and client"
```

---

## Task 5: Base Layout + Navigation + Footer

**Files:**
- Create: `systemmag-astro/src/layouts/BaseLayout.astro`
- Create: `systemmag-astro/src/components/Navigation.astro`
- Create: `systemmag-astro/src/components/Footer.astro`

**Step 1: Create `src/layouts/BaseLayout.astro`**

```astro
---
import '../styles/global.css'
import Navigation from '../components/Navigation.astro'
import Footer from '../components/Footer.astro'

interface Props {
  title?: string
  description?: string
  ogImage?: string
}

const {
  title = 'SYSTEMMAG® — Fermeture Magnétique Brevetée Made in France',
  description = 'Pionnier mondial de la fermeture magnétique intégrée dans le textile. 9 brevets internationaux. Agréé Armée US. Secteurs Défense, Médical, Mode.',
  ogImage = '/og-image.jpg',
} = Astro.props

const schemaOrg = JSON.stringify({
  '@context': 'https://schema.org',
  '@type': 'Organization',
  name: 'SYSTEMMAG',
  legalName: 'SALOMÉ SAS',
  url: 'https://systemmag.com',
  logo: 'https://systemmag.com/logo.png',
  foundingDate: '1998',
  founder: {
    '@type': 'Person',
    name: 'Éric SITBON',
  },
  address: {
    '@type': 'PostalAddress',
    streetAddress: '20 rue Bouvier',
    addressLocality: 'Paris',
    postalCode: '75011',
    addressCountry: 'FR',
  },
  contactPoint: {
    '@type': 'ContactPoint',
    telephone: '+33-1-45-08-91-41',
    email: 'contact@systemmag.com',
    contactType: 'customer service',
  },
  numberOfPatents: 9,
})
---

<!doctype html>
<html lang="fr">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{title}</title>
    <meta name="description" content={description} />
    <meta name="robots" content="index, follow" />
    <meta property="og:title" content={title} />
    <meta property="og:description" content={description} />
    <meta property="og:image" content={ogImage} />
    <meta property="og:locale" content="fr_FR" />
    <meta property="og:type" content="website" />
    <meta name="twitter:card" content="summary_large_image" />
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
    <link
      href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=Inter:wght@400;500;600&display=swap"
      rel="stylesheet"
    />
    <script type="application/ld+json" set:html={schemaOrg} />
  </head>
  <body>
    <Navigation />
    <main>
      <slot />
    </main>
    <Footer />
  </body>
</html>
```

**Step 2: Create `src/components/Navigation.astro`**

```astro
---
const navLinks = [
  { href: '/technologie', label: 'Technologie' },
  { href: '/produits', label: 'Produits' },
  { href: '/histoire', label: 'Histoire' },
  { href: '/contact', label: 'Contact' },
]

const secteurs = [
  { href: '/secteurs/defense', label: 'Défense & Militaire' },
  { href: '/secteurs/medical', label: 'Médical & Santé' },
  { href: '/secteurs/mode', label: 'Mode & Prêt-à-Porter' },
]
---

<header id="nav" class="fixed top-0 left-0 right-0 z-50 transition-all duration-300">
  <nav class="max-w-7xl mx-auto px-6 py-4 flex items-center justify-between">
    <!-- Logo -->
    <a href="/" class="flex items-center gap-3 group">
      <div class="w-8 h-8 relative">
        <div class="absolute inset-0 bg-[#1A56FF] rounded-sm"></div>
        <div class="absolute inset-1.5 bg-white rounded-full"></div>
        <div class="absolute bottom-1 right-1 w-2 h-2 bg-[#C9A84C] rounded-full"></div>
      </div>
      <span class="font-bold tracking-wider text-white text-sm uppercase">SYSTEMMAG</span>
    </a>

    <!-- Desktop Nav -->
    <div class="hidden md:flex items-center gap-8">
      {navLinks.slice(0, 1).map(link => (
        <a href={link.href} class="text-xs uppercase tracking-widest text-gray-300 hover:text-white transition-colors">
          {link.label}
        </a>
      ))}

      <!-- Secteurs dropdown -->
      <div class="relative group">
        <button class="text-xs uppercase tracking-widest text-gray-300 hover:text-white transition-colors flex items-center gap-1">
          Secteurs
          <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
          </svg>
        </button>
        <div class="absolute top-full left-0 mt-2 w-56 bg-[#1A1A24] border border-white/10 rounded-lg overflow-hidden opacity-0 invisible group-hover:opacity-100 group-hover:visible transition-all duration-200">
          {secteurs.map(s => (
            <a href={s.href} class="block px-4 py-3 text-xs text-gray-300 hover:text-white hover:bg-white/5 transition-colors border-b border-white/5 last:border-0">
              {s.label}
            </a>
          ))}
        </div>
      </div>

      {navLinks.slice(1).map(link => (
        <a href={link.href} class="text-xs uppercase tracking-widest text-gray-300 hover:text-white transition-colors">
          {link.label}
        </a>
      ))}
    </div>

    <!-- CTA -->
    <a
      href="/contact"
      class="hidden md:block px-4 py-2 text-xs uppercase tracking-widest border border-[#1A56FF] text-[#1A56FF] hover:bg-[#1A56FF] hover:text-white transition-all duration-300 rounded"
    >
      Demander un échantillon
    </a>

    <!-- Mobile toggle -->
    <button id="mobile-toggle" class="md:hidden text-white p-2" aria-label="Menu">
      <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
      </svg>
    </button>
  </nav>

  <!-- Mobile menu -->
  <div id="mobile-menu" class="hidden md:hidden bg-[#0C0C0F] border-t border-white/10 px-6 py-4">
    {[...navLinks, ...secteurs].map(link => (
      <a href={link.href} class="block py-3 text-sm text-gray-300 hover:text-white border-b border-white/5 last:border-0">
        {link.label}
      </a>
    ))}
    <a href="/contact" class="block mt-4 px-4 py-3 text-center text-sm border border-[#1A56FF] text-[#1A56FF] rounded">
      Demander un échantillon
    </a>
  </div>
</header>

<script>
  // Scroll effect
  const nav = document.getElementById('nav')
  window.addEventListener('scroll', () => {
    if (window.scrollY > 50) {
      nav?.classList.add('bg-[#0C0C0F]/95', 'backdrop-blur-md', 'border-b', 'border-white/10')
    } else {
      nav?.classList.remove('bg-[#0C0C0F]/95', 'backdrop-blur-md', 'border-b', 'border-white/10')
    }
  })

  // Mobile toggle
  document.getElementById('mobile-toggle')?.addEventListener('click', () => {
    const menu = document.getElementById('mobile-menu')
    menu?.classList.toggle('hidden')
  })
</script>
```

**Step 3: Create `src/components/Footer.astro`**

```astro
---
const currentYear = new Date().getFullYear()
---

<footer class="bg-[#0C0C0F] border-t border-white/10 py-16 mt-20">
  <div class="max-w-7xl mx-auto px-6">
    <div class="grid grid-cols-1 md:grid-cols-4 gap-10 mb-12">
      <!-- Brand -->
      <div class="col-span-1 md:col-span-1">
        <div class="flex items-center gap-3 mb-4">
          <div class="w-8 h-8 relative">
            <div class="absolute inset-0 bg-[#1A56FF] rounded-sm"></div>
            <div class="absolute inset-1.5 bg-white rounded-full"></div>
            <div class="absolute bottom-1 right-1 w-2 h-2 bg-[#C9A84C] rounded-full"></div>
          </div>
          <span class="font-bold tracking-wider text-white text-sm uppercase">SYSTEMMAG</span>
        </div>
        <p class="text-xs text-gray-400 leading-relaxed mb-4">
          Pionnier mondial de la fermeture magnétique intégrée dans le textile. Breveté. Made in France.
        </p>
        <span class="inline-flex items-center gap-2 px-3 py-1 bg-[#C9A84C]/10 border border-[#C9A84C]/30 rounded text-[#C9A84C] text-xs">
          <span class="w-1.5 h-1.5 bg-[#C9A84C] rounded-full"></span>
          Made in France · Paris 11e
        </span>
      </div>

      <!-- Secteurs -->
      <div>
        <h4 class="text-xs uppercase tracking-widest text-white font-semibold mb-4">Secteurs</h4>
        <ul class="space-y-2">
          {[
            { href: '/secteurs/defense', label: 'Défense & Militaire' },
            { href: '/secteurs/medical', label: 'Médical & Santé' },
            { href: '/secteurs/mode', label: 'Mode & Prêt-à-Porter' },
          ].map(link => (
            <li>
              <a href={link.href} class="text-xs text-gray-400 hover:text-white transition-colors">
                {link.label}
              </a>
            </li>
          ))}
        </ul>
      </div>

      <!-- Entreprise -->
      <div>
        <h4 class="text-xs uppercase tracking-widest text-white font-semibold mb-4">Entreprise</h4>
        <ul class="space-y-2">
          {[
            { href: '/technologie', label: 'Technologie' },
            { href: '/histoire', label: 'Notre Histoire' },
            { href: '/produits', label: 'Produits' },
            { href: '/articles', label: 'Articles' },
            { href: '/contact', label: 'Contact' },
          ].map(link => (
            <li>
              <a href={link.href} class="text-xs text-gray-400 hover:text-white transition-colors">
                {link.label}
              </a>
            </li>
          ))}
        </ul>
      </div>

      <!-- Contact -->
      <div>
        <h4 class="text-xs uppercase tracking-widest text-white font-semibold mb-4">Contact</h4>
        <address class="not-italic space-y-2">
          <p class="text-xs text-gray-400">20 rue Bouvier<br />75011 Paris, France</p>
          <a href="tel:+33145089141" class="block text-xs text-gray-400 hover:text-white transition-colors">
            +33 1 45 08 91 41
          </a>
          <a href="mailto:contact@systemmag.com" class="block text-xs text-gray-400 hover:text-white transition-colors">
            contact@systemmag.com
          </a>
        </address>
      </div>
    </div>

    <div class="border-t border-white/10 pt-8 flex flex-col md:flex-row justify-between items-center gap-4">
      <p class="text-xs text-gray-500">
        © {currentYear} SYSTEMMAG® – SALOMÉ SAS. Tous droits réservés. 9 brevets internationaux protégés.
      </p>
      <div class="flex gap-4">
        <a href="/llms.txt" class="text-xs text-gray-500 hover:text-gray-300 transition-colors">llms.txt</a>
        <a href="/contact" class="text-xs text-gray-500 hover:text-gray-300 transition-colors">Mentions légales</a>
      </div>
    </div>
  </div>
</footer>
```

**Step 4: Commit**

```bash
git add -A && git commit -m "feat: add BaseLayout, Navigation, Footer components"
```

---

## Task 6: Animated Counter React Island

**Files:**
- Create: `systemmag-astro/src/components/AnimatedCounter.tsx`

**Step 1: Create `src/components/AnimatedCounter.tsx`**

```tsx
import { useEffect, useRef, useState } from 'react'

interface Props {
  target: number
  suffix?: string
  prefix?: string
  duration?: number
  label: string
}

export default function AnimatedCounter({
  target,
  suffix = '',
  prefix = '',
  duration = 2000,
  label,
}: Props) {
  const [count, setCount] = useState(0)
  const [started, setStarted] = useState(false)
  const ref = useRef<HTMLDivElement>(null)

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting && !started) {
          setStarted(true)
        }
      },
      { threshold: 0.5 }
    )
    if (ref.current) observer.observe(ref.current)
    return () => observer.disconnect()
  }, [started])

  useEffect(() => {
    if (!started) return
    const steps = 60
    const stepValue = target / steps
    const stepDuration = duration / steps
    let current = 0
    const timer = setInterval(() => {
      current += stepValue
      if (current >= target) {
        setCount(target)
        clearInterval(timer)
      } else {
        setCount(Math.floor(current))
      }
    }, stepDuration)
    return () => clearInterval(timer)
  }, [started, target, duration])

  return (
    <div ref={ref} className="text-center">
      <div className="text-4xl md:text-5xl font-bold text-white mb-2" style={{ fontFamily: 'Space Grotesk, sans-serif' }}>
        {prefix}{count.toLocaleString('fr-FR')}{suffix}
      </div>
      <div className="text-xs uppercase tracking-widest text-gray-400">{label}</div>
    </div>
  )
}
```

**Step 2: Commit**

```bash
git add -A && git commit -m "feat: add AnimatedCounter React island"
```

---

## Task 7: Homepage (`/`)

**Files:**
- Create: `systemmag-astro/src/pages/index.astro`

**Step 1: Create `src/pages/index.astro`**

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro'
import AnimatedCounter from '../components/AnimatedCounter.tsx'

const advantages = [
  { icon: '🔇', title: 'Invisible', desc: 'Intégré dans le tissu, surface vierge' },
  { icon: '🤫', title: 'Silencieux', desc: 'Zéro bruit d\'accrochage ou décrochage' },
  { icon: '✋', title: 'Une main', desc: 'Fermeture instantanée sans alignement' },
  { icon: '🌊', title: '10 000 lavages', desc: 'Résistance industrielle certifiée' },
  { icon: '🔥', title: '90°C', desc: 'Compatible lavage haute température' },
  { icon: '⚓', title: 'Anti-corrosion', desc: 'Résistant brouillard salin marin' },
  { icon: '✅', title: 'OEKO-TEX · REACH', desc: 'Certifié absence de substances nocives' },
  { icon: '⚡', title: '8x plus puissant', desc: 'Polarité Alternée brevetée' },
]

const sectors = [
  {
    slug: 'defense',
    title: 'Défense & Militaire',
    badge: 'Agréé US Army',
    desc: 'Silencieux, résistant au terrain, opérationnel en conditions extrêmes. Approuvé Armée US 2014.',
    color: '#1A56FF',
    icon: '🛡️',
  },
  {
    slug: 'medical',
    title: 'Médical & Santé',
    badge: 'OEKO-TEX Certifié',
    desc: 'Une main pour hémiplégiques, Parkinson, personnes âgées. Compatible décontamination hospitalière.',
    color: '#10B981',
    icon: '⚕️',
  },
  {
    slug: 'mode',
    title: 'Mode & Prêt-à-Porter',
    badge: 'Paris Design',
    desc: 'La fermeture que les maisons de couture attendaient. Invisible, prêt à piquer, Made in France.',
    color: '#C9A84C',
    icon: '✨',
  },
]

const competitors = [
  { name: 'SYSTEMMAG', integration: true, silence: true, oneHand: true, madeFR: true, terrain: true, patents: true, defense: true, highlight: true },
  { name: 'Fidlock', integration: false, silence: true, oneHand: true, madeFR: false, terrain: true, patents: true, defense: false },
  { name: 'Velcro', integration: false, silence: false, oneHand: false, madeFR: false, terrain: false, patents: false, defense: true },
  { name: 'Fermeture Éclair', integration: false, silence: true, oneHand: false, madeFR: false, terrain: false, patents: false, defense: false },
]
---

<BaseLayout>
  <!-- Hero -->
  <section class="relative min-h-screen flex items-center justify-center overflow-hidden pt-20">
    <div class="absolute inset-0 bg-gradient-to-br from-[#0C0C0F] via-[#0D1528] to-[#0C0C0F]"></div>
    <div class="absolute inset-0" style="background: radial-gradient(ellipse 80% 50% at 50% 0%, rgba(26,86,255,0.15) 0%, transparent 60%)"></div>

    <div class="relative z-10 max-w-5xl mx-auto px-6 text-center">
      <div class="inline-flex items-center gap-2 px-3 py-1.5 border border-[#1A56FF]/30 rounded-full text-[#1A56FF] text-xs uppercase tracking-widest mb-8">
        <span class="w-1.5 h-1.5 bg-[#1A56FF] rounded-full animate-pulse"></span>
        9 brevets internationaux · Made in France depuis 1998
      </div>

      <h1 class="text-5xl md:text-7xl font-bold mb-6 leading-tight" style="font-family: 'Space Grotesk', sans-serif;">
        La fermeture magnétique<br />
        <span class="bg-gradient-to-r from-[#1A56FF] to-[#C9A84C] bg-clip-text text-transparent">
          que personne ne peut copier
        </span>
      </h1>

      <p class="text-lg text-gray-400 mb-12 max-w-2xl mx-auto leading-relaxed">
        SYSTEMMAG révolutionne la fermeture textile avec une technologie de polarité alternée brevetée.
        Invisible. Silencieuse. Indestructible.
      </p>

      <div class="flex flex-col sm:flex-row gap-4 justify-center">
        <a
          href="/contact"
          class="px-8 py-4 bg-[#1A56FF] text-white font-semibold rounded hover:bg-[#1A56FF]/90 transition-all duration-300"
        >
          Demander un échantillon gratuit
        </a>
        <a
          href="/technologie"
          class="px-8 py-4 border border-white/20 text-white rounded hover:border-white/40 transition-all duration-300"
        >
          Découvrir la technologie
        </a>
      </div>
    </div>

    <!-- Scroll indicator -->
    <div class="absolute bottom-8 left-1/2 -translate-x-1/2 flex flex-col items-center gap-2">
      <span class="text-xs text-gray-500 uppercase tracking-widest">Défiler</span>
      <div class="w-px h-12 bg-gradient-to-b from-gray-500 to-transparent"></div>
    </div>
  </section>

  <!-- Stats -->
  <section class="py-24 border-y border-white/10 bg-[#0D0D12]">
    <div class="max-w-4xl mx-auto px-6 grid grid-cols-2 md:grid-cols-4 gap-8">
      <AnimatedCounter client:visible target={9} suffix="+" label="Brevets internationaux" />
      <AnimatedCounter client:visible target={28} suffix=" ans" label="D'expertise" />
      <AnimatedCounter client:visible target={10000} label="Lavages résistés" />
      <AnimatedCounter client:visible target={4} label="Secteurs industriels" />
    </div>
  </section>

  <!-- Technology teaser -->
  <section class="py-24 max-w-7xl mx-auto px-6">
    <div class="text-center mb-16">
      <p class="text-xs uppercase tracking-widest text-[#1A56FF] mb-4">Innovation brevetée</p>
      <h2 class="text-4xl font-bold mb-4" style="font-family: 'Space Grotesk', sans-serif;">
        Polarité Alternée : <span class="text-[#1A56FF]">8× plus puissant</span>
      </h2>
      <p class="text-gray-400 max-w-xl mx-auto">
        Une configuration N-S-N-S en aimants NdFeB miniaturisés crée un champ magnétique multiplicateur.
        Le seul système textile-intégré au monde.
      </p>
    </div>

    <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
      {advantages.map((adv) => (
        <div class="p-5 bg-[#1A1A24] border border-white/10 rounded-xl hover:border-[#1A56FF]/30 transition-all duration-300 group">
          <div class="text-2xl mb-3">{adv.icon}</div>
          <h3 class="text-sm font-semibold text-white mb-1">{adv.title}</h3>
          <p class="text-xs text-gray-500 leading-relaxed">{adv.desc}</p>
        </div>
      ))}
    </div>

    <div class="text-center mt-10">
      <a href="/technologie" class="text-sm text-[#1A56FF] hover:underline">
        Explorer la technologie complète →
      </a>
    </div>
  </section>

  <!-- Sectors -->
  <section class="py-24 bg-[#0D0D12]">
    <div class="max-w-7xl mx-auto px-6">
      <div class="text-center mb-16">
        <p class="text-xs uppercase tracking-widest text-[#C9A84C] mb-4">Marchés cibles</p>
        <h2 class="text-4xl font-bold" style="font-family: 'Space Grotesk', sans-serif;">
          Une technologie, trois secteurs d'excellence
        </h2>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
        {sectors.map((s) => (
          <a
            href={`/secteurs/${s.slug}`}
            class="block p-8 bg-[#1A1A24] border border-white/10 rounded-xl hover:scale-[1.02] transition-all duration-300 group"
            style={`border-top: 2px solid ${s.color}`}
          >
            <div class="text-3xl mb-4">{s.icon}</div>
            <div class="inline-block px-2 py-0.5 rounded text-xs mb-3" style={`background: ${s.color}20; color: ${s.color}`}>
              {s.badge}
            </div>
            <h3 class="text-xl font-bold text-white mb-3">{s.title}</h3>
            <p class="text-sm text-gray-400 leading-relaxed mb-4">{s.desc}</p>
            <span class="text-xs" style={`color: ${s.color}`}>
              En savoir plus →
            </span>
          </a>
        ))}
      </div>
    </div>
  </section>

  <!-- Competitor comparison -->
  <section class="py-24 max-w-5xl mx-auto px-6">
    <div class="text-center mb-16">
      <p class="text-xs uppercase tracking-widest text-[#1A56FF] mb-4">Analyse concurrentielle</p>
      <h2 class="text-4xl font-bold" style="font-family: 'Space Grotesk', sans-serif;">
        Pourquoi SYSTEMMAG ?
      </h2>
    </div>

    <div class="overflow-x-auto">
      <table class="w-full text-sm">
        <thead>
          <tr class="border-b border-white/10">
            <th class="text-left py-4 px-4 text-gray-500 font-normal text-xs uppercase tracking-widest">Critère</th>
            {competitors.map(c => (
              <th class={`py-4 px-4 text-center text-xs uppercase tracking-widest ${c.highlight ? 'text-[#1A56FF]' : 'text-gray-500'}`}>
                {c.name}
              </th>
            ))}
          </tr>
        </thead>
        <tbody>
          {[
            { label: 'Intégration textile', key: 'integration' },
            { label: 'Silencieux', key: 'silence' },
            { label: 'Une main', key: 'oneHand' },
            { label: 'Made in France', key: 'madeFR' },
            { label: 'Résistance terrain', key: 'terrain' },
            { label: 'Portefeuille brevets', key: 'patents' },
            { label: 'Agrément Défense', key: 'defense' },
          ].map(({ label, key }) => (
            <tr class="border-b border-white/5 hover:bg-white/2 transition-colors">
              <td class="py-3 px-4 text-gray-400 text-xs">{label}</td>
              {competitors.map(c => (
                <td class="py-3 px-4 text-center">
                  {(c as any)[key]
                    ? <span class="text-green-400 text-base">✓</span>
                    : <span class="text-gray-600 text-base">✗</span>}
                </td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  </section>

  <!-- Founder story -->
  <section class="py-24 bg-[#0D0D12]">
    <div class="max-w-3xl mx-auto px-6 text-center">
      <p class="text-xs uppercase tracking-widest text-[#C9A84C] mb-8">L'origine</p>
      <blockquote class="text-xl md:text-2xl text-gray-300 italic leading-relaxed mb-8">
        "L'idée est née de la confrontation du jeune papa face à une 'envie pressante' de son fils :
        le temps de défaire les crochets de la salopette, il était trop tard."
      </blockquote>
      <cite class="text-sm text-gray-500 not-italic">— Éric SITBON, Fondateur SYSTEMMAG, 1997</cite>
      <div class="mt-8">
        <a href="/histoire" class="text-sm text-[#1A56FF] hover:underline">
          Découvrir notre histoire complète →
        </a>
      </div>
    </div>
  </section>

  <!-- CTA finale -->
  <section class="py-24">
    <div class="max-w-3xl mx-auto px-6 text-center">
      <h2 class="text-4xl font-bold mb-6" style="font-family: 'Space Grotesk', sans-serif;">
        Prêt à tester la technologie ?
      </h2>
      <p class="text-gray-400 mb-10">
        Échantillons gratuits pour industriels, créateurs et équipementiers qualifiés.
      </p>
      <div class="flex flex-col sm:flex-row gap-4 justify-center">
        <a href="/contact" class="px-8 py-4 bg-[#1A56FF] text-white font-semibold rounded hover:bg-[#1A56FF]/90 transition-all">
          Demander des échantillons
        </a>
        <a href="tel:+33145089141" class="px-8 py-4 border border-white/20 text-white rounded hover:border-white/40 transition-all">
          +33 1 45 08 91 41
        </a>
      </div>
    </div>
  </section>
</BaseLayout>
```

**Step 2: Commit**

```bash
git add -A && git commit -m "feat: add homepage with hero, stats, sectors, comparison table"
```

---

## Task 8: Technology Page

**Files:**
- Create: `systemmag-astro/src/pages/technologie.astro`

**Step 1: Create `src/pages/technologie.astro`**

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro'
---

<BaseLayout
  title="Technologie — SYSTEMMAG® Polarité Alternée NdFeB"
  description="De la physique quantique appliquée au tissu. Aimants NdFeB, Polarité Alternée brevetée 2002, 8-10x plus puissant. 9 brevets INPI + internationaux."
>
  <!-- Hero -->
  <section class="pt-32 pb-16 max-w-5xl mx-auto px-6">
    <div class="text-center">
      <p class="text-xs uppercase tracking-widest text-[#1A56FF] mb-4">Innovation brevetée depuis 1998</p>
      <h1 class="text-5xl font-bold mb-6" style="font-family: 'Space Grotesk', sans-serif;">
        De la physique quantique<br />
        <span class="text-[#1A56FF]">appliquée au tissu</span>
      </h1>
      <p class="text-gray-400 max-w-2xl mx-auto text-lg leading-relaxed">
        SYSTEMMAG exploite les propriétés quantiques des terres rares pour créer la fermeture textile la plus avancée au monde.
      </p>
    </div>
  </section>

  <!-- NdFeB Specs grid -->
  <section class="py-16 bg-[#0D0D12]">
    <div class="max-w-5xl mx-auto px-6">
      <h2 class="text-2xl font-bold text-center mb-12" style="font-family: 'Space Grotesk', sans-serif;">
        Spécifications techniques NdFeB
      </h2>
      <div class="grid grid-cols-2 md:grid-cols-3 gap-4">
        {[
          { label: 'Grades disponibles', value: 'N38 · N40 · N45 · N48 · N52' },
          { label: 'Résistance thermique', value: 'Jusqu\'à 90°C (200°C sur demande)' },
          { label: 'Cycles de lavage', value: '10 000 lavages validés' },
          { label: 'Revêtement', value: 'Anti-corrosion brouillard salin' },
          { label: 'Certifications', value: 'OEKO-TEX Standard 100 · REACH' },
          { label: 'Puissance', value: '5 à 10× aimant traditionnel' },
        ].map(spec => (
          <div class="p-6 bg-[#1A1A24] border border-white/10 rounded-xl">
            <p class="text-xs uppercase tracking-widest text-gray-500 mb-2">{spec.label}</p>
            <p class="text-white font-semibold text-sm">{spec.value}</p>
          </div>
        ))}
      </div>
    </div>
  </section>

  <!-- Alternating Polarity -->
  <section class="py-24 max-w-5xl mx-auto px-6">
    <div class="grid md:grid-cols-2 gap-16 items-center">
      <div>
        <p class="text-xs uppercase tracking-widest text-[#C9A84C] mb-4">Brevet 2002</p>
        <h2 class="text-3xl font-bold mb-6" style="font-family: 'Space Grotesk', sans-serif;">
          Le système de Polarité Alternée
        </h2>
        <p class="text-gray-400 leading-relaxed mb-6">
          La configuration N-S-N-S des Blocs Aimants miniaturisés dans un liant élastomère crée un champ magnétique de renforcement mutuel,
          multipliant la force de retenue par 8 à 10× par rapport à un aimant simple de même volume.
        </p>
        <ol class="space-y-4">
          {[
            { num: '01', title: 'Aimants NdFeB miniaturisés', desc: 'Séquence N-S-N-S dans liant élastomère souple' },
            { num: '02', title: 'Intégration textile', desc: 'Le liant élastomère permet le drapé et la couture directe' },
            { num: '03', title: 'Multiplication de puissance', desc: 'Renforcement magnétique mutuel = 8-10× la force' },
          ].map(step => (
            <li class="flex gap-4">
              <span class="text-3xl font-bold text-[#1A56FF]/30">{step.num}</span>
              <div>
                <h3 class="text-sm font-semibold text-white mb-1">{step.title}</h3>
                <p class="text-xs text-gray-500">{step.desc}</p>
              </div>
            </li>
          ))}
        </ol>
      </div>
      <div class="bg-[#1A1A24] border border-white/10 rounded-2xl p-8 text-center">
        <div class="text-7xl font-bold text-[#1A56FF] mb-4" style="font-family: 'Space Grotesk', sans-serif;">8×</div>
        <p class="text-white font-semibold mb-2">Plus puissant</p>
        <p class="text-xs text-gray-500">qu'un aimant simple de même volume</p>
        <div class="mt-8 flex justify-center gap-2">
          {['N', 'S', 'N', 'S', 'N', 'S'].map((pole) => (
            <div class={`w-8 h-8 rounded flex items-center justify-center text-xs font-bold ${pole === 'N' ? 'bg-[#1A56FF] text-white' : 'bg-[#C9A84C] text-black'}`}>
              {pole}
            </div>
          ))}
        </div>
        <p class="text-xs text-gray-500 mt-3">Configuration Polarité Alternée</p>
      </div>
    </div>
  </section>

  <!-- Patent timeline -->
  <section class="py-24 bg-[#0D0D12]">
    <div class="max-w-3xl mx-auto px-6">
      <h2 class="text-3xl font-bold text-center mb-16" style="font-family: 'Space Grotesk', sans-serif;">
        Portefeuille de brevets
      </h2>
      <div class="space-y-6">
        {[
          { year: '1998', title: 'Brevet fondateur', desc: 'Dispositif magnétique pour vêtements, chaussures et accessoires', highlight: true },
          { year: '2002', title: 'Polarité Alternée', desc: 'Système NdFeB alternance N-S pour multiplication de puissance', highlight: true },
          { year: '2006', title: 'Zip magnétique', desc: 'Fermeture prête à piquer remplaçant zip et velcro' },
          { year: '2011', title: 'Système SCE', desc: 'Energy Catalyst System — exosquelettes et vestes de combat' },
          { year: '2014', title: 'Agrément US Army', desc: 'Certification fournisseur Armée des États-Unis', highlight: true },
          { year: '2015+', title: '5 brevets spécialisés', desc: 'Chaussures, soutien-gorge, tabliers radiol. (FR+USA), casques (USA), générateur magnétique' },
        ].map((item) => (
          <div class={`flex gap-6 ${item.highlight ? 'opacity-100' : 'opacity-70'}`}>
            <div class="flex flex-col items-center">
              <div class={`w-3 h-3 rounded-full mt-1 ${item.highlight ? 'bg-[#C9A84C]' : 'bg-gray-600'}`}></div>
              <div class="w-px flex-1 bg-white/10 mt-2"></div>
            </div>
            <div class="pb-6">
              <span class={`text-xs font-bold ${item.highlight ? 'text-[#C9A84C]' : 'text-gray-500'}`}>{item.year}</span>
              <h3 class="text-white font-semibold mt-1">{item.title}</h3>
              <p class="text-xs text-gray-500 mt-1">{item.desc}</p>
            </div>
          </div>
        ))}
      </div>
    </div>
  </section>

  <!-- CTA -->
  <section class="py-16 text-center">
    <p class="text-gray-400 mb-6">Besoin d'une intégration technique sur mesure ?</p>
    <a href="/contact" class="px-8 py-4 bg-[#1A56FF] text-white font-semibold rounded hover:bg-[#1A56FF]/90 transition-all">
      Contacter le bureau d'études
    </a>
  </section>
</BaseLayout>
```

**Step 2: Commit**

```bash
git add -A && git commit -m "feat: add technology page with NdFeB specs and patent timeline"
```

---

## Task 9: Sector Pages (Defense, Medical, Fashion)

**Files:**
- Create: `systemmag-astro/src/pages/secteurs/defense.astro`
- Create: `systemmag-astro/src/pages/secteurs/medical.astro`
- Create: `systemmag-astro/src/pages/secteurs/mode.astro`

**Step 1: Create `src/pages/secteurs/defense.astro`**

```astro
---
import BaseLayout from '../../layouts/BaseLayout.astro'

const advantages = [
  { icon: '🔇', title: 'Silencieux', desc: 'Zéro bruit en opération nocturne' },
  { icon: '🧤', title: 'Une main gantée', desc: 'Fermeture instantanée avec gants tactiques' },
  { icon: '🏜️', title: 'Résistant terrain', desc: 'Sable, boue, brouillard salin — imperméable' },
  { icon: '🔄', title: '10 000 lavages', desc: 'Compatible nettoyage industriel armée' },
  { icon: '⚡', title: 'Grade N52', desc: 'Puissance magnétique maximale disponible' },
  { icon: '🇺🇸', title: 'US Army 2014', desc: 'Fournisseur agréé Armée des États-Unis' },
]
---

<BaseLayout
  title="Défense & Militaire — SYSTEMMAG® Fermeture Tactique"
  description="Fermeture magnétique silencieuse pour équipements militaires. Agréé fournisseur US Army 2014. N52 grade, résistant terrain, une main gantée."
>
  <section class="pt-32 pb-16 max-w-5xl mx-auto px-6">
    <div class="inline-block px-3 py-1 bg-[#1A56FF]/10 border border-[#1A56FF]/30 rounded text-[#1A56FF] text-xs uppercase tracking-widest mb-6">
      Agréé fournisseurs Armée US · 2014
    </div>
    <h1 class="text-5xl font-bold mb-6" style="font-family: 'Space Grotesk', sans-serif;">
      En milieu opérationnel,<br />
      <span class="text-[#1A56FF]">une fermeture qui fait du bruit<br />peut coûter une vie.</span>
    </h1>
    <p class="text-gray-400 text-lg max-w-2xl leading-relaxed">
      SYSTEMMAG est la seule fermeture textile silencieuse, résistante au terrain et opérable d'une main gantée,
      approuvée par l'US Army.
    </p>
  </section>

  <!-- Velcro comparison -->
  <section class="py-16 bg-[#0D0D12]">
    <div class="max-w-4xl mx-auto px-6">
      <h2 class="text-2xl font-bold text-center mb-12" style="font-family: 'Space Grotesk', sans-serif;">
        Velcro vs SYSTEMMAG en conditions opérationnelles
      </h2>
      <div class="grid md:grid-cols-2 gap-6">
        <div class="p-6 bg-red-500/5 border border-red-500/20 rounded-xl">
          <h3 class="text-red-400 font-semibold mb-4">⚠️ Velcro — Risques constatés</h3>
          <ul class="space-y-2 text-sm text-gray-400">
            <li>• Bruit détectable à plusieurs mètres en opération nocturne</li>
            <li>• 60-80% de perte de rétention avec contamination sable/boue</li>
            <li>• Impossible avec gants épais</li>
            <li>• Dégradation rapide en environnement humide</li>
          </ul>
        </div>
        <div class="p-6 bg-[#1A56FF]/5 border border-[#1A56FF]/20 rounded-xl">
          <h3 class="text-[#1A56FF] font-semibold mb-4">✓ SYSTEMMAG — Solution</h3>
          <ul class="space-y-2 text-sm text-gray-400">
            <li>• Fermeture magnétique silencieuse (0 dB)</li>
            <li>• Imperméable sable, boue, sel — 100% fonctionnel</li>
            <li>• Opérable d'une main gantée instantanément</li>
            <li>• Revêtement anti-corrosion brouillard salin</li>
          </ul>
        </div>
      </div>
    </div>
  </section>

  <!-- Advantages grid -->
  <section class="py-24 max-w-5xl mx-auto px-6">
    <h2 class="text-3xl font-bold text-center mb-12" style="font-family: 'Space Grotesk', sans-serif;">
      6 avantages tactiques
    </h2>
    <div class="grid grid-cols-2 md:grid-cols-3 gap-4">
      {advantages.map(adv => (
        <div class="p-6 bg-[#1A1A24] border border-[#1A56FF]/20 rounded-xl">
          <div class="text-2xl mb-3">{adv.icon}</div>
          <h3 class="text-sm font-semibold text-white mb-1">{adv.title}</h3>
          <p class="text-xs text-gray-500">{adv.desc}</p>
        </div>
      ))}
    </div>
  </section>

  <!-- Applications -->
  <section class="py-16 bg-[#0D0D12]">
    <div class="max-w-5xl mx-auto px-6">
      <h2 class="text-2xl font-bold text-center mb-10" style="font-family: 'Space Grotesk', sans-serif;">Applications militaires</h2>
      <div class="grid grid-cols-2 md:grid-cols-3 gap-4">
        {['Gilets tactiques & gilets pare-balles', 'Uniformes de combat', 'Sacs & équipements transport', 'Casques (brevet USA déposé)', 'Tabliers de protection (FR+USA)', 'Exosquelettes biomagnétiques'].map(app => (
          <div class="p-4 bg-[#1A1A24] border border-white/10 rounded-lg text-sm text-gray-300">{app}</div>
        ))}
      </div>
    </div>
  </section>

  <section class="py-16 text-center">
    <a href="/contact" class="px-8 py-4 bg-[#1A56FF] text-white font-semibold rounded hover:bg-[#1A56FF]/90 transition-all">
      Demander un devis Défense
    </a>
  </section>
</BaseLayout>
```

**Step 2: Create `src/pages/secteurs/medical.astro`**

```astro
---
import BaseLayout from '../../layouts/BaseLayout.astro'
---

<BaseLayout
  title="Médical & Santé — SYSTEMMAG® Fermeture Adaptée"
  description="Fermeture magnétique pour vêtements adaptés. Une main pour hémiplégiques, Parkinson, personnes âgées. OEKO-TEX, REACH, compatible pacemaker étudié."
>
  <section class="pt-32 pb-16 max-w-5xl mx-auto px-6">
    <div class="inline-block px-3 py-1 bg-emerald-500/10 border border-emerald-500/30 rounded text-emerald-400 text-xs uppercase tracking-widest mb-6">
      OEKO-TEX · REACH · Compatible Pacemaker Étudié
    </div>
    <h1 class="text-5xl font-bold mb-6" style="font-family: 'Space Grotesk', sans-serif;">
      SYSTEMMAG<br />
      <span class="text-emerald-400">redonne l'indépendance</span>
    </h1>
    <p class="text-gray-400 text-lg max-w-2xl leading-relaxed">
      Pour les personnes à mobilité réduite, hémiplégiques, Parkinsoniens, personnes âgées et en rééducation —
      s'habiller avec élégance et autonomie, sans aide extérieure.
    </p>
  </section>

  <!-- Benefits -->
  <section class="py-16 bg-[#0D0D12]">
    <div class="max-w-5xl mx-auto px-6 grid grid-cols-2 md:grid-cols-3 gap-4">
      {[
        { icon: '✋', title: 'Une main', desc: 'Pour hémiplégiques et amputés membres supérieurs' },
        { icon: '🎯', title: 'Sans alignement', desc: 'L\'aimant s\'oriente seul — idéal tremblements Parkinson' },
        { icon: '🌿', title: 'Biocompatible', desc: 'OEKO-TEX + REACH — absence de substances nocives' },
        { icon: '🏥', title: 'Décontamination', desc: 'Lavage hospitalier 90°C compatible' },
        { icon: '❤️', title: 'Pacemaker', desc: 'Étude d\'incidence réalisée — sécurité étudiée' },
        { icon: '🤫', title: 'Silencieux', desc: 'Pas de bruit Velcro — idéal troubles sensoriels' },
      ].map(b => (
        <div class="p-6 bg-[#1A1A24] border border-emerald-500/20 rounded-xl">
          <div class="text-2xl mb-3">{b.icon}</div>
          <h3 class="text-sm font-semibold text-white mb-1">{b.title}</h3>
          <p class="text-xs text-gray-500">{b.desc}</p>
        </div>
      ))}
    </div>
  </section>

  <!-- Target populations -->
  <section class="py-24 max-w-4xl mx-auto px-6">
    <h2 class="text-2xl font-bold text-center mb-10" style="font-family: 'Space Grotesk', sans-serif;">Populations concernées</h2>
    <div class="grid grid-cols-2 md:grid-cols-3 gap-3">
      {['Maladie de Parkinson', 'Hémiplégie / AVC', 'Sclérose en plaques', 'Amputés membres supérieurs', 'Personnes âgées (arthrose, rhumatisme)', 'Enfants handicapés', 'Rééducation post-opératoire'].map(pop => (
        <div class="p-3 bg-[#1A1A24] border border-white/10 rounded-lg text-xs text-gray-300 flex items-center gap-2">
          <span class="text-emerald-400">◆</span> {pop}
        </div>
      ))}
    </div>
  </section>

  <!-- Applications -->
  <section class="py-16 bg-[#0D0D12]">
    <div class="max-w-4xl mx-auto px-6">
      <h2 class="text-2xl font-bold text-center mb-10" style="font-family: 'Space Grotesk', sans-serif;">Applications médicales</h2>
      <div class="grid grid-cols-2 md:grid-cols-3 gap-4">
        {['Orthèses et attelles', 'Tabliers de radioprotection (brevet FR+USA)', 'Vêtements de rééducation', 'Matériel kinésithérapie (testé École de Berck)', 'Exosquelettes biomagnétiques', 'Accessoires soins quotidiens'].map(app => (
          <div class="p-4 bg-[#1A1A24] border border-white/10 rounded-lg text-sm text-gray-300">{app}</div>
        ))}
      </div>
    </div>
  </section>

  <section class="py-16 text-center">
    <a href="/contact" class="px-8 py-4 bg-emerald-500 text-white font-semibold rounded hover:bg-emerald-600 transition-all">
      Demander un échantillon Médical
    </a>
  </section>
</BaseLayout>
```

**Step 3: Create `src/pages/secteurs/mode.astro`**

```astro
---
import BaseLayout from '../../layouts/BaseLayout.astro'
---

<BaseLayout
  title="Mode & Prêt-à-Porter — SYSTEMMAG® Fermeture Invisible"
  description="La fermeture que les maisons de couture attendaient. Invisible, intégrée dans le tissu, prête à piquer. Made in France Paris 11e. OEM pour créateurs."
>
  <section class="pt-32 pb-16 max-w-5xl mx-auto px-6">
    <div class="inline-block px-3 py-1 bg-[#C9A84C]/10 border border-[#C9A84C]/30 rounded text-[#C9A84C] text-xs uppercase tracking-widest mb-6">
      Mode Paris · Intégration OEM
    </div>
    <h1 class="text-5xl font-bold mb-6" style="font-family: 'Space Grotesk', sans-serif;">
      La fermeture que les maisons<br />
      <span class="text-[#C9A84C]">de couture attendaient.</span>
    </h1>
    <p class="text-gray-400 text-lg max-w-2xl leading-relaxed">
      La surface du vêtement reste vierge. Aucune couture visible, aucun mécanisme apparent.
      La fermeture magnétique SYSTEMMAG s'intègre dans le tissu lui-même.
    </p>
  </section>

  <!-- Value pillars -->
  <section class="py-16 bg-[#0D0D12]">
    <div class="max-w-4xl mx-auto px-6 grid grid-cols-1 md:grid-cols-3 gap-6">
      {[
        { title: 'Invisible', desc: 'Intégrée dans le tissu, la surface reste parfaitement vierge. Silhouettes minimalistes possibles.', icon: '👁️' },
        { title: 'Prêt à piquer', desc: 'Livré comme un composant textile standard. S\'intègre dans les flux atelier existants sans équipement spécial.', icon: '🧵' },
        { title: 'Made in France', desc: 'Paris 11e. Qualité artisanale française avec bureau d\'études disponible pour développement sur mesure.', icon: '🇫🇷' },
      ].map(p => (
        <div class="p-8 bg-[#1A1A24] border border-[#C9A84C]/20 rounded-xl text-center">
          <div class="text-4xl mb-4">{p.icon}</div>
          <h3 class="text-lg font-bold text-white mb-3">{p.title}</h3>
          <p class="text-sm text-gray-400 leading-relaxed">{p.desc}</p>
        </div>
      ))}
    </div>
  </section>

  <!-- Inclusive fashion -->
  <section class="py-24 max-w-4xl mx-auto px-6">
    <div class="bg-[#1A1A24] border border-[#C9A84C]/20 rounded-2xl p-10">
      <h2 class="text-2xl font-bold mb-4" style="font-family: 'Space Grotesk', sans-serif;">
        Mode inclusive & adaptive fashion
      </h2>
      <p class="text-gray-400 leading-relaxed">
        SYSTEMMAG permet aux personnes à mobilité réduite de s'habiller avec élégance et autonomie,
        sans Velcro disgracieux ni boutons impossibles à manipuler. Une opportunité de marché grandissante
        pour les marques de prêt-à-porter accessible.
      </p>
    </div>
  </section>

  <!-- Applications -->
  <section class="py-16 bg-[#0D0D12]">
    <div class="max-w-4xl mx-auto px-6">
      <h2 class="text-2xl font-bold text-center mb-10" style="font-family: 'Space Grotesk', sans-serif;">Applications mode</h2>
      <div class="grid grid-cols-2 md:grid-cols-3 gap-4">
        {['Chaussures (brevet déposé)', 'Soutien-gorge (brevet)', 'Vestes et manteaux', 'Accessoires luxe', 'Sportswear & outdoor', 'Vêtements adaptive fashion'].map(app => (
          <div class="p-4 bg-[#1A1A24] border border-white/10 rounded-lg text-sm text-gray-300">{app}</div>
        ))}
      </div>
    </div>
  </section>

  <section class="py-16 text-center">
    <p class="text-gray-400 mb-6">Bureau d'études disponible pour prototypage, test et développement sur mesure.</p>
    <a href="/contact" class="px-8 py-4 border border-[#C9A84C] text-[#C9A84C] hover:bg-[#C9A84C] hover:text-black font-semibold rounded transition-all">
      Contacter le bureau d'études Mode
    </a>
  </section>
</BaseLayout>
```

**Step 4: Commit**

```bash
git add -A && git commit -m "feat: add defense, medical, fashion sector pages"
```

---

## Task 10: Products Page (Sanity dynamic)

**Files:**
- Create: `systemmag-astro/src/pages/produits/index.astro`

**Step 1: Create `src/pages/produits/index.astro`**

```astro
---
import BaseLayout from '../../layouts/BaseLayout.astro'
import { getProducts } from '../../lib/sanity'

const products = await getProducts()

// Group by family
const families: Record<string, typeof products> = {}
for (const p of products) {
  if (!families[p.family]) families[p.family] = []
  families[p.family].push(p)
}

const familyLabels: Record<string, { label: string; desc: string; color: string }> = {
  BA: { label: 'Blocs & Bandes Aimant', desc: 'Base composant pour intégrations OEM', color: '#C9A84C' },
  ZipC: { label: 'Zip C — Centré Standard', desc: 'Prêt à piquer, vêtements et accessoires', color: '#1A56FF' },
  ZipG: { label: 'Zip G — Plat Centré', desc: 'Montage à plat sur uniformes', color: '#1A56FF' },
  ZipCE: { label: 'Zip CE/GE — Haute Puissance', desc: 'Défense et médical lourd', color: '#1A56FF' },
  ZipCD: { label: 'Zip CD/GD — Double Aimant', desc: 'Sécurité renforcée double point', color: '#1A56FF' },
  Fourreaux: { label: 'Fourreaux', desc: 'Intégration bespoke sur mesure', color: '#8A8A9A' },
}
---

<BaseLayout
  title="Produits — Catalogue SYSTEMMAG® Fermetures Magnétiques"
  description="Catalogue complet SYSTEMMAG : Blocs Aimants BA, Zip C/G/CE/GE/CD/GD, Fourreaux FD/FF. Grades N38 à N52. Fiches techniques PDF téléchargeables."
>
  <section class="pt-32 pb-16 max-w-7xl mx-auto px-6">
    <div class="text-center mb-16">
      <p class="text-xs uppercase tracking-widest text-[#1A56FF] mb-4">Catalogue complet</p>
      <h1 class="text-5xl font-bold mb-6" style="font-family: 'Space Grotesk', sans-serif;">
        Nos produits
      </h1>
      <p class="text-gray-400 max-w-xl mx-auto">
        6 familles de fermetures magnétiques textiles. Grades N38 à N52. Prêts à piquer.
      </p>
    </div>

    {products.length === 0 ? (
      <!-- Fallback: show static catalog when no Sanity data -->
      <div class="space-y-12">
        {Object.entries(familyLabels).map(([key, fam]) => (
          <div>
            <div class="mb-6 pb-4 border-b border-white/10">
              <h2 class="text-xl font-bold text-white" style="font-family: 'Space Grotesk', sans-serif;">{fam.label}</h2>
              <p class="text-sm text-gray-500 mt-1">{fam.desc}</p>
            </div>
            <p class="text-sm text-gray-500 italic">
              Connectez Sanity Studio pour afficher le catalogue complet avec fiches techniques.
            </p>
          </div>
        ))}
      </div>
    ) : (
      <div class="space-y-16">
        {Object.entries(families).map(([family, items]) => {
          const meta = familyLabels[family] || { label: family, desc: '', color: '#1A56FF' }
          return (
            <div>
              <div class="mb-8 pb-4 border-b border-white/10 flex items-baseline gap-4">
                <h2 class="text-2xl font-bold text-white" style="font-family: 'Space Grotesk', sans-serif;">
                  {meta.label}
                </h2>
                <p class="text-sm text-gray-500">{meta.desc}</p>
              </div>
              <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                {items.map((p: any) => (
                  <div class="p-6 bg-[#1A1A24] border border-white/10 rounded-xl hover:border-[#1A56FF]/30 transition-all">
                    <div class="flex justify-between items-start mb-3">
                      <h3 class="text-sm font-semibold text-white">{p.title}</h3>
                      {p.sku && <span class="text-xs text-gray-500 font-mono">{p.sku}</span>}
                    </div>
                    {p.description && <p class="text-xs text-gray-400 leading-relaxed mb-3">{p.description}</p>}
                    <div class="flex flex-wrap gap-2 mt-3">
                      {p.width && (
                        <span class="px-2 py-0.5 bg-white/5 border border-white/10 rounded text-xs text-gray-400">
                          {p.width}
                        </span>
                      )}
                      {p.magnetGrade && (
                        <span class="px-2 py-0.5 bg-[#1A56FF]/10 border border-[#1A56FF]/20 rounded text-xs text-[#1A56FF]">
                          {p.magnetGrade}
                        </span>
                      )}
                    </div>
                    {p.usageNote && (
                      <p class="text-xs text-[#C9A84C] mt-3 italic">{p.usageNote}</p>
                    )}
                    {p.pdfUrl && (
                      <a
                        href={p.pdfUrl}
                        target="_blank"
                        rel="noopener"
                        class="inline-block mt-4 text-xs text-[#1A56FF] hover:underline"
                      >
                        📄 Fiche technique PDF
                      </a>
                    )}
                  </div>
                ))}
              </div>
            </div>
          )
        })}
      </div>
    )}

    <!-- Custom solution CTA -->
    <div class="mt-20 p-8 bg-[#1A1A24] border border-[#1A56FF]/20 rounded-2xl text-center">
      <h2 class="text-2xl font-bold mb-4" style="font-family: 'Space Grotesk', sans-serif;">
        Solution sur mesure ?
      </h2>
      <p class="text-gray-400 mb-6">Notre bureau d'études conçoit des systèmes de fermeture adaptés à vos contraintes spécifiques.</p>
      <a href="/contact" class="px-6 py-3 bg-[#1A56FF] text-white text-sm font-semibold rounded hover:bg-[#1A56FF]/90 transition-all">
        Parler à nos ingénieurs
      </a>
    </div>
  </section>
</BaseLayout>
```

**Step 2: Commit**

```bash
git add -A && git commit -m "feat: add products page with Sanity dynamic data + static fallback"
```

---

## Task 11: Articles Pages (Sanity dynamic)

**Files:**
- Create: `systemmag-astro/src/pages/articles/index.astro`
- Create: `systemmag-astro/src/pages/articles/[slug].astro`

**Step 1: Create `src/pages/articles/index.astro`**

```astro
---
import BaseLayout from '../../layouts/BaseLayout.astro'
import { getArticles } from '../../lib/sanity'

const articles = await getArticles()

const sectorColors: Record<string, string> = {
  defense: '#1A56FF',
  medical: '#10B981',
  fashion: '#C9A84C',
  industry: '#8A8A9A',
  technology: '#A855F7',
  general: '#6B7280',
}

const sectorLabels: Record<string, string> = {
  defense: 'Défense',
  medical: 'Médical',
  fashion: 'Mode',
  industry: 'Industrie',
  technology: 'Technologie',
  general: 'Général',
}

function formatDate(dateStr: string) {
  return new Date(dateStr).toLocaleDateString('fr-FR', { year: 'numeric', month: 'long', day: 'numeric' })
}
---

<BaseLayout
  title="Articles — SYSTEMMAG® Ressources et Analyses"
  description="Articles techniques, analyses sectorielles et ressources sur les fermetures magnétiques textiles : défense, médical, mode, industrie."
>
  <section class="pt-32 pb-24 max-w-5xl mx-auto px-6">
    <div class="text-center mb-16">
      <p class="text-xs uppercase tracking-widest text-[#1A56FF] mb-4">Ressources</p>
      <h1 class="text-5xl font-bold mb-6" style="font-family: 'Space Grotesk', sans-serif;">Articles & Analyses</h1>
      <p class="text-gray-400 max-w-xl mx-auto">
        Expertise SYSTEMMAG sur les fermetures magnétiques textiles, applications sectorielles et innovations.
      </p>
    </div>

    {articles.length === 0 ? (
      <div class="text-center py-20">
        <p class="text-gray-500 mb-4">Aucun article publié pour le moment.</p>
        <p class="text-xs text-gray-600">Ajoutez des articles via Sanity Studio (/studio)</p>
      </div>
    ) : (
      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        {articles.map((a: any) => (
          <a href={`/articles/${a.slug}`} class="block p-6 bg-[#1A1A24] border border-white/10 rounded-xl hover:border-[#1A56FF]/30 transition-all group">
            {a.sector && (
              <span
                class="inline-block px-2 py-0.5 rounded text-xs mb-3"
                style={`background: ${sectorColors[a.sector] || '#6B7280'}20; color: ${sectorColors[a.sector] || '#6B7280'}`}
              >
                {sectorLabels[a.sector] || a.sector}
              </span>
            )}
            <h2 class="text-lg font-semibold text-white mb-2 group-hover:text-[#1A56FF] transition-colors">
              {a.title}
            </h2>
            {a.excerpt && <p class="text-sm text-gray-400 leading-relaxed mb-4">{a.excerpt}</p>}
            {a.publishedAt && (
              <time class="text-xs text-gray-600">{formatDate(a.publishedAt)}</time>
            )}
          </a>
        ))}
      </div>
    )}
  </section>
</BaseLayout>
```

**Step 2: Create `src/pages/articles/[slug].astro`**

```astro
---
import BaseLayout from '../../layouts/BaseLayout.astro'
import { getArticleBySlug, sanityClient } from '../../lib/sanity'
import { PortableText } from '@portabletext/react'

export async function getStaticPaths() {
  const slugs = await sanityClient.fetch(`
    *[_type == "article" && defined(slug.current)] { "slug": slug.current }
  `)
  return slugs.map((s: { slug: string }) => ({ params: { slug: s.slug } }))
}

const { slug } = Astro.params
const article = await getArticleBySlug(slug!)

if (!article) {
  return Astro.redirect('/articles')
}

function formatDate(dateStr: string) {
  return new Date(dateStr).toLocaleDateString('fr-FR', { year: 'numeric', month: 'long', day: 'numeric' })
}
---

<BaseLayout
  title={article.seoTitle || article.title + ' — SYSTEMMAG®'}
  description={article.seoDescription || article.excerpt || ''}
>
  <article class="pt-32 pb-24 max-w-3xl mx-auto px-6">
    <a href="/articles" class="inline-flex items-center gap-2 text-xs text-gray-500 hover:text-white mb-8 transition-colors">
      ← Tous les articles
    </a>

    <header class="mb-12">
      <h1 class="text-4xl font-bold mb-4" style="font-family: 'Space Grotesk', sans-serif;">
        {article.title}
      </h1>
      {article.publishedAt && (
        <time class="text-sm text-gray-500">{formatDate(article.publishedAt)}</time>
      )}
      {article.excerpt && (
        <p class="text-lg text-gray-400 mt-4 leading-relaxed border-l-2 border-[#1A56FF] pl-4">
          {article.excerpt}
        </p>
      )}
    </header>

    <div class="prose prose-invert prose-sm max-w-none
      prose-headings:font-bold prose-headings:text-white
      prose-p:text-gray-400 prose-p:leading-relaxed
      prose-a:text-[#1A56FF] prose-a:no-underline hover:prose-a:underline
      prose-strong:text-white prose-code:text-[#C9A84C]
    ">
      {article.body && <PortableText value={article.body} client:load />}
    </div>

    <div class="mt-16 pt-8 border-t border-white/10 text-center">
      <p class="text-gray-500 mb-4 text-sm">Intéressé par nos technologies ?</p>
      <a href="/contact" class="px-6 py-3 bg-[#1A56FF] text-white text-sm font-semibold rounded hover:bg-[#1A56FF]/90 transition-all">
        Demander un échantillon
      </a>
    </div>
  </article>
</BaseLayout>
```

**Step 3: Install PortableText renderer**

```bash
npm install @portabletext/react
```

**Step 4: Commit**

```bash
git add -A && git commit -m "feat: add articles list and article detail pages with Sanity"
```

---

## Task 12: Histoire + Contact Pages

**Files:**
- Create: `systemmag-astro/src/pages/histoire.astro`
- Create: `systemmag-astro/src/pages/contact.astro`

**Step 1: Create `src/pages/histoire.astro`**

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro'

const timeline = [
  { year: '1997', title: 'L\'idée naît', desc: 'Éric SITBON, jeune papa confronté aux salopettes de son fils lors d\'une urgence — l\'idée de la fermeture magnétique invisible est née.', highlight: true },
  { year: '1998', title: 'Premier brevet', desc: 'Dépôt du brevet fondateur : dispositif magnétique pour vêtements, chaussures et accessoires.' },
  { year: '1999', title: 'Prix Ministère de la Recherche', desc: 'Lauréat du concours national "Projet Émergent" — validation officielle de l\'innovation.' },
  { year: '2000', title: 'Création SALOMÉ SARL', desc: 'Éric et Agnès SITBON créent la société. Prix "Création Développement" au concours national.' },
  { year: '2001', title: 'Champion Île-de-France', desc: 'Lauréat du concours régional d\'entreprise innovante avec le soutien de BRED / Banque Populaire / SPEF.' },
  { year: '2002', title: 'Brevet Polarité Alternée', desc: 'Le brevet central : configuration N-S-N-S multipliant la puissance magnétique par 8-10×.', highlight: true },
  { year: '2003', title: 'Outillage industriel', desc: 'Développement des outils de production en série. Passage du prototype à la fabrication industrielle.' },
  { year: '2005', title: 'MODAMONT', desc: 'Première présentation au salon professionnel MODAMONT — entrée sur le marché textile.' },
  { year: '2006', title: 'Brevet Zip magnétique', desc: 'Third patent : fermeture zip magnétique prête à piquer. Transformation en SAS avec investissement Île-de-France Développement.' },
  { year: '2011', title: 'Brevet SCE', desc: 'Energy Catalyst System : brevets pour exosquelettes, gilets de combat, équipements sportifs extrêmes.', highlight: true },
  { year: '2012', title: 'Certification REACH', desc: 'Conformité européenne REACH pour tous composants et processus. Sortie de l\'investisseur.' },
  { year: '2013', title: 'Revêtement 10 000 lavages', desc: 'Développement du revêtement anti-corrosion résistant 10 000 cycles. Grades N38/N40/N45/N48.' },
  { year: '2014', title: 'Agréé US Army', desc: 'Approbation fournisseur Armée des États-Unis. Grade N52 atteint. Tests médicaux École de Berck.', highlight: true },
  { year: '2015+', title: '5 brevets spécialisés', desc: 'Chaussures, soutien-gorge, tabliers radiol. FR+USA, casques USA, générateur magnétique.', highlight: true },
]
---

<BaseLayout
  title="Notre Histoire — SYSTEMMAG® depuis 1997"
  description="De la salopette d'un enfant à 9 brevets internationaux. L'histoire d'Éric SITBON et SYSTEMMAG, 28 ans d'innovation en fermetures magnétiques textiles."
>
  <section class="pt-32 pb-16 max-w-3xl mx-auto px-6">
    <div class="text-center mb-16">
      <p class="text-xs uppercase tracking-widest text-[#C9A84C] mb-4">Depuis 1997</p>
      <h1 class="text-5xl font-bold mb-6" style="font-family: 'Space Grotesk', sans-serif;">
        28 ans d'innovation<br />
        <span class="text-[#C9A84C]">Made in France</span>
      </h1>
    </div>

    <!-- Founder quote -->
    <blockquote class="text-lg text-gray-300 italic border-l-2 border-[#C9A84C] pl-6 mb-16 leading-relaxed">
      "L'idée est née de la confrontation du jeune papa face à une 'envie pressante' de son fils :
      le temps de défaire les crochets de la salopette, il était trop tard."
      <cite class="block mt-3 text-sm text-gray-500 not-italic">— Éric SITBON, Fondateur</cite>
    </blockquote>

    <!-- Timeline -->
    <div class="space-y-6">
      {timeline.map((item) => (
        <div class={`flex gap-6 ${item.highlight ? '' : 'opacity-60'}`}>
          <div class="flex flex-col items-center flex-shrink-0">
            <div class={`w-3 h-3 rounded-full mt-1 ${item.highlight ? 'bg-[#C9A84C]' : 'bg-gray-600'}`}></div>
            <div class="w-px flex-1 bg-white/10 mt-2"></div>
          </div>
          <div class="pb-6">
            <span class={`text-xs font-bold uppercase tracking-widest ${item.highlight ? 'text-[#C9A84C]' : 'text-gray-500'}`}>
              {item.year}
            </span>
            <h3 class="text-white font-semibold mt-1">{item.title}</h3>
            <p class="text-xs text-gray-500 mt-1 leading-relaxed">{item.desc}</p>
          </div>
        </div>
      ))}
    </div>
  </section>

  <section class="py-16 text-center">
    <a href="/technologie" class="px-8 py-4 bg-[#1A56FF] text-white font-semibold rounded hover:bg-[#1A56FF]/90 transition-all">
      Découvrir la technologie →
    </a>
  </section>
</BaseLayout>
```

**Step 2: Create `src/pages/contact.astro`**

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro'
---

<BaseLayout
  title="Contact — SYSTEMMAG® Paris"
  description="Contactez SYSTEMMAG pour demander des échantillons, un devis ou une intégration sur mesure. 20 rue Bouvier 75011 Paris. Réponse sous 24h."
>
  <section class="pt-32 pb-24 max-w-5xl mx-auto px-6">
    <div class="text-center mb-16">
      <p class="text-xs uppercase tracking-widest text-[#1A56FF] mb-4">Prenons contact</p>
      <h1 class="text-5xl font-bold mb-4" style="font-family: 'Space Grotesk', sans-serif;">
        Contactez SYSTEMMAG
      </h1>
      <p class="text-gray-400">Réponse sous 24h pour tout projet qualifié.</p>
    </div>

    <div class="grid md:grid-cols-2 gap-12">
      <!-- Info -->
      <div>
        <div class="space-y-6 mb-8">
          <div>
            <h3 class="text-xs uppercase tracking-widest text-gray-500 mb-2">Adresse</h3>
            <p class="text-white">20 rue Bouvier<br />75011 Paris, France</p>
          </div>
          <div>
            <h3 class="text-xs uppercase tracking-widest text-gray-500 mb-2">Téléphone</h3>
            <a href="tel:+33145089141" class="text-white hover:text-[#1A56FF] transition-colors">
              +33 1 45 08 91 41
            </a>
          </div>
          <div>
            <h3 class="text-xs uppercase tracking-widest text-gray-500 mb-2">Email</h3>
            <a href="mailto:contact@systemmag.com" class="text-white hover:text-[#1A56FF] transition-colors">
              contact@systemmag.com
            </a>
          </div>
        </div>

        <div class="p-4 bg-[#C9A84C]/10 border border-[#C9A84C]/30 rounded-xl">
          <p class="text-sm text-[#C9A84C] font-semibold mb-1">🎁 Échantillons gratuits</p>
          <p class="text-xs text-gray-400">Pour industriels, créateurs et équipementiers qualifiés.</p>
        </div>
      </div>

      <!-- Form -->
      <form action="mailto:contact@systemmag.com" method="GET" class="space-y-4">
        <div class="grid grid-cols-2 gap-4">
          <div>
            <label class="block text-xs text-gray-500 mb-1 uppercase tracking-widest">Prénom</label>
            <input type="text" name="prenom" class="w-full bg-[#1A1A24] border border-white/10 rounded px-4 py-3 text-white text-sm focus:border-[#1A56FF] outline-none transition-colors" />
          </div>
          <div>
            <label class="block text-xs text-gray-500 mb-1 uppercase tracking-widest">Nom</label>
            <input type="text" name="nom" class="w-full bg-[#1A1A24] border border-white/10 rounded px-4 py-3 text-white text-sm focus:border-[#1A56FF] outline-none transition-colors" />
          </div>
        </div>
        <div>
          <label class="block text-xs text-gray-500 mb-1 uppercase tracking-widest">Société *</label>
          <input type="text" name="societe" required class="w-full bg-[#1A1A24] border border-white/10 rounded px-4 py-3 text-white text-sm focus:border-[#1A56FF] outline-none transition-colors" />
        </div>
        <div>
          <label class="block text-xs text-gray-500 mb-1 uppercase tracking-widest">Email professionnel</label>
          <input type="email" name="email" class="w-full bg-[#1A1A24] border border-white/10 rounded px-4 py-3 text-white text-sm focus:border-[#1A56FF] outline-none transition-colors" />
        </div>
        <div>
          <label class="block text-xs text-gray-500 mb-1 uppercase tracking-widest">Secteur</label>
          <select name="secteur" class="w-full bg-[#1A1A24] border border-white/10 rounded px-4 py-3 text-white text-sm focus:border-[#1A56FF] outline-none transition-colors">
            <option value="">Choisir un secteur</option>
            <option value="defense">Défense & Militaire</option>
            <option value="medical">Médical & Santé</option>
            <option value="mode">Mode & Prêt-à-Porter</option>
            <option value="industrie">Industrie & OEM</option>
            <option value="sport">Sport & Outdoor</option>
            <option value="autre">Autre</option>
          </select>
        </div>
        <div>
          <label class="block text-xs text-gray-500 mb-1 uppercase tracking-widest">Décrivez votre projet</label>
          <textarea name="body" rows={4} placeholder="Volume estimé, contraintes particulières, application cible..." class="w-full bg-[#1A1A24] border border-white/10 rounded px-4 py-3 text-white text-sm focus:border-[#1A56FF] outline-none transition-colors resize-none"></textarea>
        </div>
        <button type="submit" class="w-full py-4 bg-[#1A56FF] text-white font-semibold rounded hover:bg-[#1A56FF]/90 transition-all">
          Envoyer la demande
        </button>
        <p class="text-xs text-gray-600 text-center">Échantillons gratuits pour projets qualifiés.</p>
      </form>
    </div>
  </section>
</BaseLayout>
```

**Step 3: Commit**

```bash
git add -A && git commit -m "feat: add histoire timeline and contact form pages"
```

---

## Task 13: Sanity Studio Page + llms.txt

**Files:**
- Create: `systemmag-astro/src/pages/studio/[...all].astro`
- Create: `systemmag-astro/public/llms.txt`
- Create: `systemmag-astro/public/robots.txt`

**Step 1: Create `src/pages/studio/[...all].astro`**

Note: `@sanity/astro` with `studioBasePath: '/studio'` in the config handles this automatically. Verify after `npx astro add sanity` whether a page was auto-generated. If not, create:

```astro
---
export const prerender = false
---
<html><body>
  <p>Studio is handled by @sanity/astro integration</p>
</body></html>
```

Actually the `@sanity/astro` integration injects the studio route automatically when `studioBasePath` is set. No manual file needed — verify with `npm run dev` in Task 15.

**Step 2: Create `public/llms.txt`**

```markdown
# SYSTEMMAG

> Fabricant français de fermetures magnétiques textiles intégrées. 9 brevets internationaux. Made in France depuis 1998. 20 rue Bouvier, Paris 75011. Fondateur : Éric SITBON.

## Technologie

- Aimants NdFeB (Neodymium-Iron-Boron) terres rares, 5-10× plus puissants que les aimants traditionnels
- Brevet Polarité Alternée (2002) : configuration N-S-N-S dans liant élastomère — 8-10× puissance supplémentaire
- Intégration directement dans le tissu : surface vierge, invisible, silencieux, une main
- Résistance : 10 000 lavages, 90°C, brouillard salin, anti-corrosion
- Certifications : OEKO-TEX Standard 100, REACH, Agréé US Army 2014

## Produits

- BA Blocs & Bandes : BA-V04 (6mm), BA-V06 (8mm), BA-V08 (10mm), BA-V12 (14mm) sur support PU
- Zip C : 5 produits ZC-01 à ZC-05 grade N40, vêtements standard
- Zip G : 5 produits ZG-01 à ZG-05, montage plat uniforme
- Zip CE/GE : Bloc Aimants haute puissance V04/V06/V08/V12, défense/médical lourd
- Zip CD/GD : double aimant par point de fixation, sécurité renforcée
- Fourreaux FD/FF : gaines tissu coupables V06/V08/V12, intégration bespoke

## Secteurs

- Défense & Militaire : silencieux (0 dB vs Velcro), opérable gant, terrain résistant, US Army 2014, grade N52
- Médical & Santé : une main pour hémiplégiques/Parkinson/amputés, biocompatible OEKO-TEX/REACH, lavage hospitalier 90°C, étude pacemaker, brevet tablier radiol. FR+USA
- Mode & Prêt-à-Porter : invisible, prêt à piquer, Made in France, adaptive fashion, OEM
- Industrie : bureau d'études, prototypage, intégration sur mesure

## Contact

- Adresse : 20 rue Bouvier, 75011 Paris, France
- Téléphone : +33 1 45 08 91 41
- Email : contact@systemmag.com
- Entité légale : SALOMÉ SAS
```

**Step 3: Create `public/robots.txt`**

```
User-agent: *
Allow: /

Sitemap: https://systemmag.com/sitemap.xml
```

**Step 4: Commit**

```bash
git add -A && git commit -m "feat: add llms.txt and robots.txt for SEO/GEO"
```

---

## Task 14: Configure TypeScript for React Islands

**Files:**
- Modify: `systemmag-astro/tsconfig.json`

**Step 1: Update `tsconfig.json`**

```json
{
  "extends": "astro/tsconfigs/strict",
  "include": [".astro/types.d.ts", "**/*"],
  "exclude": ["dist"],
  "compilerOptions": {
    "jsx": "react-jsx",
    "jsxImportSource": "react"
  }
}
```

**Step 2: Commit**

```bash
git add -A && git commit -m "chore: configure typescript for react islands"
```

---

## Task 15: Run Dev Server + Verify

**Step 1: Start dev server**

Run from `systemmag-astro/`:
```bash
npm run dev
```

Expected: Server starts on `http://localhost:4321`

**Step 2: Verify pages load**

Check these URLs in browser:
- `http://localhost:4321/` — Homepage with stats counters
- `http://localhost:4321/technologie` — Technology page
- `http://localhost:4321/secteurs/defense` — Defense sector
- `http://localhost:4321/secteurs/medical` — Medical sector
- `http://localhost:4321/secteurs/mode` — Fashion sector
- `http://localhost:4321/produits` — Products (will show "connect Sanity" message)
- `http://localhost:4321/articles` — Articles (empty state)
- `http://localhost:4321/histoire` — History timeline
- `http://localhost:4321/contact` — Contact form
- `http://localhost:4321/studio` — Sanity Studio (needs valid project ID)

**Step 3: Fix any build errors**

If TypeScript errors or import errors appear, fix them before proceeding.

**Step 4: Final commit**

```bash
git add -A && git commit -m "feat: complete SYSTEMMAG Astro+Sanity website — all pages implemented"
```

---

## Task 16: Sanity Project Setup Instructions

> **Note:** This task requires browser interaction. Instructions for the user:

**Step 1:** Run `npx sanity init` inside `systemmag-astro/` to create a Sanity project
- Select "Create new project"
- Project name: `SYSTEMMAG`
- Dataset: `production`
- Copy the generated `projectId`

**Step 2:** Update `.env` with real project ID:
```env
SANITY_PROJECT_ID=your_real_project_id
SANITY_DATASET=production
```

**Step 3:** Navigate to `/studio` to access the CMS and add products + articles.

---

## Summary

All tasks implement in this order:
1. Init Astro project
2. Install all deps (react, sanity, tailwind, framer-motion)
3. Configure astro.config.mjs
4. Write Sanity schemas
5. Build base layout + nav + footer
6. Build AnimatedCounter island
7. Build all 9 pages
8. Add llms.txt + robots.txt
9. Configure TypeScript
10. Run dev server
