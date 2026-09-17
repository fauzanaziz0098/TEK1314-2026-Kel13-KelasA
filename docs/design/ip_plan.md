# IP Plan - Kelompok 13

**Mata Kuliah:** TEK1314 - Keamanan Siber

## Segmen VLAN

| VLAN | Nama Segmen | Subnet | Range IP Usable |
|------|--------------|---------|-------------------|
| VLAN 10 | Attacker | 192.168.13.0/27 | 192.168.13.1 sampai 192.168.13.30 |
| VLAN 20 | Target | 192.168.13.32/27 | 192.168.13.33 sampai 192.168.13.62 |
| VLAN 30 | Monitoring | 192.168.13.64/27 | 192.168.13.65 sampai 192.168.13.94 |

## Tabel Alokasi IP

| Hostname | IP Address | VLAN | OS Direncanakan | Peran |
|-----------|-------------|-------|-------------------|--------|
| Router (interface VLAN 10) | 192.168.13.1 | VLAN 10 | - | Gateway segmen Attacker |
| Attacker Node | 192.168.13.2 | VLAN 10 | Kali Linux | Node penyerang |
| Router (sub-interface VLAN 20) | 192.168.13.33 | VLAN 20 | - | Gateway segmen Target |
| Target Server (Korban) | 192.168.13.34 | VLAN 20 | Ubuntu Server (IoT) | Node target |
| Router (sub-interface VLAN 30) | 192.168.13.65 | VLAN 30 | - | Gateway segmen Monitoring |
| Monitoring Node | 192.168.13.66 | VLAN 30 | Security Onion | Menganalisis trafik VLAN 20 |

## Jalur Koneksi

| Perangkat | Terhubung Ke | Jenis Koneksi |
|------------|----------------|------------------|
| Attacker Node | Router | Access port, VLAN 10 |
| Router | Switch | Trunk, membawa VLAN 20 dan VLAN 30 |
| Switch | Target Server | Access port, VLAN 20 |
| Switch | Monitoring Node | Access port, VLAN 30, menerima mirror dari port VLAN 20 |

## Tabel Routing

| Sumber | Tujuan | Jalur |
|---------|---------|--------|
| VLAN 10 (Attacker) | VLAN 20 (Target) | Router, inter-VLAN routing |
| Switch (port VLAN 20) | VLAN 30 (Monitoring) | Mirror port pada switch |

## Riset Port (Red Team)

| Port | Layanan | Keterangan |
|------|----------|-------------|
| 22 | SSH | Akses remote |
| 80/443 | HTTP/HTTPS | Web management |
| 1883/8883 | MQTT | Komunikasi sensor/device IoT |

## Keputusan OS Target

- OS Target: Ubuntu Server
- Skenario: Target berperan sebagai perangkat/server IoT
- Dikoordinasikan oleh: Lead