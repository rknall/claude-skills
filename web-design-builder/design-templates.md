# Web Design Templates

Ready-to-use templates for common web design patterns.

## Landing Page Template

### Modern SaaS Landing Page

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Product Name - Tagline</title>
  <style>
    :root {
      --primary: #4F46E5;
      --primary-dark: #4338CA;
      --secondary: #10B981;
      --text: #1F2937;
      --text-light: #6B7280;
      --bg: #FFFFFF;
      --bg-alt: #F9FAFB;
    }

    *, *::before, *::after {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
      line-height: 1.6;
      color: var(--text);
      background: var(--bg);
    }

    .skip-link {
      position: absolute;
      top: -40px;
      left: 0;
      background: var(--primary);
      color: white;
      padding: 8px 16px;
      text-decoration: none;
      z-index: 100;
    }

    .skip-link:focus {
      top: 0;
    }

    .container {
      max-width: 1200px;
      margin: 0 auto;
      padding: 0 1rem;
    }

    /* Header */
    header {
      padding: 1rem 0;
      border-bottom: 1px solid #E5E7EB;
    }

    nav {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .logo {
      font-size: 1.5rem;
      font-weight: 700;
      color: var(--primary);
    }

    .nav-links {
      display: flex;
      gap: 2rem;
      list-style: none;
    }

    .nav-links a {
      color: var(--text);
      text-decoration: none;
      font-weight: 500;
    }

    .nav-links a:hover,
    .nav-links a:focus {
      color: var(--primary);
    }

    .btn {
      padding: 0.75rem 1.5rem;
      border-radius: 6px;
      font-weight: 600;
      text-decoration: none;
      display: inline-block;
      transition: all 0.2s;
    }

    .btn-primary {
      background: var(--primary);
      color: white;
    }

    .btn-primary:hover,
    .btn-primary:focus {
      background: var(--primary-dark);
    }

    /* Hero Section */
    .hero {
      padding: 4rem 0;
      text-align: center;
    }

    .hero h1 {
      font-size: clamp(2rem, 5vw, 3.5rem);
      margin-bottom: 1.5rem;
      font-weight: 800;
    }

    .hero p {
      font-size: 1.25rem;
      color: var(--text-light);
      margin-bottom: 2rem;
      max-width: 600px;
      margin-left: auto;
      margin-right: auto;
    }

    .hero-cta {
      display: flex;
      gap: 1rem;
      justify-content: center;
      flex-wrap: wrap;
    }

    /* Features Section */
    .features {
      padding: 4rem 0;
      background: var(--bg-alt);
    }

    .section-title {
      text-align: center;
      font-size: 2.5rem;
      margin-bottom: 3rem;
    }

    .features-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
      gap: 2rem;
    }

    .feature-card {
      background: white;
      padding: 2rem;
      border-radius: 12px;
      box-shadow: 0 1px 3px rgba(0,0,0,0.1);
    }

    .feature-icon {
      width: 48px;
      height: 48px;
      background: var(--primary);
      border-radius: 8px;
      display: flex;
      align-items: center;
      justify-content: center;
      margin-bottom: 1rem;
      color: white;
      font-size: 1.5rem;
    }

    .feature-card h3 {
      margin-bottom: 0.5rem;
    }

    /* Mobile Menu */
    .mobile-toggle {
      display: none;
      background: none;
      border: none;
      font-size: 1.5rem;
      cursor: pointer;
    }

    @media (max-width: 768px) {
      .nav-links {
        display: none;
        position: absolute;
        top: 100%;
        left: 0;
        right: 0;
        background: white;
        flex-direction: column;
        padding: 1rem;
        box-shadow: 0 4px 6px rgba(0,0,0,0.1);
      }

      .nav-links.active {
        display: flex;
      }

      .mobile-toggle {
        display: block;
      }
    }
  </style>
