# UI/UX Testing Resources

Comprehensive guide to testing tools, methodologies, and procedures for evaluating user interfaces.

## Automated Accessibility Testing Tools

### Browser Extensions

#### axe DevTools
**Platform:** Chrome, Firefox, Edge
**Website:** https://www.deque.com/axe/devtools/

**Features:**
- Automated WCAG 2.0/2.1/2.2 testing
- Intelligent guided testing
- Highlights issues directly in browser
- Provides remediation guidance
- Export reports

**How to Use:**
1. Install browser extension
2. Open DevTools (F12)
3. Navigate to "axe DevTools" tab
4. Click "Scan ALL of my page"
5. Review issues by severity
6. Click each issue for details and how to fix

**Best For:** Daily development workflow, comprehensive scans

#### WAVE
**Platform:** Chrome, Firefox, Edge
**Website:** https://wave.webaim.org/extension/

**Features:**
- Visual feedback on accessibility issues
- Inline indicators on page
- Structure view for semantic analysis
- Contrast checker
- Free and open source

**How to Use:**
1. Install browser extension
2. Navigate to page to test
3. Click WAVE icon in toolbar
4. Review errors, alerts, and features
5. Click items for detailed information

**Best For:** Visual learners, quick checks

#### Lighthouse
**Platform:** Built into Chrome DevTools
**Website:** https://developers.google.com/web/tools/lighthouse

**Features:**
- Performance, accessibility, SEO, best practices
- Mobile and desktop testing
- Scoring system
- Actionable recommendations
- Progressive Web App audits

**How to Use:**
1. Open Chrome DevTools (F12)
2. Navigate to "Lighthouse" tab
3. Select "Accessibility" category
4. Choose device type (Mobile/Desktop)
5. Click "Generate report"
6. Review score and opportunities

**Best For:** Overall site quality assessment, CI/CD integration

### Command-Line Tools

#### Pa11y
**Platform:** Node.js
**Website:** https://pa11y.org/

**Installation:**
```bash
npm install -g pa11y
```

**Usage:**
```bash
# Test a single page
pa11y https://example.com

# Test with specific WCAG level
pa11y --standard WCAG2AA https://example.com

# Generate HTML report
pa11y --reporter html https://example.com > report.html

# Test multiple pages
pa11y-ci --config pa11y-ci.json
```

**Configuration (pa11y-ci.json):**
```json
{
  "defaults": {
    "standard": "WCAG2AA",
    "timeout": 10000
  },
  "urls": [
    "https://example.com",
    "https://example.com/about",
    "https://example.com/contact"
  ]
}
```

**Best For:** CI/CD pipelines, automated testing, bulk testing

#### axe-core
**Platform:** Node.js
**Website:** https://github.com/dequelabs/axe-core

**Installation:**
```bash
npm install --save-dev axe-core
```

**Usage with Playwright:**
```javascript
const { test, expect } = require('@playwright/test');
const AxeBuilder = require('@axe-core/playwright').default;

test('homepage should not have accessibility violations', async ({ page }) => {
  await page.goto('https://example.com');

  const accessibilityScanResults = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa'])
    .analyze();

  expect(accessibilityScanResults.violations).toEqual([]);
});
```

**Best For:** Integration testing, automated test suites

## Manual Testing Tools

### Screen Readers

#### NVDA (NonVisual Desktop Access)
**Platform:** Windows (Free)
**Website:** https://www.nvaccess.org/

**Basic Commands:**
- **Insert + Down Arrow**: Read all
- **Arrow Keys**: Navigate line by line
- **Tab**: Navigate interactive elements
- **H**: Jump to next heading
- **Insert + F7**: List all elements
- **Insert + Space**: Toggle browse/focus mode

**Testing Checklist:**
- [ ] All images have meaningful alt text
- [ ] Page title is announced
- [ ] Headings structure makes sense
- [ ] Form labels are announced
- [ ] Error messages are read aloud
- [ ] Dynamic content updates are announced
- [ ] Navigation landmarks are identified

#### JAWS (Job Access With Speech)
**Platform:** Windows (Paid, free demo)
**Website:** https://www.freedomscientific.com/products/software/jaws/

**Basic Commands:**
- **Insert + Down Arrow**: Say all
- **H**: Next heading
- **Insert + F5**: List form fields
- **Insert + F6**: List headings
- **Insert + F7**: List links
- **Insert + Space**: Toggle virtual cursor

**Testing Focus:**
- ARIA landmarks and roles
- Dynamic content updates
- Complex widgets (tabs, accordions)
- Form validation announcements

#### VoiceOver
**Platform:** macOS, iOS (Built-in)
**Website:** https://www.apple.com/accessibility/voiceover/

**Mac Commands:**
- **Cmd + F5**: Toggle VoiceOver
- **VO**: Control + Option
- **VO + Right/Left Arrow**: Navigate
- **VO + Space**: Activate
- **VO + A**: Read all
- **VO + Command + H**: Next heading

