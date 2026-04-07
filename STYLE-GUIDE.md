# ArcheryTracker.io — UI Style Guide

This file is authoritative for all visual and UI decisions. Claude Code must follow these rules for every component, page, and piece of markup it generates. Do not deviate without explicit instruction.

---

## Design Principles

1. **Mobile-first, fat-finger safe.** Every interactive element minimum 56px tall. Zone buttons minimum 88px tall. Generous tap targets always beat visual compactness.
2. **Config-driven, never hardcoded.** Zone labels, colours, and values come from discipline config. Components render what they're given.
3. **Legibility over decoration.** This app is used outdoors in variable light by people of all ages and abilities. High contrast, large text, zero visual noise.
4. **Obvious over clever.** If a user has to think about what a button does, it needs a label. Icons only appear alongside text, never alone.
5. **Earthy and purposeful.** The palette reflects the outdoor environment. Green and brown, not tech-blue. Sage surfaces, teal-brown actions.

---

## Typography

### Font

**Atkinson Hyperlegible** — designed specifically for low-vision readers and dyslexia accessibility. Load from Google Fonts:

```html
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link href="https://fonts.googleapis.com/css2?family=Atkinson+Hyperlegible:ital,wght@0,400;0,700;1,400&display=swap" rel="stylesheet" />
```

Tailwind config:

```js
fontFamily: {
  sans: ['Atkinson Hyperlegible', '-apple-system', 'BlinkMacSystemFont', 'Segoe UI', 'sans-serif'],
}
```

### Scale

| Token | Size | Weight | Usage |
|-------|------|--------|-------|
| `text-display` | 42px / 700 | Bold | Score totals, big numbers only |
| `text-h1` | 28px / 700 | Bold | Page titles (e.g. "Target 7 of 20") |
| `text-h2` | 20px / 600 | Semibold | Section headings, discipline name |
| `text-h3` | 17px / 600 | Semibold | Card headings, sub-sections |
| `text-body` | 16px / 400 | Regular | All body copy, instructions |
| `text-muted` | 14px / 400 | Regular | Secondary info, timestamps |
| `text-label` | 11px / 700 | Bold | All-caps labels, zone names, field labels |
| `text-zone` | 34px / 700 | Bold | Score value inside zone buttons |

**Rules:**
- Never go below 14px for any text a user needs to read
- Sentence case everywhere — never ALL CAPS in body copy (labels excepted)
- Line height: body = 1.65, headings = 1.15–1.3, labels = 1.2
- Letter spacing: labels = 0.07em, display numbers = -0.02em

---

## Colour Palette

### CSS Custom Properties

Define these in `:root` and use them everywhere. Never use raw hex values in component code.

```css
:root {
  /* Primary — Teal-Brown */
  --color-primary-dark:    #1A2E2E;
  --color-primary:         #2A4040;
  --color-primary-mid:     #3D6B5A;

  /* Sage — surfaces and accents */
  --color-sage:            #7AAE98;
  --color-sage-light:      #E8F0EC;
  --color-sage-xlight:     #F2F7F4;

  /* Tan — wound zone, secondary surfaces */
  --color-tan:             #C4A882;
  --color-tan-light:       #EDE0CE;
  --color-tan-xlight:      #F7F2EA;

  /* Neutrals */
  --color-cream:           #F7F9F8;
  --color-white:           #FFFFFF;
  --color-text:            #1A2424;
  --color-text-secondary:  #3D5A52;
  --color-text-muted:      #6A8A80;

  /* Semantic */
  --color-danger:          #8B2020;
  --color-danger-light:    #FAEAEA;
  --color-warning:         #7A4A00;
  --color-warning-light:   #FFF3E0;
  --color-success:         #2D4A1E;
  --color-success-light:   #E8F5DF;

  /* Shadows */
  --shadow-sm: 0 1px 4px rgba(42,64,64,0.10), 0 0 0 0.5px rgba(42,64,64,0.07);
  --shadow-md: 0 2px 8px rgba(42,64,64,0.13), 0 0 0 0.5px rgba(42,64,64,0.09);
  --shadow-lg: 0 4px 16px rgba(42,64,64,0.16), 0 0 0 0.5px rgba(42,64,64,0.10);

  /* Radii */
  --radius:    10px;
  --radius-lg: 14px;
  --radius-xl: 18px;
}
```

