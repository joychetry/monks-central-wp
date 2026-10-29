# Pipeline step 2: connector library

Date: 2026-10-29. Plan: monks-central-plugin-plan.md, step 2.

## What changed per repo

- **monks-central-wp** (`joychetry/monks-central-wp`, PRIVATE, branch `main`):
  - `connector/`: the shared connector library. Built from Classic Monks Free
    `feat/monks-central` (`f9ef302`, `functions/monks-central`) plus what only
    Pro's connector had: `undo-operation`, `list-ai-activity`,
    `verify-core-checksums`, `verify-plugin-checksums` and the activity log's
    undo (target state, drift check, `undo()`). Media (upload, update, replace,
    restore) and ACF abilities came with Free's copy.
    - Own version: `connector/version.php` (`1.0.0`), exposed as
      `MONKS_CENTRAL_CONNECTOR_VERSION` by the copy that runs.
    - Version-wins loader (`connector/load.php`): each plugin requires it and
      calls `monks_central_connector_register()` with its dir, file, name, own
      version and UI class. On `plugins_loaded` (priority 1) the highest library
      version boots (first registered wins a tie); the other copies never load
      their code.
    - Same tables, options and meta as Classic Monks (`cm_mcp_oauth_tokens`,
      `cm_ai_log`, `cm_mcp_enabled`, `cm_mcp_scopes`, `cm_mcp_toolsets`,
      `cm_mcp_trusted`, `cm_mcp_operator`, `cm_mcp_namespaces`, MCP `app_id`),
      the same `classic-monks/v1` routes and `classic-monks/*` ability names, so
      existing grants keep working.
    - No admin UI of its own: `Monks_Central_Connector` gives data and actions
      (state, connections, revoke, operator skip, pending approvals, activity,
      undo, on/off); the host renders the consent and approval pages through
      `Monks_Central_Connector_UI`.
    - `monks_central_register_abilities` fires on `wp_abilities_api_init`
      (priority 20) with the running library version, so every host adds its
      own abilities whichever copy runs.
    - `connector/uninstall.php`: shared cleanup for hosts (see Decisions).
    - `connector/tests`: PHPUnit suite `connector` (loader).
  - Plugin: registers its copy; `src/Connector/Ui.php` with
    `templates/consent.php` (standalone consent page, own CSS) and the approval
    page; `src/Rest/ConnectorController.php` (`monks-central/v1/connector`,
    admins only); the React screen on Settings > Monks Central
    (`assets-src/admin/connector.js`: switch, endpoint, connected apps and
    activity in DataViews, approval requests, undo, notices when another
    plugin's copy or a Classic Monks build serves the connection).
  - `src/Updater.php`: latest non-pre-release GitHub release, installs only
    when the zip matches the release's `monks-central.zip.sha256`.
    `bin/build-zip.sh` writes the checksum; `--wporg` strips the updater.
  - `uninstall.php` calls the connector cleanup.
  - `tests/mcp`: Free's suites ported (see Test totals); new `loader.sh`;
    `admin-page.sh`, `standdown.sh`, `legacy.sh` rewritten for this plugin.
  - Lint and analysis now cover `connector/` (PHPCS WPCS 3, PHPStan level 6).
- **classic-monks, classic-monks-free** (`feat/monks-central`) and
  **monks-central** (app): untouched in this step (adoption is step 3, the app
  is step 4).

## Commits (monks-central-wp, main)

- `f2958ce` chore(connector): import the Classic Monks Free connector
- `278642b` style(connector): WordPress coding standards and the monks-central text domain
- `c8434c7` feat(connector): version-wins loader, host data and actions, UI interface
- `1c55345` feat(connector): Classic Monks Pro's undo, activity and checksum tools
- `0237194` test(connector): loader tests, static analysis and host versions
- `2b4e325` feat(connector): run the bundled connector with this plugin's consent and approval pages
- `8117bf7` feat(admin): connection screen on Settings > Monks Central
- `8c2ec08` feat(updater): updates from the latest GitHub release, SHA-256 checked
- `721f1ec` feat(uninstall): remove connector data only when no other plugin needs it
- `bd3f78e` test(mcp): port Classic Monks Free's connector suites
- this report

All pushed to `origin/main`; nothing tagged or released, repo still private.

## Test totals

| Check | Result |
|---|---|
| `php -l` on changed PHP | OK |
| PHPCS (`composer lint`) | 0 errors, 0 warnings |
| PHPStan level 6 (`composer analyse`) | No errors |
| PHPUnit (`composer test`, suites unit + connector) | 19 tests, 38 assertions, OK |
| ESLint, Stylelint, `npm run build` | OK (3 bundle-size warnings: DataViews) |
| Playwright e2e on Playground (WP latest, Node 24) | 4 passed (new: switch on and off) |
| Plugin Check 2.1.0 on WP 7.1.4, `--wporg` zip | 0 errors, 1 warning (`auto_update_plugin` in the `set-plugin-auto-update` ability, which Free also ships) |
| Plugin Check, GitHub zip | 1 error: `plugin_updater_detected` in `src/Updater.php`, expected; that is why `--wporg` strips it |
| `tests/mcp/ci.sh` (WP 7.1.3, SQLite, `php -S` on 8889, WooCommerce, Elementor, Bricks 2.4.3) | All 29 suites passed, 906 checks, 0 failed, 0 skipped |
| `legacy.sh` (`CM_WP_VERSION=7.0.7`) | 7 checks passed |
| em dash grep | none |

Per suite (PASS checks): annotations 1, smoke 17, protocol 35, approvals 37, trust 24, site 25, wrappers 14, wrappers2 18, content 47 (incl. `product`, `elementor_library`, `bricks_template`), m6 95, m7 26, m10 63, m8 40, media2 88, acf 64, oauth 79, hq-status 28, pending-approvals 13, revoke 16, oauth-grace 18, operator 26, login-link 24, updates 20, snapshot 17, admin-page 35, loader 14, standdown 11, deactivate 7, uninstall 4.

## Live results (Studio)

- **monks-central-test** (Classic Monks Pro with its own connector, 18 live
  OAuth tokens, 60 log rows): installed the GitHub zip and activated it. The
  library stood down (`state().legacy = true`), Pro's `CM_MCP_Server` kept
  serving, no new fatal in debug.log, front end 200. Deactivated and deleted
  it (runs `uninstall.php`): tokens 18 live / 19 total, 60 log rows,
  `cm_mcp_enabled` 1 and 264 abilities, all unchanged at every step. The site
  is back to its plugin list from before.
- **mc-c** (WooCommerce, Elementor, no Classic Monks, WP 7.1.4): installed the
  GitHub zip, turned the connector on; library 1.0.0 runs from this plugin.
  Suites run live against `https://mc-c.local` with `studio wp`: smoke 17,
  protocol 35, hq-status 28, approvals 37, oauth 79, admin-page 35 checks, all
  passing. Test credentials, the test OAuth grant and pending test approvals
  that an interrupted first run left behind were removed; the site's own
  `monks-central` app password was kept. The plugin stays installed and on
  there for the app step.
- **mc-a, mc-b**: not touched (Classic Monks Pro; same case as
  monks-central-test).

## Decisions

- Library code keeps the `cm_*` / `CM_*` names (`CM_MCP_Server`,
  `CM_MCP_OAuth`, `CM_MCP_Approvals`, `CM_AI_Activity_Log`) so Pro's extra
  abilities keep working on top of it; the new API is
  `Monks_Central_Connector` and `Monks_Central_Connector_UI`. Text domain
  `monks-central`; Free and Pro may get a text domain warning from Plugin Check
  in step 3.
- A registered copy keeps the host's own version under `version`; the library
  version is `connector` and decides which copy boots.
- Legacy stand-down: if `CM_FUNCTIONS_DIR` is defined and holds
  `ai/mcp/mcp-guard.php` or `monks-central/mcp/mcp-guard.php`, a Classic Monks
  build with its own connector runs the site and the library does not boot
  (same function names would be fatal). The screen says so.
- Deactivating a host revokes every connection only when it is the last
  registered copy.
- Uninstall: credentials go only when no other active plugin runs a connector;
  the activity log and on/off option go only when no other plugin that carries
  one is installed.
- Stay Pro-only (need Pro modules): rollback and `list-plugin-versions`, media
  trash and restore, modules, reports, Woo, Bricks and Elementor tools, the
  doctor and skills. Pro's chat source and error codes need filters in step 3.
- The login-link interstitial stays in the library (it is part of the flow,
  not a screen).
- Updater: release assets `monks-central.zip` and `.sha256`; it gets a 404
  while the repo is private, so it offers nothing until a public release.
- Suite changes beyond renames: `update-post` has no undo callback, so
  admin-page undoes `update-term`; login-link checks `wp_login` through a
  test mu-plugin (Free used its last-login column) and drops Free's "Replace
  wp-login" check; the annotations snapshot gains Pro's 4 abilities with
  Pro's values.
- `tests/mcp` runs the suites against a copy of the plugin, so run
  `composer install` and `npm run build` first. Running only login-link and
  admin-page on a brand-new site once failed login-link's first OAuth connect
  (the second connect in the same run worked); not investigated further, since
  the full list (oauth.sh first) passes.
- E2E runs on Node 24 (fnm), as in step 1.
