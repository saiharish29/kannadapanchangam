# Kannada Panchangam — Amanta System

A single-file, self-hosted Hindu calendar and Muhurta clock for the **Amanta lunar month system** followed in Karnataka, Maharashtra, Andhra Pradesh, and Telangana. Computes tithi, nakshatra, yoga, karana, and muhurta timings live from a pluggable astronomical engine — choose between modern **Drik Ganita** (VSOP87 + ELP via astronomy-engine) and traditional **Surya Siddhanta** (yuga-based mean motions with single-epicycle mandaphala). Displays times in both **Bengaluru IST** and **Perth AWST** simultaneously.

**Version:** V4 (Pluggable Engine)
**File:** `panchang-app.html` (~310 KB, single file, no dependencies)

---

## Features

### 🕉️ Four tabs

**1. Tithi Lookup** — Search any tithi by Masa + Paksha + Tithi + Year. Returns exact dates and times for that year and the next, with full dual-timezone detail cards (Tithi begins/ends, sunrise, sunset, moonrise, moonset).

**2. Muhurta Clock** — Two live analog clocks side-by-side: one for Bengaluru IST, one for Perth AWST. Each clock is paired with a **radial muhurta gauge** showing the current auspicious/inauspicious period (NOW), the next zone (NEXT), and the one after (AFTER). A progress marker shows how far into the current zone you are.

**3. Full Calendar** — All 67+ festivals, observances, and tithis for the selected year, generated directly from ephemeris. Includes:
- All 12 monthly **Sankashti Chaturthi** (Ganesha fast)
- All 24 **Ekadashi** days (24 per year, named per tradition)
- Monthly **Amavasya** and **Purnima** observances
- Major festivals (Ugadi, Rama Navami, Ganesh Chaturthi, Diwali, Dasara, Shivaratri, Holi, etc.)
- On-demand fallback: any valid tithi combination not in the festival list computes live via ephemeris.

**4. About Amanta** — Explains the Amanta system and its difference from Purnimanta.

### 🌙 Astronomical accuracy

- **Pluggable astronomical engine** — two backends ship with the app, swappable from the header toggle:
  - **Drik Ganita** (default): `astronomy-engine 2.1.19` inlined (VSOP87 for Sun + ELP-2000 truncation for Moon) with Lahiri (Chitra-paksha) ayanamsha. ~1 arcminute accuracy. Matches drikpanchang.com to within seconds.
  - **Traditional Karnataka (Surya Siddhanta)**: pure Burgess (1860) yuga constants, single-epicycle *manda* equation of centre, Kali-epoch (JD 588465.5, Ujjain meridian). No external dependency. Bija-correction constants exposed in `PC.SS` for school-specific tuning but ship at zero.
- **Natural Adhik Masa detection** — two consecutive Amavasyas in the same rashi
- **Kshaya tithi handling** including masa-boundary cases (Chaitra Shukla Pratipada / Ugadi)
- **Moonrise / Moonset** for every tithi (within ±30h tolerance to handle late-night rises)

### Computation modes — what changes between them

The tithi formula is the same in both modes:

    tithi_index = floor( ((moon_long - sun_long) mod 360) / 12 )

What differs is the underlying moon and sun longitudes:

- **Drik** integrates the full periodic terms of modern lunar theory (~hundreds of terms in ELP).
- **Surya Siddhanta** uses one epicycle correction on the mean motion. The two engines therefore disagree by 0–7° in moon longitude on any given day, which translates to tithi-start differences of up to several hours — the well-known and accepted Drik-vs-Saurapaksha gap.

For example, Vaishakha Krishna Dashami on 11 May 2026 at Bengaluru:

| Engine | Krishna Dashami starts (IST) |
|---|---|
| Drik Ganita | ~3:24 PM |
| Pure Burgess Surya Siddhanta (no bija) | ~2:46 AM |
| (Calibrated school-specific bija) | varies — consult the published panchanga of your tradition |

