# Proyek Simulasi Jaringan — Cyber Range (Fase 1–3)

## Anggota Kelompok 2 Kelas A

1. Dzaky Naufal Putra Raditio (J0404241020) — Blue Team
2. Siti Syafa Laela (J0404241053) — Lead
3. Gde Saputra Pramana Artha (J0404241107) — Blue Team
4. Muhamad Arif Dermawan (J0404241134) — Red Team

## Deskripsi Skenario

Simulasi **SOC Engagement — mengamankan infrastruktur backend sistem IoT**. Target Node dibangun sebagai replika backend IoT berbasis **ThingsBoard 4.2.1.1** di atas Debian, PostgreSQL, dan HTTP/MQTT. Seluruh pengujian dilakukan hanya pada VM lab; sistem IoT asli mitra tidak disentuh.

Topologi yang dipakai di seluruh artefak:

| Node | IP | Peran |
|---|---|---|
| Firewall | `172.20.4.1` | Intermediary / akses masuk |
| Target Node | `172.20.4.2` | ThingsBoard, SSH `:2245`, PostgreSQL `:5432`, HTTP `:80`/`:9090` |
| Monitoring Node | `172.20.4.3` | Security Onion |
| Attacker Node | `172.20.4.4` | Kali Linux (Red Team) |

> Catatan: IP `192.168.2.x` dan skenario DVWA/Metasploitable pada README lama sudah tidak dipakai. `docs/design/ip_plan.md` sudah diperbarui. IP Tailscale `100.x` adalah jalur out-of-band dan tidak termasuk IP plan.

## Struktur Repositori

```
docs/
├── design/
│   ├── topology.png                  # desain awal Minggu 4 (dipertahankan)
│   ├── ip_plan.md                    # sudah diselaraskan dengan 172.20.4.x
│   └── README.md
├── presentation-minggu9/
│   ├── slides.pdf
│   └── tech_stack_declaration.md
├── presentation-final/
│   └── README.md                     # slides.pdf masih disusun
├── blueteam/
│   ├── hardening_checklist.md        # 10 kontrol A/B/C + bukti di repo
│   └── evidence/                     # screenshot before/after & deteksi
├── redteam/
│   ├── attack_plan.md
│   ├── temuan-untuk-mitigasi.md
│   └── evidence/                     # output serangan + cuplikan video
├── purple-team-log.md                # log 5 tahap per skenario
├── mitigasi/                         # bukti mitigasi per temuan
└── final/
    ├── incident_report.pdf           # DRAF — lihat STATUS.md
    ├── video_demo.mp4                # rekaman attack.mp4 (230 MB)
    └── STATUS.md
README.md
```

## Status Artefak Final

| Artefak | Status |
|---|---|
| `docs/final/incident_report.pdf` | **DRAF** — masih ada bagian `[TEMPAT]`; lihat `docs/final/STATUS.md` |
| `docs/final/video_demo.mp4` | Tersedia (rekaman asli, 5 menit 19 detik) |
| `docs/presentation-final/slides.pdf` | Belum tersedia; sedang disusun |
| `docs/purple-team-log.md` | Sudah dipindah ke path yang benar; sebagian verifikasi masih `[TEMPAT]` |

## Ringkasan Topologi

- **Attacker Zone** — `172.20.4.4`, menjalankan nmap, Hydra, Nikto, curl.
- **Intermediary Zone** — firewall `172.20.4.1`.
- **Target Zone** — `172.20.4.2`, replika ThingsBoard + PostgreSQL.
- **Monitoring Zone** — `172.20.4.3`, Security Onion (Wireshark, Sguil, Kibana).
- Semua trafik simulasi berada di `172.20.4.0/24`.
