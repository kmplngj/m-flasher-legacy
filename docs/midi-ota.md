# Native MIDI OTA (implementation slice of #15)

`m-flasher midi firmware PACKAGE.fwsc` is an explicitly selected FM-1 Felucca
OTA route. It does not enter ROM USB or silently switch to it. The running
firmware and the package's loader pull immutable requested ranges over SysEx.
No Python, mido, python-rtmidi, libusb, root helper or ROM download mode is needed
for this route. The optional native adapter uses CoreMIDI, WinMM or ALSA.

## Install and select

Successful **native MIDI CLI** Actions runs provide `m-flasher-midi-macos`
(universal Apple silicon/Intel), `m-flasher-midi-linux-x86_64` and
`m-flasher-midi-windows-x86_64`. These development artifacts are not releases.
Retain `source.json` with the executable; it records the source commit and SHA-256.
On macOS/Linux restore the downloaded executable's mode with `chmod +x m-flasher`.
Alternatively, with Rust installed, build the small CLI:

```sh
cargo build --release --locked -p m-flasher-midi
# Linux build prerequisite: libasound2-dev (or the distribution's ALSA headers).
target/release/m-flasher-midi --capabilities  # offline, no MIDI initialization
```

Cargo builds/installs the small CLI as `m-flasher-midi`, keeping it distinct from
the root GUI/ROM binary. The distributed artifact calls it `m-flasher`; it has
only MIDI operations. Prefix the commands below with `target/release/m-flasher-midi`
when running a source build.
The full GUI/ROM executable can include the same route with `--features midi`.
Default GUI/ROM builds remain unchanged; capabilities report `midi_ota: false`
when that adapter is absent. No application is run by the offline tests.

```sh
m-flasher midi info --port FM-1
m-flasher midi firmware build/felucca.fwsc --port FM-1
# If input/output names cannot be paired uniquely:
m-flasher midi firmware build/felucca.fwsc --in-port 'FM-1 input' --out-port 'FM-1 output'
```

Only one connected FM-1 may match. Other MIDI gear is never selected by default.
The exact port constraints remain in force during loader/reboot re-enumeration;
use a shared substring such as `FM-1` if names change. Ambiguous pairs are refused.
`--yes` skips the install confirmation; it does not skip file/device checks.

## Scope and pending hardware validation

Only FM-1 packages with the Felucca loader, its exact 0x93000-byte flash image,
valid FWSC/UFW tables/payload CRCs and valid loader headers/body are accepted.
There is no force option, package identity rewrite, stock detour, bootloader
operation, settings wipe or automatic ROM fallback. Firmware that refuses the
same numeric version times out honestly; Felucca versions permitting the same
version may install it. The post-reboot product/version reply cannot distinguish
PR commits sharing one version. For a same-identity exchange, the CLI explicitly
reports that identity cannot confirm replacement; retain the artifact provenance and inspect the
FM-1 ABOUT build commit.

This is ordinary OTA, not #15's fully qualified checkpoint-backed writer. It
creates **no raw backup or preboot full-image verification checkpoint**. The
running Felucca stages a loader at 0xE0000..0xE4FFF and an update record at
0xE4F00, then resets. The selected Felucca loader checks the chip key and writes
its application area 0x4000..0x92FFF; it may clear its update records. The host
does not prove physical readback, data preservation, rollback or bootability.
Make the backups required by your firmware/data transition separately. Do not
mistake a staging timeout for proof that nothing was written. After an interrupted
update, inspect the device state and rerun the same package to finish an active
loader; no automatic retry/recovery writes are initiated by the CLI.

Host evidence: independent synthetic package/transcript literals, injected MIDI
I/O and clock, range/framing/phase/send/disconnect/timeout/identity failures, and
compile-time denial of native MIDI in tests. These tests exercise no device and
execute no firmware. #15's checkpoint/transition gates and #16's external writer
qualification remain open. Real FM-1/macOS MIDI installation of this implementation
is **pending**, not established by an Actions build or merge.

For a separately authorized ordinary hardware test: connect one FM-1 via USB data
cable while Felucca is running; stop transport and notes, close other MIDI apps,
use the exact PR FWSC and invoke the command above. Confirm the printed identity
and package before answering `yes`. Expect loader handoff, writing, reboot and
matching product/version; hold HOME to open MENU, release, select SYSTEM with
ALGORITHM, ABOUT with PRESETS, then tap OCT+ and confirm the PR build commit.
Report the CLI source commit, firmware provenance, host OS, input/output names,
and observed result. Cancel at the prompt to check that no upgrade starts.
Do not use the owner's sole unit for injected writer-failure or recovery cycles.
