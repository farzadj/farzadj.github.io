# Crypto Lab

Crypto Lab is a native Android crypto-analysis app backed by a Python API on Google Cloud Run.

It is built around one workflow:

1. inspect the live chart
2. tune detector settings
3. scan for the best setup
4. validate with a historical check
5. monitor with backend alert rules
6. reuse presets across workflows

## Product Overview

Crypto Lab combines:

- a native Android chart workspace
- server-side signal detection and market-data caching
- a scanner that ranks one best actionable setup
- a historical-check workflow for deterministic strategy review
- backend alert rules with Firebase push delivery
- saved presets shared across Settings, scanner, alerts, and historical check

The app does not call the market-data provider directly. The Android client talks only to the Crypto Lab backend.

## Current Coverage

### Supported markets

- `BTC/USDT` - Bitcoin
- `ETH/USDT` - Ethereum
- `BNB/USDT` - BNB
- `SOL/USDT` - Solana
- `XRP/USDT` - XRP
- `ADA/USDT` - Cardano

### Supported timeframes

- `15m`
- `1h`
- `4h`
- `1d`

### Strategy families

- Order Block Zones
- Chart Patterns
- Break & Retest
- Fair Value Gap
- Liquidity Sweep
- Range Breakout

## Main App Areas

- `Chart`
  - native candlestick chart
  - signal overlays and trade-plan levels
  - best-setup and scanner handoff to chart
- `Data`
  - market and timeframe selection
  - live feed context control
- `Settings`
  - detector parameters
  - advanced per-strategy controls
  - saved profile workflow
- `Signals`
  - live signal list
  - proposed setup workflow
  - alert-rule summary
- `Lab`
  - scanner
  - historical check
  - alert monitoring
  - saved presets / reuse
- `Help`
  - strategy guide
  - product and workflow explanation

## Free vs Pro

Free keeps the core chart workflow available.

Pro expands the workflow with:

- broader market and timeframe access
- the premium strategy pack
- scanner scope across multiple chart contexts
- backend alert rules
- saved presets and reuse
- best-setup ranking across strategies

Exact entitlement behavior is implemented jointly in the Android client and backend subscription stack.

## Architecture

```text
Android app
  -> Crypto Lab backend
    -> Redis / cache
    -> market-data provider
    -> signal engine
    -> alert evaluator
Cloud Scheduler
  -> backend internal refresh / alert evaluation endpoints
Google Play
  -> backend subscription verification / RTDN handling
Firebase Cloud Messaging
  -> Android push delivery
```

## Repository Layout

- [`android-app/`](android-app/)
  - native Android client
  - Kotlin, Fragments, shared `TradingViewModel`
  - chart UI, settings, scanner, lab workflows, billing, alerts
- [`backend/`](backend/)
  - Python API service
  - market-data fetch, caching, detection, scanner jobs, backtesting, alerts, billing
- [`index.html`](index.html)
  - Crypto Lab public landing page
- [`privacy.html`](privacy.html)
  - short redirect page for the privacy policy route
- [`privacy-policy.html`](privacy-policy.html)
  - hosted privacy-policy page template
- [`crypto-lab-logo.png`](crypto-lab-logo.png)
  - public logo asset for website and GitHub Pages use

## Public Site

The static website files in the repo root are ready to be copied into a GitHub Pages repo.

Recommended layout inside `farzadj.github.io`:

- `crypto-lab/index.html`
- `crypto-lab/privacy.html`
- `crypto-lab/privacy-policy.html`
- `crypto-lab/crypto-lab-logo.png`

That gives you:

- `https://farzadj.github.io/crypto-lab/`
- `https://farzadj.github.io/crypto-lab/privacy-policy.html`

## Getting Started

### Android app

Open the Android module in Android Studio and use the module README for setup:

- [`android-app/README.md`](android-app/README.md)

Useful release and store docs:

- [`android-app/docs/PLAY_RELEASE_CHECKLIST.md`](android-app/docs/PLAY_RELEASE_CHECKLIST.md)
- [`android-app/docs/PLAY_STORE_LISTING.md`](android-app/docs/PLAY_STORE_LISTING.md)
- [`android-app/docs/DATA_SAFETY_DRAFT.md`](android-app/docs/DATA_SAFETY_DRAFT.md)
- [`android-app/docs/SCREENSHOT_CHECKLIST.md`](android-app/docs/SCREENSHOT_CHECKLIST.md)
- [`android-app/docs/MONETIZATION_PLAN.md`](android-app/docs/MONETIZATION_PLAN.md)

### Backend

Use the backend README for local setup and service details:

- [`backend/README.md`](backend/README.md)

Infrastructure and operations docs:

- [`backend/docs/CLOUD_RUN_SETUP.md`](backend/docs/CLOUD_RUN_SETUP.md)
- [`backend/docs/OPERATIONS.md`](backend/docs/OPERATIONS.md)
- [`backend/docs/ALERTS_ARCHITECTURE.md`](backend/docs/ALERTS_ARCHITECTURE.md)
- [`backend/docs/SUBSCRIPTION_ARCHITECTURE.md`](backend/docs/SUBSCRIPTION_ARCHITECTURE.md)
- [`backend/docs/SECURITY_HARDENING.md`](backend/docs/SECURITY_HARDENING.md)
- [`backend/docs/NEXT_STEPS.md`](backend/docs/NEXT_STEPS.md)

## Operational Notes

- Android local secrets and signing material should stay out of Git.
- Backend secrets should stay in `.env` locally and Secret Manager in production.
- Build caches such as `.gradle-local/` are local machine state and should never be committed.

The root [`.gitignore`](.gitignore) blocks common local, generated, and secret-bearing files. Keep it aligned with your actual local tooling.

## Status

This repository contains both the Android product surface and the production backend used for live charting, scanning, historical validation, alerts, presets, entitlement checks, and subscription-aware workflow gating for Crypto Lab.
