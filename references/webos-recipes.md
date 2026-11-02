# webOS recipes and pitfalls

These commands were verified on an LG 49UK7500PTA, firmware 05.50.70, SDK 4.4.0, ARM32, running Python 2.7.12 and Upstart. Paths, service names, APIs, and compatibility can differ on other firmware. Inspect first; skip unsupported settings rather than guessing substitutes.

Commands below run on the TV through the user's existing, verified SSH transport. No SSH wrapper, credentials, default TV address, or device profile is bundled. Confirm the target and account before connecting, and keep remote stdin open for Luna operations as described below.

## Transport and old-firmware constraints

- Rooted SSH normally uses root/22; Developer Mode's prisoner/9922 is a different endpoint. Do not accidentally use the latter for root maintenance.
- Keep remote stdin open while invoking Luna clients: stdin EOF can make `luna-send`/`luna-send-pub` exit before the reply arrives. Avoid `ssh -n` and `DEVNULL` stdin. A Python wrapper can hold a pipe's write end open while SSH reads its read end.
- Prefer existing SSH agent/credential integration. If an encrypted key already has a protected local credential helper, reuse it without printing its configuration or passphrase. Never place secrets in shell command arguments or logs.
- Verify the host key through the established trust mechanism. Do not bypass a changed-host-key warning. If an old TV requires `ssh-rsa` compatibility, scope that exception to the specific device, not global SSH settings.
- Large inline SSH commands were rejected on the reference TV. Transfer large files in roughly 3000-character base64 chunks to a new staging file, then validate size/SHA-256 and syntax before atomic rename. Never truncate the working helper first.
- Python 2.7 on the TV means no f-strings, pathlib, `subprocess.run`, or Python-3-only syntax. Host-side tooling may use Python 3.
- Homebrew hooks run in a minimal environment. Use `/sbin/initctl`, `/usr/bin/luna-send`, `/usr/bin/python` where verified and an explicit PATH such as `/sbin:/bin:/usr/sbin:/usr/bin`. Validate paths first and test with `env -i` before rebooting. Do not run a write-capable worker merely to check its syntax.
- A successful shell exit is not proof of a successful Luna operation. Inspect JSON `returnValue`, errors, and final subscription/install events.

## Read-only baseline

Run inside the established authorized TV session; commands/options may differ by firmware:

```sh
id
uname -m
uptime
free -m
df -h
ps -eo pid,ppid,pcpu,pmem,rss,comm
```

Read `/proc/sys/kernel/random/boot_id`, `/proc/meminfo`, and `/proc/swaps`; inspect zram's `mm_stat`, `disksize`, and `comp_algorithm` under `/sys/block/zram*` if present. Read device-mapper status/table if supported. Do not change swap while inspecting it. Observe process CPU deltas over a measured interval instead of relying only on lifetime-average `%CPU`.

Use the appropriate model/system-info API or known local metadata to establish firmware and SDK. Record only fields needed for the work; avoid publishing serial numbers and account identifiers.

## Supported privacy settings

Read current values first:

```sh
/usr/bin/luna-send -w 5000 -n 1 -f \
  luna://com.webos.settingsservice/getSystemSettings \
  '{"category":"option","keys":["watchedListCollection","webOSPromotionVideo","livePlus","promotionStreamer"]}'
```

After saving original values and receiving approval, set only present/supported keys. For example:

```sh
/usr/bin/luna-send -w 5000 -n 1 -f \
  luna://com.webos.settingsservice/setSystemSettings \
  '{"category":"option","settings":{"watchedListCollection":"off","webOSPromotionVideo":"off"}}'
```

Read back afterward and after an approved reboot. On the reference TV `livePlus` and `promotionStreamer` were already off, so there was no reason to toggle them. These options reduce specific collection/promotion features; they do not block every tracker or advertisement. Do not clear unrelated settings or revoke required agreements.

## Optional diagnostics: review, temporary test, then persist

On the reference firmware the following optional remote-diagnostics jobs were reviewed and stopped in this order:

