# Research & Implementation Notes

## Overview

This research documents the technical context and implementation approach for adding an interactive "NZ Working Holiday Readiness Quiz" to the nuevazelanda Statamic site. The quiz targets Spanish-speaking South Americans considering a working holiday in New Zealand, covering three dimensions: personal readiness, practical factors, and English language skills.

The site uses **Statamic CMS with the Peak starter kit**, leveraging **Alpine.js** for interactivity and **Tailwind CSS** for styling. The recommended approach is to create a new page builder block that integrates seamlessly with the existing architecture, keeping all quiz logic client-side with Alpine.js to avoid backend complexity.

## Research Findings

### Current Technology Stack

The site runs on Statamic CMS (Laravel-based) with the Studio1902 Peak starter kit. This provides a well-structured foundation with:

- **Templating**: Antlers templating engine (`.antlers.html` files)
- **Frontend JS**: Alpine.js v3.12.1 with plugins (collapse, focus, morph, persist)
- **CSS**: Tailwind CSS v3.3.2 with custom configuration
- **Build**: Vite v4.3.5 for asset bundling
- **Forms**: Built-in form handling with Alpine.js integration

The Alpine.js setup in [site.js](resources/js/site.js) already includes the `persist` plugin, which will be useful for saving quiz progress across page refreshes.

### Page Builder Architecture

The site uses Statamic's Replicator field type for a modular page builder system. Each block consists of:

1. **Fieldset** (`resources/fieldsets/*.yaml`) - Defines the CMS-editable fields
2. **Template** (`resources/views/page_builder/_*.antlers.html`) - Renders the block
3. **Registration** in `page_builder.yaml` - Adds the block to the page builder

Looking at existing blocks like `_form.antlers.html` and `_cta.antlers.html`, the pattern is clear: each block is a self-contained component that uses Alpine.js `x-data` directives for state management, with Tailwind classes for styling.

### Existing Component Patterns

The form component ([_form.antlers.html](resources/views/page_builder/_form.antlers.html)) demonstrates the established patterns:

- Uses `x-data="formHandler()"` for Alpine.js state
- Employs `x-cloak` to prevent flash of unstyled content
- Uses template partials for typography (`partial:typography/h1`, etc.)
- Implements responsive grid with `fluid-container grid md:grid-cols-12`
- Shows conditional content with `template x-if`
- Handles notifications with a dedicated notification component

The CTA component ([_cta.antlers.html](resources/views/page_builder/_cta.antlers.html)) shows animation patterns with `x-transition` directives and delayed appearance using `x-init` with `setTimeout`.

### Content Structure

Pages are stored as Markdown files in `content/collections/pages/` with YAML frontmatter. The page blueprint (`resources/blueprints/collections/pages/page.yaml`) imports the page_builder fieldset, allowing any page to use the modular block system.

### Styling System

The Tailwind configuration is split across multiple files:
- `tailwind.config.js` - Main config, imports others
- `tailwind.config.site.js` - Site-specific colours, fonts, and custom plugins
- `tailwind.config.peak.js` - Peak starter kit utilities
- `tailwind.config.typography.js` - Typography preset

The site uses a minimal colour palette with `primary`, `neutral`, `black`, and `white`. The primary colour uses OKLCH colour space for modern colour handling.

### Internationalisation Considerations

The site appears Spanish-first based on content files and view directories (`actividades`, `entradas`, `recursos`). The spec requires Spanish as the primary language with potential English toggle. Statamic supports multi-language setups, but for initial implementation, hardcoding Spanish content in the template is acceptable, with future i18n handled through Statamic's localisation features.

## Technical Context

### Relevant Files

**Core templates and configuration:**
- [resources/views/layout.antlers.html](resources/views/layout.antlers.html) - Main layout, loads site.js and site.css via Vite
- [resources/views/default.antlers.html](resources/views/default.antlers.html) - Default page template, iterates page_builder blocks
- [resources/js/site.js](resources/js/site.js) - Alpine.js setup with plugins
- [resources/css/site.css](resources/css/site.css) - CSS entry point, imports Tailwind
- [resources/css/custom.css](resources/css/custom.css) - Custom CSS (currently empty)

**Page builder system:**
- [resources/fieldsets/page_builder.yaml](resources/fieldsets/page_builder.yaml) - Defines available blocks
- [resources/views/page_builder/](resources/views/page_builder/) - Block templates

