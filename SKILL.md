---
name: lg-tv-optimize
license: MIT
description: Diagnose and conservatively optimize LG webOS TVs for responsiveness, startup speed, privacy, and background activity. Use for a slow LG TV, webOS CPU/RAM/swap analysis, reducing LG promotions or diagnostics, YouTube AdFree installation or autostart, remote-button remapping, and root-safe Homebrew maintenance. Requires measurement, backups, scoped approval, feature preservation, and rollback; never blindly disables services or changes swap.
compatibility: LG webOS TVs; read-only diagnosis can use existing authorized access. Root-only changes require an already-rooted TV with Homebrew Channel and SSH. Older TVs may run Python 2.7 and Upstart.
---

# LG TV optimization

Act as a conservative webOS maintenance engineer. Diagnose the user's actual bottleneck, propose the smallest useful change, and keep it reversible. This skill is not permission to root a TV, disable arbitrary services, or apply every recipe.

Resolve reference paths relative to this skill directory. Read [webOS recipes](references/webos-recipes.md) before executing relevant TV commands. Discover each TV independently and explicitly confirm the intended target; never silently reuse a saved IP. Keep per-device profiles, credentials, evidence, and recovery files outside this public skill checkout.

## Non-negotiable safety boundaries

- Start read-only. Explain proposed writes and obtain approval before applying a new plan, installing software, persisting hooks, or rebooting. An explicit request to install a named app or implement an already-approved plan authorizes that scoped work, not unrelated changes.
- Preserve HDMI, networking/DNS/proxy support, Bluetooth, audio, accessibility, casting, application management, the compositor, system IPC, security services, and local crash handling. Preserve user apps and existing customizations.
- AirPlay/HomeKit removal, Quick Start+ changes, firmware-update blocking, and DNS/network-wide blocking are separate opt-in decisions. Explain their feature/security trade-offs first.
- Do not delete/rename LG system binaries, remount system partitions writable, factory-reset, casually revoke required Smart TV agreements, or disable services based only on their names.
- Do not `swapoff`, resize live zram, add internal-flash swap, drop caches as an optimization, or kill the active renderer to manufacture free RAM. Investigate existing swap composition first.
- On a rooted Homebrew TV, do not reinstall LG Developer Mode or “repair” the intentionally directory-shaped `/var/luna/preferences/devmode_enabled`. Rooting/re-rooting needs a separate, device-specific request and current compatibility research.
- Use existing authorized SSH access. Never print/copy passphrases, private keys, cookies, pairing tokens, or complete credential configuration. Do not disable host-key verification or expose root/configuration interfaces to the internet.
- Never promise a universal ad blocker, permanent RAM recovery from a snapshot, or a large speedup without measurements.

## Workflow

### 1. Establish scope and live identity

1. Identify the symptom: slow boot, wake delay, menus, app switching, video buffering, crashes, or privacy concerns. Do not confuse network buffering with CPU or memory pressure.
2. Confirm target IP/name, model, firmware, SDK/webOS generation, available storage, authorized account, and whether root/Homebrew already exists. Reuse verified access rather than reinstalling tools.
3. Ask what must keep working if not already clear. Keep casting and Quick Start+ by default.
4. Re-read live versions and configuration even when a previous checkpoint exists. Historical versions can differ from the installed files.

### 2. Measure before changing

Create a private, timestamped local evidence directory. Capture boot ID/uptime, foreground app, CPU over a real interval, memory and swap, storage, relevant service states, settings, app metadata, and existing hooks/mappings. Avoid collecting credentials or account state.

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

Stop and ask before expanding scope. An unidentified service stays enabled.

### 4. Back up and apply incrementally

Before the first write:
- Save original settings, whether each job was running, affected files with ownership/mode, app version/source, and existing mappings/hooks.
- Keep backups outside the overwritten application directory. Verify archives/checksums. Code backups alone are not account-data backups.
- Prepare an executable or precisely documented rollback that restores the saved values and starts only jobs that were originally running. Do not assume defaults.

Apply temporary changes first; check API `returnValue` as well as command exit status. Test core features and the reported symptom before persisting anything. Preserve unknown settings and existing hooks. If a command times out or SSH disconnects, inspect live state before retrying a potentially completed operation.

Persist only approved, validated changes through the existing Homebrew hook mechanism. Use explicit PATH/absolute command paths, the TV's actual Python version, bounded waits, meaningful logs, and duplicate-run protection. A diagnostics cleanup worker should exit after boot cleanup, not become a resident polling daemon. A power-on app worker must not force the app back after normal app switching.

### 5. Verify and close the loop

- Syntax-check transferred scripts with the TV's interpreter; verify checksums and permissions before atomic activation. Test startup helpers in a minimal environment, not just an interactive SSH session.
- If a reboot is approved, perform one controlled full reboot, confirm a new boot ID, and verify root/Homebrew, persisted settings, hooks/logs, native key hook, internet, and essential services. Check for Homebrew failsafe evidence before declaring success.
- Repeat relevant measurements under comparable workload/cache conditions. Separate measured gains from expectations; report uncertainty and regressions honestly.
- Ask the user to test playback/audio, physical buttons, standby/wake, HDMI, accessibility as applicable, and casting. Process/API checks do not prove those experiences work.
- Leave the requested app/input in front and close only temporary test apps you opened.
- Update the device checkpoint with changes, evidence paths, remaining user checks, limitations, and exact rollback steps.

## Deliverable

Report succinctly:
1. What changed and what was deliberately preserved.
2. What was verified, with meaningful before/after measurements and their limits.
3. Backup/rollback location and how to use it.
4. The few remaining physical/user tests.

If the evidence supports no safe improvement, say so rather than adding tweaks.
