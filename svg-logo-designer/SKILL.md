---
name: "svg-logo-designer"
description: "Create professional SVG logos from descriptions and design specifications. Generates multiple logo variations with different layouts, styles, and concepts. Produces scalable vector graphics that can be used directly or exported to PNG. Use this skill when users ask to create logos, brand identities, icons, or visual marks for their designs."
---

# SVG Logo Designer

Create professional, scalable SVG logos from design specifications with multiple variations and layout options.

## Core Workflow

### Phase 1: Requirements Gathering

Collect these inputs before designing. Ask the user if any are missing:

1. **Brand info**: Name, industry, target audience, personality (modern/classic/playful/serious), values
2. **Logo type preference**: Wordmark, lettermark, pictorial mark, abstract mark, combination mark, or emblem
3. **Style**: Minimalist, geometric, organic, bold, elegant, playful, tech/modern, or vintage
4. **Colors**: Specific palette or user wants suggestions
5. **Technical needs**: Primary use contexts (web, print, merchandise), background requirements (light/dark/transparent)
6. **Variations**: Number of concepts (recommend 3-5), layouts needed (horizontal, vertical, square, icon-only, text-only)

### Phase 2: Design Concept Development

Create 3-5 distinct concepts exploring different visual approaches:

- Different visual metaphors related to the brand
- Different style treatments and compositions
- Negative space opportunities
- Cultural appropriateness considerations

For each concept, use this SVG scaffold:

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 200 200" width="200" height="200">
  <defs>
    <linearGradient id="gradient1" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" style="stop-color:#4F46E5;stop-opacity:1" />
      <stop offset="100%" style="stop-color:#7C3AED;stop-opacity:1" />
    </linearGradient>
  </defs>
  <g id="logo-symbol">
    <!-- Symbol/icon elements -->
  </g>
  <g id="logo-text">
    <!-- Text elements (if applicable) -->
  </g>
</svg>
```

### Phase 3: Layout Variations

For each selected concept, produce these layouts:

| Layout | Aspect Ratio | Best For |
|--------|-------------|----------|
| Horizontal lockup | Wide | Website headers, business cards |
| Vertical lockup | Tall | Social media profiles, app stores |
| Square/centered | 1:1 | Favicon, app icon, profile picture |
| Icon only | Compact | Small sizes, watermarks |
| Text only | Wide | Minimal applications |

### Phase 4: SVG Generation

Follow these mandatory patterns:

**Semantic grouping:**
```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 200 60">
  <g id="icon"><!-- Icon elements --></g>
  <g id="wordmark"><!-- Text elements --></g>
