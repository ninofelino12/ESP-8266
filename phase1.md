# PRD: Felino Arduino Workspace

**Project:** Felino Arduino Workspace
**Version:** 1.0.0
**Status:** Phase 1
**Platform:** Arduino Framework / ESP8266 / ESP32

## 1. Project Overview

Felino Arduino Workspace adalah workspace pengembangan Arduino dan ESP berbasis template terstandarisasi.

Tujuan:
- Struktur direktori konsisten.
- Dokumentasi hardware.
- GPIO mapping akurat.
- Wiring diagram.
- Library references.
- AI-readable instructions.
- Reusable project templates.

## 2. Supported Hardware

### ESP8266

| Board ID | Board |
|---|---|
| esp8266-nodemcu | NodeMCU ESP8266 |
| wemos-d1-mini | WeMos D1 Mini |
| lolin-d1-mini | LOLIN D1 Mini |

### ESP32

| Board ID | Board |
|---|---|
| esp32-devkit | ESP32 DevKit |
| esp32-cam | AI Thinker ESP32-CAM |
| esp32-s3 | ESP32-S3 DevKit |

Setiap board wajib mempunyai dokumentasi dan GPIO mapping tersendiri.

## 3. Supported Modules

### Display
- OLED 0.9 inch
- SSD1306
- SH1106
- I2C

### Other Modules
- RTC
- MicroSD
- GPRS
- RFID
- Relay
- Temperature sensors
- GPS
- OV2640 Camera
- I2C, SPI, UART modules

## 4. Directory Structure

    Felino-Arduino/
    ├── README.md
    ├── AGENTS.md
    ├── prd.md
    ├── .gemini/
    │   └── skills/
    │       └── arduino-esp/
    │           └── SKILL.md
    ├── _template/
    │   ├── README.md
    │   ├── AGENTS.md
    │   ├── docs/
    │   │   ├── PROJECT.md
    │   │   ├── HARDWARE.md
    │   │   ├── PINMAP.md
    │   │   ├── LIBRARIES.md
    │   │   ├── CONFIG.md
    │   │   └── DATAFLOW.md
    │   ├── assets/
    │   │   ├── boards/
    │   │   ├── displays/
    │   │   ├── modules/
    │   │   └── wiring/
    │   └── src/
    └── projects/

## 5. Board Documentation

Setiap board mempunyai:

- BOARD.md
- board.jpg
- pinout.png

BOARD.md menjelaskan:

1. Board identity
2. Manufacturer
3. MCU
4. Board revision
5. Flash and RAM
6. Operating voltage
7. GPIO availability
8. Reserved GPIO
9. Boot strapping pins
10. ADC/PWM/I2C/SPI/UART
11. Hardware limitations
12. Display compatibility

## 6. OLED Wiring Profiles

Profil wajib:

- ESP8266 + OLED
- ESP32 DevKit + OLED
- ESP32-S3 + OLED

Setiap profil mencakup:

- Wiring diagram
- SDA
- SCL
- VCC
- GND
- Display controller
- I2C address
- Library configuration
- Example code

Jangan mengasumsikan pin I2C setiap board sama.

## 7. AI Development Rules

AI wajib:

1. Membaca AGENTS.md.
2. Membaca PROJECT.md dan HARDWARE.md.
3. Mengidentifikasi board sebelum membuat kode.
4. Membaca PINMAP.md.
5. Memeriksa konflik pin.
6. Memeriksa kebutuhan tegangan modul.
7. Tidak mengarang spesifikasi hardware.
8. Meminta informasi jika board tidak teridentifikasi.
9. Memisahkan konfigurasi hardware dari business logic.
10. Memperbarui dokumentasi jika wiring berubah.

## 8. Project Template Requirements

Setiap proyek baru harus berasal dari _template.

Dokumentasi minimal:

- README.md
- AGENTS.md
- docs/PROJECT.md
- docs/HARDWARE.md
- docs/PINMAP.md
- docs/LIBRARIES.md
- docs/CONFIG.md
- docs/DATAFLOW.md

## 9. Development Phases

### Phase 1: Workspace Foundation

- [x] Menentukan struktur workspace.
- [x] Menentukan board registry.
- [ ] Membuat direktori template.
- [ ] Membuat dokumentasi utama.
- [ ] Membuat AI instructions.
- [ ] Membuat hardware profiles.
- [ ] Membuat OLED wiring profiles.

### Phase 2: Hardware Library

- Board profiles.
- GPIO references.
- Module profiles.
- Wiring diagrams.
- Pin conflict references.

### Phase 3: AI Integration

- AI project instructions.
- Hardware-aware code generation.
- Library recommendations.
- Code review.
- Wiring validation assistance.

### Phase 4: Project Management

- Project generator.
- Reusable templates.
- Documentation consistency.
- Project registry.

## 10. Acceptance Criteria

- [ ] Struktur direktori tersedia.
- [ ] Enam board terdaftar.
- [ ] Setiap board memiliki profil dokumentasi.
- [ ] Foto board dan pinout dapat ditambahkan.
- [ ] OLED ESP8266 dan ESP32 memiliki wiring terpisah.
- [ ] Template dapat digunakan ulang.
- [ ] AI dapat membaca instruksi.
- [ ] GPIO mapping tidak menggunakan asumsi lintas board.

## 11. Design Principles

- Documentation first.
- Hardware-specific configuration.
- Reusable templates.
- AI-friendly structure.
- Modular architecture.
- Low resource consumption.
- Easy maintenance.
- Expandable hardware registry.

---

Project: Felino Arduino Workspace
Owner: Felino
Document: prd.md