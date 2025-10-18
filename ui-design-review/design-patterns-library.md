# Accessible Design Patterns Library

Reference implementations and best practices for common UI patterns with accessibility built-in.

## Navigation Patterns

### Skip Links

**Purpose:** Allow keyboard users to bypass repetitive navigation and jump to main content.

**HTML:**
```html
<!-- First focusable element on page -->
<a href="#main-content" class="skip-link">Skip to main content</a>

<nav>
  <!-- Navigation content -->
</nav>

<main id="main-content">
  <!-- Main content -->
</main>
```

**CSS:**
```css
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: #000;
  color: #fff;
  padding: 8px;
  z-index: 100;
}

.skip-link:focus {
  top: 0;
}
```

**Accessibility:**
- Must be first focusable element
- Visible on keyboard focus
- Jumps to main content area with `id="main-content"`

### Responsive Navigation Menu

**HTML:**
```html
<nav aria-label="Main navigation">
  <button
    aria-expanded="false"
    aria-controls="main-menu"
    class="menu-toggle"
  >
    <span class="sr-only">Menu</span>
    <svg aria-hidden="true"><!-- hamburger icon --></svg>
  </button>

  <ul id="main-menu" class="nav-menu">
    <li><a href="/home" aria-current="page">Home</a></li>
    <li><a href="/about">About</a></li>
    <li><a href="/contact">Contact</a></li>
  </ul>
</nav>
```

**JavaScript:**
```javascript
const menuToggle = document.querySelector('.menu-toggle');
const menu = document.querySelector('#main-menu');

menuToggle.addEventListener('click', () => {
  const isExpanded = menuToggle.getAttribute('aria-expanded') === 'true';
  menuToggle.setAttribute('aria-expanded', !isExpanded);
  menu.hidden = isExpanded;
});

// Close on Escape key
document.addEventListener('keydown', (e) => {
  if (e.key === 'Escape' && !menu.hidden) {
    menuToggle.setAttribute('aria-expanded', 'false');
    menu.hidden = true;
    menuToggle.focus();
  }
});
```

**Accessibility:**
- `aria-label` on nav for multiple navigation regions
- `aria-expanded` indicates menu state
- `aria-current="page"` for current page
- Escape key closes menu
- Focus returns to toggle button when closed

### Breadcrumbs

**HTML:**
```html
<nav aria-label="Breadcrumb">
  <ol class="breadcrumb">
    <li><a href="/">Home</a></li>
    <li><a href="/products">Products</a></li>
    <li><a href="/products/shoes">Shoes</a></li>
    <li aria-current="page">Running Shoes</li>
  </ol>
</nav>
```

**CSS:**
```css
.breadcrumb {
  display: flex;
  gap: 0.5rem;
  list-style: none;
}

.breadcrumb li:not(:last-child)::after {
  content: '/';
  margin-left: 0.5rem;
  color: #666;
}

.breadcrumb [aria-current="page"] {
  font-weight: bold;
  color: #333;
}
```

**Accessibility:**
- Use `<nav>` with descriptive `aria-label`
- Use ordered list `<ol>` for hierarchy
- `aria-current="page"` for current location
- Last item is not a link

## Form Patterns

### Accessible Form Input

**HTML:**
```html
<div class="form-field">
  <label for="email">
    Email address
    <span aria-label="required">*</span>
  </label>
  <input
    type="email"
    id="email"
    name="email"
    autocomplete="email"
    required
    aria-required="true"
    aria-describedby="email-hint email-error"
  />
  <span id="email-hint" class="hint">
    We'll never share your email with anyone else.
  </span>
  <span id="email-error" class="error" role="alert" hidden>
    Please enter a valid email address.
  </span>
</div>
```

**CSS:**
```css
.form-field {
  margin-bottom: 1.5rem;
}

label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 500;
}

input {
  display: block;
  width: 100%;
  padding: 0.5rem;
  border: 2px solid #ccc;
  border-radius: 4px;
  font-size: 1rem;
}

input:focus {
  outline: 2px solid #0066cc;
  outline-offset: 2px;
  border-color: #0066cc;
}

input[aria-invalid="true"] {
  border-color: #d32f2f;
}

.hint {
  display: block;
  margin-top: 0.25rem;
  font-size: 0.875rem;
  color: #666;
}

.error {
  display: block;
  margin-top: 0.25rem;
  font-size: 0.875rem;
  color: #d32f2f;
}
```

