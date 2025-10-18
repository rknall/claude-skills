# UI/UX Design Review Skill

A comprehensive Claude Code skill for reviewing user interface and user experience designs with extensive accessibility analysis.

## Overview

This skill provides expert-level design review capabilities for websites and desktop applications, with a strong emphasis on WCAG 2.1/2.2 accessibility compliance and usability best practices.

## What This Skill Reviews

- **Accessibility (WCAG 2.1/2.2)**: Comprehensive compliance checking across all success criteria
- **Visual Design**: Layout, typography, color, hierarchy, consistency
- **User Experience**: Usability, user flows, interaction patterns, navigation
- **Responsive Design**: Mobile, tablet, desktop breakpoints and adaptation
- **Interactive Components**: Buttons, forms, modals, dropdowns, navigation
- **Typography & Readability**: Font choices, sizing, line height, contrast
- **Color & Contrast**: Palette cohesion, contrast ratios, color blindness
- **Navigation & IA**: Information architecture, menu structure, wayfinding
- **Forms & Data Entry**: Input design, validation, error handling
- **Desktop Applications**: Native OS patterns, keyboard shortcuts, window management

## When to Use

Activate this skill when you need to:
- Review UI/UX designs, wireframes, or mockups
- Conduct accessibility audits (WCAG compliance)
- Evaluate usability and user experience
- Assess visual design quality
- Review interaction patterns
- Analyze responsive design approaches
- Evaluate design systems or component libraries
- Review desktop application interfaces
- Check color contrast and accessibility
- Analyze information architecture

## Resources Included

### SKILL.md
The main skill definition with comprehensive review framework covering 12+ evaluation areas including extensive WCAG 2.1/2.2 compliance checking.

### wcag-checklist.md
Complete WCAG 2.1/2.2 compliance checklist with:
- All Level A requirements (minimum compliance)
- All Level AA requirements (recommended target)
- All Level AAA requirements (enhanced accessibility)
- Testing methods for each criterion
- Detailed guidance for each success criterion
- Links to official resources

### design-patterns-library.md
Accessible component implementations including:
- Navigation patterns (skip links, responsive menus, breadcrumbs)
- Form patterns (inputs, radio buttons, toggle switches)
- Modal dialogs with focus management
- Button patterns (icon buttons, loading states)
- Dropdown/combobox patterns
- Alert and notification patterns
- Accordion patterns
- Code examples with HTML, CSS, JavaScript
- Accessibility annotations for each pattern

### testing-resources.md
Tools and methodologies for testing:
- Automated accessibility testing tools
- Manual testing procedures
- Screen reader testing guides
- Color contrast analyzers
- Keyboard navigation testing
- Browser compatibility testing
- Usability testing methods

## Installation

### From Marketplace

1. Add this marketplace to Claude Code:
```bash
/plugin marketplace add rknall/claude-skills
```

2. Install the skill:
```bash
/plugin install ui-design-review
```

### Manual Installation

1. Clone or download this repository
2. Copy the `ui-design-review` directory to `~/.claude/skills/`
3. Restart Claude Code or reload skills

## Usage Examples

### Example 1: Accessibility Audit

```
I need a full WCAG 2.1 Level AA accessibility audit of my website homepage.
Here's a screenshot: [attach screenshot]

The site needs to be compliant with ADA requirements.
```

### Example 2: Design System Review

```
I'm building a component library for our design system.
Can you review these button components for accessibility and best practices?

We need to support keyboard navigation, screen readers, and meet WCAG AA standards.
```

### Example 3: Form UX Review

```
Here's our signup form design. Can you review it for usability and accessibility?

Focus areas:
- Form validation approach
- Error messaging
- Mobile experience
- Accessibility for screen readers
```

### Example 4: Responsive Design Evaluation

```
I've designed a dashboard that needs to work on mobile, tablet, and desktop.
Can you review the responsive behavior and suggest improvements?

Breakpoints: 320px, 768px, 1024px, 1440px
```

### Example 5: Color Contrast Check

```
Can you check if our color palette meets WCAG AA contrast requirements?

Primary: #3B82F6
Secondary: #10B981
Text: #1F2937
Background: #FFFFFF
```

### Example 6: Desktop Application UI

```
I'm designing a desktop app for Windows and macOS.
Can you review this mockup for platform consistency and usability?

[attach mockup]
```

## Review Output

The skill provides structured reviews including:

### 1. Executive Summary
- Overall design assessment
- Key strengths identified
- Critical issues requiring attention
- WCAG compliance level (A, AA, AAA)
- Design maturity score

### 2. Accessibility Analysis (Priority Section)
- WCAG compliance summary by level
- Critical accessibility issues with severity
- Detailed findings with WCAG criterion references
- Specific remediation steps
- User impact analysis
- Testing recommendations

### 3. Visual Design Assessment
- Strengths and concerns (HIGH/MEDIUM/LOW priority)
- Typography evaluation
- Color palette analysis
- Layout and hierarchy review
- Consistency assessment

### 4. UX & Usability Assessment
- User flow analysis
- Interaction pattern evaluation
- Navigation review
- Cognitive load analysis
- Jakob Nielsen's heuristics evaluation