### Tailwind Colour Extension

```js
colors: {
  primary: {
    dark:    '#1A2E2E',
    DEFAULT: '#2A4040',
    mid:     '#3D6B5A',
  },
  sage: {
    DEFAULT: '#7AAE98',
    light:   '#E8F0EC',
    xlight:  '#F2F7F4',
  },
  tan: {
    DEFAULT: '#C4A882',
    light:   '#EDE0CE',
    xlight:  '#F7F2EA',
  },
  cream:   '#F7F9F8',
  danger:  '#8B2020',
  warning: '#7A4A00',
}
```

### Colour Usage Rules

| Context | Colour |
|---------|--------|
| Primary action buttons | `--color-primary` background, cream text |
| App header / nav | `--color-primary-dark` background |
| Score total bar | `--color-primary` background |
| Kill zone button | `--color-primary` background |
| Vital zone button | `--color-primary-mid` background |
| Wound zone button | `--color-tan` background, `--color-primary-dark` text |
| Miss button | `--color-cream` background, muted text, tan border |
| Page background | `--color-cream` |
| Card background | `--color-white` |
| Secondary surfaces | `--color-sage-light` or `--color-tan-xlight` |
| Org picker / tabs | `--color-sage-light` container, `--color-primary` active tab |
| Grading badge | `--color-primary-dark` background, sage-xlight text |
| Danger/delete | `--color-danger` background |
| Indoor timer — normal | `--color-sage-light` background, primary text |
| Indoor timer — warning (≤30s) | `--color-warning-light` background, warning text |
| Indoor timer — alert (0s) | `--color-danger-light` background, danger text |
| Stop-on-hit banner | `--color-sage-light` background, sage border |

---

## Spacing System

Use Tailwind's default spacing scale. Key values:

| Usage | Value |
|-------|-------|
| Page horizontal padding | `px-4` (16px) |
| Card internal padding | `p-4` (16px) |
| Gap between zone buttons | `gap-2.5` (10px) |
| Gap between cards | `mb-3` (12px) |
| Section vertical gap | `py-5` (20px) |
| Form field gap | `mb-3` (12px) |
| Button row gap | `gap-2.5` (10px) |

---

## Component Specifications

### App Header

```tsx
// Dark primary-dark bar across top
// Left: app name (bold 18px) + subtitle (discipline + org, 11px muted)
// Right: grading badge if applicable, or club tag

<header className="bg-primary-dark text-cream px-4 py-3.5 flex justify-between items-center">
  <div>
    <h1 className="text-lg font-bold">ArcheryTracker</h1>
    <p className="text-xs opacity-60 mt-0.5">{discipline} · {org}</p>
  </div>
  <div className="text-right">
    {isGrading && <GradingBadge />}
  </div>
</header>
```

### Score Total Bar

```tsx
// Sits directly below app header, primary background
// Left: "Total" label + large score number
// Right: "Target X of Y" + "This target: N"

<div className="bg-primary text-sage-xlight px-4 py-3.5 flex justify-between items-center">
  <div>
    <p className="text-[11px] font-bold uppercase tracking-[0.07em] opacity-70">Total</p>
    <p className="text-[36px] font-bold leading-none">{total}</p>
  </div>
  <div className="text-right">
    <p className="text-xs opacity-65">Target {current} of {count}</p>
    <p className="text-[19px] font-semibold">This target: {targetTotal}</p>
  </div>
</div>
```

### Zone Buttons

