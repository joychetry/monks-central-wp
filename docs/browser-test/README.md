# Browser test: Monks Central 0.1.1

Date: 2026-10-29. Chromium (`/usr/bin/chromium`) driven by Playwright, 1440x900
unless noted, against the WordPress Studio sites. `WP_DEBUG` and `WP_DEBUG_LOG`
were on for mc-c and monks-central-test during the run.

- **mc-c**: Monks Central 0.1.0 alone, updated to 0.1.1 from the GitHub release.
- **monks-central-test**: Classic Monks Pro 2.6.0-dev (connector 1.1.1), with the
  v0.1.1 zip installed next to it for the coexistence check.

PR with the fixes: https://github.com/joychetry/monks-central-wp/pull/1

## Checks

| # | Check | Result | Screenshots |
| --- | --- | --- | --- |
| 1a | Dashboard > Updates "Check again" offers 0.1.1 | **Fail, fixed** (bug 1). A cached "no release" from before v0.1.1 was published hid the update for up to 12 hours. | [01a](01a-updates-check-again-stale-cache.png) |
| 1b | Updates and Plugins offer 0.1.1 from the GitHub release (after the cache expired) | Pass | [01b](01b-updates-offers-0.1.1.png), [01c](01c-plugins-update-notice.png) |
| 1c | "View details" popup before the update | Pass, notes broke mid-sentence (fixed, bug 3) | [01d](01d-view-details-before-update.png) |
| 1d | Update through wp-admin's "update now" | Pass: "Updated!", still active, version 0.1.1 | [01e](01e-update-done.png), [01f](01f-plugins-after-update.png) |
| 1e | SHA-256 check on the download | Pass. A temporary probe on `upgrader_pre_download` logged the file the updater handed over: `184082ed…6bc`, equal to the published `monks-central.zip.sha256`. Probe removed. | |
| 1f | Connections kept | Pass: the app's `monks-central` application password is still there | |
| 1g | Plugins after the update | **Fail, fixed** (bug 2): still said "There is a new version" for 0.1.1 and the admin bar counted 1 update | [01g](01g-view-details-after-update.png) |
| 2a | Settings > Monks Central renders, no console errors | Pass | [02a](02a-settings-overview.png) |
| 2b | On/off switch | Pass: off removes the MCP route (404) and hides the cards; on brings them back, existing tokens work again | [02h](02h-switch-off.png) |
| 2c | Site address and MCP endpoint | Pass, both shown. There is **no Copy button** on this screen (see gaps). | [02a](02a-settings-overview.png) |
| 2d | Connected apps: app, user, access level, approvals | Pass: Full / Site admin / Read only shown as granted | [02b](02b-connected-apps.png) |
| 2e | Operator toggle | Pass: off saves ("Connection saved."), survives reload, back on | [02c](02c-operator-off.png) |
| 2f | Revoke (throwaway `bt-throwaway`) | Pass: confirm dialog, row gone, its token now gets 401 | [02d](02d-revoke-confirm.png), [02e](02e-after-revoke.png) |
| 2g | Activity with Undo | Pass: operator changed post meta, Undo offered, "Change undone.", row marked `undone`, meta removed | [02f](02f-activity-undo-offered.png), [02g](02g-activity-after-undo.png) |
| 2h | Card intro text | **Fail, fixed** (bug 5): Connected apps and Activity intros sat flush against the card edge | [02b](02b-connected-apps.png), [08a](08a-fixed-card-padding.png) |
| 3a | OAuth: dynamic client registration, authorize URL, consent page renders | Pass | [03a](03a-consent.png) |
| 3b | Consent buttons | **Fail, fixed** (bug 4): Deny and Connect were unstyled browser buttons | [03a](03a-consent.png), [08b](08b-fixed-consent-buttons.png) |
| 3c | Deny | Pass: back to the client with `error=access_denied`, `state` and `iss` | |
| 3d | Approve with Operator ticked | Pass: scope `full`, 68 tools, shown as operator | [03b](03b-consent-filled.png) |
| 3e | Approve with Site admin + tool groups Content and Site admin | Pass: scope `site`, 45 tools, no media or user tools | [03c](03c-consent-filled.png) |
| 3f | Approve with Read only + Media | Pass: scope `read`, 1 tool (`get-media`) | |
| 3g | Token on the MCP endpoint (`initialize`, `tools/list`) | Pass for all three | |
| 3h | Consent error state (bad client) | Pass | [03d](03d-consent-error-state.png) |
| 4a | Non-operator `delete-post` asks for approval, shows in "Waiting for your approval" | Pass | [04a](04a-waiting-for-approval.png), [04a2](04a2-waiting-card.png) |
| 4b | Approve on the approve page, runs once | Pass: the retry trashed the post, the next identical call asked again | [04b](04b-approve-page.png), [04c](04c-approved.png) |
| 4c | Decline | Pass: the retry did not run, post untouched | [04d](04d-declined.png) |
| 5a | v0.1.1 zip uploaded and activated next to Classic Monks Pro | Pass: zip checksum OK, installs, activates, no notices | [05a](05a-plugins-both-active.png) |
| 5b | Exactly one connector runs | Pass: two copies registered (both 1.1.1), Classic Monks' boots, one MCP route | |
| 5c | Right screen shows, links to the other | Pass: Monks Central's screen says Classic Monks Pro runs the connection, "Open it" goes to Classic Monks > AI > MCP | [05b](05b-monks-central-screen-defers.png), [05c](05c-classic-monks-screen.png) |
| 5d | Existing grants work | Pass: a grant made before the install, and the Monks Central app's own grant (`acceptance check`: health Ok), with the plugin active and after it was deactivated again | |
| 5e | Deactivated again | Pass, site as before (plugin left installed but inactive; the test grant revoked) | [05d](05d-deactivated-again.png) |
| 6a | debug.log | Pass: no PHP errors, notices or deprecations from Monks Central or the connector (mc-c has Elementor deprecations only) | |
| 6b | JS console on the plugin's pages | Pass: none on Settings, approve, consent, plugins | |
| 6c | axe (WCAG 2 A/AA) | Settings and approve page pass. Consent: **fail, fixed** (bug 6), `color-contrast` on "name not verified" | |
| 6d | 782px admin | Pass: no page overflow; tables scroll inside their cards | [06a](06a-settings-782.png), [06b](06b-consent-782.png), [06c](06c-approve-782.png) |
| 7 | `monks doctor https://mc-c.local` after the update | Pass: "All checks passed." | |

