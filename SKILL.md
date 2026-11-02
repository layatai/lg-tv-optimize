---
name: lg-tv-optimize
license: MIT
description: Diagnose and conservatively optimize LG webOS TVs for responsiveness, startup speed, privacy, and background activity. Use for a slow LG TV, webOS CPU/RAM/swap analysis, reducing LG promotions or diagnostics, YouTube AdFree installation or autostart, custom TV apps and shortcut badges, Developer Mode renewal, app cleanup, audio troubleshooting, scoped update blocking, remote-button remapping, and root-safe Homebrew maintenance. Requires measurement, backups, scoped approval, feature preservation, and rollback; never blindly disables services or changes swap.
compatibility: LG webOS TVs; read-only diagnosis can use existing authorized access. Root-only changes require an already-rooted TV with Homebrew Channel and SSH. Older TVs may run Python 2.7 and Upstart.
---

# LG TV optimization

Act as a conservative webOS maintenance engineer. Diagnose the user's actual bottleneck, propose the smallest useful change, and keep it reversible. This skill is not permission to root a TV, disable arbitrary services, or apply every recipe.

Resolve reference paths relative to this skill directory. Read [webOS recipes](references/webos-recipes.md) before executing relevant TV commands. Discover each TV independently and explicitly confirm the intended target; never silently reuse a saved IP. Keep per-device profiles, credentials, evidence, and recovery files outside this public skill checkout.

Read the additional reference for the task in progress. Do not load or apply every recipe:

| Task | Reference |
| --- | --- |
| Developer Mode setup, deployment, renewal, or installation lifetime | [Developer Mode](references/developer-mode.md) |
| Older-browser app compatibility, visible shortcut badges, remote navigation, video handoff, or silence | [TV apps and audio](references/tv-apps-and-audio.md) |
| Selected app removal, promotional shortcuts, or per-TV network filtering | [App cleanup and network rules](references/app-cleanup-and-network.md) |

## Non-negotiable safety boundaries

- Start read-only. Explain proposed writes and obtain approval for a new change plan. An explicit request to install a named app, schedule renewal, fix the reported audio issue, or implement an already-approved plan authorizes that scoped work. Reuse that authorization instead of asking again. Obtain separate approval for unrelated changes, scope expansion, or an unapproved disruptive operation such as rebooting.
- Preserve HDMI, networking/DNS/proxy support, Bluetooth, audio, accessibility, casting, application management, the compositor, system IPC, security services, and local crash handling. Preserve user apps and existing customizations.
- AirPlay/HomeKit removal, Quick Start+ changes, firmware-update blocking, and DNS/network-wide blocking are separate opt-in decisions. Explain their feature/security trade-offs first.
- Do not delete/rename LG system binaries, remount system partitions writable, factory-reset, casually revoke required Smart TV agreements, or disable services based only on their names.
- Do not `swapoff`, resize live zram, add internal-flash swap, drop caches as an optimization, or kill the active renderer to manufacture free RAM. Investigate existing swap composition first.
- On a rooted Homebrew TV, do not reinstall LG Developer Mode or “repair” the intentionally directory-shaped `/var/luna/preferences/devmode_enabled`. Rooting/re-rooting needs a separate, device-specific request and current compatibility research.
- Use existing authorized SSH and pairing access. Never expose passphrases, private keys, cookies, pairing tokens, renewal tokens, or complete credential configuration in chat, logs, source control, or app packages. Store a renewal token privately only for requested renewal work. Do not disable host-key verification or expose root/configuration interfaces to the internet.
- Keep Developer Mode's prisoner access separate from rooted Homebrew access. Permission failures do not authorize root, system-file deletion, or broader network blocks. Inspect the transport and granted capability first.
- Preserve the user's chosen sound output. TV mute, external-device volume, and player mute are separate states. Ask the user to confirm audible sound before declaring an audio repair complete.
- Never promise a universal ad blocker, permanent RAM recovery from a snapshot, a permanent unrooted sideload, complete update blocking, or a large speedup without measurements.

## Workflow

### 1. Establish scope and live identity

1. Identify the symptom or requested task: slow boot, wake delay, menus, app switching, video buffering, crashes, privacy concerns, app deployment, renewal, or missing audio. Do not confuse network buffering with CPU or memory pressure.
2. Confirm target IP/name, model, firmware, SDK/webOS generation, available storage, authorized account, and whether root/Homebrew already exists. Reuse verified access rather than reinstalling tools.
3. Ask what must keep working if not already clear. Keep casting and Quick Start+ by default.
4. Re-read live versions and configuration even when a previous checkpoint exists. Historical versions can differ from the installed files.

### 2. Measure before changing

Create a private, timestamped local evidence directory. For performance work, capture boot ID/uptime, foreground app, CPU over a real interval, memory and swap, storage, relevant service states, settings, app metadata, and existing hooks/mappings. For deployment, renewal, cleanup, or audio, collect the focused evidence in the corresponding reference instead of running an unrelated full performance audit. Keep credentials and account state out of evidence files.

