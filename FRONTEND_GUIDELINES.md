# Frontend Design System & Guidelines
# RentKart — India's Premier Rental Marketplace
**Version:** 1.0 | **Platform:** React Native (Mobile) + Next.js (Admin Web)

---

## 1. Design Principles

1. **Clarity Over Cleverness** — Every screen must communicate intent in under 3 seconds. Escrow numbers, refund amounts, and order status must never require interpretation.
2. **Trust at Every Touchpoint** — Visual cues (verification badges, escrow breakdowns, photo evidence) must feel transparent and institutional — not startup-ish.
3. **Mobile-First, India-First** — Designs optimised for 5-inch Android screens on 4G connections. Heavy images compressed; skeleton loaders for every async state.
4. **Reduce Friction, Increase Confidence** — Especially at checkout. The "Refundable Hold" reframing is a design principle, not just a copy change — everything about escrow must feel like a safe deposit, not a payment.
5. **Consistency is Non-Negotiable** — Components built in Week 1 must look and behave identically when reused in Week 10. All styling goes through the design system — no inline one-off styles.

---

## 2. Design Tokens

### Primary Color Scale (Saffron / Warmth)
```css
--primary-50:  #FFF3EC;  /* Background tints */
--primary-100: #FFE0CC;
--primary-200: #FFC5A0;
--primary-300: #FFA574;
--primary-400: #FF8A50;
--primary-500: #FF6B1A;  /* Primary action color — CTAs, highlights */
--primary-600: #E55C0E;
--primary-700: #C44D09;
--primary-800: #9E3E06;
--primary-900: #7A3004;  /* Deep accent */
```

### Neutral / Ink Scale
```css
--neutral-50:  #FDFAF8;  /* Page background */
--neutral-100: #F5F0EE;  /* Card backgrounds, subtle fills */
--neutral-200: #EDE6E0;  /* Borders */
--neutral-300: #D6CCC4;
--neutral-400: #B8A99F;
--neutral-500: #9A857A;
--neutral-600: #6B5E58;  /* Secondary text */
--neutral-700: #4A3F3A;
--neutral-800: #2E2420;
--neutral-900: #1A1614;  /* Primary text — headings, labels */
```

### Semantic Colors
```css
/* Success — Teal */
--success-50:  #E5F4F2;
--success-500: #0A7B6C;
--success-700: #065E52;

/* Warning — Amber */
--warning-50:  #FDF6E3;
--warning-500: #C48E00;
--warning-700: #9A6F00;

/* Error — Rose */
--error-50:  #FCEEF2;
--error-500: #C0365A;
--error-700: #951D44;

/* Info — Indigo */
--info-50:  #EEEAFC;
--info-500: #3B2F8A;
--info-700: #2A2167;
```

**Usage Rules:**
- `--primary-500` — Primary CTA buttons, links, active states, highlights
- `--primary-50 / 100` — Background tints on highlighted sections
- `--neutral-900` — All body text, headings
- `--neutral-600` — Secondary text, labels, helper text
- `--neutral-200` — All border/divider lines
- `--neutral-50` — Page background; `--neutral-100` — card backgrounds
- `--success-500` — Confirmed statuses, refund received, verified badges
- `--warning-500` — Day 20/23 countdown alerts, pending review states
- `--error-500` — Form validation errors, failed states, dispute alerts
- `--info-500` — Informational badges, admin-only UI sections

---

### Typography

**Font Families:**
```css
--font-display: 'Playfair Display', Georgia, serif;   /* Headings only */
--font-sans:    'DM Sans', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
--font-mono:    'DM Mono', 'Courier New', monospace;  /* Prices, order IDs, codes */
```

**Font Size Scale (rem):**
```css
--text-xs:   0.75rem;   /* 12px — badges, metadata, fine print */
--text-sm:   0.875rem;  /* 14px — helper text, secondary labels */
--text-base: 1rem;      /* 16px — body text, inputs */
--text-lg:   1.125rem;  /* 18px — card titles, section subheads */
--text-xl:   1.25rem;   /* 20px — section headings */
--text-2xl:  1.5rem;    /* 24px — page headings */
--text-3xl:  1.875rem;  /* 30px — feature headings */
--text-4xl:  2.25rem;   /* 36px — hero/marketing headings */
```

