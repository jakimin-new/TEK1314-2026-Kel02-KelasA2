# IP Plan

Dokumen ini berisi rencana pembagian IP Address untuk seluruh node pada topologi jaringan skenario **Web & Service Exploitation (DVWA + SSH Brute Force)**.

## Ringkasan Subnet

| Zona | Subnet | Range Usable |
|---|---|---|
| Attacker Zone | 192.168.2.0/28 | .1 – .14 |
| DMZ Zone | 192.168.2.16/28 | .17 – .30 |
| Internal Zone | 192.168.2.32/27 | .33 – .62 |
| Monitoring Zone | 192.168.2.64/28 | .65 – .78 |
| VPN Tunneling | 192.168.2.240/30 | .241 – .242 |

## Tabel IP Plan

| Hostname | IP Address | OS Direncanakan | Keterangan |
|---|---|---|---|
| Attacker Node | 192.168.2.2 | Kali Linux / CyberOps | Melakukan exploitation via port 80 (DVWA) & 22 (SSH brute force) |
| Client (VPN) | 192.168.2.241 | Windows / Linux Client | Terhubung ke Intermediary Zone via VPN Tunneling |
| Firewall / Router | 192.168.2.1 (per subnet) | Intermediary Zone Gateway | Menghubungkan seluruh zona jaringan |
| Target Server - Web/DB | 192.168.2.18 | Metasploitable 2 | Target utama, port 80/TCP (DVWA) |
| Target Server - Web/DB (2) | 192.168.2.19 | Metasploitable 2 | Target cadangan pada DMZ Zone |
| Internal Server | 192.168.2.34 | Metasploitable 2 | Internal Zone, port 22/TCP (SSH weak password) |
| Internal Workstation | 192.168.2.35 | Ubuntu / Windows | Internal Zone, simulasi workstation pengguna |
| Monitoring Node | 192.168.2.66 | Security Onion | Memantau trafik pada Intermediary Zone |

## Catatan

- Target OS: **Metasploitable 2** (hemat resource, service rentan sudah tersedia secara bawaan).
- Port yang diuji:
  - **80/TCP** – HTTP (DVWA / Web Vulnerability)
  - **22/TCP** – SSH (Weak Password Testing / Brute Force)
