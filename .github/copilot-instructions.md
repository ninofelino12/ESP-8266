# Felino Arduino Workspace

## Repository purpose

This repository is a documentation-first Arduino/ESP workspace, not yet an Arduino application or library. The current implementation is:

- `README.md`: project overview and the initial supported-hardware summary.
- `phase1.md`: PRD, target structure, and acceptance criteria for the workspace.
- `prd.md`: requirements for the planned realtime logger, EEPROM configuration, AP setup portal, MMC backup, Firebase RTDB sync, and Arduino CLI builds.
- `prompt.md`: reusable AI workflow and implementation rules for future ESP projects.
- `doc/index.html`: the static board reference page published by GitHub Pages.
- `doc/`: datasheets and hardware references.
- `img/`: board photos and pinout diagrams, with source/licensing records.
- `.github/workflows/pages.yml`: builds and deploys the static documentation site.

Treat `phase1.md`, `prd.md`, and `prompt.md` as project guidance. Use `prd.md` for the data logger's functional requirements. Do not assume directories or template files described as future work in `phase1.md` already exist.

## Build, preview, and validation

There is currently no firmware source tree, PlatformIO project, Arduino sketch, package manifest, test runner, or lint configuration. Do not claim a firmware build or test result unless one was actually run.

The GitHub Pages deployment is reproduced locally with:

```sh
mkdir -p _site/img
cp -R doc/. _site/
cp -R img/. _site/img/
touch _site/.nojekyll
python3 -m http.server 8000 --directory _site
```

Open `http://localhost:8000/`. Stop the preview server before removing or changing `_site/`.

Pushing to `main` deploys changes automatically only when `doc/**`, `img/**`, or `.github/workflows/pages.yml` changes. A workflow dispatch can also be run manually. Validate HTML, links, and image paths against the `_site/` layout because deployment copies `doc/` and `img/` to different locations within the artifact.

There is no single-test command or unit-test suite yet.

## Architecture and deployment layout

GitHub Pages does not publish repository files directly:

1. `.github/workflows/pages.yml` creates `_site/`.
2. The contents of `doc/` are copied to `_site/`.
3. The contents of `img/` are copied to `_site/img/`.
4. `_site/` is uploaded as the Pages artifact and deployed.

Keep static asset paths inside `doc/index.html` compatible with this layout. Public links may refer to `./img/...` only if the corresponding file exists under `img/` and is copied by the workflow.

For future firmware projects, follow the planned `_template` structure from `phase1.md`: separate project documentation, board-specific hardware/pin maps, configuration, and source. Keep hardware configuration separate from application logic.

## Hardware conventions

Hardware-specific assumptions are authoritative only for the boards explicitly identified:

| Board ID | I2C SDA | I2C SCL |
|---|---:|---:|
| `esp8266-nodemcu` | D2 / GPIO4 | D1 / GPIO5 |
| `esp32-devkit` | GPIO21 | GPIO22 |

Never transfer one board's pin map to another board. NodeMCU ESP-12E and ESP32 DevKit mappings must remain separate.

Before wiring or generating code:

- Identify the board and board revision; ESP32 DevKit V1 variants and clones differ.
- Verify OLED controller, resolution, I2C address, and 3.3 V compatibility. Common SSD1306 address `0x3C` is not guaranteed; `0x3D` is also possible.
- Check boot/strapping pins, flash pins, ADC/PWM/I2C/SPI/UART conflicts, input-only GPIOs, and operating voltage.
- Do not assume the official ESP32-DevKitC schematic or pinout image represents every clone.
- Never invent a datasheet value or module behavior. Use the PDFs in `doc/` and `doc/SOURCES.md`, or ask for the physical board/module details.
- Keep 3.3 V logic and power requirements explicit; do not connect 5 V I2C signals to ESP GPIOs without verified compatibility.

SSD1306 OLED I2C is the only display profile currently documented. SH1106 and additional modules belong in future hardware profiles only when their documentation and wiring are explicitly established.

## Documentation conventions

- Keep board names, board IDs, pin mappings, and wiring tables synchronized across `README.md`, `doc/index.html`, and future project hardware documentation.
- Do not describe an unimplemented board, template, profile, or feature as available.
- Preserve attribution and license information when modifying or adding board images or pinouts. Update `img/SOURCES.md` and `doc/SOURCES.md` when adding sourced assets.
- If wiring or a board configuration changes, update the related README, pin map, and static reference page in the same change.
- Keep credentials and Wi-Fi secrets out of source control. Use uncommitted local configuration for secrets.
- Link locally available datasheets and evidence rather than asserting specifications without a source.