**Reference implementations:**
- [resources/views/page_builder/_form.antlers.html](resources/views/page_builder/_form.antlers.html) - Complex Alpine.js form handling
- [resources/views/page_builder/_cta.antlers.html](resources/views/page_builder/_cta.antlers.html) - Animation and transitions
- [resources/views/page_builder/_faq.antlers.html](resources/views/page_builder/_faq.antlers.html) - Accordion pattern (likely uses collapse plugin)

**Component partials:**
- [resources/views/components/_notification.antlers.html](resources/views/components/_notification.antlers.html) - Notification display
- [resources/views/components/_buttons.antlers.html](resources/views/components/_buttons.antlers.html) - Button component

### Alpine.js Data Flow

The quiz will use Alpine.js for all interactivity. Based on the existing patterns, the recommended approach is:

```javascript
x-data="{
    currentQuestion: 0,
    answers: [],
    showResults: false,
    totalQuestions: 15,

    get progress() { return ((this.currentQuestion + 1) / this.totalQuestions) * 100 },
    get totalScore() { return this.answers.reduce((a, b) => a + b, 0) },
    get categoryScores() { /* calculate per-category scores */ },
    get resultLevel() { /* determine result tier based on totalScore */ },

    selectAnswer(value) { /* handle answer selection */ },
    nextQuestion() { /* advance to next question */ },
    prevQuestion() { /* go back to previous question */ },
    restart() { /* reset quiz state */ }
}"
```

Using `$persist` from Alpine's persist plugin can save answers to localStorage, allowing users to resume if they navigate away.

### Quiz Data Structure

Questions and resources can be:
1. **Hardcoded in template** - Simpler, faster to implement
2. **CMS-editable via fieldset** - More flexible, allows non-developer edits