</head>
<body>
  <a href="#main" class="skip-link">Skip to main content</a>

  <header role="banner">
    <nav class="container" aria-label="Main navigation">
      <div class="logo">Logo</div>
      <button class="mobile-toggle" aria-label="Toggle menu" aria-expanded="false">
        ☰
      </button>
      <ul class="nav-links">
        <li><a href="#features">Features</a></li>
        <li><a href="#pricing">Pricing</a></li>
        <li><a href="#contact">Contact</a></li>
      </ul>
    </nav>
  </header>

  <main id="main" role="main">
    <section class="hero">
      <div class="container">
        <h1>Build Something Amazing</h1>
        <p>The best solution for your business needs. Fast, reliable, and easy to use.</p>
        <div class="hero-cta">
          <a href="#signup" class="btn btn-primary">Get Started</a>
          <a href="#demo" class="btn">Watch Demo</a>
        </div>
      </div>
    </section>

    <section id="features" class="features">
      <div class="container">
        <h2 class="section-title">Features</h2>
        <div class="features-grid">
          <div class="feature-card">
            <div class="feature-icon" aria-hidden="true">⚡</div>
            <h3>Fast Performance</h3>
            <p>Lightning-fast speeds that keep your users engaged.</p>
          </div>
          <div class="feature-card">
            <div class="feature-icon" aria-hidden="true">🔒</div>
            <h3>Secure by Default</h3>
            <p>Enterprise-grade security built into every layer.</p>
          </div>
          <div class="feature-card">
            <div class="feature-icon" aria-hidden="true">📊</div>
            <h3>Powerful Analytics</h3>
            <p>Gain insights with comprehensive analytics tools.</p>
          </div>
        </div>
      </div>
    </section>
  </main>

  <footer role="contentinfo">
    <div class="container">
      <p>&copy; 2025 Company Name. All rights reserved.</p>
    </div>
  </footer>

  <script>
    (function() {
      'use strict';

      const toggle = document.querySelector('.mobile-toggle');
      const nav = document.querySelector('.nav-links');

      if (toggle && nav) {
        toggle.addEventListener('click', function() {
          const expanded = this.getAttribute('aria-expanded') === 'true';
          this.setAttribute('aria-expanded', !expanded);
          nav.classList.toggle('active');
        });
      }
    })();
  </script>
</body>
</html>
```

## Form Template

### Accessible Contact Form

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Contact Us</title>
  <style>
    /* Add styles from above template */

    .form-container {
      max-width: 600px;
      margin: 4rem auto;
      padding: 2rem;
      background: white;
      border-radius: 12px;
      box-shadow: 0 4px 6px rgba(0,0,0,0.1);
    }

    .form-field {
      margin-bottom: 1.5rem;
    }

    label {
      display: block;
      margin-bottom: 0.5rem;
      font-weight: 600;
      color: var(--text);
    }

    label .required {
      color: #DC2626;
    }

    input,
    textarea,
    select {
      width: 100%;
      padding: 0.75rem;
      border: 2px solid #E5E7EB;
      border-radius: 6px;
      font-size: 1rem;
      font-family: inherit;
      transition: border-color 0.2s;
    }

    input:focus,
    textarea:focus,
    select:focus {
      outline: none;
      border-color: var(--primary);
      box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.1);
    }

    input[aria-invalid="true"],
    textarea[aria-invalid="true"] {
      border-color: #DC2626;
    }

    .hint {
      display: block;
      margin-top: 0.25rem;
      font-size: 0.875rem;
      color: var(--text-light);
    }

    .error {
      display: block;
      margin-top: 0.25rem;
      font-size: 0.875rem;
      color: #DC2626;
    }

    .success {
      padding: 1rem;
      background: #D1FAE5;
      color: #065F46;
      border-radius: 6px;
      margin-bottom: 1rem;
    }
  </style>
</head>
<body>
  <main class="form-container">
    <h1>Contact Us</h1>
    <p>Fill out the form below and we'll get back to you soon.</p>

    <form id="contact-form" novalidate>
      <div class="form-field">
        <label for="name">
          Name <span class="required" aria-label="required">*</span>
        </label>
        <input
          type="text"
          id="name"
          name="name"
          required
          aria-required="true"
          aria-describedby="name-error"
          autocomplete="name"
        />
        <span id="name-error" class="error" role="alert" hidden>
          Please enter your name
        </span>
      </div>

      <div class="form-field">
        <label for="email">
          Email <span class="required" aria-label="required">*</span>
        </label>
        <input
          type="email"
          id="email"
          name="email"
          required
          aria-required="true"
          aria-describedby="email-hint email-error"
          autocomplete="email"
        />
        <span id="email-hint" class="hint">
          We'll never share your email with anyone else.
        </span>
        <span id="email-error" class="error" role="alert" hidden>
          Please enter a valid email address
        </span>
      </div>

      <div class="form-field">
        <label for="message">
          Message <span class="required" aria-label="required">*</span>
        </label>
        <textarea
          id="message"
          name="message"
          rows="5"
          required
          aria-required="true"
          aria-describedby="message-error"
        ></textarea>
        <span id="message-error" class="error" role="alert" hidden>
          Please enter a message
        </span>
      </div>

      <button type="submit" class="btn btn-primary">Send Message</button>
    </form>
  </main>

  <script>
    (function() {
      'use strict';

      const form = document.getElementById('contact-form');

      form.addEventListener('submit', function(e) {
        e.preventDefault();

        // Clear previous errors
        const errors = form.querySelectorAll('.error');
        errors.forEach(error => error.hidden = true);

        const fields = form.querySelectorAll('input, textarea');
        fields.forEach(field => field.removeAttribute('aria-invalid'));

        let isValid = true;

        // Validate each required field
        const name = document.getElementById('name');
        if (!name.value.trim()) {
          showError(name, 'name-error');
          isValid = false;
        }

        const email = document.getElementById('email');
        if (!email.value.trim() || !isValidEmail(email.value)) {
          showError(email, 'email-error');
          isValid = false;
        }

        const message = document.getElementById('message');
        if (!message.value.trim()) {
          showError(message, 'message-error');
          isValid = false;
        }

        if (isValid) {
          // Submit form
          console.log('Form is valid, submitting...');
          showSuccess();
        }
      });

      function showError(field, errorId) {
        field.setAttribute('aria-invalid', 'true');
        const error = document.getElementById(errorId);
        error.hidden = false;
        field.focus();
      }

      function isValidEmail(email) {
        return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
      }

      function showSuccess() {
        const success = document.createElement('div');
        success.className = 'success';
        success.setAttribute('role', 'status');
        success.textContent = 'Thank you! Your message has been sent.';
        form.insertBefore(success, form.firstChild);
        form.reset();
      }
    })();
  </script>
</body>
</html>
```

