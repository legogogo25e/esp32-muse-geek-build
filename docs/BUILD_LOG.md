# ESP32 Muse Geek — Build Log

Firmware for the Waveshare ESP32-S3-GEEK (ESP32-S3R2, 16 MB flash,
2 MB PSRAM, 1.14" ST7789 135x240). Source repo `legogogo25e/esp32-muse-geek`
(private); this repo publishes the firmware + Pages flasher. ESP-IDF
v6.0.3 only. Flash: dio/80m/keep; all 5 files, app at 0x20000.

## Runs

| Run | Upstream | Built (HKT) | App size | SHA (app) | Notes |
|-----|----------|-------------|----------|-----------|-------|
| #2 | `a0769f2` | 2026-11-11 09:57 | 1,708,032 | — | First build; Geek backend + `device.display`; BLE `MuseGadget-Disp` |
| #3 | `6a7cdad` | 2026-11-11 10:17 | 1,445,888 | `f1d321e9…` | BLE `ESP-Muse-Geek`; mascot = 魔鬼仔 standing (8 frames) |
| #4 | `02f87ec` | 2026-11-11 10:31 | 1,445,888 | `0cc36837…` | Y-gap 40 fix (bottom-40-rows garble); mascot white bg |
| #5 | `990acbd9` | 2026-11-11 10:52 | 1,445,888 | `4a97d9a6…` | BLE `MuseGadget-Geek`; typing mascot v1 (keyed bg); full-screen white |
| #6 | `6d93279` | 2026-11-11 11:14 | 1,445,888 | `8fd57049…` | No agent-name title (non-ASCII name rendered as `?`) |
| #7 | `f1a87d5` | 2026-11-11 11:24 | 1,445,888 | `3e3340e9…` | Typing mascot v2 (original black bg, paws press keys) |

## Verification (board homelink-6dec20, BLE `MuseGadget-Geek-6DEC20`)

- Paired 2026-11-11 (App scan only discovers the `MuseGadget` prefix;
  `ESP-Muse-Geek` was invisible to the App although nRF Connect saw it).
- `device.health` ok; `serial.read` shows clean boots, 0 dropped lines.
- `device.display` text (`Hello Geek`, `OTA OK!`) and chunked RGB565
  image both draw. `display.draw_file` draws a VM file (used for the
  Chinese title demo: `魔鬼仔` rendered to RGB565 — the screen font is
  ASCII-only, so Chinese goes via image).
- Colour-bars test (draw_file, 6 bands) renders exactly: pipeline,
  colours and the (52, 40) GRAM offset are correct.
- OTA run #7 (`force`, same version 1.0.0): board rebooted into the new
  image, `PENDING_VERIFY` → `OTA image validated`, no rollback.
- `device.reboot`: board rebooted and reconnected on its own, health ok.

Acceptance (all passed): build, USB flash + boot, pairing, screen
status, Muse Link text + image, OTA, reboot.

## Run #8 — BOOT-button message acknowledgement

- Upstream `eb519fc0`, built 2026-11-11 11:41 HKT, app 1,445,888 B,
  sha256 `4dbec24a…`. Adds `device.message_ack` (showing / seq /
  acknowledged) and a BOOT short-press hook: tapping BOOT while a message
  is on the screen acknowledges it and restores the animation. Pairing
  confirmation and the 5 s hold to reset setup are unchanged. OTA applied
  and validated on homelink-6dec20; end-to-end verified: test message
  "Tap BOOT to confirm" (seq 1, showing, unacknowledged) → BOOT tap →
  acknowledged, animation restored (log: "button short-press
  acknowledged the on-screen message").