### 5. Responsive Design Assessment
- Breakpoint strategy review
- Mobile-specific evaluation
- Touch target size verification
- Content reflow analysis

### 6. Component & Pattern Review
- Component consistency
- State design completeness
- Pattern library alignment
- Best practices adherence

## Accessibility Compliance Levels

### Level A (Minimum)
Basic web accessibility features that must be present for some users to access content.

### Level AA (Recommended Target)
Deals with the biggest barriers for disabled users. This is the level most organizations aim for and is typically required by law.

### Level AAA (Enhanced)
Highest level of accessibility. May not be possible to achieve for all content types.

## Key Review Areas

### Accessibility (WCAG)
- ✅ Text alternatives (alt text, captions, transcripts)
- ✅ Keyboard accessibility (no keyboard traps, logical focus order)
- ✅ Color contrast (4.5:1 for normal text, 3:1 for large text/UI)
- ✅ Semantic HTML and ARIA attributes
- ✅ Screen reader compatibility
- ✅ Form accessibility (labels, error messages, validation)
- ✅ Focus management and indicators
- ✅ Time limits and animations
- ✅ Responsive and zoomable content

### Visual Design
- Layout and visual hierarchy
- Typography and readability
- Color theory and palette
- White space and balance
- Brand consistency
- Modern vs dated patterns

### User Experience
- User flows and task efficiency
- Information architecture
- Navigation clarity
- Cognitive load
- Error prevention and recovery
- Feedback mechanisms

### Responsive Design
- Mobile-first approach
- Breakpoint strategy
- Touch target sizes (44x44px minimum)
- Content adaptation
- Navigation patterns

### Components
- Button states (default, hover, focus, active, disabled)
- Form inputs and validation
- Modals and dialogs
- Loading states
- Error states

## Testing Tools Recommended

### Automated Testing
- **axe DevTools**: Browser extension for accessibility testing
- **WAVE**: Web accessibility evaluation tool
- **Lighthouse**: Chrome DevTools audit
- **Pa11y**: Automated accessibility testing

### Manual Testing
- **Keyboard navigation**: Tab through entire interface
- **Screen readers**: NVDA (Windows), JAWS (Windows), VoiceOver (macOS)
- **Color contrast**: WebAIM Contrast Checker, Stark
- **Color blindness**: Color Oracle, Sim Daltonism
- **Zoom**: Test at 200% and 400% zoom levels

### Usability Testing
- **UserTesting.com**: Remote user testing
- **Maze**: Product research platform
- **Hotjar**: Behavior analytics
- **FullStory**: Session replay

## Best Practices Checklist

- [ ] All images have appropriate alt text
- [ ] Color contrast meets WCAG AA (4.5:1 for text)
- [ ] All functionality is keyboard accessible
- [ ] Focus indicators are visible
- [ ] Form inputs have associated labels
- [ ] Error messages are clear and helpful
- [ ] Touch targets are at least 44x44px
- [ ] Content reflows without horizontal scrolling at 320px
- [ ] Headings are used hierarchically
- [ ] Skip links allow bypassing navigation
- [ ] ARIA attributes are used correctly
- [ ] Animations can be paused or disabled
- [ ] Text can be resized to 200% without loss of functionality

## Platform-Specific Guidelines

### Web
- W3C Web Content Accessibility Guidelines (WCAG)
- MDN Web Docs best practices
- Progressive enhancement principles

### Windows Desktop
- Fluent Design System
- Windows App SDK guidelines
- WinUI 3 design patterns

### macOS Desktop
- Human Interface Guidelines
- AppKit design patterns
- SwiftUI best practices

### Linux Desktop
- GNOME Human Interface Guidelines
- KDE Human Interface Guidelines
- Freedesktop.org standards

## Common Issues Found

### Critical Accessibility Issues
- Missing alt text on images
- Insufficient color contrast
- Keyboard traps in modals
- Missing form labels
- No focus indicators
- Inaccessible custom components

### UX Issues
- Unclear navigation structure
- Poor error messages
- Too many steps to complete tasks
- Inconsistent interaction patterns
- Missing loading indicators
- No confirmation for destructive actions

### Visual Design Issues
- Poor visual hierarchy
- Inconsistent spacing
- Too many font sizes/families
- Cluttered layouts
- Low contrast text

## Version History

- **1.0.0** (2025-10-18): Initial release
  - Comprehensive WCAG 2.1/2.2 review framework
  - Complete compliance checklist
  - Accessible component pattern library
  - Testing resources and tools

## Contributing

To improve this skill:
1. Update SKILL.md with new review criteria
2. Add new patterns to design-patterns-library.md
3. Update WCAG checklist as standards evolve
4. Add new testing tools and resources
5. Include platform-specific guidelines

## Resources

- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [WCAG 2.2 Updates](https://www.w3.org/WAI/WCAG22/quickref/)
- [WebAIM Resources](https://webaim.org/)
- [A11Y Project](https://www.a11yproject.com/)
- [Inclusive Components](https://inclusive-components.design/)
- [Material Design Accessibility](https://material.io/design/usability/accessibility.html)
- [Apple HIG Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility/overview/introduction/)

## License

This skill is provided as-is for use with Claude Code.

## Support

For issues, questions, or suggestions about this skill, please contact the maintainer or open an issue in the repository.