**iOS Commands:**
- **Triple-click Home/Side button**: Toggle VoiceOver
- **Swipe right/left**: Navigate
- **Double-tap**: Activate
- **Two-finger swipe down**: Read all

**Testing Focus:**
- Touch interface accessibility
- Gesture alternatives
- iOS-specific patterns

### Keyboard Testing

#### Keyboard Navigation Checklist

**Basic Navigation:**
- [ ] **Tab**: Move forward through interactive elements
- [ ] **Shift + Tab**: Move backward
- [ ] **Enter**: Activate buttons and links
- [ ] **Space**: Activate buttons, toggle checkboxes
- [ ] **Arrow keys**: Navigate within components (menus, tabs)
- [ ] **Escape**: Close modals, dropdowns, menus
- [ ] **Home/End**: Jump to beginning/end of lists

**Testing Procedure:**
1. Unplug mouse (or don't touch it)
2. Start at top of page with Tab key
3. Verify focus indicator is visible
4. Ensure logical tab order
5. Check all interactive elements are reachable
6. Verify no keyboard traps
7. Test custom components (modals, dropdowns)
8. Verify shortcut keys work
9. Ensure focus is managed properly (modals, SPAs)

**Common Issues:**
- ❌ Focus indicator removed or invisible
- ❌ Illogical tab order
- ❌ Interactive elements not keyboard accessible
- ❌ Keyboard trap in modal or widget
- ❌ No way to close modal with keyboard
- ❌ Custom controls don't respond to keyboard

### Color Contrast Tools

#### WebAIM Contrast Checker
**Website:** https://webaim.org/resources/contrastchecker/

**Features:**
- Checks contrast ratio
- WCAG AA and AAA compliance
- Suggestions for passing colors
- Lightness slider

**How to Use:**
1. Enter foreground color (text)
2. Enter background color
3. Review contrast ratio
4. Adjust colors until passing
5. Test for both normal and large text

**WCAG Requirements:**
- Normal text: 4.5:1 (AA), 7:1 (AAA)
- Large text (18pt+/14pt bold+): 3:1 (AA), 4.5:1 (AAA)
- UI components: 3:1 (AA)

#### Stark
**Platform:** Figma, Sketch, Adobe XD, Chrome
**Website:** https://www.getstark.co/

**Features:**
- Contrast checker
- Color blindness simulator
- Focus order tool
- Typography analyzer
- Real-time suggestions

**How to Use:**
1. Install plugin/extension
2. Select design elements or webpage
3. Run contrast check
4. Simulate different types of color blindness
5. Export accessibility report

**Best For:** Design phase, Figma/Sketch workflows

#### Colour Contrast Analyser (CCA)
**Platform:** Windows, macOS (Free)
**Website:** https://www.tpgi.com/color-contrast-checker/

**Features:**
- Eyedropper tool to sample colors
- Foreground/background contrast check
- WCAG 2.0/2.1 compliance
- Color simulation

**How to Use:**
1. Download and install application
2. Use eyedropper to select colors from screen
3. Review contrast ratios
4. Adjust colors as needed

**Best For:** Desktop applications, pixel-perfect testing

### Color Blindness Simulators

#### Color Oracle
**Platform:** Windows, macOS, Linux (Free)
**Website:** https://colororacle.org/

**Features:**
- Real-time color blindness simulation
- Covers deuteranopia, protanopia, tritanopia
- Full-screen overlay

**How to Use:**
1. Install application
2. Open your design/website
3. Activate simulation mode
4. Switch between different types
5. Verify all information is conveyed without color alone

#### Sim Daltonism
**Platform:** macOS, iOS (Free)
**Website:** https://michelf.ca/projects/sim-daltonism/

**Features:**
- Live preview window
- Multiple color blindness types
- Floating window you can position

**Testing Checklist:**
- [ ] Links distinguishable without color
- [ ] Form errors not relying on red color alone
- [ ] Charts use patterns in addition to color
- [ ] Status indicators use icons/text
- [ ] Important information not color-only

## Browser Testing

### Cross-Browser Testing

#### BrowserStack
**Website:** https://www.browserstack.com/

**Features:**
- Real device testing
- Desktop and mobile browsers
- Automated testing
- Local testing

**Testing Matrix:**
- Chrome (latest, latest-1)
- Firefox (latest, latest-1)
- Safari (latest, latest-1)
- Edge (latest, latest-1)
- Mobile Safari (iOS)
- Chrome Mobile (Android)

#### Responsinator
**Website:** http://www.responsinator.com/

**Features:**
- View site in multiple device sizes
- Portrait and landscape orientations
- Quick responsive testing

### Device Emulation

#### Chrome DevTools Device Mode
**How to Access:**
1. Open DevTools (F12)
2. Click device icon (Ctrl+Shift+M)
3. Select device or custom dimensions

**Features:**
- Responsive viewport
- Device emulation
- Touch simulation
- Network throttling
- Sensor simulation (geolocation, orientation)

**Testing Checklist:**
- [ ] 320px (small mobile)
- [ ] 375px (iPhone)
- [ ] 768px (tablet)
- [ ] 1024px (small desktop)
- [ ] 1440px (large desktop)

## Usability Testing

### Remote User Testing

#### UserTesting.com
**Website:** https://www.usertesting.com/

**Features:**
- Real users testing your site
- Video recordings of sessions
- Targeted demographics
- Task-based testing

**Test Plan Example:**
1. "Find and purchase a product"
2. "Create an account"
3. "Navigate to customer support"
4. "Complete the checkout process"

#### Maze
**Website:** https://maze.co/

**Features:**
- Prototype testing
- A/B testing
- First-click testing
- Heatmaps and analytics

**Metrics Tracked:**
- Task completion rate
- Time on task
- Misclick rate
- Path analysis

### Analytics & Session Recording

#### Hotjar
**Website:** https://www.hotjar.com/

**Features:**
- Heatmaps (click, move, scroll)
- Session recordings
- Feedback polls
- Conversion funnels

**Insights:**
- Where users click most
- How far users scroll
- Where users get confused
- Drop-off points

#### FullStory
**Website:** https://www.fullstory.com/

**Features:**
- Session replay
- Funnel analysis
- Error tracking
- Search and segment sessions

## Accessibility Testing Checklist

### Automated Testing (15 minutes)
- [ ] Run axe DevTools scan
- [ ] Run WAVE evaluation
- [ ] Run Lighthouse accessibility audit
- [ ] Check HTML validation
- [ ] Test with Pa11y (CI/CD)

### Manual Keyboard Testing (15 minutes)
- [ ] Navigate entire page with Tab key
- [ ] Verify visible focus indicators
- [ ] Check logical tab order
- [ ] Test all interactive elements
- [ ] Verify no keyboard traps
- [ ] Test Escape key behavior
- [ ] Check Enter/Space on buttons

### Screen Reader Testing (30 minutes)
- [ ] Test with NVDA or JAWS (Windows)
- [ ] Test with VoiceOver (macOS)
- [ ] Verify alt text on images
- [ ] Check form label associations
- [ ] Test dynamic content updates
- [ ] Verify ARIA labels on custom controls
- [ ] Check heading structure makes sense

### Visual Testing (15 minutes)
- [ ] Check color contrast ratios
- [ ] Verify text resizes to 200%
- [ ] Test with color blindness simulator
- [ ] Check responsive design
- [ ] Verify content reflows at 320px
- [ ] Test zoom to 400%

### Mobile Testing (15 minutes)
- [ ] Touch target sizes (44x44px minimum)
- [ ] Test on actual mobile device
- [ ] Verify zoom works
- [ ] Test portrait and landscape
- [ ] Check mobile navigation
- [ ] Verify form usability on mobile

## Testing Schedule

### During Development
- Automated testing on every build
- Manual keyboard testing for new components
- Color contrast checks in design phase

### Before Release
- Full WCAG audit
- Screen reader testing
- Cross-browser testing
- Mobile device testing
- Usability testing with real users

### Ongoing
- Monitor analytics for usability issues
- Collect user feedback
- Regular accessibility audits
- Stay updated with WCAG standards

## Tools Quick Reference

| Category | Tool | Platform | Cost | Best For |
|----------|------|----------|------|----------|
| Automated | axe DevTools | Browser | Free/Paid | Comprehensive scans |
| Automated | WAVE | Browser | Free | Visual feedback |
| Automated | Lighthouse | Chrome | Free | CI/CD integration |
| Automated | Pa11y | Node.js | Free | Bulk testing |
| Screen Reader | NVDA | Windows | Free | Windows testing |
| Screen Reader | JAWS | Windows | Paid | Professional testing |
| Screen Reader | VoiceOver | macOS/iOS | Free | Apple ecosystem |
| Contrast | WebAIM | Web | Free | Quick checks |
| Contrast | Stark | Design tools | Free/Paid | Design workflow |
| Color Blind | Color Oracle | Desktop | Free | Real-time simulation |
| Browser | BrowserStack | Web | Paid | Cross-browser |
| Usability | UserTesting | Web | Paid | User research |
| Analytics | Hotjar | Web | Free/Paid | Behavior analysis |

## Resources

- [W3C WAI Testing Tools List](https://www.w3.org/WAI/test-evaluate/tools/list/)
- [WebAIM Articles](https://webaim.org/articles/)
- [A11Y Project Resources](https://www.a11yproject.com/resources/)
- [Deque University](https://dequeuniversity.com/)
- [Google Web Fundamentals - Accessibility](https://developers.google.com/web/fundamentals/accessibility)