**Font Weights:**
```css
--weight-light:    300;
--weight-regular:  400;
--weight-medium:   500;
--weight-semibold: 600;
--weight-bold:     700;
--weight-extrabold: 800;  /* Display headings only */
```

**Line Heights:**
```css
--leading-tight:  1.25;  /* Headings */
--leading-snug:   1.375; /* Sub-headings */
--leading-normal: 1.5;   /* Body text */
--leading-relaxed: 1.75; /* Long-form descriptions */
```

**Usage Rules:**
- Display font (`Playfair Display`) only for H1/H2 headings — never for UI labels or body
- Monospace font for: prices (₹ amounts), order IDs, countdown timers, codes
- Minimum body text: `--text-base` (16px) — never smaller for readable content

---

### Spacing Scale
```css
--space-1:  4px;
--space-2:  8px;
--space-3:  12px;
--space-4:  16px;
--space-5:  20px;
--space-6:  24px;
--space-7:  28px;
--space-8:  32px;
--space-10: 40px;
--space-12: 48px;
--space-16: 64px;
--space-20: 80px;
```

**Usage Rules:**
- `space-2 / space-3` — Padding within compact UI elements (chips, badges)
- `space-4` — Default component internal padding
- `space-6` — Card internal padding (standard)
- `space-8` — Section internal padding (mobile)
- `space-12 / space-16` — Section vertical padding (desktop)
- `space-4` — Gap between form fields
- `space-8` — Gap between sections on a page

---

### Border Radius
```css
--radius-none: 0px;
--radius-sm:   4px;   /* Chips, badges, small buttons */
--radius-base: 6px;   /* Inputs, small cards */
--radius-md:   8px;   /* Standard components */
--radius-lg:   12px;  /* Main cards */
--radius-xl:   16px;  /* Bottom sheets, featured cards */
--radius-2xl:  24px;  /* Large modals, cover cards */
--radius-full: 9999px; /* Pills, avatar, toggle */
```

---

### Shadows
```css
--shadow-sm:  0 1px 2px 0 rgba(26, 22, 20, 0.05);
--shadow-base: 0 1px 3px 0 rgba(26, 22, 20, 0.10), 0 1px 2px -1px rgba(26, 22, 20, 0.10);
--shadow-md:  0 4px 6px -1px rgba(26, 22, 20, 0.10), 0 2px 4px -2px rgba(26, 22, 20, 0.10);
--shadow-lg:  0 10px 15px -3px rgba(26, 22, 20, 0.10), 0 4px 6px -4px rgba(26, 22, 20, 0.10);
--shadow-xl:  0 20px 25px -5px rgba(26, 22, 20, 0.10), 0 8px 10px -6px rgba(26, 22, 20, 0.10);
```

---

## 3. Layout System

### Grid (Web/Admin)
- **Container max-width:** 1100px, centered
- **Columns:** 12
- **Gutter:** 24px (desktop), 16px (tablet), 12px (mobile)

### Responsive Breakpoints
```css
--bp-sm:  640px;   /* Large mobile / small tablet */
--bp-md:  768px;   /* Tablet */
--bp-lg:  1024px;  /* Desktop */
--bp-xl:  1280px;  /* Wide desktop */
--bp-2xl: 1536px;  /* Ultra-wide */
```

### Common Layout Patterns

```jsx
/* Centered Content */
<div className="max-w-[1100px] mx-auto px-4 md:px-10">
  {children}
</div>

/* Two-Column (Content + Sidebar) */
<div className="grid grid-cols-1 lg:grid-cols-[1fr_360px] gap-8">
  <main>{content}</main>
  <aside>{sidebar}</aside>
</div>

/* Card Grid — Listings */
<div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
  {listings.map(l => <ListingCard key={l.id} {...l} />)}
</div>
```

---

## 4. Component Library

---

### Button

**Variants and States:**

