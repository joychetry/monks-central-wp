# Pipeline step 4: the app detects the plugin that runs each site's connector

Date: 2026-10-29. Plan: monks-central-plugin-plan.md, step 4.

Branch: `feat/plugin-connector` in monks-central, made from `feat/media-editor` (025fb49). That branch exists,
is not merged into `main` and has no pull request, so the step builds on it, as the plan asks.

## What changed per repo

- **monks-central** (app, `feat/plugin-connector`, 38 files, +1894 / -60):
  - Core: `describe_cm` and every check keep `plugin` {name, version} and `connector` {version, hosts} from
    `get-hq-status`, so the record follows a plugin being added, removed or switched. Classic Monks 2.5.0
    (mc-b) reports neither and still works as before.
  - Core: `Client::save_monks_central` reads the latest release of `joychetry/monks-central-wp` (a new
    `Feeds::plugin_release`, overridable like the other outside addresses):
    - it refuses drafts, pre-releases, tags that are not a version, and assets on another origin over HTTP;
    - it downloads `monks-central.zip` and `monks-central.zip.sha256`, with the same size cap and zip header
      check as the existing zip install;
    - it saves `monks-central-<version>.zip` only when the SHA-256 matches;
    - a 404 says there is no release yet, and a failed request says to check the connection.
  - App: `plugin_download` (saves into Downloads) and `site_connect_plugin` (the existing sign-in that
    switches a Basic site to the plugin connection and keeps its group, tags and notes).
  - UI, Sites:
    - The inspector's Plugin row says which plugin runs the connector and its version, e.g. "Classic Monks Pro
      2.6.0-dev, connector 1.1.0". Classic Monks 2.5.0 shows "Classic Monks, built-in connector".
    - An "Also on the site" row lists other copies that stand down ("Monks Central 1.0.0, not running").
    - The table and the filter show the plugin's name, with "Classic Monks Free" for Free.
  - UI, Basic mode: the panel says Monks Central, Classic Monks or Classic Monks Pro all work. Install Monks
    Central for me saves the checked zip and opens the site's Upload Plugin page with what to do. Connect
    through the plugin then switches the site over.
  - UI, Add a site: the recommended card is "A Monks Central plugin": "Monks Central, Classic Monks or Classic
    Monks Pro on the site. Any of them works." The empty states, the overview, the Classic Monks card and
    every Basic mode hint that any of the three plugins unlocks say the same.
  - CLI: the MCP `sites_list` names each site's plugin and connector version instead of `classic_monks`.
  - Live driver: `plugins` and `plugin-download` in `crates/core/examples/acceptance.rs`.
  - Docs: `docs/acceptance/plugin-connector.txt` (the whole live run), with screenshots in
    `docs/acceptance/plugin-connector/` and `docs/ux/plugin-connector/`.
- **monks-central-wp** (`main`): no code change. `bin/build-zip.sh` at `cbd4993` built the zip that the
  release stand-in served (`dist/`, not committed). This report and its status file are the only commit.
- **classic-monks** and **classic-monks-free**: no change. They stay at the end of step 3 on
  `feat/monks-central` (`350940f9` and `f9cff06`).

## Commits

monks-central (`feat/plugin-connector`, all pushed):
- `f01a39a` feat(core): keep which plugin runs each site's connector, and the connector's version
- `161ff09` feat(app): get the Monks Central plugin from its latest GitHub release, then connect through it
- `76197c9` fix(core): the no-connection message names all three plugins
- `7cb04bc` feat(ui): name the plugin running each site's connector
- `ae612eb` feat(ui): Install Monks Central for me in Basic mode, and copy for the three plugins
- `f116c74` test(acceptance): plugins and plugin-download in the live driver
- `027b1e5` fix(ui): a Classic Monks that reports no version shows its built-in connector
- `f5bdee6` fix(core): say there is no release yet when GitHub has none
- `b4151a2` feat(cli): sites_list names the plugin that connects each site and its connector
- `6c41c6b` fix: hints for Basic mode name all three plugins where any of them works
- `e9a229d` fix(ui): Install Monks Central for me and Connect through the plugin get a row each
- `40e0261` docs: live acceptance for the plugin connector step, with screenshots

New tests:
- core: `oauth_site_is_added` checks the plugin and connector, `a_check_sees_which_plugin_runs_the_connector`
  (switch to Monks Central, then the 2.5.0 shape), `a_basic_site_switches_to_the_plugin_connection`, and
  `the_plugin_is_saved_only_when_its_checksum_matches` (match, wrong sum, pre-release, bad tag, no assets,
  asset on another origin, 404, server away);
- CLI: `the_site_list_names_the_plugin_and_connector`;
- UI: the plugin row, the spare copy and the no-version case in `Sites.test.tsx`, and the Basic mode install
  and connect flow; the mock core and a new screen `sites-basic-get-plugin` for Playwright.

## Test totals

All checks ran on the final commit.

