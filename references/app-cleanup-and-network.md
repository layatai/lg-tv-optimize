# Remove optional apps and scope network blocks

Use these procedures only for the user's requested cleanup or update-blocking task. They are separate from memory tuning. Do not infer approval to remove an app or block updates from a general request for a faster TV.

## Inventory and remove selected apps

1. Capture installed app IDs, versions, titles, launcher entries, and the user's exact selection. Preserve explicit exceptions such as YouTube. A display name alone can be ambiguous.
2. Explain that uninstalling can remove local sign-in state or settings. Keep app backups and account-data limitations separate.
3. Choose the supported removal route for the current access. Developer Mode's prisoner account is not root, and a rejected Luna uninstall call does not justify escalating privileges or deleting app files.
4. When a removable app blocks remote uninstall, use the TV's native Home edit mode or installed-app management UI. Navigate through screenshots and the authorized remote transport. If a confirmation needs the physical remote, ask the user to complete it.
5. Re-query the installed-app list and launcher entries after each removal. Verify exact IDs against the approved selection and check that preserved apps remain installed.
6. Treat promotional shortcuts separately. An entry can remain in the app inventory while having no launcher tile or installed-store entry. Record that state as unresolved or hidden, not successfully removed. Do not delete a system directory to make the inventory look clean.

Read-only Developer Mode access and authorized SSAP remote control can complement each other. SSH can work while installed-app queries through that account are restricted. SSAP operations still require the relevant pairing permissions; never use a broad permission failure as proof that the feature does not exist.

## Disable optional promotions through supported settings

Read the relevant categories first. In an earlier Developer Mode session on the reference firmware, the supported settings included `general.homePromotion` and `option.livePlus`. Both were changed from `on` to `off` and read back successfully. A later checkpoint can have different values. Preserve already-disabled settings.

Use the settings-service API and exact category/key names verified on that TV. Store the original values privately and restore those values for rollback. These switches do not prevent every promotional app or firmware update.

## Block updates only at the requested scope

1. Separate firmware auto-update settings, app-store updates, promotional content, and network filtering. Explain which behavior each proposed change affects. Blocking firmware also delays security fixes.
2. Start with the TV's supported settings. Read and retain an already-disabled automatic-update setting rather than toggling it.
3. If the user requests network filtering, identify their actual router or DNS blocker and its supported per-device policy. Scope the rule to the intended TV, including wired and Wi-Fi interfaces when both are used. Check IPv6 and alternative DNS paths.
4. Save the existing rule state and a rollback that disables only the new rule. Preserve other clients, router administration, DHCP, DNS, streaming, casting, and the Developer Mode renewal endpoint.
5. Have the user sign in privately. Hand browser certificate warnings to the user. Never request router passwords in chat or bypass certificate validation.
6. Inspect the router's field constraints and policy meaning. Use valid `hh:mm` times within `00:00` to `23:59` when required. An enabled all-day schedule and the correct allow/block policy may be needed before a domain filter applies. Do not guess from a confusing label.
7. If a validation dialog blocks automation, ask the user to dismiss it and re-read the form. An unsaved or rejected form is not an active rule. Read back saved rules and capture the verified state.

### Treat domains as candidates, not a permanent denylist

The historical candidate set tested for the reference TV was:

```text
snu.lge.com
su.lge.com
su-ssl.lge.com
snu-dev.lge.com
su-dev.lge.com
nsu.lge.com
lgappstv.com
```

Do not apply this list automatically to another network. Verify present behavior and the router's matching semantics. The `lgappstv.com` entry can impair LG's app store and app installation or updates; make that trade-off explicit. Preserve `developer.lge.com` when Developer Mode renewal is required. Do not broadly block `lge.com` or all LG traffic.

### Verify from the TV and a control client

Test a required streaming endpoint, DNS resolution, the Developer Mode endpoint, and the intended update/store endpoints from the TV. Compare update endpoints from an unaffected client to distinguish a scoped block from a server outage. Record HTTP, DNS, or connection outcomes separately.

In the observed case, an incorrect policy unexpectedly blocked all TV DNS. Disabling that new rule restored connectivity before further changes. The corrected per-device policy allowed streaming and Developer Mode traffic while selected firmware/store probes failed from the TV and succeeded from the control client.

That evidence did not prove coverage of future hostnames, direct-IP downloads, every IPv6 route, or persistence across reboot. Report those limits. Do not describe a small domain list as a complete or permanent update block.

If normal connectivity regresses, disable the new rule first and verify recovery. Do not replace the user's DNS service or block the entire TV as a shortcut.

## Related guidance

- [Developer Mode renewal and its network dependency](developer-mode.md)
- [Audio routing and app verification](tv-apps-and-audio.md)
- [Existing settings and root-safe recipes](webos-recipes.md)
