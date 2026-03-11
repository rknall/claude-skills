---
name: "ui-design-review"
description: "Comprehensive design review for websites and desktop applications with extensive accessibility analysis. Use this skill when users ask you to review UI/UX designs, wireframes, mockups, prototypes, or deployed interfaces for usability, accessibility (WCAG compliance), visual design, interaction patterns, responsive design, and best practices for web and desktop applications."
---

# UI/UX Design Review

## Review Workflow

### Phase 1: Context Gathering

Before reviewing, establish:
- Platform(s): web, desktop (Windows/Mac/Linux), mobile
- Target WCAG compliance level (A, AA, or AAA)
- Target audience, technical proficiency, and accessibility needs
- Design system or brand guidelines in use
- Project stage (wireframe, pre-launch, live)
- Browser/OS support requirements

**Validation checkpoint:** Do not proceed until platform and compliance level are confirmed.

### Phase 2: Artifact Analysis

Analyze provided designs (screenshots, Figma links, prototypes, live URLs) across:
- Visual hierarchy and layout structure
- Color usage and contrast ratios
- Typography choices and readability
- Component patterns and state coverage
- Navigation structure and information architecture
- Responsive behavior across breakpoints
- Interactive element states (hover, active, focus, disabled, error)

**Validation checkpoint:** Confirm you have sufficient artifacts to review. Request missing materials before continuing.

### Phase 3: Accessibility Audit

Conduct a WCAG compliance review using the full [WCAG Checklist](wcag-checklist.md).

Focus on these high-impact areas first:

1. **Semantic HTML structure** - proper heading hierarchy, landmark regions, list markup
2. **Keyboard accessibility** - tab order, focus visibility, no keyboard traps
3. **Color contrast** - measure ratios against WCAG thresholds (4.5:1 normal text, 3:1 large text for AA)
4. **Text alternatives** - alt text on images, labels on icons, descriptions on complex graphics
5. **Form accessibility** - associated labels, error identification, input purpose
6. **ARIA usage** - correct roles, states, and properties where semantic HTML is insufficient

#### Common Fixes: Before/After Examples

**Missing form labels:**
```html
<!-- Before: inaccessible -->
<input type="email" placeholder="Enter email">

<!-- After: accessible -->
<label for="email">Email address</label>
<input type="email" id="email" placeholder="user@example.com"
       aria-describedby="email-hint">
<span id="email-hint" class="hint">We'll never share your email.</span>
```

**Non-semantic button:**
```html
<!-- Before: inaccessible -->
<div class="btn" onclick="submit()">Submit</div>

<!-- After: accessible -->
<button type="submit" class="btn">Submit</button>
```

**Missing landmark regions:**
```html
<!-- Before: no structure for assistive tech -->
<div class="header">...</div>
<div class="sidebar">...</div>
<div class="content">...</div>

<!-- After: semantic landmarks -->
<header role="banner">...</header>
<nav aria-label="Main navigation">...</nav>
<main>...</main>
<aside aria-label="Related content">...</aside>
<footer role="contentinfo">...</footer>
```

**Icon button without accessible name:**
```html
<!-- Before: screen reader says "button" with no context -->
<button><svg class="icon-close">...</svg></button>

<!-- After: screen reader says "Close dialog" -->
<button aria-label="Close dialog"><svg class="icon-close" aria-hidden="true">...</svg></button>
```

**Image with missing alt text:**
```html
<!-- Before -->
<img src="chart.png">

<!-- After: informative image -->
<img src="chart.png" alt="Sales increased 40% from Q1 to Q3 2025">

<!-- After: decorative image -->
<img src="divider.png" alt="" role="presentation">
```

**Custom toggle missing ARIA state:**
```html
<!-- Before: no state communicated -->
<div class="toggle active" onclick="toggle()">Dark mode</div>

<!-- After: state communicated to assistive tech -->
<button role="switch" aria-checked="true" onclick="toggle()">Dark mode</button>
```

**Live region for dynamic content:**
```html
<!-- Announce updates without moving focus -->
<div aria-live="polite" aria-atomic="true" class="status-message">
  3 items added to cart
</div>
```

**Validation checkpoint:** Every accessibility issue must include the WCAG criterion reference (e.g., 1.4.3), severity level, and a concrete fix. Verify at least the top 5 high-impact areas above are covered.

### Phase 4: Visual Design Assessment

