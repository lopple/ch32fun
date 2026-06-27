# b003fast upstream/master port

## Baseline

| Item | Value |
|---|---|
| Base | `upstream/master` |
| Base commit | `0b6473318c26c906be97e7100a5ce3755bd3909b` |
| Working branch | `work/b003fast-upstream-master` |
| Scope | Host-side minichlink support only |

This branch intentionally ports the b003fast host-side changes directly on top of `upstream/master`.
It does not include brand-specific udev rules, Board Manager package edits, or bootloader firmware changes.

## Included

| Area | Notes |
|---|---|
| Bootloader selection | Adds `b803boot` and `b003fast-test` programmer aliases. |
| Target CRC32 | Adds `-H` target-side CRC32 and `-V` write verification. |
| USER-to-boot reset | Allows the b003fast test path to request bootloader reset from compatible USER firmware. |
| Re-enumeration stability | Adds bootloader/user HID scan timing controls and quiet-by-default timing diagnostics. |
| Fixed-report writes | Keeps the b003fast test path on the fixed 340-byte report behavior with 192-byte payload writes. |
| Boot to USER | `-b` sends the boot command, closes the bootloader HID handle, waits a short settle period, and does not require USER HID by default. USER HID wait remains opt-in. |

## Excluded

| Area | Reason |
|---|---|
| Brand-specific udev rules | Not needed for this upstream/master based test release. |
| Old release workflow/docs | The previous test release was based on a different branch and is not reused. |
| Board Manager package edits | Handled separately after this release asset is available. |
| Bootloader firmware changes | Out of scope for this host-tool test release. |

## Local validation

Built on Windows with MSYS2 MinGW from this branch.

```text
minichlink version - b003fast-upstream-master-local
```

Hardware checks against the connected CH32V003 target:

```text
minichlink -C b003fast-test -i
LASTEXITCODE=0
```

```text
minichlink -C b003fast-test -H flash 9120
crc32=0x4e744587 address=0x08000000 length=9120
LASTEXITCODE=0
```

```text
minichlink -C b003fast-test -V -w user_reboot_feature.ino.bin flash
verify_crc32 expected=0x4e744587 actual=0x4e744587
crc32_match=true
LASTEXITCODE=0
```

```text
MINICHLINK_TIMING=1 minichlink -C b003fast-test -b
b003fast_boot_settle_ms=500
b003fast_boot_user_wait_enabled=0
LASTEXITCODE=0
```

## Release requirements

The replacement release asset should:

- be built from this upstream/master based branch,
- keep `minichlink.exe` under `bin/`,
- include required runtime DLLs under `bin/`,
- avoid brand-specific names in the release tag, asset names, archive contents, metadata, and release notes,
- publish exact SHA-256 and byte size for downstream package-index work.

## Test release

| Item | Value |
|---|---|
| Release | `b003fast-upstream-20260627.1` |
| URL | https://github.com/lopple/ch32fun/releases/tag/b003fast-upstream-20260627.1 |
| Type | GitHub prerelease |
| Target commit | `b536f20ccfa83d4a15636a6fd37f55110da92ccc` |
| Windows asset | `minichlink-b003fast-upstream-20260627.1-windows-x86_64.zip` |
| Windows asset size | `228640` bytes |
| Windows asset SHA-256 | `66c3213bd23e23d20d027eae8eec9fb0235245e19a09a7ee521aeb8078a55aa4` |
| Checksum asset | `SHA256SUMS-windows.txt` |

Downloaded asset verification after publishing:

```text
minichlink-b003fast-upstream-20260627.1-windows-x86_64.zip 228640 bytes
sha256: 66c3213bd23e23d20d027eae8eec9fb0235245e19a09a7ee521aeb8078a55aa4
```

Published archive layout:

```text
minichlink-b003fast-upstream-20260627.1-windows-x86_64/LICENSE
minichlink-b003fast-upstream-20260627.1-windows-x86_64/metadata.json
minichlink-b003fast-upstream-20260627.1-windows-x86_64/README.md
minichlink-b003fast-upstream-20260627.1-windows-x86_64/bin/libusb-1.0.dll
minichlink-b003fast-upstream-20260627.1-windows-x86_64/bin/minichlink.exe
```


