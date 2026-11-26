# TEK1314-2026-Kel13-KelasA — Pengujian & Pengamanan Server IoT KIRA (Smart Chicken Feeder)

Repositori  **Kelompok 13 (Kelas A)** untuk Proyek Akhir (PBL) Mata Kuliah **TEK1314 — Keamanan Siber**, Program Studi Teknologi Rekayasa Komputer, Sekolah Vokasi, IPB University.

---

## 1. Ringkasan Akhir Proyek

**KIRA (Smart Chicken Feeder)** adalah sistem otomasi pemberian pakan ayam berbasis IoT yang menggunakan mikrokontroler **ESP32**, sensor jarak laser **VL53L0X**, **motor servo**, dan **motor auger DC** untuk memantau serta menyalurkan pakan secara *real-time*. Data telemetri dikirimkan melalui **HTTP REST API** ke server backend **Node.js Express** dan disimpan pada basis data **MongoDB**.

Dalam proyek ini, tim membangun replika infrastruktur jaringan tersegmen pada lingkungan virtual untuk menguji dan memperkuat ketahanan backend KIRA melalui siklus **Purple Team Loop (Tahap 1–5)**:
1. **Fase 1 — Arsitektur & Baseline Hardening (Blue Team):** Membagi jaringan ke dalam tiga subnet `/27` menggunakan **MikroTik CHR**, mengaktifkan pencerminan trafik **TZSP** menuju **Security Onion (`192.168.13.66`)**, serta menerapkan 9 kontrol *hardening* (UFW *default deny*, port SSH kustom `2222/tcp`, Fail2ban, isolasi MongoDB di `127.0.0.1:27017`, variabel lingkungan `.env`, dan *logging* Morgan).
2. **Fase 2 — Pengujian Penetrasi (Red Team):** Mengeksekusi tiga skenario serangan dari Kali Linux (`192.168.13.2`) terhadap Target Server Ubuntu (`192.168.13.34`), yaitu pemindaian layanan (**SCN-01** dengan `nmap`), serangan kamus SSH (**SCN-02** dengan `Hydra`), dan injeksi data sensor palsu tanpa autentikasi (**SCN-03** dengan `Burp Suite Repeater`).
3. **Fase 3 — Remediasi & Verifikasi Ulang (Purple Team Loop):** Menutup kerentanan *unauthenticated telemetry spoofing* pada endpoint `POST /api/telemetry/volume` dengan menerapkan **`server.js.hardened`** (wajib validasi header `x-api-key`) dan me-*restart* layanan melalui PM2. Pengujian ulang menggunakan **Burp Suite Repeater** mengonfirmasi bahwa permintaan tanpa API Key ditolak dengan **`HTTP/1.1 401 Unauthorized`**, Fail2ban memblokir percobaan *brute-force* SSH, dan seluruh lalu lintas terekam di Security Onion.

---

## 2. Anggota Kelompok 13 (Kelas A)

| Nama Lengkap | NIM | Peran & Tanggung Jawab |
| :--- | :--- | :--- |
| **Naila Darussalamah** | J0404241059 | **Team Lead** — Koordinasi Proyek, Analisis Insiden & Dokumentasi Final |
| **Fauzan Aziz** | J0404241129 | **Blue Team** — Konfigurasi MikroTik CHR, Hardening Backend & Security Onion |
| **Ismail Daffa Antariksa** | J0404241176 | **Blue Team** — Manajemen Server Target, MongoDB & Pemantauan Trafik TZSP |
| **Luthfi Kurnia Siahaan** | J0404241150 | **Red Team** — Eksekusi Pengujian Penetrasi (`nmap`, `Hydra`, `Burp Suite`) |

---

## 3. Artefak Final (Minggu 14–15)

| Artefak | Lokasi File | Keterangan |
| :--- | :--- | :--- |
| **Incident Report Final (PDF)** | [docs/final/incident_report.pdf](docs/final/incident_report.pdf) | Laporan insiden lengkap berbasis standar NIST SP 800-61r2 |
| **Video Demo Purple Team Loop** | [docs/final/video_demo.mp4](docs/final/video_demo.mp4) | Video demonstrasi eksekusi serangan, mitigasi, dan verifikasi ulang |
| **Slide Presentasi Final (PDF)** | [docs/presentation-final/slides.pdf](docs/presentation-final/slides.pdf) | Materi presentasi akhir evaluasi proyek & siklus Purple Team |

---

## 4. Tautan Seluruh Dokumen Fase 1–3

### Fase 1: Desain Arsitektur, Tech Stack & Baseline Hardening
* **Deklarasi Tech Stack:** [docs/presentation-minggu9/tech_stack_declaration.md](docs/presentation-minggu9/tech_stack_declaration.md)
* **Rencana Alokasi IP & Subnetting:** [docs/design/ip_plan.md](docs/design/ip_plan.md)
* **Diagram Topologi Jaringan:** [docs/design/topology.png](docs/design/topology.png)
* **Checklist Hardening Blue Team:** [docs/blueteam/hardening_checklist.md](docs/blueteam/hardening_checklist.md)
* **Bukti Implementasi Blue Team:** [docs/blueteam/evidence/](docs/blueteam/evidence/)

### Fase 2: Pengujian Red Team
* **Attack Plan & Hasil Pengujian Red Team:** [docs/redteam/attack_plan.md](docs/redteam/attack_plan.md)
* **Bukti Tangkapan Layar Red Team:** [docs/redteam/evidence/](docs/redteam/evidence/)

### Fase 3: Kolaborasi Purple Team Loop
* **Log Siklus Purple Team (Tahap 1–5):** [docs/purple-team-log.md](docs/purple-team-log.md)