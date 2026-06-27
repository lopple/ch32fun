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
| Included | `931584e`, `e18b5b8` | USER firmware feature reset into b003fast bootloader | Pending | Required to make user-to-bootloader transitions usable without a manual reset. | Needs hardware `-i` / `-V -w` check. |
| Included | `b404ee8`, `e18b5b8` | Bootloader re-enumeration and USER scan timing | Pending | Stabilizes HID open after reset and after `-b`. | Needs repeated write/boot check. |
| Included | `940b532` | Default USER wait after `-b` | Pending | Prevents stale HID when the next write starts too soon after booting user firmware. | Target: 20/20 `write+CRC -> -b`. |
| Included | `6ff1bb1`, `0a6da09`, `ca323d7` | Timing diagnostics gated by `MINICHLINK_TIMING` | Pending | Keeps logs quiet by default while preserving measurement detail. | Check quiet default and timing-enabled output. |
| Included | `156fe21`, `7b60fcc`, `00382f6` | HID send retry delay tuning | Pending | Previous hardware runs were stable with a short retry delay. | Check no regression in repeated writes. |
| Deferred | `06dd931`, `e4bb916`, `43d8fe3`, `5e91f71`, `f5ea3f7`, `16c5e5e`, `69074f1` | RAM/vector write, RAM flasher, W128/W192 fast write | Not ported in this pass | Speed work is separated from the first stability release. Current upstream already has large-report support, so this needs fresh measurement. | Future experiment branch. |
| Deferred | `23edc3a` | Larger readback chunk | Not ported in this pass | Useful for readback speed, but not required for the stability release. | Future measurement. |
| Deferred | `29e426a` | GET omission experiment | Not ported in this pass | Related to fast-write protocol tuning; keep out of initial stability pass. | Future experiment branch. |
| Dropped | `5a3953e`, `03d80ae` | CRC during write | Not ported | Previously measured slower and removed from mainline. | Historical result only. |
| Dropped | `3bc83f0`, `2762d31` | All-FF page skip | Not ported | Removed from the old mainline because real uploads rarely benefit. | Historical result only. |
| Dropped | `f444f22` | USER direct open experiment | Not ported | The experiment was withdrawn in the old fork. | Historical result only. |
| Deferred | `13b626e`, `73b09f4` | Release packaging and notices | Not ported in this pass | Test release assets should be created after local hardware validation. | Future release branch/workflow. |

## Test Release Gate

Before cutting a personal test release from this branch:

- Windows host `minichlink` build succeeds without committing generated binaries.
- `minichlink -C b003fast-test -i` reaches a protocol-level chip info response.
- `minichlink -C b003fast-test -H flash 9120` returns a target-side CRC32.
- `minichlink -C b003fast-test -V -w <user image> flash` writes and verifies.
- `minichlink -C b003fast-test -b` waits for USER HID by default.
- `write+CRC -> -b` passes 20 consecutive cycles without stale HID.

## Not In Scope For This Stability Pass

- Bootloader firmware changes.
- D- disconnect timing changes.
- RAM/vector write and no-reply fast write.
- 512-byte or 276-byte report experiments.
- Streaming write, LZ compression, or write-time CRC integration.
- Arduino package index URL updates.