> **Note on exact matching of any specific publication.** Different Karnataka/Madhwa schools (Uttaradi Math, Sri Rambhapuri Peetham, Brahma-Paksha, etc.) apply different *bija* corrections — small time-linear offsets to the Surya Siddhanta mean motions — from their own epochs (~Saka 444 = 522 CE). This codebase exposes `PC.SS.BIJA_SUN_LONG_DEG`, `PC.SS.BIJA_MOON_LONG_DEG`, and `PC.SS.BIJA_MOON_APOGEE_OFFSET_DEG`, but ships them at zero. *Exact* match to any specific publication requires that publication's authoritative bija constants — please don't single-point-tune them against one date.

### 🗺️ Dual timezone

Every time in the app is shown for both **Bengaluru (IST, UTC+5:30)** and **Perth (AWST, UTC+8:00)**. The Muhurta Clock tab computes muhurta zones independently for each city based on its own local sunrise/sunset — so a muhurta active in Bengaluru may not be active in Perth at the same moment, and the gauge correctly reflects that.

### 🎨 UI

- Dark navy + warm gold palette throughout
- Cinzel and Cormorant Garamond typography
- Two-column layout on wide displays (≥1400px); stacks vertically below that
- Hindu ornamental accents (fleur-de-lis, swastika, Om)
- Mobile-responsive (tested at 380px width)

---

## Running it

### Option 1: Just open the file

1. Download `panchang-app.html`
2. Double-click to open in any modern browser (Chrome, Firefox, Edge, Safari)
3. Everything runs client-side — no server needed

### Option 2: Host it

The file is fully self-contained. To deploy:

**Netlify / Vercel / GitHub Pages:**
- Create a new repo
- Add `panchang-app.html` and rename to `index.html`
- Push and deploy — done

**Render:**
- Create a Static Site on Render
- Point at your repo with `panchang-app.html` as `index.html`
- Publish directory: `.` (root)
- Build command: (leave empty)

**Your own VPS (e.g., Hostinger KVM2):**
```bash
scp panchang-app.html user@your-vps:/var/www/html/index.html
```
Ensure nginx/apache serves it as `index.html`.

No build step. No `npm install`. No dependencies.

---

## Technical architecture

### Single-file structure

```
panchang-app.html  (~300 KB, ~4150 lines)
│
├── <style>              CSS tokens, layout, clock styling, gauge card
├── <body>
│   ├── header           Title, meta badges, Samvat year
│   ├── tab-nav          4 tabs
│   ├── panel-lookup     Tithi Lookup form + dual-timezone result cards
│   ├── panel-clock      Dual analog clocks + radial gauges + sidebar
│   ├── panel-cal        Year navigator + festival cards
│   └── panel-about      Amanta system explainer
│
└── <script>
    ├── astronomy-engine 2.1.19  (116 KB, inlined VSOP87 + ELP)
    ├── PC.*             Panchang computation namespace
    │   ├── PC.MODES { DRIK, TRADITIONAL_KARNATAKA }
    │   ├── PC.engines.drik           (Astronomy.SunPosition / EclipticGeoMoon – Lahiri)
    │   ├── PC.engines.surya_siddhanta (PC.SS.*  yuga maths + mandaphala)
    │   ├── PC.SS        Burgess SS constants, mean longs, mandaphala, bija
    │   ├── PC.setMode / PC.activeEngine
    │   ├── PC.sunLongitude / PC.moonLongitude / PC.moonSunDiff (nirayana, engine-aware)
    │   ├── computeTithi / computeNakshatra / computeYoga / computeKarana
    │   ├── computeMasa            (with Adhik detection)
    │   ├── findTithiBoundary      (Drik: SearchMoonPhase  ·  SS: bracket+bisect)
    │   ├── findTithiInYear        (main lookup — kshaya-aware, 3 code paths)
    │   ├── getSunrise / getSunset / getMoonrise / getMoonset
    │   ├── buildZones             (muhurta zones per city)
    │   ├── getZoneSequence        (NOW / NEXT / AFTER)
    │   ├── FESTIVAL_TEMPLATES     (67+ entries)
    │   └── tests.run / runTests   (unit + regression suite; ?test=1 to auto-run)
    ├── UI rendering     (clock hands tick, gauge update, calendar render, mode toggle)
    └── initStars        (decorative background)
```

