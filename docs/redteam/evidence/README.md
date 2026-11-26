# Red Team Evidence — Index

Output verbatim dari terminal Attacker (kali), target `172.20.4.2`.

| Tanggal | Skenario | File | Isi |
|---------|----------|------|-----|
| 2026-11-26 | S1 Recon | `2026-11-26_s1_fullscan.txt` | Full scan 65535 port (`-sS -sV -O`), 795 dtk |
| 2026-11-26 | S1 Recon | `2026-11-26_s1_http_check.txt` | `curl -i` :80 dan :9090 → ThingsBoard identik |
| 2026-11-26 | S1 Recon | `2026-11-26_s1_nse_banner.txt` | NSE banner + http-headers + http-methods |
| 2026-11-26 | S2 Exploit | `2026-11-26_s2_ssh_brute.txt` | NSE ssh-brute + Hydra ke :2245 → key-only, gagal |
| 2026-11-26 | S2 Exploit | `2026-11-26_s2_pgsql_brute.txt` | NSE pgsql-brute + psql :5432 → kredensial default diganti; versi TB tidak bocor |
| 2026-11-26 | S2 Exploit | `2026-11-26_s2_hydra_pg_full.txt` | Hydra 5 user x 5000 pass → semua child gagal connect — indikasi rate-limit/blokir aktif |
| 2026-11-26 | S3 Web/API | `2026-11-26_s3_nikto.txt` | Nikto :9090 dan :80 → missing security headers |
| 2026-11-26 | S3 Web/API | `2026-11-26_s3_default_creds.txt` | Kredensial default ThingsBoard → Invalid username or password |
| 2026-11-26 | S3 Web/API | `2026-11-26_s3_deep_enum.txt` | Deep enum: NO TLS, Swagger ter-expose, header inkonsisten |
| 2026-11-26 | S3 Web/API | `2026-11-26_s3_swagger_enum.txt` | Swagger UI publik: versi 4.2.1.1 terbongkar, /api/noauth/* terbuka |
| 2026-11-26 | S3 Web/API | `2026-11-26_s3_noauth_policy.txt` | userPasswordPolicy bocor: min 6, tanpa kompleksitas — lemah |
| 2026-11-26 | S3 Web/API | `2026-11-26_s3_noauth_ws_actuator.txt` | WS telemetry 101 tanpa auth; /actuator terekspos terbatas; token endpoints anti-enumerasi |
| 2026-11-26 | S3 Web/API | `2026-11-26_s3_ws_auth_test.txt` | Klien WS: subscribe tanpa AUTH → 1008 policy violation — vektor WS tertutup |

## Ringkasan temuan 26 Nov 2026

1. Port open: 80 (Tomcat), 2245 (SSH), 5432 (PostgreSQL), 9090 (Tomcat). 65531 filtered.
2. SSH :2245 key-only — brute-force password ditolak protokol.
3. :80 dan :9090 menyajikan ThingsBoard identik (HTTP 200, build 23 Dec 2025).
4. X-Frame-Options dan X-Content-Type-Options absen di kedua port HTTP.
