# Changelog

## v1.1.1 (Build 27) — community pre-release

Same as v1.1.0, plus:

- **Xbox GIP rumble holds until the next level.** Rumble packets used repeat count `0x00`, so
  the motors ran for one 2.55 s period and then stopped while the game still asked for rumble.
  They now use `0xFF`, as the Linux xpad driver does. *Console-tested* with a wired Xbox
  Series X|S (FW 10.60).
- **"Test rumble" button for USB pads** in Controller settings, backed by
  `POST /api/usb/rumble_test?slot=N`. It runs the motors for 1 s, ignoring game rumble
  meanwhile. *Console-tested* with a wired Xbox Series X|S.
- **Diagnostics:** the log reports game rumble from the haptics capture (every 5 s in which
  some arrives) and each rumble level sent to a USB pad (at most once a second, plus every stop).

### Known issue: game rumble does not reach USB pads
Game rumble for USB pads depends on the Avcap2 haptics capture from PR #18 ("Game rumble via
Avcap2" in Settings). With a wired Xbox Series X|S in *Marvel Rivals* on FW 10.60, the stream
opened and delivered about 47 chunks a second, but every chunk was silent. This held:
- with the DualSense disconnected;
- emulating a DualSense and a DualShock 4;
- after binding the virtual pad to the signed-in user (`POST /api/rebind?slot=0`).

A real DualSense rumbled in the same game. The USB rumble path itself works (see Test rumble),
so the console does not route this game's rumble to the capture for these virtual pads.

## v1.1.0 (Build 24) — community pre-release

Built on top of upstream v1.0.4 (Build 19). It combines the pull requests that were open
upstream, plus fixes from their review threads. Every item lists who did it and how far it
has been tested.

"Console-tested" means someone ran it on a PS5. "Host-tested" means only the unit tests in
`tests/` cover it.

### Console testing of this build
This exact payload (`OmniPad-PS5-1.1.0-b24.elf`) was tested on FW 10.60 with a wired Xbox
Series X|S, an Xbox One X, a Switch Pro and a generic USB pad. All worked, with no lost or
stuck inputs. The earlier integration build (b23, PRs #17 to #21) passed the same test.

### USB transport
- **Xbox One / Series GIP protocol and reliable USB_FS IN transfers** — PR #17 (eIBart0 / gabrielskaminski), fixes #12.
  One IN transfer stays armed with no timeout and is re-armed only after it completes. USB
  slots are event-driven, with no 500 ms auto-neutral. *Console-tested* with a wired Xbox
  Series X|S and an Xbox One X. The same approach also fixed dropped inputs on an 8BitDo Ultimate 2C
  dongle (jpemendonca) and on a Turtle Beach Rematch Advanced (BluntEXE); both tested their
  own equivalent implementations.
- **IN reports no longer lost while an OUT packet is sent** — from BluntEXE's review of PR #17.
  An IN completion that arrived while an ACK, LED or rumble packet was going out was taken as
  the OUT completion. The input was dropped and the IN endpoint was never re-armed.
  *Console-tested* with the Xbox pads above, which send ACK packets constantly.
- **Licensed GIP pads detected by interface descriptor** — from BluntEXE's review of PR #17.
  Xbox mode used to be enabled only for Microsoft's VID. Pads such as the Turtle Beach Rematch
  (`0e6f:034b`) went through the XInput parser and produced phantom input from GIP frames (for
  example the Guide release). Microsoft pads behave as before (*console-tested*). Licensed GIP
  pads are *not console-tested*.
- **USB worker tick**: 250 µs for GIP, 4 ms (~250 Hz) for everything else.

### Controllers
- **8BitDo Ultimate 2C (`2dc8:3106`)**: XInput player LED sent on the OUT endpoint — PR #21 (jpemendonca). *Console-tested* on FW 13.60. Generic XInput pads still work (*console-tested*).
- **GameSir G7 Pro / G7 SE**: HID parser, per-slot retention, View+Menu → PS shortcut — PR #19 (SacaSoh). *Author-tested* on FW 10.20. The G7 Pro dongle D-pad, intermittent before, works with this build (SacaSoh, PS5 Pro, FW 10.20, *console-tested*). The View+Menu shortcut is enabled only for the G7 Pro HID identity (`3537:1022`).
- **Nintendo Switch Pro / Switch-mode dongles**: full `0x30` report parser and USB init — PR #20 (SacaSoh). An 8BitDo Ultimate 2 in Switch mode worked on the author's console. An official Switch Pro works on FW 10.60 (*console-tested*).
- **Thrustmaster T598 wheel (`044f:b696`)** as a DualSense — PR #23 (igorgump). *Console-tested*.
- **Mad Catz SF IV FightPad (`0738:8828`)**: parser built from the capture in issue #10. *Host-tested.*
- **Nintendo Wii U GameCube adapter (`057e:0337`)**: PR #22 (Jinx9110). The parser was rewritten to the adapter's real protocol (start command `0x13`, 37-byte `0x21` reports, port 1 only). *Host-tested, experimental.*

### Bluetooth, haptics, dashboard
- **AnyPad Bluetooth stack, DualSense haptics as vibration, 8BitDo Ultimate 2 BT profile, button remapping, per-controller settings, single-page dashboard with "Pair controller"** — PR #18 (BearTS, ported from AnyPad). **Bluetooth is not validated on a console yet: experimental.**

### Build
- GitHub Actions workflow: host tests plus a PS5 payload build with the repo's Dockerfile (PS5 Payload SDK v0.43).

### Not done yet
- Controllers that still need captures or descriptors: #2, #3, #4, #5, #6, #7, #8, #11, #13, #14, #24.
- FW 4.00 (#25) is outside the supported range (7.00 – 13.60).