1. `crashreportd`
2. `uploadd`
3. `rdxd`
4. `remotelogger`

This is NOT a cross-model service blacklist. Before touching any job, read its Upstart definition under `/etc/init/`, inspect dependencies, triggers, subprocesses, and references from other jobs. Record its original state using `/sbin/initctl status JOB`. If its role remains ambiguous, leave it running.

For an individually approved/reviewed job, use `/sbin/initctl stop JOB`, not binary deletion or `kill -9`. Confirm both Upstart state and the associated process state; briefly observe for respawn. Re-test networking, app launch, playback, audio, HDMI, accessibility, casting, and stability as applicable.

Preserve `crashd` (local crash handling), `sdx` (not sufficiently identified), `admanager` (on-demand/insufficient justification), `pacrunner` (network proxy support), and all essential services unless a separate task establishes a safe, explicit reason otherwise.

If approved temporary changes pass validation, a Homebrew boot hook can invoke a bounded cleanup worker that stops only the reviewed jobs, checks again after boot completion for late startup, and exits. Use absolute paths, logs, and a timeout. Do not install an always-running service-killing loop. Make rollback remove only the new hook, stop only its own worker, restore saved settings/files, and restart only jobs that were running before the pass, respecting their dependencies.

## Memory and performance interpretation

The reference TV had approximately 843.6 MiB usable RAM and a 700 MiB `/dev/mapper/eswap` combining zram and flash. Its zram device was already 600 MiB with LZ4; roughly 244 MiB original data occupied 89 MiB RAM at one inspection. Additional flash swap was not justified.

These values are historical, not tuning targets. A configured zram size is not its current physical allocation. Swap usage alone does not prove thrashing. Measure pressure, CPU, I/O and the user-visible symptom before recommending anything, and do not change live swap casually.

A previous three-sample test measured median foreground registration:
- HDMI1: 0.180 → 0.126 seconds.
- YouTube: 0.214 → 0.215 seconds.

This did not establish a general UI speedup. The verified startup improvement was reducing the custom helper's additional wait from five seconds to one. Cache and workload differences prevented permanent RAM-saving claims.

## YouTube fork installation/update

For NicholasBly's fork, inspect the current README, release notes, assets and repository manifest:
- https://github.com/NicholasBly/youtube-webos
- https://api.github.com/repos/NicholasBly/youtube-webos/releases/latest
- https://raw.githubusercontent.com/NicholasBly/youtube-webos/main/repo.json

Select a release explicitly compatible with the TV. At the recorded 0.8.4 release, the standard `youtube.leanback.v4_0.8.4_all.ipk` supported legacy webOS 3–6; the separate webOS 22+ asset did not belong on a webOS 4 TV. Re-check compatibility for future releases instead of assuming the naming remains unchanged. Do not present the historical version as perpetually latest.

1. Inspect live `appinfo.json` and application-manager metadata, available space, and the foreground app. Do not trust the display name or old checkpoint alone.
2. Download the chosen IPK locally; verify a trusted published SHA-256, inspect the contained app ID/version, and create a per-file digest manifest.
3. Back up the current app code and relevant package metadata outside the app directory; verify the archive. Back up affected hooks/mappings separately. Do not claim code archives include sign-in/cookie data.
4. For an existing homebrew YouTube with the same ID, close and update in place; do not uninstall unnecessarily. The README's official-YouTube-removal prerequisite is not a reason to uninstall the already-compatible homebrew app. Removing a conflicting official app on another TV needs explicit approval and backup considerations.
5. On an already-rooted TV, use the existing Homebrew installer. For this service version, the request is:

```sh
/usr/bin/luna-send -i -w 30000 -f \
  luna://org.webosbrew.hbchannel.service/install \
  '{"ipkUrl":"VERIFIED_RELEASE_URL","ipkHash":"VERIFIED_SHA256_HEX","subscribe":true}'
```

Fill placeholders only after verification. `ipkHash` here is a hex string; the repository manifest represents hashes differently. Verify the installed service API on other versions. Observe a successful final `finished` event, not only “Downloading” or “Installing.” If the request times out/disconnects, inspect state before retrying.

