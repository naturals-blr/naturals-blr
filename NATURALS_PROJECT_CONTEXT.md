# Naturals Salon & Spa Bangalore — Full Project Context

> Complete reference document covering all systems built across all sessions.  
> Last updated: March 2026

---

## Table of Contents

1. [Business Overview](#1-business-overview)
2. [Stores](#2-stores)
3. [Infrastructure & Hosting](#3-infrastructure--hosting)
4. [Data Layer — Google Sheets](#4-data-layer--google-sheets)
5. [Website — Architecture](#5-website--architecture)
6. [Website — Pages](#6-website--pages)
7. [Website — Build Pipeline](#7-website--build-pipeline)
8. [Website — Design System](#8-website--design-system)
9. [Website — Key Features](#9-website--key-features)
10. [Review Automation Bot](#10-review-automation-bot)
11. [Button & Colour Standards](#11-button--colour-standards)
12. [Image Folder Structure](#12-image-folder-structure)
13. [Decisions & Conventions Log](#13-decisions--conventions-log)
14. [Pending / To-Do](#14-pending--to-do)

---

## 1. Business Overview

**Brand:** Naturals Salon & Spa  
**Location:** Bangalore, India  
**Scale:** 5 stores across Bangalore  
**Target audience:** 25–55 year olds, ~70% female  
**Positioning:** Premium but accessible. India's most trusted salon network, 25+ years expertise.  
**Price indicator:** `₹₹` (mid-premium)  
**GitHub Organisation:** `naturals-blr`  
**Live website:** `https://naturals-blr.github.io`

---

## 2. Stores

| Store ID | Store Name | Area | Pincode |
|---|---|---|---|
| `Store_N78` | Naturals JPNagar 5th Phase | JP Nagar | 560078 |
| `Store_N77` | Naturals Elements Mall | Nagavara | 560077 |
| `Store_N36` | Naturals Devasandra | KR Puram | 560036 |
| `Store_N05` | Naturals Frazer Town | Pulikeshi Nagar | 560005 |
| `Store_N43` | Naturals Hennur | Hennur Cross | 560043 |

### Store N78 — JPNagar 5th Phase
- **Address:** No. 305, Anand Onyx, 1st Floor, 15th Cross, 100 Ft Ring Road, JP Nagar 5th Phase, Bengaluru 560078
- **Phone:** +91 87926 42299
- **Landline:** +91 80 4130 4444
- **Email:** jpnagar2.naturals@gmail.com
- **Facebook:** Naturals.Salon.JPNagar5Phase
- **Instagram:** @Naturals.Salon.JPNagar5Phase

### Store Order (Display Priority)
```
Store_N78 → Store_N77 → Store_N36 → Store_N05 → Store_N43
```
This order is hardcoded in `build.py` as `STORE_ORDER`.

---

## 3. Infrastructure & Hosting

### GitHub
- **Organisation:** `naturals-blr`
- **Website repo:** `naturals-blr.github.io` (GitHub Pages, served at the root domain)
- **Review bot repo:** separate repo in the same org

### Hosting
- **Static site:** GitHub Pages (free, auto-deploys from `main` branch)
- **Build trigger:** GitHub Actions — runs on every push to `main`, or manual trigger

### Secrets / Environment Variables
| Variable | Used in | Purpose |
|---|---|---|
| `GTM_ID` | `build.py` | Google Tag Manager container ID (placeholder — not yet set) |
| `PIXEL_ID` | `build.py` | Meta/Facebook Pixel ID (placeholder — not yet set) |

### Python Dependencies (`requirements.txt`)
```
jinja2
requests
```

---

## 4. Data Layer — Google Sheets

**Sheet ID:** `1wy0_josh4L-C0GXWNnRG8QEO9F8km5SMrDefemChFUo`  
**Access:** Public (CSV export via `gviz/tq` endpoint — no auth required)

### Sheet Tabs

| Tab | Purpose |
|---|---|
| `store_details` | All store data — addresses, phones, hours, URLs |
| `services` | Full services menu with prices and store availability |
| `offers` | Active promotions per store with expiry dates |
| `stylists` | Stylist profiles with store assignment |

### `store_details` Key Columns
```
Store_ID, Store_Name, Area, Pincode, Address_Full,
Phone_Raw, Landline_Raw,
Appointment_URL, Google_Maps_URL, Google_Review_URL,
Store_Page_URL, Active_Status,
Facebook_URL, Instagram_URL,
Mon_Fri_Hours, Saturday_Hours, Sunday_Hours,
Hero_Number, seo_intro
```

### `services` Key Columns
```
Service_Name, Category, Description, Duration,
Regular_Cost, Member_Cost, Gender,
Store_N78, Store_N77, Store_N36, Store_N05, Store_N43
```
Store columns contain `Yes` / `No` — service appears on that store's page only if `Yes`.

### `offers` Key Columns
```
Store_ID, Offer_Title, Offer_Description,
Discount_Text, Valid_till, Image_URL
```
`Valid_till` format: `DD-Mon-YYYY` (e.g. `13-Mar-2026`)

### `stylists` Key Columns
```
Stylist_Name, Store_ID, Specialization, Experience, Active_Status
```

### Phone Normalisation (in `build.py`)
Raw value from sheet (e.g. `918792642299`) is split into three derived fields:
```python
Phone_Display  = "+91 87926 42299"   # shown to user
Phone_Tel      = "+918792642299"     # used in href="tel:..."
WhatsApp_Number = "918792642299"     # used in wa.me/... (no + prefix)
```

---

## 5. Website — Architecture

### Stack
- **Templates:** Jinja2 (`.html.j2` files)
- **Build script:** Python 3 (`build/build.py`)
- **Output:** Static HTML files committed to repo root
- **No JavaScript frameworks** — vanilla JS only
- **No external CSS frameworks** — custom CSS with CSS variables
- **Fonts:** Google Fonts — Cormorant Garamond (headings) + Jost (body)

### Template File Structure
```
build/
├── build.py                    # Main build script
└── templates/
    ├── macros.j2               # ALL shared macros (buttons, footer, header JS, sticky bar, etc.)
    ├── header.j2               # Global header partial (included by all pages)
    ├── index.html.j2           # Homepage
    ├── services.html.j2        # Services & Pricing page
    ├── offers.html.j2          # Offers page
    ├── contact.html.j2         # Contact page
    ├── about.html.j2           # About Us page
    ├── booking-policy.html.j2  # Booking Policy page
    ├── cancellation-policy.html.j2  # Cancellation Policy page
    └── store.html.j2           # Individual store page (rendered 5× — one per store)
```

### Generated Output Structure
```
/ (repo root)
├── index.html
├── services.html
├── offers.html
├── contact.html
├── about.html
├── booking-policy.html
├── cancellation-policy.html
└── stores/
    ├── jpnagar.html
    ├── elements-mall.html
    ├── devasandra.html
    ├── frazer-town.html
    └── hennur.html
```

### Jinja2 Environment Settings
```python
autoescape=False
trim_blocks=True
lstrip_blocks=True
```
Global functions injected: `is_yes`, `norm_gender`, `gtm_id`, `pixel_id`

---

## 6. Website — Pages

### Homepage (`index.html.j2`)
- **Active page marker:** `active_page = 'home'` (transparent header that becomes solid on scroll)
- **Sections:** Hero → 6 Service Category cards → Store cards → Featured Offers → Testimonials → CTA band → Footer
- **Store cards:** Each store shows Book (blue), WhatsApp (green), Call (grey) buttons — each triggers `storePickerThen()` to record store selection then fire action
- **Featured offers:** 3 nearest-expiry offers pulled from sheet
- **Service categories (homepage):**
  1. Haircuts & Styling
  2. Hair Colour & Highlights
  3. Hair Spa & Treatments
  4. Skin Care & Facials
  5. Bridal & Makeup
  6. Men's Grooming
- **Min prices:** Dynamically computed from sheet and shown as "from ₹X" on category cards

### Services Page (`services.html.j2`)
- **3-row sticky filter bar:**
  - Row 1 — **Location** (teal pills): All Stores + one per store
  - Row 2 — **Gender** (gold pills): All · Female · Male · Unisex
  - Row 3 — **Category** (lavender pills): dynamically built from JS, always wraps (never scrolls)
- **Filter logic:** Store change resets gender + rebuilds category row; Gender change rebuilds category row; Category filters are never reset by other filters
- **Services rendered** client-side from baked-in JSON (`ALL_DATA`)
- **URL params:** `?store=Store_N78&gender=female&cat=facial` — all pre-applicable on load

### Store Pages (`store.html.j2`)
- **Base path:** `../` (pages live in `stores/` subdirectory)
- **Active page marker:** `active_page = 'stores'`
- **Sections:** Hero → Info bar → About → Services promo → Offers → Stylists → Contact → Sidebar
- **Stylist images:** `../images/stylists/{slugified_name}.png` with `onerror` fallback to `../images/stylists/stylist_default.png`
- **Slug formula:** `stylist_name.lower().replace(' ','_').replace("'",'')`

### Other Pages
| Page | `active_page` | Notes |
|---|---|---|
| `offers.html` | `offers` | Dynamic offers by store from sheet |
| `contact.html` | `contact` | Book/WA/Call/Directions buttons per store |
| `about.html` | `home` (no active state) | Brand story, franchise info, store grid, CTA |
| `booking-policy.html` | `home` | Static policy content |
| `cancellation-policy.html` | `home` | Static policy content |

---

## 7. Website — Build Pipeline

### GitHub Actions Workflow (`.github/workflows/build.yml`)
```yaml
Trigger:   push to main  OR  manual workflow_dispatch
Python:    3.11
Steps:     checkout → pip install → python build/build.py → git commit & push
Staged:    index.html, services.html, offers.html, contact.html,
           about.html, booking-policy.html, cancellation-policy.html, stores/
```

### Build Script Flow (`build.py`)
```
1. Fetch 4 sheet tabs via Google Sheets gviz CSV endpoint
2. Filter + order active stores (STORE_ORDER list)
3. Enrich phone numbers (normalise to Display/Tel/WhatsApp)
4. Compute featured_offers (3 nearest expiry)
5. Compute offers_by_store
6. Compute min_prices per homepage service category
7. Initialise Jinja2 env with global helpers
8. Render and write: index, services, 5×store, contact, offers,
   cancellation-policy, booking-policy, about
9. Minify HTML (strip comments, collapse blank lines, strip leading whitespace)
```

### HTML Minification
Light minification applied to all pages:
- Remove HTML comments (preserve IE conditionals)
- Collapse 3+ blank lines → 1
- Strip leading whitespace per line
- Collapse 2+ spaces → 1

---

## 8. Website — Design System

### CSS Variables (via `theme_vars()` macro)
```css
--dark:        #0d1f14      /* deep forest green-black */
--green:       #2d6a4f      /* brand green */
--green-mid:   #40916c
--green-soft:  #74c69d
--gold:        #c9a84c      /* brand gold */
--gold-pale:   #e8d197
--cream:       #f8f6f0      /* page background */
--text:        #1a1a1a
--text-mid:    #5a5a5a
--text-light:  #888
--nav-bg:      rgba(13,31,20,0.95)
--card-hover-border: #40916c
```

### Theme Toggle
Two themes available — **Luxury** (default dark forest) and **Naturals** (brand purple-dark).  
Floating pill toggle in bottom-left corner. Theme stored in `localStorage`.

### Typography
- **Headings:** Cormorant Garamond, serif (300/400/600 weights, italic variant)
- **Body / UI:** Jost, sans-serif (300/400/500/600 weights)

### Responsive Breakpoints
| Breakpoint | Behaviour |
|---|---|
| `> 1024px` | Full desktop — all columns visible |
| `≤ 1024px` | Tablet — desktop nav hidden, hamburger shown, 2-col grids |
| `≤ 640px` | Mobile — 1-col everything, body padding-bottom 62px for sticky bar |
| `≤ 400px` | Narrow mobile — tighter padding, smaller hero h1 |

---

## 9. Website — Key Features

### Global Header (`header.j2`)
- Fixed, 70px tall
- Transparent on homepage (becomes solid on scroll via JS)
- Solid (`data-solid="true"`) on all inner pages from load
- Desktop nav hidden at ≤1024px, hamburger shown
- **Active nav item** highlighted per page via `active_page` variable
- Stores dropdown with area sub-labels
- `header.j2` is `{% include %}`-ed by every page

### Hamburger Navigation
- Slide-in drawer from **right**, 300px wide
- Overlay dims the page
- **Closes on:** close button, overlay click, Escape key, any nav link tap, any internal link tap
- All drawer links have `onclick="closeDrawer()"`
- Document-level click listener as safety net
- Sub-menu expansion for "Our Stores" accordion

### Sticky Mobile Bar (`sticky_mobile_bar()` macro)
- Fixed at bottom of screen on mobile (hidden on desktop)
- Shows 4 actions: **Book** (blue) · **WhatsApp** (green) · **Call** (grey) · **Directions** (red)
- Reads selected store from `localStorage`
- If no store selected → opens Store Picker modal
- `body { padding-bottom: 62px }` on mobile so content isn't hidden behind bar

### Store Picker Modal (`store_picker_modal()` macro)
- Triggered by sticky bar when no store is selected
- Shows all active stores as cards (name, area, Book button)
- Selection stored in `localStorage` as `naturals-store` JSON object
- Persists across pages — once selected, all pages use that store's URLs

### `storePickerThen()` — Homepage CTA Function
On the homepage store cards, Book/WhatsApp/Call buttons use `storePickerThen()`:
1. Silently records the store in localStorage
2. Updates the store indicator in the header
3. After 60ms delay, fires the action (opens booking URL, WA link, or initiates call)

### Store Indicator
Small pill in header showing selected store name. Appears after store selection. Hidden on mobile.

### Services Filter (3-row)
| Row | Label | Colour | Behaviour |
|---|---|---|---|
| 1 | Location | Teal `#52B788` | Resets gender + rebuilds category row |
| 2 | Gender | Gold `var(--gold)` | Rebuilds category row |
| 3 | Category | Lavender `#B39DDB` | Filters only — never auto-reset |

Category buttons are always visible (wrap to new lines, never scroll).

### JSON-LD Schemas
Every store page has `LocalBusiness` schema injected in `<head>` with:
- `name`, `address`, `telephone`, `url`, `openingHours`, `priceRange: "₹₹"`, `image`

### Scroll-to-Top Button
Fixed bottom-right, appears after 320px scroll, repositioned to 80px from bottom on mobile (above sticky bar).

---

## 10. Review Automation Bot

**Repository:** `naturals-blr/naturals-review-bot` (GitHub)  
**Runtime:** Node.js  
**Config:** YAML-based (`config/stores.yml`, `config/brand.yml`)  
**Triggered by:** GitHub Actions (scheduled cron jobs)

### Features

| # | Feature | Schedule | Description |
|---|---|---|---|
| 1 | **Reply to Google Reviews** | Daily 9 AM IST | AI-generated replies to all new reviews across 5 stores using per-store brand voice |
| 2 | **Review → Social Amplification** | Daily | Drafts Instagram/Facebook caption from every 5★ review → saves to `social_queue.csv` |
| 3 | **Negative Review Alert** | Daily | WhatsApp + email to store manager within hours of 1–2★ review. Includes suggested personal follow-up message |
| 4 | **Weekly Reports** | Monday | HTML email report to each store manager + C-level (Sophia) with ratings, volume, sentiment |
| 5 | **Review Solicitation** | Daily | WhatsApp message to customers 2 hours after appointment |
| 6 | **Competitor Benchmarking** | Weekly Sunday | Monitors competitor Google ratings, alerts on significant changes |
| 7 | **Sophia Daily Briefing** | Daily | Morning HTML email summary for C-level with cross-store KPIs, charts |

### File Structure
```
config/
  stores.yml          # All 5 store data — Google location IDs, contacts, managers
  brand.yml           # Brand voice, tone, reporting settings
  loader.js           # YAML loader with caching
  validateConfig.js   # Joi schema validation
  googleAuth.js       # OAuth2 client with auto-refresh

steps/
  replyToReviews.js   # Core review reply engine
  weeklyReport.js     # Email reporting
  negativeAlerts.js   # WhatsApp + email alerts
  socialAmplification.js  # Social post drafting
  reviewSolicitation.js   # Post-appointment WhatsApp
  competitorBenchmark.js  # Competitor monitoring

utils/
  dryRun.js           # DRY_RUN wrapper — safe testing
  duplicateGuard.js   # Prevents duplicate replies
  sanitizeReply.js    # Cleans AI output before posting
  auditLog.js         # Append-only CSV log
  emailSender.js      # Gmail/nodemailer transport
  emailTemplates.js   # HTML email templates

prompts/
  brandVoice.js       # Per-store Claude prompt builder
```

### Safety Guards
- `duplicateGuard.js` — never replies to same review twice
- `sanitizeReply.js` — strips AI artefacts before posting
- `DRY_RUN` mode — logs actions without posting (safe testing)
- Joi schema validation on every startup
- Audit log for all reply attempts

### Reporting Structure
- **Per-store manager:** Weekly email with their store's stats only
- **C-level (Sophia):** Weekly cross-store summary + daily morning briefing
- **Email transport:** Gmail via Nodemailer (SendGrid upgrade path available)

---

## 11. Button & Colour Standards

### Global Button Classes (defined in `button_css()` macro in `macros.j2`)

| Button | Class | Background | Text | Use |
|---|---|---|---|---|
| Book / Appointment | `.btn-book` | `#1A73E8` (Google Blue) | `#fff` | All booking CTAs |
| WhatsApp | `.btn-wa` | `#25D366` (WhatsApp Green) | `#fff` | All WA links |
| Call | `.btn-call` | `#5F6368` (Grey) | `#fff` | All phone call links |
| Directions | `.btn-maps` | `#D93025` (Red) | `#fff` | All Google Maps links |
| Gold | `.btn-gold` | `var(--gold)` | `var(--dark)` | Brand accent CTAs |
| Ghost | `.btn-ghost` | transparent | `rgba(255,255,255,0.75)` | Dark-bg secondary CTAs |

**All standard buttons:** `min-height: 48px`, `border-radius: 6px`, `font-size: 11px`, `letter-spacing: 1.5px`, uppercase.

### Homepage Store Card Buttons (compact variant)
Smaller versions (`.st-btn`) with same colours, `min-height: 36px`, `padding: 9px 13px`.

### Filter Pill Colours (Services page)
| Row | Colour | Active Background |
|---|---|---|
| Location (teal) | `#52B788` | `rgba(27,67,50,0.9)` |
| Gender (gold) | `var(--gold)` | `rgba(201,168,76,0.15)` |
| Category (lavender) | `#B39DDB` | `rgba(179,157,219,0.15)` |

---

## 12. Image Folder Structure

```
images/
├── heroes/          # Hero/banner images per store (JPG)
│   └── {store-slug}.jpg   e.g. jpnagar.jpg, elements-mall.jpg
├── services/        # Services section banner
│   └── services-banner.jpg
├── offers/          # Offer images
│   └── {offer-slug}.jpg
├── stylists/        # Stylist profile photos (PNG)
│   ├── stylist_default.png    ← REQUIRED fallback
│   └── {first_last}.png       e.g. priya_nair.png
└── naturals-logo.png          # Brand logo (white version for dark header)
```

**Stylist image naming:** lowercase, spaces → underscores, apostrophes removed.  
Example: `"Priya Nair"` → `priya_nair.png`  
**Fallback:** `onerror` triggers `stylist_default.png` if individual file not found.

**Store page paths use `../` prefix** (e.g. `../images/stylists/priya_nair.png`) because store pages live in `stores/` subdirectory.

---

## 13. Decisions & Conventions Log

| Decision | Choice | Reason |
|---|---|---|
| Static vs dynamic site | Static (Jinja2 → HTML) | Free hosting, fast, no server needed |
| Data source | Google Sheets (public CSV) | Non-technical team can update without code |
| Booking system | External Appointment URL (Setmore-compatible) | Setmore iframe available as future upgrade |
| Phone storage | Raw digits in sheet → normalised in build | Single source of truth, all formats derived |
| Category filter type | Pill buttons (not dropdown) | Better UX, always visible, touchable |
| Category filter mobile | Always wrap (not horizontal scroll) | User requested "always visible as list" |
| Hamburger direction | Slide-in from right | Modern UX, overlay dims page |
| Store picker trigger | No-store-selected → modal | Forces deliberate store selection for personalisation |
| Social icons in footer | Removed | User requested removal |
| Directions button colour | Red `#D93025` | Matches Google Maps branding, user-specified |
| Book button colour | Blue `#1A73E8` | Google Calendar / booking blue, user-specified |
| WhatsApp button colour | Green `#25D366` | WhatsApp brand green, user-specified |
| Call button colour | Grey `#5F6368` | Neutral, user-specified |
| Stylist image fallback | `stylist_default.png` | Avoid broken image, no external CDN dependency |
| JSON-LD price range | `₹₹` | Matches value positioning |
| Theme toggle | Floating pill bottom-left | Non-intrusive, persistent via localStorage |
| HTML minification | Light (keep readable) | Performance without obfuscation |
| GitHub org name | `naturals-blr` | Clean, customer-facing, shortest |

---

## 14. Pending / To-Do

### Website
- [ ] Add real store hero images to `images/heroes/` (one per store)
- [ ] Add stylist photos to `images/stylists/` (naming: `first_last.png`)
- [ ] Fill `seo_intro` column in `store_details` sheet (150–200 words per store, for SEO)
- [ ] Add Google Tag Manager ID to `build.py` → `GTM_ID`
- [ ] Add Meta Pixel ID to `build.py` → `PIXEL_ID`
- [ ] Add Setmore iframe booking widget (currently uses direct URL, iframe slot commented out)
- [ ] Add Google Maps embed to contact page and store pages
- [ ] Add `Google_Review_URL` column to store_details sheet for the Review us quick-link
- [ ] Verify all store `Appointment_URL` values in sheet are live

### Review Bot
- [ ] Add Google Business Profile location IDs to `stores.yml` (required for API)
- [ ] Set up Google OAuth credentials and store as GitHub secret
- [ ] Configure WhatsApp Business API credentials
- [ ] Test dry-run mode end to end before enabling live posting
- [ ] Set up `GMAIL_USER` and `GMAIL_APP_PASSWORD` secrets for email reports

### Infrastructure
- [ ] Set up custom domain for GitHub Pages (e.g. `naturalsblr.in`) with CNAME record
- [ ] Enable GitHub Pages in repo settings → Source: `main` branch, root `/`

---

## Quick Reference — Key IDs

| Item | Value |
|---|---|
| Google Sheet ID | `1wy0_josh4L-C0GXWNnRG8QEO9F8km5SMrDefemChFUo` |
| GitHub Org | `naturals-blr` |
| Website URL | `https://naturals-blr.github.io` |
| Store order | N78 → N77 → N36 → N05 → N43 |
| localStorage key (store) | `naturals-store` |
| localStorage key (theme) | `naturals-theme` |

---

*This document was generated from 27 development sessions spanning Feb 23 – Mar 1, 2026.*
