# PRD: ESP Realtime Data Logger

**Project:** Felino Arduino ESP Data Logger
**Versi:** 1.0.0
**Status:** Rancangan
**Framework:** Arduino
**Build tool:** Arduino CLI

## 1. Ringkasan

Membuat aplikasi realtime data logger untuk ESP8266 NodeMCU, ESP32 DevKit, dan ESP32-CAM AI Thinker. Data dicatat ke kartu MMC/microSD sebagai salinan lokal, kemudian dikirim ke Firebase Realtime Database (RTDB). Data yang belum berhasil dikirim tetap di kartu dan dikirim ulang ketika koneksi pulih.

Kredensial Wi-Fi dan konfigurasi environment Firebase disimpan pada EEPROM (termasuk EEPROM emulation pada flash untuk board yang menggunakannya). Jika perangkat tidak berhasil tersambung ke Wi-Fi, perangkat menyediakan Access Point konfigurasi dengan halaman web lokal untuk mengubah environment. Setiap target memiliki konfigurasi pin dan build sendiri, serta dapat dikompilasi menggunakan Arduino CLI.

## 2. Tujuan

- Mencatat data secara lokal agar koneksi internet atau Firebase yang terputus tidak langsung menyebabkan kehilangan data.
- Mengirim data ke Firebase RTDB dan mengirim ulang antrean lokal yang belum terkonfirmasi.
- Memungkinkan Wi-Fi dan konfigurasi Firebase diubah di perangkat melalui halaman web Access Point.
- Mendukung ESP8266 NodeMCU, ESP32 DevKit, serta ESP32-CAM AI Thinker dengan pin dan konfigurasi board terpisah.
- Menyediakan script Arduino CLI untuk compile satu target atau ketiga target.

## 3. Di luar cakupan

- Menentukan jenis sensor, format payload final, periode sampling, zona waktu, dan metode autentikasi Firebase sebelum diputuskan untuk proyek aktual.
- Mengunggah firmware ke board atau menjalankan serial monitor secara otomatis.
- Menganggap semua board ESP32 DevKit atau ESP32-CAM memiliki revisi dan komponen yang sama.
- Menghapus data lokal yang belum dikonfirmasi terkirim untuk mengatasi ruang kartu yang penuh.

## 4. Target hardware dan board

| Target | Profil awal | FQBN awal |
|---|---|---|
| ESP8266 | NodeMCU berbasis ESP-12E | `esp8266:esp8266:nodemcuv2` |
| ESP32 | ESP32 DevKit umum | `esp32:esp32:esp32` |
| ESP32-CAM | AI Thinker ESP32-CAM | `esp32:esp32:esp32cam` |

FQBN pada tabel adalah nilai awal untuk package board Arduino yang sesuai. Sebelum implementasi final, verifikasi FQBN terhadap versi core yang dipasang menggunakan `arduino-cli board listall` atau dokumentasi package board. Simpan konfigurasi board di satu tempat yang digunakan oleh script dan dokumentasi.

Board ESP32-CAM harus dinyatakan sebagai profil AI Thinker. Jika hardware aktual bukan AI Thinker, identifikasi model dan FQBN terlebih dahulu; jangan mengasumsikan profil atau pinout yang sama.

### Profil OLED 0,9 inci I2C

OLED 0,9 inci adalah periferal opsional. Pin I2C disimpan pada konfigurasi masing-masing target agar dapat diubah kemudian tanpa mengubah logika aplikasi bersama.

| Target | SDA awal | SCL awal | Catatan |
|---|---|---|---|
| ESP8266 NodeMCU | D2 / GPIO4 | D1 / GPIO5 | Nilai awal sesuai profil NodeMCU di repository. |
| ESP32 DevKit | GPIO21 | GPIO22 | Nilai awal; verifikasi board aktual. |
| ESP32-CAM AI Thinker | Belum ditetapkan | Belum ditetapkan | Pilih setelah memeriksa pin kamera, microSD, dan revisi board yang digunakan. |

