# Chronolocation Calculator · SIG-TLS-002

Verify the date, time, and location of a still image or video using solar position. Returns sun altitude, azimuth, sunrise, sunset, optional shadow-length verification, and a copy-ready dossier report. Built for newsroom verification desks, accountability researchers, and working investigators.

Tool ID: **SIG-TLS-002**
Version: **v0.1**
Standard: [LST-001 v1.0.3](https://signalandshadow.io/verification-standards)
Live: [signalandshadow.io/chronolocation-calculator](https://signalandshadow.io/chronolocation-calculator)

---

## What it does

Given coordinates, a date, and a time (with optional UTC offset), the tool computes:

- **Sun altitude** in degrees above (or below) the horizon
- **Sun azimuth** as a compass bearing
- **Sunrise, solar noon, and sunset** in UTC
- **24-hour solar arc** showing the sun's path across the day with the analysed moment marked
- **Sun-position diagram** showing the sun on a compass dome
- **Location map** with the sun's azimuth projected as a 500m line from the analysed point
- **Shadow verification** (optional): given an object's height and the measured shadow length, the tool calculates the expected shadow at the computed sun altitude and flags discrepancies outside a 15% tolerance

The output is a copy-ready text report formatted to the SS-DOSSIER convention, including the input parameters, the formula applied, intermediate values, and a confidence assessment.

## Use cases

- **Verifying time-of-day claims** in eyewitness imagery (does the shadow direction match the time the source claims?)
- **Geolocating** unknown-location imagery where a known landmark casts a measurable shadow
- **Cross-checking metadata**: an image's EXIF timestamp can be checked against the sun position visible in the scene
- **Pre-flight verification** before staging outdoor photography or video for accuracy

Typical accuracy is ±0.1° on altitude and azimuth (matching NOAA reference values within rounding). Sunrise and sunset times round to the nearest 15 minutes due to sample interval.

## How it works

The solar position calculation uses the standard NOAA / Jean Meeus algorithm:

```
sin(θs) = sin(φ)·sin(δ) + cos(φ)·cos(δ)·cos(h)
```

where φ is observer latitude, δ is solar declination, and h is local hour angle. The implementation is vanilla JavaScript with no dependencies beyond [Leaflet](https://leafletjs.com) for the map picker.

For background on the methodology and how the tool fits into a verification workflow, see the field-ready reference cards on signalandshadow.io.

## Repository contents

```
.
├── index.html        # The tool. Self-contained: HTML, inline CSS, inline JS.
├── README.md         # This file.
├── LICENSE           # MIT.
└── CNAME             # Custom-domain configuration for GitHub Pages.
```

That's it. No build step, no package.json, no toolchain.

## Deployment

This tool is designed to be embedded as an iframe on `signalandshadow.io/chronolocation-calculator`. It also works as a standalone page.

### GitHub Pages (standalone)

1. **Create a new public repo** on GitHub: `signalandshadow/chronolocation-calculator`.
2. **Drag this folder** into GitHub Desktop and commit all four files.
3. Push to `main`.
4. In the repo on github.com, go to **Settings → Pages**.
5. Under **Build and deployment**, set **Source** to `Deploy from a branch`, branch `main`, folder `/ (root)`. Save.
6. Wait roughly a minute. The repo will deploy to `https://signalandshadow.github.io/chronolocation-calculator/`.

### Custom domain

The included `CNAME` file points to `chronolocation-calculator.signalandshadow.io`. To use this:

1. In your DNS provider, add a CNAME record:
   - **Host**: `chronolocation-calculator`
   - **Value**: `signalandshadow.github.io`
2. In **Settings → Pages → Custom domain**, enter `chronolocation-calculator.signalandshadow.io` and tick **Enforce HTTPS** once the certificate provisions.

If you'd rather host on the apex `signalandshadow.io/chronolocation-calculator` path (no subdomain), you'd need a reverse-proxy or path-aware DNS, which GitHub Pages doesn't support natively. The intended deployment is via iframe embed (see below).

### Iframe embed (production)

On the `signalandshadow.io/chronolocation-calculator` page, embed the deployed tool with:

```html
<iframe
  id="ss-tls-002-frame"
  src="https://chronolocation-calculator.signalandshadow.io/"
  style="width: 100%; border: none; min-height: 1400px;"
  title="Chronolocation Calculator"
  loading="lazy"
></iframe>

<script>
  // Auto-resize the iframe to fit content, and scroll the parent when results render.
  // Mirrors the convention used by the OSINT Tool Database (sas-tools-* messages).
  window.addEventListener('message', (e) => {
    if (!e.data || typeof e.data !== 'object') return;
    if (e.data.tool !== 'SIG-TLS-002') return;

    const frame = document.getElementById('ss-tls-002-frame');
    if (!frame) return;

    if (e.data.type === 'ss-tls-002-height' && typeof e.data.height === 'number') {
      frame.style.height = (e.data.height + 16) + 'px';
    }
    if (e.data.type === 'ss-tls-002-scroll-to-top') {
      frame.scrollIntoView({ behavior: 'smooth', block: 'start' });
    }
  });
</script>
```

The tool sends two `postMessage` events to the parent:

- `ss-tls-002-height` (continuously): current document height in pixels. Use it to size the iframe so there's no internal scrollbar.
- `ss-tls-002-scroll-to-top` (after results render): a hint that the user has just hit "Calculate" and the parent should scroll the iframe back to the top.

Both are no-ops when the page is loaded standalone (when `window.parent === window`).

## Local development

The tool is a single static HTML file. Any local web server works:

```bash
cd chronolocation-calculator
python3 -m http.server 8000
# open http://localhost:8000/
```

Opening `index.html` directly via `file://` will work for the form and the calculations, but the map tiles may not load due to browser file-protocol restrictions.

## Dependencies

- **[Leaflet](https://leafletjs.com) 1.9.4** for the map picker. Loaded from `cdn.jsdelivr.net` with `crossorigin="anonymous"` so error reporting works correctly.
- **[OpenStreetMap](https://www.openstreetmap.org) tiles** for the map background. Subject to [OSM's tile usage policy](https://operations.osmfoundation.org/policies/tiles/); for heavy production traffic, consider switching to a hosted tile provider.
- **[Geist](https://fonts.google.com/specimen/Geist)** and **[JetBrains Mono](https://fonts.google.com/specimen/JetBrains+Mono)** from Google Fonts.

If any of these fail to load (CDN blocked, ad-blocker, offline), the tool degrades gracefully:

- No Leaflet → coordinates can still be entered manually; the map is replaced with a notice.
- No tiles → the map container shows "TILE SERVER UNREACHABLE" with the same fallback guidance.
- No fonts → falls back to system sans-serif and monospace.

The solar math runs entirely in the browser with no network dependency.

## Editorial standard

All copy follows [LST-001 v1.0.3](https://signalandshadow.io/verification-standards):

- No em dashes anywhere
- British English, AP Style
- Practitioner voice
- Specific numbers over vague tiers
- Provenance per claim where the operational note is non-obvious

## Algorithm attribution

Solar position math: NOAA Solar Position Algorithm, derived from Jean Meeus, *Astronomical Algorithms* (2nd ed.), Willmann-Bell, 1998. Implementation is the simplified short-form variant accurate to ±0.01° within ±100 years of J2000.

## Verification

| Field | Value |
| --- | --- |
| Tool ID | SIG-TLS-002 |
| Version | v0.1 |
| Verifier | DB |
| Last verified | 2026-05-06 |
| Standard | LST-001 v1.0.3 |
| Algorithm | NOAA / Meeus solar position |

Verified end-to-end against four reference cases:

- Eiffel Tower, summer solstice 2026 noon UTC: altitude 64.53°, azimuth 183.93° S
- Singapore equinox noon: altitude ~88° (near zenith)
- Reykjavík January noon: altitude ~5° (low winter sun)
- Geneva winter midnight: altitude -66° (sun deep below horizon)

All within ±0.1° of NOAA reference values.

## License

MIT. See [LICENSE](./LICENSE).

## Citation

If you use this tool in published work:

> Bowler, D. (2026). *Chronolocation Calculator (SIG-TLS-002 v0.1)* [Computer software]. Signal & Shadow. https://signalandshadow.io/chronolocation-calculator

---

© 2026 Signal & Shadow, Versoix Geneva. Part of the [Signal & Shadow Intelligence Hub](https://signalandshadow.io).