## Bugs and fixes

All fixes are in this plugin, none in the connector library, so nothing needs
syncing to Classic Monks.

| # | Bug | Fix | Commit |
| --- | --- | --- | --- |
| 1 | "Check again" on Dashboard > Updates served the updater's 12-hour cache, so a cached "no release" (or an older release) hid a new version for up to 12 hours. | `Updater::release()` skips the cache on `load-update-core.php` with `force-check`, as core does for wp.org. | `fix(updater)` |
| 2 | Right after an update, wp-admin kept offering the version just installed. Core re-runs the update check in the request that installed it, where `MONKS_CENTRAL_VERSION` is still the old value. | `Updater::offer()` compares against core's `checked` version (read from disk), falling back to the constant. | `fix(updater)` |
| 3 | "View details" changelog broke lines mid-sentence (markdown list items wrap onto indented lines). | Join indented continuation lines before `wpautop()`. | `fix(updater)` |
| 4 | Consent page Deny and Connect rendered as bare browser buttons: `buttons.css` only styles `.wp-core-ui .button`. | `wp-core-ui` on the consent `<body>`. | `fix(ui)` |
| 5 | Connected apps and Activity intro text flush against the card edge (DataViews drops the card body's side padding). | 24px inline padding for that text. | `fix(ui)` |
| 6 | "name not verified" tag on the consent page under 4.5:1 contrast. | Text colour `#50575e`. | `fix(ui)` |

Tests: PHPUnit (22 tests, three new for bugs 1 to 3), `composer lint`,
`composer analyse`, `npm run lint:css`, `npm run build`, `npm run test:e2e`.
The fixes were also checked in the browser on mc-c (screenshots 08a, 08b,
"Check again" refreshing a cached "no release"), then mc-c was put back to the
exact v0.1.1 files.

## Gaps (not bugs, not changed)

The brief describes connection cards with tool-group editing, Save and
Disconnect, Copy buttons and a "Waiting for approval" table with Review. That is
Classic Monks' AI > MCP tab. Monks Central's own screen has none of these:

- No Copy button next to the site address or the endpoint.
- Access level and tool groups are chosen on the consent page only; the screen
  shows the access level but cannot change it or the tool groups.
- "Waiting for your approval" is a list of links; two requests for the same tool
  read the same ("delete-post from bt-agent") until you open them.
- Activity Details holds raw JSON on one line, so the table is about twice the
  card width and scrolls sideways (Undo stays visible in the sticky column).

## Notes

- monks-central-test's stored admin password no longer works. Resetting it would
  revoke that user's OAuth grants, so the browser used a session cookie minted
  with WP-CLI instead.
- `monks doctor` from `monks-central/target/debug` reports version 0.2.0.
- Test data left on mc-c: plugin 0.1.1, connections `bt-operator` and `bt-agent`,
  posts 74 and 75 (75 in the trash), activity rows.
