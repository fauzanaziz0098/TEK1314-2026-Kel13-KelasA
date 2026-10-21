# Hardening Checklist - Target Server (192.168.13.34)

| ID  | Kategori        | Kontrol Hardening                        | Target Layanan / File      | Status   | Link Bukti (Evidence) |
| --- | --------------- | ---------------------------------------- | -------------------------- | -------- | --------------------- |
| A1  | Access & Auth   | Nonaktifkan Login Root Langsung          | `/etc/ssh/sshd_config`     | Progress |                       |
| A2  | Access & Auth   | Ganti Port Default SSH (22 -> 2222)      | `/etc/ssh/sshd_config`     | Progress |                       |
| A3  | Access & Auth   | Implementasi Fail2ban (Anti Brute-Force) | `/etc/fail2ban/jail.local` | Progress |                       |
| B1  | Service Surface | Host Firewall (UFW) Whitelist Port       | UFW (Allow 2222 & 3000)    | Progress |                       |
| B2  | Service Surface | Binding Localhost MongoDB (127.0.0.1)    | `/etc/mongod.conf`         | Progress |                       |
| B3  | Service Surface | Patch Paket & Minimalisasi Service       | Sistem Operasi Ubuntu      | Progress |                       |
| C1  | Data & App      | Pemisahan Kredensial DB ke File `.env`   | `.env` Backend Express     | Progress |                       |
| C2  | Data & App      | Audit Logging Request Masuk              | Middleware Express.js      | Progress |                       |
| C3  | Data & App      | Validasi Input & Rate Limiter API        | Endpoint `/api/telemetry`  | Progress |                       |
