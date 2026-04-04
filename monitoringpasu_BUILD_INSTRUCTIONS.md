# MONITORINGPASU.CZ — Build Instructions for Developer
## Complete Technical Specification for Coding & Design

---

## 1. BRAND SYSTEM (from REMA TIP TOP CD Manual 2017)

### Colors (exact values from brand manual)

```css
:root {
  /* Logo colors — primary palette */
  --rtt-black: #1a1a18;        /* REMA TIP TOP Black — PANTONE Black, RAL 9017 */
  --rtt-red: #ef232a;          /* REMA TIP TOP Red — PANTONE 485 C, RAL 3020 */

  /* Background colors */
  --rtt-dark-gray: #494948;    /* REMA TIP TOP Dark Gray — PANTONE 446 C, RAL 7012 */
  --rtt-medium-gray: #919292;  /* REMA TIP TOP Medium Gray — PANTONE 423 C, RAL 7045 */

  /* Accent colors */
  --rtt-green: #00A983;        /* REMA TIP TOP Green — PANTONE Green C */
  --rtt-blue: #10BAE7;         /* REMA TIP TOP Blue — PANTONE 298 C */
  --rtt-yellow: #FFCC00;       /* REMA TIP TOP Yellow — PANTONE 109 C */

  /* Functional */
  --white: #ffffff;
  --off-white: #f5f5f5;
  --text-body: #1a1a18;
  --text-muted: #494948;
  --border-light: #e0e0e0;
}
```

### Typography

**Corporate font:** Pill Gothic 600 MG OTC (licensed — not available for web use without purchase).
**Web substitute for headlines:** Barlow Condensed (Google Fonts) — condensed industrial feel that closely matches the brand's typographic energy.
**Web substitute for body:** Barlow (Google Fonts) — clean, readable, Czech diacritics fully supported.
**Fallback (per brand manual for internal use):** Arial.

```css
/* Font loading */
@import url('https://fonts.googleapis.com/css2?family=Barlow+Condensed:wght@600;700;800&family=Barlow:wght@400;500;600&display=swap');

/* Typography scale */
--font-display: 'Barlow Condensed', Arial, sans-serif;
--font-body: 'Barlow', Arial, sans-serif;

--text-hero: 3.5rem;      /* 56px — H1 hero headline */
--text-h2: 2rem;           /* 32px — section headings */
--text-h3: 1.5rem;         /* 24px — sub-headings */
--text-body: 1.0625rem;    /* 17px — body text */
--text-small: 0.875rem;    /* 14px — captions, helper text */
--text-micro: 0.75rem;     /* 12px — legal, footer */
```

### Logo files

| Variant | File | Use |
|---------|------|-----|
| Full color (4c) | `LOGO_REMA_TIP_TOP_color.png` | White backgrounds, light sections |
| White on dark | Top half of `LOGO_REMA_TIP_TOP_white_and_red.png` | Hero section (dark background), footer |
| On red background | Bottom half of `LOGO_REMA_TIP_TOP_white_and_red.png` | CTA buttons area if needed |

**Logo rules from brand manual:**
- Minimum protection zone: 1/3 of logo height as white space on all sides
- Minimum logo height: 8mm print / 40px web
- Permitted backgrounds: white, black, or REMA TIP TOP red ONLY within protection zone
- Logo must NEVER be placed on colored backgrounds, gradients, or busy photos within its protection zone

### Hero image
- File: `news_mcube_cam.webp` (uploaded — smartphone on tripod filming belt through inspection window)
- Use as: Large visual element in hero section, NOT as full-bleed background behind text

---

## 2. TECH STACK

| Component | Technology | Notes |
|-----------|-----------|-------|
| Framework | Next.js 14+ (App Router) OR plain HTML+Tailwind | Ship whatever is faster; single page, no routing needed |
| Styling | Tailwind CSS v3+ | Custom config with brand colors above |
| Hosting | Vercel (free tier) | Deploy on push |
| Domain | monitoringpasu.cz | DNS: A record → Vercel |
| Form backend | Google Sheets (via Google Apps Script webhook) | See section 5 |
| Email notifications | Google Apps Script (triggered on form submission) | Sends to sales team + auto-reply to customer |
| File upload (video) | Cloudflare R2 (free tier: 10GB storage, 10M requests/mo) OR Uploadthing | Presigned URL upload, max 5GB |
| Analytics | Plausible.io OR Umami (self-hosted) | GDPR-compliant, no cookie banner needed |
| Sample report | Static images embedded on page | User uploads anonymized report PDF — render as image carousel |

