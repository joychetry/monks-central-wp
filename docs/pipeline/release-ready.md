# Pipeline step 5: release-ready

Date: 2026-10-29. Plan: monks-central-plugin-plan.md, step 5.

The plugin is ready for its first GitHub release. Nothing is tagged, released or public; the decisions left are
Joy's and are listed in `docs/release/checklist.md`.

## What changed per repo

- **monks-central-wp** (`main`):
  - `bin/build-zip.sh`:
    - `README.md` and `CHANGELOG.md` stay out of the zip, and the build fails if a dev file gets in (dotfiles,
      tests, docs, bin, sources, lock files, maps, `package.json` and the like).
    - `composer.json` stays in, next to `vendor/`, because Plugin Check asks for it.
    - The zip has 128 entries: `monks-central.php`, `uninstall.php`, `readme.txt`, `composer.json`, `src/`,
      `connector/` (without its tests), `templates/`, `build/` and the Composer autoloader.
    - `--wporg` still leaves `src/Updater.php` out.
  - Version 1.0.0 in the header, the constant, readme.txt `Stable tag`, `package.json` and the PHP test
    bootstraps. `tests/mcp/loader.sh` now reads the version instead of pinning 0.1.0.
  - `readme.txt` covers:
    - the description and a five-step installation from GitHub Releases;
    - a FAQ: the app, versions, Classic Monks, what an app can do, disconnecting, deactivate and delete,
      updates;
    - **External services**: only the GitHub update check and download, with what is sent (WordPress adds the
      site address to the User-Agent), when, and GitHub's terms and privacy links. The requests that happen only
      when an app or the user asks are named too: plugin checksums from downloads.wordpress.org, image imports,
      an app's client metadata, the site's own pages;
    - the 1.0.0 changelog.
  - `README.md`:
    - installing from GitHub Releases: the zip asset, not "Source code"; `sha256sum -c`; Upload Plugin or
      WP-CLI; the switch; connecting in the app;
    - how updates are checked, what is inside, and how releasing works.
  - `CHANGELOG.md`: Monks Central 1.0.0 (dated "Unreleased" until Joy picks the day) instead of the starter's
    history.
  - `.github/workflows/release.yml`: runs on a `v*` tag.
    - It checks the tag against `Version` and `Stable tag` and takes the notes from the version's CHANGELOG
      section.
    - It runs Plugin Check on the `--wporg` build, then builds `monks-central.zip` and `monks-central.zip.sha256`.
    - `gh release create --draft --verify-tag` makes a draft, normal (not pre-release) release with both files.
  - `.github/workflows/ci.yml`: Plugin Check now runs on the `--wporg` build too. On the GitHub build it fails
    with `plugin_updater_detected`, which the updater is meant to trigger.
  - `tests/release/fresh-site.sh`: the release test below, repeatable before each release.
  - `docs/release/checklist.md`: Joy's decisions and the release steps, by Actions or by hand.
- **monks-central** (app, `feat/plugin-connector`): no change. Its `monks` CLI and acceptance driver ran the
  connection in the fresh-site test.
- **classic-monks** and **classic-monks-free** (`feat/monks-central`): no change. The mix suite used them as they
  are.

## Commits (monks-central-wp, `main`, all pushed)

- `62a5db5` build(zip): leave README, CHANGELOG and composer.json out, fail on any dev file
- `82f66e8` chore: version 1.0.0
- `89a6af4` docs(readme): description, FAQ, external services and the 1.0.0 changelog
- `4e901d3` docs(changelog): Monks Central 1.0.0 in place of the starter's history
- `861d0e2` ci(release): a v* tag builds the zip and its SHA-256 into a draft release
- `84c789a` test(release): the built zip on a clean site, connected by the app, updated from a fake release
- `6894617` docs: name the switch as the screen does, Allow Monks Central to connect
- `247b2ab` fix(zip): keep composer.json next to vendor/, as Plugin Check asks
- `ec35579` fix(release): run Plugin Check on the wp.org build, which has no updater
- `8c7db71` fix(ci): run Plugin Check on the wp.org build, which has no updater
- `1696117` docs(release): checklist of Joy's decisions and the release steps
- `acf9028` docs(readme): install from GitHub Releases, what is inside, releasing

## Test totals

All on the final commit.