The core MVP component. Config-driven — accepts zone definitions, current arrow number, values.

```tsx
// Grid of 3 columns, min-height 88px each
// Kill / A zone:   primary dark green
// Vital / B zone:  primary mid green
// Wound / C zone:  tan / brown
// Miss:            full-width below, cream background, muted

<div className="grid grid-cols-3 gap-2.5">
  {zones.map(zone => (
    <button
      key={zone.label}
      onClick={() => onScore(zone)}
      className="rounded-[14px] py-3 flex flex-col items-center justify-center min-h-[88px] shadow-md"
      style={{ background: zoneColour(zone.label), color: zoneTextColour(zone.label) }}
    >
      <span className="text-[11px] font-bold uppercase tracking-[0.06em] opacity-80">
        {zone.display}
      </span>
      <span className="text-[34px] font-bold leading-none my-0.5">
        {zone.value}
      </span>
      <span className="text-[10px] opacity-65">Zone {zone.label}</span>
    </button>
  ))}
</div>
<button className="w-full min-h-[56px] mt-2.5 rounded-[10px] bg-cream border-[1.5px] border-tan-light text-text-muted text-base font-medium">
  Miss — 0 pts
</button>
```

**Zone colour mapping function:**
```ts
function zoneColour(label: string): string {
  // First zone in config = primary (Kill/5/Bull)
  // Second zone = primary-mid (Vital/4/Inner)
  // Third zone = tan (Wound/3/Outer)
  // Resolved by index from discipline config, not by label string
}
```

**Always resolve zone colours by index position in the config array, never by matching on label strings like "Kill" or "A".**

### Diminishing Arrow Hint

```tsx
// Shown above zone buttons when discipline.special_rules.diminishing_score = true
<div className="flex items-center gap-2 mb-2">
  <span className="text-[11px] font-bold uppercase tracking-[0.07em] text-text-secondary">
    Arrow {arrowNumber}
  </span>
  <span className="bg-primary-dark text-sage text-[11px] font-semibold rounded-full px-2 py-0.5">
    max {maxValue} pts
  </span>
</div>
```

### Stop-on-Hit Banner

```tsx
// Shown after a hit is recorded on IFAA Animal round
// Replaces remaining arrow buttons — non-blocking, no modal
<div className="bg-sage-light border-[1.5px] border-sage rounded-[10px] px-4 py-3 flex items-center gap-2.5">
  <div className="w-2.5 h-2.5 rounded-full bg-primary-mid flex-shrink-0" />
  <p className="text-sm font-semibold text-primary">
    Hit recorded — move to next target
  </p>
</div>
```

### Indoor Timer

```tsx
// Sits above target number in the score bar area
// Three visual states driven by time remaining

const timerClass = timeRemaining > 30
  ? 'bg-sage-light text-primary'           // normal
  : timeRemaining > 0
    ? 'bg-warning-light text-warning'      // amber warning
    : 'bg-danger-light text-danger'        // red alert

<div className={`${timerClass} rounded-[10px] px-4 py-2 text-[22px] font-bold text-center tabular-nums`}>
  {formatTime(timeRemaining)}
</div>
```

### Post-Target Distance Reveal

```tsx
// Only shown when discipline.special_rules.distance_visibility = 'after_target'
// Brief interstitial after "Next →" is tapped — auto-dismisses in 2s or on tap
<div className="bg-sage-light rounded-[14px] p-5 text-center shadow-md">
  <p className="text-[11px] font-bold uppercase tracking-[0.07em] text-text-muted mb-1">
    Target {targetNumber} distance
  </p>
  <p className="text-[32px] font-bold text-primary leading-none">
    {distanceDisplay}  {/* e.g. "36m / 39.4yds" */}
  </p>
  <p className="text-sm text-text-muted mt-2">Tap to continue</p>
</div>
```

### Arrow Result Pills