Evaluate:
- **Hierarchy**: visual weight distribution, layout balance, grid usage
- **Color**: palette cohesion, semantic color usage, dark mode support, color-blind safety
- **Typography**: font choices, type scale, line length (45-75 chars), line height (1.5-1.8 body)
- **Spacing**: consistent whitespace, density appropriate for context
- **Consistency**: design token usage, component styling uniformity

See [Design Patterns Library](design-patterns-library.md) for accessible pattern implementations.

### Phase 5: UX & Usability Assessment

Evaluate against Nielsen's 10 heuristics:
- System status visibility, user control, consistency, error prevention
- Recognition over recall, flexibility, minimalist design, error recovery

Focus feedback on:
- Task completion efficiency and cognitive load
- Navigation clarity and information architecture depth (3 levels max)
- Error prevention, messaging quality, and recovery paths
- Empty states, loading states, and edge cases
- Platform convention adherence

### Phase 6: Responsive Design & Layout

Evaluate:
- Breakpoint strategy and content reflow
- Touch targets (minimum 44x44px)
- Mobile navigation adaptation
- Typography and image scaling
- Form usability on small screens

**Desktop-specific:** window resizing, multi-monitor support, native OS patterns, keyboard shortcuts, context menus, drag-and-drop.

### Phase 7: Component & Interaction Review

For each interactive component, verify:
- [ ] All states designed: default, hover, focus, active, disabled, error, loading, success
- [ ] Touch targets meet 44x44px minimum
- [ ] Focus indicators are visible and meet 3:1 contrast
- [ ] Loading states prevent double submission
- [ ] Error messages are specific and actionable
- [ ] Destructive actions require confirmation

See [Design Patterns Library](design-patterns-library.md) for accessible implementations of common components (modals, dropdowns, accordions, tabs).

**Validation checkpoint:** Confirm every interactive element has been reviewed for keyboard access and state coverage.

## Review Output Format

Structure every review as:

### 1. Executive Summary
- Overall assessment (1-2 paragraphs)
- Key strengths
- Critical issues requiring immediate attention
- Current WCAG compliance level estimate

### 2. Accessibility Findings (Priority Section)

**WCAG Compliance Summary:**
| Level | Violations | Status |
|-------|-----------|--------|
| A     | count     | Pass/Fail |
| AA    | count     | Pass/Fail |
| AAA   | count     | Advisory |

**For each issue, provide:**
- WCAG criterion (e.g., "1.4.3 Contrast (Minimum)")
- Severity: Critical / High / Medium / Low
- Description and user impact
- Concrete fix (code example where applicable)

### 3. Visual Design Findings
Prioritized list: HIGH (layout/hierarchy problems) > MEDIUM (inconsistencies) > LOW (polish)

### 4. UX & Usability Findings
Prioritized list: HIGH (task blockers) > MEDIUM (friction points) > LOW (enhancements)

### 5. Responsive Design Findings
Issues at specific breakpoints with recommended fixes.

### 6. Component Review
Missing states, pattern violations, and design system alignment issues.

### 7. Recommended Next Steps
Prioritized action items with effort estimates (quick win / moderate / significant).

## Priority Classification

| Priority | Criteria | Examples |
|----------|----------|----------|
| **Critical** | Blocks core functionality or access | WCAG A violations, keyboard traps, missing alt on functional images |
| **High** | Significantly impairs experience | WCAG AA violations, major usability issues, poor mobile experience |
| **Medium** | Creates friction with workarounds | Visual inconsistencies, minor usability issues, AAA recommendations |
| **Low** | Polish and refinement | Aesthetic improvements, edge cases, future enhancements |

## Testing Recommendations

After the review, recommend appropriate testing from the [Testing Resources](testing-resources.md) guide based on the issues found. At minimum, suggest:
- Automated scan (axe DevTools or Lighthouse)
- Manual keyboard navigation test
- Screen reader spot-check (VoiceOver on macOS, NVDA on Windows)
- Color contrast verification on flagged elements

## Reference Standards

- **Accessibility:** WCAG 2.1/2.2, Section 508, EN 301 549
- **Platform guidelines:** Material Design, Apple HIG, Fluent Design, GNOME/KDE HIG
- **Web standards:** W3C, MDN best practices

Full WCAG criteria details: [WCAG Checklist](wcag-checklist.md)
Accessible pattern implementations: [Design Patterns Library](design-patterns-library.md)
Testing tools and procedures: [Testing Resources](testing-resources.md)
