# esp32-muse-geek-build

用 GitHub Actions 编译 [legogogo25e/esp32-muse-geek](https://github.com/legogogo25e/esp32-muse-geek) 的 ESP32 固件，不用在本地装 ESP-IDF。

源码仓是 Geek 专用（Waveshare ESP32-S3-GEEK 一块板，带状态屏和 `device.display` 命令）；本仓只负责编译、发布固件和网页烧录。

- 工具链：`espressif/idf:v6.0.3`（上游要求 ESP-IDF v6.0.3，其他版本不支持）
- 源码默认编 `main` 分支，可在 Run workflow 时改 `upstream_ref` 钉到某个 commit
- 板子：`waveshare-s3-geek` = Waveshare ESP32-S3-GEEK（ESP32-S3R2，16 MB flash，2 MB PSRAM，1.14" ST7789 状态屏）
- 产物：`muse-gadget.bin`（app-only，可用于 OTA）+ `bootloader.bin` + `partition-table.bin` + `ota_data_initial.bin` + `phy_init_data.bin` + `flasher_args.json`，见 Actions 产物，保留 30 天；每次成功编译还会把同一套文件发布到 GitHub Pages（见下节），不用登录就能下载
- **固件里不带 SDK token**，烧好后按源码仓 README 用串口 / BLE 再设置 token

## 最新固件（GitHub Pages，免登录下载）

每次编译成功后，工作流自动把固件发布到：

- 网站：https://legogogo25e.github.io/esp32-muse-geek-build/
- 清单：https://legogogo25e.github.io/esp32-muse-geek-build/manifest.json
- 板子清单：https://legogogo25e.github.io/esp32-muse-geek-build/firmware/waveshare-s3-geek/manifest.json

`manifest.json` 的结构（`boards.<board>` 里）：`board` / `target` / `project_version` / `upstream_sha` / `idf_version` / `built_at_utc` / `github_run`，以及 `flash`（`mode` / `freq` / `size`）和 `files` 数组，每项含 `name` / `url`（相对路径）/ `address`（十进制烧录地址）/ `size` / `sha256`。

**烧录地址一律以 manifest / `flasher_args.json` 为准，不要硬编码 IDF 默认值。** muse-gadget 的实测地址（S3 板）：分区表 `0x10000`（不是 IDF 默认的 0x8000）、app（muse-gadget.bin）`0x20000`（ota_0 槽）；bootloader 地址和 flash 参数以本次编译的 manifest 为准。

5 个文件都要烧：`ota_data_initial.bin` 会把 OTA 槽位选择重置回 ota_0，没有它，之前从 ota_1 启动过的板子会继续跑旧固件。

## 网页烧录

本仓根目录的 `flasher.html` 就是烧录页（发布时由工作流拷进 Pages）：打开后自动从上面的 Pages 地址拉最新固件和 manifest，核对 SHA-256，用 esptool-js（Web Serial）在桌面 Chrome / Edge 里把固件烧到插在电脑上的 ESP32，烧完自动重启并打开串口监视器看启动 log。Web Serial 只在桌面 Chrome / Edge 可用，Safari、iPhone、微信内置浏览器都不支持。

## 首次设置（只做一次）

1. **PAT**：在 GitHub 建一个 fine-grained PAT，只读、只限 `legogogo25e/esp32-muse-geek` 一个仓（源码仓是 private，工作流 checkout 需要它），加到本仓 Settings → Secrets → Actions，名字 `UPSTREAM_REPO_TOKEN`。PAT 值不要贴在聊天里。
2. **Pages**：本仓 Settings → Pages，Source 选 GitHub Actions。
3. **工作流**：`.github/workflows/build-esp32.yml` 只能经浏览器网页上传（GitHub App 没有 workflows 权限，API 推会报 403）。

## 怎么跑一次

1. 打开仓库的 **Actions** 标签页，左侧选 **Build ESP32 firmware**。
2. 点 **Run workflow**，选：
   - `board`：默认 `waveshare-s3-geek`
   - `upstream_ref`：默认 `main`，想钉死就填源码仓的 commit SHA
   - `project_version`：默认 `1.0.0`。OTA 有版本门：新版本不比运行中的新会被跳过，所以每次发新版都要涨版本号
3. 等运行变绿，点进那次运行，到底部 **Artifacts** 下载 `firmware-waveshare-s3-geek-1.0.0`。
4. 也可以直接 push 改动到 `.github/workflows/build-esp32.yml`，push 到 main 也会触发一次（默认就编 Geek 板）。

## 拿到 .bin 后怎么烧

首次烧录必须走 USB 串口（空片没有 OTA 代码）：用 Pages 上的 `flasher.html`，或按产物里的 `flasher_args.json` 的地址用 esptool 烧。

之后更新可走 Wi-Fi OTA：把 `muse-gadget.bin` 放到 GitHub Pages 上，用 `device.ota` 命令给板子一个 HTTPS URL，板子自己下载、校验、重启。新固件启动后 300 秒内连不上控制通道会自动回滚。

## 常见问题

- **运行失败先看哪一步**：`Build firmware` 是编译本身；`Checkout esp32-muse-geek` 失败多半是 `upstream_ref` 填错，或 PAT 没覆盖源码仓（先查 `UPSTREAM_REPO_TOKEN` 是否就位）。
- **改了源码想重编**：不用改仓库代码，Run workflow 时改 `upstream_ref` 为新 commit 即可。
- **改了烧录页**：推完 `flasher.html` 不会自动上线，要再触发一次工作流重新发布 Pages，跑完再到线上核对改动。
- **每次编译的 `.bin` SHA 都不同是正常的**，固件内嵌了编译时间戳；大小一样、SHA 不同不代表代码变了。
