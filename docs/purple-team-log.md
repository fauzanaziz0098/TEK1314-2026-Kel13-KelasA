# Purple Team Log - Kelompok 13

Setiap skenario pada [`attack_plan.md`](./redteam/attack_plan.md) dicatat melalui lima tahap. Siklus dinyatakan tertutup bila serangan yang berhasil pada Tahap 1 gagal atau terblokir pada Tahap 5. Menjadi isi utama Vulnerability Assessment Report Minggu 11.

| Tahap                 | Red Team                                          | Blue Team                                                 | Bukti                                     |
| :-------------------- | :------------------------------------------------ | :-------------------------------------------------------- | :---------------------------------------- |
| 1. Serang (Baseline)  | Eksekusi skenario pada kondisi hardening saat ini | Amati deteksi di Security Onion dan log                   | Screenshot serangan + status deteksi awal |
| 2. Deteksi            | -                                                 | Alert atau log yang muncul, atau catatan tidak terdeteksi | Screenshot alert / hasil grep log         |
| 3. Analisis           | Teknik dan tujuan serangan                        | Root cause, kontrol yang kurang                           | Catatan analisis                          |
| 4. Respons (Mitigasi) | -                                                 | Kontrol tambahan: rule alert, patch, firewall             | Diff konfigurasi                          |
| 5. Verifikasi         | Ulangi serangan yang sama                         | Konfirmasi serangan gagal atau terblokir                  | Screenshot percobaan ulang                |

Baris tidak boleh dihapus. Kegagalan deteksi tetap dicatat sebagai temuan. Bukti disimpan di `docs/redteam/evidence/` atau `docs/blueteam/evidence/`. Status: `Belum`, `Berjalan`, `Selesai`.

## Ringkasan Status

| Skenario | Kategori                 |   1   |   2   |   3   |   4   |   5   |
| :------- | :----------------------- | :---: | :---: | :---: | :---: | :---: |
| SCN-01   | Reconnaissance           | Belum | Belum | Belum | Belum | Belum |
| SCN-02   | Exploitation / Akses     | Belum | Belum | Belum | Belum | Belum |
| SCN-03   | Web / API / Protokol IoT | Belum | Belum | Belum | Belum | Belum |
