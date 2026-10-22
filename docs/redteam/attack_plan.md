# Red Team Attack Plan: Pengujian Keamanan Target Node Replika

Dokumen ini memuat rencana pengujian penetrasi dan verifikasi keamanan oleh Red Team terhadap Target Node replika IoT Server. Rencana ini disusun untuk memverifikasi efektivitas kontrol pertahanan (Blue Team) serta mengidentifikasi celah keamanan nyata pada arsitektur sistem berbasis IoT.

---

## 1. Lingkungan & Target Pengujian

- **Attacker Node:** `192.168.13.2` (Kali Linux - Segmen Attacker)
- **Target Node (Replika):** `192.168.13.34` (Ubuntu Server - Segmen Target)
- **Router Gateway:** MikroTik CHR (`192.168.13.1` / `192.168.13.33`)
- **Layanan pada Target:**
  - Remote Management: OpenSSH (Custom Port `2222/tcp`)
  - Application Layer: Node.js Express REST API (Port `3000/tcp`)
  - Database Layer: MongoDB (Localhost Binding `127.0.0.1:27017`)

---

## 2. Matriks Rencana Skenario Serangan

| ID Skenario | Kategori Serangan        | Layanan / Endpoint Target                         | Tools yang Digunakan |
| :---------- | :----------------------- | :------------------------------------------------ | :------------------- |
| **SCN-01**  | Reconnaissance           | Seluruh port Target (`192.168.13.34`)             | `nmap`               |
| **SCN-02**  | Exploitation / Akses     | Service SSH (`Port 2222/tcp`)                     | `Hydra`              |
| **SCN-03**  | Web / API / Protokol IoT | Endpoint Telemetri (`POST /api/telemetry/volume`) | `Burp Suite`         |

---

## 3. Rincian Skenario Serangan

### Skenario SCN-01: Network Reconnaissance & Service Enumeration

- **Kategori:** Reconnaissance
- **Tools Wajib:** `nmap`
- **Target Spesifik:** `192.168.13.34` (Port 22, 2222, 3000, 27017)
- **Korelasi Celah / Tech Stack:**
  Pengujian attack surface untuk mengevaluasi apakah konfigurasi host firewall (UFW), pemindahan port SSH ke 2222, dan pembatasan bind IP MongoDB ke `127.0.0.1` berhasil menyembunyikan layanan internal dari pemindaian eksternal.
- **Metodologi Pengujian:**
  - Melakukan _TCP SYN scan_ dan _service banner enumeration_ terhadap port standar dan non-standar target.
  - Membandingkan respons status port antara _closed_ dan _filtered_.
- **Hasil yang Diharapkan:**
  - Port bawaan SSH (22) dan port database (27017) berstatus `filtered`.
  - Hanya port layanan aktif publik (`2222/tcp` dan `3000/tcp`) yang terdeteksi terbuka.

---

### Skenario SCN-02: SSH Credential Brute-Force & Lockout Assessment

- **Kategori:** Exploitation / Akses
- **Tools Wajib:** `Hydra`
- **Target Spesifik:** Layanan OpenSSH pada `192.168.13.34:2222`
- **Korelasi Celah / Tech Stack:**
  Pengujian ketahanan mekanisme autentikasi SSH terhadap serangan tebakan kredensial berulang (_dictionary attack_), serta validasi keaktifan daemon `fail2ban` dalam memantau port kustom 2222.
- **Metodologi Pengujian:**
  - Menjalankan serangan kamus menggunakan wordlist terhadap user target pada port kustom.
  - Mengamati perilaku koneksi saat percobaan login yang gagal melampaui batas toleransi (`maxretry`).
- **Hasil yang Diharapkan:**
  - Upaya koneksi Hydra terputus secara sepihak (_Connection refused / timed out_).
  - Alamat IP penyerang (`192.168.13.2`) tercatat masuk daftar pemblokiran sementara (_jail_) oleh Fail2Ban di sisi Target Server.

---

### Skenario SCN-03: Unauthenticated Sensor Spoofing & Telemetry Injection

- **Kategori:** Web / API / Protokol IoT
- **Tools Wajib:** `Burp Suite`
- **Target Spesifik:** Endpoint HTTP `POST http://192.168.13.34:3000/api/telemetry/volume`
- **Korelasi Celah / Tech Stack:**
  Pengujian integritas data telemetri IoT (_Device Authentication Oversight_). Pada endpoint `/api/telemetry/volume`, pengembang hanya memasang filter validasi format tipe data (_input validation_), namun tidak menerapkan mekanisme verifikasi identitas perangkat (seperti _API Key_, tanda tangan HMAC, atau sertifikat mTLS). Akibatnya, siapa pun di jaringan dapat memalsukan data sensor.
- **Metodologi Pengujian:**
  - Menyiapkan payload data sensor buatan yang nilainya berada dalam rentang validasi yang sah (contoh: `{"jarak_cm": 5.0, "estimasi_persen": 95.0}`).
  - Mengirimkan payload tersebut secara langsung ke endpoint tanpa menyertakan kredensial atau token autentikasi menggunakan Burp Suite (Repeater).
  - Memverifikasi apakah data masukan asing tersebut diterima dan diproses oleh aplikasi.
- **Hasil yang Diharapkan:**
  - Server mengembalikan respons `HTTP/1.1 201 Created` karena payload lolos validasi numerik dan tidak dicegat oleh autentikasi.
  - Data palsu berhasil masuk ke koleksi `log_sistems` di database MongoDB, membuktikan adanya celah pemalsuan metrik sensor (_telemetry spoofing_) yang dapat mengacaukan pemantauan pakan.