```tsx
// Used in previous-target summary card
// Colour matches zone button colours — same index-based mapping
<div className="flex gap-1.5 flex-wrap">
  {arrows.map((arrow, i) => (
    <span
      key={i}
      className="rounded-full px-3 py-1 text-[13px] font-bold"
      style={{ background: zoneColour(arrow.label), color: zoneTextColour(arrow.label) }}
    >
      {arrow.label} · {arrow.value}
    </span>
  ))}
</div>
```

### Card

```tsx
<div className="bg-white rounded-[14px] shadow-md p-4">
  {children}
</div>
```

### Org Picker / Tabs

```tsx
<div className="flex bg-sage-light rounded-[10px] p-1 shadow-sm">
  {['ABA', 'IFAA'].map(org => (
    <button
      key={org}
      onClick={() => setOrg(org)}
      className={`flex-1 py-3 rounded-[7px] text-base font-semibold transition-all ${
        selected === org
          ? 'bg-primary text-cream shadow-sm'
          : 'text-text-muted'
      }`}
    >
      {org}
    </button>
  ))}
</div>
```

### Primary Button

```tsx
<button className="bg-primary text-cream rounded-[10px] px-7 py-4 text-[17px] font-semibold min-h-[56px] shadow-md w-full">
  {label}
</button>
```

### Secondary Button

```tsx
<button className="bg-sage-light text-primary border-[1.5px] border-sage rounded-[10px] px-6 py-4 text-[17px] font-semibold min-h-[56px] shadow-sm w-full">
  {label}
</button>
```

### Ghost Button

```tsx
<button className="bg-transparent text-text-secondary border-[1.5px] border-tan-light rounded-[10px] px-5 py-3.5 text-base font-medium min-h-[56px]">
  {label}
</button>
```

### Form Input

```tsx
<div>
  <label className="block text-[13px] font-semibold text-text-secondary mb-1.5">
    {label}
  </label>
  <input
    className="w-full border-[1.5px] border-sage rounded-[10px] px-4 py-3.5 text-base text-text bg-white font-sans outline-none focus:border-primary-mid"
    placeholder={placeholder}
  />
</div>
```

### Badges & Pills

```tsx
// Grading — dark primary
<span className="bg-primary-dark text-sage-xlight rounded-full px-3.5 py-1 text-[12px] font-bold uppercase tracking-[0.05em]">
  Official Grading
</span>

// Informational — sage
<span className="bg-sage-light text-primary rounded-full px-2.5 py-1 text-[12px] font-semibold">
  Grading eligible
</span>

// Neutral — tan
<span className="bg-tan-xlight text-[#4A3010] border border-tan-light rounded-full px-2.5 py-1 text-[12px] font-semibold">
  Social round
</span>

// Required / error
<span className="bg-danger-light text-danger rounded-full px-2.5 py-1 text-[12px] font-semibold">
  Required
</span>
```

### Grading Style Blocker

```tsx
// Shown in-line when grading round has shooters without style/division set
// Not a modal — renders inline where the "Start scoring" button would be
<div className="bg-tan-xlight border-[1.5px] border-tan-light rounded-[14px] p-4">
  <p className="text-[15px] font-semibold text-text mb-1">Set style before scoring</p>
  <p className="text-sm text-text-muted">
    All shooters must confirm their shooting style and age division for a grading round.
  </p>
  <button className="mt-3 bg-primary text-cream rounded-[10px] px-5 py-3 text-[15px] font-semibold min-h-[48px] w-full shadow-md">
    Set my style
  </button>
</div>
```

---

## Page Backgrounds & Layout

