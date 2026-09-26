# Blakeinstein's Umbrel App Store

A personal [Umbrel Community App Store](https://github.com/getumbrel/umbrel-community-app-store)
for apps not in the official store, or running ahead of it.

## Add this store

In umbrelOS, open **Settings → App Store → Community App Stores** and add:

```
https://github.com/Blakeinstein/umbrel-app-store
```

It then appears as **Blakeinstein App Store**, and its apps install like any
other.

## Apps

| App | ID | Port | Upstream |
| --- | --- | --- | --- |
| SparkyFitness | `blakeinstein-sparkyfitness` | 3019 | [CodeWithCJ/SparkyFitness](https://github.com/CodeWithCJ/SparkyFitness) |

### SparkyFitness

Self-hosted nutrition, exercise, and body metrics tracking — a self-hostable
alternative to MyFitnessPal.

After installing, open the app and create an account. **The first account
created becomes the administrator**, so make yours before sharing it.

umbrelOS handles the rest: a bundled PostgreSQL 18, secrets derived from the
device seed (so sessions and stored two-factor secrets survive restarts and
updates), data under `~/umbrel/app-data/blakeinstein-sparkyfitness/data/`, and
inclusion in umbrelOS backups.

The web UI sits behind your Umbrel login. The API (`/api`, `/health-data`,
`/uploads`, `/mcp`) is exempt, because the SparkyFitness mobile app, Apple
Health and Google Fit sync, and API-key clients cannot send an Umbrel session
cookie; those routes are protected by SparkyFitness's own authentication. To
connect the mobile app, point it at `http://umbrel.local:3019` or your Umbrel's
LAN IP.

Not included: the Garmin integration service. Email, OIDC single sign-on, and
outbound proxy settings are not exposed as install options — use the
[Docker Compose deployment](https://codewithcj.github.io/SparkyFitness/install/docker-compose)
if you need them.

This package is also proposed for the
[official Umbrel App Store](https://github.com/getumbrel/umbrel-apps), where its
app ID is `sparkyfitness` with no prefix. The two are **separate apps** to
umbrelOS, with separate data directories, so switching means moving
`~/umbrel/app-data/<app-id>/data/` across by hand.

## Migrating an existing SparkyFitness instance

Importing a backup means carrying two secrets across, not just the data.
`SPARKY_FITNESS_API_ENCRYPTION_KEY` decrypts provider credentials stored in the
database and `BETTER_AUTH_SECRET` encrypts stored 2FA secrets. Decryption is not
forgiving — a mismatched key raises an error rather than returning empty — so
restore the data with the wrong keys and every stored integration credential
breaks and anyone with 2FA is locked out.

Install the app **first**, then place the secrets and restart.
`~/umbrel/app-data/<app-id>/` is created by the install, so files written there
beforehand can be replaced by the installer. `hooks/pre-start` never overwrites
a file that already exists, so a file written after the install is kept on every
later start.

1. Install the app from this store and let it finish starting.
2. Read the two values out of the old instance's `.env`.
3. Stop the app, write the files, start it again:
   ```sh
   ssh umbrel@umbrel.local
   APP=blakeinstein-sparkyfitness
   D=~/umbrel/app-data/$APP/data/secrets

   umbreld client apps.stop.mutate --appId $APP
   sudo mkdir -p "$D"
   printf '%s' 'OLD_SPARKY_FITNESS_API_ENCRYPTION_KEY' | sudo tee "$D/api_encryption_key" >/dev/null
   printf '%s' 'OLD_BETTER_AUTH_SECRET'                | sudo tee "$D/better_auth_secret" >/dev/null
   sudo chmod 600 "$D"/*
   umbreld client apps.start.mutate --appId $APP
   ```
4. Confirm the server read them from the files rather than generating its own:
   ```sh
   umbreld client apps.logs.query --appId $APP | grep -i secret
   # [Secrets] Loaded secret for SPARKY_FITNESS_API_ENCRYPTION_KEY from file ...
   # [Secrets] Successfully loaded 2 secrets from files.
   ```
   `No secrets loaded from files` means the installed `docker-compose.yml` is an
   older copy without the `_FILE` variables. Remove and re-add this store in
   umbrelOS so it re-pulls, then reinstall.
5. On the old instance take a backup from **Settings → Admin → Backup**, then
   upload that `sparkyfitness_full_backup_*.tar.gz` under the same screen here.
   Restore **wipes the current database** and replaces it with the archive's,
   and restores uploads too, so do it before entering real data.

The database password does not need migrating — restore replays into the new
instance's own database using its own credentials.

Settings that live only in environment variables, such as SMTP and OIDC single
sign-on, are not carried by a backup and are not exposed here. Add them to the
`server` service in `docker-compose.yml` if you need them.

## Adding an app to this store

1. Create a directory named `blakeinstein-<app>`. Community stores require every
   app ID to start with the store ID from `umbrel-app-store.yml`.
2. Add `umbrel-app.yml` and `docker-compose.yml`. The
   [official store's packages](https://github.com/getumbrel/umbrel-apps) are the
   best reference.
3. Set `app_proxy`'s `APP_HOST` to `blakeinstein-<app>_<service>_1`. Container
   names are built from the app ID, so a missing prefix produces a proxy that
   cannot reach the app.
4. Give the app a `port` no other installed app uses, and an `icon:` URL —
   community stores render the icon from the manifest rather than hosting it.
5. Pin every image to a multi-arch manifest-list digest covering `linux/amd64`
   and `linux/arm64`, so the store works on both a PC and a Raspberry Pi.

Validate against the official linter before pushing:

```sh
git clone --depth 1 https://github.com/getumbrel/umbrel-apps.git /tmp/umbrel-apps
cp -r blakeinstein-<app> /tmp/umbrel-apps/
cd /tmp/umbrel-apps && npm install
npm run lint:apps -- blakeinstein-<app> --check-images
```
