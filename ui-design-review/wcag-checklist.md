# WCAG 2.1/2.2 Compliance Checklist

A comprehensive checklist for evaluating Web Content Accessibility Guidelines compliance.

## How to Use This Checklist

- Check each item during design review
- Note the WCAG level: A (minimum), AA (recommended), AAA (enhanced)
- Document violations with specific examples
- Provide remediation steps for each issue

## Perceivable

Information and user interface components must be presentable to users in ways they can perceive.

### 1.1 Text Alternatives (Level A)

Provide text alternatives for any non-text content.

- [ ] **1.1.1** All images have appropriate alt text
  - Informative images describe the content
  - Functional images (buttons, links) describe the action
  - Decorative images have empty alt="" attribute
  - Complex images have long descriptions

### 1.2 Time-based Media (Level A/AA)

Provide alternatives for time-based media.

- [ ] **1.2.1** Audio-only and video-only content has text alternatives (A)
- [ ] **1.2.2** Captions are provided for all pre-recorded audio in video (A)
- [ ] **1.2.3** Audio description or text alternative for pre-recorded video (A)
- [ ] **1.2.4** Captions are provided for all live audio (AA)
- [ ] **1.2.5** Audio description for all pre-recorded video (AA)

### 1.3 Adaptable (Level A/AA)

Create content that can be presented in different ways without losing information.

- [ ] **1.3.1** Info and relationships can be programmatically determined
  - Semantic HTML elements used correctly (headings, lists, tables)
  - Form labels are associated with inputs
  - ARIA landmarks identify page regions
  - Data tables use proper markup (th, scope, headers)

- [ ] **1.3.2** Meaningful sequence is preserved without CSS
  - Content order is logical when stylesheets are disabled
  - Reading order matches visual order
  - Tab order is logical

- [ ] **1.3.3** Instructions don't rely solely on sensory characteristics
  - Don't use "Click the round button" (use "Click the Save button")
  - Don't use "See the box on the right" (use "See the Related Articles section")
  - Don't rely only on color ("Click the green button" → "Click the Submit button")

- [ ] **1.3.4** Content works in portrait and landscape (AA)
  - No orientation lock (unless essential)
  - Content adapts to orientation changes

- [ ] **1.3.5** Input purpose can be programmatically determined (AA)
  - Autocomplete attributes used for common inputs
  - `autocomplete="name"`, `autocomplete="email"`, etc.

### 1.4 Distinguishable (Level A/AA/AAA)

Make it easier for users to see and hear content.

- [ ] **1.4.1** Color is not the only visual means of conveying information (A)
  - Links are distinguishable without color alone (underline or icon)
  - Form errors use icons or text, not just red color
  - Charts use patterns or labels in addition to color

- [ ] **1.4.2** Audio control is available (A)
  - Auto-playing audio can be paused, stopped, or muted
  - Auto-play lasts less than 3 seconds

- [ ] **1.4.3** Contrast ratio is at least 4.5:1 for normal text, 3:1 for large text (AA)
  - Normal text: 4.5:1 minimum
  - Large text (18pt+/14pt bold+): 3:1 minimum
  - UI components and graphics: 3:1 minimum

- [ ] **1.4.4** Text can be resized up to 200% without loss of content or functionality (AA)
  - No horizontal scrolling at 200% zoom
  - Content doesn't overlap or get cut off
  - All functionality remains available

- [ ] **1.4.5** Images of text are avoided (AA)
  - Use actual text instead of text in images
  - Exceptions: logos, essential images (charts, screenshots)

- [ ] **1.4.6** Contrast ratio is at least 7:1 for normal text, 4.5:1 for large text (AAA)

- [ ] **1.4.10** Content reflows without horizontal scrolling at 320px width (AA)
  - Content adapts to small viewports
  - No two-dimensional scrolling required
  - Exceptions: images, maps, data tables, complex interfaces

- [ ] **1.4.11** Non-text contrast is at least 3:1 (AA)
  - UI components (buttons, inputs, controls)
  - Graphical objects (icons, charts)
  - Focus indicators

- [ ] **1.4.12** Text spacing can be adjusted without loss of content (AA)
  - Line height: 1.5x font size minimum
  - Paragraph spacing: 2x font size minimum
  - Letter spacing: 0.12x font size minimum
  - Word spacing: 0.16x font size minimum

- [ ] **1.4.13** Content on hover or focus is dismissible, hoverable, and persistent (AA)
  - Tooltips/popovers can be dismissed (Esc key)
  - Mouse can hover over the tooltip content
  - Content remains visible until user dismisses it

