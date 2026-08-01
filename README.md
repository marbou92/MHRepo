# MHRepo

Published Tachiyomi/Mihon extensions for [MHExtensions](https://github.com/marbou92/MHExtensions).

## Add to Mihon

```
https://raw.githubusercontent.com/marbou92/MHRepo/main/repo.json
```

**Settings → Browse → Extension repos → Add**

## Available extensions

| Extension | Version | Language | NSFW | Site |
|-----------|---------|----------|------|------|
| Comix | v1.4.21 | All | Mixed | [comix.to](https://comix.to) |
| ManhuaRMTL | v1.4.5 | English | NSFW | [manhuarmtl.com](https://manhuarmtl.com) |

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
└── README.md          # This file
```

### `repo.json`

```json
{
  "index_v2": "https://raw.githubusercontent.com/marbou92/MHRepo/main/index.pb",
  "meta": {
    "name": "MHExtensions",
    "website": "https://github.com/marbou92/MHExtensions",
    "signingKeyFingerprint": "<SHA-256 fingerprint>"
  }
}
```

Mihon fetches `repo.json`, reads `meta.signingKeyFingerprint` to verify APK signatures, then fetches `index_v2` (the protobuf binary at `index.pb`) to get the extension list.

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
