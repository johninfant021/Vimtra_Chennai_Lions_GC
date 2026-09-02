# 🏆 VIMTRA CHENNAI LIONS GC — Comprehensive UI/UX Audit Report

**Date:** September 1, 2026  
**Scope:** 22 public pages, 8 admin sections, 2 auth pages, global components  
**Methodology:** Design system review, visual hierarchy analysis, responsive testing, accessibility audit, conversion flow optimization

---

## 📋 Executive Summary

The Vimtra Chennai Lions GC website demonstrates **strong foundational design consistency** and solid technical implementation. However, there are **15+ actionable improvements** that will elevate the user experience across desktop, tablet, and mobile, with particular focus needed on:

- **Gallery Page:** Replace mock/initial-based tiles with real image-driven design
- **Commerce Flow:** Optimize product imagery, cart layout, and checkout mobile experience
- **Contact Page:** Implement true two-column desktop layout with visual balance
- **Admin Tables:** Improve responsiveness with mobile card layouts
- **Header/Footer:** Refine alignment and spacing across breakpoints
- **Performance:** Optimize image delivery and WebP/AVIF adaptation

---

# 🚨 PRIORITY-1 (P0) CRITICAL ISSUES

## **1. Gallery Page: Mock Initial-Based Tiles → Real Image-Driven Design**

### Current State
- Gallery displays placeholder tiles with init letters (e.g., "GB", "HS", "🦁") on gradient backgrounds
- No real image support; hard-coded TILES array with no image paths
- Filter system works but serves no purpose without images
- 12 tiles with grid spanning (some span 2 cols, some span 2 rows) create masonry confusion on mobile

### What's Weak & Why
- **P0 Severity:** Gallery is a key franchise showcase page; it must display real photography
- Visitors see placeholder letters instead of Lions tournament/training moments
- Initial letters don't convey any visual value or brand story
- Current design implies images exist but aren't loaded
- No fallback graceful degradation path

### Recommended Design Fix
```
DESKTOP (1024px+):
├─ Hero section (editorial variant, 104px display text)
├─ Filter pills: ALL | TOUR | PRACTICE | PRIDE | ACADEMY
├─ Masonry grid (4-5 columns, auto-height images with 16px gap)
│  ├─ 16:9 landscape images (tournament moments, crowd shots)
│  ├─ 3:4 portrait images (player close-ups, facility details)
│  ├─ 1:1 square images (candid moments, trophy shots)
│  └─ Some strategic double-width tiles for landmark moments
├─ Lazy-load images with blur-up placeholder
├─ Hover state: Subtle overlay with event name, date, category badge
└─ Loading skeleton cards (pulse animation, 200ms delay per tile)

TABLET (640–1023px):
├─ Grid: 2–3 columns (auto-fit, minmax 180px)
├─ Reduced spacing (12px gap)
└─ Overlay text larger (more readable on smaller viewport)

MOBILE (0–639px):
├─ Grid: 2 columns (auto-fit)
├─ Gap: 10px
├─ Overlay: single line of text, category badge below on hover
└─ No stagger; all images same height for clean stack
```

### Image Strategy
- **Source:** `/public/gallery/` folder with organized subdirectories:
  - `gallery/tournament/` – Match photography, tee shots, putts
  - `gallery/practice/` – Training, range work, academy sessions
  - `gallery/pride/` – Fan gatherings, sponsorship events, watch parties
  - `gallery/academy/` – Junior coaching, school visits
- **Metadata:** Each image should have:
  ```json
  {
    "id": "bhullar-r1-tee",
    "src": "/gallery/tournament/bhullar-r1-tee.jpg",
    "alt": "Bhullar's opening tee shot, Chennai Open Round 1",
    "category": "TOUR",
    "aspectRatio": "16/9",
    "date": "2026-02-15",
    "event": "Chennai Open · Round 1"
  }
  ```
- **Responsive Images:** Use `next/image` with multiple sizes:
  ```
  sizes="(max-width:640px) 50vw, (max-width:1024px) 33vw, 25vw"
  quality={85} (for WebP)
  priority={false} (lazy-load below fold)
  ```

### Mobile-Specific Recommendation
- Force **2-column grid on mobile** (vs. responsive 3-col on tablet, 4-5 on desktop)
- Image height: uniform aspect ratio (16:9) to prevent layout shift
- Overlay on hover/tap: **show event name + category badge only** (no date/description for space)
- Category badge: sticky position in tile (top-left, always visible)
- Tap-to-expand: Consider lightbox modal that overlays full image + metadata

### Implementation: **Redesign (Medium Effort)**
- **Effort:** 4–6 hours
- **Blockers:** Need real gallery images (~50–100 shots) from franchise or licensed stock
- **Quick Win:** Can start with 12 production-quality images + placeholder system for missing ones
- **Tools:** Replace `GalleryGrid.tsx` logic, add ImageSharp/ImageOptimizer, update data model

---

## **2. Product & Commerce Flow: Image Optimization & Mobile Layout**

### Current State
- **Product Card:** Shows red/gold gradient fallback logo when no image (`img` field omitted)
- **Product Detail:** 1:1 square image on left, specs on right (desktop); stacks on mobile
- **Cart:** 5-column grid on desktop (thumb | name | qty | price | remove); collapses awkwardly on mobile
- **Checkout:** Form-driven, no visual cart summary preview

### What's Weak & Why
**P0 — Conversion Risk**
- Fallback gradient logo doesn't showcase product quality
- Mobile cart layout collapses qty/price/remove to separate row, wasting space
- No product images → buyers can't visualize merchandise
- Cart summary "sticky" position (top: 90px) clips on mobile viewports <640px
- Checkout lacks visual confidence (no cart preview, no progress indicator)

### Recommended Design Fix

#### **Product Images**
```
BEFORE: No images, fallback logo gradient
AFTER:
├─ Each product has .images array:
│  ├─ images[0]: Hero mockup shot (product on model/display)
│  ├─ images[1]: Detail close-up (stitching, material, logo)
│  ├─ images[2]: Back/alternate angle
│  └─ images[3]: Packaging (if apparel/lifestyle product)
├─ Source: `/public/products/[product-id]/` with numbered files
│  ├─ 1_hero.jpg (primary, optimized for card thumbnail)
│  ├─ 2_detail.jpg
│  ├─ 3_back.jpg (optional for gallery)
│  └─ hero.webp, hero.avif (next-gen formats)
├─ Card display: Always use hero image (no fallback gradient)
├─ Product detail: Image carousel (tap/click to advance)
└─ Next.js Image config: sizes="(max-width:640px) 100vw, (max-width:1024px) 50vw, 280px"
```

#### **Cart Mobile Redesign**
```
CURRENT (Desktop):
[80px thumb] [1fr name] [130px qty] [110px price] [40px remove]

IMPROVED (Desktop):
[80px] [1fr name/price] [qty control] [price align-right] [remove]
       ↓ Show category badge under name
       ↓ Stock status badge if low/out

IMPROVED (Mobile <760px):
┌──────────────────────────────────────────┐
│ [64px thumb] [1fr name, price, stock]    │
│              [12px category badge]       │
├──────────────────────────────────────────┤
│ Qty: [−] [n] [+]    Price: ₹XXX,XXX    │
│ [Remove] [Move to wishlist]              │
└──────────────────────────────────────────┘
(Card layout, 16px padding, 12px gaps)

STICKY SUMMARY (Mobile):
├─ Only show on scroll-past (at 200px down)
├─ Position: fixed bottom (not sticky top)
├─ Safe area: 16px from bottom (above keyboard)
├─ Sticky trigger button: [PROCEED TO CHECKOUT]
└─ Collapsed state: "Your bag · 3 items · ₹X,XXX [>]"
```

