# IP Plan — Replika Backend IoT (ThingsBoard)

Dokumen ini menggantikan IP plan lama yang memakai `192.168.2.x` dan skenario DVWA/Metasploitable. Seluruh simulasi Fase 2–3 memakai **`172.20.4.0/24`**.

## Ringkasan Subnet

| Zona | Subnet | Keterangan |
|---|---|---|
| Simulasi utama | `172.20.4.0/24` | Semua node Attacker, Firewall, Target, Monitoring |
| Out-of-band | `100.64.0.0/10` (Tailscale) | Akses administrasi dari host, bukan bagian dari topologi penilaian |

## Tabel IP Plan

| Hostname | IP Address | OS / Platform | Layanan relevan | Keterangan |
|---|---|---|---|---|
| Firewall / Router | `172.20.4.1` | Debian | SSH `:2244`, routing | Node perantara; akses masuk ke target |
| Target Node | `172.20.4.2` | Debian | HTTP `:80`, SSH `:2245`, PostgreSQL `:5432`, ThingsBoard `:9090` | Replika backend IoT; objek hardening & serangan |
| Monitoring Node | `172.20.4.3` | Security Onion | Wireshark, Sguil, Kibana | Sensor deteksi |
| Attacker Node | `172.20.4.4` | Kali Linux | nmap, Hydra, Nikto, curl | Red Team |

## Port yang Diuji

| Port | Layanan | Digunakan untuk |
|---|---|---|
| `80/TCP` | Apache Tomcat | redirect / reverse proxy ke ThingsBoard |
| `9090/TCP` | ThingsBoard | dashboard, REST API, WebSocket telemetry |
| `2245/TCP` | OpenSSH | administrasi target (key-only; bukan port 22) |
| `5432/TCP` | PostgreSQL | database ThingsBoard |

## Catatan

- SSH target sudah dipindah dari `22` ke **`2245`**.
- Firewall mengizinkan SSH hanya dari jalur tertentu; koneksi langsung dari Tailscale bisa kena DROP.
- `docs/design/topology.png` dari Minggu 4 dipertahankan sesuai ketentuan modul, tetapi angka IP di dokumen ini yang dipakai sebagai acuan narasi.
