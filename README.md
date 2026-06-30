# CinderBlock

**Content Blocking Engine for Ember Browser**

CinderBlock is Ember's built-in content blocking engine, forked from [uBlock Origin](https://github.com/gorhill/uBlock). It is not an extension — it is a first-class component of Ember, compiled as part of the browser and deeply integrated into the Ember Shell.

## What It Does

CinderBlock blocks ads, trackers, coin miners, popups, annoying anti-blockers, malware sites, and more. It inherits uBlock Origin's core architecture: a highly efficient network request filter, a cosmetic filter engine, a scriptlet injection system, and a full dashboard UI.

## Features

- **Network Request Filtering** — Intercepts all HTTP/HTTPS requests, matches against compiled filter lists, blocks or allows before reaching the network
- **Cosmetic Filtering** — CSS-based element hiding to remove ad containers, cookie banners, popups, and other annoyances
- **Scriptlet Injection** — JavaScript snippets injected into page context to neutralize anti-adblock, tracking, and fingerprinting scripts
- **Dashboard** — Full UI accessible via toolbar button, keyboard shortcut (`Ctrl+Shift+B`), or `ember://cinderblock`
- **Element Picker** — Point-and-click tool to generate cosmetic filter rules for any page element
- **Network Logger** — Real-time log of all network requests made by the current tab
- **Dynamic Filtering** — Per-domain request type matrix for advanced users
- **Per-Site Rules** — Whitelist, disable cosmetic filtering, or disable scriptlet injection on specific domains

## Filter Lists

CinderBlock ships with a curated default filter list set:

| List | Category | Default |
|---|---|---|
| AdGuard Base | Ads | on |
| AdGuard Tracking Protection | Tracking | on |
| AdGuard Annoyances | Cookie banners, popups | on |
| EasyList | Ads | on |
| EasyPrivacy | Tracking | on |
| Peter Lowe's Ad and Tracking | Ads + Tracking | on |
| uBlock Origin Filters | Ads | on |
| uBlock Annoyances | Annoyances | on |
| Dan Pollock's hosts | Malware, spam | on |
| PhishTank | Phishing domains | on |
| HAGEZI Multi Pro | Ads + Tracking + Malware | on |
| StevenBlack Adware + Malware | Adware, malware | on |
| oisd big | Comprehensive | on |
| StevenBlack Gambling | Gambling domains | off |
| Adult content | Adult content | off |

## Configuration

CinderBlock reads its initial state from the `[blocking]` section of `ember.toml`:

```toml
[blocking]
enabled = true
update_interval_hours = 24

[blocking.lists]
adguard_annoyances = true
stevenblack_gambling = false
adult_content = false
```

User settings are stored in `~/.config/ember/cinderblock/`.

## Building

```bash
# Install dependencies
npm install

# Build
npm run build

# Load unpacked in Chromium developer mode
```

## Project Structure

- `src/` — Source code (JS, CSS, HTML, images)
- `platform/` — Browser-specific builds (Chromium, Firefox, MV3)
- `dist/` — Build output
- `tools/` — Build and CI scripts
- `assets/` — Filter list registry

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

GPL-3.0 (inherited from uBlock Origin upstream)

## Attribution

CinderBlock is based on [uBlock Origin](https://github.com/gorhill/uBlock) by Raymond Hill and contributors. uBlock Origin is licensed under the GNU General Public License v3.0.

## Part of

The Cinder Project / Ember Browser
