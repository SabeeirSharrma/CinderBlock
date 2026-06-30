## Description

**CinderBlock**, an efficient [MV3 API-based](https://developer.chrome.com/docs/extensions/mv3/intro/) content blocker for Ember Browser.

CinderBlock is entirely declarative, meaning there is no need for a permanent CinderBlock process for the filtering to occur, and CSS/JS injection-based content filtering is [performed reliably](https://developer.chrome.com/docs/extensions/reference/scripting/#method-registerContentScripts) by the browser itself rather than by the extension. This means that CinderBlock itself does not consume CPU/memory resources while content blocking is ongoing — CinderBlock's service worker process is required _only_ when you interact with the popup panel or the option pages.

The default ruleset corresponds to at least uBlock Origin's default filterset:

- uBlock Origin's built-in filter lists
- EasyList
- EasyPrivacy
- Peter Lowe's Ad and tracking server list
- AdGuard Base, Tracking Protection, and Annoyances
- HAGEZI Multi Pro
- StevenBlack Adware + Malware
- Dan Pollock's hosts
- oisd big

You can add more rulesets by visiting the options page — click the _Cogs_ icon in the popup panel.