---

## 3. PAGE ARCHITECTURE

Single-page app. Smooth-scroll between sections. No separate routes.

### HTML structure:
```
<body>
  <header>        — Sticky nav: logo + "Chci bezplatnou diagnostiku" CTA + "Stáhnout ceník (PDF)" link
  <section#hero>  — Section 1: Hero
  <section#problem> — Section 2: Problem vs Solution
  <section#process>  — Section 3: How It Works (4 steps + filming guide)
  <section#report>   — Section 4: Sample Report viewer
  <section#form>     — Section 5: Lead Capture (two-path form)
  <section#pricing>  — Section 6: Pricing table + commercial terms
  <section#benefits> — Section 7: Value proposition (6 cards)
  <section#trust>    — Section 8: Trust signals
  <footer>           — Section 9: Footer
</body>
```

### Mobile behavior:
- Sticky bottom CTA bar appears after scrolling past hero: `position: fixed; bottom: 0;` with "Chci bezplatnou diagnostiku" button
- Disappears when user scrolls to the form section (IntersectionObserver)
- All sections stack vertically, single column
- Form fields full-width, large touch targets (min 48px height)
- Video upload supports direct camera capture on mobile (`accept="video/*" capture="environment"`)

---

## 4. SECTION-BY-SECTION DESIGN SPECS

### Section 1: HERO
**Layout:** Two columns on desktop (60/40). Full-width stacked on mobile.
**Left column:**
- H1: "Znáte stav vašich pásů?" — `font-family: var(--font-display); font-weight: 800; font-size: var(--text-hero); color: var(--white); text-transform: uppercase;`
  *(Confirmed by Michal — this is the final H1.)*
- Subheadline: "Natočte pás mobilem. Umělá inteligence vyhodnotí poškození, která pouhým okem neuvidíte. Zprávu s mapou defektů dostanete do 5 pracovních dnů. První diagnostika zdarma." — `font-family: var(--font-body); font-weight: 400; font-size: 1.25rem; color: rgba(255,255,255,0.85); line-height: 1.6;`
- Primary CTA: "Chci bezplatnou diagnostiku" — `background: var(--rtt-red); color: white; padding: 16px 32px; font-weight: 700; font-size: 1.125rem; border: none; cursor: pointer;` → smooth-scrolls to `#form`
- Secondary link: "Stáhnout ceník (PDF) ↓" — `color: rgba(255,255,255,0.7); text-decoration: underline;` → opens PDF in new tab

**Right column:** Hero image (`news_mcube_cam.webp`) with subtle white border/frame. On mobile, image appears above text.

**Background:** `background: var(--rtt-black);` — solid dark, per brand manual. Subtle noise/grain texture overlay at 3-5% opacity for depth (CSS: `background-image: url(noise.svg); opacity: 0.04;`).

**Trust bar below hero:** Full-width strip, `background: var(--rtt-dark-gray);`
Content: REMA TIP TOP logo (white variant, small) + "Přes 100 let zkušeností v údržbě dopravníkových systémů | Pobočky v Brně a Praze"
`font-size: var(--text-small); color: var(--rtt-medium-gray);`

---

### Section 2: PROBLEM vs SOLUTION
**Background:** `var(--white)`
**Heading:** "Jak dnes probíhá kontrola pásů — a jak může" — H2
**Layout:** Comparison table, two columns with divider.
Left column header: "Běžná praxe" (slight red tint `var(--rtt-red)` as accent)
Right column header: "S MCube CAM" (green accent `var(--rtt-green)`)

4 rows as per v2 spec. Icons: simple line icons (Lucide or custom SVGs) next to each row.

**Below table — 3 stat callouts:**
Large text + supporting line. Arranged horizontally on desktop, stacked on mobile.
- "Neodhalit poškození včas stojí řádově statisíce." — number-style emphasis on "statisíce"
- "Jedna havárie spoje zastaví výrobu na hodiny až dny."
- "Většina poškození vzniká postupně — a je viditelná na záznamu dřív než okem."

`font-family: var(--font-display); font-size: 1.5rem; font-weight: 700; color: var(--rtt-black);`
Supporting text: `font-family: var(--font-body); font-size: var(--text-body); color: var(--text-muted);`

---

### Section 3: HOW IT WORKS
**Background:** `var(--off-white)`
**Heading:** "Celý postup ve 4 krocích"