| Check | Result |
|---|---|
| `composer lint` (PHPCS) | ok |
| `composer analyse` (PHPStan) | ok, no errors |
| `composer test` (PHPUnit) | 19 tests, 38 assertions |
| `npm run lint:js`, `npm run lint:css`, `npm run build` | ok |
| `tests/mcp/ci.sh` on 8889 with WooCommerce, Elementor and Bricks 2.4.3 | 29 suites, 906 checks passed |
| `tests/mix/mix.sh` on 8888 (every mix with Classic Monks Free and Pro) | 57 checks passed |
| `tests/release/fresh-site.sh` on 8888 + 8889 | 19 checks passed |
| Plugin Check, `--wporg` zip | 0 errors, 1 warning (`auto_update_plugin` in the connector's auto-update ability) |
| Plugin Check, GitHub zip | 1 error, `plugin_updater_detected` (the updater, by design; the workflows check the wp.org build) |
| `actionlint` on both workflows | ok |
| Release workflow's tag and notes steps, run locally | v1.0.0 passes with 30 lines of notes; v1.0.1 and v1x0x0 refused |
| Build guard | failed the build when `composer.json` was still on its list, and matched a planted `tests/` file and `README.md` |
| Em dashes | none |

Not run: `npm run test:e2e`, since nothing on screen changed. The release workflow itself has not run on
GitHub, because that needs a tag.

## Live results: fresh site

`tests/release/fresh-site.sh`: clean WordPress 7.1.3 (SQLite, `php -S` on 8888, nothing else installed); the app in
a throwaway data folder whose keychain item is deleted at the end.

| What | Result |
|---|---|
| Install | `wp plugin install dist/monks-central.zip --activate`: active, 1.0.0, no PHP errors. The MCP endpoint answers 404 until the admin turns on Allow Monks Central to connect (through the screen's REST route). |
| `monks doctor http://127.0.0.1:8888` | All checks passed (data folder, schema 10, keychain, TLS, site answered 200). |
| Connect the app | `connect-oauth`: the site's own consent page, all 4 tool groups and operator ticked. "added Monks Central fresh site ... WordPress 7.1.3, operator true, degraded false, 68 abilities, health Ok". The site lists one connection. The app reads "plugin Monks Central 1.0.0, connector 1.1.0". |
| Update offer | A fake GitHub release on 127.0.0.1:8889 (`releases/latest` JSON for v1.0.1, the zip and its `.sha256`). A test-only mu-plugin sends the site's api.github.com request there. WordPress offers 1.0.1. |
| Wrong checksum | "The downloaded update does not match its SHA-256 checksum, so it was not installed." 1.0.0 stays. |
| Right checksum | 1.0.1 installed and active. The connector stays on, the connection survives, the app's check is Ok, and it reads Monks Central 1.0.1. No PHP errors. |
| Settings > Monks Central from the zip (browser) | The switch, the site address, connector 1.1.0, the app under Connected apps (admin, Full, Operator), Activity "Nothing yet". |

Found and fixed during the run:
- The first run installed a zip built before the version bump, and the test caught it.
- The driver's login cookie is made for HTTPS Studio sites, so the test's `studio` shim makes the plain-HTTP cookie.
- A second run reused WP-CLI's cached 1.0.1 zip from the first. The updater refused it, since its checksum did not
  match. The test now uses its own WP-CLI cache, and I deleted the stale cache file.
- The docs named the switch "Connect to Monks Central", which is the card's heading. They now use the switch's
  label.
- Plugin Check findings: `composer.json` and the workflows' check target.

Studio sites (mc-a, mc-b, mc-c, monks-central-test) and the app's default data folder were not touched in this step:
their test data and grants are as step 4 left them. Ports used: 8888 and 8889 only.

## Decisions

- **1.0.0, not 0.x.** The goal asks for a 1.0.0 changelog. The connector is at 1.1.0 and already runs on
  Classic Monks sites. Joy can still pick another number (see the checklist).
- **Plugin Check runs on the wp.org build in both workflows.** The GitHub build has the updater on purpose, and
  Plugin Check rejects every self-updater. The wp.org build is what a reviewer would see.
- **composer.json ships, README.md and CHANGELOG.md do not.** wp.org asks for `composer.json` beside `vendor/`.
  readme.txt carries the changelog for sites.
- **The draft release is the safety catch.** The workflow never publishes. The updater and the app ignore drafts
  and pre-releases, so no site sees anything until a person publishes it.
- **The fresh-site test uses WP-CLI for the upload and the switch.** WordPress core's REST API can't upload a zip.
  The admin screen was checked in a browser on the same site afterwards.
- **Joy's decisions are in `docs/release/checklist.md`:**
  - making the repo public, which the updater and Install Monks Central for me need;
  - the version and the release date;
  - Actions billing;
  - wp.org later: the `--wporg` zip, readme changes, slug and contributor;
  - the unmerged app and Classic Monks branches.