| Check | Result |
|---|---|
| `scripts/sidecar.sh` | ok |
| `cargo fmt --all --check` | ok |
| `cargo clippy --workspace --all-targets --locked -- -D warnings` | ok |
| `cargo test --workspace --locked` | 157 passed, 0 failed (core 136, IPC 9, app 6, CLI 3 + 2 + 1) |
| `scripts/check-headers.sh` | ok |
| `tsc --noEmit` | ok |
| vitest | 23 files, 326 tests passed |
| Playwright (full) | 514 passed, 0 failed, 268 skipped (screenshot and comparison runs, off unless `SHOTS` or `COMPARE` is set) |
| `tauri build --no-bundle` | ok (`target/release/monks-central`) |

## Live results

Studio sites in the app's default data folder, as left by step 3: mc-a and monks-central-test on Classic Monks
Pro 2.6.0-dev (connector 1.1.0), mc-b on Classic Monks 2.5.0, and mc-c in Basic mode with Monks Central 0.1.0
(connector 1.0.0). The preview app was stopped for the run and started again after. Full transcript:
`monks-central/docs/acceptance/plugin-connector.txt`.

| What | Result |
|---|---|
| Plugin shown per site | MC A and Monks Central Test "Classic Monks Pro 2.6.0-dev, connector 1.1.0"; mc-b "Classic Monks, built-in connector"; mc-c Basic. Same in the real window and in an agent's `sites_list`. |
| Install Monks Central for me (mc-c, Basic) | Latest release saved, its SHA-256 equal to the published one. A wrong published checksum saved nothing. The site's Upload Plugin step, played by WP-CLI with the saved zip, took mc-c from connector 1.0.0 to 1.1.0. |
| Real GitHub | "There is no Monks Central release on GitHub yet. Try again later." in the core and in the window (the repo is private and has no release). |
| Connect through the plugin | In a throwaway data folder: mc-c in Basic mode switched to the plugin connection: "Monks Central 0.1.0, connector 1.1.0", operator, 68 tools. Then removed, signing out on the site. |
| Phases 0 to 7, media, editor | All PASS again (doctor, refresh, WordPress 7.0 refused, sign in again, inventory, rollback, two-site update job, content edits with block hashes kept, Elementor and Bricks refused, login links, comments, settings Undo, users, approvals, socket and MCP plans, broken update rolled back, security, publish to 3 sites, report PDF, zip install, media replace and restore, ACF save and Undo). |
| Old Classic Monks | mc-b (2.5.0) answers checks, refuses what it can't do with "update" wording and takes a plan from an agent, as before. |
| Grants at the end | Same as before the run: MC A 1 Monks Central + 1 AI agents, mc-b 1, Monks Central Test 1, mc-c none (Basic). |

Test data kept: MC A recipe 90, image 91, attachment 95, page 12's two edited lines, comment 7, draft 93;
mc-c post 70, comment 5, draft 72 and Monks Central on connector 1.1.0; "November opening hours (plugin
connector)" on MC A, Monks Central Test and mc-c. Monks Central Test's comment 2, which was pending test data,
was put back to pending.

Found during the live run and fixed, each with a test: mc-b's panel said "not known" (`027b1e5`); GitHub's
404 read as a network problem (`f5bdee6`); the two Basic mode buttons overlapped in the real window
(`e9a229d`); Basic mode hints named only Classic Monks (`6c41c6b`).

## Decisions

- **Install Monks Central for me is a guided install.**
  - Why: WordPress core REST installs plugins only by WordPress.org slug and has no zip upload. A Basic
    site has only an Application Password, which wp-admin does not accept for its upload form. The existing
    zip install job needs a connector on the site.
  - So the app does what it can: it fetches the latest GitHub release with the same download limits as the
    zip install, checks the SHA-256 and saves the zip. It then opens the site's Upload Plugin page and says
    the one step left. Connect through the plugin finishes the switch with the existing sign-in, keeping
    group, tags and notes.
  - The existing zip install path itself ran on Monks Central Test in Phase 7 (installed, refused again
    without Replace, deleted).
- **A local release stand-in.** monks-central-wp is private and has no release, and this step must not tag
  or release. So the live install used a server on 127.0.0.1:8889 that served the GitHub `releases/latest`
  shape, with the zip from `main` `cbd4993` and its `.sha256`. The driver points at it with
  `MONKS_PLUGIN_RELEASE`. The real address was tried too.
- **mc-c stays Basic in the default data folder.** The switch to the plugin connection was tested on mc-c in
  a throwaway folder, then signed out, so the person's setup and grants are unchanged.
- **Free is shown as "Classic Monks Free"**, since the connector reports Free's host name as "Classic
  Monks". A site with no connector data falls back to "Classic Monks".
- **Copy sweep scope.** Hints changed where any of the three plugins gives the feature (password reset, media
  replace, one-click login, install from a link, custom fields, revisions and trash). Classic Monks reports,
  module features and "Update Classic Monks" messages keep their wording, since only Classic Monks has them.
- **Playwright's 300-site timing test** can run slower than its 100 ms budget while 4 workers run beside it,
  as in the media and editor steps. It failed that way once during this step (116 and 130 ms) and passed
  alone 3 times; it passed in the final full run.