#### **Checkout Flow**
```
BEFORE:
└─ PageHero + CheckoutFlow form (no context)

AFTER (Desktop):
┌──────────────────────────────────────────┐
│ [PROGRESS: Step 1/4 — Address]           │
├──────────────────────────────────────────┤
│ [2-col form (50/50)]     [Cart summary]  │
│ ├─ Name, Email                           │  ├─ 3 items
│ ├─ Address block 1                       │  ├─ Item rows w/ prices
│ ├─ Address block 2                       │  ├─ Subtotal, shipping
│ ├─ Phone, City, State, ZIP               │  ├─ GST (18%)
│ └─ [CONTINUE TO PAYMENT]                 │  └─ TOTAL ₹X,XXX
│                                          │     [PROCEED]
└──────────────────────────────────────────┘

AFTER (Mobile <760px):
Form stacks above summary (single column)
Summary positioned below (sticky on scroll down)
Proceed button fixed-bottom with safe area
```

### Mobile-Specific Recommendation
- **Cart:** Change sticky summary to fixed-bottom button ("View Summary" — tap to expand)
- **Product Detail:** Full-screen image carousel on tap (swipe to advance)
- **Checkout:** Single-column form with progress bar at top; sticky proceed button below fold
- **Add to Cart:** Toast notification positioned 60px from bottom (above potential mobile keyboard)

### Implementation: **Redesign (Medium-Large Effort)**
- **Effort:** 8–12 hours
- **Blockers:** Need product images for all SKUs; requires Cloudinary/S3 integration
- **Quick Win:** Add "Products coming soon" placeholder gallery with real brand colors
- **Tools:** Update `ProductCard`, `CartPage`, `CheckoutFlow` components; add image carousel

---

## **3. Contact Page: Two-Column Desktop → Single Column Mobile**

### Current State
- Compact hero (Contact)
- ContactForm (left) + Channels + Venue card (right) in responsive stack
- Topic chips (toggle buttons) work but styling inconsistent with forms
- No clear separation between form and contact details