**Layout:** 4 cards in horizontal row (desktop) or vertical stack (mobile).
Each card:
- Large step number (1–4) — `font-family: var(--font-display); font-size: 3rem; color: var(--rtt-red); font-weight: 800;`
- Step title in caps: "NATOČTE" / "NAHRAJTE" / "VYHODNOTÍME" / "ZPRÁVA"
- Icon (custom SVG or Lucide): camera / upload-cloud / cpu / file-text
- Description paragraph

**Below cards: "Co k natáčení potřebujete" subsection**
Simple list with check icons. Link to PDF setup manual.

**Prominent badge:** "Zprávu obdržíte do 5 pracovních dnů" — styled as a tag/pill, `background: var(--rtt-red); color: white; padding: 8px 20px; border-radius: 4px; font-weight: 600;`

---

### Section 4: SAMPLE REPORT
**Background:** `var(--rtt-black)` — dark, to make the report images pop
**Heading:** "Jak vypadá výstup" — white text

**Layout:** Image carousel/lightbox showing 4 key report pages (rendered from the Třinecké železárny report, anonymized). Files available:
- `report_sample_cover.png` — Cover page (client name redacted to "Ukázkový klient")
- `report_sample_summary.png` — AI summary with events table (115 events, 4 categories)
- `report_sample_beltmap_start.png` — Belt map Images 1-4 showing splice detection (green box) and first defects
- `report_sample_beltmap_critical.png` — Belt map Images 21-24 showing critical zone 145-164m with dense surface damage annotations (red=edge, blue=surface, orange=anomaly)

**IMPORTANT:** The belt map pages (beltmap_start, beltmap_critical) are the hero images — they show the colored bounding boxes around detected defects on the actual belt scan. These are what sell the product. Lead with these, not the cover page.

**Anonymization note:** The cover page has had "Trinecké železárny" replaced with "Ukázkový klient" and belt ID "274" replaced with "P-01". The summary and recommendations pages still contain references — use the belt map pages primarily in the carousel, and the cover/summary as secondary slides.

---

### Section 5: LEAD CAPTURE FORM
**Background:** `var(--white)`
**Heading:** "Objednat diagnostiku"

**Two-path layout:** Two cards side by side (desktop) / stacked (mobile):

**Card 1: "Nahraji video sám/sama"**
`border: 2px solid var(--rtt-red); border-radius: 8px; padding: 32px;`
Icon: upload-cloud
When selected → expands to full form below

**Card 2: "Chci návštěvu technika"**
`border: 2px solid var(--border-light); border-radius: 8px; padding: 32px;`
Icon: user-check
"Nemůžete natočit video sami? Přijedeme k vám."
When selected → shows simplified form (name, company, email, phone, site, preferred date)

**Full form (Card 1 expanded):**

**Section "O vás" (About you):**
| Field | Type | Required | Placeholder/Helper |
|-------|------|----------|-------------------|
| Jméno a příjmení | text | Yes | — |
| Společnost | text | Yes | — |
| E-mail | email | Yes | — |
| Telefon | tel | Yes | +420... |
| Lokalita / závod | text | Yes | Název závodu nebo adresa |

**Section "O pásu" (About the belt):**
| Field | Type | Required | Placeholder/Helper |
|-------|------|----------|-------------------|
| Název / ID dopravníku | text | Yes | Např. "Pás č. 3 — třídírna" |
| Umístění pásu | text | Yes | Popis, kde se pás nachází |
| Rychlost pásu (m/s) | number | Yes | Helper: "Najdete v dokumentaci nebo na štítku pohonu. Nevíte-li přesně, uveďte odhad." |
| Šířka pásu (mm) | number | Yes | Např. 800, 1000, 1200, 1600 |
| Délka pásu (m) | Yes | Yes | Celková délka nekonečného pásu |
| Poznámky | textarea | No | Cokoli, co by nám pomohlo — stáří pásu, známé problémy, typ materiálu... |

**Above the "O pásu" section, display:**
"Nemáte všechny údaje po ruce? Vyplňte, co znáte — zbytek doplníme při krátkém telefonátu."
`font-size: var(--text-small); color: var(--text-muted); font-style: italic;`

**Video upload zone:**
```html
<div class="upload-zone">
  <!-- Large drop zone with dashed border -->
  <input type="file" accept="video/mp4,video/quicktime,video/*" capture="environment" />
  <p>Přetáhněte video sem nebo klikněte pro výběr souboru</p>
  <p class="helper">Přijímáme .mp4 a .mov do 5 GB</p>
  <!-- Progress bar appears during upload -->
  <!-- After upload: green checkmark + filename + size -->
</div>
```