| Page | Background | Notes |
|------|-----------|-------|
| All pages | `bg-cream` (#F7F9F8) | Never pure white for page bg |
| Cards / content surfaces | `bg-white` | Use shadow-md |
| Secondary surfaces | `bg-sage-light` or `bg-tan-xlight` | Tabs, input containers |
| App header | `bg-primary-dark` | Full bleed |
| Score bar | `bg-primary` | Full bleed, below header |

**Page structure:**
```
<AppHeader />          ← full bleed, primary-dark
<ScoreBar />           ← full bleed, primary (scoring pages only)
<main className="px-4 py-4 space-y-3">
  {/* cards and content */}
</main>
<BottomNav />          ← full bleed, white, shadow above
```

---

## Distance Display Format

Always show both units. Leading unit depends on the discipline's `distance_unit` field.

```ts
// ABA (metric first)
formatDistance(36, 'metric')   // → "36m / 39.4yds"

// IFAA (imperial first)
formatDistance(50, 'imperial') // → "50 yds / 45.7m"

// Conversion: 1 yard = 0.9144 metres
// Round metres to 1 decimal, round yards to nearest integer
```

---

## Iconography

- Icons used alongside text labels only — never standalone
- Use **Lucide React** (`lucide-react`) — consistent, accessible, clean
- Icon size: `16px` inline with text, `20px` for standalone button icons
- Icon colour always inherits from parent text colour — never set separately
- Recommended icons: `QrCode`, `UserPlus`, `ChevronRight`, `ChevronLeft`, `CheckCircle`, `AlertCircle`, `Clock`, `History`, `Printer`

---

## Accessibility

- Minimum touch target: 56px × 56px (zone buttons: 88px × min full column width)
- Colour contrast: all text must meet WCAG AA (4.5:1 for body, 3:1 for large text)
- All interactive elements must have visible focus styles: `focus:ring-2 focus:ring-primary focus:ring-offset-2`
- Never rely on colour alone to convey meaning — always pair with a label or icon
- All images and QR codes must have descriptive `alt` text
- Form inputs must have associated `<label>` elements — no placeholder-only labelling

---

## Responsive Breakpoints

This app is mobile-first. All layout decisions start at the smallest screen.

| Breakpoint | Width | Notes |
|------------|-------|-------|
| Base (mobile) | 0–639px | Primary design target — 375px iPhone width |
| `sm` | 640px+ | Wider phones / small tablets — minor adjustments only |
| `md` | 768px+ | Tablet — two-column layout allowed on summary/history pages |
| `lg` | 1024px+ | Desktop — centre-column layout, max-width 480px for scoring pages |

**Scoring page max-width:** Never wider than `max-w-sm` (384px) even on desktop. Zone buttons are designed for thumbs — stretching them across a large screen looks wrong.

---

## Animation & Transitions

Keep animations minimal and purposeful. This app is used outdoors on older devices — do not use animations that require GPU compositing.

| Element | Animation | Duration |
|---------|-----------|---------|
| Tab switching | `transition-colors` | 150ms |
| Button press | `active:scale-[0.97]` | Instant |
| Stop-on-hit banner | Fade in `opacity-0 → opacity-100` | 200ms |
| Post-target reveal | Slide up `translate-y-2 → translate-y-0` | 250ms |
| Timer warning state | `transition-colors` | 300ms |
| Page transitions | None — instant navigation | — |

No bounce, spring, or decorative animations. No `transform: scale` on zone buttons (too laggy on low-end devices).

---

## What Not To Do

- ❌ Never use `#2196F3`, `#4CAF50`, or any standard "tech blue/green" — the palette is earthy
- ❌ Never use raw hex values in component code — always use CSS variables or Tailwind tokens
- ❌ Never make a button smaller than 56px tall
- ❌ Never show distances during scoring (unless `distance_visibility = 'always'`)
- ❌ Never use colour alone to distinguish zone buttons — always include the label and value
- ❌ Never use modals or blocking overlays for rules reminders (stop-on-hit, grading gate are inline)
- ❌ Never use `font-weight: 600` or `700` in Tailwind `font-semibold` / `font-bold` for body copy — reserve bold for headings and numbers only
- ❌ Never add decorative gradients, patterns, or background images
- ❌ Never use ALL CAPS for anything other than `text-label` sized elements
