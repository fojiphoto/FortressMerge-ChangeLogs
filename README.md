# FortressMerge-ChangeLogs
# Fortress Merge — WebGL Platform Changelog

Full history of all WebGL platform work across branches.
Last updated: April 2026

---

## PHASE 1 — WebGL Foundation
### Branch: `WebGL-Jan2026` | December 2025 – January 2026

> Goal: Strip the mobile build down to a clean WebGL prototype — reduce bundle size, fix platform-specific systems, prepare for web publishing.

- Branched off from mobile `0.3.0` build as the first dedicated WebGL branch
- **Jan 05** — Initial WebGL size optimization pass — removed unnecessary native plugins and unused assets
- **Jan 06** — Fixed loading issue blocking the game from starting on WebGL
- **Jan 09** — Migrated entire save system from file system to `PlayerPrefs` (WebGL has no file system access)
- **Jan 12** — Removed remaining extras and unused packages leftover from the mobile build
- **Jan 19** — Sprite compression pass applied across all textures to reduce download bundle size

---

## PHASE 2 — Build Stabilisation & Platform Prep
### Branch: `WebGL-Jan2026` / `Pley_FirebaseSDK-Test-March26` | January – February 2026

> Goal: Get a stable deployable WebGL build — fix UI orientation, add rewarded ads, resolve skill bugs, reduce size further.

- **Jan 27** — Further bundle size reduction; re-added missing assets that were over-stripped
- **Jan 28** — Full trace logging build generated for platform debugging session
- **Jan 29** — Fixed broken skills system on WebGL; UI setup completed; settings build configuration updated
- **Feb 02** — Removed `JetBrains.Annotations` package; additional build size reduction pass
- **Feb 03** — Fixed store screen orientation issue; rewarded video ads integrated for the first time on WebGL; misc platform stability fixes
- **Feb 04** — Added Xsolla plugin for web payments; non-English fonts added; UI orientation improvements across all screen sizes

---

## PHASE 3 — Pley Platform Integration (3 Weeks of R&D)
### Branch: `Pley_FirebaseSDK-Test-March26` | February – March 2026

> Goal: Enable mobile players (Android/iOS) to sync their existing game save data to the Pley web build using a cloud-backed identity system — so a player's progress follows them across platforms.

### Week 1 — Pley SDK Investigation (Feb 04 – ~Feb 11)
- **Feb 04** — Started Pley platform integration immediately after the base WebGL build was stable
- Integrated the Pley SDK and began reading through their documentation for data persistence support
- Discovered the Pley SDK documentation was incomplete and unclear regarding Firestore usage — no official working example was provided for game save sync
- Opened support tickets with the Pley team; responses were slow and did not address the Firestore-specific questions
- Identified the core requirement: players on Android/iOS have existing save data (coins, level, heroes, perks) — the web build must load that same data so they do not start from scratch

### Week 2 — Firebase + Firestore External Testing (~Feb 11 – ~Feb 20)
- Pivoted away from the Pley SDK approach and began integrating Firebase + Firestore directly into the WebGL build as an independent data layer
- Implemented Firestore read/write for game save data — player profile, level progress, currency, unlocks
- Hosted test builds externally on GitHub Pages to run end-to-end data sync validation outside of the Pley environment
- Confirmed Firestore sync was working correctly in isolation — data written from a mobile build was readable by the web build
- Hit a platform blocker: Pley loads the game inside an **iframe**, and their internal authentication plugin does not properly support **Google ID Sign-in** or **Firestore** — only standard Firebase Analytics and crash logging are supported inside the iframe context
- Google Sign-in OAuth redirects are blocked by the iframe sandbox, making the standard Firebase Auth flow non-functional on Pley

### Week 3 — Pley Team Collaboration & Final Solution (~Feb 20 – Mar 12)
- Escalated the iframe/auth issue directly to the Pley engineering team and entered a collaborative debugging phase
- Shared build logs, iframe console errors, and OAuth failure traces with Pley support over multiple sessions
- Pley team investigated their internal plugin limitations and confirmed that standard Google Sign-in cannot be used inside their iframe environment
- **Mar 12** — Pley team provided an official solution: use an **external token-based authentication flow** — the game requests an auth token from outside the iframe, passes it into the game context, and uses that token to authenticate the Google ID and load the player's Firestore data
- Clean build produced implementing this external token approach and submitted to Pley for review
- **Mar 24** — External token Google Sign-in auth and Firestore data retrieval fully validated — player data syncs correctly from mobile to web on the Pley platform

---

## PHASE 4 — CrazyGames Launch
### Branch: `Crazy-April26` | March – April 2026

> Goal: Integrate CrazyGames SDK, pass CrazyGames QA, submit and iterate builds until platform approval.

- **Mar 31** — First CrazyGames build submitted — CrazyGames SDK integrated, ads wired up
- **Apr 03** — First Poki build uploaded in parallel (initial `Poki-April26` branch)
- **Apr 06** — CrazyGames reviewer feedback addressed — UI, ad timing, and UX changes applied
- **Apr 08** — Second build submitted to CrazyGames with all feedback changes
- **Apr 13** — New build sent with additional issues fixed from reviewer round 2
- **Apr 14** — Language/localisation changes resolved — translation auto-conversion bug fixed
- **Apr 15** — Internal QA pass completed; full external QA sign-off received
- **Apr 16** — Third build uploaded to CrazyGames platform
- **Apr 17** — [BUG FIX] Game Music silent on first launch — mixer parameter corrected from -80 dB to 0 dB; `AudioSettingsBootstrapper` created to guarantee audio initialises regardless of which scene loads first
- **Apr 17** — [BUG FIX] `DontDestroyOnLoad` console warning — added `transform.SetParent(null)` guard in `GameManager` before `DontDestroyOnLoad`
- **Apr 20** — Final issues addressed — CrazyGames build complete and approved

