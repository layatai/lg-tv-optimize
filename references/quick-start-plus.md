# Quick Start+ and display/wake diagnosis

Use this reference for **Quick Start+ / Khởi động nhanh+**, recurrent black screens around wake, or an explicit request to disable its menu switch. A general performance request does not authorize these changes. Turning the feature off and disabling its UI control are separate operations; confirm which the user wants.

The internal APIs and Settings-app UI binding below were inspected on an LG 49UK7500PTA, firmware 05.50.70, SDK 4.4.0. API/read-back verification succeeded; physical grey-out and reboot persistence were not verified in that reference observation. This is a reference case, not a universal LG policy API. Ordinary UI instructions do not require root. Descriptor writes require existing authorized root access and firmware-specific validation; do not root a TV just to grey out this switch.

## Black-screen evidence

Start read-only and establish when the failure occurs: only on power-on/wake, after an app/input switch, or during uninterrupted viewing. Keep the user's chosen HDMI/audio route and source-device state unchanged.

- Capture boot ID, uptime, foreground app, power state/reason, relevant display/service errors, and the current Quick Start+ value. Preserve focused evidence before a reboot clears volatile logs; do not collect credentials or account state.
- Distinguish logical state from physical output. `Active`, backlight 100, a connected video pipeline, an app screenshot, or a successful Home-key acknowledgement does not prove the LCD is lit or a native menu was visible. Inspector screenshots can omit hardware video planes.
- If an authorized native-menu test is invisible, ask the user what they actually see. A black HDMI source alone is not a conclusive explanation, but an acknowledged command alone does not prove a menu rendered either.
- If an LCD remains black, a phone flashlight held close at an angle can reveal a faint picture. A faint image supports a backlight/LED-driver fault; no image is inconclusive. Do not open the casing. Recommend professional service when the evidence supports a hardware fault.
- Treat silence separately: TV volume, player mute, and external-device volume/power are independent. Preserve ARC or the selected output; do not switch to internal speakers silently.
- Unsupported RPC methods, undocumented numeric enums, and generic boot flags such as `snapshot-resume-done` are not hardware diagnoses or proof that Quick Start+ caused a failure.

A full reboot restoring the picture is consistent with a transient display/driver/wake state, but does not establish Quick Start+ as the cause or exclude intermittent hardware. Likewise, do not blame root, privacy settings, or a particular app without evidence. If the failure also occurs on another input with a custom launcher disabled, that launcher alone is not a sufficient explanation.

## Turn off the feature

Quick Start+ keeps the TV in a standby mode intended to make the next startup faster; the reference Settings help also warns of higher possible energy use. Turning it off can make startup slower. Other enabled features can affect the actual standby path, so do not assume an off switch always guarantees a particular kernel boot path.

On the reference TV, the UI path is:

**Settings → All Settings → General → Quick Start+ → Off**

Vietnamese: **Tất cả cài đặt → Cài đặt chung → Khởi động nhanh+ → Tắt**.

Menu nesting varies by model and firmware. Read the live setting before a remote write:

```sh
/usr/bin/luna-send -w 5000 -n 1 -f \
  luna://com.webos.settingsservice/getSystemSettings \
  '{"category":"option","keys":["quickStartMode"]}'
```

After backing up the original value and obtaining scoped approval, change only this supported key if needed:

```sh
/usr/bin/luna-send -w 5000 -n 1 -f \
  luna://com.webos.settingsservice/setSystemSettings \
  '{"category":"option","settings":{"quickStartMode":"off"}}'
```

If it is already off, preserve it rather than writing again. Check `returnValue` and read the value back. Ordinary feature off does **not** mean the switch is disabled in the UI.

### Compare normal wakes

Keep other settings, apps, hooks, HDMI devices, and audio routing constant. On the user's next normal off/on cycles, record picture/menu visibility and boot ID/uptime when available. Include a standby interval comparable to normal use. Obtain approval before remotely switching power, inputs, or apps; do not reboot merely to force a test.

Several stable wakes with Quick Start+ off support a resume-related hypothesis, not an absolute causal proof. Record the number of cycles and observation period. If the failure recurs with it off, investigate other causes. Do not re-enable it merely to provoke another blackout unless the user explicitly wants that comparison.

## Disable the UI switch

This is a separate, root-only, internal-metadata operation—not a documented consumer lock feature. Use it only when the user explicitly requests a greyed-out switch and the actual firmware exposes the needed API and UI binding.

### Read and back up

```sh
/usr/bin/luna-send -w 5000 -n 1 -f \
  luna://com.webos.settingsservice/getSystemSettingDesc \
  '{"category":"option","keys":["quickStartMode"]}'
/usr/bin/luna-send -w 5000 -n 1 -f \
  luna://com.webos.settingsservice/getSystemSettingValues \
  '{"category":"option","key":"quickStartMode"}'
```

