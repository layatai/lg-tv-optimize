# Maintain Developer Mode installations

Use this procedure only for an active LG Developer Mode installation with authorized `prisoner` access. For rooted Homebrew installations, use the existing root workflow. Never replace a directory-shaped `/var/luna/preferences/devmode_enabled` with a token file.

## Pair and deploy without root

1. Confirm the intended TV, model, firmware, and account. Use the existing webOS CLI configuration when available.
2. If pairing is missing, ask the user to enable **Key Server** in LG's Developer Mode app. Handle its passphrase privately. Keep keys and pairing files outside the skill checkout.
3. Check the CLI help for the installed version. On the tested TV profile, `ares-shell` was unsupported. `ares-novacom --device DEVICE_NAME --run 'READ_ONLY_COMMAND'` provided the authorized shell transport.
4. Package, install, and launch the approved app with the configured device name:

```sh
	ares-package APP_DIRECTORY --outdir OUTPUT_DIRECTORY
	ares-install --device DEVICE_NAME VERIFIED_PACKAGE.ipk
	ares-launch --device DEVICE_NAME APP_ID
```

5. Verify the installed app ID and version, then poll for the intended foreground app. Test content, remote navigation, and playback. Installation success alone does not prove the app works.

Developer Mode SSH does not grant root or unrestricted system-app removal. The CLI can manage Developer Mode packages; use the TV's supported app-management UI for other removable apps. Follow [app cleanup](app-cleanup-and-network.md) before removing anything.

## Renew directly with LG

LG's official app supports **EXTEND** while the session is active. Its documentation says an expired session cannot be extended. The community Dev Manager also uses LG's session endpoints. Treat these endpoints as an implementation dependency, not a guaranteed public API.

The verified request contract is:

| Operation | HTTPS endpoint on `developer.lge.com` | Query parameter |
| --- | --- | --- |
| Check remaining time | `/secure/CheckDevModeSession.dev` | `sessionToken` |
| Reset the timer | `/secure/ResetDevModeSession.dev` | `sessionToken` |

The parameter is case-sensitive. A successful check returned `result: "success"`, `errorCode: "200"`, and a remaining time such as `1000:00:00` in `errorMsg`. Check the current response schema before relying on those fields.

Implement renewal as follows:

1. Obtain the active token from the regular file `/var/luna/preferences/devmode_enabled` through the authorized TV transport. Capture stdout privately. Do not print the token or a complete renewal URL.
2. Store the token in a private directory with mode `0700` and a file with mode `0600`, outside the repository and app package. Validate the token before using it. The tested token contained only ASCII letters and digits.
3. When the TV is reachable, refresh the cached token. With a valid cached token, the scheduler can contact LG while the TV is off. Do not wake the TV for this check.
4. Build the HTTPS URL inside the HTTP client. Keep the token out of shell arguments, process listings, browser history, scheduler prompts, and logs. Send it only to LG. Preserve certificate validation, reject redirects, and set request timeouts.
5. Read the remaining time, request renewal, then read the remaining time again. Require valid JSON and `result: "success"` for every response. HTTP 200 or a reset acknowledgement alone is insufficient.
6. Verify that the timer increased, or that it was already near the observed maximum. Record the times and verification date without the token. Do not assume the maximum is always exactly 999 hours.
7. On rejection or expiry, report the failure. Restore Developer Mode through LG's app as needed, obtain a new token, and check whether sideloaded apps need reinstalling. Do not claim renewal can recover an expired installation.

A real request on the reference TV increased the server-reported time from approximately 998.5 hours to `1000:00:00`. This proves that renewal request, not indefinite future availability.

## Schedule and verify renewal

Use the user's available scheduler. A daily check leaves margin before the observed 1000-hour limit. Keep credentials in the protected local file, not in the task definition. Refresh the token when possible and report rejected tokens or failed verification.

For Codex, discover its automation tool and prefer a recurring task in the existing chat. Create the task only after the renewal command succeeds, then inspect the saved schedule, status, and notification policy. Use the user's timezone. Report failures without daily success chatter unless the user requests it. Local tasks require the computer and desktop app to be running; disclose that dependency. A cloud scheduler needs its own authorized secret storage and is a separate deployment decision.

Keep the token, renewal command, and scheduler configuration available outside temporary build directories. Record how to pause or remove the schedule and delete its saved token. Do not disable Developer Mode as a rollback because that can remove sideloaded apps.

## Explain installation lifetime accurately

- Developer Mode installations require an active session. Automatic renewal manages that requirement; it does not remove it.
- Rooted Homebrew installations do not require Developer Mode renewal. Confirm current exploit compatibility for the exact model and firmware before proposing root. This skill does not authorize rooting. After an authorized migration to root, review obsolete renewal schedules and retire them within that migration plan. Do not let a stale renewal task reinstall Developer Mode or replace the root installation's directory marker.
- LG's app-store route requires submission and approval. Every sideloaded dependency must also be considered. Publishing a catalog app alone does not make a sideloaded video player permanent.

Do not promise immunity to firmware updates, factory resets, account changes, or LG endpoint changes.

## Sources

- [LG Developer Mode setup and extension](https://webostv.developer.lge.com/develop/getting-started/developer-mode-app)
- [Dev Manager session-check implementation](https://github.com/webosbrew/dev-manager-desktop/blob/main/src-tauri/src/plugins/devmode.rs)
- [Dev Manager renewal template](https://github.com/webosbrew/dev-manager-desktop/blob/main/src/app/info/renew-script/renew-script.sh.ts)
- [webOS Homebrew: Developer Mode lifetime](https://www.webosbrew.org/devmode/)
- [webOS Homebrew: root versus Developer Mode](https://www.webosbrew.org/rooting/)
- [LG app approval process](https://webostv.developer.lge.com/distribute/app-approval-process)
- [Codex local scheduled-task requirements](https://learn.chatgpt.com/docs/automations?surface=app)
