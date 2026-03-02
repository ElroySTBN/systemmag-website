# SYSTEMMAG — Base de connaissances projet

> Documentation de référence pour le projet de refonte du site web SYSTEMMAG.
> Ce repo contient toutes les données, analyses et interviews permettant à un agent de comprendre SYSTEMMAG en profondeur.

---

## Qui est SYSTEMMAG ?

**SYSTEMMAG** — PME française fondée en 1998 par Éric Sitbon, Paris 11e (20 rue Bouvier).
- **9 brevets internationaux** (INPI, USPTO, EPO, OMPI)
- **9 lignes de fabrication robotisées** conçues en interne
- **100% Made in France**
- Technologie : rail magnétique continu à polarités alternées — silencieux, utilisable avec gants, résistant eau/sable, stérilisable

**Clients :** Plusieurs armées (depuis 2014), NASA/Axiom Space, Outlier NYC, Chantal Thomas, secteur médical/radiologie

---

## Structure de la documentation

```
docs/
├── prd/                    ← Documents de référence produit
│   ├── PRD-MASTER-SITE-SYSTEMMAG.md   ← LIRE EN PREMIER
│   └── 2026-01-12_prd-initial.md
│
├── analyses/               ← Analyses stratégiques et techniques
│   ├── 2026-01-27_document-maitre-eric-sitbon.md   ← Storytelling & citations Eric
│   ├── 2026-01-29_analyse-rapport-nasa.md
│   ├── 2026-02-04_fiche-technique-materiaux.md
│   ├── attestation-jc-peuzin-cnrs-2005.md
│   └── ...
│
├── research/               ← Veille marché et concurrentielle
│   ├── 2026-01-12_market-systemes-fermeture.md
│   ├── 2026-01-13_competitive-deep-dive.md
│   └── ...
│
├── interviews/             ← Interviews équipe SYSTEMMAG
│   ├── 2026-01-20_questionnaire-eric.md
│   ├── 2026-01-24_interview-frederique.md
│   ├── 2026-01-27_interview-mathis-done.md
│   └── ...
│
├── wikipedia/              ← Article Wikipedia rédigé
│   └── 2026-02-09_wikipedia-v8-complete.md
│
├── linkedin/               ← Contenu LinkedIn
│   ├── 2026-02-09_linkedin-profil-eric-v3.md
│   └── 2026-02-02_linkedin-posts-v2-corrige.md
│
├── prompts/                ← Système de prompts
│   └── 2026-02-09_systeme-prompt-claude-systemmag.md
│
└── planning/               ← Stratégie et planning
    ├── 2026-01-25_plan-campagne-3-mois.md
    ├── 2026-01-25_strategie-referencement.md
    └── ...
```

---

## Règles vocabulaire ABSOLUES (validées par Éric Sitbon)

| ❌ INTERDIT | ✅ OBLIGATOIRE |
|-------------|---------------|
| "US Army" / "armée américaine" | **"plusieurs armées"** |
| "atelier" | **"usine"** |
| "aimants" seul | **"systèmes magnétiques"** |
| "membre NAUMD" (direct) | **"référencé via A+ Products"** |
| "Salomon" | **NE PAS MENTIONNER** |

---

## Point de départ recommandé pour un agent

1. Lire `docs/prd/PRD-MASTER-SITE-SYSTEMMAG.md`
2. Lire `docs/analyses/2026-01-27_document-maitre-eric-sitbon.md`
3. Lire `docs/interviews/2026-01-27_interview-mathis-done.md`
