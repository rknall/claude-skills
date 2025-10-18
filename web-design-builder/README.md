# Web Design Builder Skill

Create professional, accessible HTML5/JavaScript web designs with automatic Playwright verification.

## Overview

This skill generates complete, production-ready web designs from specifications, with automatic accessibility and functionality testing using Playwright MCP integration.

## Key Features

- **Complete Design Generation**: HTML5, CSS3, JavaScript from specifications
- **Automatic Verification**: Playwright MCP integration for testing (when available)
- **Accessibility First**: WCAG 2.1 Level AA compliance built-in
- **Responsive Design**: Mobile-first, tested across breakpoints
- **Modern Frameworks**: Vanilla, Tailwind CSS, React, Vue, Alpine.js support
- **Interactive Testing**: Keyboard navigation, form validation, component functionality
- **Visual Verification**: Screenshots at multiple breakpoints
- **Performance Optimization**: Load time analysis, resource optimization

## When to Use

- Create landing pages, websites, or web applications
- Build responsive web interfaces from specifications
- Generate design mockups or prototypes
- Refactor existing HTML/CSS/JavaScript code
- Create accessible, WCAG-compliant designs
- Build component libraries or design systems

## Playwright MCP Integration

### Automatic Detection

The skill automatically detects if Playwright MCP is installed. If not available:
- User is informed before starting
- Verification steps are skipped
- Manual testing guidance is provided
- Installation instructions are offered

### When Playwright MCP is Available

Automatic verification includes:
1. **Accessibility Testing**
   - WCAG compliance checking
   - Color contrast verification
   - Keyboard navigation testing
   - Screen reader compatibility

2. **Visual Testing**
   - Screenshots at mobile (375px), tablet (768px), desktop (1440px)
   - Full-page captures
   - Responsive layout verification

3. **Functionality Testing**
   - Form validation
   - Interactive components
   - JavaScript functionality
   - Error handling

4. **Performance Analysis**
   - Load time measurement
   - Resource size analysis
   - Optimization recommendations

## Workflow

### Phase 1: Requirements Gathering
- Design scope and type
- Target audience and use case
- Technical preferences (framework, browser support)
- Accessibility requirements
- **Check Playwright MCP availability**

### Phase 2: Design Generation
- Create semantic HTML5 structure
- Responsive CSS with mobile-first approach
- Accessible JavaScript with progressive enhancement
- Save to file

### Phase 3: Verification (if Playwright MCP available)
- Launch browser and load design
- Run accessibility tests
- Capture responsive screenshots
- Test interactive functionality
- Analyze performance

### Phase 4: Verification Report
- Comprehensive accessibility compliance report
- Visual testing results
- Functionality test outcomes
- Performance metrics
- Prioritized recommendations

### Phase 5: Iteration
- Fix critical issues
- Apply improvements
- Re-verify
- Deliver final design

## Supported Frameworks

### Vanilla HTML/CSS/JS
- No dependencies
- Simple and fast
- Best for: Static sites, simple interactions

### Tailwind CSS
- Utility-first CSS
- Rapid development
- Best for: Prototypes, modern interfaces

### React
- Component-based
- Virtual DOM
- Best for: Complex SPAs, interactive apps

### Vue
- Progressive framework
- Easy to learn
- Best for: Medium complexity apps

### Alpine.js
- Lightweight (15KB)
- Declarative syntax
- Best for: Progressive enhancement

## Installation

```bash
/plugin marketplace add rknall/claude-skills
/plugin install web-design-builder
```

## Example Usage

### Basic Landing Page
```
Create a landing page for a SaaS product with:
- Hero section with CTA
- Features grid (3 columns)
- Pricing table
- Contact form

Use Tailwind CSS and make it fully accessible.
```

### Dashboard Interface
```
Build an admin dashboard with:
- Sidebar navigation
- Top header with user menu
- Stats cards
- Data table
- Charts

Responsive design, dark mode support.
```

### Contact Form
```
Create an accessible contact form with:
- Name, email, message fields
- Real-time validation
- Success/error states
- WCAG AA compliant

Include keyboard navigation support.
```

## Installing Playwright MCP

If Playwright MCP is not installed:

```bash
# Install Playwright MCP server
npm install -g @playwright/mcp-server

# Configure in Claude Code
claude code mcp add playwright

# Restart Claude Code
```

## Design Templates

The skill includes templates for:
- SaaS Landing Pages
- Contact Forms
- Admin Dashboards
- Product Pages
- Portfolio Sites

See `design-templates.md` for complete examples.

## Accessibility Features

All designs include:
- ✅ Semantic HTML5 elements
- ✅ ARIA landmarks and labels
- ✅ Keyboard navigation support
- ✅ Focus indicators (3:1 contrast)
- ✅ Color contrast compliance (4.5:1 minimum)
- ✅ Skip links
- ✅ Screen reader friendly
- ✅ Responsive and zoomable
- ✅ Form labels and validation
- ✅ Alt text for images

## Deliverables

- Complete HTML/CSS/JavaScript files
- Verification report (if MCP available)
- Screenshots at multiple breakpoints
- Usage documentation
- Customization guide
- Deployment instructions

## Version History

- **1.0.0** (2025-10-18): Initial release
  - Playwright MCP integration
  - Multiple framework support
  - Accessibility-first approach
  - Comprehensive verification

## License

This skill is provided as-is for use with Claude Code.