---

## PHASE 5 — Deferred Font System
### Applied to: `Crazy-April26` + `PokiNew-April26` | April 2026

> Goal: Reduce initial download size for all WebGL platforms by streaming non-English fonts on demand instead of bundling them upfront.

- Created `DeferredFontLoader.cs` with two loading paths:
  - Bootstrap path — returning non-English players load fonts during the loading screen (blocking, invisible)
  - Level-1 background path — first-time players stream fonts from CDN during Level 1, complete before fonts are needed
- Updated `LocalizationData.cs` — added runtime font slots, `AreFontsLoaded` property, and 3-level font fallback
- Updated `GameSettings.cs` — injects deferred fonts when Localization addressable finishes loading
- Updated `GameSceneManager.cs` — calls `DeferredFontLoader.StartBackgroundLoad()` on init
- [BUG FIX] Bootstrap "stuck on Finalising" — returning non-English players were stuck on the loading screen; font loading now correctly awaited before bootstrap signals completion

---

## PHASE 6 — Full Poki SDK Integration + IAP Removal
### Branch: `PokiNew-April26` | April 2026

> Goal: Replace the CrazyGames SDK entirely with the Poki SDK, remove all real-money IAP flows, and configure ad delivery to meet Poki platform requirements.

### Core SDK Swap
- Removed — Entire `CrazySDK/` package, `CrazyGamesIntegration.cs`, and `crazySDK.jslib` deleted
- Created — `PokiIntegration.cs` — full singleton replacing `CrazyGamesIntegration`
- Created — `PokiSDKBridge.jslib` — JS bridge exposing `gameplayStart`, `gameplayStop`, `commercialBreak`, `rewardedBreak` to C#
- Created — `poki-template/` — custom WebGL template (`index.html`, `index.json`) configured for Poki embedding
- Updated — `SaveSystem.cs` — all `CrazySDK.Data.*` calls replaced with `PlayerPrefs.*`
- Updated — `AppBootstrap.cs` — Poki SDK init wait replaces CrazyGames SDK init wait
- Updated — `ProfilePopUp.cs` — removed CrazyGames username fetch; uses local player name
- Updated — `SettingsPopUp.cs` — `GameEnd/GameStart` signals ported to PokiIntegration
- Updated — `Analytics.cs` — `GameEnd/GameStart` signals ported to PokiIntegration

### Ad System — Poki Requirements
- Updated — `ISManager.cs` — commercial breaks now call `PokiIntegration.ShowMidgameAd`; `GameManager.PauseTime(true/false)` wraps every ad so DOTween timers pause correctly
- Updated — `RVManager.cs` — rewarded video ported to `PokiIntegration`; `CanShowRV` includes `IsRewardedVideoAvailable` check
- Updated — `GameSceneManager.cs` — `ISType.EndLevel` commercial break added to `OnGameEnded()` — fires on both level complete AND level fail
- Configured — `WebGLRemoteConfigInitializer.cs` — ad config updated for Poki:
  - Cold-start buffer: 540 s → 60 s
  - Starts from level: 8 → 2
  - Cooldown between breaks: 3 minutes (new key)
  - Cooldown after rewarded: 5 min → 3 min
  - End-level break: off → on
  - Perks screen break: on → off

### IAP Complete Removal
- Updated — `IAPManager.cs` — `PurchaseProduct()` is a no-op; no real-money purchase ever initiates
- Updated — `RealMoneyCost.cs` — `Pay()` is a no-op; `OnSuccess` never called — ultimate backstop
- Updated — `PackPopUpController.cs` — `Open()` and `BuyPack()` are no-ops; all external callers silently do nothing
- Updated — `UnderTakerPackPopUpController.cs` — popup hides itself on open; `BuyPack()` is a no-op
- Updated — `UnderTakerDiscountedPackPopUpController.cs` — same treatment as UnderTaker pack popup
- Updated — `GooglePlayPassHandler.cs` — `BuyGooglePlayPass()` is a no-op
- Updated — `FirstRVPopUpController.cs` — `ShowPopUp()` always returns false; "buy SkipIt" gate bypassed; rewarded video starts immediately
- Updated — `FirstISPopUpController.cs` — "See No Ads Offer" button is a no-op; watch-ad button goes directly to Poki commercial break
- Updated — `NoAdsShopItemDisplay.cs` — entire No Ads shop item hidden and deactivated
- Updated — `ShopManager.cs` — `NoAds` and `SpecialOffers` categories skipped entirely; never spawned for Poki users
- Updated — `ShopItemDisplay.cs` — any item with `IsRealMoney = true` immediately hidden in `Init()`; `firstTimePurchaseIcon` always disabled
- Updated — `IAPPackNotBoughtCondition.cs` — `IsUnlocked()` always returns false; all IAP-prompt menu icons permanently hidden

---

## Summary

| Period         | Branch                        | Milestone                                    |
|----------------|-------------------------------|----------------------------------------------|
| Jan 2026       | WebGL-Jan2026                 | WebGL foundation — save system, size, loading |
| Jan–Feb 2026   | WebGL-Jan2026                 | Ads, UI orientation, Xsolla, font base        |
| Feb–Mar 2026   | Pley_FirebaseSDK-Test-March26 | Pley platform — 3 weeks R&D, Firebase, Firestore, external token auth |
| Mar–Apr 2026   | Crazy-April26                 | CrazyGames — SDK, QA, 3 build rounds          |
| Apr 2026       | Crazy-April26                 | Music fix, DDOL fix, font streaming system    |
| Apr 2026       | PokiNew-April26               | Poki SDK — full integration, IAP removal, ads |
