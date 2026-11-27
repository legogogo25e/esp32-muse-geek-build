# ESP32 Muse Geek — Build Log

Firmware for the Waveshare ESP32-S3-GEEK (ESP32-S3R2, 16 MB flash,
2 MB PSRAM, 1.14" ST7789 135x240). Source repo `legogogo25e/esp32-muse-geek`
(private); this repo publishes the firmware + Pages flasher. ESP-IDF
v6.0.3 only. Flash: dio/80m/keep; all 5 files, app at 0x20000.

Current status (2026-11-26): latest run #41, version
`1.0.2-beta.1a0895e`, running on the board in avatar animation mode;
remote updates go through `geek.ota` (upstream removed the standard
`device.ota` Link command). Since run #41, the Geek Config Space
gained GIF upload conversion (`tools/gif_to_ganim.py` on the VM) —
no firmware change, so no new run.
The table below covers the early runs (#2–#23); later runs are the
dated sections further down.

## Runs

| Run | Upstream | Built (HKT) | App size | SHA (app) | Notes |
|-----|----------|-------------|----------|-----------|-------|
| #2 | `a0769f2` | 2026-11-11 09:57 | 1,708,032 | — | First build; Geek backend + `device.display`; BLE `MuseGadget-Disp` |
| #3 | `6a7cdad` | 2026-11-11 10:17 | 1,445,888 | `f1d321e9…` | BLE `ESP-Muse-Geek`; mascot = 魔鬼仔 standing (8 frames) |
| #4 | `02f87ec` | 2026-11-11 10:31 | 1,445,888 | `0cc36837…` | Y-gap 40 fix (bottom-40-rows garble); mascot white bg |
| #5 | `990acbd9` | 2026-11-11 10:52 | 1,445,888 | `4a97d9a6…` | BLE `MuseGadget-Geek`; typing mascot v1 (keyed bg); full-screen white |
| #6 | `6d93279` | 2026-11-11 11:14 | 1,445,888 | `8fd57049…` | No agent-name title (non-ASCII name rendered as `?`) |
| #7 | `f1a87d5` | 2026-11-11 11:24 | 1,445,888 | `3e3340e9…` | Typing mascot v2 (original black bg, paws press keys) |
| #11 | `33f033c1` | 2026-11-11 14:06 | 1,576,960 | `3fb64521…` | Landscape GUI; live on board |
| #12 | `26305b4d` | 2026-11-11 15:00 | 1,576,960 | `03a494bc…` | Timed full run (6m47s end to end); no code change |
| #13 | — | 2026-11-11 16:10 | — | — | **Build FAILED** — ADC line-fitting API not on S3 |
| #14 | `5872e30` | 2026-11-11 16:18 | 1,642,496 | `79b2f167…` | Build ok; **OTA rolled back** — register JSON > 8192 cap |
| #15 | `66f51c7` | 2026-11-11 16:32 | 1,642,496 | `7a98cdf8…` | External UART1/I2C/GPIO6; live on board |
| #16 | `6e2fa950` | 2026-11-11 17:36 | 1,642,496 | `2899abb1…` | External-devices page; live on board |
| #17 | `3ca7b4ff` | 2026-11-11 17:45 | 1,642,496 | `f95df086…` | External page `?` garble fix; live on board |
| #18 | `93052707` | 2026-11-11 18:14 | 1,642,496 | `c7a2c9c1…` | External page v3 (scale-2, user layout); live on board |
| #19 | `382b82fc` | 2026-11-11 19:40 | — | — | **Build FAILED** — `-Werror=format-truncation` in led_status.c |
| #20 | `1c7a325b` | 2026-11-11 19:53 | 1,642,496 | `f6ffded4…` | serial.config + i2c.config; live on board |
| #21 | `0884e618` | 2026-11-12 09:58 | 1,642,496 | `9c0d655d…` | UART rings 100 KB (PSRAM); live on board |
| #22 | `b1c558c2` | 2026-11-12 10:34 | 1,642,496 | `cb5536d7…` | External page section colours; live on board |
| #23 | `ca54ea12` | 2026-11-12 11:12 | 1,642,496 | `93a0705b…` | gpio.pull (GPIO6 internal pull); live on board |

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

## Run #9 — 16x16 CJK bitmap font (Chinese text, no image)

- Upstream `bdfdc2c3`, built 2026-11-11 11:57 HKT, app 1,576,960 B,
  sha256 `c9f68f91…`. device.display text is now UTF-8: ASCII uses the
  5x8 pixel font, the common Simplified Chinese characters (GB2312
  level 1, 3866 glyphs, `main/font_cjk16.h`, generated by
  `tools/gen_cjk16_font.py` from Noto Sans CJK SC Bold) draw directly;
  other characters show as `?`. Lines wrap by display width, never
  mid-character.
- OTA took three attempts: the 1st download stalled (~170 s, no
  progress), the 2nd finished but failed validation
  (`ESP_ERR_OTA_VALIDATE_FAILED`, corrupted download - the image on
  Pages verified good), the 3rd succeeded; the board rebooted,
  "OTA image validated", no rollback. The board stayed on run #8
  through both failures.
- End-to-end verified: text "中文不用图片了" drew (1 line), the user
  saw it on screen and pressed BOOT; ack reads acknowledged, animation
  restored.

## Run #10 — landscape GUI (build FAILED)

- Upstream `5f976c38`, run 42968763326. Build failed in led_status.c:
  `COLOR_GREEN` defined but not used (`-Werror=unused-const-variable`),
  plus unused `to_full_scale`. Cause: the landscape GUI never paints a
  coloured bar or dot, so nothing in the Geek build references them; the
  other colours are still referenced by `blink`/`breathe` helpers. No
  manifest was produced.

## Run #11 — landscape GUI (live on board)

- Upstream `33f033c1`, built 2026-11-11 14:06:33 HKT, app 1,576,960 B,
  sha256 `3fb64521…`. Run 42969715934, success. Manifest verified: same 5
  addresses, downloaded SHA matches, magic 0xE9, strings contain
  "Connected" and "pool.ntp.org".
- Fix: `COLOR_GREEN` and `to_full_scale` are now gated out for the Geek
  (`#if !CONFIG_HOMEHUB_LED_BACKEND_WAVESHARE_S3_GEEK_ST7789`); still
  defined for the other backends. Host test
  `tests/test_geek_landscape_gui.py` asserts the guards so they do not
  regress. `tests/test_link_spi_display_backends.py` updated: Geek
  mirrors for rotation 1, gap (40, 53, 0x00), no title; its C probes
  gained the std headers the pure helpers need. Full host suite: 532
  tests, only the known environmental failures remain (missing IDF
  managed components / FreeInk).
- Screen is now 240x135 landscape (rotation 1, MADCTL 0x60, GRAM gap
  (40, 53)), black background, white text/icons. Top status bar: WiFi
  signal bars, HKT clock via SNTP, green dot + "Connected"/"Offline".
  Mascot animation centred below it. A message covers the animation
  until BOOT acknowledges it; the status bar stays visible. Backlight
  turns off after 30 s idle (never while a message is showing); a BOOT
  press while it is off only wakes it; a new message wakes it.
  Full-width image chunks are at most 7 rows (1792 px / 240).
- OTA applied first try (`{"ok": true, "rebooting": true, "status":
  "applied"}`); board back in ~80 s, health ok, boot log shows
  "LED status ready: Waveshare ESP32-S3-GEEK ST7789 240x135 display
  (BL=7)" and "OTA image validated", no rollback. The boot log also
  caught a real backlight wake in the wild: a BOOT press 36 s after boot
  logged "button short-press woke the backlight; not acknowledging"
  (the backlight had turned off after 30 s idle, the press woke it
  only).
- End-to-end verified on board: Chinese text "横屏 GUI 已上线" drew
  (1 line, not truncated); user confirmed the screen is the right way
  up, the status bar shows WiFi, the clock and Connected, and the
  message is white on black in the main area; BOOT press → ack reads
  acknowledged, showing false, animation restored.

## Run #12 — timed full run (no code change)

- Upstream `26305b4d`, built 2026-11-11 ~15:00 HKT, app 1,576,960 B,
  sha256 `03a494bc…`. Run 42975992392, success. Same size as run #11
  (same code, SHA differs only by build timestamp). Full pipeline
  timed 6 min 47 s: trigger 3 s, build + publish 4 min 01 s, manifest
  verify 31 s, OTA + verify 2 min 15 s. OTA (`force`, same version
  1.0.0) first try, board back, health ok, log shows "OTA image
  validated". `gh workflow run` dispatch verified end to end.

## Run #13 — ADC calibration (build FAILED)

- Run 42984954822. Build failed in `main/geek_external.c`:
  `adc_cali_line_fitting_config_t` unknown type, and
  `adc_cali_create_scheme_line_fitting` implicitly declared. Cause:
  line-fitting calibration is ESP32/S2/C2 only; the ESP32-S3 uses the
  curve-fitting scheme. No manifest was produced.

## Run #14 — external interfaces (OTA rolled back)

- Upstream `5872e30`, built 2026-11-11 16:18:44 HKT, app 1,642,496 B,
  sha256 `79b2f167…`. Run 42985559676, build success, manifest verified.
  Fix for run #13: `adc_cali_curve_fitting_config_t` +
  `adc_cali_create_scheme_curve_fitting` (chan = ADC_CHANNEL_5).
- OTA applied (`{"ok": true, "rebooting": true, "status": "applied"}`),
  but the board never came back online and rolled back to run #12
  after 300 s. Root cause: the 8 new commands grew `link.register` to
  ~8.6 KB, past the 8192-byte preallocated print buffer in
  `build_register_json()` (`main/noise_control.cpp`), so it returned
  NULL, the board reconnected in a loop and never validated the OTA.
  No crash, no useful log line — just "could not build link.register".

## Run #15 — external UART1 / I2C / GPIO6 (live on board)

- Upstream `66f51c7`, built 2026-11-11 16:32:45 HKT, app 1,642,496 B,
  sha256 `7a98cdf8…`. Run 42987442145, success. Manifest verified: same
  5 addresses, downloaded SHA matches.
- Fix for run #14: the register print cap raised 8192 → 16384, with a
  comment explaining why. Regression test
  `tests/test_geek_external.py::test_register_json_fits_its_print_buffer`
  estimates the register size (run #12 measured 5626 bytes + the Geek
  block) and fails if it exceeds the cap. AGENTS.md "Adding a Link
  command" documents the cap as bound #6.
- What it adds (all Geek-gated, `main/geek_external.c`):
  `console.read`/`console.write` (renamed from `serial.read`/`serial.write`;
  UART0 stays the console), `serial.read`/`serial.write` (device UART1,
  TX GPIO13 / RX GPIO14, 8N1, default 115200, 2 KB RX ring),
  `i2c.scan`/`i2c.read`/`i2c.write` (SDA GPIO16 / SCL GPIO17, 100 kHz,
  internal pull-ups, hex data), `gpio.read`/`gpio.write`/`gpio.adc`
  (only GPIO6; ADC1 CH5, 0–3.3 V, curve-fitting calibrated). 19
  commands total (was 11).
- OTA applied first try; board back in ~90 s, health ok. Boot log:
  "sent link.register (8629 bytes)", "link.register acked", "OTA image
  validated (control WS up)", no rollback. 8629 > 8192 confirms the
  run #14 diagnosis.
- End-to-end verified on board: `console.read` returns the log (rename
  works); `i2c.scan` returns `[]` (nothing connected); `gpio.adc`
  GPIO6 reads 386 mV (raw 436, calibrated); `gpio.write` 1 then
  `gpio.read` reads back 1, then write 0; `gpio.read` pin 13 rejected
  ("only GPIO6 is available"); `serial.write` "hello uart1" wrote 11
  bytes; `serial.read` returns empty (nothing connected); `i2c.read`
  on address 0x50 fails cleanly (`ESP_ERR_INVALID_RESPONSE`, no crash);
  `device.display` Chinese text "外接接口就绪" drew.
- Not yet verified: UART1 loopback (`serial.write` → `serial.read`)
  needs a jumper shorting GPIO13↔GPIO14 on the H2 header (user's hands).
  **Verified 2026-11-11 ~16:40 after the user shorted them:** the RX ring
  first returned ~2.7 KB of garbage that had accumulated while RX floated
  before the short (a floating UART RX receives noise); after draining it,
  a clean write of "LOOPBACK-OK-456" read back byte-for-byte as UTF-8, and
  a hex write of "deadbeef" read back as 4 bytes hex. Both encodings loop
  back correctly.

## Run #16 — external-devices page (live on board)

- Upstream `6e2fa950`, built 2026-11-11 17:36:00 HKT, app 1,642,496 B,
  sha256 `2899abb1…`. Run 42996114634, success (3m20s). Manifest
  verified: same 5 addresses, downloaded SHA matches, strings contain
  "display.show_external" and "external-devices page".
- What it adds (Geek only, `main/led_status.c` +
  `main/geek_external.c`, `tests/test_geek_external_page.py`, 16 tests):
  a status page for the external interfaces, shown by a double BOOT
  press (the double-press was ignored before) or `display.show_external`;
  a short BOOT press or `display.show_animation` goes back to the
  animation. The status bar shows the firmware version (`v1.0.0`,
  from `esp_app_get_description()`) instead of the clock while the page
  is on; the main page keeps the clock. Contents, scale-1 pixel font
  (6 px/char, 8 px tall): UART1 baud / TX-RX pins / TX-RX byte counts,
  ADC GPIO6 + live voltage, I2C frequency / SDA-SCL pins / found
  addresses / TX-RX byte counts. Byte counts are cumulative since boot
  (`geek_external_stats()`; a register byte before an I2C read counts
  as 1 TX byte); I2C addresses are the last scan's result, cached
  (`geek_external_i2c_cached_addrs()`), with a scan on page entry.
  The ADC and the counts refresh ~2 Hz from the animation task; the
  backlight stays on while the page is showing. `s_status_clock` is
  `char[40]` (was `[6]`) so the version fits.
- OTA applied first try; board back in ~90 s, health ok. Boot log:
  "sent link.register (9070 bytes)" (was 8629; the new command added
  ~440 bytes, still well under the 16384 cap), "link.register acked",
  "OTA image validated (control WS up)", no rollback.
- End-to-end verified on board: the user double-pressed BOOT and the
  page showed (log: "button double-press toggled external page
  (showing=1)"), short-pressed to hide it, double-pressed again;
  `display.show_external` works from Link; `gpio.adc` GPIO6 reads
  31 mV (raw 35, floating pin).
- **Bug found by the user on first sight:** the page showed a bunch of
  `?` garble. Cause: the draw loop read past the end of short lines
  ("ADC" is 3 chars) into uninitialised stack in the 40-byte text
  buffer, and the garbage bytes rendered as `?`. Fixed in run #17.

## Run #17 — external page `?` garble fix (live on board)

- Upstream `3ca7b4ff`, built 2026-11-11 17:45:25 HKT, app 1,642,496 B,
  sha256 `f95df086…`. Run 42997321192, success. Manifest verified:
  same 5 addresses, downloaded SHA matches. Only `main/led_status.c`
  and the test changed.
- Fix: each page line now records its length (`strlen` after
  `snprintf`), and the draw loop stops at it (`if (ci >= len) break;`)
  instead of reading past the string into uninitialised stack.
  Regression test
  `tests/test_geek_external_page.py::test_draw_loop_never_reads_past_a_line`.
- OTA applied first try; board back in ~90 s, health ok, "OTA image
  validated", no rollback. Register still 9070 bytes.
- End-to-end verified on board: the user double-pressed to the page;
  `serial.write` "STATS-TEST-001" (14 B) + "STATS-TEST-002" (14 B) +
  "HELLO-UART1" (11 B) = 39 B TX, and the loopback (GPIO13↔GPIO14
  still shorted) read back all 39 B as UTF-8, so the page shows
  TX 39 B / RX 39 B. The `?` garble is gone (user to confirm visually).

## Run #18 — external page v3, scale-2 font (build done, OTA pending)

- Upstream `93052707`, built 2026-11-11 18:14:41 HKT, app 1,642,496 B,
  sha256 `c7a2c9c1…`. Run 43001478067, success. Manifest verified:
  same 5 addresses, downloaded SHA matches. Firmware strings contain
  the new v3 formats ("UART1 Tx(%d) Rx(%d)", "Baud %d 8N1",
  "ADC (IO%d) %d.%03dV", "I2C SDA(%d) SCL(%d)", "Addr %s %dkHz").
- What it changes (user-confirmed layout, `main/led_status.c`,
  `tests/test_geek_external_page.py`, mockup
  `prep/mockup_external_v3.py`): the page now uses the scale-2 pixel
  font (12 px/char, 16 px tall) throughout — the scale-1 text was too
  small. The GPIO pins sit on the section's own line in brackets
  (`Tx(13)`, `Rx(14)`, `SDA(16)`, `SCL(17)`, `(IO6)`), and the UART
  baud + 8N1 and the I2C address + frequency share a line with their
  section, so the page is 7 rows: `UART1 Tx(13) Rx(14)` /
  `Baud 115200 8N1` / `TX 39B RX 39B` / `ADC (IO6) 0.386V` /
  `I2C SDA(16) SCL(17)` / `Addr 0x48 100kHz` / `TX 123B RX 456B`.
  The user's two long lines (37 and 35 chars) were split in two because
  a 240 px line at scale 2 only fits 20 chars. The I2C frequency is
  100 kHz (shown as `100kHz`), not 100 Hz. The draw loop does the
  scale-2 glyph lookup (`gx / scale`, `ly / scale`) and still stops at
  each line's length (the run #17 `?` fix).
- Host tests: affected modules 39/39 pass; full suite excl. avatar
  561 tests, 3 failures + 11 errors = the known environmental
  baseline (no regressions).
- OTA applied first try (`{"ok": true, "rebooting": true, "status":
  "applied"}`); board back in ~90 s, health ok (uptime 91 s). Boot
  log: "sent link.register (9070 bytes)", "link.register acked",
  "OTA image validated (control WS up)", no rollback.
- End-to-end verified on board: `display.show_external` opens the v3
  page; `serial.write` "V3-BIG-FONT-OK" (14 B) looped back and read
  as 14 B UTF-8, so the page shows TX 14B / RX 14B; `gpio.adc` GPIO6
  reads 0 mV (raw 0, floating pin). (README + AGENTS.md already
  updated to the v3 layout and pushed.)

## Run #19 — serial.config + i2c.config (build FAILED)

- Upstream `382b82fc`, run 43013444372. Build failed in
  `main/led_status.c`: `'snprintf' output may be truncated before the
  last format character [-Werror=format-truncation=]` at the Baud line
  (`"Baud %d %d%c%s"`). Cause: the stop-bits string is chosen at
  runtime, so the compiler could not bound the `%s` and assumed the
  output might exceed the 48-byte line buffer. No manifest was
  produced. (Also a lesson: `gh run watch ... | tail` hides the watch's
  exit code — the pipeline exits with tail's status, so a failed build
  looked successful. Check the run conclusion directly.)

## Run #20 — serial.config + i2c.config (build done, OTA pending)

- Upstream `1c7a325b`, built 2026-11-11 19:53:59 HKT, app 1,642,496 B,
  sha256 `f6ffded4…`. Run 43015314518, success. Manifest verified:
  same 5 addresses, downloaded SHA matches. Firmware strings contain
  "serial.config", "i2c.config" and the new error messages.
- Fix for run #19: the stop-bits format is now `"%.3s"` ("1.5" is the
  longest), so the compiler can prove the Baud line fits; and the
  external-page line buffer grew 48 → 64 bytes because the Addr line
  (`"Addr %s %dkHz"`) can reach ~52 bytes once the frequency comes
  from `i2c.config` at runtime instead of the compile-time constant
  (100000/1000 = 100, 3 chars). Regression test
  `tests/test_geek_external_page.py` asserts the bounded format and
  the 64-byte buffer.
- What it adds (`main/geek_external.c`, `main/app.c`,
  `main/noise_control.cpp`, `tests/test_geek_external.py`, 5 new
  tests): `serial.config` (baud 300-3000000, data_bits 5-8, parity
  none/even/odd, stop_bits 1/1.5/2; all optional, no params returns the
  current config; changing it flushes the RX ring) and `i2c.config`
  (frequency 10000-1000000 Hz, default device address 0-127 or -1 to
  clear; all optional). `i2c.read` / `i2c.write` now take an optional
  address and fall back to the default; with neither set they error
  and point at `i2c.config`. The I2C clock is per device
  (`scl_speed_hz`), so changing the frequency takes effect on the next
  transaction without rebuilding the bus. The external page shows the
  live serial format and the live I2C frequency. 22 commands total
  (was 20).
- Host tests: affected modules 44/44 pass; full suite excl. avatar
  566 tests, 3 failures + 11 errors = the known environmental
  baseline (no regressions).
- **OTA verified 2026-11-11 ~21:43 HKT** (run #20, force: true): applied
  first try, board back in ~100 s, health ok (uptime 104 s). Boot log:
  "sent link.register (10350 bytes)", "link.register acked",
  "OTA image validated (control WS up)", no rollback. Register grew
  9070 → 10350, well under the 16384 cap. 22 commands live.
- Board checks: `serial.config` query returns 115200/8/none/1; set to
  9600/7/even/2, read back the same, restored to 115200/8/none/1.
  `i2c.config` query returns frequency 100000, address null; set to
  400000 + default address 72, read back the same; `i2c.read` without
  an address then used the default and failed cleanly
  (`ESP_ERR_INVALID_RESPONSE`, nothing at 0x48); cleared the default
  with -1 and restored 100000 Hz; `i2c.read` without an address then
  errors cleanly ("no address given and no default address configured
  (use i2c.config to set one)"). `display.show_external` opens the
  page (baud line and frequency now live from the config).
- (README + AGENTS.md already updated to the config commands and
  pushed.)

## Run #21 — serial RX/TX rings 100 KB (build done, OTA pending)

- Upstream `0884e618`, built 2026-11-12 09:58:53 HKT, app 1,642,496 B,
  sha256 `9c0d655d…`. Run 43137768923, success. Manifest verified:
  same 5 addresses, downloaded SHA matches.
- What it changes (`main/geek_external.h`, `main/geek_external.c`,
  `tests/test_geek_external.py`, 3 new tests): the UART1 RX ring grows
  2048 → 100 KB and the TX ring 512 → 100 KB. Internal SRAM cannot
  hold them (~120 KB free at runtime), so they rely on
  `CONFIG_SPIRAM_USE_MALLOC=y` + `ALWAYSINTERNAL=16384` in
  `sdkconfig.defaults`: the driver allocates the rings through the
  generic heap (`xRingbufferCreate`), so allocations this large land
  in PSRAM. If the install fails, init retries with 2048/512 so the
  port comes up degraded instead of dead, and the boot log line
  "device serial ready … RX buf %u B, TX buf %u B" reports the sizes
  that actually installed.
- Host tests: affected modules pass; full suite excl. avatar 569
  tests, 3 failures + 11 errors = the known environmental baseline.
- **OTA verified 2026-11-12 ~10:15 HKT** — but the first attempt
  (10:01) stalled: the `device.ota` invoke timed out at 30 s and every
  later invoke timed out for ~11 min while the board still showed
  online (heartbeats are a separate task). The board's WAN was having
  a bad patch — the recovery boot's log shows its VM-lease fetch
  needing 3 attempts over ~47 s (`ESP_ERR_HTTP_EAGAIN`,
  `ESP_ERR_HTTP_CONNECT`) — so the 1.6 MB download apparently crawled
  with the command handler occupied. A `device.reboot` invoke did get
  through and rebooted the board back into run #20 (its boot log had
  the old serial line and the old heap numbers), and the second OTA
  attempt went through in ~2.5 min on the recovered network. Lesson:
  an OTA invoke timeout plus unresponsive commands does not mean the
  board is dead — check `device.list` (still online) and be patient;
  reboot unwedges it.
- Proof on board: boot log "device serial ready: UART1 TX GPIO13 RX
  GPIO14 @ 115200 baud, **RX buf 102400 B, TX buf 102400 B**" (no
  fallback warning), "OTA image validated (control WS up)", no
  rollback. `device.health`: PSRAM free 1,487,324 → **1,285,092 B**
  (−202,232 ≈ the two 100 KB rings plus overhead), internal free
  unchanged (~122 KB). Round-trip at 9600 8N1 (the peer setup):
  `serial.write` "hello agent" (11 B) read back byte-for-byte.
- (README + AGENTS.md already updated to the 100 KB rings and pushed.)

## Run #22 — external page section colours (build done, OTA pending)

- Upstream `b1c558c2`, built 2026-11-12 10:34:34 HKT, app 1,642,496 B,
  sha256 `cb5536d7…`. Run 43141956081, success. Manifest verified:
  same 5 addresses, downloaded SHA matches.
- What it changes (`main/led_status.c`,
  `tests/test_geek_external_page.py`, 1 new test): the external page's
  sections get distinct font colours — UART cyan, ADC yellow, I2C
  magenta (user-approved from mockup `prep/mockup_external_v4.py`);
  the status bar stays white. `lcd_ext_line_t` gains a `color` field
  set per line in the builder, and the draw loop paints each glyph
  pixel with its line's colour instead of one shared white.
- Host tests: affected modules pass (incl. the unused-symbol check);
  full suite excl. avatar 570 tests, 3 failures + 11 errors = the
  known environmental baseline.
- **OTA verified 2026-11-12 ~10:56 HKT**: applied first try, board
  back in ~90 s, health ok (uptime 106 s, PSRAM free unchanged at
  1,285,092 B — the 100 KB rings still in PSRAM). Boot log: "OTA image
  validated (control WS up)", register 10350 bytes, no rollback. The
  serial config was re-applied after the reboot (9600 8N1, the peer
  setup; config does not survive a reboot) and the external page was
  opened remotely for the user's visual check of the colours. Note:
  from this run on, the user says OTA needs no per-run go-ahead —
  build, OTA and verify directly.
- (README + AGENTS.md already updated to the colours and pushed.)

## Run #23 — gpio.pull (build + OTA verified)

- Upstream `ca54ea12`, built 2026-11-12 11:12:39 HKT, app 1,642,496 B,
  sha256 `93a0705b…`. Run 43146282218, success. Manifest verified:
  same 5 addresses, downloaded SHA matches; firmware strings contain
  "gpio.pull". 23 commands total (was 22).
- What it adds (`main/geek_external.h/.c`, `main/app.c`,
  `main/noise_control.cpp`, `tests/test_geek_external.py`, 3 new
  tests): `gpio.pull` gets/sets the internal pull of GPIO6
  (`"up"` / `"down"` / `"none"`, `pin` optional defaulting to 6; omit
  `pull` to query). The setting persists in `s_gpio_pull` and is
  applied by a shared `gpio6_config_input()` whenever the pin is
  configured as an input — `gpio.read` included; setting it also
  switches the pin to input mode. Default `"none"` (floating), as
  before. Naming: the user suggested `gpio.setting`, then
  `gpio.pull`, and took `gpio.pull` on the assistant's recommendation
  (name says exactly what it controls; `.config` stays reserved for
  multi-field configuration).
- OTA applied first try (the user has lifted the per-run OTA
  go-ahead: build → OTA → verify directly); board back in ~95 s,
  health ok, no rollback. Board checks on the floating pin:
  query → `"none"`; set `"down"` → `gpio.read` level 0, `gpio.adc`
  0 mV / raw 0 (held at ground, no more floating wander); set `"up"`
  → level 1, ADC raw 4095 (3159 mV calibrated at the rail); restored
  to `"none"` afterwards. The serial config was re-applied after the
  reboot (9600 8N1, the peer setup).
- (README + AGENTS.md already updated to gpio.pull and pushed.)

## Run #29 — SWD step 1: swd.connect / swd.disconnect (USB-flashed, real-target verified)

- Upstream `esp32-muse-geek` @ `1f91fbf0`: new `main/geek_swd.c/.h`
  — a bit-banged SWD master (SWCLK GPIO16 / SWDIO GPIO17 / nRESET
  GPIO6), CMSIS-DAP-shaped transfers with the WAIT/FAULT discipline,
  an AHB-AP memory layer, and pin arbitration with I2C/GPIO through
  the geek_external claim/release. `swd.connect` (fingerprint +
  halt, STM32F103 profile gate) and `swd.disconnect` are the only
  commands exposed in this step. The register print cap rose
  16384 → 24576 (register now 12,711 B). Two builds: GitHub run
  45354911657 (v1) and run 45360634014 (the fix below); final app
  1,642,496 B, sha256 `a83bcc01…`; both manifests verified.
- **OTA was unavailable for this run**: `device.ota` has
  disappeared from the platform's invokable command list (invokes
  are rejected before reaching the board; the firmware still
  registers it, and fresh registrations — `swd.*` included — do
  arrive in the list). Both builds were flashed over USB with the
  flasher page instead; NVS (pairing, Wi-Fi) untouched.
- The first on-board connect failed with a protocol error at every
  clock and reset combination. Root cause was in the firmware, not
  the wiring: the SWD pins were configured `GPIO_MODE_OUTPUT`,
  which disables the pad input path, so SWDIO read back as a
  constant 0. Fixed to `GPIO_MODE_INPUT_OUTPUT` and added a raw-ACK
  log line — "bad ack 7" then proved the input path alive and the
  line silent until the target's SWCLK/SWDIO pair was oriented
  correctly (the original orientation was right).
- Verified against a real Blue Pill (STM32F103C8T6): DP
  0x1BA01477, AP IDR 0x14770011, DBGMCU 0x20036410 (DEV_ID 0x410),
  flash 64 KB, UID read, RDP off, core halted, `profile_ok` true.
  `i2c.*` / `gpio.*` refused during the session and worked again
  after `swd.disconnect`; the target was left running.

## Run #30 — SWD steps 2+3: session commands, stm32.* flash chain, flash_file (USB-flashed, real-target verified)

- Upstream `esp32-muse-geek` @ `dc806468`: session commands
  (`swd.read_mem` / `swd.write_mem` / `swd.halt` / `swd.run` /
  `swd.regs` / `swd.reset`), the STM32F1 register-method flash
  chain (`main/geek_stm32.c`: `stm32.flash_begin` / `flash_chunk` /
  `stm32.flash_program` / `stm32.flash_status` / `stm32.erase` /
  `stm32.read_flash`), and `stm32.flash_file`. Two builds: GitHub
  run 45363313902 (steps 2+3) and run 45366390532 (adds
  flash_file); final app 1,708,032 B, sha256 `cfdf28b8…`; both
  manifests verified (5 files, addresses, app SHA).
- **OTA still unavailable** (platform-side, see run #29); both
  builds were flashed over USB with the flasher page.
- Step 2 verified on the real Blue Pill (STM32F103C8T6): vector
  read of the old firmware (SP 0x20005000 / Reset 0x080039E4),
  regs (PC 0x08001cd8), a RAM write/read round-trip byte-exact,
  run, and a sysreset the session survived.
- Step 3's first attempt never touched the flash: the 18,716 B
  template image was staged through agent-relayed base64 chunks;
  every chunk's cumulative length checked out, but
  `flash_program`'s SHA-256 gate rejected the staged image — a
  same-length transcription corruption somewhere in the relay.
  The gate did its job. Conclusion: agent-relayed base64 is not a
  firmware transport at KB scale.
- Fix (this run): `stm32.flash_file {path, sha256, erase?,
  verify?}` — the board fetches the file itself with ranged
  `GET /fs/raw/<path>` over the Link session (the
  `display.draw_file` machinery; geek_stm32.c keeps its own
  generation-token fetch state), stages it, and runs the
  identical program flow behind the same SHA gate.
- Verified end to end: `stm32.flash_file` on the VM-built
  template (`Bluepill_Template_C8.bin`, 18,716 B, sha256
  `b61f3376…`) returned programmed 18716 / verified true /
  19 pages / reset in 8,396 ms. The vector table afterwards read
  SP 0x20005000 / Reset 0x08003569 (the template; was
  0x080039E4), the core halted at PC 0x080009f6 inside the new
  image, then ran; disconnect left I2C and GPIO6 working. One
  artifact: a read issued in the instant after the job's target
  reset returned all zeros once; an immediate re-read was
  correct.
- Wiring correction (later the same day): the earlier note that
  the target UART sat on PB10/PB11 with NRST unconnected was a
  misreading of the user's photo — the user pointed this out and
  both were then proven empirically. Serial is bidirectional on
  PA9/PA10: the template's 1 Hz `heartbeat seq=N` lines stream
  into the chat-post side chat, and a `serial.write` of
  "hello pill" came back as the template's echo
  `RX[11]: hello pill`. NRST is wired: with RCC_CSR (0x40021024)
  flags cleared and read back as 0, a `swd.reset` hardware pulse
  (GPIO6) set exactly PINRSTF (0x04000000); a sysreset set
  SFTRSTF. The edit → rebuild → reflash → serial-proof loop is
  fully closed.

## Run #31 — feature configuration: `device.features` pin-function assignment (USB-flashed, acceptance matrix passed)

- Upstream `esp32-muse-geek` @ `9f4463dd`: `main/geek_features.c`
  (NVS keys `feat_serial` / `feat_h1` / `feat_pin6`, defaults
  on / i2c / adc; shared coerce: `pin6: rst` falls back to adc
  unless `h1: swd`), registration gating in
  `build_register_json()` + `device.features` command, reboot
  semantics (reply first, then `restart_task`), hardware-init
  gating in `geek_external_init()` (UART1 / I2C bus / ADC only
  when configured), retired SWD/I2C bus arbitration (claim/release
  only flip the ownership flag), config-driven external page
  (sections collapse upward; SWD section 3 green rows incl.
  `RST (IO6)`), and the Geek Config Space 「外接功能」 section
  (slug `geek-pei-zhi` at the time). GitHub run 45543019501;
  app 1,708,032 B, sha256 `d6152160…`; manifest verified.
- Two build failures on the way: run 45541394696 failed at link
  (`geek_features.h` lacked `extern "C"` guards for the C++
  register builder), and the first 45541939587 publish failed
  transiently at deploy-pages (rerun of the failed job fixed it).
  A board-side bug was found after the first flash:
  `device.features` returned a bare payload without the
  `{"ok": true, "payload": {...}}` envelope and the invoke failed
  as `device_error: unknown` (fixed + host test, run
  45543019501).
- Acceptance matrix, all passed on the real board (2026-11-25):
  defaults `{serial: true, h1: "i2c", pin6: "adc"}`;
  switch to `{h1: "swd", pin6: "rst"}` (reboot, fingerprint
  exact, `i2c.scan` unregistered in SWD mode); `h1` back to i2c
  while `pin6: rst` coerced to adc (`pin6_coerced: true`);
  illegal `{pin6: "rst"}` with `h1: i2c` refused; serial off/on
  (heartbeat posts stop / resume); external page in the SWD combo
  confirmed by the user ("一致"). Full suite 646 tests: 3
  failures + 11 errors, the known environmental baseline.
- `device.ota` still platform-blocked; the user USB-flashed
  twice. Board left in `{serial: true, h1: "swd", pin6: "rst"}`
  for the follow-on work.

## Run #32 — follow-ons: SWD target-model row, beta versioning, chain link icon (USB-flashed, 1.0.0-beta.ea72c80)

- Upstream `esp32-muse-geek` @ `ea72c808`: SWD model row on the
  external page (one-shot probe on page open: connect, read the
  fingerprint, disconnect; `geek_swd_model_str()` maps DEV_ID +
  flash size, e.g. `STM32F103 64KB`, `SWD no target` on failure;
  live fingerprint while a session is open), version display
  (`lcd_version_short()`: dev builds `beta.<short SHA>`, staged
  releases `v<version>`), and the status-bar chain icon (green
  interlocked links when connected, red broken links when
  offline; replaces the dot + "Connected" word on both pages).
  Full suite 655 tests: 3 failures + 11 errors, the known
  environmental baseline, no regressions.
- Workflow change (`.github/workflows/build-esp32.yml`, commit
  `e33d04e`, made through the browser — the `gh` token has no
  `workflow` scope, API pushes 403): new "Resolve firmware
  version" step defaults `PROJECT_VER` to
  `1.0.0-beta.<upstream short SHA>` when no version is passed;
  staged releases pass an explicit version (next staged: 1.0.1).
  GitHub run 45554851669; manifest verified: `project_version`
  `1.0.0-beta.ea72c80`, upstream `ea72c808…`, app 1,708,032 B,
  sha256 `9d467ac1…`, binary contains `SWD no target` /
  `STM32F103` / `device.features`. Build job passed first try;
  the Publish job failed transiently at Deploy to GitHub Pages
  (same hiccup as run 45541939587) and succeeded on
  `gh run rerun --failed`. Lesson: `gh run watch | tail` exits 0
  through `tail` — trust only `gh run view --json
  status,conclusion`.
- Board acceptance (2026-11-25, user USB-flashed): `device.health`
  reports `1.0.0-beta.ea72c80`; config survived the flash;
  probe log `probe: target STM32F103 64KB`; `swd.connect` /
  `swd.disconnect` clean. Board restored to defaults
  `{serial: true, h1: "i2c", pin6: "adc"}` through the rebuilt
  Geek Config Space (slug `geek-config`, English; old
  `geek-pei-zhi` deleted) — request #1 applied end to end, though
  the apply task did not write the snapshot back and it was
  filled in manually (`features` + firmware fields verified on
  read-back). Screen eyeball check (model row, beta top bar,
  chain icon) still pending with the user.

## Run #33 — probe no-halt fix + GPIO6 pin ownership (2026-11-25)

- Bug (user report 14:13): opening the SWD external page stopped
  the STM32 — heartbeats froze at seq 2741 while SWD showed idle.
  Reproduced on board: the page-entry probe ran a full
  `geek_swd_connect`, which writes DHCSR_HALT; the target ended up
  stuck in HAL_UART_Transmit's TC wait (PC 0x0800162c, g_seq 2742)
  and did not resume after the probe's disconnect. Revived with
  `swd.reset` sysreset (heartbeats restarted from seq 0; the
  target had also recovered on its own to seq 2840+ before the
  reset). The exact silicon-level chain (why TC never set / the
  timeout never fired) is still open; the fix removes the trigger.
- Fix (TDD, 2 new host tests): new `geek_swd_fingerprint()` — the
  same switch/power-up/fingerprint sequence as connect, but it
  never writes DHCSR and never freezes the watchdogs, so the core
  keeps running; the probe task uses it instead of connect.
  `geek_swd_connect` is now a thin wrapper over
  `swd_connect_inner(..., halt=true)`; sessions still halt. Also
  fixes the whole-branch review's Important finding: `pins_setup()`
  / `pins_release()` only configure GPIO6 when `pin6 == rst`
  (before, a session silently dropped a user-configured GPIO6
  output and biased the ADC row with a pull-up). Docs updated
  (probe wording in README/AGENTS.md, plus the stale Iron-rules
  Connected/Offline line and the app size 1,576,960 → 1,708,032 B).
  Full suite 657 tests: 3 failures + 11 errors, the known
  environmental baseline, no regressions.
- GitHub run 45562038130; manifest verified: `project_version`
  `1.0.0-beta.20cccc7`, upstream `20cccc76…`, app 1,708,032 B,
  sha256 `0c04c53a…` (matches the downloaded binary), binary
  contains `fingerprint` (4) / `SWD no target` / `device.features`.
  Build job passed first try; the Publish job failed transiently
  at Deploy to GitHub Pages (third time) and succeeded on
  `gh run rerun --failed`.
- Board acceptance: PASSED 2026-11-25 ~14:47 (user USB-flashed
  1.0.0-beta.20cccc7): probe log `fingerprint: … halted 0`,
  model read `STM32F103 64KB`, heartbeats 639→670 continuous
  across the page open. Regression check found two separate
  issues (see run #34): an intermittent session-resume stall
  (~61 s of silence after one connect/disconnect; self-recovered)
  and a ~4 min chat_post delivery stall (beats accumulated in
  the ring, then resumed; one serial.read blocked the Link worker
  ~106 s and the tunnel re-established). Both are pre-existing,
  not regressions of this fix.

## Run #34 — verified SWD resume on disconnect (2026-11-25)

- Follow-up to run #33's regression finding: a plain
  `swd.connect` + `swd.disconnect` once left the target not
  printing for ~61 s, and the disconnect still reported success.
  Root cause in the firmware: `geek_swd_run()` was a
  fire-and-forget DHCSR_RUN write (never checked S_HALT cleared)
  and `geek_swd_disconnect()` ignored its return value.
- Fix (TDD, 6 new host tests in GeekSwdResumeTest): new
  `poll_dhcsr_bit_clear()`; `geek_swd_run()` writes DHCSR_RUN
  then polls until S_HALT clears (100 ms), retries the write +
  poll once, and translates a poll timeout into the new
  `GEEK_SWD_ERR_RESUME` code; `geek_swd_disconnect()` releases
  the session either way but returns the run error, so a stuck
  resume is reported instead of a silent success; the
  `swd.disconnect` handler tells a resume failure apart from
  "no session" and `swd_err_message` names it ("the target core
  did not resume (S_HALT stayed set); retry swd.connect, or
  reset the target"). Docs: README disconnect bullet + header
  comment. Full suite 662 tests: 3 failures + 11 errors, the
  known environmental baseline, no regressions (662 = 657 + 5
  new; a 6th ordering test was added after the first build
  attempt, see below).
- First build attempt (run 45568091707) FAILED at compile:
  `swd_op_error` was defined after `swd_disconnect_command`,
  so the new call in the disconnect handler hit
  `-Werror=implicit-function-declaration`. Fixed by moving the
  helper above the disconnect handler (with a comment) and
  adding a host test pinning the definition order. Affected
  modules 132/132 pass.
- GitHub run 45569034133; manifest verified: `project_version`
  `1.0.0-beta.ec8a658`, upstream `ec8a6582…`, app 1,708,032 B,
  sha256 `8a6ed9c1…` (matches the downloaded binary), binary
  contains `did not resume` (2) / `device.features` (4). Build
  and Publish jobs both passed first try.
- Board acceptance: PASSED 2026-11-25 ~15:51 (user USB-flashed
  1.0.0-beta.ec8a658): version + config {serial:true, h1:"swd",
  pin6:"adc"} confirmed; three connect/disconnect cycles, each
  resumed promptly (beats consecutive, ~2 s spacing, no gap —
  cycle 1: 2454–2460, cycle 2: 2477–2484, cycle 3: 2499–2506);
  probe still works on the new firmware (page open:
  `fingerprint: … halted 0`, `probe: target STM32F103 64KB`,
  beats 2512→2513 exactly on schedule, continuous through
  2531+). No session left open; board clean. The chat_post
  delivery stall (~4 min, seen during run #33 verification) did
  not recur in this session and remains a separate open item.

## Run #35 — chat_post posting switch, default OFF (2026-11-25)

- Trigger: user asked whether the automatic posting of UART1
  lines to the side chat can be controlled by the agent, and
  approved a dedicated switch; then set the factory default to
  OFF ("默认关闭吧").
- Change: `chat_post` joins the feature configuration
  (geek_features): persisted in NVS (`feat_chat_post`), factory
  default **off**; `device.features` accepts/reports it like the
  other three fields (change takes effect by reboot). The
  chat_post task now starts only when `serial && chat_post`, so
  with posting off nothing drains the UART1 RX ring and the
  agent's `serial.read` sees the lines instead (the task and
  serial.read consume the same ring — mutually exclusive per
  byte). Single start site pinned by test. TDD: 4 new/updated
  host tests (defaults+persistence, registration param, handler
  report/accept, gate + single start site). Affected modules
  136/136 pass; full suite 667 tests: 3 failures + 11 errors,
  the known environmental baseline, no regressions (667 = 662 +
  1 ordering test from run #34 + 4 new).
- A default-ON build was produced first (run 45576461998, Build
  ok / Publish transient failure) and superseded by the user's
  default-OFF decision before any flash; not published further.
- GitHub run 45577139204; manifest verified: `project_version`
  `1.0.0-beta.b797809`, upstream `b797809c…`, app 1,708,032 B,
  sha256 `12b72adf…` (matches the downloaded binary), binary
  contains `feat_chat_post` / `chat_post must be a boolean`.
  Publish job failed transiently once, rerun succeeded.
- Board acceptance: PASSED 2026-11-25 ~16:55 (user USB-flashed
  1.0.0-beta.b797809): `device.features` reported
  {serial:true, h1:"swd", pin6:"adc", chat_post:false} — the
  factory default, no stored key; boot log
  `features: serial=on h1=swd pin6=adc chat_post=off` with no
  chat_post task and no queued lines; `serial.read` returned
  heartbeat lines (seq 4255–4305, 1024 B, 176 B left). Turned
  on (`chat_post:true`, reboot): `chat_post=on`, session from
  NVS, posted HTTP 200 (seq 4320–4331). Turned back off
  (reboot): persisted `chat_post:false`, `serial.read` works
  again (seq 4335+). Board left with posting off, per the
  user's choice.

## Run #36 — geek.ota remote update command (2026-11-25)

- Upstream muse-gadget-sdk removed the standard `device.ota` Link command
  (PR #402, merged 2026-11-14); the platform no longer exposes it for this
  board, so agent OTA invokes were rejected before reaching the firmware.
- This build exposes the same OTA engine as `geek.ota` (registration +
  link.invoke handler in `noise_control.cpp`; BLE setup-time `device.ota`
  unchanged). Tests: affected modules 49/49; full suite 668 tests with only
  the known baseline (3 failures + 11 errors).
- Build run 45587671464: Build + Publish both SUCCESS. Version
  `1.0.0-beta.127fc6f`, upstream `127fc6fcd259eac19aadedc1c250120bcc5e17f9`,
  built 2026-11-25T17:44:14+08:00. Manifest verified: 5 files at the
  standard addresses; app 1,708,032 B, SHA-256
  f3b65263eb2a (prefix; full hash in manifest), binary contains `geek.ota`.
- Board acceptance: PASSED (2026-11-25 ~17:46). User USB-flashed the
  bootstrap build; `device.describe` lists `geek.ota` (url + force);
  `geek.ota` self-test with the Pages URL and `force: true` returned
  `{status: applied, rebooting: true}`; after reboot the log shows
  PENDING_VERIFY then `OTA image validated (control WS up)`, config
  persisted (serial on, h1=swd, pin6=adc, chat_post off). Remote updates
  are restored via `geek.ota`.

## Run #37 — SDK upgrade: configurable animation + Link ports (2026-11-26)

- Selective ports from muse-gadget-sdk (e8d2c98) on branch
  `upgrade/sdk-animation-config`, merged to main (ade7be4):
  - Configurable status animation (`display.animation`): neutral
    built-in reactor core (24 frames), off, or a custom `.ganim`
    pack fetched from the VM, CRC-checked, played from PSRAM with
    fallback to the built-in. The personal mascot assets left the
    repo; the mascot pack lives on the VM only.
  - `token_fp` (no more token-prefix logging), `link_events` /
    `link_probe` (connectivity history in `device.health`), and
    `tunnel_relay` (`net.forward` LAN port forwards + NAPT input
    guard). `error_report` deliberately not ported (panic path).
  - Apache-2.0 `LICENSE` added for the planned open-source release.
- Tests: full suite 680 tests, failures=3 / errors=11 = the known
  environmental baseline. Two build fixes along the way
  (`-Werror=stringop-truncation`, tunnel harness stub).
- Builds: beta run 45752057044 (`1.0.0-beta.ade7be4`, app
  1,773,568 B, SHA-256 e42675a1…) then staged run 45752734814
  (`1.0.1`, app SHA-256 b5b2b6cd…). Manifests verified (5 files,
  standard addresses, size + SHA match).
- Board acceptance: PASSED (2026-11-26 ~12:05). Beta OTA applied,
  `OTA image validated`; `device.health` shows
  `connectivity_events`; `display.animation` reports the reactor
  built-in (24 frames), a custom mascot pack fetched and loaded
  (`active: custom`), then back to builtin; `net.forward` UDP
  open/close returned the expected expiries. Staged 1.0.1 OTA
  applied without force; board running 1.0.1.
- Follow-up done: the workflow's beta base is now `1.0.2-beta.`
  (build-repo commit e542506, via the browser; the gh token has no
  workflow scope).

### Run #38 — 2026-11-26 screenshot (display.screenshot)

- New Link command `display.screenshot`: the ST7789 has no MISO
  wired (write-only), so `lcd_draw()` mirrors every successful draw
  into a PSRAM shadow framebuffer (240×135 RGB565). Capture freezes
  the shadow under the LCD lock and serves an RLE stream
  (`rgb565-rle`, pure-header codec `geek_screenshot_rle.h`) in
  4096-byte base64 chunks; `chunk: 0` captures fresh, later chunks
  serve the frozen frame. `tools/geek_screenshot.py` decodes chunks
  to PNG. Limits: GRAM content only (physical defects invisible;
  `backlight_on` flags a dark screen), animation frames may mix
  two frames mid-draw.
- Tests: new `test_geek_screenshot` (C-probe RLE round-trips +
  wiring); full suite 685 tests, failures=3 / errors=11 = baseline.
- Build run 45760559571: `1.0.2-beta.cf4b5b3`, app 1,773,568 B,
  SHA-256 b5255b31…; manifest verified.
- Board acceptance: PASSED (2026-11-26 ~13:40). OTA applied;
  screenshots decoded and eyeballed by the agent: animation off
  (status bar + black main area), reactor core animation, external
  page top (version bar `beta.cf4b5b3`, cyan UART1 row), and a
  Chinese text message rendered correctly. Error paths:
  `chunk: 99` and `chunk: -1` both return `invalid_param`.

### Run #39 — 2026-11-26 product control migration (Phase 3)

- Geek-only Link commands moved behind the SDK product-integration
  hooks: registration now lives in
  `components/geek_product_control/integration/geek_product_control_extension.inc`
  (selected in the root CMakeLists, forwarded as build properties),
  dispatch in `geek_product_command()` (main/app.c, NULL for non-Geek
  commands). `main/noise_control.cpp` keeps only the three upstream
  hooks plus the shared OTA engine (`geek.ota` unchanged). No
  user-visible change: same commands, same feature gating, same
  register content (17,480 bytes on the wire).
- Tests: new `test_geek_product_control` (9 tests); registration
  assertions repointed at the component .inc; full suite 695 tests,
  failures=3 / errors=11 / skipped=7 = baseline.
- Build run 45769387015: `1.0.2-beta.b635c64`, app 1,773,568 B,
  SHA-256 d5b3b419…; manifest verified.
- Board acceptance: PASSED (2026-11-26 ~14:45). OTA applied and
  validated; `device.features` config intact (serial on, h1=swd,
  pin6=adc, chat_post off); `gpio.adc` 3159 mV; external page probe
  `fingerprint … halted 0`, model STM32F103 64KB; `display.screenshot`
  metadata + chunk 0 OK; `swd.connect` fingerprint profile_ok,
  `swd.disconnect` resumed STM32 heartbeats immediately
  (seq 6421+ via `serial.read`).

### Run #40 — 2026-11-26 avatar sync (Phase 3)

- `display.animation` gains mode `avatar`: follow the paired
  Muse's current avatar still image. The identity reply's
  `result.avatar` (image path + assets revision, parsed by the
  ported `avatar_identity`) feeds `geek_animation`; the WebP is
  fetched from the VM in ranged GETs (512 KB cap), decoded with
  libwebp (decoder only, pinned v1.6.0 archive) to at most
  112 px edge, held in PSRAM and drawn centred in the main area.
  Cache is PSRAM-only (no files partition on this board): it
  re-fetches after a reboot and whenever the identity revision
  changes; until a fetch succeeds the built-in animation plays.
  No partition change, OTA-able.
- Tests: `GeekAvatarSyncTest` (5 tests, identity parser C probe);
  full suite 700 tests, failures=3 / errors=11 / skipped=7 =
  baseline.
- Builds: 45771556766 FAILED (C++ mangling of
  `avatar_identity_parse`; fixed with extern "C" guards),
  45772653392 OK but the avatar fetch buffers exceeded the PSRAM
  budget on board; 45773252194 `1.0.2-beta.55a9e20`, app
  1,839,104 B, SHA-256 7a07fc97…; manifest verified.
- Board acceptance: PASSED (2026-11-26 ~15:05). OTA applied and
  validated; log shows `avatar identity updated` then
  `avatar image loaded: 112x112`; `display.animation` reported
  mode/avatar active/loaded. Board restored to builtin after
  the check.

### Run #41 — 2026-11-26 avatar colour fix

- First on-board avatar screenshot showed swapped colours: the
  WebP decoder emits byte-swapped RGB565 (`WEBP_SWAP_16BIT_CSP`),
  so `lcd_draw_avatar_image` now byte-swaps pixels on the way
  into the panel buffer.
- Build 45774816544, `1.0.2-beta.1a0895e`, app 1,839,104 B.
- Board acceptance (user-requested switch to avatar mode):
  full 5-chunk screenshot assembly verified correct colours;
  external-page round trip returns to the avatar; explicit
  reboot keeps avatar mode and re-fetches the image. Board
  left in avatar mode.

## Run #42 — TF serial logger (2026-11-27)
- Build run 45973475865, version 1.0.2-beta.36ec942 (feat/tflog,
  merged to main 36ec942). Publish job transient failure; full
  rerun succeeded.
- Prototype gates: G1 app +65,536 B; G2 mount/write/CRC and
  pull-reinsert recovery PASS; G3 heartbeat zero-drop PASS.
- Board acceptance: OTA applied; device.features tflog=true
  reboot; log.status running/mounted; segment 00000001.LOG with
  #H/#A anchor and receipt-stamped #L heartbeat lines; log.search
  returns matching line. Board left with tflog ON.

## Run #43 — TF log timestamped names + delete (2026-11-27)
- Builds 45978657189 (1.0.2-beta.2db2042) and 45991964332
  (1.0.2-beta.cc6270b), merged to main cc6270b.
- Segment filenames carry the segment start wall time
  (YYYYMMDD-HHMMSS-<seq>.LOG, local time); before time sync a
  segment is UNSYNCED-<boot>-<seq>-<mono_ms>.LOG and archives to
  the timestamped name when sync lands. seq stays the stable ID;
  log.list/read return filename. New log.delete refuses the
  segment being written.
- Board acceptance: OTA applied; new segments timestamped
  (20261127-180636-00000003.LOG, 20261127-194131-00000004.LOG);
  legacy segment readable; log.delete on the active segment
  refused ("segment is being written"), on a closed segment
  deleted and the list updated. The unsynced-to-archive
  transition itself was not observed on board (RTC kept time
  across OTA reboots); cold-boot check still pending.

## Run #44 — TF log record filter (2026-11-27)
- Build 45995094799, version 1.0.2-beta.27c8e20, merged to main
  27c8e20.
- log.config: enabled + up to 8 line prefixes (case-insensitive,
  word-boundary), sliding post-match context window (0-10000 ms,
  default 1000, re-armed by each match, context capped at 200
  lines per trigger), optional pre-match context from a 16-line
  RAM ring. Context lines are recorded as #L ctx; #H/#C carry
  the filter config; status reports matched/context/filtered.
- Board acceptance: OTA applied; prefixes #error,#warning
  filtered 8 heartbeat lines in 15 s with 0 matches; prefixes
  heartbeat matched 7 lines; Geek Config save path verified end
  to end (page task set the board config). Context (ctx) lines
  were not observed on board: the only serial source emits one
  matching line every ~2 s. Board left with filter enabled,
  prefixes #error,#warning, post 1000 ms, pre 0.
