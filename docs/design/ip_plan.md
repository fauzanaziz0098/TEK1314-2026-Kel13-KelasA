# IP Plan - Kelompok 13

**Mata Kuliah:** TEK1314 - Keamanan Siber

## IP Address

| Hostname | IP Address | Gateway | OS |
|---|---|---|---|
| Router - Attacker | 192.168.13.1/27 | - | - |
| Attacker Node | 192.168.13.2/27 | 192.168.13.1 | Kali Linux |
| Target Server | 192.168.13.34/27 | 192.168.13.33 | Ubuntu Server |
| Monitoring Node | 192.168.13.66/27 | 192.168.13.65 | Security Onion |

## Segmen Jaringan

| Segmen | Subnet |
|---|---|
| Attacker | 192.168.13.0/27 |
| Target | 192.168.13.32/27 |
| Monitoring | 192.168.13.64/27 |

## Koneksi

- Attacker Node > Router
- Target Server > Router
- Monitoring Node > Router
- Router melakukan routing antarsegmen.