1. Verify each response and the exact category/key. Distinguish `quickStartMode: "off"` (feature value) from `ui.active: false` (control disabled) and `ui.visible: true` (control still shown).
2. Save the complete value and descriptor replies privately, along with enough context to detect concurrent changes. Keep a verified host copy and, where practical, a protected TV-side recovery copy outside active hook directories. Keep all personal evidence outside the public skill checkout.
3. Verify that this firmware's Settings app actually honors `ui.active` for this control. On the reference app, its disabled-state calculation includes `!descriptor.ui.active`. Inspect relevant source or authoritative firmware-specific documentation read-only; do not patch the app to manufacture this behavior.
4. Inspect the actual method and accepted schema. A getter's returned fields are not automatically writable. On the reference firmware, introspection exposed `setSystemSettingDesc`; read-only inspection of its running SettingsService's embedded schema resolved an initial schema rejection. Do not modify/copy entire system binaries or bypass access controls for discovery.
5. If the request is both off and greyed out, verify the feature is off before the descriptor write. If it needs changing, use the separately approved feature operation above. Immediately before writing, re-read and compare against the backup; stop if newer work appeared.

### Build a narrowly scoped payload

Call `luna://com.webos.settingsservice/setSystemSettingDesc` through the existing authorized transport. Construct the payload from the validated backup, preserving its UI attributes, value type, allowed values, and validation behavior; change only `ui.active` to false.

The following is the **reference payload shape**, not defaults to paste onto an uninspected TV:

```json
{
  "category": "option",
  "key": "quickStartMode",
  "ui": {
    "active": false,
    "visible": true,
    "displayName": "quickStartMode",
    "widget": "N"
  },
  "vtype": "ArrayExt",
  "values": {
    "arrayExt": [
      {"value": "on", "active": true, "visible": true},
      {"value": "off", "active": true, "visible": true}
    ]
  },
  "valueCheck": true
}
```

The reference setter accepted `category`, `key`, `ui`, `vtype`, `values`, and `valueCheck` in this payload. Do **not** send the entire getter reply: read-back fields such as `dbtype`, `dimension`, and `condition` were not accepted by that firmware's setter schema. Do not alter the storage/volatile flags, allowed on/off values, or factory descriptors to enforce this UI restriction.

The service may expose the default global scope as an additional `app_id: ""` after writing. On this verified global key, that was benign normalization, not a per-app override. Compare it explicitly: permit absent versus empty only when global scope is independently established and all other semantics match. Never ignore a nonempty app ID or unrelated difference. Do not copy an empty `app_id` into a request: this firmware required a nonempty string if that field was supplied.

### Verify and handle errors

- Require API `returnValue: true`, then re-read both value and descriptor immediately and after a short delay. Verify feature off, UI inactive/visible, original allowed values, validation, conditions, and other metadata semantics intact.
- A schema rejection is not permission to guess repeatedly. Read current state first and inspect this firmware's accepted fields. If nothing changed, do not issue an unnecessary rollback write. On disconnect or ambiguous acknowledgement, inspect before retrying.
- If a successful write produces unexpected changes, restore the saved supported descriptor fields and verify the result before doing anything else. Do not silently substitute `setSystemSettingFactoryDesc`, direct database edits, a Settings-app patch, hotel mode, or a continuous setting-forcing worker.
- Ask the user to close/reopen Settings and confirm the switch is greyed out and cannot be selected. Metadata read-back and source inspection establish expected behavior, not physical visual confirmation.
- Native metadata is saved through the settings service; no new Homebrew persistence hook is needed for this operation. Verify survival across a later user/approved reboot before claiming reboot persistence. Firmware, reset, or policy changes may replace it.
- This only restricts the normal Settings UI. It is **not a security boundary** against root or another privileged API caller. Do not promise permanent enforcement or a proven black-screen repair.

### Roll back the UI restriction

Re-read live state first and check for newer changes. Build a setter payload from the original backed-up **supported fields**, restoring its original UI activity/visibility and other UI attributes. Apply it with `setSystemSettingDesc`, check the response, then read both descriptor and feature value back.

For the reference case, restoring `ui.active: true` restored the control's active flag while leaving `quickStartMode: "off"`; physical UI confirmation remained pending. Unlocking the UI does not require turning the feature on, changing YouTube startup, restarting a service, or rebooting. Restore the feature value separately only if that broader rollback is requested. Never copy unsupported read-back fields into the restore payload.

## Keep YouTube startup independent

A custom YouTube power-on/wake hook does not require Quick Start+ to be on. Preserve the user's chosen Quick Start+ value when enabling, disabling, or restoring that hook. See [autostart lifecycle](webos-recipes.md#power-onwake-autostart).

Report these checks separately: hook enabled/executable, worker running, new-boot log of a successful launch, foreground registration, and physically visible video/audio. Native LG last-input/app restoration can resume YouTube without the custom worker; a foreground app alone does not prove the hook ran. Startup logs do not prove first-frame timing, a working panel, or a black-screen cause.
