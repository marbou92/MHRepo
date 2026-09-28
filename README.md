# MHRepo

Mihon extension repository hosting signed, self-published builds from [MHExtensions](https://github.com/marbou92/MHExtensions) — nine sources for sites that mainstream catalogs dropped or never covered, with built-in Cloudflare handling wherever the site needs it.

## Add to Mihon

```
https://raw.githubusercontent.com/marbou92/MHRepo/main/repo.json
```

**Settings → Browse → Extension repos → Add** — then refresh the Extensions screen and install from the "MarBou" section.

## Available extensions

| Extension | Version | Language | Content | Site |
|-----------|---------|----------|---------|------|
| Atsumaru | v1.4.28 | English | Mixed | [atsu.moe](https://atsu.moe) |
| Comix | v1.4.44 | All | Mixed | [comix.to](https://comix.to) |
| Kagane | v1.6.49 | English | Mixed | [kagane.to](https://kagane.to) |
| MangaBall | v1.6.5 | All | Mixed | [mangaball.com](https://mangaball.com) |
| ManhuaRMTL | v1.6.89 | All | Mixed | [manhuarmtl.com](https://manhuarmtl.com) |
| MKissa | v1.6.23 | English | Mixed | [mkissa.to](https://mkissa.to) |
| Swarm | v1.4.1 | All | Mixed | [swarm.ws](https://swarm.ws) |
| WeebCentral | v1.4.1 | English | Mixed | [weebcentral.com](https://weebcentral.com) |
| XComic | v1.4.1 | All | Mixed | [xcomic.me](https://xcomic.me) |

> The table above reflects the latest publish. The always-current, machine-readable list is [`index.json`](index.json) — it is regenerated automatically by CI on every publish, so it may briefly be one release ahead of this file.

### Extension notes

- **Atsumaru** — atsu.moe via its open JSON API (Typesense-powered search; no Cloudflare gate).
- **Comix** — multilingual (`all`) source for comix.to: signed API requests, image decryption and descrambling, per-source preferences.
- **Kagane** — kagane.to (Komga-fork API): silent bearer-token refresh, images served from an open CDN.
- **MangaBall** — rebuilt for the new mangaball.com (the old .net domain is gone): talks straight to the site's JSON API — no CSRF tokens — with advanced search filters (status, demographic, language, year, genres), a NSFW toggle and rich descriptions. The source id is independent of the domain, so the .net → .com migration preserved every library entry.
- **ManhuaRMTL** — machine-translated manhua/manhwa in 12 languages, selectable per chapter, with an on-device OCR text overlay that re-renders instantly when its settings change.
- **MKissa** — mkissa.to (GraphQL persisted queries); chapter pages are gated behind a Turnstile captcha that the in-app solver clears automatically, replaying each request with a freshly signed payload.
- **Swarm** — swarm.ws: chapter language (including English, Arabic and French) picked in settings, official-translation preference, Comix-style options; the encrypted page API is decrypted inside the extension.
- **WeebCentral** — weebcentral.com with full advanced search (sort, order, status, type, adult, official, include/exclude genres), a NSFW toggle, tag chips and blocked-genre filtering.
- **XComic** — xcomic.me and its three same-backend mirrors (xcomic.net, comik.to, yona.to): pick a mirror or add your own in the extension settings; chapters are grouped per scanlation team with English, Arabic and French translations.

## Cloudflare handling

Five sources (Comix, ManhuaRMTL, Kagane, MangaBall, MKissa) sit behind active Cloudflare protection. The shared solver bundled in the extensions:

- **Detects real challenges strictly** (`cf-mitigated` header or challenge-page body markers), so ordinary JSON API 403s never purge a still-valid `cf_clearance`;
- **Runs the WebView attached to a real window** (parked invisibly behind the app's own UI) so Cloudflare Turnstile renders and auto-solves — a never-attached WebView reports `visibilityState=hidden` and silently can never solve;
- **Self-heals before expiry** — the solver remembers when each site's clearance was last minted. When a request arrives while the clearance is more than a few hours old, it silently re-verifies it in the background (the site root loads in the attached WebView; if a challenge appears it is solved exactly as usual, and if the page loads clean the clearance was still valid). Returning after hours away costs a 5–15 second invisible pause on the first load instead of a manual WebView trip — and most of the time nothing happens at all.

WeebCentral and XComic are also fronted by Cloudflare but currently serve regular clients without a challenge; the solver ships in those extensions as a fallback. Atsumaru and Swarm use open APIs with no Cloudflare gate.

## How it works

This repo is automatically updated by the [MHExtensions](https://github.com/marbou92/MHExtensions) CI:

1. Source code changes are pushed to MHExtensions
2. The **Release & Publish** workflow builds signed release APKs
3. The publish script (`publish-repo.py`) pushes APKs, JARs, and the protobuf index here
4. The jsDelivr CDN cache is purged so updates appear immediately

## Repository structure

```
MHRepo/
├── repo.json          # Repo descriptor (points Mihon to index.pb)
├── index.pb           # Protobuf v2 index (gzip-compressed binary — what Mihon reads)
├── index.json         # Protobuf v2 index in JSON format (human-readable)
├── index.min.json     # Legacy v1 marker file (for old Tachiyomi apps)
├── index.html         # Web listing page
├── apk/               # All signed release APKs
├── jar/               # All signed extension JARs
├── icon/              # Per-extension icons
└── README.md          # This file
```

### `repo.json`

```json
{
  "index_v2": "https://raw.githubusercontent.com/marbou92/MHRepo/main/index.pb",
  "meta": {
    "name": "MarBou",
    "website": "https://github.com/marbou92/MHExtensions",
    "signingKeyFingerprint": "<SHA-256 fingerprint>"
  }
}
```

Mihon fetches `repo.json`, reads `meta.signingKeyFingerprint` to verify APK signatures, then fetches `index_v2` (the protobuf binary at `index.pb`) to get the extension list.

## Compatibility

Extensions are built against extension library **1.4** (Atsumaru, Comix, Swarm, WeebCentral, XComic) and **1.6** (Kagane, MangaBall, ManhuaRMTL, MKissa). Any current Mihon / Tachiyomi-fork release supports both — if an extension shows as "incompatible", update your app first.

## Signing

All APKs are signed with a personal keystore. The SHA-256 fingerprint of the signing certificate is included in `repo.json` → `meta.signingKeyFingerprint`. Mihon uses this to verify that APKs come from a trusted source.

## CDN

- **APKs/JARs** → jsDelivr CDN (`cdn.jsdelivr.net`) — fast global delivery
- **Index files** → `raw.githubusercontent.com` (5-minute cache for fast updates) + jsDelivr purge on publish

## Related

- **Source code:** [marbou92/MHExtensions](https://github.com/marbou92/MHExtensions)
- **Build infrastructure:** [Keiyoushi/extensions-source](https://github.com/keiyoushi/extensions-source)
- **App:** [Mihon](https://github.com/mihonapp/mihon)

## License

See [LICENSE](LICENSE).
