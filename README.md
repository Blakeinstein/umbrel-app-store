# SparkyFitness Umbrel App Store

An [Umbrel Community App Store](https://github.com/getumbrel/umbrel-community-app-store)
that ships [SparkyFitness](https://github.com/CodeWithCJ/SparkyFitness) — a
self-hosted nutrition, exercise, and body metrics tracker.

## Install

1. In umbrelOS, open **Settings → App Store → Community App Stores**.
2. Add this repository's URL:
   ```
   https://github.com/Blakeinstein/sparkyfitness-umbrel-app-store
   ```
3. Install **SparkyFitness** from the store that appears.
4. Open the app and create an account. **The first account created becomes the
   administrator**, so make yours before sharing the app.

## What umbrelOS handles for you

| | |
| --- | --- |
| App URL | `http://umbrel.local:3019` |
| Database | Bundled PostgreSQL 18, no setup |
| Secrets | Derived from the device seed, stable across restarts and updates |
| Data | `~/umbrel/app-data/sparky-sparkyfitness/data/` |
| Backups | Included in umbrelOS backups |

Secrets are derived rather than generated per boot, so sessions and stored
two-factor secrets survive restarts and updates.

## Access

The web UI sits behind your Umbrel login. The API (`/api`, `/health-data`,
`/uploads`, `/mcp`) is exempt, because the SparkyFitness mobile app, Apple
Health and Google Fit sync, and API-key clients cannot send an Umbrel session
cookie. Those routes are protected by SparkyFitness's own authentication.

To connect the mobile app, point it at `http://umbrel.local:3019` or your
Umbrel's LAN IP.

## Relationship to the official App Store

This package is also proposed for the
[official Umbrel App Store](https://github.com/getumbrel/umbrel-apps), where its
app ID is `sparkyfitness`. Community stores must prefix every app ID with the
store ID, so here it is `sparky-sparkyfitness` and the container hostnames in
`docker-compose.yml` carry that prefix.

The two installs are therefore **separate apps** with separate data directories.
Migrating between them means moving `~/umbrel/app-data/<app-id>/data/` across
by hand.

## Limitations

- The Garmin integration service is not included.
- Email, OIDC single sign-on, and outbound proxy settings are not exposed as
  install options. Use the
  [Docker Compose deployment](https://codewithcj.github.io/SparkyFitness/install/docker-compose)
  if you need them.