**JavaScript:**
```javascript
const emailInput = document.getElementById('email');
const emailError = document.getElementById('email-error');

emailInput.addEventListener('blur', () => {
  if (!emailInput.validity.valid) {
    emailInput.setAttribute('aria-invalid', 'true');
    emailError.hidden = false;
  } else {
    emailInput.removeAttribute('aria-invalid');
    emailError.hidden = true;
  }
});
```

**Accessibility:**
- Label explicitly associated with input via `for`/`id`
- Required indicator in label (not just asterisk)
- `autocomplete` attribute for auto-fill
- `aria-describedby` links to hint and error text
- `aria-invalid` when validation fails
- Error message has `role="alert"` for announcement

### Radio Button Group

**HTML:**
```html
<fieldset>
  <legend>Choose your shipping method</legend>
  <div class="radio-group">
    <div class="radio-option">
      <input
        type="radio"
        id="standard"
        name="shipping"
        value="standard"
        checked
      />
      <label for="standard">
        Standard (3-5 business days)
      </label>
    </div>
    <div class="radio-option">
      <input
        type="radio"
        id="express"
        name="shipping"
        value="express"
      />
      <label for="express">
        Express (1-2 business days)
      </label>
    </div>
  </div>
</fieldset>
```

**Accessibility:**
- `<fieldset>` groups related radio buttons
- `<legend>` provides group label
- Each radio has explicit label
- Keyboard navigation with arrow keys (native behavior)

### Toggle Switch

**HTML:**
```html
<div class="toggle-switch">
  <input
    type="checkbox"
    id="notifications"
    role="switch"
    aria-checked="false"
  />
  <label for="notifications">
    Enable notifications
  </label>
</div>
```

**CSS:**
```css
.toggle-switch {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.toggle-switch input[type="checkbox"] {
  appearance: none;
  position: relative;
  width: 44px;
  height: 24px;
  background: #ccc;
  border-radius: 12px;
  cursor: pointer;
  transition: background 0.3s;
}

.toggle-switch input[type="checkbox"]::before {
  content: '';
  position: absolute;
  top: 2px;
  left: 2px;
  width: 20px;
  height: 20px;
  background: white;
  border-radius: 50%;
  transition: transform 0.3s;
}

.toggle-switch input[type="checkbox"]:checked {
  background: #0066cc;
}

.toggle-switch input[type="checkbox"]:checked::before {
  transform: translateX(20px);
}

.toggle-switch input[type="checkbox"]:focus {
  outline: 2px solid #0066cc;
  outline-offset: 2px;
}
```

**Accessibility:**
- `role="switch"` indicates toggle behavior
- `aria-checked` reflects state
- Minimum 44x44px touch target
- Clear focus indicator
- Label describes purpose

## Modal Patterns

### Accessible Modal Dialog

**HTML:**
```html
<button id="open-modal">Open Dialog</button>

<div
  id="modal"
  role="dialog"
  aria-modal="true"
  aria-labelledby="modal-title"
  aria-describedby="modal-description"
  hidden
  class="modal-overlay"
>
  <div class="modal-content">
    <h2 id="modal-title">Confirm Action</h2>
    <p id="modal-description">
      Are you sure you want to delete this item? This action cannot be undone.
    </p>
    <div class="modal-actions">
      <button id="confirm-btn">Delete</button>
      <button id="cancel-btn">Cancel</button>
    </div>
    <button
      id="close-modal"
      aria-label="Close dialog"
      class="close-button"
    >
      &times;
    </button>
  </div>
</div>
```

**CSS:**
```css
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  position: relative;
  background: white;
  padding: 2rem;
  border-radius: 8px;
  max-width: 500px;
  width: 90%;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.close-button {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: none;
  border: none;
  font-size: 2rem;
  cursor: pointer;
  padding: 0;
  width: 44px;
  height: 44px;
}
```

