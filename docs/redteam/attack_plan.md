# Red Team Attack Plan & Hasil Pengujian: Target Node Replika

Dokumen ini memuat rencana pengujian penetrasi beserta hasil eksekusi aktual oleh Red Team terhadap Target Node replika IoT Server (**KIRA Smart Chicken Feeder**). Rencana ini disusun untuk memverifikasi efektivitas kontrol pertahanan (Blue Team) serta mengidentifikasi celah keamanan nyata pada arsitektur sistem berbasis IoT.

---

## 1. Lingkungan & Target Pengujian

* **Attacker Node:** `192.168.13.2` (Kali Linux — Segmen Attacker)
* **Target Node (Replika):** `192.168.13.34` (Ubuntu Server — Segmen Target)
* **Router Gateway:** MikroTik CHR (`192.168.13.1` / `192.168.13.33`)
* **Target User:** `kel13`
* **Layanan pada Target:**
  * Remote Management: OpenSSH (`Custom Port 2222/tcp`)
  * Application Layer: Node.js Express REST API (`Port 3000/tcp`)
  * Database Layer: MongoDB (`Localhost Binding 127.0.0.1:27017`)

---

## 2. Matriks Rencana & Status Skenario Serangan

| ID Skenario | Kategori Serangan | Layanan / Endpoint Target | Tools yang Digunakan | Status | Evidence |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **SCN-01** | Reconnaissance | Seluruh port Target (`192.168.13.34`) | `nmap` | **SUKSES** | `evidence/scn1_Reconnaissance.png` |
| **SCN-02** | Exploitation / Akses | Service SSH (`Port 2222/tcp`) | `Hydra` | **GAGAL / DIBLOKIR** (Ditolak Fail2ban) | `evidence/scn2_BruteForce.png` |
| **SCN-03** | Web / API / Protokol IoT | Endpoint Telemetri (`POST /api/telemetry/volume`) | `Burp Suite` | **SUKSES** (Sebelum hardening) / **DITOLAK 401** (Sesudah hardening) | `evidence/scn3_BurpSuite_Before.png`, `evidence/scn3_BurpSuite_After.png` |

---

## 3. Rincian Skenario & Hasil Pengujian

### Skenario SCN-01: Network Reconnaissance & Service Enumeration

* **Kategori:** Reconnaissance
* **Tools Wajib:** `nmap`
* **Target Spesifik:** `192.168.13.34` (Port `22`, `2222`, `3000`, `27017`)
* **Korelasi Celah / Tech Stack:** Pengujian *attack surface* untuk mengevaluasi apakah konfigurasi *host firewall* (UFW), pemindahan port SSH ke `2222`, dan pembatasan *bind IP* MongoDB ke `127.0.0.1` berhasil menyembunyikan layanan internal dari pemindaian eksternal.
* **Metodologi Pengujian:**
  * Melakukan TCP SYN scan dan *service banner enumeration* terhadap port standar dan non-standar target:
    ```bash
    nmap -sS -sV -p 22,2222,3000,27017 192.168.13.34
    ```
  * Membandingkan respons status port antara `closed`, `filtered`, dan `open`.
* **Hasil yang Diharapkan:**
  * Port bawaan SSH (`22`) dan port database (`27017`) berstatus `filtered`.
  * Hanya port layanan aktif publik (`2222/tcp` dan `3000/tcp`) yang terdeteksi terbuka.
* **Hasil Temuan Aktual:**
  * Port standar SSH (`22/tcp`) dan port database MongoDB (`27017/tcp`) terdeteksi `filtered` / tertutup dari akses luar.
  * Layanan aktif publik yang terbuka dan merespons adalah **Port `2222/tcp` (OpenSSH)** dan **Port `3000/tcp` (Node.js Express API)**.
  * Status **SCN-01** dinyatakan **SUKSES** (`evidence/scn1_Reconnaissance.png`).

---

### Skenario SCN-02: SSH Credential Brute-Force & Lockout Assessment

* **Kategori:** Exploitation / Akses
* **Tools Wajib:** `Hydra`
* **Target Spesifik:** Layanan OpenSSH pada `192.168.13.34:2222`
* **Korelasi Celah / Tech Stack:** Pengujian ketahanan mekanisme autentikasi SSH terhadap serangan tebakan kredensial berulang (*dictionary attack*), serta validasi keaktifan daemon `Fail2ban` dalam memantau port kustom `2222`.
* **Metodologi Pengujian:**
  * Menyiapkan file wordlist kustom (`pass.txt` / `users.txt` dan `passwords.txt`) di Kali Linux dan menjalankan serangan kamus terhadap akun `kel13` pada port `2222`:
    ```bash
    echo "kelompok13" > pass.txt
    hydra -l kel13 -P pass.txt ssh://192.168.13.34:2222
    # atau menggunakan daftar user & password:
    hydra -L ~/users.txt -P ~/passwords.txt -s 2222 ssh://192.168.13.34 -t 4 -V
    ```
  * Mengamati perilaku koneksi saat percobaan login gagal melampaui batas toleransi (`maxretry`).
* **Hasil yang Diharapkan:**
  * Upaya koneksi Hydra terputus secara sepihak (*Connection refused / timed out*).
  * Alamat IP penyerang (`192.168.13.2`) tercatat masuk daftar pemblokiran sementara (*jail*) oleh Fail2ban di sisi Target Server.