## Dashboard Template

### Admin Dashboard Layout

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Dashboard</title>
  <style>
    /* Base styles */
    :root {
      --sidebar-width: 250px;
      --header-height: 60px;
    }

    body {
      margin: 0;
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      background: #F3F4F6;
    }

    /* Layout Grid */
    .dashboard {
      display: grid;
      grid-template-areas:
        "sidebar header"
        "sidebar main";
      grid-template-columns: var(--sidebar-width) 1fr;
      grid-template-rows: var(--header-height) 1fr;
      min-height: 100vh;
    }

    /* Sidebar */
    .sidebar {
      grid-area: sidebar;
      background: #1F2937;
      color: white;
      padding: 1rem;
    }

    .sidebar nav ul {
      list-style: none;
      padding: 0;
    }

    .sidebar nav a {
      display: block;
      padding: 0.75rem 1rem;
      color: #D1D5DB;
      text-decoration: none;
      border-radius: 6px;
      transition: background 0.2s;
    }

    .sidebar nav a:hover,
    .sidebar nav a[aria-current="page"] {
      background: #374151;
      color: white;
    }

    /* Header */
    .header {
      grid-area: header;
      background: white;
      border-bottom: 1px solid #E5E7EB;
      padding: 0 2rem;
      display: flex;
      align-items: center;
      justify-content: space-between;
    }

    /* Main Content */
    .main-content {
      grid-area: main;
      padding: 2rem;
    }

    .stats-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
      gap: 1.5rem;
      margin-bottom: 2rem;
    }

    .stat-card {
      background: white;
      padding: 1.5rem;
      border-radius: 8px;
      box-shadow: 0 1px 3px rgba(0,0,0,0.1);
    }

    .stat-value {
      font-size: 2rem;
      font-weight: 700;
      color: #1F2937;
    }

    .stat-label {
      color: #6B7280;
      font-size: 0.875rem;
    }

    @media (max-width: 768px) {
      .dashboard {
        grid-template-areas:
          "header"
          "main";
        grid-template-columns: 1fr;
        grid-template-rows: var(--header-height) 1fr;
      }

      .sidebar {
        position: fixed;
        left: -250px;
        top: 0;
        bottom: 0;
        z-index: 100;
        transition: left 0.3s;
      }

      .sidebar.active {
        left: 0;
      }
    }
  </style>
</head>
<body>
  <div class="dashboard">
    <aside class="sidebar" role="navigation" aria-label="Main navigation">
      <h2>Dashboard</h2>
      <nav>
        <ul>
          <li><a href="#overview" aria-current="page">Overview</a></li>
          <li><a href="#analytics">Analytics</a></li>
          <li><a href="#users">Users</a></li>
          <li><a href="#settings">Settings</a></li>
        </ul>
      </nav>
    </aside>

    <header class="header" role="banner">
      <h1>Dashboard Overview</h1>
      <button class="mobile-menu-toggle" aria-label="Toggle menu">☰</button>
    </header>

    <main class="main-content" role="main">
      <div class="stats-grid">
        <div class="stat-card">
          <div class="stat-value">1,234</div>
          <div class="stat-label">Total Users</div>
        </div>
        <div class="stat-card">
          <div class="stat-value">$12,345</div>
          <div class="stat-label">Revenue</div>
        </div>
        <div class="stat-card">
          <div class="stat-value">98.5%</div>
          <div class="stat-label">Uptime</div>
        </div>
      </div>
    </main>
  </div>
</body>
</html>
```

These templates provide solid starting points for common web design patterns with accessibility and responsiveness built in.
