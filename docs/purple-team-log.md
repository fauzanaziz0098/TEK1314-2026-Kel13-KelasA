# Purple Team Log - Kelompok 13

Catatan siklus kolaborasi Red Team dan Blue Team untuk proyek KIRA Smart Chicken Feeder (Target Ubuntu VM `192.168.13.34`, Kali Attacker `192.168.13.2`, Router MikroTik CHR, dan Security Onion `192.168.13.66` via TZSP sniffer). Setiap skenario pada `attack_plan.md` dicatat melalui lima tahap: **1. Serang**, **2. Deteksi**, **3. Analisis**, **4. Respons**, dan **5. Verifikasi**.

| Tahap | Red Team | Blue Team | Bukti |
| :--- | :--- | :--- | :--- |
| 1. Serang (Baseline) | Eksekusi skenario serangan pada kondisi hardening saat ini | Amati deteksi di Security Onion dan log | Screenshot serangan + status deteksi awal |
| 2. Deteksi | - | Alert atau log yang muncul, atau catatan tidak terdeteksi | Screenshot alert / hasil grep log |
| 3. Analisis | Teknik dan tujuan serangan | Root cause, kontrol yang kurang | Catatan analisis |
| 4. Respons (Mitigasi) | - | Kontrol tambahan: rule alert, patch, firewall | Diff konfigurasi |
| 5. Verifikasi | Ulangi serangan yang sama | Konfirmasi serangan gagal atau terblokir | Screenshot percobaan ulang |

Baris tidak boleh dihapus. Kegagalan deteksi tetap dicatat sebagai temuan. Bukti disimpan di `docs/redteam/evidence/` atau `docs/blueteam/evidence/`. Status: `Belum`, `Berjalan`, `Selesai`.

## Ringkasan Status

| Skenario | Kategori | 1 | 2 | 3 | 4 | 5 |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| SCN-01 | Reconnaissance | Selesai | Selesai | Selesai | Selesai | Selesai |
| SCN-02 | Exploitation / Akses | Selesai | Selesai | Selesai | Selesai | Selesai |
| SCN-03 | Web / API / Protokol IoT | Selesai | Selesai | Selesai | Selesai | Selesai |

## Detail Catatan per Skenario

### SCN-01: Network Reconnaissance & Service Enumeration (`nmap`)
- **1. Serang:** Menjalankan `nmap -sS -sV -p 22,2222,3000,27017 192.168.13.34` dari Kali (`192.168.13.2`). Port `22` dan `27017` `filtered`, port `2222` (SSH) dan `3000` (API Node.js Express) `open`.
- **2. Deteksi:** Trafik cerminan TZSP diterima Security Onion (`192.168.13.66`) dan tercatat pada log jaringan/IDS.
- **3. Analisis:** Pemetaan port dan versi layanan terbuka; port `2222` dan `3000` memang dibuka pada whitelist UFW sementara banner versi layanan masih terlihat.
- **4. Respons:** Mempertahankan kebijakan UFW (`default deny incoming`, hanya membuka `2222/tcp` dan `3000/tcp`) serta binding localhost MongoDB (`127.0.0.1:27017`).
- **5. Verifikasi:** Konfigurasi port dan firewall tetap terjaga sesuai baseline keamanan server.

### SCN-02: SSH Credential Brute-Force & Lockout Assessment (`Hydra`)
- **1. Serang:** Menjalankan `hydra -L ~/users.txt -P ~/passwords.txt -s 2222 ssh://192.168.13.34 -t 4 -V` ke layanan OpenSSH port `2222`.
- **2. Deteksi:** Percobaan login gagal berulang tercatat di `/var/log/auth.log` dan terpantau pada monitoring jaringan; IP `192.168.13.2` terkena blokir otomatis Fail2ban (`sudo fail2ban-client status sshd`).
- **3. Analisis:** Autentikasi password SSH masih aktif sebelum migrasi penuh ke SSH key, namun Fail2ban berhasil membatasi upaya brute-force.
- **4. Respons:** Memastikan jail `sshd` Fail2ban aktif di port `2222` (`unbanip 192.168.13.2` dilakukan sementara setelah verifikasi blokir agar pengujian skenario berikutnya berjalan lancar).
- **5. Verifikasi:** IP penyerang terblokir otomatis oleh Fail2ban saat percobaan brute-force berulang.

### SCN-03: Unauthenticated Sensor Spoofing & Telemetry Injection (`Burp Suite Repeater`)
- **1. Serang:** Mengirim `POST /api/telemetry/volume HTTP/1.1` ke `192.168.13.34:3000` lewat **Burp Suite Repeater** dengan payload `{"jarak_cm": 9.9, "estimasi_persen": 90.9}` (serta uji payload `{"jarak_cm": 5.0, "estimasi_persen": 95.0}`) tanpa header autentikasi. Sebelum mitigasi, server merespons `HTTP/1.1 201 Created` dan menyimpan data ke MongoDB (`iotdb`, koleksi `log_sistems`).
- **2. Deteksi:** Request HTTP POST terekam melalui cerminan TZSP / log HTTP di Security Onion, tanpa alert khusus dari aturan tambahan Suricata.
- **3. Analisis:** Endpoint `/api/telemetry/volume` pada `server.js` awal belum memeriksa header autentikasi perangkat (`x-api-key`), sehingga data sensor palsu dapat masuk ke basis data.
- **4. Respons:** Menyalin `~/iot-backend/server.js.hardened` ke `~/iot-backend/server.js` untuk mewajibkan validasi `x-api-key`, lalu me-restart layanan dengan `pm2 restart iot-backend --update-env`.
- **5. Verifikasi:** Mengirim ulang request spoofing yang sama di **Burp Suite Repeater** tanpa `x-api-key` menghasilkan **`HTTP/1.1 401 Unauthorized`**, dan pengecekan `db.log_sistems.find().sort({_id:-1}).limit(1).pretty()` di `iotdb` memastikan tidak ada data palsu baru yang masuk.