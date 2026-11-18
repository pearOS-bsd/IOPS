# IOPS

The power-source part of IOKit: `IOPowerSources.h` (`IOPSCopyPowerSourcesInfo`, `IOPSCopyPowerSourcesList`, `IOPSGetPowerSourceDescription`, `IOPSNotificationCreateRunLoopSource`, `IOPSGetProvidingPowerSourceType`, `IOPSCopyExternalPowerAdapterDetails`) and `IOPMLib` basics. On a Mac these symbols live in `IOKit.framework`; this repository holds that part separately so it can be built and tested on its own, and the `IOKit` repository re-exports it (same install name, same symbols).

**Status: planned. No code yet.**

## Layer
Layer 2 (needs CoreFoundation run loop sources) on top of layer 0 `IOKit` (user side). Used by: the battery menu extra (ControlCenter / SystemUIServer), `powerd`-style daemons, Energy Saver / Battery settings, apps that show the battery.

## What the real Control Center does (macOS 26.7.2)
Its startup trace (`research/cc-trace/real-controlcenter-startup-macos26.7.2.txt`) looks up `com.apple.PowerManagement.control` (powerd) and connects to `com.apple.powerui.smartChargeManager`; it links BatteryUIKit and PowerUI. Our replacement (`apps/controlcenter-cocoa/CCExtras.m`) shows the Battery menu-bar item when `IOPSCopyPowerSourcesList` returns at least one source (the Mac VM has none, so neither the real nor our item appears).
Dictionary keys the battery UI reads (to confirm on a real laptop): `kIOPSNameKey`, `kIOPSCurrentCapacityKey`, `kIOPSMaxCapacityKey`, `kIOPSIsChargingKey`, `kIOPSPowerSourceStateKey`, `kIOPSTimeToEmptyKey`, `kIOPSTimeToFullChargeKey`.

## Plan (FreeBSD backend)
1. Source of truth: `acpi_battery` / `acpi_acad`: `sysctl hw.acpi.battery.{life,state,time,units}`, `acpiconf -i 0`, `/dev/acpi` ioctls (`ACPIIO_BATT_GET_BIF`, `ACPIIO_BATT_GET_BST`), `apm(8)`; AC state from `hw.acpi.acline`. Devd events (`system=ACPI subsystem=CMBAT`/`ACAD`) drive notifications.
2. A power-source provider in `d` (the `powerd` role: Mach service `com.apple.PowerManagement.control`) so that the library, like Apple's, asks a daemon; the library part builds the CFDictionary descriptions.
3. Run-loop source for change notifications (`IOPSNotificationCreateRunLoopSource`) fed by devd through the daemon.
4. Compare with Apple: dump `IOPSCopyPowerSourcesInfo` / descriptions on a real MacBook (a friend's, as for Solarium) and on the VM (empty), REA of IOKit's `IOPS*` functions in the dyld cache.

## Existing code to reuse
`apps/controlcenter-cocoa/CCPlatform.c` (`ccp_battery_*`), `apps/topbar/ControlCenter.m` (`BatteryPct`, `Charging`).

## Rules for this repository
- Our code: BSD-2-Clause. Apple files are never copied here: reference material is read from the user's own Mac (`macdeps`), notes and measured values are written in our own words (`docs/`).
- No GPL/LGPL code. Upstream code, if any, only under `upstream/` with a `PROVENANCE.md`.
- Before writing a function: search the code index and `where-is.sh` (rule 3); do exactly what Apple does, with evidence recorded here (rule 4).
- Test on the FreeBSD VM; CI is the shared `hygiene` workflow (`pearOS-bsd/.github`).
