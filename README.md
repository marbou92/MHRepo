# MHRepo

Mihon extension repository hosting signed, self-published builds from [MHExtensions](https://github.com/marbou92/MHExtensions) — six sources for sites that mainstream catalogs dropped or never covered, all with built-in Cloudflare handling.

## Add to Mihon

```
https://raw.githubusercontent.com/marbou92/MHRepo/main/repo.json
```

**Settings → Browse → Extension repos → Add** — then refresh the Extensions screen and install from the "MarBou" section.

## Available extensions

| Extension | Version | Language | Content | Site |
|-----------|---------|----------|---------|------|
| Comix | v1.4.44 | All | Mixed | [comix.to](https://comix.to) |
| ManhuaRMTL | v1.6.85 | All | NSFW | [manhuarmtl.com](https://manhuarmtl.com) |
| Atsumaru | v1.4.28 | English | Mixed | [atsu.moe](https://atsu.moe) |
| Kagane | v1.6.45 | English | Mixed | [kagane.to](https://kagane.to) |
| MangaBall | v1.6.1 | English | Mixed | [mangaball.net](https://mangaball.net) |
| MKissa | v1.6.19 | English | Mixed | [mkissa.to](https://mkissa.to) |

> The table above reflects the latest publish. The always-current, machine-readable list is [`index.json`](index.json) — it is regenerated automatically by CI on every publish, so it may briefly be one release ahead of this file.

### Extension notes

- **Comix** — multilingual (`all`) source for comix.to: signed API requests, image decryption and descrambling, per-source preferences.
- **ManhuaRMTL** — machine-translated manhua/manhwa in 12 languages, selectable per chapter.
- **Atsumaru** — atsu.moe via its open JSON API (Typesense-powered search, no Cloudflare bypass needed).
- **Kagane** — kagane.to (Komga-fork API): silent bearer-token refresh, images served from an open CDN.
- **MangaBall** — mangaball.net; uses the exact same Cloudflare bypass method as Comix.
- **MKissa** — mkissa.to (GraphQL persisted queries); chapter pages are gated behind a Turnstile captcha that the in-app solver clears automatically — no manual tap required.

## Cloudflare-protected sources

All six sources sit behind Cloudflare. The shared solver bundled in the extensions:

- **Detects real challenges strictly** (`cf-mitigated` header or challenge-page body markers), so ordinary JSON API 403s never purge a still-valid `cf_clearance`;
- **Runs the WebView attached to a real window** (parked invisibly behind the app's own UI) so Cloudflare Turnstile renders and auto-solves — a never-attached WebView reports `visibilityState=hidden` and silently can never solve;
- **Keeps clearance for the long term** — measured TTL is 365 days on kagane.to and manhuarmtl.com, so the "checking your browser" step should be a one-time event per site.

If a source ever shows a challenge prompt, solving it once is enough; there is no hourly re-check.

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

Extensions are built against extension library **1.4** (Comix, Atsumaru) and **1.6** (ManhuaRMTL, Kagane, MangaBall, MKissa). Any current Mihon / Tachiyomi-fork release supports both — if an extension shows as "incompatible", update your app first.

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
