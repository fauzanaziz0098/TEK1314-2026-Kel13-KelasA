# Skenario Proyek - Kelompok 13

Proyek ini mensimulasikan skenario keamanan siber pada perangkat IoT dalam lingkungan lab yang terisolasi.

Topologi terdiri dari:
- Attacker Node (`192.168.13.2/27`) - Kali Linux
- Target Server (`192.168.13.34/27`) - Ubuntu Server
- Monitoring Node (`192.168.13.66/27`) - Security Onion
- Router sebagai penghubung antarsegmen jaringan.

Setiap node terhubung langsung ke Router dan berada pada subnet yang berbeda. Router digunakan untuk melakukan routing antarsegmen.

Path dokumentasi:
- [`docs/design/topology.png`](./docs/design/topology.png)
- [`docs/design/ip_plan.md`](./docs/design/ip_plan.md)