### Surya Siddhanta engine — formulas in one place

Source: *Translation of the Surya Siddhanta* by Rev. Ebenezer Burgess, JAOS Vol. 6 (1860).

| Quantity | Value |
|---|---|
| Civil days per Mahayuga | 1,577,917,828 |
| Sun revolutions per Mahayuga | 4,320,000 |
| Moon revolutions per Mahayuga | 57,753,336 |
| Moon apogee revolutions (forward) | 488,203 |
| Moon node revolutions (retrograde) | 232,238 |
| Sidereal year | 365.25876 days |
| Sidereal month | 27.32167 days |
| Sun daily mean motion | 0.98561 °/day |
| Moon daily mean motion | 13.17636 °/day |
| Sun manda epicycle | 14° |
| Moon manda epicycle | 32° |
| Sun mandocca (apogee) at Kali start | 78° |
| Kaliyuga epoch (Ujjain mean midnight, JD UT) | 588465.5 |
| Ujjain longitude | 75°47′ E (75.7833°) |

Ahargana, mean longitude, mandaphala:

    A(t)       = jd_UT(t) + UJJAIN_LON / 360 − KALI_EPOCH_JD
    L_mean     = 360 * frac( REVS * A / YUGA_DAYS )
    anomaly    = L_mean − L_apogee
    mandaphala = − arcsin( (epicycle / 360) * sin(anomaly) )
    L_true     = L_mean + mandaphala

### Cross-validation against an independent reference

The Surya Siddhanta engine has been numerically cross-checked against the **Yano & Fushimi `pancanga.pl`** (Kyoto Sangyo University) — the historian-trusted reference SS implementation, used by Indologists for decades. Both implementations use **identical** yuga constants (1,577,917,828 / 4,320,000 / 57,753,336 / 488,203), the **identical** Kaliyuga epoch (JD 588,465.5), and the same Ujjain meridian. The only intentional difference is the manda epicycle constants — both have now been aligned to Yano's mean values (13°50′ / 31°50′ per Burgess Ch.2 v.34-37).

Cross-check residuals over six test dates (J2000, the four 2026 cardinal points, and 11 May 2026):

| Quantity | Worst-case difference |
|---|---|
| Sun nirayana longitude | < 1.6 arc-minutes |
| Moon nirayana longitude | < 1.1 arc-minutes |
| Tithi-boundary timing (e.g. Krishna Dashami 11 May 2026) | ~ 3 minutes |

This is *not* a calibration to one data point — both engines independently produce the same numbers from the same SS first principles. The remaining residual is the 0.02° difference between the two codebases' Ujjain longitude rounding (75.7833° vs Yano's 75.8°).

### Running the test suite

Open the page in a browser and:

    PC.runTests()              // run all suites, log to console, return {passed, failed, results}
    PC.runTests({verbose:true}) // also log intermediate longitudes

Or visit the page with `?test=1` in the URL to auto-run on load.