## Operable

User interface components and navigation must be operable.

### 2.1 Keyboard Accessible (Level A)

Make all functionality available from a keyboard.

- [ ] **2.1.1** All functionality is keyboard accessible
  - No mouse-only interactions
  - Custom controls have keyboard support
  - All interactive elements can be activated via keyboard

- [ ] **2.1.2** No keyboard traps exist
  - Users can tab through all content and back out
  - If focus is trapped (e.g., modal), there's a documented way to escape
  - Instructions provided if non-standard keyboard navigation

- [ ] **2.1.4** Single-character keyboard shortcuts can be turned off or remapped (A)
  - Shortcuts only active when component has focus
  - Can be disabled or customized

### 2.2 Enough Time (Level A/AAA)

Provide users enough time to read and use content.

- [ ] **2.2.1** Time limits can be adjusted (A)
  - User can turn off, adjust, or extend time limits
  - Warning given before time expires
  - At least 20 seconds to respond to warning

- [ ] **2.2.2** Moving, blinking, scrolling content can be paused (A)
  - Carousels have pause button
  - Auto-scrolling can be stopped
  - Exceptions: essential animations under 5 seconds

### 2.3 Seizures and Physical Reactions (Level A/AAA)

Do not design content that can cause seizures.

- [ ] **2.3.1** Content doesn't flash more than 3 times per second (A)
  - No rapidly flashing content
  - Animation effects are below threshold

### 2.4 Navigable (Level A/AA/AAA)

Provide ways to help users navigate and find content.

- [ ] **2.4.1** Skip links allow bypassing repeated content (A)
  - "Skip to main content" link is first focusable element
  - Skip links to bypass navigation, search, etc.

- [ ] **2.4.2** Page titles are descriptive and unique (A)
  - Every page has a unique, descriptive `<title>`
  - Title identifies page purpose
  - Format: "Page Name - Site Name"

- [ ] **2.4.3** Focus order preserves meaning and operability (A)
  - Tab order is logical
  - Focus order matches visual order
  - No unexpected focus changes

- [ ] **2.4.4** Link purpose is clear from link text or context (A)
  - Avoid "Click here" or "Read more"
  - Link text describes destination
  - Context provides clarity if needed

- [ ] **2.4.5** Multiple ways to locate pages exist (AA)
  - Search functionality
  - Site map
  - Navigation menu
  - Breadcrumbs

- [ ] **2.4.6** Headings and labels are descriptive (AA)
  - Headings clearly describe content
  - Form labels are clear and descriptive

- [ ] **2.4.7** Focus indicator is visible (AA)
  - Clear outline or highlight on focused elements
  - 3:1 contrast ratio for focus indicators
  - Not removed with `outline: none` without replacement

- [ ] **2.4.8** Information about user's location is available (AAA)
  - Breadcrumbs show current location
  - Current nav item is highlighted
  - Page title indicates location

### 2.5 Input Modalities (Level A/AAA)

Make it easier for users to operate functionality through various inputs.

- [ ] **2.5.1** All functionality works with pointer gestures (A)
  - Multi-point or path-based gestures have single-pointer alternatives
  - Pinch-to-zoom has zoom buttons
  - Swipe gestures have button alternatives

- [ ] **2.5.2** Pointer cancellation prevents accidental activation (A)
  - Actions occur on up-event (mouseup/touchend), not down-event
  - Or action can be aborted/undone

- [ ] **2.5.3** Labels match accessible names (A)
  - Visible label text is included in accessible name
  - "Submit" button has accessible name "Submit"

- [ ] **2.5.4** Motion actuation can be disabled (A)
  - Shaking device to undo can be turned off
  - UI alternatives provided

- [ ] **2.5.5** Target size is at least 44x44 CSS pixels (AAA)
  - Touch targets meet minimum size
  - Sufficient spacing between targets

## Understandable

Information and user interface operation must be understandable.

### 3.1 Readable (Level A/AA/AAA)

Make text content readable and understandable.

- [ ] **3.1.1** Page language is identified (A)
  - `<html lang="en">` attribute is set
  - Correct language code used

- [ ] **3.1.2** Language changes are marked up (AA)
  - `lang` attribute used for content in different language
  - `<span lang="es">Hola</span>`

- [ ] **3.1.3** Unusual words are defined (AAA)
  - Jargon, idioms, and slang are explained
  - Glossary or inline definitions provided

### 3.2 Predictable (Level A/AA)

Make web pages appear and operate in predictable ways.