**JavaScript:**
```javascript
const modal = document.getElementById('modal');
const openBtn = document.getElementById('open-modal');
const closeBtn = document.getElementById('close-modal');
const cancelBtn = document.getElementById('cancel-btn');
let lastFocusedElement;

// Open modal
openBtn.addEventListener('click', () => {
  lastFocusedElement = document.activeElement;
  modal.hidden = false;

  // Focus first focusable element
  const firstFocusable = modal.querySelector('button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])');
  firstFocusable?.focus();

  // Trap focus
  trapFocus(modal);
});

// Close modal
function closeModal() {
  modal.hidden = true;
  lastFocusedElement?.focus();
}

closeBtn.addEventListener('click', closeModal);
cancelBtn.addEventListener('click', closeModal);

// Close on Escape
document.addEventListener('keydown', (e) => {
  if (e.key === 'Escape' && !modal.hidden) {
    closeModal();
  }
});

// Close on overlay click
modal.addEventListener('click', (e) => {
  if (e.target === modal) {
    closeModal();
  }
});

// Focus trap
function trapFocus(element) {
  const focusableElements = element.querySelectorAll(
    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
  );
  const firstFocusable = focusableElements[0];
  const lastFocusable = focusableElements[focusableElements.length - 1];

  element.addEventListener('keydown', (e) => {
    if (e.key !== 'Tab') return;

    if (e.shiftKey) {
      if (document.activeElement === firstFocusable) {
        lastFocusable.focus();
        e.preventDefault();
      }
    } else {
      if (document.activeElement === lastFocusable) {
        firstFocusable.focus();
        e.preventDefault();
      }
    }
  });
}
```

**Accessibility:**
- `role="dialog"` and `aria-modal="true"`
- `aria-labelledby` references dialog title
- `aria-describedby` references dialog description
- Focus moved to modal when opened
- Focus trapped within modal
- Escape key closes modal
- Focus returned to trigger on close
- Close button has `aria-label`

## Button Patterns

### Icon Button

**HTML:**
```html
<button aria-label="Delete item" class="icon-button">
  <svg aria-hidden="true" focusable="false">
    <!-- trash icon -->
  </svg>
</button>
```

**Accessibility:**
- `aria-label` provides accessible name
- `aria-hidden="true"` hides icon from screen readers
- `focusable="false"` prevents icon from receiving focus

### Loading Button

**HTML:**
```html
<button
  id="submit-btn"
  aria-live="polite"
  aria-busy="false"
>
  <span class="button-text">Submit</span>
  <span class="spinner" hidden aria-hidden="true"></span>
</button>
```

**JavaScript:**
```javascript
const submitBtn = document.getElementById('submit-btn');
const buttonText = submitBtn.querySelector('.button-text');
const spinner = submitBtn.querySelector('.spinner');

submitBtn.addEventListener('click', async () => {
  // Start loading
  submitBtn.setAttribute('aria-busy', 'true');
  submitBtn.disabled = true;
  buttonText.textContent = 'Submitting...';
  spinner.hidden = false;

  // Simulate async operation
  await fetch('/api/submit');

  // End loading
  submitBtn.setAttribute('aria-busy', 'false');
  submitBtn.disabled = false;
  buttonText.textContent = 'Submit';
  spinner.hidden = true;
});
```

**Accessibility:**
- `aria-busy` indicates loading state
- Button text changes to describe current state
- Button disabled during loading
- Spinner has `aria-hidden="true"`

## Dropdown/Select Patterns

### Custom Dropdown (Combobox)

**HTML:**
```html
<div class="combobox-wrapper">
  <label id="combo-label" for="combo-input">
    Choose a fruit
  </label>
  <div class="combobox">
    <input
      type="text"
      id="combo-input"
      role="combobox"
      aria-autocomplete="list"
      aria-expanded="false"
      aria-controls="combo-listbox"
      aria-labelledby="combo-label"
    />
    <ul
      id="combo-listbox"
      role="listbox"
      aria-labelledby="combo-label"
      hidden
    >
      <li role="option" id="option-1">Apple</li>
      <li role="option" id="option-2">Banana</li>
      <li role="option" id="option-3">Cherry</li>
    </ul>
  </div>
</div>
```