```jsx
/* PRIMARY — Default */
<button className="
  inline-flex items-center justify-center gap-2
  px-5 py-3 rounded-lg
  bg-[#FF6B1A] text-white font-semibold text-sm
  transition-all duration-200
  hover:bg-[#E55C0E]
  focus:outline-none focus:ring-2 focus:ring-[#FF6B1A] focus:ring-offset-2
  disabled:opacity-50 disabled:cursor-not-allowed
  active:scale-[0.98]
">Rent Now</button>

/* SECONDARY */
<button className="
  inline-flex items-center justify-center gap-2
  px-5 py-3 rounded-lg
  bg-[#F5F0EE] text-[#1A1614] font-semibold text-sm
  border border-[#EDE6E0]
  hover:bg-[#EDE6E0]
  focus:outline-none focus:ring-2 focus:ring-[#FF6B1A] focus:ring-offset-2
  disabled:opacity-50 disabled:cursor-not-allowed
">View Details</button>

/* OUTLINE */
<button className="
  inline-flex items-center justify-center gap-2
  px-5 py-3 rounded-lg
  bg-transparent text-[#FF6B1A] font-semibold text-sm
  border-2 border-[#FF6B1A]
  hover:bg-[#FFF3EC]
  focus:outline-none focus:ring-2 focus:ring-[#FF6B1A] focus:ring-offset-2
  disabled:opacity-50 disabled:cursor-not-allowed
">Schedule Return</button>

/* GHOST */
<button className="
  inline-flex items-center justify-center gap-2
  px-5 py-3 rounded-lg
  bg-transparent text-[#6B5E58] font-medium text-sm
  hover:bg-[#F5F0EE] hover:text-[#1A1614]
  focus:outline-none focus:ring-2 focus:ring-[#FF6B1A] focus:ring-offset-2
">Cancel</button>

/* DANGER */
<button className="
  inline-flex items-center justify-center gap-2
  px-5 py-3 rounded-lg
  bg-[#C0365A] text-white font-semibold text-sm
  hover:bg-[#951D44]
  focus:outline-none focus:ring-2 focus:ring-[#C0365A] focus:ring-offset-2
  disabled:opacity-50 disabled:cursor-not-allowed
">Suspend Account</button>

/* LOADING STATE — wrap any button */
<button disabled className="... opacity-75 cursor-not-allowed">
  <svg className="animate-spin h-4 w-4" viewBox="0 0 24 24">...</svg>
  Processing…
</button>
```

**Sizes:**
```jsx
/* SMALL */  className="px-3 py-1.5 text-xs rounded-md"
/* MEDIUM */ className="px-5 py-3 text-sm rounded-lg"     // default
/* LARGE */  className="px-6 py-4 text-base rounded-xl"
```

**Usage Rules:**
- Primary button: one per screen maximum — the most important action (Rent Now, Pay, Confirm)
- Secondary: supplementary actions (View, Edit, Back)
- Outline: alternative paths (Schedule Return vs Cancel)
- Ghost: low-priority or destructive-adjacent actions
- Danger: irreversible admin actions only

---

### Input Fields

```jsx
/* TEXT INPUT — Default */
<div className="flex flex-col gap-1.5">
  <label className="text-sm font-medium text-[#1A1614]">
    Item Value (₹)
  </label>
  <input
    type="number"
    placeholder="e.g. 25000"
    className="
      w-full px-4 py-3 rounded-lg
      bg-white text-[#1A1614]
      border border-[#EDE6E0]
      text-sm placeholder:text-[#B8A99F]
      transition-colors duration-150
      focus:outline-none focus:ring-2 focus:ring-[#FF6B1A] focus:border-transparent
      disabled:bg-[#F5F0EE] disabled:text-[#9A857A] disabled:cursor-not-allowed
    "
  />
  <p className="text-xs text-[#6B5E58]">Full market value — used to calculate refundable hold</p>
</div>

/* INPUT — Error State */
<input className="
  ... (same as above)
  border-[#C0365A]
  focus:ring-[#C0365A]
" />
<p className="text-xs text-[#C0365A] flex items-center gap-1">
  <AlertCircle className="w-3 h-3" /> Item value must be between ₹500 and ₹5,00,000
</p>

/* INPUT — Success State */
<input className="
  ... border-[#0A7B6C] focus:ring-[#0A7B6C]
" />
<p className="text-xs text-[#0A7B6C]">✓ Looks good</p>

/* TEXTAREA */
<textarea
  rows={4}
  className="
    w-full px-4 py-3 rounded-lg resize-none
    bg-white text-[#1A1614]
    border border-[#EDE6E0] text-sm
    focus:outline-none focus:ring-2 focus:ring-[#FF6B1A] focus:border-transparent
    placeholder:text-[#B8A99F]
  "
/>
```