* **Hasil Temuan Aktual:**
  * Pada percobaan *brute-force* dengan Hydra (`hydra -L users.txt -P passwords.txt -s 2222 ssh://192.168.13.34 -t 4 -V -I`), tidak ada kredensial valid yang berhasil ditemukan (`0 valid password found`) karena daemon Fail2ban (`sshd` jail) aktif memblokir IP penyerang `192.168.13.2`, sehingga koneksi berikutnya ditolak (`Connection refused`).
  * Status serangan **SCN-02** dinyatakan **GAGAL / DIBLOKIR** (kontrol Fail2ban bekerja sesuai ekspektasi, lihat `evidence/scn2_BruteForce.png`).

---

### Skenario SCN-03: Unauthenticated Sensor Spoofing & Telemetry Injection

* **Kategori:** Web / API / Protokol IoT
* **Tools Wajib:** `Burp Suite`
* **Target Spesifik:** Endpoint HTTP POST `http://192.168.13.34:3000/api/telemetry/volume`
* **Korelasi Celah / Tech Stack:** Pengujian integritas data telemetri IoT (*Device Authentication Oversight*). Pada endpoint `/api/telemetry/volume`, pengembang awalnya hanya memasang filter validasi format tipe data (*input validation*), namun tidak menerapkan mekanisme verifikasi identitas perangkat (seperti API Key `x-api-key`, tanda tangan HMAC, atau sertifikat mTLS). Akibatnya, siapa pun di jaringan dapat memalsukan data sensor.
* **Metodologi Pengujian:**
  * Menyiapkan payload data sensor buatan pada tab **Burp Suite Repeater** yang nilainya berada dalam rentang validasi yang sah tanpa menyertakan kredensial atau token autentikasi, lalu mengirimkannya ke `192.168.13.34:3000`:
    ```http
    POST /api/telemetry/volume HTTP/1.1
    Host: 192.168.13.34:3000
    Content-Type: application/json
    Content-Length: 42

    {
      "jarak_cm": 9.9,
      "estimasi_persen": 90.9
    }
    ```
  * Memverifikasi apakah data masukan asing tersebut diterima dan diproses oleh aplikasi di MongoDB (`iotdb`).
* **Hasil yang Diharapkan:**
  * Sebelum mitigasi: Server mengembalikan respons `HTTP/1.1 201 Created` dan data palsu masuk ke koleksi `log_sistems` di database MongoDB.
  * Setelah mitigasi (`server.js.hardened` dengan validasi `x-api-key`): Server menolak request tanpa kunci sah dengan respons `HTTP/1.1 401 Unauthorized`.
* **Hasil Temuan Aktual:**
  * **Baseline (Sebelum Hardening):** Server merespons dengan status **`HTTP/1.1 201 Created`** (`"status": "success"`), dan data telemetri palsu berhasil masuk serta tercatat permanen di database MongoDB (`db.log_sistems.find().pretty()`).
  * **Verifikasi Ulang (Sesudah Hardening):** Setelah Blue Team menerapkan `server.js.hardened` (validasi header `x-api-key`), percobaan pengiriman ulang payload yang sama lewat Burp Suite Repeater ditolak dengan status **`HTTP/1.1 401 Unauthorized`** dan tidak ada data palsu baru yang masuk ke database.
  * Status **SCN-03** dinyatakan **SUKSES** (`evidence/scn3_BurpSuite_Before.png`, `evidence/scn3_BurpSuite_After.png`).

---

## 4. Kesimpulan & Rekomendasi Mitigasi

Berdasarkan hasil pengujian penetrasi pada **SCN-01**, **SCN-02**, dan **SCN-03**, ditemukan beberapa celah keamanan signifikan pada arsitektur lingkungan target IoT:
* Pengujian *reconnaissance* (**SCN-01**) menunjukkan bahwa layanan publik teridentifikasi terbuka pada Port `2222/tcp` (OpenSSH) dan Port `3000/tcp` (Node.js Express API), sementara port standar SSH (`22`) dan MongoDB (`27017`) berhasil difilter dari akses eksternal.
* Pengujian *brute-force* SSH (**SCN-02**) gagal menembus sistem karena mekanisme pembatasan percobaan login (**Fail2ban**) aktif memblokir IP penyerang (`Connection refused`), namun migrasi ke autentikasi berbasis kunci publik (SSH Key) tetap disarankan untuk memperkuat pertahanan.
* Pengujian *telemetry injection* (**SCN-03**) via Burp Suite membuktikan bahwa endpoint HTTP POST (`/api/telemetry/volume`) awalnya menerima data sensor palsu (`201 Created`) tanpa autentikasi perangkat, sehingga memanipulasi koleksi `log_sistems` di MongoDB.

**Rekomendasi Mitigasi:**
1. Mempertahankan autentikasi perangkat berbasis header **`x-api-key`** pada backend Express.js (`server.js.hardened`) serta validasi skema payload agar permintaan tidak sah ditolak (`401 Unauthorized`).
2. Mempertahankan aturan **Fail2ban** pada layanan OpenSSH port `2222/tcp` serta beralih ke autentikasi berbasis **SSH Key**.
3. Mempertahankan pembatasan akses jaringan via **UFW** (`default deny incoming`), pengikatan database pada `127.0.0.1:27017`, dan pemantauan lalu lintas kontinu melalui TZSP *sniffer* ke **Security Onion**.