Letakkan definisi pin OLED dalam file konfigurasi per target, misalnya `targets/<target>/board_config.h`, dengan nama konsisten seperti `OLED_SDA_PIN` dan `OLED_SCL_PIN`. Konfigurasi harus mudah diedit dan tidak boleh memakai satu nilai global untuk semua board. Perubahan pin wajib diikuti pembaruan `docs/PINMAP.md`.

Ukuran 0,9 inci tidak cukup untuk menyimpulkan controller, resolusi, alamat I2C, atau tegangan modul. Verifikasi informasi tersebut dari label/datasheet modul sebelum mengaktifkan OLED; jangan menganggap semua modul menggunakan SSD1306 atau alamat `0x3C`.

## 5. Persyaratan fungsional

### 5.1 Konfigurasi dan EEPROM

- Simpan konfigurasi Wi-Fi (SSID dan password) serta environment Firebase yang diperlukan di EEPROM atau EEPROM emulation yang sesuai dengan core board.
- Environment Firebase minimal mendukung database URL, path RTDB, dan informasi autentikasi perangkat yang dipilih. Dokumentasikan metode autentikasi dan hanya minta field yang diperlukan; jangan mengasumsikan database dapat diakses tanpa autentikasi atau menyimpan private key service account di perangkat.
- Gunakan skema konfigurasi berversi dan validasi ukuran/isi sebelum menyimpan agar data konfigurasi tetap muat di kapasitas EEPROM yang tersedia pada setiap target.
- Tulis konfigurasi hanya setelah pengguna menekan simpan. Jangan melakukan write berulang pada setiap loop atau percobaan koneksi.
- Simpan konfigurasi baru secara aman: konfigurasi lama tetap digunakan jika validasi atau penyimpanan gagal. Sediakan cara untuk menghapus konfigurasi dan kembali ke mode setup.
- Jangan menampilkan password atau token lengkap kembali di halaman web atau log serial. Nilai rahasia harus ditandai sebagai rahasia dan tidak dimasukkan ke source control.
- EEPROM emulation dapat menyimpan data sebagai bagian dari flash dan tidak otomatis terenkripsi. Dokumentasikan batasan ini serta proteksi akses yang digunakan.

### 5.2 Wi-Fi dan halaman konfigurasi Access Point

- Saat boot, coba sambungkan ke Wi-Fi tersimpan selama batas waktu yang dapat dikonfigurasi.
- Jika koneksi Wi-Fi gagal atau terputus dan gagal pulih setelah kebijakan retry yang terdokumentasi, aktifkan Access Point konfigurasi dan web server lokal.
- Halaman konfigurasi, yang hanya tersedia melalui jaringan AP setup, memungkinkan pengguna mengubah SSID/password Wi-Fi dan field environment Firebase. Berikan status koneksi Wi-Fi, status RTDB, serta status kartu, tetapi jangan membocorkan kredensial.
- Lindungi Access Point konfigurasi dengan password; jangan memakai password universal yang sama untuk perangkat produksi. Dokumentasikan cara pengguna memperoleh atau mengatur password setup.
- Validasi input sebelum menyimpan. Setelah konfigurasi valid disimpan, coba koneksi ulang dan tampilkan hasilnya secara jelas. Jika koneksi gagal, pertahankan konfigurasi yang dapat dipulihkan dan sediakan akses kembali ke portal.
- Kegagalan RTDB saat Wi-Fi masih tersambung tidak boleh dianggap sebagai kegagalan Wi-Fi atau menghapus kredensial. Data tetap dicatat di kartu dan pengiriman diantrekan.

### 5.3 Pencatatan lokal MMC/microSD

- Catat setiap sampel/event ke MMC/microSD dan pastikan penulisan berhasil sebelum mencoba mengirim record ke RTDB, sehingga kartu menjadi sumber backup lokal, bukan hanya tempat menyimpan data setelah kegagalan jaringan.
- Simpan record dengan skema berversi, ID perangkat, sequence/ID unik, timestamp atau penanda kualitas waktu, dan payload data. Format file append-only yang dapat dipulihkan setelah kehilangan daya harus dipilih dan didokumentasikan.
- Periksa status mount, kapasitas, dan hasil penulisan. Jika kartu tidak tersedia atau penulisan gagal, nyatakan kondisi degraded secara eksplisit; jangan melaporkan record sebagai tersimpan lokal.
- Sediakan kebijakan rotasi/retensi untuk data yang sudah terkonfirmasi terkirim. Jangan menghapus record yang masih menunggu pengiriman secara diam-diam. Jika ruang habis, tampilkan kesalahan dan pertahankan data pending.
- Pin SPI dan konfigurasi kartu ditetapkan per board setelah verifikasi. Pada ESP32-CAM, perhitungkan konflik dengan pin kamera dan microSD onboard; jangan menganggap dua perangkat penyimpanan dapat memakai pin yang sama tanpa desain bus yang benar.