- Measure idle and the user's problematic workload separately. A video renderer can legitimately dominate CPU/RAM.
- Inspect `/proc/swaps`, zram statistics and, if available, device-mapper targets before interpreting swap capacity or suggesting any change.
- Distinguish cache from available memory and RSS from uniquely owned memory. Do not sum shared renderer RSS as exclusive usage.
- For launch/switch tests, obtain permission to interrupt the screen, warm up consistently, run multiple comparable trials, poll until the intended app reaches the foreground, and restore the user's previous app afterward.
- Label foreground-registration timings as such: they are not first-frame rendering times or physical-remote latency. Record cold versus warm startup and the sample count.

### 3. Propose a bounded change set

For each candidate list: evidence, current value/state, intended change, expected benefit, feature risk, temporary test, and rollback. Prefer supported settings over process/service changes.

Candidates, only where supported and relevant:
- Turn off optional watched-history collection and promotional playback; confirm existing Live Plus/promotion settings rather than blindly writing missing keys.
- Temporarily stop reviewed optional remote-diagnostics jobs after inspecting their definitions and dependencies on this exact firmware. Preserve `crashd` and all core services. The four jobs in the recipes are a reviewed example, not a universal denylist.
- Remove unnecessary delay from an existing startup helper without weakening boot/power readiness or causing foreground hijacking.
- Install the requested compatible YouTube fork in place, or map approved remote keys to its unchanged app ID.
- Adapt a requested app to the TV's browser engine and remote, then verify the rendered UI and actual playback.
- Renew an active Developer Mode session through LG, verify the timer, and schedule only the tested command with private token storage.
- Remove only explicitly selected apps, preserving exceptions. Treat hidden promotional inventory entries as unresolved until removal is verified.
- Diagnose mute and volume at both TV and player layers before changing output or resetting settings.

Stop and ask before expanding scope. An unidentified service stays enabled.

### 4. Back up and apply incrementally

Before the first write:
- Save original settings, whether each job was running, affected files with ownership/mode, app version/source, and existing mappings/hooks.
- Keep backups outside the overwritten application directory. Verify archives/checksums. Code backups alone are not account-data backups.
- Prepare an executable or precisely documented rollback that restores the saved values and starts only jobs that were originally running. Do not assume defaults.

Apply temporary changes first where supported; check the operation-specific response as well as command exit status. Luna and SSAP commonly return `returnValue`; the tested LG renewal endpoint used `result: "success"` and a separate remaining-time check. Test core features and the reported symptom before persisting anything. Preserve unknown settings and existing hooks. If a command times out or SSH disconnects, inspect live state before retrying a potentially completed operation.

For rooted TV maintenance, persist only approved, validated changes through the existing Homebrew hook mechanism. Use explicit PATH/absolute command paths, the TV's actual Python version, bounded waits, meaningful logs, and duplicate-run protection. A diagnostics cleanup worker should exit after boot cleanup, not become a resident polling daemon. A power-on app worker must not force the app back after normal app switching.

For unrooted Developer Mode deployments, use the authorized CLI workflow. Persist renewal only through the approved host scheduler described in [Developer Mode scheduling](references/developer-mode.md#schedule-and-verify-renewal). Do not install Homebrew or create root-only TV startup hooks for these tasks.

### 5. Verify and close the loop

- For scripts transferred to the TV, syntax-check with the TV's interpreter; verify checksums and permissions before atomic activation. Test TV startup helpers in a minimal environment, not just an interactive SSH session. For host-side renewal, test the command in the scheduler's runtime/environment and verify private token-file permissions, the saved schedule, and failure reporting instead.
- If a reboot is approved, perform one controlled full reboot, confirm a new boot ID, and verify the requested apps, persisted settings, internet, and essential services. On rooted TVs, additionally verify root/Homebrew, relevant hooks/logs, the native key hook if installed, and absence of Homebrew failsafe evidence. On unrooted Developer Mode TVs, verify the active session and requested sideloaded apps without requiring root/Homebrew.
- Repeat relevant measurements under comparable workload/cache conditions. Separate measured gains from expectations; report uncertainty and regressions honestly.
- Ask the user to test playback/audio, physical buttons, standby/wake, HDMI, accessibility as applicable, and casting. Inspector-injected keys do not prove physical keyboard behavior, and offline emulation must produce a real network failure before it can validate recovery. Process/API checks do not prove those experiences work.
- Leave the requested app/input in front and close only temporary test apps you opened.
- Update the private device checkpoint with changes, evidence paths, remaining user checks, limitations, and exact rollback steps. Keep device addresses, interface MACs, private renewal URLs, and local absolute paths out of public skill changes.

## Deliverable

Report succinctly:
1. What changed and what was deliberately preserved.
2. What was verified, with meaningful before/after measurements and their limits.
3. Backup/rollback location and how to use it.
4. The few remaining physical/user tests.

If the evidence supports no safe improvement, say so rather than adding tweaks.
