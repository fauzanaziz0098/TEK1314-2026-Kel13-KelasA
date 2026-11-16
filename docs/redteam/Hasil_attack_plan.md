PENGUJIAN ATTACK PLAN RED TEAM

**Topik:** Skenario Pengujian Keamanan SCN-01 (Reconnaissance), SCN-02 (Brute-Force SSH), dan SCN-03 (Telemetry Spoofing)  
**Kelompok :** 13  
**Mata Kuliah :** Keamanan Siber

# 1. PENDAHULUAN & ARSITEKTUR LINGKUNGAN LAB

Praktikum ini mensimulasikan pengujian penetrasi keamanan (Red Team) terhadap infrastruktur server IoT replika untuk mengevaluasi kerentanan layanan pada mesin target virtual.

- **Attacker Node:** Kali Linux (192.168.13.2 / terminal attacker)
- **Target Node (Victim):** Ubuntu VM (192.168.13.34)
- **Router Gateway:** MikroTik CHR (192.168.13.1 / 192.168.13.33)
- **Layanan Target:**
  - OpenSSH Remote Management (Port 2222/tcp)
  - Node.js Express Backend API (Port 3000/tcp)
  - Database MongoDB (Localhost Binding 127.0.0.1:27017)
- **Target User:** kel13

# 2. SKENARIO 1 (SCN-01): NETWORK RECONNAISSANCE & SERVICE ENUMERATION

- **Kategori:** Reconnaissance
- **Tools Wajib:** nmap
- **Target Spesifik:** 192.168.13.34
- **Metodologi Pengujian:**
  - Melakukan TCP SYN scan dan service banner enumeration terhadap target untuk memetakan attack surface.
  - Memverifikasi port aktif serta status layanan internal versus eksternal.
- **Hasil Temuan:**
  - Port standar SSH (22) dan port database MongoDB (27017) terdeteksi filtered / tertutup dari luar.
  - Layanan aktif publik yang terbuka dan merespons adalah Port 2222/tcp (OpenSSH) dan Port 3000/tcp (Node.js Express API).
  - Status SCN-01 dinyatakan SUKSES.
  - evidence : [evidence/scn1.png](evidence/scn1.png)

# 3. SKENARIO 2 (SCN-02): SSH CREDENTIAL BRUTE-FORCE & LOCKOUT ASSESSMENT

- **Kategori:** Exploitation / Akses
- **Tools Wajib:** Hydra
- **Target Spesifik:** Layanan OpenSSH pada 192.168.13.34:2222
- **Metodologi Pengujian:**
  - Menyiapkan file wordlist kustom (pass.txt) di Kali Linux.
  - Menjalankan dictionary attack terhadap akun pengguna kel13 pada port kustom 2222.

<table><thead><tr><th><pre><code>echo "kelompok13" &gt; pass.txt
hydra -l kel13 -P pass.txt ssh://192.168.13.34:2222</code></pre></th></tr></thead></table>

- **Hasil Temuan:**
  - Hydra berhasil menemukan kredensial yang valid.
  - Valid Credential Found: login: kel13 | password: kelompok13.
  - Status SCN-[evidence/scn2.png](evidence/scn2.png)

# 4. SKENARIO 3 (SCN-03): UNAUTHENTICATED SENSOR SPOOFING & TELEMETRY INJECTION

- **Kategori:** Web / API / Protokol IoT
- **Tools Wajib:** curl / Burp Suite
- **Target Spesifik:** Endpoint HTTP POST <http://192.168.13.34:3000/api/telemetry/volume>
- **Metodologi Pengujian:**
  - Menguji validasi payload backend dan mengirimkan data sensor palsu tanpa token autentikasi.
  - Mengirim payload JSON yang disesuaikan

curl -X POST <http://192.168.13.34:3000/api/telemetry/volume> \\

\-H "Content-Type: application/json" \\

\-d '{"jarak_cm": 777777, "estimasi_persen": 0, "volume": 999, "status": "spoofed_by_attacker"}'

- **Hasil Temuan:**
  - Server merespons dengan status sukses (200 OK / success).
  - Data telemetri palsu berhasil masuk dan tercatat permanen di database MongoDB (db.log_sistems.find().pretty()).
  - Status SCN-03 dinyatakan SUKSES.
  - evidence : [evidence/scn3.png](evidence/scn3.png)

# 5. KESIMPULAN & REKOMENDASI MITIGASI

Berdasarkan hasil pengujian penetrasi pada SCN-01, SCN-02, dan SCN-03, ditemukan beberapa celah keamanan signifikan pada arsitektur lingkungan target IoT. Pengujian reconnaissance (SCN-01) menunjukkan bahwa layanan publik teridentifikasi terbuka pada Port 2222/tcp (OpenSSH) dan Port 3000/tcp (Node.js Express API), sementara port standar seperti SSH (22) dan MongoDB (27017) berhasil difilter dari akses eksternal. Namun, serangan SSH credential brute-force (SCN-02) berhasil menemukan kredensial valid pengguna kel13 akibat lemahnya kebijakan kata sandi dan ketiadaan mekanisme lockout (seperti Fail2ban). Selain itu, pengujian telemetry injection (SCN-03) membuktikan bahwa endpoint HTTP POST (/api/telemetry/volume) menerima data sensor palsu tanpa proses autentikasi maupun validasi input, sehingga memungkinkan manipulasi data yang tersimpan permanen pada database MongoDB. Untuk memitigasi risiko tersebut, direkomendasikan pengetatan kebijakan autentikasi melalui penerapan Fail2ban dan autentikasi berbasis kunci SSH, pengimplementasian token autentikasi (seperti JWT atau API Key) serta validasi skema payload pada endpoint API telemetri, serta pembatasan akses jaringan menggunakan firewall secara berkala.