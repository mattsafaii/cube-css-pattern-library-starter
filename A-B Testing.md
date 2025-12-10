# A/B Testing with Pattern Variants

The pattern library system doubles as an A/B test archive, allowing you to document experiments, track results, and preserve winning variations as permanent patterns.

## Setting Up A/B Test Variants

### 1. Create Test Variants

For each A/B test variation, create a pattern variant following the standard pattern creation process:

```bash
# Control (Variant A)
npm run patterns:create -- -p "patterns/hero/variants" -n "hero-control" -t "Hero Control"

# Treatment (Variant B)
npm run patterns:create -- -p "patterns/hero/variants" -n "hero-treatment-cta" -t "Hero Treatment CTA"
```

Or use data-based variants if only content/classes differ:

```bash
npm run patterns:create -- -p "patterns/button/variants" -n "button-variant-a" -t "Button Variant A" -sm true
npm run patterns:create -- -p "patterns/button/variants" -n "button-variant-b" -t "Button Variant B" -sm true
```

### 2. Add A/B Test Metadata

In each variant's `.json` file, add test metadata:

```json
{
  "title": "Hero - Treatment B",
  "abTest": {
    "testId": "hero-cta-q4-2025",
    "hypothesis": "Changing CTA from 'Get Started' to 'Try Free' will increase conversions",
    "startDate": "2025-11-01",
    "endDate": "2025-11-15",
    "status": "completed",
    "winner": true,
    "metrics": {
      "conversionRate": "4.2%",
      "sampleSize": "12,450",
      "confidence": "95%",
      "lift": "+23%"
    },
    "notes": "Significant improvement in mobile conversions specifically"
  },
  "context": {
    "headline": "Start Your Free Trial Today",
    "ctaText": "Try Free",
    "ctaClass": "btn btn-primary btn-large"
  }
}
```

### 3. Metadata Field Reference

**Required fields:**

- `testId`: Unique identifier for the test (kebab-case recommended)
- `hypothesis`: What you're testing and why
- `status`: `"draft"`, `"running"`, `"completed"`, or `"archived"`

**Optional fields:**

- `startDate`: Test start date (YYYY-MM-DD)
- `endDate`: Test end date (YYYY-MM-DD)
- `winner`: Boolean indicating if this variant won
- `metrics`: Object containing test results
  - `conversionRate`: Primary metric
  - `sampleSize`: Number of visitors/sessions
  - `confidence`: Statistical confidence level
  - `lift`: Percentage improvement over control
- `notes`: Additional observations or insights

## Common A/B Test Scenarios

### Content/Copy Testing (Data-Based Variants)

Best for testing different text, classes, or simple prop changes with shared markup.

**Parent pattern** (`button.njk`):

```njk
<button class="{{ data.classes }}">
  {{ data.text }}
</button>
```

**Variant A** (`variants/a/button-a.json`):

```json
{
  "title": "Button - Variant A",
  "abTest": {
    "testId": "cta-wording-nov-2025",
    "hypothesis": "Action-oriented CTAs perform better",
    "status": "running"
  },
  "context": {
    "text": "Get Started Now",
    "classes": "btn btn-primary"
  }
}
```

**Variant B** (`variants/b/button-b.json`):

```json
{
  "title": "Button - Variant B",
  "abTest": {
    "testId": "cta-wording-nov-2025",
    "hypothesis": "Value-focused CTAs perform better",
    "status": "running"
  },
  "context": {
    "text": "Start Free Trial",
    "classes": "btn btn-primary"
  }
}
```

### Layout/Structure Testing (File-Based Variants)

Best for testing different HTML structures, component arrangements, or visual hierarchies.

Each variant gets its own `.njk` and `.json` file with completely different markup.

**Structure:**

```
patterns/hero/
├── hero.njk
├── hero.json
├── hero.md
└── variants/
    ├── image-left/
    │   ├── hero-image-left.njk    # Image on left side
    │   └── hero-image-left.json
    └── image-right/
        ├── hero-image-right.njk   # Image on right side
        └── hero-image-right.json
```

### Visual Design Testing

Test color schemes, button sizes, spacing, etc. using CSS class variations.

```json
{
  "title": "CTA Card - High Contrast",
  "abTest": {
    "testId": "cta-visibility-q4-2025",
    "hypothesis": "Higher contrast increases click-through",
    "status": "completed",
    "winner": false,
    "metrics": {
      "conversionRate": "2.1%",
      "lift": "-15%"
    },
    "notes": "High contrast decreased conversions, users reported feeling pressured"
  },
  "context": {
    "bgClass": "bg-primary",
    "textClass": "text-white",
    "ctaClass": "btn btn-white btn-large"
  }
}
```

## Managing Test Results

### 1. During Testing

Set `status: "running"` and update the variant's documentation as data comes in.

### 2. After Completion

Update the winning variant:

```json
{
  "abTest": {
    "status": "completed",
    "winner": true,
    "endDate": "2025-11-15",
    "metrics": { ... }
  }
}
```

Update losing variant(s):

```json
{
  "abTest": {
    "status": "completed",
    "winner": false,
    "metrics": { ... },
    "notes": "Why this variant lost and what we learned"
  }
}
```

### 3. Promoting Winners

When a variant wins and becomes the new default:

1. Copy winning variant's code to parent pattern
2. Update parent's `.json` with winning data
3. Archive test variants by setting `status: "archived"`
4. Document in parent's `.md` file (see next section)

## Documenting Tests in Pattern .md Files

Each pattern has a `.md` file for documentation that renders in the pattern library UI. Use this to create an archive of A/B tests and learnings.

### Recommended Structure

