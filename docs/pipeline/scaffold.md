# Pipeline step 1: scaffold

Date: 2026-10-29. Plan: monks-central-plugin-plan.md, step 1.

## What changed per repo

- **monks-central-wp** (new, `joychetry/monks-central-wp`, PRIVATE, branch `main`):
  created from `wp-plugin-starter` at `c5c5d36` (all tracked files, no
  history), `git init`, `gh repo create --private`, pushed.
  - `bin/rename.sh` now accepts a prefix with underscores, then ran
    `bin/rename.sh monks-central "Monks Central" monks_central`: slug and text
    domain `monks-central`, namespace `MonksCentral`, hooks and options
    `monks_central_`, constants `MONKS_CENTRAL_`, handles and CSS classes
    `monks-central-`, REST namespace `monks-central/v1`, JS global
    `monksCentralAdmin`, main file `monks-central.php`.
  - Example module removed: `src/Modules/Example`, the front-end bundle, the
    example settings fields, the DataForm settings card, the blueprint's sample
    posts, and their PHPUnit and e2e tests. Registry, settings option, settings
    REST route, admin screen (DataViews modules table) and ability category stay.
  - Floors: WordPress 7.1 (header, readme.txt, PHPCS `minimum_wp_version`,
    docs), PHP 8.1 (unchanged: header, composer platform, PHPCompatibility 8.1-,
    CI matrix 8.1 and 8.3).
  - Header, readme.txt, README, composer and npm descriptions describe the site
    connector for the Monks Central app. AGENTS.md/CLAUDE.md say the repo is
    private.
  - `.github/workflows/ci.yml` kept as is (renamed paths only).
- **wp-plugin-starter, classic-monks, classic-monks-free, monks-central (app)**:
  untouched in this step.

## Commits (monks-central-wp, main, each pushed)

- `3fa12e3` chore: import wp-plugin-starter c5c5d36 without history
- `ca4c2b6` feat(rename): accept prefixes with underscores
- `fbe16a3` refactor: rename starter to Monks Central
- `64da656` refactor: remove the example module
- `0583037` chore: require WordPress 7.1
- `5a7e6ee` docs: describe the plugin as the Monks Central site connector
- this report

## Test totals (local)

| Check | Result |
| --- | --- |
| `composer install`, `npm ci` (fresh) | OK |
| `php -l` on every PHP file | OK |
| PHPCS (`composer lint`) | 0 errors, 0 warnings |
| PHPStan level 6 (`composer analyse`) | 0 errors |
| PHPUnit (`composer test`) | 6 tests, 11 assertions, OK |
| ESLint, Stylelint, `npm run build` | OK (3 bundle-size warnings, same as the starter) |
| Playwright e2e on Playground (WP latest = 7.1.4, port 9500) | 3 passed |
| `npm run plugin-zip` + Plugin Check 2.1.0 on WP 7.1.4 (wp-cli + SQLite, harness pattern) | No errors found |

## Live results

None needed for this step: the plugin has no connector yet, so the Studio sites
(mc-a, mc-b, mc-c, monks-central-test) and their grants were not touched.

GitHub Actions: every run on this repo fails in about 5 seconds because GitHub
did not start the jobs: "recent account payments have failed or your spending
limit needs to be increased". This is account billing for private-repo minutes,
not the code (it also hits the unchanged starter import). The workflow is kept;
fix billing or the spending limit to get CI back.

## Decisions

- Copy method: `git archive` honors the starter's `export-ignore` and dropped
  the dev files, so the import commit was amended (seconds after creation,
  nothing else on the repo) with every tracked file via `git ls-files`.
- Prefix `monks_central_`: `bin/rename.sh` only allowed letters and digits,
  so it now maps an underscore prefix to `monks-central` for handles, CSS and
  REST and `monksCentral` for JS globals. Plain prefixes behave as before. Not
  ported back to wp-plugin-starter (out of scope).
- No clash with Classic Monks Free: its connector uses `cm_mcp_*` options,
  `classic-monks/v1` REST and `monks_central_*` global functions; this plugin
  uses namespaced classes and the `monks_central_settings` option.
- The settings card was removed with the example fields (no fields left); the
  `Settings` class keeps its string, integer and boolean handling for the
  connector step.
- E2E runs on Node 24 (fnm); Playground's `fs-ext` has no prebuilt binary for
  the system Node 26. CI already uses Node 24.
- `docs/STACK.md`, `REUSE.md`, `MODULES.md` and `CHANGELOG.md` stay the
  starter's until release-ready.
