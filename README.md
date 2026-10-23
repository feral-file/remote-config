# remote-config

Settings the Feral File app and every FF1 download while they run. Nothing here is documentation: words people read live in `feral-file/ff3` (feralfile.com, the app's license agreement and privacy policy included) and `feral-file/docs` (docs.feralfile.com).

## What is here and who reads it

| Path | What it is | Read by |
|------|------------|---------|
| `configs/ff-app.json` | The app's remote config: feed operators, feature flags, related settings | The app, from the Cloudflare Pages host (`https://ff-app-docs.pages.dev/ff-app.json`) |
| `configs/daily-recovery-2026-09-10.json` | Daily's temporary playlist (below) | The app, through `ff-app.json` |
| `configs/display.json` | The FF1 player's default playlist and its duration | Every FF1, at `https://raw.githubusercontent.com/feral-file/remote-config/main/configs/display.json`, once FF OS builds carry that address (`PUB_DOC_URL`, a variable in `feral-file/ffos`). Until then FF1s read the copy in `feral-file/feral-file-docs`, and an edit goes to both. |
| `configs/app/` | `versions.json`, `version_compatibility.json` (which app and FF OS versions work together), `apple_model_identifier.json` | The app's version and FF1-compatibility checks, from the first release that reads them here. App 1.8.33 and older read the copies on `feral-file/feral-file-docs`' `master` branch, and an edit goes to both. |
| `agreements/` | The app's license agreement and privacy policy | App 1.8.33 and older. The text is edited in `feral-file/ff3` (`data/content/legal/app-eula.md`, `app-privacy-policy.md`); later releases show those pages, and a change is copied here while the older installs remain. |

Before each deploy, CI parses every JSON file under `configs/` and verifies every signed DP-1
playlist with the pinned `@feralfile/cli` release. It then writes
`configs/version.json` with the Git commit and deployment environment so you can
tell which revision is live.

## Deployment

GitHub Actions deploys the **`configs/`** directory to **Cloudflare Pages** when you push to **`main`** or **`dev`**, or when you run the workflow manually. The Pages project keeps its name, so the host stays `ff-app-docs.pages.dev`.

- **`main`** → Production environment (default for `workflow_dispatch` when not specified inline).
- **`dev`** → Development environment.

### Required repository configuration

Configure these in the GitHub repo (Settings → Secrets and variables → Actions):

| Name | Type | Description |
|------|------|--------------|
| `CLOUDFLARE_API_TOKEN` | Secret | Token with permission to deploy to Cloudflare Pages. |
| `CLOUDFLARE_ACCOUNT_ID` | Variable | Cloudflare account ID. |
| `CLOUDFLARE_PROJECT_NAME` | Variable | Pages project name that serves the remote configs. |

The workflow is defined in `.github/workflows/cloudflare-pages-deploy.yml`.

## Impact on production

A merge to `main` changes what the app and every FF1 read. `ff-app.json` changes live app behavior after clients refresh remote config; `display.json` changes the default playlist on every FF1; `configs/app/` decides which app and FF OS versions are told to update. Coordinate with the mobile team before altering operator lists or behavioral flags.

## Maintenance

- Edit a file in a branch, open a PR, merge when reviewed.
- After deploy, confirm the Pages URL serves `ff-app.json` and optional `version.json` as expected.

## Temporary Daily recovery — September 10, 2026

`configs/ff-app.json` temporarily points Daily at `daily-recovery-2026-09-10.json`. This separately signed recovery copy omits only **Each and Every Command AP** (`ff4ea942-d73d-4355-b370-a6cb9966d183`, scheduled September 10). Its 30.75 MB text preview exhausts Android WebView memory in app 1.8.0. All 76 other entries, artwork metadata, and scheduled dates are preserved: September 10 falls back to **Predictive Art Bot AP**, and September 11 selects **Minos AP** normally.

The original signed playlist is unchanged. The recovery copy has a new identity and is signed by the available operator key; it does not claim the original curator's signature. It is a static snapshot, so subsequent edits to the original feed will not reach Daily while this URL is configured.

After September 10 has ended for app users, or a safe renderer ships, restore `daily.playlist_url` to `https://feed.feralfile.com/api/v1/playlists/feral-file-daily-f77fe04c` and deploy. Keep the recovery JSON available for clients that still have the temporary URL cached. Restoring the config is a separate deployment; it is not scheduled automatically. Clients refresh remote config on cold launch or a Daily refresh.

### Extension — September 15, 2026

The recovery copy is also the only Daily clients read, and it ran out on **October 16**. On 2026-09-15 it was extended in place with 30 entries for **October 17 through November 15** so the URL already cached by clients keeps working. Entries rotate across the feed playlists of the NODE and Refik Anadol Studio channels — `OCCUPY` (`4036c79e-23db-4d86-8f27-21713e32aea5`), `Unsupervised — MoMA — Dreams` (`ebe2f3f2-68e4-4b81-8b0e-f570939c576d`), `Machine Hallucinations — Coral` (`2494954f-40e4-412c-bd45-10e3b51e414f`), `Unsupervised — MoMA — Burned — MoMA Dreams` (`2ed0b439-479d-4e07-8f61-9dcd11b016d9`) — one item per day in playlist order, moving to the next playlist each day and skipping a playlist once it is exhausted. Item bodies are copied verbatim from `https://feed.feralfile.com/api/v1/playlists/<id>`; only `displayAt` is added. The existing 76 entries and their dates are unchanged.

The extended document was signed by the Feral File curator wallet (`did:pkh:eip155:1:0x7D5dA3DAAdf000b9e239e45a106a9F241bDaA8b0`) and published to the feed as `https://feed.feralfile.com/api/v1/playlists/feral-file-daily-temporary-recovery-60647c56` (new playlist id `60647c56-db21-4d14-9496-088e2ae3b48a`, feed co-signature included). At that time, `configs/daily-recovery-2026-09-10.json` contained the document served by that feed URL, kept under the filename clients already have cached. The original feed playlist `feral-file-daily-f77fe04c` was not extended; restoring `daily.playlist_url` to it would drop back to an October 16 end date. The next extension should follow the same step: append entries after the last `displayAt`, sign, publish, copy the served bytes here, verify.

### Re-signing — September 24, 2026

The September 24 artwork CDN URL update changed 76 item sources and invalidated the previous signatures. The edited 106-item playlist was signed in the Publisher and published as `https://feed.feralfile.com/api/v1/playlists/feral-file-daily-temporary-recovery-7f3c0f4e` (playlist id `7f3c0f4e-84d3-4206-ac36-fe073e11fbb9`, curator wallet `did:pkh:eip155:1:0xE02BDdd66c0dbFc132e95620089a02cEddAaF2cf`). The file at `configs/daily-recovery-2026-09-10.json` now contains the signed feed response; its 106 items match the edited version exactly. The static URL used by clients remains the same. For later edits, publish the updated document and copy the signed feed response here before deploying.

### Rebuild and re-signing — October 23, 2026

The 30 entries appended on September 15 (October 17 through November 15) were copied before the feed added manifest thumbnails, so none of them carried a thumbnail or an on-chain provenance and the app rendered "No thumbnail" for every one whose source has no image extension (`feral-file/ff-app#1142`). Each of those entries now takes its body verbatim from the current feed item with the same `source` — `node-occupy-cf499e88`, `refik-moma-dreams-f580d1c1`, `refik-coral-b4d72428`, `refik-moma-burned-fd1d00b9` — keeping only its own `id`, `slug` and `displayAt`. That brings manifest thumbnails to the OCCUPY, Dreams and Coral entries and on-chain provenance to the Burned entries, which the app enriches from the indexer. The other 76 entries are unchanged.

This version was not published to the feed. It was signed fresh (`ff-cli sign -r curator --replace-signatures`, `did:key:z6Mkn7RkgQShT8CTKC6RBEaBt8iWXNgLUREbGgWTSomBhFkD`), which drops the previous curator and feed signatures; CI's `@feralfile/cli` verify accepts it.