```markdown
# Pattern Name

Brief description of what this pattern is and when to use it.

## Usage

\`\`\`njk
{{ design.patterns.renderPattern('pattern-name', {
  prop: "value"
}) | safe }}
\`\`\`

## Variants

### Variant Name

Description of what this variant is for.

## Design Notes

- Key design decisions
- Usage guidelines
- Dependencies on other patterns

## A/B Test History

### Test Name (Month Year)

- **Winner:** Brief description of winning variant
- **Test ID:** `test-id-here`
- **Metrics:** Conversion rate, lift percentage
- **Key Learning:** What you learned from this test
- **Variants:** See `variants/folder-name/`

### Previous Test Name (Month Year)

...
```

### Example Pattern Documentation

**File:** `src/pattern-library/patterns/hero/hero.md`

```markdown
# Hero Pattern

Large, attention-grabbing section typically used at the top of landing pages. Combines headline, subheadline, CTA button, and optional hero image.

## Usage

\`\`\`njk
{{ design.patterns.renderPattern('hero', {
  headline: "Your headline here",
  subheadline: "Supporting text",
  ctaText: "Call to action",
  ctaHref: "/signup"
}) | safe }}
\`\`\`

## Variants

### Max Variant

Desktop layout with side-by-side content and image. Used at viewport widths above 768px.

### Min Variant

Mobile layout with stacked content. Optimized for touch targets and smaller screens.

## Design Notes

- Headline should be 8-12 words maximum for readability
- CTA button uses primary brand color from design tokens
- Background supports both solid colors and gradients
- Hero image should be 1200x800px minimum for retina displays

## Required Patterns

- [Button](/pattern-library/pattern/button) - For CTA
- [Container](/pattern-library/pattern/container) - For content wrapper

## A/B Test History

### Hero Headline Test (Dec 2025)

- **Winner:** Benefit-focused headline (+23% conversion)
- **Test ID:** `hero-headline-dec-2025`
- **Metrics:** 4.2% conversion (vs 3.4% control), 95% confidence, 12,450 sample size
- **Key Learning:** "Ship Products in Half the Time" outperformed "Build Better Websites Faster" - benefit-focused messaging resonates more than feature-focused
- **Variants:** See `variants/control/` and `variants/treatment/`

### Hero CTA Wording Test (Nov 2025)

- **Winner:** "Start Free Trial" (+18% click-through)
- **Test ID:** `hero-cta-nov-2025`
- **Metrics:** 8.3% CTR (vs 7.0% control)
- **Key Learning:** Value-oriented CTAs ("Free Trial") outperform action-oriented ("Get Started")
- **Variants:** Archived in `variants/cta-test/`
```

### Benefits of Documentation

1. **Institutional Knowledge** - Preserve learnings even if you step away from the project
2. **Avoid Repeat Tests** - See what's already been tested before running duplicates
3. **Pattern Evolution** - Track how patterns improved over time
4. **Design Rationale** - Document why current implementation is what it is
5. **Quick Reference** - See usage examples without digging through code

## Best Practices

1. **One test ID per experiment** - Use the same `testId` across all variants in a test
2. **Document hypothesis** - Always state what you're testing and why
3. **Preserve losers** - Keep losing variants archived for future reference
4. **Add learnings** - Use `notes` field to capture insights beyond metrics
5. **Update status** - Keep status current: draft → running → completed → archived
6. **Link to analytics** - Consider adding a `dashboardUrl` field linking to test results

## Example: Complete A/B Test Flow

```bash
# 1. Create variants for hero CTA test
npm run patterns:create -- -p "patterns/hero/variants" -n "hero-control" -t "Hero Control"
npm run patterns:create -- -p "patterns/hero/variants" -n "hero-treatment" -t "Hero Treatment"
```

**Control variant** (`variants/control/hero-control.json`):

```json
{
  "title": "Hero - Control",
  "abTest": {
    "testId": "hero-headline-dec-2025",
    "hypothesis": "Feature-focused headline will increase signups",
    "startDate": "2025-12-01",
    "status": "running"
  },
  "context": {
    "headline": "Build Better Websites Faster",
    "subheadline": "Our design system helps you ship quality products.",
    "ctaText": "Get Started"
  }
}
```

**Treatment variant** (`variants/treatment/hero-treatment.json`):

```json
{
  "title": "Hero - Treatment",
  "abTest": {
    "testId": "hero-headline-dec-2025",
    "hypothesis": "Benefit-focused headline will increase signups",
    "startDate": "2025-12-01",
    "status": "running"
  },
  "context": {
    "headline": "Ship Quality Products in Half the Time",
    "subheadline": "Join 10,000+ teams using our design system.",
    "ctaText": "Start Free Trial"
  }
}
```

After test completion, update both with results and promote the winner to the parent pattern.

## Additional Use Cases for Variants

### A/B Testing Use Cases

- **CTA variations** - Different button text, value propositions, urgency language
- **Layout experiments** - Image placement, column layouts, card grids
- **Copy variations** - Long-form vs short-form, feature vs benefit-focused
- **Visual hierarchy** - Button prominence, color schemes, icon placement

### Other Variant Applications

- **Internationalization** - Different languages, RTL/LTR layouts, content lengths
- **User roles** - Admin vs user dashboards, permission-based features
- **Device optimization** - Touch vs mouse interfaces, mobile vs desktop patterns
- **Accessibility** - High contrast, reduced motion, screen reader optimized
- **Progressive disclosure** - Collapsed/expanded, summary/detail views
- **State management** - Error states, empty states, loading/skeleton states
- **Seasonal variations** - Holiday themes, promotional banners, special events
