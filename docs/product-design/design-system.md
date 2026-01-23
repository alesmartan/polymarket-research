# Design System
## Prediction Market Platform

**Version:** 1.0
**Last Updated:** January 2026
**Status:** Active
**Owner:** Design Team

---

## Table of Contents

1. [Brand Identity Guidelines](#1-brand-identity-guidelines)
2. [Color Palette](#2-color-palette)
3. [Typography Scale](#3-typography-scale)
4. [Spacing and Grid System](#4-spacing-and-grid-system)
5. [Component Library](#5-component-library)
6. [Design Tokens](#6-design-tokens)
7. [Accessibility Guidelines](#7-accessibility-guidelines)
8. [Animation Principles](#8-animation-principles)
9. [Icon System](#9-icon-system)

---

## 1. Brand Identity Guidelines

### Brand Personality

| Attribute | Description | Expression |
|-----------|-------------|------------|
| **Trustworthy** | Users trust us with their money and data | Clean design, clear information, security indicators |
| **Intelligent** | We aggregate wisdom from the crowd | Data-forward design, smart defaults, insightful analytics |
| **Accessible** | Everyone can participate | Simple language, intuitive flows, inclusive design |
| **Dynamic** | Markets move fast, so do we | Real-time updates, responsive interactions, live data |
| **Modern** | Cutting-edge technology, refined aesthetics | Contemporary design patterns, smooth animations |

### Brand Voice

**Do:**
- Use clear, jargon-free language
- Be direct and confident
- Explain complex concepts simply
- Celebrate user successes
- Acknowledge uncertainty honestly

**Don't:**
- Use aggressive trading language ("HODL", "to the moon")
- Overpromise returns or outcomes
- Be condescending to beginners
- Use fear-based messaging
- Hide risks or fees

### Logo Usage

**Primary Logo:**
- Minimum clear space: 1x logo height on all sides
- Minimum size: 32px height (digital), 12mm (print)
- Always use approved color variations

**Logo Variations:**
- Full color (primary)
- Monochrome white (dark backgrounds)
- Monochrome dark (light backgrounds)
- Icon only (constrained spaces)

### Imagery Style

**Photography:**
- Authentic, diverse representation
- Natural lighting
- Avoid stock photo cliches
- Focus on decision-making moments

**Illustrations:**
- Geometric, clean lines
- Consistent stroke weights
- Brand color palette
- Abstract data visualizations

---

## 2. Color Palette

### Light Mode

#### Primary Colors

```css
/* Primary - Used for CTAs, links, active states */
--color-primary-50: #EEF2FF;
--color-primary-100: #E0E7FF;
--color-primary-200: #C7D2FE;
--color-primary-300: #A5B4FC;
--color-primary-400: #818CF8;
--color-primary-500: #6366F1;  /* Main primary */
--color-primary-600: #4F46E5;
--color-primary-700: #4338CA;
--color-primary-800: #3730A3;
--color-primary-900: #312E81;
```

#### Semantic Colors

```css
/* Success - Profit, positive outcomes, confirmations */
--color-success-50: #ECFDF5;
--color-success-100: #D1FAE5;
--color-success-200: #A7F3D0;
--color-success-300: #6EE7B7;
--color-success-400: #34D399;
--color-success-500: #10B981;  /* Main success */
--color-success-600: #059669;
--color-success-700: #047857;

/* Error - Loss, negative outcomes, errors */
--color-error-50: #FEF2F2;
--color-error-100: #FEE2E2;
--color-error-200: #FECACA;
--color-error-300: #FCA5A5;
--color-error-400: #F87171;
--color-error-500: #EF4444;  /* Main error */
--color-error-600: #DC2626;
--color-error-700: #B91C1C;

/* Warning - Caution, pending states */
--color-warning-50: #FFFBEB;
--color-warning-100: #FEF3C7;
--color-warning-200: #FDE68A;
--color-warning-300: #FCD34D;
--color-warning-400: #FBBF24;
--color-warning-500: #F59E0B;  /* Main warning */
--color-warning-600: #D97706;
--color-warning-700: #B45309;

/* Info - Informational, neutral highlights */
--color-info-50: #EFF6FF;
--color-info-100: #DBEAFE;
--color-info-200: #BFDBFE;
--color-info-300: #93C5FD;
--color-info-400: #60A5FA;
--color-info-500: #3B82F6;  /* Main info */
--color-info-600: #2563EB;
--color-info-700: #1D4ED8;
```

#### Neutral Colors

```css
/* Neutrals - Text, backgrounds, borders */
--color-neutral-0: #FFFFFF;
--color-neutral-50: #F9FAFB;
--color-neutral-100: #F3F4F6;
--color-neutral-200: #E5E7EB;
--color-neutral-300: #D1D5DB;
--color-neutral-400: #9CA3AF;
--color-neutral-500: #6B7280;
--color-neutral-600: #4B5563;
--color-neutral-700: #374151;
--color-neutral-800: #1F2937;
--color-neutral-900: #111827;
--color-neutral-950: #030712;
```

### Dark Mode

```css
/* Dark mode semantic mappings */
[data-theme="dark"] {
  /* Backgrounds */
  --color-bg-primary: #0F0F0F;
  --color-bg-secondary: #171717;
  --color-bg-tertiary: #212121;
  --color-bg-elevated: #262626;

  /* Text */
  --color-text-primary: #FAFAFA;
  --color-text-secondary: #A3A3A3;
  --color-text-tertiary: #737373;
  --color-text-disabled: #525252;

  /* Borders */
  --color-border-primary: #262626;
  --color-border-secondary: #404040;
  --color-border-focus: #6366F1;

  /* Adjust primary for dark mode legibility */
  --color-primary-500: #818CF8;
  --color-primary-600: #6366F1;
}
```

### Trading-Specific Colors

```css
/* Buy/Sell - Yes/No */
--color-buy: #10B981;      /* Green for Yes/Buy */
--color-buy-hover: #059669;
--color-buy-bg: rgba(16, 185, 129, 0.1);

--color-sell: #EF4444;     /* Red for No/Sell */
--color-sell-hover: #DC2626;
--color-sell-bg: rgba(239, 68, 68, 0.1);

/* Probability gradient */
--color-prob-low: #EF4444;    /* 0-30% */
--color-prob-mid: #F59E0B;    /* 30-70% */
--color-prob-high: #10B981;   /* 70-100% */
```

### Color Accessibility

| Combination | Light Mode Ratio | Dark Mode Ratio | WCAG Level |
|-------------|------------------|-----------------|------------|
| Primary on white | 4.5:1 | N/A | AA |
| Text primary on bg | 12:1 | 15:1 | AAA |
| Text secondary on bg | 4.7:1 | 5.2:1 | AA |
| Success on white | 4.5:1 | N/A | AA |
| Error on white | 4.5:1 | N/A | AA |

---

## 3. Typography Scale

### Font Family

```css
/* Primary font - Interface text */
--font-family-sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;

/* Monospace - Numbers, code, addresses */
--font-family-mono: 'JetBrains Mono', 'SF Mono', 'Fira Code', monospace;
```

### Type Scale

```css
/* Font sizes - Using a 1.25 ratio (Major Third) */
--font-size-xs: 0.75rem;    /* 12px */
--font-size-sm: 0.875rem;   /* 14px */
--font-size-base: 1rem;     /* 16px */
--font-size-lg: 1.125rem;   /* 18px */
--font-size-xl: 1.25rem;    /* 20px */
--font-size-2xl: 1.5rem;    /* 24px */
--font-size-3xl: 1.875rem;  /* 30px */
--font-size-4xl: 2.25rem;   /* 36px */
--font-size-5xl: 3rem;      /* 48px */
--font-size-6xl: 3.75rem;   /* 60px */
```

### Font Weights

```css
--font-weight-normal: 400;
--font-weight-medium: 500;
--font-weight-semibold: 600;
--font-weight-bold: 700;
```

### Line Heights

```css
--line-height-none: 1;
--line-height-tight: 1.25;
--line-height-snug: 1.375;
--line-height-normal: 1.5;
--line-height-relaxed: 1.625;
--line-height-loose: 2;
```

### Typography Presets

```css
/* Headings */
.heading-1 {
  font-size: var(--font-size-4xl);
  font-weight: var(--font-weight-bold);
  line-height: var(--line-height-tight);
  letter-spacing: -0.02em;
}

.heading-2 {
  font-size: var(--font-size-3xl);
  font-weight: var(--font-weight-bold);
  line-height: var(--line-height-tight);
  letter-spacing: -0.01em;
}

.heading-3 {
  font-size: var(--font-size-2xl);
  font-weight: var(--font-weight-semibold);
  line-height: var(--line-height-snug);
}

.heading-4 {
  font-size: var(--font-size-xl);
  font-weight: var(--font-weight-semibold);
  line-height: var(--line-height-snug);
}

/* Body text */
.body-large {
  font-size: var(--font-size-lg);
  font-weight: var(--font-weight-normal);
  line-height: var(--line-height-relaxed);
}

.body-base {
  font-size: var(--font-size-base);
  font-weight: var(--font-weight-normal);
  line-height: var(--line-height-normal);
}

.body-small {
  font-size: var(--font-size-sm);
  font-weight: var(--font-weight-normal);
  line-height: var(--line-height-normal);
}

/* Utility text */
.caption {
  font-size: var(--font-size-xs);
  font-weight: var(--font-weight-medium);
  line-height: var(--line-height-normal);
  letter-spacing: 0.02em;
}

.label {
  font-size: var(--font-size-sm);
  font-weight: var(--font-weight-medium);
  line-height: var(--line-height-none);
}

/* Numbers and data */
.number-large {
  font-family: var(--font-family-mono);
  font-size: var(--font-size-3xl);
  font-weight: var(--font-weight-semibold);
  font-variant-numeric: tabular-nums;
}

.number-base {
  font-family: var(--font-family-mono);
  font-size: var(--font-size-base);
  font-weight: var(--font-weight-medium);
  font-variant-numeric: tabular-nums;
}
```

---

## 4. Spacing and Grid System

### Spacing Scale

```css
/* Base unit: 4px */
--space-0: 0;
--space-0.5: 0.125rem;  /* 2px */
--space-1: 0.25rem;     /* 4px */
--space-1.5: 0.375rem;  /* 6px */
--space-2: 0.5rem;      /* 8px */
--space-2.5: 0.625rem;  /* 10px */
--space-3: 0.75rem;     /* 12px */
--space-3.5: 0.875rem;  /* 14px */
--space-4: 1rem;        /* 16px */
--space-5: 1.25rem;     /* 20px */
--space-6: 1.5rem;      /* 24px */
--space-7: 1.75rem;     /* 28px */
--space-8: 2rem;        /* 32px */
--space-9: 2.25rem;     /* 36px */
--space-10: 2.5rem;     /* 40px */
--space-11: 2.75rem;    /* 44px */
--space-12: 3rem;       /* 48px */
--space-14: 3.5rem;     /* 56px */
--space-16: 4rem;       /* 64px */
--space-20: 5rem;       /* 80px */
--space-24: 6rem;       /* 96px */
--space-28: 7rem;       /* 112px */
--space-32: 8rem;       /* 128px */
```

### Grid System

```css
/* Container widths */
--container-sm: 640px;
--container-md: 768px;
--container-lg: 1024px;
--container-xl: 1280px;
--container-2xl: 1536px;

/* Grid configuration */
--grid-columns: 12;
--grid-gutter: var(--space-6);  /* 24px default */
--grid-margin: var(--space-4);   /* 16px mobile */

/* Responsive gutters */
@media (min-width: 768px) {
  :root {
    --grid-gutter: var(--space-8);  /* 32px tablet+ */
    --grid-margin: var(--space-8);
  }
}

@media (min-width: 1280px) {
  :root {
    --grid-gutter: var(--space-10); /* 40px desktop */
    --grid-margin: var(--space-12);
  }
}
```

### Layout Components

```css
/* Page container */
.container {
  width: 100%;
  max-width: var(--container-xl);
  margin-inline: auto;
  padding-inline: var(--grid-margin);
}

/* Grid */
.grid {
  display: grid;
  gap: var(--grid-gutter);
}

.grid-cols-1 { grid-template-columns: repeat(1, minmax(0, 1fr)); }
.grid-cols-2 { grid-template-columns: repeat(2, minmax(0, 1fr)); }
.grid-cols-3 { grid-template-columns: repeat(3, minmax(0, 1fr)); }
.grid-cols-4 { grid-template-columns: repeat(4, minmax(0, 1fr)); }
.grid-cols-6 { grid-template-columns: repeat(6, minmax(0, 1fr)); }
.grid-cols-12 { grid-template-columns: repeat(12, minmax(0, 1fr)); }

/* Flex utilities */
.flex { display: flex; }
.flex-col { flex-direction: column; }
.items-center { align-items: center; }
.justify-between { justify-content: space-between; }
.gap-2 { gap: var(--space-2); }
.gap-4 { gap: var(--space-4); }
.gap-6 { gap: var(--space-6); }
```

### Breakpoints

```css
/* Mobile-first breakpoints */
--breakpoint-sm: 640px;   /* Large phones */
--breakpoint-md: 768px;   /* Tablets */
--breakpoint-lg: 1024px;  /* Small laptops */
--breakpoint-xl: 1280px;  /* Desktops */
--breakpoint-2xl: 1536px; /* Large desktops */
```

---

## 5. Component Library

### Buttons

#### Button Variants

```css
/* Base button styles */
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-2);
  padding: var(--space-2.5) var(--space-4);
  font-size: var(--font-size-sm);
  font-weight: var(--font-weight-medium);
  line-height: var(--line-height-none);
  border-radius: var(--radius-lg);
  transition: all 150ms ease;
  cursor: pointer;
  border: none;
}

.btn:focus-visible {
  outline: 2px solid var(--color-primary-500);
  outline-offset: 2px;
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* Primary button */
.btn-primary {
  background: var(--color-primary-600);
  color: white;
}
.btn-primary:hover:not(:disabled) {
  background: var(--color-primary-700);
}

/* Secondary button */
.btn-secondary {
  background: var(--color-neutral-100);
  color: var(--color-neutral-900);
}
.btn-secondary:hover:not(:disabled) {
  background: var(--color-neutral-200);
}

/* Outline button */
.btn-outline {
  background: transparent;
  border: 1px solid var(--color-neutral-300);
  color: var(--color-neutral-700);
}
.btn-outline:hover:not(:disabled) {
  background: var(--color-neutral-50);
}

/* Ghost button */
.btn-ghost {
  background: transparent;
  color: var(--color-neutral-700);
}
.btn-ghost:hover:not(:disabled) {
  background: var(--color-neutral-100);
}

/* Buy/Sell buttons */
.btn-buy {
  background: var(--color-buy);
  color: white;
}
.btn-buy:hover:not(:disabled) {
  background: var(--color-buy-hover);
}

.btn-sell {
  background: var(--color-sell);
  color: white;
}
.btn-sell:hover:not(:disabled) {
  background: var(--color-sell-hover);
}
```

#### Button Sizes

```css
.btn-sm {
  padding: var(--space-1.5) var(--space-3);
  font-size: var(--font-size-xs);
}

.btn-md {
  padding: var(--space-2.5) var(--space-4);
  font-size: var(--font-size-sm);
}

.btn-lg {
  padding: var(--space-3) var(--space-6);
  font-size: var(--font-size-base);
}

.btn-xl {
  padding: var(--space-4) var(--space-8);
  font-size: var(--font-size-lg);
}
```

### Cards

```css
/* Base card */
.card {
  background: var(--color-bg-primary);
  border: 1px solid var(--color-border-primary);
  border-radius: var(--radius-xl);
  overflow: hidden;
}

/* Card sections */
.card-header {
  padding: var(--space-4) var(--space-5);
  border-bottom: 1px solid var(--color-border-primary);
}

.card-body {
  padding: var(--space-5);
}

.card-footer {
  padding: var(--space-4) var(--space-5);
  border-top: 1px solid var(--color-border-primary);
  background: var(--color-bg-secondary);
}

/* Interactive card */
.card-interactive {
  cursor: pointer;
  transition: all 150ms ease;
}
.card-interactive:hover {
  border-color: var(--color-border-secondary);
  box-shadow: var(--shadow-md);
}

/* Market card */
.card-market {
  display: flex;
  flex-direction: column;
  gap: var(--space-3);
  padding: var(--space-4);
}

.card-market-title {
  font-size: var(--font-size-base);
  font-weight: var(--font-weight-semibold);
  color: var(--color-text-primary);
  line-height: var(--line-height-snug);
}

.card-market-odds {
  display: flex;
  gap: var(--space-2);
}

.card-market-stat {
  font-size: var(--font-size-xs);
  color: var(--color-text-secondary);
}
```

### Modals

```css
/* Modal overlay */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: var(--space-4);
  z-index: 50;
  animation: fadeIn 150ms ease;
}

/* Modal container */
.modal {
  background: var(--color-bg-primary);
  border-radius: var(--radius-2xl);
  box-shadow: var(--shadow-2xl);
  max-width: 480px;
  width: 100%;
  max-height: 90vh;
  overflow: auto;
  animation: slideUp 200ms ease;
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: var(--space-5);
  border-bottom: 1px solid var(--color-border-primary);
}

.modal-title {
  font-size: var(--font-size-lg);
  font-weight: var(--font-weight-semibold);
}

.modal-close {
  padding: var(--space-2);
  border-radius: var(--radius-md);
  color: var(--color-text-secondary);
}
.modal-close:hover {
  background: var(--color-neutral-100);
}

.modal-body {
  padding: var(--space-5);
}

.modal-footer {
  display: flex;
  gap: var(--space-3);
  justify-content: flex-end;
  padding: var(--space-4) var(--space-5);
  border-top: 1px solid var(--color-border-primary);
  background: var(--color-bg-secondary);
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
```

### Form Elements

```css
/* Input base */
.input {
  width: 100%;
  padding: var(--space-2.5) var(--space-3);
  font-size: var(--font-size-sm);
  background: var(--color-bg-primary);
  border: 1px solid var(--color-border-primary);
  border-radius: var(--radius-lg);
  transition: border-color 150ms ease;
}

.input:hover {
  border-color: var(--color-border-secondary);
}

.input:focus {
  outline: none;
  border-color: var(--color-primary-500);
  box-shadow: 0 0 0 3px var(--color-primary-100);
}

.input:disabled {
  background: var(--color-bg-secondary);
  cursor: not-allowed;
}

.input-error {
  border-color: var(--color-error-500);
}
.input-error:focus {
  box-shadow: 0 0 0 3px var(--color-error-100);
}

/* Label */
.label {
  display: block;
  margin-bottom: var(--space-1.5);
  font-size: var(--font-size-sm);
  font-weight: var(--font-weight-medium);
  color: var(--color-text-primary);
}

/* Helper text */
.helper-text {
  margin-top: var(--space-1.5);
  font-size: var(--font-size-xs);
  color: var(--color-text-secondary);
}

.helper-text-error {
  color: var(--color-error-600);
}

/* Select */
.select {
  appearance: none;
  background-image: url("data:image/svg+xml,..."); /* Chevron */
  background-repeat: no-repeat;
  background-position: right var(--space-3) center;
  padding-right: var(--space-10);
}

/* Checkbox */
.checkbox {
  width: var(--space-4);
  height: var(--space-4);
  border: 1px solid var(--color-border-secondary);
  border-radius: var(--radius-sm);
  cursor: pointer;
}

.checkbox:checked {
  background: var(--color-primary-600);
  border-color: var(--color-primary-600);
}

/* Toggle */
.toggle {
  position: relative;
  width: 44px;
  height: 24px;
  background: var(--color-neutral-200);
  border-radius: 9999px;
  cursor: pointer;
  transition: background 150ms ease;
}

.toggle::after {
  content: '';
  position: absolute;
  top: 2px;
  left: 2px;
  width: 20px;
  height: 20px;
  background: white;
  border-radius: 50%;
  transition: transform 150ms ease;
}

.toggle:checked {
  background: var(--color-primary-600);
}

.toggle:checked::after {
  transform: translateX(20px);
}
```

### Trading Components

```css
/* Order form */
.order-form {
  display: flex;
  flex-direction: column;
  gap: var(--space-4);
  padding: var(--space-4);
  background: var(--color-bg-secondary);
  border-radius: var(--radius-xl);
}

/* Outcome selector */
.outcome-selector {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: var(--space-2);
}

.outcome-btn {
  padding: var(--space-3);
  border-radius: var(--radius-lg);
  text-align: center;
  font-weight: var(--font-weight-semibold);
  border: 2px solid transparent;
  transition: all 150ms ease;
}

.outcome-btn-yes {
  background: var(--color-buy-bg);
  color: var(--color-buy);
}
.outcome-btn-yes:hover,
.outcome-btn-yes.active {
  border-color: var(--color-buy);
}

.outcome-btn-no {
  background: var(--color-sell-bg);
  color: var(--color-sell);
}
.outcome-btn-no:hover,
.outcome-btn-no.active {
  border-color: var(--color-sell);
}

/* Amount input */
.amount-input-wrapper {
  position: relative;
}

.amount-input {
  padding-left: var(--space-8);
  font-family: var(--font-family-mono);
  font-size: var(--font-size-xl);
}

.amount-currency {
  position: absolute;
  left: var(--space-3);
  top: 50%;
  transform: translateY(-50%);
  color: var(--color-text-secondary);
}

/* Order summary */
.order-summary {
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
  padding: var(--space-3);
  background: var(--color-bg-tertiary);
  border-radius: var(--radius-lg);
}

.order-summary-row {
  display: flex;
  justify-content: space-between;
  font-size: var(--font-size-sm);
}

.order-summary-label {
  color: var(--color-text-secondary);
}

.order-summary-value {
  font-weight: var(--font-weight-medium);
  font-family: var(--font-family-mono);
}

/* Probability display */
.probability-badge {
  display: inline-flex;
  align-items: center;
  gap: var(--space-1);
  padding: var(--space-1) var(--space-2);
  border-radius: var(--radius-md);
  font-size: var(--font-size-sm);
  font-weight: var(--font-weight-semibold);
  font-family: var(--font-family-mono);
}

.probability-high {
  background: var(--color-success-100);
  color: var(--color-success-700);
}

.probability-mid {
  background: var(--color-warning-100);
  color: var(--color-warning-700);
}

.probability-low {
  background: var(--color-error-100);
  color: var(--color-error-700);
}
```

---

## 6. Design Tokens

### Complete Token System (CSS Variables)

```css
:root {
  /* ==================== */
  /* PRIMITIVE TOKENS     */
  /* ==================== */

  /* Colors - Blue */
  --primitive-blue-50: #EEF2FF;
  --primitive-blue-100: #E0E7FF;
  --primitive-blue-200: #C7D2FE;
  --primitive-blue-300: #A5B4FC;
  --primitive-blue-400: #818CF8;
  --primitive-blue-500: #6366F1;
  --primitive-blue-600: #4F46E5;
  --primitive-blue-700: #4338CA;
  --primitive-blue-800: #3730A3;
  --primitive-blue-900: #312E81;

  /* Colors - Green */
  --primitive-green-50: #ECFDF5;
  --primitive-green-100: #D1FAE5;
  --primitive-green-200: #A7F3D0;
  --primitive-green-300: #6EE7B7;
  --primitive-green-400: #34D399;
  --primitive-green-500: #10B981;
  --primitive-green-600: #059669;
  --primitive-green-700: #047857;

  /* Colors - Red */
  --primitive-red-50: #FEF2F2;
  --primitive-red-100: #FEE2E2;
  --primitive-red-200: #FECACA;
  --primitive-red-300: #FCA5A5;
  --primitive-red-400: #F87171;
  --primitive-red-500: #EF4444;
  --primitive-red-600: #DC2626;
  --primitive-red-700: #B91C1C;

  /* Colors - Gray */
  --primitive-gray-0: #FFFFFF;
  --primitive-gray-50: #F9FAFB;
  --primitive-gray-100: #F3F4F6;
  --primitive-gray-200: #E5E7EB;
  --primitive-gray-300: #D1D5DB;
  --primitive-gray-400: #9CA3AF;
  --primitive-gray-500: #6B7280;
  --primitive-gray-600: #4B5563;
  --primitive-gray-700: #374151;
  --primitive-gray-800: #1F2937;
  --primitive-gray-900: #111827;
  --primitive-gray-950: #030712;

  /* Spacing primitives */
  --primitive-space-1: 4px;
  --primitive-space-2: 8px;
  --primitive-space-3: 12px;
  --primitive-space-4: 16px;
  --primitive-space-5: 20px;
  --primitive-space-6: 24px;
  --primitive-space-8: 32px;
  --primitive-space-10: 40px;
  --primitive-space-12: 48px;
  --primitive-space-16: 64px;

  /* Border radius */
  --primitive-radius-none: 0;
  --primitive-radius-sm: 4px;
  --primitive-radius-md: 6px;
  --primitive-radius-lg: 8px;
  --primitive-radius-xl: 12px;
  --primitive-radius-2xl: 16px;
  --primitive-radius-full: 9999px;

  /* ==================== */
  /* SEMANTIC TOKENS      */
  /* ==================== */

  /* Background colors */
  --color-bg-primary: var(--primitive-gray-0);
  --color-bg-secondary: var(--primitive-gray-50);
  --color-bg-tertiary: var(--primitive-gray-100);
  --color-bg-inverse: var(--primitive-gray-900);

  /* Text colors */
  --color-text-primary: var(--primitive-gray-900);
  --color-text-secondary: var(--primitive-gray-600);
  --color-text-tertiary: var(--primitive-gray-500);
  --color-text-inverse: var(--primitive-gray-0);
  --color-text-disabled: var(--primitive-gray-400);

  /* Border colors */
  --color-border-primary: var(--primitive-gray-200);
  --color-border-secondary: var(--primitive-gray-300);
  --color-border-focus: var(--primitive-blue-500);

  /* Interactive colors */
  --color-interactive: var(--primitive-blue-600);
  --color-interactive-hover: var(--primitive-blue-700);
  --color-interactive-active: var(--primitive-blue-800);

  /* Status colors */
  --color-success: var(--primitive-green-500);
  --color-success-bg: var(--primitive-green-50);
  --color-error: var(--primitive-red-500);
  --color-error-bg: var(--primitive-red-50);

  /* Trading colors */
  --color-buy: var(--primitive-green-500);
  --color-buy-hover: var(--primitive-green-600);
  --color-sell: var(--primitive-red-500);
  --color-sell-hover: var(--primitive-red-600);

  /* ==================== */
  /* COMPONENT TOKENS     */
  /* ==================== */

  /* Button */
  --button-primary-bg: var(--color-interactive);
  --button-primary-bg-hover: var(--color-interactive-hover);
  --button-primary-text: var(--color-text-inverse);
  --button-padding-x: var(--primitive-space-4);
  --button-padding-y: var(--primitive-space-2);
  --button-radius: var(--primitive-radius-lg);

  /* Card */
  --card-bg: var(--color-bg-primary);
  --card-border: var(--color-border-primary);
  --card-radius: var(--primitive-radius-xl);
  --card-padding: var(--primitive-space-5);

  /* Input */
  --input-bg: var(--color-bg-primary);
  --input-border: var(--color-border-primary);
  --input-border-hover: var(--color-border-secondary);
  --input-border-focus: var(--color-border-focus);
  --input-radius: var(--primitive-radius-lg);
  --input-padding-x: var(--primitive-space-3);
  --input-padding-y: var(--primitive-space-2);

  /* Modal */
  --modal-bg: var(--color-bg-primary);
  --modal-radius: var(--primitive-radius-2xl);
  --modal-overlay-bg: rgba(0, 0, 0, 0.5);

  /* ==================== */
  /* SHADOWS              */
  /* ==================== */

  --shadow-sm: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -2px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -4px rgba(0, 0, 0, 0.1);
  --shadow-xl: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 8px 10px -6px rgba(0, 0, 0, 0.1);
  --shadow-2xl: 0 25px 50px -12px rgba(0, 0, 0, 0.25);

  /* ==================== */
  /* Z-INDEX              */
  /* ==================== */

  --z-dropdown: 10;
  --z-sticky: 20;
  --z-fixed: 30;
  --z-modal-backdrop: 40;
  --z-modal: 50;
  --z-popover: 60;
  --z-tooltip: 70;
  --z-toast: 80;
}

/* Dark mode overrides */
[data-theme="dark"] {
  --color-bg-primary: var(--primitive-gray-950);
  --color-bg-secondary: var(--primitive-gray-900);
  --color-bg-tertiary: var(--primitive-gray-800);
  --color-bg-inverse: var(--primitive-gray-0);

  --color-text-primary: var(--primitive-gray-50);
  --color-text-secondary: var(--primitive-gray-400);
  --color-text-tertiary: var(--primitive-gray-500);
  --color-text-inverse: var(--primitive-gray-900);

  --color-border-primary: var(--primitive-gray-800);
  --color-border-secondary: var(--primitive-gray-700);

  --color-interactive: var(--primitive-blue-500);
  --color-interactive-hover: var(--primitive-blue-400);
}
```

---

## 7. Accessibility Guidelines

### WCAG 2.1 AA Compliance

Our platform targets WCAG 2.1 Level AA compliance as the baseline, with Level AAA where feasible.

### Perceivable

#### Color and Contrast

| Requirement | Standard | Implementation |
|-------------|----------|----------------|
| Text contrast (normal) | 4.5:1 minimum | All body text meets 4.5:1 |
| Text contrast (large) | 3:1 minimum | Headlines 18px+ meet 3:1 |
| Non-text contrast | 3:1 minimum | Icons, borders, focus rings |
| Color not sole indicator | Use shape/text | Profit/loss uses +/- signs |

**Trading-Specific Considerations:**
- Green/red are used for buy/sell, but always paired with labels
- Charts include texture patterns in addition to color
- Color-blind safe palette option available

#### Text and Content

```css
/* Minimum readable text */
.body-text {
  font-size: 16px;  /* Minimum for body text */
  line-height: 1.5; /* 150% for readability */
}

/* Resizable text - support up to 200% zoom */
html {
  font-size: 100%; /* Don't lock font size */
}

/* Maximum line length for readability */
.content-block {
  max-width: 75ch;
}
```

### Operable

#### Keyboard Navigation

```css
/* Visible focus indicators */
*:focus-visible {
  outline: 2px solid var(--color-border-focus);
  outline-offset: 2px;
}

/* Skip links */
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  z-index: 100;
}

.skip-link:focus {
  top: 0;
}
```

**Keyboard Shortcuts:**

| Action | Shortcut |
|--------|----------|
| Search markets | `/` or `Cmd+K` |
| Go to portfolio | `G` then `P` |
| Go to markets | `G` then `M` |
| Close modal | `Esc` |
| Submit order | `Enter` (when focused) |

#### Touch Targets

```css
/* Minimum touch target size: 44x44px */
.btn,
.link,
.interactive {
  min-height: 44px;
  min-width: 44px;
}

/* Adequate spacing between targets */
.btn-group > * + * {
  margin-left: var(--space-2);
}
```

#### Motion and Animation

```css
/* Respect reduced motion preference */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

/* Auto-playing content controls */
.chart-animation {
  /* Pause button always visible */
}
```

### Understandable

#### Clear Language

- Use plain language (8th grade reading level)
- Define technical terms on first use
- Provide glossary for trading terminology
- Error messages explain how to fix issues

#### Consistent Navigation

- Navigation remains in same location across pages
- Components behave consistently
- Patterns are predictable (modals close with X or Esc)

#### Error Prevention

```html
<!-- Confirmation for significant actions -->
<dialog class="confirmation-modal">
  <h2>Confirm Your Order</h2>
  <p>You are about to buy 100 shares of "Yes" at $0.65</p>
  <p><strong>Total cost: $65.00</strong></p>
  <button class="btn-secondary">Cancel</button>
  <button class="btn-primary">Confirm Order</button>
</dialog>
```

### Robust

#### Semantic HTML

```html
<!-- Proper heading hierarchy -->
<h1>Market: Will X happen?</h1>
<h2>Price Chart</h2>
<h2>Order Book</h2>
<h3>Buy Orders</h3>
<h3>Sell Orders</h3>
<h2>Place Order</h2>

<!-- Landmarks -->
<header role="banner">...</header>
<nav role="navigation">...</nav>
<main role="main">...</main>
<footer role="contentinfo">...</footer>

<!-- Form accessibility -->
<label for="amount">Amount (USD)</label>
<input
  id="amount"
  type="number"
  aria-describedby="amount-help"
  aria-invalid="false"
/>
<span id="amount-help">Enter amount between $1 and $10,000</span>
```

#### ARIA Labels

```html
<!-- Price change indicator -->
<span aria-label="Price increased by 5%">
  <span aria-hidden="true">+5%</span>
</span>

<!-- Live regions for real-time updates -->
<div
  role="status"
  aria-live="polite"
  aria-label="Current probability"
>
  65%
</div>

<!-- Complex components -->
<div
  role="tablist"
  aria-label="Order type"
>
  <button role="tab" aria-selected="true">Market</button>
  <button role="tab" aria-selected="false">Limit</button>
</div>
```

### Accessibility Checklist

- [ ] All images have meaningful alt text
- [ ] Videos have captions and transcripts
- [ ] Forms have visible labels
- [ ] Error messages are clear and specific
- [ ] Focus order is logical
- [ ] No keyboard traps
- [ ] Color contrast passes WCAG AA
- [ ] Text can be resized to 200%
- [ ] Page works without CSS
- [ ] Page works without JavaScript (graceful degradation)
- [ ] Screen reader tested (VoiceOver, NVDA)
- [ ] Tested at different zoom levels

---

## 8. Animation Principles

### Core Principles

1. **Purpose-Driven:** Every animation should serve a function
2. **Subtle:** Animations enhance, never distract
3. **Fast:** Keep durations short (150-300ms typical)
4. **Natural:** Use easing curves that feel organic
5. **Accessible:** Respect prefers-reduced-motion

### Duration Guidelines

| Type | Duration | Use Case |
|------|----------|----------|
| **Micro** | 100-150ms | Button states, hover effects |
| **Short** | 150-250ms | Toggles, dropdowns, tooltips |
| **Medium** | 250-350ms | Modals, page transitions |
| **Long** | 350-500ms | Complex sequences, onboarding |

### Easing Functions

```css
/* Standard easings */
--ease-in: cubic-bezier(0.4, 0, 1, 1);
--ease-out: cubic-bezier(0, 0, 0.2, 1);
--ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);

/* Spring-like for bouncy interactions */
--ease-spring: cubic-bezier(0.34, 1.56, 0.64, 1);

/* Default for most transitions */
--ease-default: var(--ease-out);
```

### Common Animations

```css
/* Fade in */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

/* Slide up (for modals, toasts) */
@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(16px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Scale in (for dropdowns, popovers) */
@keyframes scaleIn {
  from {
    opacity: 0;
    transform: scale(0.95);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

/* Price change flash */
@keyframes priceFlash {
  0% { background-color: transparent; }
  50% { background-color: var(--color-success-bg); }
  100% { background-color: transparent; }
}

/* Loading pulse */
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

/* Skeleton loading */
@keyframes shimmer {
  0% { background-position: -200% 0; }
  100% { background-position: 200% 0; }
}

.skeleton {
  background: linear-gradient(
    90deg,
    var(--color-bg-secondary) 25%,
    var(--color-bg-tertiary) 50%,
    var(--color-bg-secondary) 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}
```

### Interactive State Transitions

```css
/* Button hover/active */
.btn {
  transition:
    background-color 150ms var(--ease-out),
    transform 150ms var(--ease-out),
    box-shadow 150ms var(--ease-out);
}

.btn:hover {
  transform: translateY(-1px);
  box-shadow: var(--shadow-md);
}

.btn:active {
  transform: translateY(0);
  box-shadow: none;
}

/* Card hover */
.card-interactive {
  transition:
    border-color 150ms var(--ease-out),
    box-shadow 200ms var(--ease-out);
}

.card-interactive:hover {
  box-shadow: var(--shadow-lg);
}

/* Input focus */
.input {
  transition:
    border-color 150ms var(--ease-out),
    box-shadow 150ms var(--ease-out);
}
```

### Loading States

```css
/* Button loading */
.btn-loading {
  position: relative;
  color: transparent;
  pointer-events: none;
}

.btn-loading::after {
  content: '';
  position: absolute;
  width: 16px;
  height: 16px;
  border: 2px solid currentColor;
  border-right-color: transparent;
  border-radius: 50%;
  animation: spin 600ms linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

/* Content loading placeholder */
.loading-placeholder {
  border-radius: var(--radius-md);
  animation: pulse 2s infinite;
}
```

---

## 9. Icon System

### Icon Guidelines

**Style:**
- Stroke-based icons (2px stroke weight)
- Rounded caps and joins
- 24x24px base size
- Consistent optical sizing

**Usage:**
- Use icons to support text, not replace it
- Maintain consistent meaning across the app
- Provide accessible labels for icon-only buttons

### Icon Sizes

```css
--icon-xs: 12px;
--icon-sm: 16px;
--icon-md: 20px;
--icon-lg: 24px;
--icon-xl: 32px;
--icon-2xl: 48px;
```

### Core Icon Set

#### Navigation Icons

| Icon | Name | Usage |
|------|------|-------|
| Home | `home` | Dashboard/home navigation |
| Search | `search` | Search input, market discovery |
| Menu | `menu` | Mobile hamburger menu |
| Close | `x` | Close modals, dismiss items |
| Back | `arrow-left` | Navigation back |
| External | `external-link` | External links |

#### Trading Icons

| Icon | Name | Usage |
|------|------|-------|
| Trending Up | `trending-up` | Price increase, profit |
| Trending Down | `trending-down` | Price decrease, loss |
| Chart | `line-chart` | Price charts, analytics |
| Wallet | `wallet` | Balance, deposits |
| Swap | `swap` | Trade, exchange |
| Clock | `clock` | Pending, time remaining |

#### Status Icons

| Icon | Name | Usage |
|------|------|-------|
| Check | `check` | Success, completed |
| Alert | `alert-circle` | Warning, attention |
| Info | `info` | Information, help |
| Error | `x-circle` | Error, failed |

#### Action Icons

| Icon | Name | Usage |
|------|------|-------|
| Plus | `plus` | Add, create |
| Minus | `minus` | Remove, decrease |
| Edit | `pencil` | Edit item |
| Copy | `copy` | Copy to clipboard |
| Share | `share` | Share content |
| Filter | `filter` | Filter options |
| Sort | `sort` | Sort options |

#### User Icons

| Icon | Name | Usage |
|------|------|-------|
| User | `user` | Profile, account |
| Settings | `settings` | Preferences |
| Bell | `bell` | Notifications |
| Trophy | `trophy` | Achievements, leaderboard |

### Icon Component

```css
/* Base icon styles */
.icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.icon-xs { width: var(--icon-xs); height: var(--icon-xs); }
.icon-sm { width: var(--icon-sm); height: var(--icon-sm); }
.icon-md { width: var(--icon-md); height: var(--icon-md); }
.icon-lg { width: var(--icon-lg); height: var(--icon-lg); }
.icon-xl { width: var(--icon-xl); height: var(--icon-xl); }

/* Icon colors */
.icon-primary { color: var(--color-text-primary); }
.icon-secondary { color: var(--color-text-secondary); }
.icon-success { color: var(--color-success); }
.icon-error { color: var(--color-error); }
.icon-interactive { color: var(--color-interactive); }
```

### Icon + Text Patterns

```css
/* Icon with text (left) */
.icon-text {
  display: inline-flex;
  align-items: center;
  gap: var(--space-2);
}

/* Icon button */
.icon-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: var(--space-2);
  border-radius: var(--radius-md);
  color: var(--color-text-secondary);
  transition: all 150ms var(--ease-out);
}

.icon-btn:hover {
  background: var(--color-bg-secondary);
  color: var(--color-text-primary);
}

/* Badge with icon */
.badge-icon {
  display: inline-flex;
  align-items: center;
  gap: var(--space-1);
  padding: var(--space-1) var(--space-2);
  border-radius: var(--radius-full);
  font-size: var(--font-size-xs);
  font-weight: var(--font-weight-medium);
}
```

---

## Appendix

### A. Design Resources

**Figma Libraries:**
- Design System Components
- Icon Library
- Illustration Assets

**Code Repositories:**
- Component Storybook
- Design Tokens Package

### B. Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | Jan 2026 | Initial design system |

### C. References

- [Web3 Design Principles](https://merge.rocks/blog/web3-design-in-2024-best-principles-and-patterns)
- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/standards-guidelines/wcag/)
- [Design Tokens Best Practices](https://designsystem.digital.gov/design-tokens/)
- [Component Library Patterns](https://www.uxpin.com/studio/blog/best-practices-for-scalable-component-libraries/)