---

### Cards

```jsx
/* STANDARD LISTING CARD */
<div className="
  bg-white rounded-xl border border-[#EDE6E0]
  overflow-hidden
  shadow-sm hover:shadow-md
  transition-shadow duration-200
  cursor-pointer
">
  <div className="relative aspect-[4/5] bg-[#F5F0EE]">
    <img src={photo} alt={title} className="w-full h-full object-cover" />
    <span className="absolute top-2 left-2 bg-white/90 text-xs font-medium px-2 py-0.5 rounded-full text-[#6B5E58]">
      {condition}
    </span>
  </div>
  <div className="p-4">
    <p className="text-sm font-semibold text-[#1A1614] line-clamp-2">{title}</p>
    <div className="flex items-center gap-1 mt-1">
      <Star className="w-3 h-3 fill-[#C48E00] text-[#C48E00]" />
      <span className="text-xs text-[#6B5E58]">{rating} ({totalRentals})</span>
    </div>
    <div className="mt-3 flex items-end justify-between">
      <div>
        <p className="text-xs text-[#6B5E58]">From</p>
        <p className="font-mono text-base font-semibold text-[#1A1614]">₹{dailyRate}/day</p>
      </div>
      <p className="text-xs text-[#9A857A]">Hold: ₹{itemValue}</p>
    </div>
  </div>
</div>

/* INFO / SUMMARY CARD */
<div className="bg-white rounded-xl border border-[#EDE6E0] p-6 shadow-sm">
  {children}
</div>

/* ACCENT CARD (with left border — for callouts) */
<div className="bg-white rounded-xl border border-[#EDE6E0] border-l-4 border-l-[#FF6B1A] p-5">
  {children}
</div>

/* ESCROW BREAKDOWN CARD */
<div className="bg-[#1A1614] rounded-xl p-6 text-white">
  <p className="text-xs text-white/50 mb-4 uppercase tracking-wider font-medium">Payment Breakdown</p>
  <div className="space-y-3">
    <div className="flex justify-between">
      <span className="text-sm text-white/70">Refundable Security Hold</span>
      <span className="font-mono font-semibold">₹24,500</span>
    </div>
    <div className="flex justify-between">
      <span className="text-sm text-white/70">Delivery Charge</span>
      <span className="font-mono font-semibold">₹150</span>
    </div>
    <div className="border-t border-white/10 pt-3 flex justify-between">
      <span className="text-sm font-semibold text-[#FF9355]">Total to Pay</span>
      <span className="font-mono font-bold text-[#FF9355]">₹24,650</span>
    </div>
    <p className="text-xs text-[#0A7B6C] mt-2">↩ You get ₹24,500 back after return</p>
  </div>
</div>
```

---

### Modals

```jsx
/* MODAL WRAPPER */
<div className="fixed inset-0 z-50 flex items-end sm:items-center justify-center">
  {/* Overlay */}
  <div
    className="absolute inset-0 bg-black/50 backdrop-blur-sm"
    onClick={onClose}
    aria-hidden="true"
  />
  {/* Content */}
  <div
    className="relative w-full max-w-md bg-white rounded-t-2xl sm:rounded-2xl p-6 shadow-xl
               animate-in slide-in-from-bottom sm:animate-in sm:zoom-in-95 duration-200"
    role="dialog"
    aria-modal="true"
    aria-labelledby="modal-title"
  >
    <div className="flex items-center justify-between mb-5">
      <h3 id="modal-title" className="text-lg font-semibold text-[#1A1614]">{title}</h3>
      <button onClick={onClose} className="p-1.5 rounded-lg hover:bg-[#F5F0EE] text-[#6B5E58]">
        <X className="w-5 h-5" />
      </button>
    </div>
    <div>{children}</div>
    <div className="flex gap-3 mt-6">
      <button onClick={onClose} className="flex-1 ... secondary-button">Cancel</button>
      <button onClick={onConfirm} className="flex-1 ... primary-button">Confirm</button>
    </div>
  </div>
</div>
```