For v1, a hybrid approach is recommended: hardcode questions in the template (they're fixed per the spec), but make result resources CMS-editable so links can be updated without code changes.

## Decisions & Rationale

### Page Builder Block vs. Standalone Page

We chose to implement the quiz as a page builder block rather than a standalone template because this matches the existing site architecture. Every content page uses the page builder system, and adding the quiz as a block means it can be placed on any page, combined with other blocks (like an intro article above it), and managed through the same CMS interface content editors already know.

A standalone template would require creating a separate route, blade/antlers view, and potentially a new collection. This adds complexity and deviates from established patterns without providing meaningful benefits.

### Client-Side Scoring

The spec explicitly states "Score calculated client-side (no sensitive data stored)" and "No user account required". This means we don't need any backend logic for the quiz itself. All scoring and result calculations happen in Alpine.js, which:

- Eliminates the need for API endpoints or form submissions
- Ensures privacy - no quiz data leaves the browser
- Enables instant results without page reloads
- Simplifies implementation significantly

The only optional server interaction would be the "email results" feature (future enhancement), which could use Statamic's built-in form handling.

### Question Display: One Per Screen

The spec calls for "One question per screen (progress bar visible)" with back navigation. This is a deliberate UX decision to reduce cognitive load and encourage thoughtful responses. The implementation uses conditional rendering (`x-show`) to display one question at a time while maintaining all answers in Alpine's state.

Alternative approaches like a scrolling single-page quiz or modal-based questions were considered but rejected because the single-question-per-screen pattern better matches the self-reflection goal of the quiz and provides clearer progress indication.

### Scoring System Implementation

The 1-5 Likert scale maps directly to numeric values. Question 15 (language improvement steps) uses multiple-choice mapped to scores rather than a scale, which requires special handling in the answer component.

Score calculation:
- Total: Sum of all 15 answers (range: 15-75)
- Per category: Sum of 5 answers each (range: 5-25)
- Result tier: Based on total score thresholds (15-35, 36-50, 51-65, 66-75)

The weakest category determines which resources to prioritise in results, implemented as a simple comparison of category scores.

### Hardcoded Questions, CMS-Editable Resources

Questions are fixed by the spec and unlikely to change frequently. Hardcoding them in the Antlers template:

- Reduces CMS complexity (no need for repeater fields with 15+ items)
- Makes the template self-contained and easier to version control
- Avoids potential issues with CMS users accidentally breaking the quiz structure

Resources, however, may need updates (URLs change, new resources become available). The fieldset will include a structured resources section allowing CMS editors to update links without touching code.

### Spanish-First, No Toggle for v1

While the spec mentions "English toggle for bilingual users", implementing proper i18n adds significant complexity. For v1, the quiz will be Spanish-only, matching the rest of the site. The template structure will support future localisation by using translatable string keys where appropriate, but the toggle feature is deferred.

### Accessibility Approach

The quiz must be keyboard navigable with proper ARIA labels. Implementation will use:

- `role="radiogroup"` for answer options
- `aria-labelledby` linking questions to their text
- `aria-current="step"` for progress indication
- Focus management when navigating between questions
- Colour + icons + text for result indicators (not colour alone)

The existing Peak toolkit provides accessible form patterns that will be adapted for the quiz interface.

## Implementation Notes

### Files to Create

1. **Fieldset**: `resources/fieldsets/quiz.yaml`
   - Title, introduction text, resources configuration
   - Minimal fields since questions are hardcoded

2. **Template**: `resources/views/page_builder/_quiz.antlers.html`
   - Full quiz UI with Alpine.js logic
   - Question display, progress bar, results

3. **Registration**: Update `resources/fieldsets/page_builder.yaml`
   - Add quiz block to "Interactive" group

4. **Styles**: Optional additions to `resources/css/custom.css`
   - Progress bar styling
   - Score gauge/bar styling (if needed beyond Tailwind utilities)

### Component Structure

The template should be organised into logical sections:

```html
<!-- Quiz container with Alpine state -->
<section x-data="quiz()" class="...">

    <!-- Progress bar -->
    <div class="...">...</div>

    <!-- Question display (one at a time) -->
    <template x-for="(question, index) in questions" :key="index">
        <div x-show="currentQuestion === index" class="...">
            <!-- Question text -->
            <!-- Answer options -->
            <!-- Navigation buttons -->
        </div>
    </template>

    <!-- Results screen -->
    <div x-show="showResults" class="...">
        <!-- Score display -->
        <!-- Category breakdown -->
        <!-- Personalised resources -->
        <!-- Retake/share buttons -->
    </div>

</section>
```

### Answer Option UI

For the 1-5 scale, use clickable cards rather than a slider for better accessibility and mobile experience:

```html
<div class="flex gap-2 justify-center">
    <template x-for="n in 5">
        <button
            @click="selectAnswer(n)"
            :class="{ 'ring-2 ring-primary': answers[currentQuestion] === n }"
            class="w-12 h-12 rounded-full border-2 ..."
        >
            <span x-text="n"></span>
        </button>
    </template>
</div>
```

With labels below showing the scale meaning (1 = "Nada preparado", 5 = "Muy preparado").

### Progress Persistence

Using Alpine's `$persist` plugin:

```javascript
answers: $persist([]).as('quiz-answers'),
currentQuestion: $persist(0).as('quiz-progress'),
```

This saves state to localStorage, allowing users to resume. Add a "Empezar de nuevo" button that clears persisted state.

### Results Calculation

```javascript
get categoryScores() {
    return {
        personal: this.answers.slice(0, 5).reduce((a, b) => a + b, 0),
        practical: this.answers.slice(5, 10).reduce((a, b) => a + b, 0),
        language: this.answers.slice(10, 15).reduce((a, b) => a + b, 0)
    }
},

get weakestCategory() {
    const scores = this.categoryScores;
    return Object.entries(scores).reduce((a, b) => a[1] < b[1] ? a : b)[0];
}
```

### Resource Display Logic

Based on `weakestCategory`, show relevant resources. These should be defined in the fieldset and passed to the template:

```yaml
# In fieldset
resources:
  personal:
    - title: "Comunidad de chilenos en NZ"
      url: "https://facebook.com/..."
  practical:
    - title: "Immigration NZ"
      url: "https://immigration.govt.nz/..."
  language:
    - title: "British Council"
      url: "https://britishcouncil.org/..."
```

### Testing Considerations

- Test all 4 result tiers by manipulating answers
- Verify keyboard navigation through entire quiz
- Test on mobile viewports (the spec mentions "clickable cards or slider")
- Confirm localStorage persistence works correctly
- Test with screen reader to verify ARIA implementation

### Edge Cases

- **Incomplete quiz**: If user has partial progress, show "Continuar" vs "Comenzar"
- **All equal category scores**: Show general resources, not category-specific
- **Question 15 multiple choice**: Needs different UI (checkboxes mapped to score)
- **Browser without localStorage**: Graceful degradation, just don't persist

### Future Enhancement Hooks

Structure the code to support future features without major refactoring:

- **Email results**: Add a form at results screen, use Statamic forms
- **Analytics tracking**: Add event emissions for key interactions
- **Bilingual toggle**: Use Statamic's localisation for question/label strings
- **PDF export**: Generate via client-side library or server endpoint
