# b003fast minichlink Porting Log

This log tracks which local fork changes were ported from the old `ch32fun-fork`
history into the rebuilt `cnlohr/ch32fun` based fork. The current scope assumes
the bootloader firmware stays unchanged; this pass only changes host-side
`minichlink` behavior.

## Baseline

| Item | Ref |
|---|---|
| New working repo | `E:\codex\uiapduino\ch32fun-upstream-rebase` |
| Old reference repo | `E:\codex\uiapduino\ch32fun-fork` |
| New base branch | `lopple/rebuild-on-cnlohr` |
| Working branch | `work/b003fast-minichlink-stability` |
| Old mainline source | `ch32fun-fork custom/main` |
| Test bootloader assumption | Existing b003fast-compatible bootloader firmware |

## Porting Status

| Status | Old commit(s) | Feature | New commit | Reason / notes | Verification |
|---|---|---|---|---|---|
| Included | `7b90cc9` | `b003fast-test` programmer alias | Already in `lopple/rebuild-on-cnlohr` | Needed for pid.codes test PID `1209:000a`. | Build smoke only before this stability pass. |
| Included | `b88e3a0`, `ddcb03e` | Target-side CRC32 hash and `-V` write verify | Already in `lopple/rebuild-on-cnlohr` | Required for short post-write verification without full readback. | Build smoke only before this stability pass. |
| Included | `931584e`, `e18b5b8` | USER firmware feature reset into b003fast bootloader | `77f8591` | Required to make user-to-bootloader transitions usable without a manual reset. | [Stability report](b003fast-minichlink-stability-report.md) passed repeated USER-to-BL resets during 20-cycle write+CRC runs. |
| Included | `b404ee8`, `e18b5b8` | Bootloader re-enumeration and USER scan timing | `77f8591`, `e23fb95`, `ee83f6c` | Stabilizes HID open after reset and before writes. `-b` forces USER start mode, then succeeds after a short settle without requiring USER HID. | [Stability report](b003fast-minichlink-stability-report.md) passed default boot and explicit USER HID wait checks. |
| Included | `940b532` | Optional USER HID wait after `-b` | `5736525`, `37b872f`, `ee83f6c` | USER HID wait is now opt-in with `B003FUN_WAIT_USER_AFTER_BOOT=1`, because USER firmware may not expose HID. The opt-in timeout remains adjustable with `B003FUN_USER_SCAN_TIMEOUT_MS`. | [20-cycle opt-in run](b003fast-minichlink-stability-report.md) passed. |
| Included | `6ff1bb1`, `0a6da09`, `ca323d7` | Timing diagnostics gated by `MINICHLINK_TIMING` | `6ed554b` | Keeps logs quiet by default while preserving measurement detail. | Hardware timing output captured in the stability report. |
| Included | `156fe21`, `7b60fcc`, `00382f6` | HID send retry delay tuning | `d587ecf` | Previous hardware runs were stable with a short retry delay. | [Stability report](b003fast-minichlink-stability-report.md) passed 20/20 write+CRC with default retry delay. |
| Included | `dd06eca` | Fixed 340-byte b003fast-test reports and block writes | `ae647f5`, `b2e53c1` | The existing unchanged bootloader does not pass upstream scratchpad autodetection reliably, so only `b003fast-test` keeps the old fixed report size. The general upstream large-report path remains unchanged for other programmers. 192-byte block writes avoid the slow 64-byte fallback. | [Stability report](b003fast-minichlink-stability-report.md): `-i` passed and 9120-byte write+CRC is about `10.7 s`. |
| Deferred | `06dd931`, `e4bb916`, `43d8fe3`, `5e91f71`, `f5ea3f7`, `16c5e5e`, `69074f1` | RAM/vector write, RAM flasher, W128/W192 fast write | Not ported in this pass | Speed work is separated from the first stability release. Current upstream already has large-report support, so this needs fresh measurement. | Future experiment branch. |
| Deferred | `23edc3a` | Larger readback chunk | Not ported in this pass | Useful for readback speed, but not required for the stability release. | Future measurement. |
| Deferred | `29e426a` | GET omission experiment | Not ported in this pass | Related to fast-write protocol tuning; keep out of initial stability pass. | Future experiment branch. |
| Dropped | `5a3953e`, `03d80ae` | CRC during write | Not ported | Previously measured slower and removed from mainline. | Historical result only. |
| Dropped | `3bc83f0`, `2762d31` | All-FF page skip | Not ported | Removed from the old mainline because real uploads rarely benefit. | Historical result only. |
| Dropped | `f444f22` | USER direct open experiment | Not ported | The experiment was withdrawn in the old fork. | Historical result only. |
| Included | `13b626e` | Manual test release workflow | `7d59959`, `1389b13` | Adds a workflow_dispatch path and packages tag-named assets with a single archive root. The first Windows Board Manager test asset was published manually because `workflow_dispatch` requires the workflow file to exist on the default branch. | [Test release report](b003fast-minichlink-test-release.md) records the published asset, checksum, install result, and upload test. |
| Deferred | `73b09f4` | Third-party notices and final package metadata | Not ported in this pass | Final release contents and package-index URLs should be handled after hardware validation. | Future release/package-index work. |

## Test Release Gate

Before cutting a personal test release from this branch:

- Windows host `minichlink` build succeeds without committing generated binaries.
- `minichlink -C b003fast-test -i` reaches a protocol-level chip info response.
- `minichlink -C b003fast-test -H flash 9120` returns a target-side CRC32.
- `minichlink -C b003fast-test -V -w <user image> flash` writes and verifies.
- `minichlink -C b003fast-test -b` boots USER without requiring USER HID; `B003FUN_WAIT_USER_AFTER_BOOT=1` enables explicit USER HID confirmation.
- `write+CRC -> -b` passes 20 consecutive cycles when explicit USER HID confirmation is requested with `B003FUN_WAIT_USER_AFTER_BOOT=1`; see [stability report](b003fast-minichlink-stability-report.md).

## Not In Scope For This Stability Pass

- Bootloader firmware changes.
- D- disconnect timing changes.
- RAM/vector write and no-reply fast write.
- 512-byte or 276-byte report experiments.
- Streaming write, LZ compression, or write-time CRC integration.
- Arduino package index URL updates.
