# b003fast minichlink Test Release

## Release

| Item | Value |
|---|---|
| Release | `b003fast-test-20260627.1` |
| URL | https://github.com/lopple/ch32fun/releases/tag/b003fast-test-20260627.1 |
| Source branch | `test/b003fast-minichlink-preview` |
| Source commit | `1389b1376c98ef962860dfabdedeed4ddc85eb77` |
| Scope | Windows Board Manager validation asset only |

## Assets

| Asset | Size | SHA-256 |
|---|---:|---|
| `minichlink-b003fast-test-20260627.1-windows-x86_64.zip` | `228612` | `77321d420228e364ec3b72ef35cb8d379a523b737ebd811a5eaf0ff27bf15941` |
| `SHA256SUMS-windows.txt` | `122` | `3a178ad463ac546a2cd525e92eb3915dcf88c0f59190feb59abab51534e766f4` |

The downloaded zip was verified after publication. Its archive root is:

```text
minichlink-b003fast-test-20260627.1-windows-x86_64/
```

The root contains `minichlink.exe`, `libusb-1.0.dll`, `README.md`, `metadata.json`, and `LICENSE`.

## Local Build Evidence

The Windows executable was built locally with MSYS2 MinGW from the source commit above. The staged executable reports:

```text
minichlink version - b003fast-test-20260627.1-1389b13
```

Dependency inspection showed only one non-system runtime DLL requirement:

```text
DLL Name: libusb-1.0.dll
```

`libusb-1.0.dll` is included in the archive root so the Arduino tool does not depend on the developer PATH.

## Board Manager Test Index

A temporary package index was created at:

```text
E:\codex\uiapduino\tmp\b003fast-test-release-20260627-1\package_uiap_b003fast_test_index.json
```

The test index keeps the existing core dependency name and version:

```text
UIAP:minichlink-2982dfd@1.0.0
```

Only the Windows system entry was redirected to the test release asset. This is required because the current installed `platform.txt` refers to `runtime.tools.minichlink-2982dfd-1.0.0.path` directly.

## Install Result

Arduino CLI installed the test package in a dedicated data directory:

```text
E:\codex\uiapduino\tmp\b003fast-test-release-20260627-1\arduino-cli-data
```

Installed tool path:

```text
E:\codex\uiapduino\tmp\b003fast-test-release-20260627-1\arduino-cli-data\packages\UIAP\tools\minichlink-2982dfd\1.0.0\minichlink.exe
```

The installed tool worked with the explicit b003fast programmer selection:

```text
minichlink -C b003fast-test -i
LASTEXITCODE=0
```

The same installed tool failed without `-C b003fast-test` because the default programmer path probes the old `1209:b003` PID, not the current `1209:000a` test PID.

## Upload Test

The installed core's temporary `platform.txt` was patched only inside the test data directory:

```text
tools.minichlink.upload.pattern.windows="{path}{cmd}" -C b003fast-test -V -w "{build.path}\{build.project_name}.bin" flash
```

With that patch, Arduino CLI upload using an existing binary passed:

```text
arduino-cli upload --fqbn UIAP:ch32v:CH32V00x_EVT:pnum=CH32V003V1DOT4,upload_method=minichlink --input-file <user_reboot_feature.ino.bin>
```

Result:

```text
Image written.
verify_crc32 expected=0x4e744587 actual=0x4e744587
crc32_match=true
LASTEXITCODE=0
```

## Findings

- The minichlink test release asset itself is usable through Board Manager on Windows.
- The current UIAPduino core upload recipe is not sufficient for the `b003fast-test` PID because it does not pass `-C b003fast-test`.
- A core or package-index test variant must add the programmer selection before normal Board Manager upload testing can pass without manual patching.
- `arduino-cli compile` hit an unrelated `riscv-none-embed-ar` rename failure on this machine: `unable to rename core.a; reason: Permission denied`. Upload was validated separately with `arduino-cli upload --input-file`.

## Next Step

For a Board Manager preview that users can install without manual patching, prepare a test core/package-index variant that updates the minichlink upload pattern for b003fast and avoids changing the existing stable `minichlink-2982dfd@1.0.0` entry in-place.