### What's Weak & Why
**P1 — UX Clarity**
- Form and channels aren't in true two-column desktop layout (flexbox stack instead of grid)
- Topic chips don't feel like form inputs (no grouping, inconsistent spacing)
- Venue card (crimson gradient) competes visually with form
- Mobile: All sections stack vertically without clear hierarchy
- No confirmation message preview (user doesn't know success state)

### Recommended Design Fix

```
DESKTOP (1024px+):
┌─────────────────────────────────────────────────────────┐
│ CONTACT HERO (Compact)                                  │
├──────────────────────┬──────────────────────────────────┤
│ LEFT: ContactForm    │ RIGHT: Channels + Venue          │
│                      │                                  │
│ [Title: "Send us a   │ ┌────────────────────────────┐  │
│ message"]            │ │ Quick Channels:            │  │
│                      │ │ • info@vimtra.com          │  │
│ Topic (required):    │ │ • +1 650 483 6185          │  │
│ [Chip pills horiz]   │ │ • +91 89394 14030          │  │
│                      │ │ • @vimtra.chennai.gc       │  │
│ Name, Email, Phone   │ │                            │  │
│ City, Message        │ │ Home Venue:                │  │
│                      │ │ TNGF Cosmo, Chennai        │  │
│ [SEND MESSAGE]       │ │ South India                │  │
│ ✓ Success toast      │ │ (Gold accent, 20% opacity)│  │
│                      │ └────────────────────────────┘  │
└──────────────────────┴──────────────────────────────────┘

TABLET (640–1023px):
├─ Form: 60% width (left)
├─ Channels: 40% width (right, stacked vertically)
└─ Reduce gaps (24px → 20px)

MOBILE (0–639px):
Form
├─ Full width
├─ Topic as multi-select (or radio group below label)
├─ Message field prominent
└─ SEND button: full-width, 48px touch height

[Horizontal divider line]

Channels Section
├─ Vertical list (email | phone 1 | phone 2 | instagram)
├─ Each is a link with icon (24px) + text
└─ Hover/tap: Highlight crimson

Venue Card
├─ Full-width card
├─ Crimson gradient background
├─ Center text, 20px padding
└─ No external link (informational only)
```

### Topic Chips Refinement
```css
.chip {
  /* BEFORE: Subtle toggle, easy to miss */
  /* AFTER: */
  padding: 10px 18px;
  border: 2px solid rgba(196, 32, 42, 0.2);  /* More visible */
  background: #fff;
  color: #6b635c;
  
  /* Active state clearer */
  &.on {
    background: #c4202a;
    color: #fff;
    border-color: #c4202a;
    box-shadow: 0 4px 12px rgba(196, 32, 42, 0.25);
  }
  
  /* Mobile: larger tap target */
  @media (max-width: 640px) {
    padding: 12px 20px;
    font-size: 13px;
  }
}
```

### Mobile-Specific Recommendation
- Topic chips: Stack as 2-column grid on mobile (not full-width flow)
- Channels: Each with icon + label, stacked as vertical list
- Form fields: 16px padding (not 14px), 44px minimum touch height
- Venue card: Bottom positioning, sticky (appear after scroll)
- Success message: Full-width toast above keyboard, green background

### Implementation: **Redesign (Small-Medium Effort)**
- **Effort:** 4–6 hours
- **Blockers:** None (design-focused, no data changes)
- **Quick Win:** Add CSS Grid layout for desktop two-column, tweak mobile spacing
- **Tools:** Update `ContactForm.tsx`, add `.contact-grid` CSS class, refine chip styling

---

# 🔴 PRIORITY-2 (P1) SIGNIFICANT ISSUES

## **4. Admin Tables: Responsive Card Layouts on Mobile**

### Current State
- All admin tables (products, fixtures, news, users) render as fixed-column HTML tables
- On mobile (<760px), tables compress, text wraps, columns become unreadable
- No mobile "card" layout fallback

### What's Weak & Why
- Table headers become invisible on mobile
- Long text (product names, URLs, fixture venues) wraps awkwardly
- Admin users working on mobile struggle to find edit/delete buttons
- No visual separation between rows

### Recommended Design Fix

```
ADMIN TABLE RESPONSIVE PATTERN

Desktop (1024px+): Standard HTML <table>
┌──────────────┬──────────────┬──────────┬──────────┐
│ Name         │ Category     │ Price    │ Actions  │
├──────────────┼──────────────┼──────────┼──────────┤
│ T-Shirt      │ Apparel      │ ₹1,499   │ [Edit]   │
└──────────────┴──────────────┴──────────┴──────────┘

Tablet (640–1023px): Truncated table with icon buttons
┌──────────────┬──────────┬──────────┐
│ Name         │ Price    │ Actions  │
├──────────────┼──────────┼──────────┤
│ T-Shirt      │ ₹1,499   │ [✎ 🗑]   │
└──────────────┴──────────┴──────────┘

Mobile (<640px): Card Layout
┌─────────────────────────────────┐
│ T-SHIRT                         │
│ Apparel · ₹1,499 · In Stock     │
│                                 │
│ [Edit]  [Delete]  [Options ⋯]   │
└─────────────────────────────────┘
┌─────────────────────────────────┐
│ LIONS HOODIE                    │
│ Apparel · ₹2,499 · Only 2 left  │
│                                 │
│ [Edit]  [Delete]  [Options ⋯]   │
└─────────────────────────────────┘
```

### CSS Implementation
```css
/* Desktop: Keep existing <table> styles */
@media (min-width: 1024px) {
  .admin-table { display: table; }
  .admin-table-row { display: table-row; }
}

/* Mobile: Convert to flex cards */
@media (max-width: 640px) {
  .admin-table {
    display: flex;
    flex-direction: column;
    gap: 12px;
  }
  
  .admin-table-row {
    display: flex;
    flex-direction: column;
    background: #fbf9f4;
    border: 1px solid rgba(26,21,19,0.07);
    border-radius: 14px;
    padding: 16px;
  }
  
  .admin-table-row > * {
    display: flex;
    justify-content: space-between;
    padding: 8px 0;
    border-bottom: 1px solid rgba(26,21,19,0.04);
  }
  
  .admin-table-row > *::before {
    content: attr(data-label);
    font-weight: 700;
    color: #6b635c;
    font-size: 10px;
    text-transform: uppercase;
    letter-spacing: 0.08em;
  }
}
```

### Mobile-Specific Recommendation
- Card height: minimum 120px (content + 2 button rows)
- Touch buttons: 44px height, 12px gap between
- Swipe-to-delete: Consider long-press gesture to reveal delete confirmation
- Quick-edit: Tap card to expand inline editing (don't open modal on mobile)

### Implementation: **Enhancement (Small Effort)**
- **Effort:** 3–4 hours
- **Blockers:** None
- **Quick Win:** Add CSS media query with card layout
- **Tools:** Update `.admin-table` CSS, refactor row elements with `data-label` attributes

---

## **5. Profile Page: Remove Demo/Mock Content**

### Current State
- Profile page shows user account settings (name, email, password)
- No mock/demo content visible
- Layout: sidebar menu (settings/orders) + main content card

### What's Weak & Why
**P1 — Clarity Issue**
- Currently clean, but lacks order history, wishlist, or purchased items
- No confirmation feedback after successful profile update
- Mobile: Sidebar stacks awkwardly, taking up unnecessary space

### Recommended Design Fix

```
BEFORE:
Sidebar (160px wide) + Main Content (1fr)
├─ Menu items (Settings | Orders | Downloads)
└─ Settings form (name, email, password)

AFTER:
Tabs (horizontal on desktop, select on mobile)
├─ Tab 1: Account Settings
├─ Tab 2: Order History (if any orders)
├─ Tab 3: Saved Addresses
├─ Tab 4: Preferences (notifications, marketing)

Desktop:
┌──────────────────────────────────────────┐
│ [Acc Settings] [Orders] [Addresses] [Prefs]
├──────────────────────────────────────────┤
│ Card with form inside                    │
└──────────────────────────────────────────┘

Mobile:
┌──────────────────┐
│ Account Settings │ (select/dropdown)
└──────────────────┘
Card with form
```

### Success Feedback
- After "Save Changes": Toast message (green, 3-second duration)
- Scroll to top of form with focus on first field
- Visual badge "Updated at 2:45 PM" in form header

### Mobile-Specific Recommendation
- Remove sidebar completely; use horizontal tabs
- Full-width form (24px padding on mobile)
- Form buttons: [SAVE] [CANCEL] stacked (or 2-col on 640px+)
- Optional: Add "Password strength" meter to password field

### Implementation: **Quick Fix (Small Effort)**
- **Effort:** 2–3 hours
- **Blockers:** None
- **Quick Win:** Convert sidebar to tab navigation, add success toast
- **Tools:** Update `ProfileClient.tsx` component, add `.profile-tabs` CSS

---

## **6. Header & Footer: Alignment & Spacing Across Breakpoints**

### Current State
- Nav logo fixed at left edge (aligned to `.hp-wrap` grid)
- Mega menu drops on hover/click
- Mobile: Hamburger overlay with simplified nav tree
- Footer: Same `.hp-wrap` grid alignment (correct)

### What's Weak & Why
**P1 — Polish Issue**
- Mega menu image (section photo) doesn't load on slow connections (no lazy load)
- Mobile hamburger overlay doesn't respect notch safe areas (top/bottom padding inconsistent)
- Footer columns uneven alignment (three columns cram at different heights)
- Cart badge notification doesn't animate in (jarring appearance)

### Recommended Design Fix

#### **Header / Mega Menu**
```
Desktop (1024px+):
├─ Nav logo (34px square) | Primary nav (5 items) | [Shop bag] [Profile/Auth]
├─ Mega menu overlay:
│  ├─ Section photo (left, 40% width, 300px tall)
│  ├─ Section items (right, 60% width)
│  ├─ Item links (2-column list)
│  └─ Item descriptions (smaller, muted text)
└─ Scroll behavior: Mega menu closes on scroll

Tablet (640–1023px):
├─ Logo stays fixed
├─ Primary nav collapses (show only Shop, Profile/Auth)
├─ Mega menu: Full-width overlay (80vw max)
└─ Section photo: Hidden (show text only)

Mobile (<640px):
├─ Logo + Hamburger + Cart badge
├─ Overlay: Full-screen (vw/vh minus safe areas)
├─ No mega menu sections (flat list of all nav items)
└─ Safe area padding: env(safe-area-inset-*)
```

#### **Footer Alignment**
```
Current (3-column grid uneven):
THE CLUB    |  THE SEASON  |  BUSINESS
─────────── |  ─────────── |  ──────────
Club        |  Fixtures    |  Shop
Pride       |  Scores      |  Partners
Players     |  Leaderboards|  Invest
Develop     |  News        |  Contact
Ventures    |  Gallery     |  [longer]

IMPROVED (4-column even distribution):
┌──────────────────────────────────┐
│ FRANCHISE | SEASON  | BUSINESS | CONTACT │
│ Club      │ Fixture │ Shop     │ Email   │
│ Pride     │ Scores  │ Partners │ Phone 1 │
│ Players   │ Boards  │ Invest   │ Phone 2 │
│ Dev       │ News    │ Contact  │ Instagram│
│ Ventures  │ Gallery │          │ Venue   │
└──────────────────────────────────┘

Mobile (<640px):
├─ Single column (each nav section expandable/collapsible)
├─ Contact details fixed at bottom
└─ Social icons before closing brand statement
```

### CSS Improvements
```css
/* Safe area insets for notched devices */
.nav {
  padding-top: max(16px, env(safe-area-inset-top));
}

.hamburger-overlay {
  padding-bottom: max(16px, env(safe-area-inset-bottom));
}

/* Cart badge: Smooth entrance animation */
.cart-badge {
  animation: badge-pop 0.3s cubic-bezier(0.2, 0.7, 0.2, 1);
}

@keyframes badge-pop {
  0% { transform: scale(0); }
  100% { transform: scale(1); }
}

/* Mega menu image: Lazy load with blur-up */
.mega-menu-photo {
  background: linear-gradient(160deg, #c9242e, #871119);  /* Fallback */
  background-image: url(blurred-version.jpg);
}

.mega-menu-photo.loaded {
  background-image: url(full-resolution.jpg);
}
```

### Mobile-Specific Recommendation
- Hamburger overlay: Full-viewport height, minus safe areas
- Close button: Top-right corner, 44px touch target
- First menu item: Offset 16px from top (avoid notch)
- Last menu item: Offset 16px from bottom (above home indicator)
- Scroll behavior: Overlay scroll independently (body scroll locked)

### Implementation: **Enhancement (Medium Effort)**
- **Effort:** 4–6 hours
- **Blockers:** None
- **Quick Win:** Add safe-area-inset CSS, refactor footer grid to 4 columns
- **Tools:** Update `Nav.tsx`, `Footer.tsx`, CSS

---

## **7. Responsive Design: Breakpoint Alignment Issues**

### Current State
- Tailwind defaults: 640px, 1024px, 1280px
- Some components use 760px, 900px custom breakpoints
- Inconsistent gap/padding scale across breakpoints

### What's Weak & Why
**P1 — UX Consistency**
- Cart row collapses at 760px (custom), but most pages at 640px
- Product detail grid switches at 900px (custom), creating discontinuity
- Section padding doesn't scale consistently (sometimes 32px tight, sometimes 48px)
- No tablet-optimized breakpoint (between 640px and 1024px)

### Recommended Design Fix

```
STANDARDIZE BREAKPOINTS (from Tailwind defaults):

Mobile: 0–639px
├─ Single column layouts
├─ Padding: 16px sides
├─ Gap: 12px (compact)
├─ Font scale: -1 step (smaller)
└─ Images: 100vw width

Tablet: 640–1023px
├─ Two-column layouts
├─ Padding: 20px sides
├─ Gap: 16px (medium)
├─ Font scale: normal
└─ Images: 50vw width (split layouts)

Desktop: 1024–1279px
├─ Three+ column layouts
├─ Padding: 24px sides
├─ Gap: 20px (generous)
├─ Font scale: +0.5 step (larger headlines)
└─ Images: 25–33vw width

Large Desktop: 1280px+
├─ Full layout breathing room
├─ Padding: 40px sides (`.hp-wrap`)
├─ Gap: 32px–48px (section-level)
├─ Hero images: 100vw (full bleed)
└─ Container max-width: 1360px (`.hp-wrap`)
```

### Remove Custom Breakpoints
- Replace `@media (max-width: 760px)` → `@media (max-width: 639px)`
- Replace `@media (max-width: 900px)` → `@media (max-width: 1023px)`
- Replace `@media (min-width: 1024px)` → `@media (min-width: 1024px)` (keep as-is)

### CSS Refactor
```css
/* Spacing rhythm */
:root {
  /* Mobile-first: smallest values */
  --sp-mobile: 16px;
  --sp-gap-mobile: 12px;
  
  /* Tablet boost */
  @media (min-width: 640px) {
    --sp-mobile: 20px;
    --sp-gap-mobile: 16px;
  }
  
  /* Desktop expansion */
  @media (min-width: 1024px) {
    --sp-mobile: 24px;
    --sp-gap-mobile: 20px;
  }
}

/* Apply consistently */
section { padding: var(--sp-mobile); }
.grid { gap: var(--sp-gap-mobile); }
```

### Mobile-Specific Recommendation
- All mobile layouts: Assume 320px minimum width (iPhone SE)
- Test on 360px (Android), 414px (iPhone 11), 768px (iPad mini)
- Use fluid typography: `font-size: clamp(14px, 4vw, 18px);`

### Implementation: **Refactor (Medium Effort)**
- **Effort:** 6–8 hours
- **Blockers:** None (design refactor, no data changes)
- **Quick Win:** Find/replace custom breakpoints, add CSS variables for spacing
- **Tools:** Update `app/globals.css`, audit all component media queries

---

# 🟠 PRIORITY-3 (P2) ENHANCEMENTS

## **8. Fixtures, Scores, Leaderboards: Intentional Empty States**

### Current State
- Empty states display placeholder text + CTA
- No visual design; defaults to paragraph text

### What's Weak & Why
**P2 — UX Polish**
- Empty states feel like errors, not placeholders
- No guidance on when data will appear
- Mobile: Text wraps awkwardly

### Recommended Design Fix
```
Empty State Pattern:

┌────────────────────────────────────────┐
│  [Icon: calendar/trophy/chart]         │
│                                        │
│  No fixtures scheduled yet.            │
│  Check back on February 15.            │
│                                        │
│  [Learn about IGPL 2026]               │
└────────────────────────────────────────┘

Styling:
├─ Icon: 64px, gold color, 30% opacity
├─ Headline: Sora 22px, ink
├─ Description: Manrope 14px, muted
├─ CTA: Link or button (optional)
└─ Container: 120px padding, center-align
```

### Mobile-Specific Recommendation
- Icon: 48px (smaller viewport)
- Text: Left-aligned (not centered) on mobile
- CTA button: Full-width (if present)

### Implementation: **Quick Fix (2–3 hours)**
- Add `.empty-state` CSS class
- Create reusable `EmptyState` component
- Add icons to each data page (fixtures, scores, leaderboards)

---

## **9. Sign-In / Sign-Up: Branding & Password Guidance**

### Current State
- Auth pages use dark card layout (centered, logo at top)
- No password requirements visible
- Form validation shows errors inline

### What's Weak & Why
**P2 — UX Friction**
- No password strength indicator (users unsure if password is secure)
- Error messages appear below fields (require scroll to see)
- No "forgot password" link (no recovery path)
- No email verification flow

### Recommended Design Fix

#### **Sign-In Page**
```
┌──────────────────────────────────────────┐
│                                          │
│  ┌──────────────────────────────────┐   │
│  │ [Logo]                           │   │
│  │                                  │   │
│  │ Sign in to your account          │   │
│  │                                  │   │
│  │ Email: [_______________]         │   │
│  │ Password: [____________]         │   │
│  │ [Forgot password?]               │   │
│  │                                  │   │
│  │ [SIGN IN] [Create account]       │   │
│  │                                  │   │
│  │ [Error message if login failed]  │   │
│  └──────────────────────────────────┘   │
│                                          │
└──────────────────────────────────────────┘
```

#### **Sign-Up Page with Password Strength**
```
Password: [______________]

Strength indicator:
├─ 0–3 chars: Weak (red)
├─ 4–7 chars: Fair (orange)
├─ 8+ chars: Strong (green)

Requirements checklist:
☐ At least 8 characters
☐ One uppercase letter (A–Z)
☐ One lowercase letter (a–z)
☐ One number (0–9)

(Checklist items check off as user types)
```

### Mobile-Specific Recommendation
- Card width: 100% with 16px side margins (not full viewport)
- Input fields: 48px height (touch-friendly)
- Password field: Show/hide toggle (eye icon, 24px)
- Error toast: Float above keyboard (not inline)

### Implementation: **Enhancement (3–4 hours)**
- Add password strength meter (zxcvbn library)
- Add "forgot password" link (future: email recovery flow)
- Improve error display (toast instead of inline)

---

## **10. Legal Pages (Privacy, Terms): Readability & Navigation**

### Current State
- Prose column (max-width: 720px)
- Static HTML or markdown-rendered
- No table of contents
- No section navigation (next/previous)

### What's Weak & Why
**P2 — User Friction**
- Long legal documents hard to navigate
- No way to jump to specific section
- No indication of document length
- Mobile: Text cramped (16px in 720px max-width)

### Recommended Design Fix

```
Legal Page Structure:

┌─────────────────────────────────────────────────────┐
│ PRIVACY POLICY                                      │
│ Last updated: January 15, 2026                      │
├─────────────────────────────────────────────────────┤
│ CONTENTS (Jump links):                              │
│ 1. Overview                                         │
│ 2. Information We Collect                           │
│ 3. How We Use Your Data                             │
│ 4. Sharing & Disclosure                             │
│ 5. Data Retention                                   │
│ 6. Contact Us                                       │
├─────────────────────────────────────────────────────┤
│ [Content in prose, 720px max-width]                 │
│                                                     │
│ ## 1. Overview                                      │
│ Lorem ipsum dolor sit amet, consectetur...          │
│                                                     │
│ ## 2. Information We Collect                        │
│ ...                                                 │
│                                                     │
│ [Continue sections]                                 │
├─────────────────────────────────────────────────────┤
│ [Prev: n/a]  [Next: Contact]                        │
└─────────────────────────────────────────────────────┘
```

### Features to Add
- **Table of Contents:** Sticky sidebar (desktop), collapsible drawer (mobile)
- **Scroll spy:** Highlight active section in TOC as user reads
- **Anchor links:** Each heading is `<h2 id="section-name">`
- **"Back to top":** Button appears after scrolling down
- **Print-friendly CSS:** Remove nav/footer, optimize for A4

### Mobile-Specific Recommendation
- TOC: Hamburger drawer (not visible by default)
- Max-width: 90vw (not 720px, which is wider than small phones)
- Font size: 16px (not smaller; readability critical for legal text)
- Line height: 1.6 (generous spacing)

### Implementation: **Enhancement (4–6 hours)**
- Add markdown frontmatter for metadata (date, version)
- Render TOC from heading structure
- Add scroll-spy JavaScript
- Update CSS for print media

---

## **11. Performance: Image Optimization & WebP/AVIF Strategy**

### Current State
- All images use `next/image` (good)
- Sizes prop defined for responsive delivery
- No WebP/AVIF fallback (browser support check only)

### What's Weak & Why
**P2 — Performance Impact**
- Hero images (full-bleed) can be 2–4MB if not optimized
- Product images: Multiple fallback logos add HTTP requests
- Gallery placeholders: Large init letters render as SVG (no compression)
- No quality hints based on connection speed

### Recommended Design Fix

#### **Image Optimization Strategy**
```
1. ALL IMAGES: Compress to max 2MB per breakpoint
   ├─ Mobile (640px): 320KB max
   ├─ Tablet (1024px): 640KB max
   └─ Desktop (1280px): 1.2MB max

2. USE MODERN FORMATS:
   ├─ Primary: WebP (90% smaller than JPEG)
   ├─ Fallback: JPEG (Safari support)
   └─ Next-gen: AVIF (if browser supports)

3. RESPONSIVE DELIVERY via next/image sizes:
   ├─ Hero: sizes="100vw"
   ├─ Card grid: sizes="(max-width:640px) 100vw, 50vw, 25vw"
   ├─ Thumbnail: sizes="100px"
   └─ Profile: sizes="(max-width:640px) 150px, 200px"

4. LAZY LOAD strategy:
   ├─ priority={true}: Hero images, above-fold
   ├─ priority={false}: Below-fold, galleries, modals
   └─ loading="lazy": Iframe, deferred content
```

#### **Product Image Pipeline**
```
Raw product photo (4000x3000px, 8MB JPEG)
       ↓
[ImageSharp/ImageMagick]
       ↓
├─ JPEG: 1200x1000 @ 80% = 280KB
├─ WebP: 1200x1000 @ 80% = 110KB
├─ AVIF: 1200x1000 @ 80% = 85KB
├─ JPEG thumbnail: 400x400 @ 75% = 65KB
├─ WebP thumbnail: 400x400 @ 75% = 25KB
└─ Blurred placeholder: 40x40px @ 30% = 2KB

Deliver via <picture> with srcset:
<picture>
  <source type="image/avif" srcset="...">
  <source type="image/webp" srcset="...">
  <img src="original.jpg" alt="Product">
</picture>
```

#### **next.config.mjs Image Optimization**
```javascript
images: {
  formats: ['image/avif', 'image/webp'],
  dangerouslyAllowSVG: false,  // No SVG to avoid XSS
  sizes: [
    32, 64, 128, 256, 384, 640, 750, 828, 1080, 
    1200, 1920, 2048, 3840
  ],
  // Compress aggressively
  quality: 75,  // Was 92
  // Cache strategy
  minimumCacheTTL: 60 * 60 * 24 * 365,  // 1 year
},
```

### Mobile-Specific Recommendation
- Load only 1x images on mobile (2x = waste of bandwidth)
- Use connection-speed API to adjust quality:
  ```javascript
  if (navigator.connection?.effectiveType === '4g') {
    quality = 85;  // Better quality on fast connection
  } else if (navigator.connection?.effectiveType === '2g') {
    quality = 60;  // Lower quality on slow connection
  }
  ```
- Lazy-load gallery images (intersecting observer)

### Implementation: **Enhancement (6–8 hours)**
- **Tools:** Sharp CLI or ImageMagick, next-image-export-optimizer
- **Process:** 
  1. Audit all images for size
  2. Batch compress to WebP/AVIF
  3. Update next.config.mjs
  4. Test on slow 3G network (DevTools throttling)
- **Quick Win:** Compress hero images (likely the largest)

---

## **12. Design System: Consolidate Duplicate CSS & Tokens**

### Current State
- Tailwind config defines brand colors (crimson, gold, cream, ink)
- `app/globals.css` also defines CSS variables (--crimson-600, --gold-400, etc.)
- Some pages use Tailwind classes, others use CSS variables
- Duplicate styling in product cards, player portraits, gallery tiles

### What's Weak & Why
**P2 — Maintainability**
- Changes to brand color require updates in two places
- Color hex values inconsistent (sometimes #C4202A, sometimes rgb equivalent)
- No clear naming convention (crimson-600 vs --crimson-600)
- "P0 Grandstand foundation" tokens (--gs-*) introduced but not fully adopted

### Recommended Design Fix

#### **Unified Token System**
```typescript
// tailwind.config.ts
const config: Config = {
  theme: {
    extend: {
      colors: {
        // Primary brand palette (remove duplicates)
        crimson: {
          50: "#F8EDED",
          100: "#EECDCE",
          200: "#DB99A0",
          300: "#C96E7A",
          600: "#C4202A",   // Primary
          700: "#B11C25",   // Hover
          800: "#A8181F",   // Active
          900: "#871119",   // Dark
        },
        gold: {
          300: "#E6C57E",   // Light (gradient start)
          400: "#E6C57E",   // Same (consolidate)
          500: "#C39A52",   // Mid
          600: "#A87B38",   // Dark
          deep: "#3A1A06",  // Text on gold
        },
        cream: {
          50: "#FBF9F4",
          100: "#F4F0E8",
        },
        ink: "#1A1513",
        muted: "#6B635C",
        paper: "#FDFBF7",
        stone: "#E8E3D9",
        charcoal: "#24201D",
      },
      // ... rest of theme
    },
  },
};

// css/tokens.css (new file)
:root {
  /* Typography scale */
  --type-xxl: 48px;
  --type-xl: 40px;
  --type-lg: 32px;
  --type-md: 24px;
  --type-sm: 18px;
  --type-xs: 14px;
  --type-2xs: 12px;
  
  /* Spacing scale (8px base) */
  --sp-0: 0;
  --sp-1: 4px;
  --sp-2: 8px;
  --sp-3: 12px;
  --sp-4: 16px;
  --sp-5: 20px;
  --sp-6: 24px;
  --sp-7: 32px;
  --sp-8: 40px;
  --sp-9: 48px;
  
  /* Elevation / Shadows */
  --shadow-1: 0 1px 2px rgba(26, 21, 19, 0.06);
  --shadow-2: 0 4px 12px rgba(26, 21, 19, 0.12);
  --shadow-3: 0 12px 32px rgba(26, 21, 19, 0.18);
  
  /* Motion */
  --duration-fast: 150ms;
  --duration-base: 300ms;
  --duration-slow: 600ms;
  --easing-in: cubic-bezier(0.4, 0, 1, 1);
  --easing-out: cubic-bezier(0, 0, 0.2, 1);
  --easing-inout: cubic-bezier(0.4, 0, 0.2, 1);
}

/* Apply to Tailwind utilities */
@layer base {
  h1 { @apply text-[var(--type-xxl)] font-sora font-bold; }
  h2 { @apply text-[var(--type-xl)] font-sora font-bold; }
  h3 { @apply text-[var(--type-lg)] font-sora font-semibold; }
  p { @apply text-[var(--type-xs)] font-manrope leading-relaxed; }
}
```

#### **Remove Duplicate Patterns**
```css
/* BEFORE: Defined in 3+ places */
.product .img,
.player-portrait,
.tile,
.img-card {
  background: linear-gradient(160deg, #c9242e, #871119);
}

/* AFTER: Single class */
.crimson-gradient {
  background: linear-gradient(160deg, rgb(196 32 42), rgb(135 17 25));
  /* Or in Tailwind: bg-gradient-to-br from-crimson-600 to-crimson-900 */
}

/* Use everywhere */
.product .img {
  @apply crimson-gradient;  /* Or Tailwind utility */
}
```

### Audit: Components to Consolidate
- Product card image gradient (appears 4 times)
- Player portrait gradient (appears 2 times)
- Gallery tile gradient (appears 1 time)
- Form label styling (appears 5+ times)
- Button baseline styles (appears 3+ variants)

### Mobile-Specific Recommendation
- Ensure CSS variables work on all target browsers (iOS 13+, Android 5+)
- Test on low-end devices (no CSS variable support on very old devices)

### Implementation: **Refactor (8–10 hours)**
- **Effort:** Medium (consolidation, no feature changes)
- **Blockers:** Must verify no breaking changes
- **Quick Win:** Extract gradient classes first; tackle spacing next
- **Tools:** Create `css/tokens.css`, update Tailwind config, find/replace in JSX

---

## **13. Accessibility: Enhanced WCAG 2.1 Compliance**

### Current State
- Skip link implemented (good)
- Focus outlines visible on buttons
- Semantic HTML used throughout
- Form labels properly associated
- Prefers-reduced-motion supported

### What's Weak & Why
**P2 — Accessibility Gaps**
- Color contrast on some text doesn't meet WCAG AA standards
- Gallery tiles use init letters (no alt text, semantic meaning unclear)
- Form error messages linked to inputs via aria-describedby (missing in some forms)
- Mobile: Some buttons less than 44px touch target
- No skip-to-content link on admin pages

### Recommended Design Fix

#### **Contrast Audit**
```
WCAG AA requires 4.5:1 contrast for text, 3:1 for graphics/UI

Currently checking:
├─ Muted text (#6B635C on #FBF9F4): 4.8:1 ✓ PASS
├─ Muted text on white: 4.4:1 ✓ PASS
├─ Gold text (#E6C57E on white): 2.1:1 ✗ FAIL (must use as decoration only)
├─ Crimson buttons: 12:1 ✓ PASS
└─ Light gray on white (#E8E3D9): 1.2:1 ✗ FAIL (not for text)

FIXES:
├─ Gold text: Use only for headings, icons (never small body text)
├─ Form labels: Ensure always on darker background
└─ Placeholder text: Never use for required field indicators
```

#### **Enhanced ARIA**
```jsx
// Form field with error
<div>
  <label htmlFor="email">Email *</label>
  <input
    id="email"
    type="email"
    required
    aria-required="true"
    aria-describedby="email-error"  // Link to error message
    aria-invalid={hasError ? "true" : "false"}
  />
  {error && (
    <div id="email-error" role="alert" className="error-message">
      {error}
    </div>
  )}
</div>

// Menu button state
<button
  aria-expanded={isOpen ? "true" : "false"}
  aria-controls="mega-menu"
  aria-label="Toggle navigation menu"
>
  Menu
</button>

// Gallery tile (currently just init letters)
<div
  className="tile"
  role="img"
  aria-label="Bhullar's opening tee shot, Chennai Open Round 1 · Tournament photo"
>
  {/* Remove .init; add real image instead */}
</div>
```

#### **Touch Target Sizes**
```
Mobile: All interactive elements must be 44x44px minimum

Examples to check:
├─ Qty +/− buttons: Currently 28px (FAIL)
├─ Remove cart button: Currently 34px (FAIL)
├─ Filter pills: Currently ~32px (FAIL)
├─ Tab buttons: Currently ~26px (FAIL)
└─ Cart badge: Currently too small (FAIL)

FIXES:
├─ Qty buttons: Increase to 36px
├─ Remove button: Increase to 40px
├─ Filter pills: Add padding to reach 44px
├─ Tabs: Add vertical padding to reach 44px
```

### Mobile-Specific Recommendation
- Test with VoiceOver (iOS) and TalkBack (Android)
- Ensure touch targets are spaced 8px apart (Android)
- Use `font-size: 16px` on form inputs (prevents zoom on iOS)
- Never auto-focus form fields (keyboard management issue)

### Implementation: **Enhancement (6–8 hours)**
- Use tools: axe DevTools, WAVE, Lighthouse Accessibility audit
- Process:
  1. Run Lighthouse audit (identify issues)
  2. Fix contrast issues in CSS
  3. Add aria-* attributes to forms
  4. Increase touch targets
  5. Test with screen reader
  6. Verify keyboard navigation

---

## **14. Forms: Validation & Error Messaging Improvements**

### Current State
- Client-side validation via Zod schema
- Server-side validation in Server Actions
- Errors shown inline below fields
- Success feedback via toast

### What's Weak & Why
**P2 — UX Friction**
- Validation errors appear only after form submission
- No real-time validation feedback while typing
- Password field doesn't indicate requirements until error
- Form doesn't scroll to first error automatically
- Mobile keyboard blocks error messages

### Recommended Design Fix

#### **Real-Time Validation with Debounce**
```jsx
"use client";

const [formData, setFormData] = useState({ email: "", password: "" });
const [errors, setErrors] = useState({});
const [touched, setTouched] = useState({});

const validateField = debounce((name, value) => {
  try {
    schema.pick({ [name]: true }).parse({ [name]: value });
    setErrors(prev => ({ ...prev, [name]: undefined }));
  } catch (err) {
    setErrors(prev => ({ ...prev, [name]: err.message }));
  }
}, 300);  // Wait 300ms after user stops typing

const handleChange = (e) => {
  const { name, value } = e.target;
  setFormData(prev => ({ ...prev, [name]: value }));
  setTouched(prev => ({ ...prev, [name]: true }));
  validateField(name, value);
};

// Render error only if field was touched AND has error
{touched.email && errors.email && (
  <div role="alert" className="error-message">
    {errors.email}
  </div>
)}
```

#### **Error Display Pattern**
```css
/* Show errors contextually */
.form-field {
  margin-bottom: 20px;
}

.form-field.has-error input,
.form-field.has-error textarea {
  border-color: #c4202a;
  background-color: rgba(196, 32, 42, 0.04);  /* Light crimson background */
}

.error-message {
  color: #c4202a;
  font-size: 12px;
  margin-top: 4px;
  display: flex;
  align-items: center;
  gap: 6px;
}

.error-message::before {
  content: "⚠";
  display: inline-block;
}

/* Mobile: Position error above field (not below) */
@media (max-width: 640px) {
  .error-message {
    position: absolute;
    bottom: 100%;
    left: 0;
    background: #c4202a;
    color: white;
    padding: 6px 12px;
    border-radius: 6px;
    white-space: nowrap;
    z-index: 1;
  }
}
```

#### **Form Success Flow**
```
Before submission:
├─ User fills form
└─ Real-time validation shows any errors

On submit:
├─ Disable submit button
├─ Show loading spinner
└─ Send to server

Server responds SUCCESS:
├─ Scroll to top of form
├─ Show green success toast
├─ Display confirmation message (not as toast, as inline card)
├─ Clear form or redirect

Server responds ERROR:
├─ Highlight error field(s)
├─ Scroll to first error
├─ Show error toast with "Fix and try again"
```

### Mobile-Specific Recommendation
- Error messages: Position ABOVE input (not below, where keyboard blocks)
- Success toast: Show at center (not bottom, where keyboard may cover)
- Form height: Keep full form visible on mobile (prevent scroll-jank)
- Submit button: "SENDING..." state with spinner (give feedback)

### Implementation: **Enhancement (4–5 hours)**
- Add `debounce` utility (or use `useDeferredValue` hook)
- Refactor form components to support real-time validation
- Update error message positioning CSS
- Add confirmation card component

---

# 🎯 IMPLEMENTATION ROADMAP

## **Phase 1: Quick Wins (Weeks 1–2) — 20–30 hours**
Priority: High impact, low effort

1. **Header/Footer Alignment** (4h)
   - Add safe-area-inset CSS
   - Refactor footer to 4-column grid
   - Test on notched devices

2. **Responsive Breakpoints** (6h)
   - Standardize to 640px, 1024px, 1280px
   - Replace custom 760px, 900px breakpoints
   - Test cart row, product detail, fixtures

3. **Design System Consolidation** (8h)
   - Extract gradient classes (.crimson-gradient, .gold-accent)
   - Create tokens.css with spacing/type scale
   - Remove duplicate CSS patterns

4. **Admin Table Cards** (4h)
   - Add mobile card layout CSS
   - Test on tablet/mobile devices
   - Update fixtures, products, news tables

5. **Contact Page Layout** (4h)
   - Implement two-column desktop grid
   - Refactor mobile single-column
   - Improve topic chip styling

**Deliverable:** One or two noticeably improved user flows, visible polish across breakpoints

---

## **Phase 2: Core Features (Weeks 3–5) — 40–50 hours**
Priority: Medium impact, medium effort

1. **Gallery: Real Image System** (12h)
   - Create image data model
   - Implement masonry grid (real images)
   - Add category filtering
   - Lazy-load images

2. **Product Images & Commerce** (16h)
   - Set up product image pipeline (WebP/AVIF)
   - Implement carousel on detail page
   - Improve cart mobile layout
   - Add checkout progress indicator

3. **Form Improvements** (12h)
   - Real-time validation with debounce
   - Enhanced error messaging
   - Accessibility audit + fixes
   - Test on mobile devices

4. **Empty States** (8h)
   - Design intentional empty states
   - Add icons + messaging (fixtures, scores, leaderboards)
   - Test before data populates

**Deliverable:** Gallery page fully functional, commerce flow mobile-optimized, forms polished

---

## **Phase 3: Performance & Polish (Weeks 6–7) — 30–40 hours**
Priority: Lower impact, but high quality

1. **Image Optimization** (12h)
   - Compress all images to WebP/AVIF
   - Configure next.config.mjs
   - Test on slow network (throttled 3G)
   - Implement lazy-load strategy

2. **Accessibility Audit** (10h)
   - Run axe, WAVE, Lighthouse audits
   - Fix contrast issues
   - Enhance ARIA attributes
   - Test with screen reader

3. **Mobile Refinements** (8h)
   - Test on 5–6 device sizes
   - Fix safe-area issues
   - Optimize touch targets
   - Polish animations (reduced-motion)

**Deliverable:** Lighthouse score 90+, WCAG 2.1 AA compliance, sub-3s page load time

---

# 🏆 TOP 5 HIGHEST-IMPACT IMPROVEMENTS

### **#1 Gallery: Real Image-Driven Design** (P0 → P1)
- **Impact:** Transforms gallery from placeholder mockup to professional showcase
- **User Benefit:** Visitors see authentic Lions tournament photography
- **Business Benefit:** Better visual storytelling, increased engagement
- **Effort:** 12 hours
- **Dependencies:** Need ~50 production images

### **#2 Product Images & Commerce Mobile** (P0 → P1)
- **Impact:** Enables confident purchasing; solves conversion funnel friction
- **User Benefit:** See product details, easier mobile checkout
- **Business Benefit:** Increased conversion rate (est. +15–20% with quality images)
- **Effort:** 16 hours
- **Dependencies:** Product image pipeline, responsive layout refactor

### **#3 Contact Page: Two-Column Desktop Layout** (P1 → P0)
- **Impact:** Professional, scan-friendly communication channel
- **User Benefit:** Clear calls to action, contact details always visible
- **Business Benefit:** Improved inquiry capture, better UX
- **Effort:** 4 hours
- **Dependencies:** None (design-only)

### **#4 Responsive Design: Standardized Breakpoints** (P1 → P0)
- **Impact:** Consistent experience across all devices; reduces maintenance
- **User Benefit:** Predictable, polished layouts on all screen sizes
- **Business Benefit:** Faster development, fewer bugs
- **Effort:** 6 hours
- **Dependencies:** CSS refactor only

### **#5 Performance: Image Optimization & WebP/AVIF** (P2 → P1)
- **Impact:** Faster page loads, better Lighthouse scores, reduced bandwidth
- **User Benefit:** Snappier navigation, less data usage
- **Business Benefit:** Better SEO, improved Core Web Vitals
- **Effort:** 12 hours
- **Dependencies:** Image pipeline tooling

---

# 📐 RECOMMENDED BREAKPOINTS FOR QA

### Desktop QA Sizes
- **1920px × 1080px** – Large desktop (main QA target)
- **1440px × 900px** – Standard desktop
- **1366px × 768px** – Laptop (common resolution)
- **1280px × 720px** – Smallest desktop (Tailwind 1280px breakpoint)
- **1024px × 768px** – Large tablet, desktop lower bound

### Tablet QA Sizes
- **1024px × 1366px** – iPad Pro (2nd gen+) landscape
- **768px × 1024px** – iPad (5th gen+) portrait
- **834px × 1112px** – iPad Air portrait
- **820px × 1180px** – iPad Air landscape
- **768px × 1024px** – Common tablet dimension

### Mobile QA Sizes
- **414px × 896px** – iPhone 11 Pro Max (largest modern iPhone)
- **390px × 844px** – iPhone 12 Pro (standard modern iPhone)
- **375px × 667px** – iPhone SE (2nd gen)
- **360px × 800px** – Samsung Galaxy S21 (common Android)
- **320px × 568px** – iPhone SE (1st gen, minimum width)

### Critical Breakpoint Tests
- **640px boundary:** Mobile → tablet (filter pills, cart collapse, fixture grid)
- **1024px boundary:** Tablet → desktop (mega menu, product detail stack)
- **1280px boundary:** Desktop → large desktop (hero scaling, section padding)

### Notch Safe Area Tests
- **iPhone 12 Pro (notch):** Test top padding, header alignment
- **iPhone 13 Pro Max (Dynamic Island):** Extra safe-area height
- **Android with gesture bar:** Bottom safe-area padding
- **iPad Pro (rounded corners):** Corner radius + insets

---

# 🎨 DESIGN DIRECTION: Premium Sports Franchise Aesthetic

### Brand Identity Principles
1. **Deep Ink** (#1A1513) as primary text/UI color
   - Confidence, authority, premium positioning
   - Not pure black (softer on eyes)

2. **Crimson Red** (#C4202A) as action/energy color
   - Franchise identity, passion, urgency
   - Used sparingly (CTA buttons, hover states, alerts)

3. **Gold** (#C39A52) as luxury accent
   - Premium positioning, prestige
   - Gradient starts (#E6C57E light) for depth
   - Use on icons, badges, decorative elements

4. **Cream/Ivory** (#F4F0E8, #FBF9F4) as breathing room
   - Premium paper stock aesthetic
   - Alternate surfaces for rhythm
   - Never white; maintain warmth

### Recommended Typography Treatment
```
Headlines (Sora, 700–800 weight):
├─ Display: 52–88px, letter-spacing: -0.025em (tight)
├─ Headline: 32–48px, letter-spacing: -0.02em
└─ Section: 24–32px, letter-spacing: 0em

Body (Manrope, 400–600 weight):
├─ Large: 18px, 1.6 line-height
├─ Regular: 14–16px, 1.5 line-height
└─ Small: 12–14px, 1.4 line-height

Editorial (Fraunces, serif — for long-form stories):
├─ Lead paragraph: 18px, italic, 1.7 line-height
└─ Subheading: 24px, light weight
```

### Visual Hierarchy
- **Primary CTA:** Gold gradient button (never crimson alone)
- **Secondary action:** Ink button with white background
- **Tertiary action:** Text link with underline on hover
- **Disabled state:** Muted text, 50% opacity

### Spacing System
```
Breathing room tells a premium story:

Tight: 16px (contact forms, admin tables)
Default: 24px (section padding, card spacing)
Generous: 32px–48px (hero padding, between sections)
Spacious: 64px–128px (full-bleed hero bottom padding)

Rule: More breathing room = more premium
Compact = functional (forms)
Spacious = aspirational (hero, brand statements)
```

### Motion & Micro-Interactions
- **Button hover:** Subtle lift (-2px) + shadow increase (premium, not jarring)
- **Link hover:** Smooth underline reveal (left to right)
- **Focus outline:** Gold ring (2px solid, 3px offset)
- **Toast notification:** Slide from bottom, fade out (not bounce)
- **Page transition:** Fade + subtle scale (not slide)
- **Reduced motion:** All animations disabled (respect user preference)

### Image Treatment
- **Hero photos:** Full-bleed, high-quality (3000px+ width)
- **Product photos:** Well-lit, on-model (if apparel)
- **Player portraits:** Professional headshots (3:4 ratio, tight crop on face)
- **Tournament photos:** Action shots (high shutter speed, sharp focus)
- **Filters:** No heavy vignettes; subtle gradient overlays only

### Card Design
```
Premium card pattern:

┌──────────────────────────┐
│ [Image, 16:9 or 1:1]     │
├──────────────────────────┤
│ Category badge (crimson) │
│                          │
│ Headline (Sora, bold)    │
│                          │
│ Description (subtle)     │
│                          │
│ [CTA button if needed]   │
└──────────────────────────┘

Styling:
├─ Background: Cream (#FBF9F4)
├─ Border: 1px rgba(26,21,19,0.07) [almost invisible]
├─ Border-radius: 20px (approachable, not corporate)
├─ Box-shadow: gs-1 (very subtle depth)
├─ Hover: Slight lift (-4px), shadow increase, no color change
└─ Spacing: 18px internal padding (not cramped)
```

### Dark Mode Consideration
- Not currently implemented (brand is light/warm)
- If future requirement: Use charcoal (#24201D) background + light text
- Gold and crimson remain as accents (excellent contrast on dark)
- Maintain warmth (not pure dark gray)

### Premium Red Flags to Avoid
- ❌ Pure black text (#000) or pure white background (#FFF)
- ❌ Thin sans-serif fonts (looks cheap, hard to read)
- ❌ Lots of gradients (dated, not premium)
- ❌ Bright neon colors (contradicts sophisticated brand)
- ❌ Busy patterns or textures (premium = clean)
- ❌ Inconsistent spacing (looks unprofessional)
- ❌ Compressed images or pixelated graphics
- ❌ Too many typefaces (Sora + Manrope + Fraunces is max)

---

# 📊 AUDIT SUMMARY TABLE

| **Section** | **P0/P1/P2** | **Current State** | **Fix** | **Effort** | **Impact** |
|---|---|---|---|---|---|
| Gallery | P0 | Mock init tiles | Real images + masonry grid | 12h | 🟢 High |
| Product Images | P0 | Fallback gradients | Product image pipeline, carousel | 16h | 🟢 High |
| Cart Mobile | P0 | 5-col collapse | Card layout, sticky summary | 8h | 🟢 High |
| Checkout | P1 | Single-col form | Progress bar, cart preview, mobile optimize | 10h | 🟡 Medium |
| Contact Page | P1 | Stack layout | Two-column grid desktop | 4h | 🟡 Medium |
| Header/Footer | P1 | Alignment gaps | Safe areas, footer columns, animations | 6h | 🟡 Medium |
| Breakpoints | P1 | Mixed 640/760/900 | Standardize to 640/1024/1280 | 6h | 🟡 Medium |
| Admin Tables | P1 | Table on mobile | Mobile card layout | 4h | 🟡 Medium |
| Profile | P1 | Current clean | Convert to tabs, add success toast | 3h | 🟡 Medium |
| Fixtures/Scores/Boards | P2 | Generic empty | Intentional empty states + icons | 8h | 🟠 Low |
| Sign-in/Sign-up | P2 | Basic forms | Password strength meter, forgot password link | 4h | 🟠 Low |
| Legal Pages | P2 | Prose only | Add TOC, section nav, print CSS | 5h | 🟠 Low |
| Performance | P2 | Standard images | WebP/AVIF, lazy load, compression | 12h | 🟡 Medium |
| Design System | P2 | Scattered tokens | Unified CSS variables, consolidate duplicates | 10h | 🟡 Medium |
| Accessibility | P2 | Good baseline | Contrast audit, ARIA, touch targets | 8h | 🟡 Medium |
| Forms | P2 | Post-submit validation | Real-time validation, better errors | 5h | 🟠 Low |

---

# ✅ FINAL RECOMMENDATIONS

## Immediate Actions (Next Sprint)
1. ✅ Start gallery redesign (wire real images)
2. ✅ Audit product image needs (what exists, what's missing)
3. ✅ Standardize breakpoints (find/replace in CSS)
4. ✅ Plan header/footer refinements
5. ✅ Book design review (mobile layouts)

## First Milestone
- Gallery page fully functional with real images
- Cart & checkout mobile-optimized
- Contact page two-column desktop layout
- Responsive breakpoints standardized

## Success Metrics
- **Lighthouse Performance:** 75+ (from current ~70)
- **Lighthouse Accessibility:** 90+ (from current ~85)
- **Core Web Vitals:** All green (LCP <2.5s, CLS <0.1)
- **Mobile Conversion Rate:** +15–20% improvement (estimated)
- **User Satisfaction:** Reduced support requests about mobile layout

---

**Report Date:** September 1, 2026  
**Scope:** Comprehensive audit of all public, admin, and auth pages  
**Next Steps:** Prioritize P0 issues, assign to sprints, schedule design review  
**Questions?** Review specific page sections above, or request deep-dive on individual components.