6. Check installed/registered metadata and compare the installed application files against the downloaded package. Launch and poll for foreground. Confirm the expected app ID still matches autostart, remapping, and HackerTube handoff.
7. Ask the user to test video/audio, sign-in and remote controls. GREEN opens this fork's settings in the documented release. Do not force maximum video quality as an optimization.

The stock Homebrew YouTube listing can be a different build with the same ID. Manually reinstalling it can replace the fork. For future updates, use the chosen fork's releases/repository; changing Homebrew repositories is a separate, explicit configuration change, not a side effect to hide.

## App launch, foreground, close

Verified API examples:

```sh
/usr/bin/luna-send -w 5000 -n 1 -f \
  luna://com.webos.applicationManager/launch '{"id":"youtube.leanback.v4"}'
/usr/bin/luna-send -w 5000 -n 1 -f \
  luna://com.webos.applicationManager/getForegroundAppInfo '{}'
/usr/bin/luna-send -w 5000 -n 1 -f \
  luna://com.webos.applicationManager/closeByAppId '{"id":"youtube.leanback.v4"}'
```

On this firmware `close` with `{id: ...}` failed with “no app matched by pid”; `closeByAppId` works. Polling for foreground is more reliable than checking once after one second. A window becoming foreground does not prove its contents have rendered or playback works.

## Power-on/wake autostart

Keep an existing working solution rather than stacking another autostart mechanism on top.

The tested helper subscribes to `com.webos.service.tvpower/power/getPowerState`. On a transition to `Active` it waits one second, checks `com.webos.bootManager/getBootStatus` for readiness (`firstAppLaunched` and `signals.core-boot-done`), re-checks active power, and launches once. Its maximum readiness wait is 120 seconds. It cancels pending launch on standby, deduplicates repeated notifications, locks against duplicate workers, and reconnects subscriptions. It must not relaunch merely because the user switches apps.

Preserve readiness checks while reducing delay. Test cold boot, duplicate Active events, delayed readiness, failed launch/retry, standby before the due time, wake, timeout, and ordinary app switching.

The fork README also documents EIM input registration for last-input autostart. It can keep YouTube resident and increase idle memory use; do not enable it in addition to a working power-on hook without explaining and approving that trade-off.

## Remote mappings and local-only configuration

LG Input Hook 1.5.1 used `/home/root/.config/lginputhook/keybinds.json`. Merge only approved keys; preserve all others. The tested mappings were:

```json
{
  "1037": {"action": "launch", "id": "youtube.leanback.v4"},
  "1038": {"action": "launch", "id": "youtube.leanback.v4"}
}
```

On the reference remote these are Netflix and Amazon. Other remotes can differ; verify event codes instead of overwriting existing meanings blindly. Native initialization was logged in `/tmp/lginput-hook-lginput2.log`.

For this version, `/media/developer/apps/usr/palm/services/org.webosbrew.inputhook.service/service.js` had a configuration HTTP server listening on port 1842. Back up the file and, with approval, change `server.listen(1842);` to `server.listen(1842, "127.0.0.1");`. Confirm the actual listener address after the service is restarted by its supported mechanism. Do not expose configuration without authentication to the LAN/internet. Use a trusted SSH tunnel if remote access is needed.

Updating/reinstalling Input Hook can overwrite this patch. Inspect and re-verify the listener after updates; adapt to the new source/API rather than blindly applying a stale text replacement.

## Controlled full reboot

Only after warning the user, saving rollback, and obtaining approval:

```sh
/usr/bin/luna-send -a com.webos.service.secondscreen.gateway -w 5000 -n 1 -f \
  luna://com.webos.service.sleep/shutdown/machineReboot '{"reason":"remoteKey"}'
```

A disconnect is expected. Reconnect with bounded retries and compare boot IDs. Distinguish full reboot from standby/Quick Start+ wake. Re-check persisted settings, logs, root/Homebrew and the essential features in the main workflow; a running process alone is not proof of working casting or playback.