**Submit button:**
"Odeslat a získat diagnostiku zdarma"
`background: var(--rtt-red); color: white; width: 100%; padding: 18px; font-size: 1.125rem; font-weight: 700;`

**Below submit button, micro-copy:**
"Žádné skryté poplatky, žádné automatické objednávky. Zprávu dostanete a rozhodnutí je na vás."
`font-size: var(--text-small); color: var(--text-muted); text-align: center;`

**Path B section (below the form cards):**
"Potřebujete nejprve více informací?"
Two buttons:
- "Stáhnout ceník MCube CAM (PDF)" → opens PDF
- "Domluvit osobní schůzku" → short contact form (name, company, email, phone)

---

### Section 6: PRICING
**Background:** `var(--off-white)`
**Heading:** "Kolik diagnostika stojí"
**Subheading:** "Ceník uvádíme otevřeně. Žádné 'napište nám pro cenovou nabídku.'"

**Pricing table:**
Styled with `border-collapse; border: 1px solid var(--border-light);`
Header row: `background: var(--rtt-black); color: white;`
Alternating rows: white / `var(--off-white)`
Last row ("> 5 000 m"): `font-style: italic;`

| Délka pásu | Cena diagnostiky |
|---|---|
| 0 – 500 m | 350 € |
| 501 – 1 000 m | 600 € |
| 1 001 – 2 000 m | 1 100 € |
| 2 001 – 3 000 m | 1 600 € |
| 3 001 – 5 000 m | 2 600 € |
| > 5 000 m | Na dotaz |

**Three commercial points below** — each as a card with left border accent:
1. `border-left: 4px solid var(--rtt-green);` — "První diagnostika zdarma..." 
2. `border-left: 4px solid var(--rtt-blue);` — "Cena diagnostiky se odečítá..."
3. `border-left: 4px solid var(--rtt-yellow);` — "Pravidelný monitoring..."

---

### Section 7: VALUE PROPOSITION
**Background:** `var(--white)`
**Heading:** "Co vám MCube CAM přinese"
**Layout:** 3×2 grid (desktop), single column (mobile)
6 cards with icon + title + description. Icons from Lucide: shield-check, eye, clock, hard-hat, clipboard-check, trending-up. Card styling: `border: 1px solid var(--border-light); border-radius: 8px; padding: 24px;`

---

### Section 8: TRUST SIGNALS
**Background:** `var(--rtt-dark-gray)` (dark)
**Layout:** Centered text block
- REMA TIP TOP logo (white variant)
- MCube portfolio text
- CZ office locations
- GDPR note
All text: `color: var(--rtt-medium-gray);` with key terms in `color: var(--white);`

---

### Section 9: FOOTER
**Background:** `var(--rtt-black)`
- Company name: REMA TIP TOP INCO — CZ spol. s r.o.
- Address: Vídeňská 110, Brno
- Phone: +420 724 090 425
- Email: michal.feigler@rematiptop.cz
- Links: rema-tiptop.de | pogumovani.cz
- Language toggle: CZ / EN (EN can be placeholder for v1)
- © 2026 REMA TIP TOP INCO CZ

---

## 5. FORM BACKEND — Google Sheets + Email

### Google Apps Script webhook:

```javascript
// Deploy as web app → "Anyone" can access
function doPost(e) {
  var data = JSON.parse(e.postData.contents);
  
  // 1. Write to Google Sheet
  var sheet = SpreadsheetApp.openById('SHEET_ID').getActiveSheet();
  sheet.appendRow([
    new Date(),                    // Timestamp
    data.name,                     // Jméno
    data.company,                  // Společnost
    data.email,                    // E-mail
    data.phone,                    // Telefon
    data.location,                 // Lokalita
    data.conveyorId,               // ID dopravníku
    data.beltLocation,             // Umístění pásu
    data.beltSpeed,                // Rychlost
    data.beltWidth,                // Šířka
    data.beltLength,               // Délka
    data.notes,                    // Poznámky
    data.videoUrl,                 // URL nahraného videa
    data.formType,                 // "video_upload" nebo "technician_visit"
    "new"                          // Status
  ]);
  
  // 2. Notify sales team
  MailApp.sendEmail({
    to: 'michal.feigler@rematiptop.cz',
    subject: '🔔 Nová poptávka MCube CAM — ' + data.company,
    htmlBody: buildSalesNotification(data)
  });
  
  // 3. Auto-reply to customer
  MailApp.sendEmail({
    to: data.email,
    subject: 'Potvrzení přijetí — diagnostika MCube CAM',
    htmlBody: buildCustomerAutoReply(data)
  });
  
  return ContentService.createTextOutput(JSON.stringify({status: 'ok'}));
}
```

