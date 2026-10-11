# Console setup and troubleshooting (AI-assisted)

This guide records the installation and startup verified on October 11, 2026.
The console reported firmware **12.020.000 / 12.02** in its native startup log.
It ran etaHEN experimental, ShadowMountPlus 1.7-beta5fix1 and the published
PS5SX2 2.02 app. This is evidence for that combination, not a compatibility
claim for every firmware or HEN.

## Release files

Download the app from [PS5SX2 2.02, tag vk-285-162](https://github.com/Swordpdf/PS5SX2/releases/tag/vk-285-162).
The standard release payload is `ps5sx2-aio.elf`. For the elevation failure
below, the separate `PS5SXHelper.elf` from [release 2.00, tag vk-285-139](https://github.com/Swordpdf/PS5SX2/releases/tag/vk-285-139)
worked with the 2.02 app; the app was not downgraded.

| File | Source release | SHA-256 |
| --- | --- | --- |
| `PS5SX2-vk-285-162.zip` | 2.02 | `6030dba6eca71c7d8ef2f1e4c0055ae0f665b3d61e7a33fc26262140be1c0de0` |
| `ps5sx2-aio.elf` | 2.02 | `b52c7a70c266826974314417bc059063b6e8ab0b8cc9537471f65f0e4915560d` |
| `PS5SXHelper.elf` | 2.00 | `737d95a8a1545447c4166b094005704465e420f79f54e882e30e7f9e4cde6210` |

On Windows, `Get-FileHash -Algorithm SHA256 -LiteralPath 'path-to-file'`
checks a downloaded asset. The release assets and helper matched the official
hashes, and Windows Defender found no threats. These checks establish download
integrity and the scan result; they do not prove console software is bug-free.

## Manual installation over FTP

1. Run the usual jailbreak and its homebrew support, including kstuff. Start
   ShadowMountPlus. If the HEN supplies kstuff already, use that existing setup.
2. Connect an FTP client to the PS5's LAN address on the port shown by its FTP
   service. The tested etaHEN server used port **1337** and anonymous login.
   Use binary transfers.
3. Extract the release ZIP. Inside its top-level release directory,
   `PPSA99203/` contains the app and `PCSX2/` contains its resources.
4. Upload `PCSX2/` to `/data/PCSX2/`. For a new installation, upload the complete
   `PPSA99203/` folder outside ShadowMountPlus's scan roots first, then move it
   to `/data/homebrew/PPSA99203/` after all files finish. Preserve existing BIOS,
   games, saves and changed settings when updating an existing installation.
5. The app folder must contain `eboot.bin`, `sce_module/libc.prx` and the complete
   `sce_sys/` directory, including `param.json`. Do not add an extra release-folder
   level underneath `/data/homebrew/`.
6. Wait for ShadowMountPlus to register the app. Its default scan interval is
   15 seconds and source-stability wait is 10 seconds. PS5SX2 should appear on
   the home screen or in Game Library.

Installation success was confirmed in `/data/shadowmount/debug.log` by
`AppInstallAll bridge consumed: PPSA99203 result=0x00000000` and `Installed NEW!`.
A read-only copy of the console's application database also contained its title
and icon. Copying files alone is not the registration check; do not edit the
application database manually.

### FTP executable readback

etaHEN's FTP `RETR` handler detects SELF headers and tries to decrypt them,
including an uploaded `libc.prx` or `eboot.bin`. In this session that download
stalled; renaming the file did not help because detection uses the header.
See [etaHEN's FTP implementation](https://github.com/etaHEN/etaHEN/blob/main/Source%20Code/util/source/ftp.c).

The 60 other release files were checked by SHA-256 readback. The two SELF files
were checked by successful binary upload and their stored sizes from `MLSD`;
their local sources came from the verified ZIP. Do not claim a raw remote SHA-256
comparison for SELF files when the server transforms or stalls the download.

## Helper and black-screen recovery

The console jailbreak, app registration and app elevation are separate steps.
ShadowMountPlus can register and launch the app without an elevation helper
answering PS5SX2's request.

Read `boot.log` before changing renderer settings. These were the failure stages:

| Log evidence | Meaning in the tested session |
| --- | --- |
| `etaHEN request: not taken within 1500 ms` | No compatible app helper answered the request. |
| `no acknowledgement within 10000 ms` | The app's Lapy elevation fallback had no answering daemon. |
| `daemon_held ... stage=donor_release error=92 ... attached=1 target_stopped=1` in `lapy-daemon.log` | The AIO daemon stopped during elevation and held the app suspended. This happened before drawing the interface or checking the BIOS. |

For that AIO failure, this recovery worked:

1. Restart the PS5 normally to clear the resident helper and stopped app.
2. Relaunch the usual jailbreak and ShadowMountPlus. Leave PS5SX2 closed.
3. Load the verified separate `PS5SXHelper.elf` from release 2.00 through the
   payload manager or an available ELF loader. The tested external loader
   accepted the ELF on **9021**. A loader restricted to console localhost must
   be reached through the on-console payload manager instead of a PC sender.
4. Confirm `/data/PS5SXHelper.log` reports `starting build=mountroot-v3`,
   the app-jailbreak listener, `/data mount hook installed` and
   `PS5SX2 Helper Loaded!`.
5. Launch PS5SX2. This helper reported `whitelist=built-in` for `PPSA99203` and
   `fully jailbroken`. A `/data/whitelist.txt` containing `PPSA99203` on its own
   line was also created, but the log's built-in match means the file was not
   established as necessary for this title.

Do not load both helper implementations together. In the tested installation,
the separate helper is saved at
`/data/pldmgr/payloads/PS5SX2/PS5SXHelper.elf`, with matching `.elf.json` metadata.
The failing AIO menu copy was renamed `ps5sx2-aio.elf.disabled`.
After future restarts, load the separate helper once after the jailbreak and
before opening PS5SX2. Automatic loading was not configured or tested.

### Automatic updates and remote logging in the AIO

The AIO combines an installer and a session-log uploader with the helper.
For this diagnostic run, empty files at these paths disabled installation,
patch downloads and remote log uploads before the AIO was started:

```text
/data/PCSX2/logs/installer-no-install
/data/PCSX2/logs/installer-no-log-upload
```

`installer.log` explicitly confirmed `install: switched off`,
`patches: switched off with the install` and `logger: switched off`.
The earlier standalone installer's switches use different paths:
`/data/PS5SX2-Installer/no-install` and `no-log-upload` in that same directory.
Use the paths belonging to the actual payload version, and verify its log.
The separate helper used for the successful launch does not run the installer.

## BIOS and games

The app can open with both folders empty. A missing-BIOS message and an empty
game shelf are expected then; neither means registration failed.

```text
/data/PCSX2/bios/ps2-0200a-20040614.bin
/data/PCSX2/games/My PS2 Game.iso
```

Use lowercase `bios` and `games`. `SCPH-70012.bin` in the message is an example,
not a required filename: the emulator reads the BIOS contents. Use your own
PS2 BIOS dump and game images; keep them outside source repositories.
The main BIOS is commonly 4 MiB; the parser accepts 4–8 MiB and validates its
ROM directory/version. Small `.NVM`, `.MEC` or `.ROM1` sidecars alone are not the
main BIOS. Supported game image formats include `.iso`, `.chd`, `.cso` and `.zso`;
the shelf can also list PS2 ELF homebrew.

After the first successful interface launch, a user-provided local
`ps2-0200a-20040614.bin` was validated as ROMVER `0200AC20040614` (USA v2.00,
June 14, 2004), copied to the BIOS path above and verified by SHA-256 readback.
The user then confirmed that PS5SX2 loaded the BIOS. A subsequent homebrew
startup test is recorded below. Close and reopen the app after adding files if
its display remains stale.

### Free homebrew test

[Dungeon Game by glampert](https://github.com/glampert/ps2-homebrew/tree/main/source/demos/dungeon_game)
is a free third-person dungeon crawler. The developer supplies a prebuilt ELF
with its assets embedded, so it needs no disc image or loose asset files.
The source is MIT licensed; preserve the supplied license with the download.

For this installation, `source/demos/bin/dungeon_game.elf` was downloaded from
commit `602441a6877a3136709d6320664340d52e3027a1` and copied to
`/data/PCSX2/games/dungeon_game.elf`, alongside `dungeon_game-LICENSE.txt`.
It is a 4,386,402-byte ELF32 little-endian MIPS executable. Its SHA-256 is
`25faee2f98483f7f86d0dd7043e7b506c998728989347c8a38ae8af49c0a1af4`;
the FTP readback matched, and Windows Defender found no threats.
The companion `/api/games` endpoint detected it as `dungeon_game`, and
`/api/state` subsequently reported `mode: game` with that ELF selected.
The app's native log recorded its ELF boot and about 60 fps at 100% emulation
speed over a short, roughly 24-second session. This verifies startup; extended
gameplay, controls and level completion were not independently checked.

Select `dungeon_game` from the shelf. Move with the left stick, attack with
Cross (X), and rotate the camera with the right stick. Triangle returns to
the game's menu. This is a **PS2** ELF: load it through PS5SX2, not the PS5's
port-9021 payload loader.

## Logs and acceptance checks

| Component | Log |
| --- | --- |
| App startup and elevation | `/data/PCSX2/logs/boot.log` |
| Settings and session history | `/data/PCSX2/logs/settings.log` |
| Emulator / error output, when created | `/data/PCSX2/logs/emulog.txt`, `stderr.log`, `pf.log` |
| Separate helper | `/data/PS5SXHelper.log` |
| AIO startup and elevation daemon | `/data/PCSX2/logs/payload-init.log`, `lapy-daemon.log` |
| AIO installer and logger state | `/data/PCSX2/logs/installer.log` |
| Registration and launch mounts | `/data/shadowmount/debug.log` |

If `logs/` does not exist, the app writes its logs directly in `/data/PCSX2/`.
Check both locations and the logged PID/build so an old session is not mistaken
for the current attempt.

The successful startup reported elevation in 23 ms, `JIT probe ok`,
`[frontend] up in 759 ms (ok)` and the first frame after 768 ms.
Its companion settings page returned HTTP 200 at `http://<PS5_IP>:8844/`.
The server tries the next seven ports if 8844 is occupied; use the address in
`boot.log` or the on-screen QR code. The user confirmed the visible missing-BIOS
screen first, then a successful BIOS launch after the local file transfer.

## OnionHEN relationship

The separate PS5SX2 helper is based on OnionHEN, but loading it is not the same
as loading the complete [tomchapin/onionHEN](https://github.com/tomchapin/onionHEN)
stack. That fork was cloned separately; it was not built or run in this test.
Its semantic `[app_jailbreak]` config uses `exact_title_ids` and
`title_id_prefixes`; do not assume the legacy helper's `whitelist.txt` applies
to the full stack. A full-stack migration needs its own compatibility test.