### 5.4 Pengiriman Firebase RTDB dan pemulihan antrean

- Kirim record ke path RTDB yang dikonfigurasi menggunakan koneksi TLS dengan verifikasi sertifikat; jangan memakai mode TLS tidak aman untuk produksi.
- Record baru yang sudah tersimpan lokal dikirim ketika Wi-Fi dan RTDB tersedia. Kegagalan koneksi, timeout, atau respons server tidak sukses harus mempertahankan record sebagai pending.
- Kirim ulang record pending setelah koneksi pulih dengan retry/backoff yang memiliki batas dan tidak menghambat pengambilan sampel.
- Gunakan ID record stabil sebagai key/path RTDB agar retry tidak membuat duplikasi data. Tandai record terkirim hanya setelah menerima konfirmasi sukses dari RTDB, dan simpan checkpoint sinkronisasi yang dapat dipulihkan setelah restart.
- Kegagalan RTDB tidak boleh menghentikan pencatatan MMC atau membuat perangkat berpindah ke AP selama koneksi Wi-Fi masih tersedia.
- Tentukan sampling interval, format payload, path RTDB, autentikasi, dan batas retensi berdasarkan kebutuhan proyek/sensor yang sebenarnya; jangan mengarang nilai atau skema final.
- Gunakan waktu UTC dari NTP saat tersedia. Jika offline dan tidak ada RTC, simpan uptime/sequence serta tandai bahwa waktu kalender belum tersinkron; jangan membuat timestamp kalender palsu.

### 5.5 OLED dan status perangkat

- OLED 0,9 inci bersifat opsional. Jika dipasang, tampilkan status ringkas seperti Wi-Fi/AP, RTDB, MMC, dan jumlah record pending tanpa memperlihatkan rahasia.
- Pin SDA/SCL tetap mengikuti konfigurasi per target pada bagian 4 dan dapat diedit tanpa mengubah logika logger.

## 6. Struktur proyek yang diharapkan

```text
.
├── README.md
├── arduino-cli.yaml             # konfigurasi index board tambahan bila diperlukan
├── scripts/
│   └── compile.sh               # kompilasi satu atau semua target
├── lib/
│   └── FelinoCommon/            # library lokal untuk kode bersama
│       ├── library.properties
│       └── src/
│           ├── ConfigStore/
│           ├── Connectivity/
│           ├── DataLogger/
│           ├── FirebaseClient/
│           └── Storage/
├── targets/
│   ├── esp8266/
│   │   ├── esp8266.ino          # entry point sketch Arduino
│   │   └── board_config.h       # konfigurasi pin khusus target
│   ├── esp32/
│   │   ├── esp32.ino
│   │   └── board_config.h
│   └── esp32cam/
│       ├── esp32cam.ino
│       └── board_config.h
├── docs/
│   ├── HARDWARE.md
│   ├── PINMAP.md
│   ├── LIBRARIES.md
│   ├── CONFIGURATION.md
│   └── DATAFLOW.md
└── .gitignore                   # mengecualikan hasil build dan konfigurasi lokal
```

Setiap entry point harus berada di direktori sketch yang dikenali Arduino CLI dan nama file `.ino` harus sesuai dengan nama direktori sketch tersebut. Jika layout yang dipilih memerlukan direktori sketch tambahan, script wajib menunjuk ke direktori tersebut secara eksplisit.