**Close behaviour:** Click overlay, press Escape, or click X — all close the modal. Focus trapped inside while open.

---

### Alerts / Toasts

```jsx
/* TOAST — Success */
<div className="flex items-start gap-3 p-4 bg-[#E5F4F2] border border-[#0A7B6C]/20 rounded-lg">
  <CheckCircle className="w-5 h-5 text-[#0A7B6C] flex-shrink-0 mt-0.5" />
  <div>
    <p className="text-sm font-semibold text-[#065E52]">Refund Processed</p>
    <p className="text-xs text-[#0A7B6C] mt-0.5">₹17,000 will be credited in 5–7 business days</p>
  </div>
</div>

/* TOAST — Warning */
<div className="flex items-start gap-3 p-4 bg-[#FDF6E3] border border-[#C48E00]/20 rounded-lg">
  <AlertTriangle className="w-5 h-5 text-[#C48E00] flex-shrink-0 mt-0.5" />
  <div>
    <p className="text-sm font-semibold text-[#9A6F00]">Return Reminder</p>
    <p className="text-xs text-[#C48E00] mt-0.5">5 days remaining. Schedule your return now.</p>
  </div>
</div>

/* TOAST — Error */
<div className="flex items-start gap-3 p-4 bg-[#FCEEF2] border border-[#C0365A]/20 rounded-lg">
  <XCircle className="w-5 h-5 text-[#C0365A] flex-shrink-0 mt-0.5" />
  <div>
    <p className="text-sm font-semibold text-[#951D44]">Payment Failed</p>
    <p className="text-xs text-[#C0365A] mt-0.5">Your payment could not be processed. Please try again.</p>
  </div>
</div>

/* TOAST — Info */
<div className="flex items-start gap-3 p-4 bg-[#EEEAFC] border border-[#3B2F8A]/20 rounded-lg">
  <Info className="w-5 h-5 text-[#3B2F8A] flex-shrink-0 mt-0.5" />
  <div>
    <p className="text-sm font-semibold text-[#2A2167]">KYC Under Review</p>
    <p className="text-xs text-[#3B2F8A] mt-0.5">Your documents are being verified. Usually done in 24 hours.</p>
  </div>
</div>
```

---

### Loading States

```jsx
/* SKELETON — Listing Card */
<div className="bg-white rounded-xl border border-[#EDE6E0] overflow-hidden animate-pulse">
  <div className="aspect-[4/5] bg-[#EDE6E0]" />
  <div className="p-4 space-y-2">
    <div className="h-4 bg-[#EDE6E0] rounded w-3/4" />
    <div className="h-3 bg-[#EDE6E0] rounded w-1/2" />
    <div className="h-5 bg-[#EDE6E0] rounded w-1/3 mt-3" />
  </div>
</div>

/* FULL PAGE SPINNER (rare — use skeletons instead) */
<div className="flex items-center justify-center h-48">
  <div className="w-8 h-8 border-2 border-[#FF6B1A] border-t-transparent rounded-full animate-spin" />
</div>

/* INLINE BUTTON SPINNER */
<svg className="animate-spin h-4 w-4" viewBox="0 0 24 24" fill="none">
  <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4" />
  <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z" />
</svg>
```

---

### Empty States