**JavaScript:**
```javascript
const combobox = document.getElementById('combo-input');
const listbox = document.getElementById('combo-listbox');
const options = listbox.querySelectorAll('[role="option"]');
let activeIndex = -1;

// Open listbox
combobox.addEventListener('focus', () => {
  combobox.setAttribute('aria-expanded', 'true');
  listbox.hidden = false;
});

// Keyboard navigation
combobox.addEventListener('keydown', (e) => {
  switch(e.key) {
    case 'ArrowDown':
      e.preventDefault();
      activeIndex = Math.min(activeIndex + 1, options.length - 1);
      updateActiveOption();
      break;
    case 'ArrowUp':
      e.preventDefault();
      activeIndex = Math.max(activeIndex - 1, 0);
      updateActiveOption();
      break;
    case 'Enter':
      if (activeIndex >= 0) {
        selectOption(options[activeIndex]);
      }
      break;
    case 'Escape':
      closeListbox();
      break;
  }
});

function updateActiveOption() {
  options.forEach((option, index) => {
    if (index === activeIndex) {
      option.setAttribute('aria-selected', 'true');
      combobox.setAttribute('aria-activedescendant', option.id);
      option.scrollIntoView({ block: 'nearest' });
    } else {
      option.removeAttribute('aria-selected');
    }
  });
}

function selectOption(option) {
  combobox.value = option.textContent;
  closeListbox();
}

function closeListbox() {
  combobox.setAttribute('aria-expanded', 'false');
  listbox.hidden = true;
  activeIndex = -1;
}
```

**Accessibility:**
- `role="combobox"` for input
- `aria-expanded` indicates dropdown state
- `aria-controls` links to listbox
- `aria-activedescendant` tracks active option
- Arrow keys navigate options
- Enter selects option
- Escape closes dropdown

## Alert Patterns

### Success Message

**HTML:**
```html
<div role="status" aria-live="polite" class="alert alert-success">
  <svg aria-hidden="true"><!-- checkmark icon --></svg>
  <span>Your changes have been saved successfully.</span>
</div>
```

### Error Message

**HTML:**
```html
<div role="alert" aria-live="assertive" class="alert alert-error">
  <svg aria-hidden="true"><!-- error icon --></svg>
  <span>An error occurred. Please try again.</span>
</div>
```

**Accessibility:**
- `role="status"` for non-critical updates
- `role="alert"` for critical messages
- `aria-live="polite"` waits for pause
- `aria-live="assertive"` interrupts immediately
- Icons are decorative (`aria-hidden`)

## Accordion Pattern

**HTML:**
```html
<div class="accordion">
  <h3>
    <button
      aria-expanded="false"
      aria-controls="panel-1"
      id="accordion-1"
      class="accordion-trigger"
    >
      Section 1
      <span class="accordion-icon" aria-hidden="true">+</span>
    </button>
  </h3>
  <div
    id="panel-1"
    role="region"
    aria-labelledby="accordion-1"
    class="accordion-panel"
    hidden
  >
    <p>Content for section 1.</p>
  </div>
</div>
```

**JavaScript:**
```javascript
const triggers = document.querySelectorAll('.accordion-trigger');

triggers.forEach(trigger => {
  trigger.addEventListener('click', () => {
    const expanded = trigger.getAttribute('aria-expanded') === 'true';
    const panel = document.getElementById(trigger.getAttribute('aria-controls'));
    const icon = trigger.querySelector('.accordion-icon');

    trigger.setAttribute('aria-expanded', !expanded);
    panel.hidden = expanded;
    icon.textContent = expanded ? '+' : '−';
  });
});
```

**Accessibility:**
- Button wraps heading text
- `aria-expanded` indicates state
- `aria-controls` links to panel
- Panel has `role="region"`
- `aria-labelledby` links panel to heading
- Icon is decorative

## Best Practices Summary

### General Principles
1. Use semantic HTML first
2. Add ARIA only when semantic HTML isn't sufficient
3. Ensure keyboard accessibility for all interactions
4. Provide visible focus indicators
5. Test with actual assistive technologies
6. Don't rely on color alone
7. Maintain proper heading hierarchy
8. Provide alternative text for images
9. Ensure sufficient color contrast
10. Support screen reader announcements

### Common Mistakes to Avoid
- ❌ Removing focus outlines without replacement
- ❌ Using `<div>` or `<span>` for buttons
- ❌ Placeholder text as labels
- ❌ Click handlers on non-interactive elements
- ❌ Missing alt text on images
- ❌ Poor color contrast
- ❌ Keyboard traps
- ❌ Auto-playing audio/video
- ❌ Time limits without extensions
- ❌ Unlabeled form controls

### Testing Checklist
- ✅ Keyboard-only navigation
- ✅ Screen reader testing
- ✅ Automated accessibility scans
- ✅ Color contrast verification
- ✅ Zoom to 200%
- ✅ Focus indicator visibility
- ✅ Touch target sizes
- ✅ Error message clarity
- ✅ Form label associations
- ✅ Semantic HTML validation
