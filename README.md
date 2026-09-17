## Skenario Proyek - Kelompok 13

Proyek ini mensimulasikan skenario keamanan siber pada perangkat bertipe IoT dalam lingkungan lab yang terisolasi. Topologi terdiri dari Attacker Node yang terhubung ke Router, Router terhubung ke Switch lewat trunk, dan Switch menghubungkan Target Server serta Monitoring Node.

- VLAN 10, Attacker Node (`192.168.13.0/27`), Kali Linux, digunakan untuk simulasi pengujian celah keamanan terhadap Target Server.
- VLAN 20, Target Server (`192.168.13.32/27`), Ubuntu Server, merepresentasikan perangkat IoT dan menjadi objek analisis Red Team.
- VLAN 30, Monitoring Node (`192.168.13.64/27`), Security Onion, menerima mirrored traffic dari port VLAN 20 pada switch.

Router melakukan inter-VLAN routing sehingga trafik dari Attacker menuju Target selalu melewati router sebelum masuk ke switch.

Dokumentasi desain lengkap tersedia di [`docs/design/topology.png`](./docs/design/topology.png) dan [`docs/design/ip_plan.md`](./docs/design/ip_plan.md).