```jsx
/* STANDARD EMPTY STATE */
<div className="flex flex-col items-center justify-center py-16 px-6 text-center">
  <div className="w-16 h-16 bg-[#F5F0EE] rounded-2xl flex items-center justify-center mb-4">
    <ShoppingBag className="w-8 h-8 text-[#B8A99F]" />
  </div>
  <h3 className="text-base font-semibold text-[#1A1614] mb-2">No listings found</h3>
  <p className="text-sm text-[#6B5E58] max-w-xs mb-6">
    Try removing some filters or search for something else.
  </p>
  <button className="... outline-button">Clear Filters</button>
</div>
```

---

## 5. Accessibility Guidelines

- **WCAG Standard:** 2.1 Level AA compliance for web admin panel
- **Color Contrast:** Minimum 4.5:1 for all text on background; 3:1 for large text (≥18px)
- **Focus Indicators:** 2px solid `#FF6B1A` ring, 2px offset — never `outline: none` without replacement
- **Touch Targets:** Minimum 44×44px for all interactive elements on mobile
- **ARIA Requirements:**
  - All modals: `role="dialog"`, `aria-modal="true"`, `aria-labelledby`
  - All form inputs: `<label>` associated via `htmlFor` or `aria-label`
  - Icon-only buttons: `aria-label` describing action
  - Status updates (order state change): `aria-live="polite"`
  - Errors: `aria-describedby` linking input to error message
- **Keyboard Navigation:** All flows completable via keyboard; Tab order follows visual order; modals trap focus

---

## 6. Animation Guidelines

- **Default duration:** 200ms
- **Page transitions:** 300ms
- **Modal enter:** `slide-in-from-bottom` on mobile, `zoom-in-95` on desktop — 200ms ease-out
- **Easing:** `ease-out` for entrances; `ease-in` for exits; `ease-in-out` for transforms
- **What to animate:** Only `transform` and `opacity` — never animate `height`, `width`, `margin`, `padding` (causes layout thrash)
- **Skeleton shimmer:** `animate-pulse` (Tailwind built-in) — use on all loading states
- **Reduced motion:** All animations wrapped in `@media (prefers-reduced-motion: reduce) { animation: none; }`

---

## 7. Icon System

- **Library:** Lucide React 0.359.0 — https://lucide.dev
- **Standard sizes:** 16px (inline text icons), 20px (button icons), 24px (navigation, feature icons), 32px (empty state icons)
- **Usage patterns:**
  - Always pair with text where possible (never icon-only CTAs without `aria-label`)
  - `strokeWidth={1.5}` for display/feature icons; `strokeWidth={2}` for UI icons (buttons, inputs)
  - Icons must inherit `currentColor` — never hardcode icon fill color

**Key Icons Used:**
```
ShoppingBag — Listings, buyer flow
Store — Seller profile
Package — Orders, delivery
Camera — Inspection, photo upload
Shield — Trust, verification, escrow
Star — Rating, reviews
Clock — Countdown, timeline
AlertTriangle — Warnings, Day 20+ alerts
CheckCircle — Success, confirmed
XCircle — Error, failed
ArrowLeft — Back navigation
ChevronRight — List item navigation
```

---

## 8. Responsive Design

### Mobile-First Approach
All components styled for 375–430px viewport first; breakpoints layer on top via Tailwind's `md:`, `lg:` prefixes.

### Touch Targets
- All buttons, list items, and tappable cards: minimum `min-h-[44px]` or `min-w-[44px]`
- Bottom navigation tabs: `h-16` with centered icon + label

### Component Adaptations by Breakpoint

| Component | Mobile (<768px) | Desktop (≥1024px) |
|---|---|---|
| Navigation | Bottom tab bar (4 tabs) | Top nav bar with sidebar (admin) |
| Filter Panel | Full-screen bottom sheet | Inline left sidebar |
| PDP Layout | Single column, sticky bottom CTA | Two-column: photos left, details right |
| Checkout | One step per screen | Single page with side summary |
| Listing Grid | 2 columns | 3–4 columns |
| Modal | Bottom sheet (slide up, full width) | Centered dialog (max-w-md) |
| Escrow Breakdown | Collapsible accordion | Always-visible card |
| Admin Tables | Not shown on mobile | Full table with column visibility toggles |
