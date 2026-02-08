---
name: tailwind-v4
description: Tailwind CSS v4 configuration and usage guidelines for Phoenix projects, plus UI/UX design principles. Use when styling Phoenix applications or building user interfaces.
---

# Tailwind CSS v4 & UI/UX Guidelines

## Tailwind CSS v4 Configuration

Tailwind CSS v4 **no longer needs a `tailwind.config.js`** and uses a new import syntax in `app.css`:

```css
@import "tailwindcss" source(none);
@source "../css";
@source "../js";
@source "../../lib/your_app_web";
```

- **Always use and maintain this import syntax** in the `app.css` file for projects generated with `phx.new`
- **Never** use `@apply` when writing raw CSS
- **Always** manually write your own tailwind-based components instead of using daisyUI for a unique, world-class design

## Asset Bundling

Out of the box **only the app.js and app.css bundles are supported**:
- You cannot reference an external vendor'd script `src` or link `href` in the layouts
- You must import vendor deps into app.js and app.css to use them
- **Never write inline `<script>` tags within templates**

## UI/UX Design Principles

- **Produce world-class UI designs** with a focus on usability, aesthetics, and modern design principles
- Implement **subtle micro-interactions** (e.g., button hover effects, smooth transitions)
- Ensure **clean typography, spacing, and layout balance** for a refined, premium look
- Focus on **delightful details** like hover effects, loading states, and smooth page transitions

## Best Practices

### Responsive Design
- Use Tailwind's responsive prefixes (`sm:`, `md:`, `lg:`, `xl:`, `2xl:`) for breakpoints
- Design mobile-first, then enhance for larger screens

### Color and Theming
- Use Tailwind's color scales consistently (e.g., `blue-500`, `blue-600` for primary actions)
- Define custom colors in the CSS using CSS custom properties if the default palette doesn't fit

### Typography
- Use Tailwind's font size scale (`text-sm`, `text-base`, `text-lg`, etc.) for consistent hierarchy
- Ensure sufficient contrast ratios for accessibility (WCAG 2.1 AA minimum)

### Spacing and Layout
- Use Tailwind's spacing scale consistently (`p-4`, `m-6`, `gap-3`, etc.)
- Prefer flexbox (`flex`) and grid (`grid`) for layouts over absolute positioning
- Use `max-w-*` containers to maintain readable line lengths

### Animations and Transitions
- Use Tailwind's built-in transitions: `transition-colors`, `transition-transform`, `transition-all`
- Keep animations subtle and purposeful (150-300ms duration)
- Provide `motion-reduce:` variants for accessibility

### Dark Mode
- Use Tailwind's `dark:` variant for dark mode support
- Test both modes to ensure readability and contrast
