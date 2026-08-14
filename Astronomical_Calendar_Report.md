# 🔭 Astronomical Calendar — Technical Report & Documentation
## Baselius College, Kottayam (Autonomous)
### Postgraduate Department of Physics & Centre for Research

---

## 📋 Table of Contents
1. [Project Overview](#project-overview)
2. [Technology Stack](#technology-stack)
3. [Architecture & Code Structure](#architecture--code-structure)
4. [Detailed Code Explanation](#detailed-code-explanation)
5. [Features Implemented](#features-implemented)
6. [Event Database Statistics](#event-database-statistics)
7. [Security Implementation](#security-implementation)
8. [Deployment & PWA](#deployment--pwa)
9. [File Structure](#file-structure)

---

## 1. Project Overview

The **Astronomical Calendar** is a web-based application developed for the Postgraduate Department of Physics at Baselius College, Kottayam. It serves as a comprehensive calendar showcasing **467 scientific events** across the year 2026, including scientist birthdays, discoveries, astronomical events, and memorial days.

### Key Highlights
- **467 events** covering every day of the year
- **21 Indian & Kerala scientists** featured prominently
- **Single-file architecture** — entire app in one `index.html`
- **Progressive Web App (PWA)** — installable on phones
- **Admin panel** with multi-layer security
- **Multi-source references** — Wikipedia, Google Scholar, Britannica, Google
- **Responsive design** — works on desktop, tablet, and mobile

### Initiative By
- 💡 **IEDC** (Innovation & Entrepreneurship Development Cell)
- 🎯 **IIC** (Institution's Innovation Council)
- ⚛️ **Department of Physics**

---

## 2. Technology Stack

| Technology | Purpose |
|-----------|---------|
| **HTML5** | Page structure, semantic markup |
| **CSS3** | Styling, animations, responsive design |
| **Vanilla JavaScript** | All logic, rendering, state management |
| **Google Fonts (Inter)** | Typography |
| **Material Symbols** | Icons for UI elements |
| **Service Worker** | PWA caching, offline support |
| **GitHub Pages** | Hosting & deployment |

> **No frameworks used.** The entire application is built with pure HTML, CSS, and JavaScript — no React, Vue, Angular, or any build tools required.

---

## 3. Architecture & Code Structure

The application follows a **single-file architecture** where all CSS, HTML, and JavaScript are contained in one `index.html` file (~2500 lines, ~161 KB).

### Code Organization (Inside `index.html`)

```
┌─────────────────────────────────────────────┐
│  <head>                                      │
│  ├── Meta tags (SEO, PWA, Security)          │
│  ├── CSP (Content Security Policy)           │
│  ├── Google Fonts links                      │
│  └── <style> (ALL CSS - ~900 lines)          │
│       ├── Layout (app, sidebar, main)        │
│       ├── Calendar grid                      │
│       ├── Event types & colors               │
│       ├── Modal styles                       │
│       ├── Admin panel styles                 │
│       ├── Physics dept profile               │
│       ├── Responsive breakpoints             │
│       └── Custom scrollbar                   │
├─────────────────────────────────────────────┤
│  <body>                                      │
│  ├── Sidebar (logo, dept profile, events)    │
│  ├── Main area (topbar, calendar grid)       │
│  ├── Modal overlay (event details)           │
│  ├── Admin panel overlay                     │
│  └── Signature footers                       │
├─────────────────────────────────────────────┤
│  <script> (ALL JavaScript - ~600 lines)      │
│  ├── Constants (months, event types)         │
│  ├── EVENT DATA ARRAY (467 events)           │
│  ├── State variables                         │
│  ├── render() function                       │
│  ├── Event handlers                          │
│  ├── Search functionality                    │
│  ├── Modal functions                         │
│  ├── Security (SHA-256, rate limit)          │
│  ├── Admin CRUD operations                   │
│  └── Init sequence                           │
└─────────────────────────────────────────────┘
```

---

## 4. Detailed Code Explanation

### 4.1 CSS Design System

The CSS uses a carefully crafted design system:

```css
/* Color palette */
Primary:     #4f46e5 (Indigo)
Background:  #f8f9fb (Light grey)
Text:        #1a1a2e (Dark navy)
Secondary:   #6b7280 (Grey)
Success:     #059669 (Green)
Warning:     #d97706 (Amber)
Danger:      #ea580c (Orange)
```

**Layout:** Flexbox-based two-column layout
```css
.app { display:flex; height:100vh; }
.sidebar { width:320px; overflow-y:scroll; }
.main { flex:1; overflow:auto; }
```

**Calendar Grid:** CSS Grid with 7 columns (Sun-Sat)
```css
.cal-grid { display:grid; grid-template-columns:repeat(7,1fr); }
.cal-cell { min-height:85px; cursor:pointer; }
```

**Responsive Design:** Three breakpoints
- `@media (max-width:1024px)` — Tablet (sidebar becomes slide-out)
- `@media (max-width:768px)` — Mobile phones
- `@media (max-width:380px)` — Small phones

### 4.2 Event Data Structure

Each event is a JavaScript object with 7 properties:

```javascript
{
    m: 8,                    // Month (1-12)
    d: 11,                   // Day (1-31)
    t: 'birthday',           // Type (astro|moon|eclipse|intl|birthday|memorial|achievement)
    i: '🔬',                // Emoji icon
    n: 'E.C.G. Sudarshan',  // Event name
    x: 'Description...',     // Description text
    f: 'Quantum Optics'      // Scientific field
}
```

**Event Types & Color Coding:**
| Type | Color | Used For |
|------|-------|----------|
| `astro` | Blue (#0284c7) | Astronomical events |
| `moon` | Purple (#7c3aed) | Moon phases |
| `eclipse` | Orange (#ea580c) | Solar/lunar eclipses |
| `birthday` | Indigo (#4f46e5) | Scientist birthdays |
| `memorial` | Rose (#e11d48) | Death anniversaries |
| `achievement` | Green (#059669) | Discoveries, awards |
| `intl` | Sky blue (#0284c7) | International days |

### 4.3 The `render()` Function — Heart of the Calendar

The `render()` function is called every time the state changes. It:

1. **Calculates calendar layout** — first day of month, total days
2. **Generates calendar grid HTML** — creates cells for each day
3. **Populates events** — filters events for current month, injects into cells
4. **Updates sidebar** — shows event list for selected day or month
5. **Highlights today** — special styling for current date

```javascript
function render() {
    // 1. Calculate first day and total days
    const fd = new Date(2026, cM, 1).getDay();  // First day of month
    const td = new Date(2026, cM+1, 0).getDate(); // Total days

    // 2. Build calendar grid
    let html = '';
    for(let d=1; d<=td; d++) {
        const evs = E.filter(e => e.m === cM+1 && e.d === d);
        // Create cell with day number and event bars
        html += `<div class="cal-cell" onclick="selDay(${d})">
            <div class="cell-day">${d}</div>
            <div class="cell-events">${eventHTML}</div>
        </div>`;
    }

    // 3. Update sidebar event list
    // Shows all events for selected day or entire month
}
```

### 4.4 Search System

The search feature filters events in real-time:

```javascript
document.getElementById('searchInput').addEventListener('input', function() {
    const q = this.value.toLowerCase();
    // Filter events matching name, description, or field
    const results = E.filter(e =>
        e.n.toLowerCase().includes(q) ||
        e.x.toLowerCase().includes(q) ||
        e.f.toLowerCase().includes(q)
    );
    // Re-render event list with results
});
```

### 4.5 Modal System (Event Details + Wikipedia)

When an event is clicked, a modal popup shows:

```javascript
function openM(idx) {
    const e = E[idx];
    // Extract clean search term from event name
    let searchTerm = e.n.split('—')[0].split('(')[0].trim();

    // Generate 4 source links
    let body = `<p>${e.x}</p>`;
    body += `<div class="read-more-section">`;
    body += `<a href="wikipedia..." class="wiki-link wiki">🌐 Wikipedia</a>`;
    body += `<a href="scholar..." class="wiki-link scholar">📄 Google Scholar</a>`;
    body += `<a href="britannica..." class="wiki-link britannica">📗 Britannica</a>`;
    body += `<a href="google..." class="wiki-link google">🔍 Google</a>`;
    body += `</div>`;
}
```

### 4.6 Admin Panel

The admin panel allows adding/deleting custom events:

```javascript
// Authentication flow
async function adminAuth() {
    // 1. Check if already authenticated
    // 2. Check lockout status
    // 3. Prompt for password
    // 4. Hash with SHA-256
    // 5. Compare with stored hash
    // 6. Start session timer (30 min)
}

// Save custom event
function saveCustomEvent() {
    // 1. Read form inputs
    // 2. Sanitize all inputs (XSS prevention)
    // 3. Validate lengths
    // 4. Save to localStorage
    // 5. Sign data (tamper detection)
    // 6. Log action (audit trail)
}
```

---

## 5. Features Implemented

### Core Features
- ✅ Monthly calendar grid with event bars
- ✅ Day selection with event list in sidebar
- ✅ Event detail modal with descriptions
- ✅ Month navigation (prev/next)
- ✅ "Today" button to jump to current date
- ✅ Real-time event search
- ✅ Event count statistics pills
- ✅ Custom scrollbar on sidebar

### Content Features
- ✅ 467 curated scientific events
- ✅ Indian & Kerala scientists featured
- ✅ Multi-source reference links (Wikipedia, Scholar, Britannica, Google)
- ✅ Physics Department profile with accordion sections
- ✅ College logo and branding
- ✅ IEDC, IIC, Physics signature footer

### Technical Features
- ✅ Progressive Web App (installable)
- ✅ Offline support via Service Worker
- ✅ Network-first caching strategy
- ✅ Responsive design (desktop, tablet, mobile)
- ✅ Admin panel for custom events
- ✅ SHA-256 password hashing
- ✅ Rate limiting & account lockout
- ✅ Session timeout (30 min)
- ✅ XSS protection (input sanitization)
- ✅ Data integrity signing
- ✅ Admin audit logging
- ✅ Content Security Policy (CSP)
- ✅ HTTPS enforcement
- ✅ Delete confirmation dialogs

---

## 6. Event Database Statistics

### Total Events: 467

| Event Type | Count | Percentage |
|-----------|-------|-----------|
| 🎂 Birthdays | 244 | 52.2% |
| 🏆 Achievements | 93 | 19.9% |
| 🕯️ Memorials | 50 | 10.7% |
| 🔭 Astronomical | 36 | 7.7% |
| 🌙 Moon Phases | 24 | 5.1% |
| 🌍 International Days | 14 | 3.0% |
| 🌑 Eclipses | 6 | 1.3% |

### Coverage by Month
Every single day of the year (366 days including Feb 29) has at least one event.

### Indian Scientists Featured: 25+
Including C.V. Raman, Ramanujan, Bose, Kalam, Bhabha, Sarabhai, Chandrasekhar, and more.

### Kerala Scientists Featured: 7
E.C.G. Sudarshan (Kottayam), Anna Mani (Peermade), Tessy Thomas (Alappuzha), K. Radhakrishnan (Irinjalakuda), G. Madhavan Nair (Thiruvananthapuram), V.P.N. Nampoori (CUSAT), Nambi Narayanan.

---

## 7. Security Implementation

### Multi-Layer Security Architecture

```
┌─────────────────────────────────────────────┐
│ Layer 1: Content Security Policy (CSP)       │
│  - Blocks unauthorized scripts & resources   │
├─────────────────────────────────────────────┤
│ Layer 2: HTTPS Enforcement                   │
│  - Auto-redirects HTTP → HTTPS               │
├─────────────────────────────────────────────┤
│ Layer 3: SHA-256 Password Hashing            │
│  - No plaintext password in source code      │
├─────────────────────────────────────────────┤
│ Layer 4: Rate Limiting                       │
│  - 5 attempts → 5-minute lockout             │
├─────────────────────────────────────────────┤
│ Layer 5: Session Management                  │
│  - 30-minute auto-timeout                    │
│  - sessionStorage (cleared on tab close)     │
├─────────────────────────────────────────────┤
│ Layer 6: Input Sanitization                  │
│  - HTML entity escaping (XSS prevention)     │
│  - Input length limits                       │
├─────────────────────────────────────────────┤
│ Layer 7: Data Integrity                      │
│  - Hash signature on stored data             │
│  - Tamper detection on page load             │
├─────────────────────────────────────────────┤
│ Layer 8: Audit Logging                       │
│  - All admin actions logged with timestamp   │
│  - Browser info recorded                     │
└─────────────────────────────────────────────┘
```

---

## 8. Deployment & PWA

### GitHub Pages Deployment
- **Repository:** `ameenks318-mdk/astro-calendar`
- **Live URL:** https://ameenks318-mdk.github.io/astro-calendar/
- **Branch:** `gh-pages`

### PWA Configuration

**manifest.json:**
```json
{
    "name": "Astronomical Calendar",
    "short_name": "AstroCalendar",
    "start_url": "./",
    "display": "standalone",
    "theme_color": "#4f46e5",
    "background_color": "#f8f9fb"
}
```

**Service Worker (sw.js):**
- Strategy: **Network-first** (always serves fresh content when online)
- Cache name: `astro-calendar-v4`
- Fallback: Serves cached version when offline
- Auto-cleans old cache versions on activate

---

## 9. File Structure

```
astro-calendar/
├── index.html          (161 KB - Main application)
├── logo.webp           (College logo banner)
├── manifest.json       (PWA configuration)
├── sw.js              (Service Worker)
├── events_data.csv     (Exported event data)
└── Astronomical_Calendar_Events_Complete.csv
```

---

## 📌 How to Use

### For Visitors
1. Visit https://ameenks318-mdk.github.io/astro-calendar/
2. Browse months using ◀ ▶ arrows
3. Click any day to see events in the sidebar
4. Click any event to see full details + reference links
5. Use the search bar to find specific events

### For Admin
1. Click the 🔒 lock icon in the topbar
2. Enter password: `baselius2024`
3. Add or delete custom events
4. Session auto-expires after 30 minutes

### To Install as App
1. Open the site in Chrome
2. Click the "Install" prompt or menu → "Add to Home Screen"
3. The calendar now works as a standalone app, even offline!

---

*Report generated on August 14, 2026*
*Postgraduate Department of Physics & Centre for Research*
*Baselius College, Kottayam (Autonomous)*
