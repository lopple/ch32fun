# b003fast minichlink stability verification

## Summary

- Branch: `work/b003fast-minichlink-stability`
- Host build: temporary Windows executable at `E:\codex\uiapduino\tmp\ch32fun-rebase-build\ch32fun-rebase-minichlink-stability.exe`
- Generated executable was not committed.
- Target: CH32V003 with the existing b003fast-compatible bootloader firmware.
- USER image: `E:\codex\uiapduino\reviews\uiapduino-bootloader-v141\artifacts\user-reboot-feature-uart-command\user_reboot_feature.ino.bin` (`9120` bytes)

## Commands

```text
argv: [minichlink] [-C] [b003fast-test] [-i]
argv: [minichlink] [-C] [b003fast-test] [-H] [flash] [9120]
argv: [minichlink] [-C] [b003fast-test] [-V] [-w] [user_reboot_feature.ino.bin] [flash]
argv: [minichlink] [-C] [b003fast-test] [-b]
```

## Results

| Check | Result | Notes |
|---|---:|---|
| Windows host build | pass | Built to a temporary path outside tracked files. |
| `-i` protocol health | pass | Reached chip info with fixed 340-byte reports. |
| `-H flash 9120` | pass | Returned `crc32=0x4e744587`. |
| Single `-V -w` after fixed-report block write | pass | `9120` bytes, CRC match, about `10.7 s`; block write part about `6.7 s`. |
| `write+CRC -> -b` with old 5 s default boot wait | fail | Reached cycle 16, then `-b` timed out at about `5.0 s`; write and CRC were still successful. |
| `write+CRC -> -b` with `B003FUN_USER_SCAN_TIMEOUT_MS=15000` | pass | 20/20 write+CRC and 20/20 boot-to-USER succeeded. |
| `write+CRC -> -b` with new default 15 s boot wait | pass | 20/20 write+CRC and 20/20 boot-to-USER succeeded without setting `B003FUN_USER_SCAN_TIMEOUT_MS`. |

## 20-cycle default run

- Log: `E:\codex\uiapduino\tmp\ch32fun-rebase-build\stability-20cycle-20260627-default15000.log`
- CRC result: 20/20 `crc32_match=true`
- Stale HID result: no stale HID failure in the passing run
- Write elapsed range: about `10.6 s` to `11.8 s`
- Block write time: about `6.6 s` to `6.7 s`
- Boot wait timeout setting: `15000 ms`
- Observed USER boot wait range: about `719 ms` to `2328 ms`

## Notes

- The existing unchanged bootloader did not accept upstream scratchpad-size autodetection reliably. `b003fast-test` therefore uses fixed 340-byte reports while leaving the upstream large-report path unchanged for other programmers.
- Fixed 340-byte reports alone fell back to 64-byte writes and took about `48 s` for the 9120-byte image. Enabling block writes with a 192-byte payload reduced the same write+CRC flow to about `10.7 s`.
- A reset-to-USER blob experiment was rejected: it left the target in bootloader HID when BOOT start mode had been forced externally. The adopted path keeps the existing run-app blob and only forces USER start mode before boot.