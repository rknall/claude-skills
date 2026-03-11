# Framework-Specific Guidelines

## Tailwind CSS

**Best for**: Rapid prototyping, utility-first development, consistent design systems.

**CDN Setup:**
```html
<script src="https://cdn.tailwindcss.com"></script>
<script>
  tailwind.config = {
    theme: {
      extend: {
        colors: {
          primary: '#0066cc',
        }
      }
    }
  }
</script>
```

**Example:**
```html
<div class="container mx-auto px-4">
  <h1 class="text-4xl font-bold text-gray-900 mb-4">Welcome</h1>
  <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">
    Click Me
  </button>
</div>
```

## React

**Best for**: Component-based SPAs, complex interactions, large ecosystems.

**Example:**
```jsx
function DesignComponent() {
  const [isOpen, setIsOpen] = React.useState(false);

  return (
    <div className="container">
      <button
        onClick={() => setIsOpen(!isOpen)}
        aria-expanded={isOpen}
      >
        Toggle
      </button>
      {isOpen && (
        <div className="content">Content here</div>
      )}
    </div>
  );
}
```

## Alpine.js

**Best for**: Lightweight reactivity (15KB), no build step, progressive enhancement.

**Example:**
```html
<div x-data="{ open: false }">
  <button @click="open = !open" :aria-expanded="open">
    Toggle
  </button>
  <div x-show="open" x-transition>
    Content here
  </div>
</div>
```

## Vue

**Best for**: Progressive adoption, template-based components, moderate complexity apps.

Use the Vue CDN for prototypes or the Vue CLI for production builds.