### Auto-reply email template (Czech):

**Subject:** Potvrzení přijetí — diagnostika MCube CAM

**Body:**
```
Dobrý den, [JMÉNO],

děkujeme za váš zájem o diagnostiku dopravníkového pásu pomocí MCube CAM.

Vaši poptávku jsme přijali a právě ji zpracováváme. Zde je, co můžete očekávat:

✓ Do 1 pracovního dne vám zavoláme a ověříme údaje.
✓ Video předáme ke zpracování pomocí AI analýzy.
✓ Do 5 pracovních dnů od ověření údajů obdržíte kompletní zprávu 
  s mapou defektů, fotografiemi a hodnocením závažnosti.
✓ Ke zprávě přiložíme konkrétní nabídku na opravu zjištěných závad.

Máte dotaz? Zavolejte na +420 724 090 425 nebo odpovězte na tento e-mail.

---

💡 TIP: Znáte záplatu FIX'N GO®?

Rychlé a trvalé řešení průrazů a poškození dopravních pásů — bez 
vulkanizace, bez dlouhých odstávek. Záplata s vysokou odolností 
proti otěru (50 mm³), šrouby integrovanými do pryže a hladkým 
povrchem kompatibilním se všemi stěrači.

Ideální pro okamžitou opravu defektů, které odhalí MCube CAM.

→ Více informací: https://www.pogumovani.cz/zaplata-fix-n-go
→ Objednat: odpovězte na tento e-mail nebo zavolejte +420 724 090 425

---

S pozdravem,
Michal Feigler
REMA TIP TOP INCO — CZ spol. s r.o.
Vídeňská 110, Brno
+420 724 090 425 | michal.feigler@rematiptop.cz
www.monitoringpasu.cz
```

---

## 6. ASSETS CHECKLIST

| Asset | Status | Action needed |
|-------|--------|---------------|
| REMA TIP TOP logo (color, 4c) | ✅ Extracted from CD Manual | File: `LOGO_REMA_TIP_TOP_color.png` |
| REMA TIP TOP logo (white on dark) | ✅ Extracted | File: `LOGO_REMA_TIP_TOP_white_and_red.png` (top half) |
| Hero image (smartphone on tripod) | ✅ Uploaded | File: `news_mcube_cam.webp` |
| Brand colors | ✅ From CD Manual | See CSS variables above |
| Sample MCube CAM report | ✅ Uploaded + anonymized | Files: `report_sample_cover.png`, `report_sample_summary.png`, `report_sample_beltmap_start.png`, `report_sample_beltmap_critical.png`, `report_sample_recommendations.png` |
| Fix'n Go product info | ✅ From pogumovani.cz | URL: https://www.pogumovani.cz/zaplata-fix-n-go |
| Sales contact email | ✅ Confirmed | michal.feigler@rematiptop.cz |
| Contact phone | ✅ Confirmed | +420 724 090 425 |
| Price list PDF | 🔨 To be created | Branded PDF with pricing table — can generate from spec |
| Setup manual PDF | 🔨 To be created | Proprietary one-page PDF, CZ + EN bilingual |
| Favicon | 🔨 To create | Crop from REMA TIP TOP logo — red "TIP TOP" mark at 32x32 |

---

## 7. DEPLOYMENT CHECKLIST

1. [ ] Set up Vercel project, connect to Git repo
2. [ ] Configure DNS: monitoringpasu.cz → Vercel
3. [ ] Create Google Sheet for form submissions
4. [ ] Deploy Google Apps Script webhook
5. [ ] Set up Cloudflare R2 bucket for video uploads
6. [ ] Configure CORS on R2 for presigned URL uploads from monitoringpasu.cz
7. [ ] Test form submission → Sheet + email notification + auto-reply
8. [ ] Test video upload (large file, mobile, various formats)
9. [ ] Embed sample report images (once uploaded by Michal)
10. [ ] Create and upload price list PDF
11. [ ] Mobile testing (iPhone Safari, Android Chrome)
12. [ ] Set up Plausible analytics
13. [ ] Lighthouse audit (target: 90+ on all metrics)
14. [ ] SSL certificate (automatic via Vercel)
15. [ ] Test Czech diacritics rendering across all sections

---

*Build ready. All blocking items resolved. Remaining to-create: price list PDF, setup manual PDF (CZ+EN bilingual one-pager), favicon.*
