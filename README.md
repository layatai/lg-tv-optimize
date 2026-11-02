# LG TV Optimize

An [Agent Skills](https://agentskills.io/) skill for **conservative, evidence-driven LG webOS TV optimization**.

It guides a coding agent through diagnosing responsiveness problems, making small reversible changes, and verifying the results without treating every background service or occupied megabyte as a problem.

## What it covers

- CPU, RAM, swap/zram, storage, boot, and app-switch diagnostics.
- Quick Start+ wake comparisons and reversible UI-switch disabling, with firmware-schema checks and careful black-screen diagnosis.
- Supported settings for optional LG collection and promotions.
- Review and temporary testing of optional diagnostics services.
- Compatible YouTube AdFree fork installation and updates.
- Developer Mode deployment, private-token renewal, and verified scheduling.
- Older-browser app compatibility, remote navigation, and visible shortcut badges.
- TV and player audio troubleshooting while preserving HDMI ARC or other chosen outputs.
- Exact app-selection cleanup and opt-in per-TV update filtering with connectivity checks.
- Boot/wake autostart and approved remote-button mappings.
- Backups, rollback, minimal-environment testing, and post-reboot verification.

## Safety by design

The skill starts read-only and requires scoped approval before changes. It preserves core services, HDMI, networking, Bluetooth, audio, accessibility, casting, and existing customizations.

It does **not** automatically root TVs, delete LG system binaries, resize swap, disable casting, change Quick Start+, block firmware updates, or promise universal ad blocking. It distinguishes measured foreground-registration timings from actual rendering and physical-remote latency.

This repository contains instructions and reference material, not an installer, root exploit, or unattended optimization script. The agent needs the user's existing authorized access. Root-only tasks require an already-rooted TV with Homebrew Channel and SSH; this skill does not establish that access.

## Install

### Pi

Clone into Pi's user skills directory, choosing another location if a skill with this name is already installed:

```sh
git clone https://github.com/layatai/lg-tv-optimize.git \
  ~/.pi/agent/skills/lg-tv-optimize
```

Run `/reload` in an existing Pi session, then invoke:

```text
/skill:lg-tv-optimize Diagnose my LG TV's slow menus and propose safe changes
```

Other example requests:

```text
/skill:lg-tv-optimize Inspect memory and swap without changing anything
/skill:lg-tv-optimize Install a compatible NicholasBly YouTube AdFree release
/skill:lg-tv-optimize Review my existing boot helper and remote mappings
/skill:lg-tv-optimize Inspect Quick Start+ and explain how to disable its UI switch safely
/skill:lg-tv-optimize Keep my active Developer Mode session renewed automatically
/skill:lg-tv-optimize Diagnose missing YouTube sound without changing my soundbar output
/skill:lg-tv-optimize Adapt my web app to this TV and show working shortcut badges
```

### Other Agent Skills-compatible agents

Place the repository in your agent's documented skills directory and load `SKILL.md`. For agents that support the shared location, a typical installation is:

```sh
git clone https://github.com/layatai/lg-tv-optimize.git \
  ~/.agents/skills/lg-tv-optimize
```

Skill discovery and invocation vary by agent. See your agent's documentation.

## Compatibility and limitations

The recipes were verified on an **LG 49UK7500PTA**, firmware **05.50.70**, SDK **4.4.0**, ARM32, with Python **2.7.12** and Upstart. They are a reference case, not proof that every command or service has the same meaning on every LG TV.

An agent must establish the actual model, firmware, capabilities, and user requirements before applying any recipe. Unsupported settings should be skipped rather than guessed. Video/audio quality, physical remote buttons, standby/wake, and casting still require user testing. Developer Mode renewal depends on an active session and LG's service. Local scheduled renewal also depends on its host running. Historical update domains are candidates to verify, not a universal denylist.

## Files

- [`SKILL.md`](SKILL.md): scope, safety boundaries, workflow, and reporting requirements.
- [`references/webos-recipes.md`](references/webos-recipes.md): APIs, firmware-specific examples, compatibility notes, and implementation pitfalls.
- [`references/quick-start-plus.md`](references/quick-start-plus.md): feature off versus UI disabled, native descriptor validation/rollback, display evidence, and controlled wake comparisons.
- [`references/developer-mode.md`](references/developer-mode.md): deployment, secure renewal, scheduler verification, and installation lifetime.
- [`references/tv-apps-and-audio.md`](references/tv-apps-and-audio.md): older browser engines, shortcut badges, media handoff, inspector limits, and audio diagnosis.
- [`references/app-cleanup-and-network.md`](references/app-cleanup-and-network.md): verified app removal, promotions, and scoped update blocks with rollback.

Keep personal TV addresses, SSH configuration, credentials, device profiles, logs, app backups, and recovery files **outside this checkout**. No personal device profile or SSH helper is included.

## Related projects

- [webOS Homebrew Channel](https://github.com/webosbrew/webos-homebrew-channel)
- [NicholasBly's YouTube webOS fork](https://github.com/NicholasBly/youtube-webos)

This is an independent community skill, not an official LG or YouTube product. Linked projects have their own licenses and compatibility requirements; their application packages are not bundled here.

## License

[MIT](LICENSE).