</svg>
```

**Color management -- define once, reuse throughout:**
```xml
<defs>
  <style>
    .primary { fill: #4F46E5; }
    .secondary { fill: #10B981; }
    .text { fill: #1F2937; }
  </style>
</defs>
<rect class="primary" x="0" y="0" width="100" height="100" />
```

**Accessibility -- always include title and desc:**
```xml
<svg role="img" aria-labelledby="logo-title logo-desc">
  <title id="logo-title">Company Name Logo</title>
  <desc id="logo-desc">A blue circular icon with the company name</desc>
  <!-- Logo content -->
</svg>
```

**Optimization rules:**
- Use `viewBox` for scalability; avoid fixed pixel sizes
- Remove unnecessary attributes and invisible elements
- Combine paths where possible
- Use `<symbol>` and `<use>` for repeated elements
- Minimize decimal precision (max 2 places)

### Phase 5: Validation

Run these checks on every generated SVG before presenting to the user:

1. **SVG syntax check**: Verify well-formed XML -- all tags closed, attributes quoted, namespace declared (`xmlns="http://www.w3.org/2000/svg"`)
2. **Multi-size rendering test**: Confirm the logo renders correctly at 16px (favicon), 64px (icon), 200px (standard), and 1000px (large print) by checking that `viewBox` is set and no fixed `width`/`height` values break scaling
3. **Accessibility verification**: Confirm `<title>` and `<desc>` elements are present, `role="img"` is set, and `aria-labelledby` references valid IDs
4. **Color contrast**: Verify primary text/icon colors meet WCAG AA contrast ratio (4.5:1) against intended backgrounds
5. **Cross-variation consistency**: Ensure icon, colors, and proportions are consistent across all layout variations of the same concept
6. **File size**: SVG should be under 10KB for simple logos, under 50KB for complex ones

If any check fails, fix the issue before presenting the logo.

### Phase 6: Presentation

Present logos in this structured format:

```markdown
# Logo Design Concepts

## Concept 1: [Name/Theme]

### Design Rationale
[Visual metaphors used, how it represents the brand]

### Primary Logo (Horizontal)
[SVG code]
**Usage:** Headers, navigation, business cards
**Dimensions:** 200x60px (scalable)

### Vertical Layout
[SVG code]

### Icon Only
[SVG code]

### Color Variations
- Full Color: Primary #XXXXXX, Secondary #XXXXXX
- Monochrome Dark: #1F2937
- Monochrome Light: #FFFFFF
- Reversed: For dark backgrounds
```

### Phase 7: File Delivery

Save SVG files using this naming convention:
```
company-name-logo-concept1-horizontal.svg
company-name-logo-concept1-vertical.svg
company-name-logo-concept1-icon.svg
company-name-logo-concept1-horizontal-monochrome.svg
```

Use the Write tool to save each variation to a `logos/` directory.

For export instructions (SVG to PNG), usage guidelines, file organization, and web implementation examples, see [README.md](./README.md).

## SVG Code Patterns

### Wordmark

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 300 80" role="img" aria-labelledby="wm-title">
  <title id="wm-title">Company Wordmark</title>
  <defs>
    <style>
      .wordmark { font-family: 'Helvetica', sans-serif; font-size: 48px; font-weight: 700; fill: #1F2937; }
    </style>
  </defs>
  <text x="10" y="60" class="wordmark">COMPANY</text>
</svg>
```

### Geometric Icon

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100" role="img" aria-labelledby="geo-title">
  <title id="geo-title">Company Icon</title>
  <defs>
    <linearGradient id="grad" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" style="stop-color:#4F46E5" />
      <stop offset="100%" style="stop-color:#7C3AED" />
    </linearGradient>
  </defs>
  <polygon points="50,5 95,27.5 95,72.5 50,95 5,72.5 5,27.5" fill="url(#grad)" stroke="#312E81" stroke-width="2" />
  <circle cx="50" cy="50" r="20" fill="#FFFFFF" />
</svg>
```

### Abstract Mark

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100" role="img" aria-labelledby="abs-title">
  <title id="abs-title">Company Abstract Mark</title>
  <path d="M10,50 Q30,20 50,50 T90,50 Q70,80 50,50 T10,50 Z" fill="#10B981" opacity="0.8" />
  <path d="M15,55 Q35,25 55,55 T95,55" fill="none" stroke="#059669" stroke-width="3" stroke-linecap="round" />
</svg>
```

### Combination Mark

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 200 80" role="img" aria-labelledby="combo-title">
  <title id="combo-title">Company Logo</title>
  <g id="icon">
    <circle cx="40" cy="40" r="30" fill="#4F46E5" />
    <path d="M30,35 L35,45 L50,25" stroke="#FFFFFF" stroke-width="3" fill="none" stroke-linecap="round" stroke-linejoin="round" />
  </g>
  <g id="text">
    <text x="85" y="45" font-family="Arial, sans-serif" font-size="28" font-weight="700" fill="#1F2937">COMPANY</text>
  </g>
</svg>
```

## Iteration Process

After presenting initial concepts:

1. **Gather feedback**: Which concept resonates? What to keep/change?
2. **Refine**: Adjust colors, proportions, and details on the selected concept
3. **Re-validate**: Run all Phase 5 validation checks on refined versions
4. **Deliver final package**: All layouts, all color variations, usage guidelines

For complete logo type descriptions, color specifications, export instructions, and usage guidelines, see [README.md](./README.md).
