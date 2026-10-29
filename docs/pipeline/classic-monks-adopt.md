# Pipeline step 3: Classic Monks Free and Pro adopt the connector

Date: 2026-10-29. Plan: monks-central-plugin-plan.md, step 3.

## What changed per repo

- **monks-central-wp** (`main`): the library grew what a host's own screen needs, then went to `1.1.0`.
  - Filters for the host's tool groups, default groups, scope wording, error hints, instructions, activity
    caller, `get-hq-status` modules and extra status fields (`monks_central_toolsets`,
    `monks_central_default_toolsets`, `monks_central_scope_descriptions`, `monks_central_error_hints`,
    `monks_central_instructions`, `monks_central_activity_caller`, `monks_central_hq_modules`,
    `monks_central_hq_status`, `monks_central_tool_enabled`, `monks_central_load_abilities`).
  - A permissions API on `Monks_Central_Connector` for a host's screen: `set_permissions()` (scope, groups,
    trusted, namespaces), `set_operator_skip()`, `revoke()`, `pending()`, `activity()`, `undo()`,
    `set_enabled()`.
  - The 401 for an Application Password that was not made for MCP now says what works ("This Application
    Password was not created for MCP. Connect over OAuth, or use a connection made for MCP.").
  - `tests/mix/mix.sh`: every plugin mix on throwaway `php -S` sites (see Test totals).
- **classic-monks-free** (`feat/monks-central`):
  - `connector/` is an exact copy of the library, pinned in `connector.lock`. `bin/sync-connector.sh` copies
    it, and `--check` fails CI (workflow step and `tests/mcp/ci.sh`) when the copy differs.
  - Its own connector code is gone. `functions/monks-central/monks-central.php` registers the copy with the
    loader, adds Free's wording through the filters, and handles the screen's forms (`admin-post.php`,
    nonce per form, `manage_options`).
  - The screen is Dashboard › Monks Central, a live tab (never cached). The Connection subtab has the switch,
    site address and endpoint, then each connected app in a `cm-drawer-card` (access, tool groups, skip
    approvals, operator flag, Save, Disconnect), then Waiting for approval. The Activity subtab has Review
    and Undo.
  - The consent and approval pages (`admin/partials/monks-central-*.php`) use the dashboard components on
    `cm-framework.css`. Free and Pro both active: the copy loaded second stays off and shows a notice.
  - Docs and screenshots: `docs/features-docs/monks-central/README.md` (connection screen, activity, consent
    page).
- **classic-monks** (Pro, `feat/monks-central`):
  - Bundles the same `connector/`, with the same lock, sync script and CI checks as Free.
  - Removed its MCP server, OAuth, approvals, guard, activity log class, media helpers, consent view and CSS,
    and the 69 abilities the library ships.
  - Pro's own 139 ability files now register on `monks_central_register_abilities`, so they reach every
    connection whichever copy runs.
  - `functions/ai/mcp/monks-central.php` holds the Pro glue:
    - keeps the MCP switch shared (see Decisions);
    - adds Pro's tool groups (SEO, Bricks, Plugin modules), scope wording, 8 error hints, instructions,
      licence and module status;
    - answers the AI › MCP Server AJAX through `Monks_Central_Connector`.
  - AI › MCP Server keeps its Connections table and edit drawer (access, skip approvals, operator "Also skip
    the always-ask tools", tool groups, Revoke) and adds Waiting for approval and Activity with Undo. When
    another plugin's copy runs, a notice links to that plugin's screen.
  - The AI tab is live (never cached). The consent and approval pages are the same as Free's, with Pro
    wording.
  - New `CM_AI_Activity_Admin` keeps Logs › AI Activity working on the library's log.
  - Docs and screenshots: `docs/features-docs/ai-in-classic-monks/mcp-server.md` (subtab, connection
    drawer, consent page).
- **monks-central** (app): no change needed in this step.

## Commits

monks-central-wp (`main`):
- `c647b2d` feat(connector): filters for the host plugin's tool groups, hints and status
- `a942f4f` feat(connector): permissions API and abilities for a host's own UI
- `87fd22d` chore(connector): version 1.1.0
- `235016d` fix(connector): the 401 for a non-MCP Application Password names what works
- `d578066` test(mix): every mix of Classic Monks, Classic Monks Pro and Monks Central
- `c4b733e` test(mcp): expect the connector's 401 wording for a non-MCP password

classic-monks-free (`feat/monks-central`):
- `8fc9e8e` build(connector): bundle the Monks Central connector library
- `ebd4e66` feat(monks-central): run on the bundled connector, with a dashboard tab
- `e05c356` ci(plugin-check): check the plugin, not the bundled connector library
- `3850f4e` build(connector): sync the library at 235016d
- `97908b4` fix(monks-central): table columns, switch row and consent buttons
- `a6e1f3b` docs(monks-central): the connection screen, activity and consent page
- `1153548` test(mcp): expect the library's 401 wording for a non-MCP password
- `f9cff06` chore(monks-central): note why Plugin Check's nonce and prefix warnings do not apply

classic-monks (`feat/monks-central`):
- `f11d6f93` build(connector): bundle the Monks Central connector library
- `c9f3cc9e` feat(mcp): run on the bundled connector, with approvals and activity in AI › MCP Server
- `350940f9` docs(mcp): the connector, AI › MCP Server, its drawer and the consent page

## Test totals

All runs used WP 7.1.3, SQLite and `php -S` on 8888 or 8889, with WooCommerce, Elementor and Bricks 2.4.3.

| Run | Result |
|---|---|
| Free `tests/mcp/ci.sh` | All 28 suites passed, 906 checks, 0 failed |
| Pro `tests/mcp/ci.sh` | All 43 suites passed, 1428 checks, 0 failed (includes the new `connector.sh`) |
| `tests/mix/mix.sh` (this repo) | 57 checks, all passed |
| Monks Central `tests/mcp/ci.sh` on library 1.1.0 | 29 suites, 902 passed; the one failure was the outdated 401 wording in `trust.sh`, fixed in `c4b733e`, and `trust` reran green (24 checks) |
| `composer lint`, `composer analyse`, `composer test` (this repo) | green (PHPUnit 19 tests) |
| Plugin Check on the Free zip (`connector/` excluded) | 0 errors (main 0, branch base 6). 2311 warnings vs 2310 on main; the extra one is the `CM_Monks_Central_UI` class name. |

What the mix test covers:
- Monks Central alone, Free alone, Pro alone.
- Free + Monks Central and Pro + Monks Central, at equal versions (a tie goes to the plugin loaded first)
  and with either side newer.
- All three: Pro runs and Free stands down. Then Pro removed (Free runs), Free removed (Monks Central runs),
  and Pro added back (Pro runs).
- At every step: exactly one connector copy is loaded and the newest version runs. One grant made at the
  start still connects, keeps its scope and runs `get-hq-status`; Pro's `get-features` is served when Pro is
  active, also while Monks Central's copy runs.

## Live results

Pro build `classic-monks-2.6.0.zip` (header 2.6.0-dev, connector 1.1.0) installed with `plugin install --force`
over the existing copy on **mc-a** and **monks-central-test**. Plugin folders and SQLite databases were backed
up first to `~/.cache/cm-adopt-backup/<site>/`.

| Site | Before | After |
|---|---|---|
| mc-a | Pro 2.6.0-dev, own connector; `cm_mcp_enabled` 1 in both stores; 82 OAuth token rows, 184 log rows | Connector 1.1.0 runs as "Classic Monks Pro"; switch 1 in both stores; 82 token rows, 184 log rows |
| monks-central-test | same; 19 token rows, 60 log rows | same; 19 token rows, 60 log rows |
| mc-b | Pro 2.5.0 | untouched (2.5.0) |
| mc-c | Basic (Monks Central 0.1.0) | untouched |

Grant checks after the upgrade, on mc-a and monks-central-test:
- `monks doctor https://<site>.local`: All checks passed (also on mc-b and mc-c).
- `monks run <site> get-hq-status` with the app's existing operator grants: the `connection` (oauth,
  operator, scope full, 6 groups, trusted), `mcp`, `modules`, `licence` and `pending_approvals` fields are
  identical before and after.
- `get-features` (a Pro tool), `get-site-snapshot` and `list-ai-activity` answer through the same grants.
- No new PHP errors.

## Decisions

- **The MCP switch lives in `wp_options`.**
  - Why: every plugin's copy reads `cm_mcp_enabled` from `wp_options` at `plugins_loaded` 1, so a site keeps
    its switch whichever plugin runs the connection.
  - Pro mirrors it both ways (`cm_update_option_cm_mcp_enabled` and `update_option_cm_mcp_enabled`) and
    migrates its own value once at load.
  - Pro's `pre_option_cm_mcp_enabled` filter (installed at `plugins_loaded` 7) is removed at 8 so it cannot
    hide the shared value.
  - Under WP-CLI and in admin, Pro batches its own option writes until shutdown, so the mirror lands at the
    end of that request. The tests read it back in the next one.
- **Plugin Check skips `connector/`.** The library is checked in its own repo with the `monks-central` text
  domain; inside Classic Monks it raised 451 TextDomainMismatch warnings that would only go away by forking
  the copy, which would break the identical-copy rule.
- **Load order.**
  - A tie goes to the plugin loaded first. `active_plugins` is sorted, so on a test site
    `classic-monks-pro/` loads before `classic-monks/` and `monks-central/`.
  - On the live sites Pro sits in `classic-monks/`, and Free cannot share that folder. That is why only the
    Pro build went live; the Free build (`dist/classic-monks.zip`, connector included) was tested on
    `php -S` sites.
  - With Free and Pro both active, the second Classic Monks stays off with a notice instead of a fatal error.
- **Pro's tools need its AI module on**, as before this step: Pro's abilities are its AI abilities. The
  library's tools work either way.
- **Screens.**
  - Free gets a dashboard tab; Pro's screen stays inside AI › MCP Server, where its existing JS drawer
    already edits each connection.
  - Both reuse the dashboard components (`cm-section-header`, `cm-column`, `cm-drawer-card`, `cm-switch`,
    `cm-notice`, `cm-field-description`, `cm-table-container`).
  - The only inline style is `--cm-table-cols` on a table container, the same pattern the logging tabs
    use.
  - The consent page adds `wp-core-ui` and pins both decision buttons to one height in `cm-framework.css`.
- **mc-b stays on 2.5.0 and mc-c on Basic**, as asked. The library's legacy stand-down covers a 2.5.0 Pro
  next to a newer copy.
