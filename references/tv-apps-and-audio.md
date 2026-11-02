# Deploy usable apps on older webOS TVs

Use these checks when building or adapting a requested web app. Keep app development separate from system tuning. Preserve app IDs, existing sign-in state, root hooks, and approved remote mappings.

## Match the installed browser engine

1. Read LG's engine table and identify the actual app engine. The tested webOS 4.0 generation used Chromium 53. SDK, marketed webOS version, and firmware version are different identifiers.
2. Inspect the requested site's scripts, styles, API, and embedding policy. A modern site can fail before rendering because of unsupported syntax. `X-Frame-Options: DENY` or CSP `frame-ancestors` can also prevent an iframe wrapper.
3. Choose the smallest usable port. For the reference HackerTube client, ES5 JavaScript, bounded XHR requests, and flex layouts avoided unsupported optional chaining, async syntax, CSS Grid, flex gap, and `aspect-ratio`.
4. Request a bounded page of videos. Twelve cards per page worked on the reference TV; measure before treating that as a universal limit. Abort obsolete requests, ignore stale replies, validate API data, and render untrusted text through `textContent`.
5. Keep clear loading, empty, error, and retry states. Bound network waits and handle service throttling. Preserve focus after app switching.

### Keep packaged-app CSP narrow

In the tested file-based app, `script-src 'self'` and `style-src 'self'` alone blocked local scripts and styles. The working policy explicitly allowed `file:` for those resource types, while restricting API and image access to the app's required origins.

Diagnose the actual console error before changing policy. Do not remove CSP or allow arbitrary remote scripts. The reference manifest also used `vendorExtension.allowCrossDomain: true`; verify that behavior on the target platform and retain explicit origin restrictions in CSP. This observation is not proof that a normal website permits cross-origin requests.

## Show working shortcuts beside their controls

- Use large text, an obvious focus outline, safe screen margins, and predictable D-pad movement. Keep focused cards visible without scrolling unnecessarily on every horizontal move.
- Add small, rounded keycap badges to the relevant controls. Treat a mockup as a visual reference unless the user requests its actual feature, such as a Commands panel.
- Keep badge labels and keyboard dispatch in one mapping where practical. Avoid shortcut text leaking into page titles when reading a button's `textContent`.
- Pause action shortcuts while typing in inputs, textareas, or editable content. Preserve caret movement and modified keyboard shortcuts. Ignore held-key repeats for toggles.
- Keep shortcuts inside the active dialog. Back or Escape closes the dialog or returns through the app before invoking platform Back. Disable unavailable paging shortcuts.
- Show contextual Like and Save badges on the focused card. Those actions must affect that video, not a stale selection.

The following DOM key codes were verified on the reference TV and are consistent with LG's documented remote controls:

| Input | DOM key code |
| --- | --- |
| Left, Up, Right, Down | 37, 38, 39, 40 |
| OK | 13 |
| Back | 461 |
| Red, Green, Yellow, Blue | 403, 404, 405, 406 |

Short presses of number buttons produced the normal digit codes on the tested TV. Do not override LG's system Home key or assume long presses behave like short presses. Keys reserved by the system can vary with remote and firmware.

SSAP pointer-input button names differ from DOM key codes. In the tested transport, number button names were the strings `"0"` through `"9"`, not `"NUM_0"`. The latter produced unrelated events. Confirm the actual event before assigning a shortcut. Root-level native remapping is a different mechanism; see [remote mappings](webos-recipes.md#remote-mappings-and-local-only-configuration).

## Verify video handoff and playback

For the requested NicholasBly YouTube fork, the tested handoff used:

```json
{
	"id": "youtube.leanback.v4",
	"params": {
		"contentTarget": "https://www.youtube.com/tv?v=VIDEO_ID"
	}
}
```

Validate the video ID and confirm the installed player's current deep-link contract. Keep the player separate from the catalog app. A launch acknowledgement or foreground registration does not prove that the selected video is playing.

Inspect the exact intended app target. Verify the selected video ID, playback time advancing, ready state, pause state, mute state, volume, and media errors. Read decoded-audio counters when available. Ask the user to confirm audible sound and physical remote behavior; inspector-injected keyboard events do not prove a USB keyboard works.

## Diagnose silence before changing audio routing

1. Read the foreground app, TV mute state, volume, and sound output. With an authorized SSAP connection, the tested requests were `ssap://audio/getVolume` and `ssap://audio/getSoundOutput`.
2. If those requests return `401 insufficient permissions`, inspect the pairing manifest and the granted permissions. The tested connection required `CONTROL_AUDIO`. Use the normal pairing flow when needed; do not diagnose permission failure as an audio fault.
3. If the route is HDMI ARC, Bluetooth, or optical, confirm whether the user expects an external device. Preserve that route unless a change is requested. Do not silently switch a soundbar user to internal speakers.
4. Raise zero volume only to a modest level. Confirm the resulting volume and mute state. The reference TV accepted `ssap://audio/setVolume` with `{"volume":10}`. PyWebOSTV also documents `ssap://audio/setMute` with `{"mute":false}`. Check support and read the resulting state before declaring success.
5. Check the active player's `video.muted`, `video.volume`, pause state, errors, and available audio-decoding counters. TV-level mute and player-level mute are independent. A playing video with decoded audio can still be muted.
6. If the player is unintentionally muted, unmute it through its supported UI or authorized app-debugging session. Do not disable deliberate SponsorBlock mute segments or other intentional behavior. Observe whether it becomes muted again.
7. Verify the route and player state again, then ask whether the user can hear sound. If silence remains, use LG's sound test and inspect the external device's power, input, cable, and format support before proposing resets.

In the reference incident, HDMI ARC was correct, reported TV volume began at zero, and YouTube's video element was muted. A modest volume increase and player unmute restored sound, which the user confirmed. This is an observed cause, not a universal fix.

## Use inspector evidence carefully

- Resolve a fresh inspector target after reinstall or relaunch. Match the app ID or expected URL exactly; ignore unrelated targets.
- Keep asynchronous requests bounded. Some old inspectors did not complete the WebSocket close handshake. Stop only the helper process and port forward created for this task.
- Keep the same connection open for connection-scoped Network settings. Check actual traffic before trusting an emulation setting.
- The reference webOS inspector continued returning live HTTP 200 responses during offline emulation. A failed simulation did not prove broken app recovery. Mark offline recovery unverified until a real, scoped connection-loss test succeeds.
- Confirm rendered content with a fresh screenshot. Media overlays can be absent from TV screenshots, so a black capture alone does not prove failed playback.

## Sources

- [LG web engine versions](https://webostv.developer.lge.com/develop/specifications/web-api-and-web-engine)
- [LG remote controls and key codes](https://webostv.developer.lge.com/develop/guides/magic-remote)
- [NicholasBly YouTube fork](https://github.com/NicholasBly/youtube-webos)
- [PyWebOSTV pointer-button names and audio controls](https://github.com/supersaiyanmode/PyWebOSTV/blob/master/pywebostv/controls.py)
- [LG no-audio troubleshooting](https://www.lg.com/us/support/help-library/what-to-check-if-your-lg-tv-has-no-sound--20155311239786)
