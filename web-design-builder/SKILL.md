---
name: "web-design-builder"
description: "Create and refactor HTML5/JavaScript web designs from specifications or descriptions. Generates complete, accessible, responsive web designs with modern frameworks. Automatically verifies designs using Playwright MCP for accessibility and functionality testing. Use this skill when users ask to create web designs, mockups, landing pages, web applications, or refactor existing HTML/CSS/JS designs."
---

# Web Design Builder

Create professional HTML5/JavaScript web designs from specifications, with automatic accessibility and functionality verification using Playwright MCP.

## Core Workflow

### Phase 1: Requirements Gathering

When a user requests a web design:

1. **Clarify the Design Scope**
   - Design type (landing page, dashboard, form, SPA, etc.)
   - Target audience and use case
   - Required features and functionality
   - Content: provided or placeholder?
   - Brand colors, fonts, or design system
   - Responsive requirements (mobile-first?)

2. **Technical Preferences**
   - Framework: Vanilla HTML/CSS/JS, Tailwind CSS, React, Vue, or Alpine.js
   - See [framework-guidelines.md](./framework-guidelines.md) for framework-specific setup and patterns
   - Browser support and accessibility level (WCAG AA minimum)

3. **Check Playwright MCP Availability**

   Check if `mcp__playwright` tools are available (e.g., `mcp__playwright__navigate`, `mcp__playwright__screenshot`).

   - **Available**: "Playwright MCP detected. Design will be automatically verified."
   - **Not available**: "Playwright MCP is not installed. Design verification will be skipped." Provide installation instructions from [playwright-mcp-setup.md](./playwright-mcp-setup.md) if it exists, otherwise direct the user to install `@playwright/mcp-server` via npm and configure it in Claude Code.

### Phase 2: Design Generation

Generate a complete HTML/CSS/JS design. Use templates from [design-templates.md](./design-templates.md) as starting points for common patterns (landing pages, forms, dashboards).

**Required standards for all generated designs:**

- Semantic HTML5 with proper heading hierarchy (h1-h6)
- ARIA landmarks and attributes where needed
- Skip links, focus indicators, keyboard navigation support
- WCAG 2.1 Level AA: color contrast 4.5:1 minimum
- Responsive layout using CSS Grid/Flexbox, mobile-first
- CSS custom properties for theming
- Progressive enhancement for JavaScript
- Accessible form labels, error messages with `role="alert"`

**Accessible Modal Pattern** (reuse in any design needing modals):
```javascript
function openModal(modalId) {
  const modal = document.getElementById(modalId);
  const lastFocused = document.activeElement;
  modal.hidden = false;
  modal.setAttribute('aria-modal', 'true');
  const firstFocusable = modal.querySelector(
    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
  );
  firstFocusable?.focus();
  modal.dataset.lastFocused = lastFocused;
  trapFocus(modal);
}

function closeModal(modalId) {
  const modal = document.getElementById(modalId);
  modal.hidden = true;
  const lastFocused = document.querySelector(
    `[data-last-focused="${modal.id}"]`
  );
  lastFocused?.focus();
}
```

**Save the design to a file:**
- Single-file HTML (CSS/JS inline) for mockups
- Separate files for production builds
- Use descriptive filenames (e.g., `landing-page.html`, `dashboard.html`)

### Phase 3: Verification (requires Playwright MCP)

Skip this phase entirely if Playwright MCP is not available. Provide a manual testing checklist instead.

#### Step 1: Load and Screenshot

```javascript
await mcp__playwright__navigate({ url: 'file:///path/to/design.html' });
await mcp__playwright__screenshot({ fullPage: true, path: 'design-full.png' });
```

#### Step 2: Responsive Testing

```javascript
const breakpoints = [
  { width: 375, height: 667, name: 'mobile' },
  { width: 768, height: 1024, name: 'tablet' },
  { width: 1440, height: 900, name: 'desktop' }
];
for (const bp of breakpoints) {
  await mcp__playwright__setViewportSize({ width: bp.width, height: bp.height });
  await mcp__playwright__screenshot({ path: `design-${bp.name}.png` });
}
```

#### Step 3: Accessibility Audit

- Check WCAG violations, color contrast, heading hierarchy
- Tab through all interactive elements; verify visible focus indicators and logical tab order
- Verify ARIA landmarks, form labels, alt text, dynamic content announcements
- Test Escape key for modals/dropdowns; confirm no keyboard traps

#### Step 4: Functionality Testing

- Test form validation: required fields, error messages, success states
- Test interactive components: buttons, modals, dropdowns, tabs
- Check browser console for JavaScript errors

### Phase 4: Validate-Fix-Retry Loop

After verification (or manual review), iterate until all issues are resolved:

```
REPEAT:
  1. Identify issues from verification report (or manual checklist)
  2. Fix each issue in the design file
  3. Re-run verification (Phase 3) or re-check manually
  4. IF no critical/high issues remain → BREAK
  5. IF issues persist after 3 iterations → report remaining issues to user
```

**Priority for fixes:**
1. **Critical**: Accessibility violations (missing labels, no keyboard access, contrast failures)
2. **High**: Broken functionality, layout issues at any breakpoint
3. **Medium**: Performance optimizations, missing error states
4. **Low**: Polish (animations, dark mode, print styles)

### Phase 5: Deliver

Provide the user with:

1. **Complete design files** (HTML, CSS, JS, assets)
2. **Verification report** (if Playwright MCP was used):
   - Accessibility compliance results
   - Responsive screenshots
   - Functionality test results
   - Remaining recommendations by priority
3. **Manual testing checklist** (if Playwright MCP was not available):
   - Open in browser, resize to mobile/tablet/desktop
   - Tab through all interactive elements
   - Test all forms and interactive components
   - Run through a screen reader or use browser accessibility tools
4. **Next steps**: suggested improvements, production checklist

## Quick Reference

| Resource | Description |
|----------|-------------|
| [design-templates.md](./design-templates.md) | Ready-to-use HTML templates (landing page, form, dashboard) |
| [README.md](./README.md) | User-facing documentation and installation guide |
