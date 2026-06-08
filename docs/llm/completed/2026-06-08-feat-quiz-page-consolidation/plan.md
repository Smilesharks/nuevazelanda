# Work Plan: Quiz Page Review and Improvement

**Issue**: N/A (internal quality review)
**Status**: Draft

## Problem

The Working Holiday NZ quiz page (`resources/views/page_builder/_quiz.antlers.html`) has structural issues that likely break functionality and create a confusing user experience. A recent commit (c22130f) added a new hero section, but this was prepended rather than integrated, leaving two independent Alpine.js components on the same page.

**Impact**: Users attempting to start the quiz from the new hero section will interact with a separate Alpine instance from the one controlling the actual quiz questions. Progress, state, and navigation will not sync between the two sections.

## Proposal

Consolidate the quiz into a single, coherent component that preserves the best elements of both the new hero design and the original quiz functionality. Address the misleading statistics and visual inconsistencies as part of this consolidation.

## Deliverables

1. **Single Alpine.js component** - one `x-data="quiz()"` binding controlling the entire page
2. **Unified visual design** - consistent colour palette (decide: yellow/gold or emerald/green)
3. **Locally-hosted hero image** - no external Google URL dependencies
4. **Honest statistics section** - either remove unverifiable claims or replace with accurate data
5. **Single start point** - one clear call-to-action button, not two competing ones

## Key Decisions

### 1. Visual Identity: Yellow is the Brand Colour

Research confirmed that **yellow/gold is the site's primary brand colour**, not emerald/green. Evidence from existing components:

- `_hero_home.antlers.html`: `bg-yellow-50`
- `_cta.antlers.html`: `bg-yellow-400`, `bg-yellow-500`
- `_entry_list.antlers.html`: `bg-yellow-50`, `border-yellow-800/10`, `bg-yellow-400/20`
- Share buttons: `hover:fill-yellow-600`

**Decision**: Use yellow as the primary colour for the quiz hero and CTA elements. Keep rose/amber/emerald as the three category indicator colours within the quiz questions and results (these work well as distinct category identifiers).

### 2. Statistics Section: Replace with Facts

Current stats make unverifiable claims ("10,000+ sudamericanos", "98% precisión") that could undermine trust on a site that provides genuine, helpful information.

**Decision**: Replace with factual information that matches the quiz content:
- "15 preguntas" → actual question count
- "3 categorías" → actual category count (Personal, Práctico, Inglés)
- "5 minutos" → matches the intro text in quiz.md

### 3. Hero Image: Use Local Asset

Local images already exist that can replace the external Google CDN URL:
- `public/images/cover.jpg` (2251×900, 329KB) - landscape format
- `public/images/antes-de-emigrar.jpg` (1024×682, 189KB) - thematically relevant

**Decision**: Use `antes-de-emigrar.jpg` as it's thematically appropriate for the "preparing for your Working Holiday" context.

### 4. Layout: Centred, Not Side-by-Side

The site uses centred layouts consistently (`fluid-container`, `max-w-lg mx-auto`, etc.). The new hero's side-by-side layout breaks this pattern.

**Decision**: Use a centred layout matching `_hero_home.antlers.html` pattern.

## Scope

### In Scope
- Merge the two `<section>` elements into one coherent component
- Remove duplicate start buttons and quiz headers
- Replace external image URL with local asset
- Update statistics to verifiable claims or remove
- Unify colour scheme across hero and quiz body

### Out of Scope
- Quiz question content changes (questions themselves are well-written)
- Quiz logic changes (scoring, categories, persistence all work correctly)
- Adding analytics or tracking
- Translations to other languages

## Acceptance Criteria

1. Page renders with a single Alpine.js `quiz()` component
2. "Comenzar quiz" button starts the quiz and shows the first question
3. Progress persists across page refreshes (existing `$persist` behaviour maintained)
4. No console errors related to duplicate component definitions
5. Hero image loads from local path, not external URL
6. Statistics section contains only verifiable information
7. Visual design uses consistent colour palette throughout

## Open Questions

All questions resolved through research:

1. ~~**Image source**~~ → Use `public/images/antes-de-emigrar.jpg` (local, thematically appropriate)
2. ~~**Stats section placement**~~ → Keep stats section but with factual content
3. ~~**Hero layout**~~ → Use centred layout to match site patterns

## Related Work

- Commit c22130f: "feat: add Working Holiday readiness quiz" - introduced the current structure
- The quiz partial is used via `page_builder`, so changes here affect any page that includes the quiz block