Kode umum ditempatkan sebagai library lokal Arduino di `lib/FelinoCommon/` hanya jika API dan perilakunya kompatibel dengan core ESP8266 dan ESP32. Script harus meneruskan direktori `lib/` ke Arduino CLI melalui opsi `--libraries`. EEPROM, Wi-Fi, SPI/SD, dan Firebase dapat memerlukan adapter atau konfigurasi per platform. Perbedaan tersebut harus ditangani secara eksplisit, bukan dengan memakai pin atau asumsi hardware yang sama.

## 7. Persyaratan Arduino CLI

### Instalasi dan setup

- Dokumentasikan versi minimum Arduino CLI yang didukung atau cara memeriksa versi yang digunakan.
- Dokumentasikan penambahan board index dan instalasi core yang dibutuhkan:
  - `esp8266:esp8266`
  - `esp32:esp32`
- Sebutkan perintah pemasangan core dan dependensi yang diperlukan; jangan menginstal atau mengunduh dependency diam-diam saat compile.
- Jika core atau CLI belum tersedia, script harus gagal dengan pesan yang menyebut prasyarat yang hilang dan perintah setup terkait.
- Perintah setup dan compile harus dapat dijalankan ulang tanpa merusak konfigurasi pengguna.

Contoh alur setup yang harus divalidasi terhadap versi Arduino CLI dan URL index yang digunakan:

```sh
arduino-cli version
arduino-cli core update-index
arduino-cli core install esp8266:esp8266
arduino-cli core install esp32:esp32
```

Jika board index ESP8266 tidak terdaftar pada konfigurasi CLI, dokumentasikan URL index resmi yang diperlukan dan cara menambahkannya melalui `arduino-cli config`; jangan menimpa konfigurasi global pengguna.

### Script kompilasi

`scripts/compile.sh` harus:

- Menerima target `esp8266`, `esp32`, `esp32cam`, atau `all`.
- Mengompilasi hanya target yang diminta; `all` mengompilasi ketiganya secara berurutan dan mengembalikan exit code bukan-nol jika salah satu build gagal.
- Menggunakan FQBN target yang terverifikasi dan sketch directory masing-masing.
- Menggunakan output build terpisah per target, misalnya `build/esp8266/`, `build/esp32/`, dan `build/esp32cam/`.
- Meneruskan kegagalan Arduino CLI tanpa mengubahnya menjadi status sukses.
- Menampilkan target yang sedang dikompilasi dan lokasi output.
- Menerima pemanggilan dari direktori mana pun dengan menentukan root proyek berdasarkan lokasi script.
- Menampilkan penggunaan dan gagal dengan pesan yang jelas untuk argumen tidak dikenal atau toolchain yang belum siap.
- Tidak mengunggah firmware, menghapus file pengguna, atau melakukan instalasi jaringan sebagai bagian dari compile.

Contoh antarmuka yang diharapkan:

```sh
./scripts/compile.sh esp8266
./scripts/compile.sh esp32
./scripts/compile.sh esp32cam
./scripts/compile.sh all
```

### Dependensi dan konfigurasi

- Semua library untuk sensor, kartu MMC/microSD, halaman web, Firebase RTDB, dan OLED dicatat beserta nama paket Arduino CLI dan versi atau kebijakan versinya di `docs/LIBRARIES.md`.
- Gunakan konfigurasi build yang dapat direproduksi; dokumentasikan opsi board yang non-default.
- Hasil build dan file konfigurasi lokal/rahasia tidak boleh masuk version control.

## 8. Baseline firmware

- Setiap target memiliki sketch yang mengimplementasikan logger, konektivitas, konfigurasi, dan adapter hardware yang dibutuhkan.
- Sumber data/sensor harus dapat ditambahkan tanpa mengikat transport RTDB atau format penyimpanan ke satu jenis sensor tertentu.
- Pin SDA/SCL OLED dan pin SPI MMC harus berasal dari konfigurasi target yang dapat diedit dan diverifikasi.
- Target ESP32-CAM harus mempertahankan batasan pin kamera dan periferal berdasarkan modul yang teridentifikasi; jangan menggunakan GPIO kamera untuk fungsi lain.

## 9. Dokumentasi proyek

`README.md` menjelaskan:

