# CINDERBLOCK_SPEC.md
# CinderBlock — Content Blocking Engine
# The Cinder Project

**Status:** Planning
**Version:** 0.1-spec
**Maintainer:** The Cinder Project
**Upstream:** uBlock Origin (https://github.com/gorhill/uBlock)
**Parent project:** Ember Browser

---

## 1. What CinderBlock Is

CinderBlock is Ember's built-in content blocking engine, forked from uBlock Origin. It is not an extension — it is a first-class component of Ember, compiled as part of the browser and deeply integrated into the Ember Shell.

CinderBlock inherits uBlock Origin's core architecture: a highly efficient network request filter, a cosmetic filter engine, a scriptlet injection system, and a full dashboard UI. On top of this, CinderBlock ships with a curated default filter list set, removes all external service dependencies, and integrates with Ember's config and update systems.

CinderBlock is not distributed separately. It has no independent update channel. It ships with Ember and updates with Ember via CPAC. Filter lists are the only thing that update independently, fetched as plaintext from their source URLs on a configurable schedule.

---

## 2. Fork Rationale

uBlock Origin is the gold standard for content blocking. The fork exists for the following reasons:

| Reason | Detail |
|---|---|
| **Auditability** | CinderBlock is compiled as part of Ember's build pipeline. No pre-built extension binary. |
| **No external update channel** | uBlock has its own update mechanism. CinderBlock removes this entirely — Ember/CPAC owns updates. |
| **No cloud sync** | uBlock supports cloud backup of settings. Removed — settings live in `ember.toml` and local storage only. |
| **No extension store dependency** | uBlock is distributed via browser extension stores. CinderBlock has no store dependency. |
| **Ember-native integration** | CinderBlock reads from and writes to Ember's config system. The dashboard is themed to match Ember Shell. |
| **Curated defaults** | CinderBlock ships with a specific default list set tuned for Ember. uBlock's defaults are more conservative. |

### 2.1 Fork Network

CinderBlock **keeps its fork network connected to upstream uBlock Origin**. It does not detach into a fully independent repository.

The rationale: the changes CinderBlock makes (removing cloud sync, update pings, extension packaging, branding) are all peripheral to the core engine. The filter matching, cosmetic filtering, and scriptlet injection code is kept untouched. This means upstream changes almost never conflict with CinderBlock's modifications.

Staying connected means CinderBlock inherits engine improvements and security patches from uBlock Origin automatically. Detaching would mean manually backporting those fixes forever — a significant and unnecessary maintenance burden.

Merge strategy: when syncing from upstream, review changes against CinderBlock's patch surface. Engine and filter changes merge cleanly. Any upstream changes to removed components (sync, update mechanism, manifest) are discarded.

---

## 3. Architecture

### 3.1 Components Inherited from uBlock Origin

```
┌──────────────────────────────────────────────────┐
│                  CinderBlock                     │
│                                                  │
│  ┌────────────────────────────────────────────┐  │
│  │           Network Request Filter           │  │
│  │  Intercepts all requests, matches against  │  │
│  │  compiled filter lists, blocks or allows   │  │
│  └────────────────────────────────────────────┘  │
│                                                  │
│  ┌────────────────────────────────────────────┐  │
│  │         Cosmetic Filter Engine             │  │
│  │  CSS-based element hiding (##filters)      │  │
│  │  Removes ad placeholders, cookie banners   │  │
│  └────────────────────────────────────────────┘  │
│                                                  │
│  ┌────────────────────────────────────────────┐  │
│  │         Scriptlet Injection Engine         │  │
│  │  Injects JS snippets to neutralize         │  │
│  │  anti-adblock, fix breakage, remove        │  │
│  │  tracking calls that CSS can't reach       │  │
│  └────────────────────────────────────────────┘  │
│                                                  │
│  ┌──────────────┐  ┌──────────────────────────┐  │
│  │   Dashboard  │  │     Element Picker       │  │
│  │   (WebUI)    │  │  Point-and-click block   │  │
│  └──────────────┘  └──────────────────────────┘  │
│                                                  │
│  ┌──────────────┐  ┌──────────────────────────┐  │
│  │   Network    │  │    Dynamic Filtering     │  │
│  │   Logger     │  │    (per-domain matrix)   │  │
│  └──────────────┘  └──────────────────────────┘  │
└──────────────────────────────────────────────────┘
```

### 3.2 What Is Removed from uBlock Origin

- Extension store manifest and packaging (`manifest.json` as a standalone extension)
- Self-update mechanism (AMO/CWS update checks)
- Cloud backup / sync (browser.storage.sync usage)
- uBlock Origin branding (name, icon, about page)
- Default filter list set (replaced with CinderBlock's own defaults — see Section 5)
- Links to uBlock Origin's GitHub, wiki, and support forums in the dashboard

### 3.3 What Is Added

- Ember Shell integration (dashboard themed to match Ember's active theme)
- Config bridge: CinderBlock reads `[blocking]` section from `ember.toml` on launch
- Ember-native toolbar button (replaces extension popup with native Shell component)
- CinderBlock branding (name, icon, about page crediting upstream uBlock Origin)

---

## 4. Getting Started — Fork Setup & Strip-Down

The very first thing done after forking uBlock Origin is stripping it down to the clean core. No features are added until the strip is complete and the build is verified. This section defines exactly what to do and in what order.

### Step 1 — Fork & Clone

Fork `gorhill/uBlock` into The Cinder Project org as `cinderblock`. Clone locally:

```bash
git clone https://github.com/SabeeirSharrma/CinderBlock
cd CinderBlock
git remote add upstream https://github.com/gorhill/uBlock
```

### Step 2 — Remove Extension Packaging

uBlock ships as a browser extension with a `manifest.json`. CinderBlock is not an extension — remove all extension store packaging:

- Delete `manifest.json` (MV2 and MV3 variants)
- Delete `_locales/` directory (extension i18n format — will be replaced with Ember's i18n system later)
- Delete `web_accessible_resources/` extension manifest entries
- Remove any references to `browser.runtime.id`, `chrome.runtime.id` used for store identity

### Step 3 — Remove the Self-Update Mechanism

uBlock checks for updates via AMO (Firefox) and CWS (Chrome). Remove entirely:

- Delete `src/js/selfie.js` update-check logic
- Remove all `browser.runtime.onUpdateAvailable` listeners
- Remove all references to `browser.runtime.requestUpdateCheck`
- Remove update notification UI from the dashboard
- Search codebase for `updateAvailable`, `onUpdateAvailable`, `checkForUpdate` — remove all hits

### Step 4 — Remove Cloud Sync

uBlock uses `browser.storage.sync` to back up settings to the browser's cloud account. Remove entirely:

- Replace all `browser.storage.sync` calls with `browser.storage.local`
- Delete cloud backup/restore UI from the dashboard settings page
- Remove sync conflict resolution logic
- Search codebase for `storage.sync` — every hit should be removed or replaced with `storage.local`

### Step 5 — Remove External Service Dependencies

uBlock phones home in a few places. Remove all of them:

- Remove uBlock's own filter list update URLs from the default list registry (will be replaced in Step 7)
- Remove any analytics or telemetry (search for `telemetry`, `analytics`, `ping`)
- Remove links to `github.com/gorhill/uBlock`, the uBlock wiki, and the uBlock support forum from all UI pages
- Remove the "Support uBlock Origin" / donation prompts

### Step 6 — Remove uBlock Branding

- Replace the uBlock Origin name with CinderBlock everywhere (JS, HTML, CSS, comments)
- Replace the uBlock icon set with CinderBlock icons (placeholder icons acceptable at this stage)
- Update the about page (`dashboard/about.html`) with CinderBlock name and upstream attribution (see Section 7)

### Step 7 — Replace Default Filter Lists

uBlock ships with its own default list selection. Replace with CinderBlock's defaults:

- Open the filter list registry file (`assets/assets.json` or equivalent)
- Remove uBlock's default list entries
- Add CinderBlock's full default list set (see Section 5.1) with correct source URLs
- Set gambling and adult content lists as disabled by default
- Verify all source URLs are reachable

### Step 8 — Verify the Build

At this point the strip is complete. Verify:

```bash
# Install dependencies
npm install

# Run the build
npm run build

# Confirm the following are gone:
# - No storage.sync references
# - No update check network calls
# - No AMO/CWS manifest artifacts
# - uBlock branding fully replaced
```

Load the unpacked extension in Chromium's developer mode and confirm the dashboard opens, filter lists load, and basic blocking works on a test page.

Only after this verification passes does any new CinderBlock-specific work begin.

---

## 5. Features

### 5.1 Network Request Filtering

Intercepts all HTTP/HTTPS requests made by the browser. Each request is matched against compiled filter lists using uBlock's efficient trie-based matching algorithm. Blocked requests are cancelled before they reach the network.

Supports all standard filter syntax:
- Basic URL pattern matching (`||ads.example.com^`)
- Domain anchors, path filters, query string filters
- Exception rules (`@@`)
- Option modifiers (`$third-party`, `$script`, `$image`, `$xhr`, etc.)
- Hosts file format (for StevenBlack, Dan Pollock, etc.)

### 5.2 Cosmetic Filtering

CSS-based element hiding applied after page load. Removes:
- Ad containers and placeholders
- Cookie consent banners
- Newsletter popups
- Notification permission prompts
- Chat widgets and floating overlays
- "Subscribe to continue reading" overlays

Supports:
- Generic cosmetic filters (`##.ad-banner`)
- Domain-specific filters (`example.com##.sponsored`)
- Extended CSS selectors (`:has()`, `:not()`, `:matches-css()`)
- Procedural cosmetic filters

### 5.3 Scriptlet Injection

JavaScript snippets injected into page context to neutralize behaviors that CSS cannot reach:
- Anti-adblock detection scripts
- Tracking calls embedded in page JS
- Cookie consent wall JS
- Fingerprinting scripts

### 5.4 Dashboard

Full CinderBlock dashboard accessible via:
- Toolbar button
- Keyboard shortcut (default: `Ctrl+Shift+B`, remappable in `ember.toml`)
- `ember://cinderblock` URL

Dashboard sections:

| Section | Description |
|---|---|
| **Overview** | Total requests blocked, bandwidth saved, per-session stats |
| **Filter Lists** | Enable/disable lists, update individual lists, add custom list URLs, view list metadata |
| **My Filters** | User-authored custom filter rules in plaintext |
| **My Rules** | Per-site allow/block rules, whitelist management |
| **Trusted Sites** | Domains where CinderBlock is fully disabled |
| **Advanced** | Dynamic filtering matrix, raw filter list stats |

### 5.5 Element Picker

Activated via the toolbar button or `Ctrl+Shift+E` (remappable). Allows the user to:
- Click any element on a page to generate a cosmetic filter rule for it
- Preview what will be hidden before committing
- Edit the generated rule before saving
- Save the rule to My Filters

### 5.6 Network Logger

Real-time log of all network requests made by the current tab, accessible via the dashboard. Each entry shows:
- Request URL
- Request type (script, image, XHR, frame, etc.)
- Origin domain
- Block/allow status
- Which filter rule matched (if blocked)

The logger is off by default (performance) and toggled on in the dashboard. It does not persist across sessions.

### 5.7 Dynamic Filtering

Per-domain request type matrix for advanced users. Allows fine-grained control such as:
- Block all third-party scripts globally
- Allow third-party scripts on specific trusted domains
- Block all third-party frames except on listed domains

Presented as a color-coded matrix in the dashboard (inherited from uBlock Origin's advanced mode).

### 5.8 Per-Site Rules

Users can define rules scoped to specific domains:
- Disable CinderBlock entirely for a domain (whitelist)
- Disable cosmetic filtering only for a domain
- Disable scriptlet injection only for a domain
- Custom filter rules scoped to a domain

Per-site rules are stored locally and are part of the CinderBlock config export.

---

## 6. Filter Lists

### 6.1 Default List Set

| List | Category | Default | Source |
|---|---|---|---|
| AdGuard Base | Ads | ✅ on | `https://filters.adtidy.org/extension/ublock/filters/2.txt` |
| AdGuard Tracking Protection | Tracking | ✅ on | `https://filters.adtidy.org/extension/ublock/filters/3.txt` |
| AdGuard Annoyances | Cookie banners, popups | ✅ on | `https://filters.adtidy.org/extension/ublock/filters/14.txt` |
| EasyList | Ads | ✅ on | `https://easylist.to/easylist/easylist.txt` |
| EasyPrivacy | Tracking | ✅ on | `https://easylist.to/easylist/easyprivacy.txt` |
| Peter Lowe's Ad and Tracking | Ads + Tracking | ✅ on | `https://pgl.yoyo.org/adservers/serverlist.php?hostformat=adblockplus` |
| uBlock Origin Filters | Ads | ✅ on | `https://raw.githubusercontent.com/uBlockOrigin/uAssets/master/filters/filters.txt` |
| uBlock Annoyances | Annoyances | ✅ on | `https://raw.githubusercontent.com/uBlockOrigin/uAssets/master/filters/annoyances.txt` |
| Dan Pollock's hosts | Malware, spam | ✅ on | `https://someonewhocares.org/hosts/zero/hosts` |
| PhishTank | Phishing domains | ✅ on | `https://phishtank.org/phishtank.txt` |
| HAGEZI Multi Pro | Ads + Tracking + Malware | ✅ on | `https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/pro.txt` |
| StevenBlack Adware + Malware | Adware, malware | ✅ on | `https://raw.githubusercontent.com/StevenBlack/hosts/master/alternates/adware-malware/hosts` |
| oisd big | Comprehensive | ✅ on | `https://big.oisd.nl` |
| StevenBlack Gambling | Gambling domains | ⬜ off | `https://raw.githubusercontent.com/StevenBlack/hosts/master/alternates/gambling/hosts` |
| Adult content | Adult content | ⬜ off | `https://raw.githubusercontent.com/StevenBlack/hosts/master/alternates/porn/hosts` |

### 6.2 Custom Lists

Users can add any custom filter list via URL from the dashboard. Requirements:
- Must be a direct URL to a plaintext file
- Must use uBlock Origin / AdGuard compatible syntax, or hosts file format
- No authentication required (public URL)

Custom lists are stored by URL reference, not by content. They are fetched and compiled on the same update schedule as built-in lists.

### 6.3 List Update Behavior

- Default update interval: every **24 hours**
- Configurable in `ember.toml` under `[blocking]`:

```toml
[blocking]
update_interval_hours = 24   # set to 0 to disable auto-update
```

- Updates are plaintext HTTP GET requests to the list source URL
- No binary downloads, no signed packages, fully transparent
- Lists are compiled into an efficient internal format after fetch
- Manual update available per-list from the dashboard
- CinderBlock itself does **not** auto-update — it ships with Ember and updates via CPAC

---

## 7. Configuration Integration

CinderBlock reads its initial state from the `[blocking]` section of `ember.toml`:

```toml
[blocking]
enabled = true
update_interval_hours = 24

# List overrides (optional — defaults apply if not specified)
[blocking.lists]
adguard_annoyances = true
stevenblack_gambling = false
adult_content = false
```

Per-site rules, custom filters, and dynamic filtering rules are stored separately in:

```
~/.config/ember/cinderblock/
  rules.txt          # My Filters (user-authored rules)
  per-site.json      # Per-site rule overrides
  dynamic.json       # Dynamic filtering matrix
  trusted.txt        # Whitelisted domains
```

These files are human-readable and version-controllable.

---

## 8. Branding & Attribution

CinderBlock is a fork of uBlock Origin. The following attribution is displayed on the CinderBlock about page (`ember://cinderblock/about`):

> CinderBlock is based on uBlock Origin by Raymond Hill and contributors.
> uBlock Origin is licensed under the GNU General Public License v3.0.
> Source: https://github.com/gorhill/uBlock

CinderBlock is licensed under **GPL-3.0** in compliance with uBlock Origin's license. This applies to the CinderBlock component only — Ember's other components may use a different license.

---

## 9. Versioned Roadmap

CinderBlock versioning follows Ember's release cycle. CinderBlock does not have independent releases.

### v0.3 (ships with Ember v0.3)
- Fork established from uBlock Origin upstream
- All Google/AMO/CWS dependencies removed
- Cloud sync removed
- Default filter list set bundled and active
- Opt-in lists available (gambling, adult)
- Custom list URL support
- 24-hour auto-update for lists
- Basic dashboard (filter lists, my filters, trusted sites)
- Ember Shell toolbar button integration

### v0.7 (ships with Ember v0.7)
- Element picker
- Network logger
- Dynamic filtering matrix
- Per-site rules UI in dashboard
- Full dashboard (all sections)
- Ember theme integration (dashboard matches active Ember theme)

### v1.0 (ships with Ember v1.0)
- Stable, fully documented
- CinderBlock config reference in Ember docs
- All features from uBlock Origin advanced mode implemented

---

## 10. Future Considerations

- **CinderBlock rule sharing** — community-maintained per-site rule sets installable via CPAC (e.g. `cpac install cinderblock-rules-youtube`)
- **DNS-level blocking integration** — CinderBlock rules optionally synced to Ember's DoH resolver for a second layer of blocking
- **Declarative Net Request migration** — Chromium is pushing toward MV3's DNR API; CinderBlock should track this and evaluate impact on dynamic filtering capabilities
- **Custom scriptlet library** — user-contributed scriptlets installable via CPAC, audited before publishing

---

## 11. Project Info

**Repository:** https://github.com/SabeeirSharrma/CinderBlock (to be transferred to The Cinder Project org when created)
**License:** GPL-3.0 (inherited from uBlock Origin upstream)
**Upstream:** uBlock Origin — https://github.com/gorhill/uBlock
**Part of:** The Cinder Project / Ember Browser