- [ ] **3.2.1** Focus doesn't trigger unexpected changes (A)
  - Receiving focus doesn't open popups
  - Focus doesn't redirect to another page
  - Focus doesn't change context without warning

- [ ] **3.2.2** Input doesn't trigger unexpected changes (A)
  - Entering data doesn't automatically submit forms
  - Selecting radio button doesn't redirect
  - Changes require explicit submit/save action
  - Or users are warned before context changes

- [ ] **3.2.3** Navigation is consistent across pages (AA)
  - Navigation appears in same location on each page
  - Menu items in same order

- [ ] **3.2.4** Components are identified consistently (AA)
  - Same icons/labels used for same functions
  - Search icon always means search
  - "Save" button always labeled "Save"

### 3.3 Input Assistance (Level A/AA/AAA)

Help users avoid and correct mistakes.

- [ ] **3.3.1** Form errors are identified (A)
  - Error messages clearly identify which field has error
  - Error is described in text, not just color

- [ ] **3.3.2** Labels and instructions are provided (A)
  - Every input has a label
  - Required fields are marked
  - Format requirements are explained

- [ ] **3.3.3** Error suggestions are offered (AA)
  - Provide helpful suggestions to fix errors
  - "Email format incorrect. Example: user@example.com"
  - Suggest corrections for misspellings

- [ ] **3.3.4** Critical actions can be reversed, checked, or confirmed (AA)
  - Financial transactions can be reversed
  - Data deletion requires confirmation
  - Form data can be reviewed before submission

## Robust

Content must be robust enough to work with current and future technologies.

### 4.1 Compatible (Level A/AA)

Maximize compatibility with current and future user agents, including assistive technologies.

- [ ] **4.1.1** HTML is valid and properly nested (A)
  - Opening and closing tags match
  - Elements nested correctly
  - IDs are unique

- [ ] **4.1.2** Name, role, value are programmatically determinable (A)
  - Form inputs have labels
  - Custom controls have ARIA roles
  - State changes are communicated (aria-expanded, aria-checked)

- [ ] **4.1.3** Status messages can be perceived without focus (AA)
  - Success messages use `role="status"` or `aria-live="polite"`
  - Urgent messages use `role="alert"` or `aria-live="assertive"`
  - Loading states announced to screen readers

## Additional Best Practices

Beyond WCAG requirements, consider these best practices:

### Screen Reader Support
- [ ] ARIA landmarks define page structure (banner, navigation, main, contentinfo)
- [ ] Headings are used hierarchically (h1 → h2 → h3, no skipping)
- [ ] Images in links are given appropriate alt text
- [ ] Icon-only buttons have accessible labels
- [ ] Visually hidden text provides context where needed

### Keyboard Navigation
- [ ] Tab order is logical and efficient
- [ ] Escape key closes modals and dropdowns
- [ ] Arrow keys navigate within component groups
- [ ] Enter/Space activate buttons and links
- [ ] Custom keyboard shortcuts are documented

### Forms
- [ ] Field labels are above or left of inputs (not placeholder text)
- [ ] Error messages appear near the field with the error
- [ ] Required fields marked with asterisk AND text "(required)"
- [ ] Success confirmation after form submission
- [ ] Form data preserved if validation fails

### Focus Management
- [ ] Focus moved to modal when opened
- [ ] Focus returned to trigger when modal closes
- [ ] Focus not lost when content updates dynamically
- [ ] Focus visible and meets 3:1 contrast ratio

### Content Structure
- [ ] Only one h1 per page
- [ ] Headings don't skip levels
- [ ] Lists used for list content (ul, ol, dl)
- [ ] Tables used for tabular data only
- [ ] Table headers (th) identify rows and columns

## Testing Checklist

- [ ] Automated testing with axe DevTools
- [ ] Automated testing with WAVE
- [ ] Lighthouse accessibility audit
- [ ] Keyboard-only navigation test
- [ ] Screen reader test (NVDA/JAWS/VoiceOver)
- [ ] Color contrast verification
- [ ] 200% zoom test
- [ ] Responsive design test (320px width minimum)
- [ ] Color blindness simulation
- [ ] Touch target size verification

## Resources

- [WCAG 2.1 Official Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [WCAG 2.2 Updates](https://www.w3.org/WAI/WCAG22/quickref/)
- [WebAIM WCAG Checklist](https://webaim.org/standards/wcag/checklist)
- [A11Y Project Checklist](https://www.a11yproject.com/checklist/)
- [axe DevTools Browser Extension](https://www.deque.com/axe/devtools/)