- Target board dan FQBN masing-masing.
- Prasyarat Arduino CLI dan langkah setup core.
- Cara mengompilasi satu target dan seluruh target.
- Lokasi hasil build, langkah upload manual, dan cara memeriksa log serial bila diperlukan.
- Langkah setup awal AP, koneksi Wi-Fi, konfigurasi RTDB, serta pemulihan data pending.
- Batasan hardware, wiring MMC/OLED, dan cara mengakses konfigurasi perangkat.

`docs/HARDWARE.md` mencatat board, modul, revisi, sumber referensi, dan asumsi yang telah diverifikasi. `docs/PINMAP.md` mencatat pemakaian pin terpisah per target untuk OLED, MMC, kamera, dan periferal lain. `docs/CONFIGURATION.md` menjelaskan field konfigurasi, penyimpanan EEPROM, dan portal AP tanpa mempublikasikan nilai rahasia. `docs/DATAFLOW.md` menjelaskan alur sample → MMC → RTDB, retry, checkpoint, dan retensi. `docs/LIBRARIES.md` mencatat seluruh dependensi eksternal.

Jangan menyatakan pengiriman Firebase, pemulihan data, atau operasi hardware berhasil hanya berdasarkan keberhasilan kompilasi.

## 10. Kriteria penerimaan

- [ ] Tersedia tiga sketch entry point: ESP8266 NodeMCU, ESP32 DevKit, dan ESP32-CAM AI Thinker, masing-masing dengan konfigurasi pin terpisah.
- [ ] Setiap target mempunyai FQBN yang diverifikasi terhadap core board yang didukung.
- [ ] `./scripts/compile.sh <target>` membangun hanya target tersebut.
- [ ] `./scripts/compile.sh all` membangun ketiga target dan gagal jika ada satu build gagal.
- [ ] Build artifacts tiap target tidak saling menimpa.
- [ ] Script memberi pesan setup yang dapat ditindaklanjuti saat Arduino CLI atau core belum tersedia.
- [ ] Tidak ada upload atau instalasi dependency tersembunyi ketika menjalankan compile.
- [ ] README mencakup setup dan perintah compile untuk satu target maupun semua target.
- [ ] Hardware dan pin terdokumentasi terpisah; tidak ada asumsi pemetaan GPIO lintas-board.
- [ ] Pin SDA/SCL OLED dapat diedit per target; pin ESP32-CAM tidak ditetapkan sebelum konflik hardware diverifikasi.
- [ ] Kredensial Wi-Fi dan environment RTDB tersimpan di EEPROM/EEPROM emulation dan dapat diperbarui melalui portal AP tanpa ditampilkan kembali sebagai teks rahasia.
- [ ] Perangkat mengaktifkan portal AP setelah Wi-Fi gagal tersambung sesuai timeout/retry yang didokumentasikan.
- [ ] Setiap record ditulis ke MMC sebelum dianggap aman, dan status MMC yang gagal terlihat jelas.
- [ ] Record yang gagal dikirim tetap pending dan berhasil dikirim ulang setelah Wi-Fi/RTDB pulih tanpa membuat duplikasi akibat retry.
- [ ] Record pending tidak dihapus otomatis saat kartu penuh atau perangkat restart.
- [ ] Ketiga target berhasil dikompilasi; perilaku Wi-Fi/AP, EEPROM, MMC, RTDB, dan pemulihan antrean diuji sesuai hardware yang didukung.

## 11. Tahapan implementasi

1. Tetapkan FQBN dan versi core yang didukung untuk ketiga board.
2. Tentukan sensor/sumber data, sampling interval, schema payload, autentikasi Firebase, path RTDB, dan kebijakan waktu.
3. Verifikasi konfigurasi pin Wi-Fi, OLED, MMC, serta kamera untuk masing-masing board.
4. Implementasikan penyimpanan konfigurasi EEPROM dan portal provisioning AP.
5. Implementasikan pencatatan MMC, antrean offline, pengiriman RTDB idempotent, retry, dan pemulihan setelah restart.
6. Implementasikan script compile Arduino CLI dan dokumentasikan setup/dependensi.
7. Uji build untuk semua target serta uji offline, restart/power loss, kartu penuh/tidak tersedia, dan RTDB tidak tersedia pada hardware aktual.