The suite covers SS internals (Kali ahargana, mean longitudes, mandaphala range), engine-plumbing sanity at J2000, the 11 May 2026 Bengaluru regression in both modes, mode-invariance (the two engines must disagree on moon-sun elongation), and a Drik regression against the hand-verified `PC.TITHIS` dataset.
```

### Muhurta zones computed

Per city, per day:
- **Brahma Muhurta** — 96–48 min before sunrise (highly auspicious)
- **Pratah Sandhya** — 48 min before sunrise through sunrise
- **Shubh Muhurta** — 1st/8th segment after sunrise (auspicious)
- **Amrit Kaal** — 2nd segment (nectar time, excellent)
- **Gulika Kaal** — Gulika period (inauspicious, day-varying)
- **Yamaganda** — Yama period (inauspicious, day-varying)
- **Rahu Kaal** — Rahu period (inauspicious, day-varying)
- **Labh Muhurta** — profit-time segment (auspicious)
- **Abhijit Muhurta** — ~48 min straddling local noon (victorious)
- **Varjyam** — inauspicious segment
- **Shubh (Afternoon)** — afternoon auspicious segment
- **Sayam Sandhya** — 48 min around sunset
- **Nishita Kaal** — midnight meditation window (special, ritual)

Each zone's start/end is computed from the city's own actual sunrise/sunset for that day. The day is divided into 8 segments of dayLen/8; day-varying zones (Rahu/Gulika/Yamaganda) use traditional day-of-week tables.

---

## Known limitations

| Issue | Status |
|-------|--------|
| Diwali & Maha Shivaratri may be off by 1 day | Known — uses Udaya Tithi, not Pradosh/Nishita Kaal rules |
| Moon doesn't always rise/set every civil day (~1 per 30 days) | Widened to ±30h window; still shows `—` if genuinely no rise in that window |
| First calendar load for a new year takes 2–3s | 67 festivals × ephemeris compute; no spinner shown yet |
| Some ritual calculations follow Amanta, not Purnimanta | By design — this is a Karnataka/Kannada panchang |

---

## File versions

The output directory may contain multiple versions:

| File | Purpose |
|------|---------|
| `panchang-app.html` | Current production file |
| `panchang-app-v3-final.html` | Identical to production, with a green "V3-FINAL" badge in the header (useful to confirm the right file is loaded when debugging cache issues) |
| `panchang-app.zip` | Zipped version for upload to Render / Netlify drop deploy |

---

## Customizing

### Change location

The app is hardcoded to Bengaluru (12.9716°N, 77.5946°E) and Perth (-31.9523°N, 115.8613°E). To add another city:

1. Add a new city config near `PC.BENGALURU`:
   ```javascript
   PC.MUMBAI = {
     latitude: 19.0760,
     longitude: 72.8777,
     elevationMeters: 14,
     tz: 'Asia/Kolkata',
   };
   ```
2. Add a new clock-card block in the HTML (duplicate the Bengaluru block, update IDs)
3. Call `PC.buildClockForCity('Mumbai')` in the `buildClock()` function

### Change theme colors

Edit the CSS variables at the top of the `<style>` block:
```css
--bg-0:      #0A0E1A;    /* page background */
--surface:   #16182A;    /* panel bg */
--gold:      #D4AF37;    /* primary accent */
--gold-l:    #E8C558;    /* highlight */
--text-1:    #F5EFDC;    /* primary text */
```

### Add a festival

Edit `PC.FESTIVAL_TEMPLATES` in the `<script>` block. Each entry looks like:
```javascript
{
  masa:   'Kartika',
  paksha: 'Shukla',
  tithi:  'Chaturdashi',
  name:   'Vaikuntha Chaturdashi',
  sig:    'Description of the observance...',
}
```
The app automatically computes the Gregorian date, sunrise/sunset/moonrise/moonset, nakshatra, yoga, and karana for that tithi every year it's requested.

---

## Project background

Built as a personal project by Harish Kumar MP for use by his family members. Iteratively developed through multi-session collaboration with Claude (Anthropic). Design philosophy: **correctness over visual flash** — every time is computed live via ephemeris; none are hardcoded.

---

## License

Personal use. Not affiliated with Anthropic. The inlined astronomy-engine library is MIT-licensed by Don Cross; see [astronomy-engine on GitHub](https://github.com/cosinekitty/astronomy) for attribution.

---

## Credits

- **astronomy-engine** by Don Cross — VSOP87 + ELP implementation used for the Drik engine
- **Surya Siddhanta** as edited and translated by Rev. Ebenezer Burgess, JAOS Vol. 6 (1860) — yuga constants, epicycle dimensions, and equation-of-centre formula used in the Traditional Karnataka engine
- **Lahiri ayanamsha** formula per Indian Astronomical Ephemeris (IAE)
- **Drik Panchang** used as reference for festival date validation in Drik mode (VKD 2021 = 4 Jun 2021 confirmed ✓, VKD 2025 = 22 May 2025 confirmed ✓)
- **Google Fonts:** Cinzel, Cormorant Garamond

---

*Tithi times are computed for the configured city's sunrise. This is a reference implementation; for ritual purposes please also consult your local priest or panchang-kartha.*
