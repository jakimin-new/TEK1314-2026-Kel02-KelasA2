# Purple Team Log — Fase 2

Target: `172.20.4.2` | Attacker: `172.20.4.4` | SIEM: `172.20.4.3`

## S1 — Recon (nmap + banner)
- Baseline 22 Okt 2026: 22/open SSH; 9090/open Tomcat; 5432/open PostgreSQL 9.6+
- 25 Nov 2026 (koreksi Blue Team): Thingsboard seharusnya publik/internet sesuai cerita. Blokir total tadi = kesalahan config, sudah dibuka. Restart serangan dari awal dengan target reachable.
- 26 Nov 2026 (REPLAY resmi, evidence `2026-11-26_*`): full scan `-p- -sV -O` 795 dtk → **80/open Tomcat, 2245/open SSH 10.0p2, 5432/open PostgreSQL, 9090/open Tomcat**, 65531 filtered. `curl -i` :80 & :9090 → ThingsBoard identik (HTTP 200, build 23 Dec 2025). NSE banner+headers+methods terekam lengkap. Port 22 lama filtered (SSH pindah 22→2245).

## S2 — Exploitation (SSH 2245)
- Tahap 1 Serang (26 Nov 2026, evidence `2026-11-26_s2_ssh_brute.txt`): NSE `ssh-brute` → tanpa kredensial valid. `hydra -s 2245 -t 4` → `[ERROR] does not support password authentication`. Wordlist mini 5 user/6 pass, rate rendah.
- Tahap 1 Serang tambahan (26 Nov 2026 12:13, evidence `2026-11-26_s2_pgsql_brute.txt`): NSE `pgsql-brute` :5432 → tanpa kredensial valid. `psql -U postgres` manual 2x → `FATAL: password authentication failed`. Kredensial default PostgreSQL sudah diganti — kategori A terverifikasi di layer database. Versi ThingsBoard tidak bocor via halaman utama (`grep -i version` kosong) — cek CVE-2025-34281 butuh konfirmasi versi dari Blue Team.
- Tahap 1 brute PG massal (26 Nov 2026 13:23, evidence `2026-11-26_s2_hydra_pg_full.txt`): Hydra 5 user × 5000 pass (rockyou-head, `-t 4`) → semua child gagal connect, 0 percobaan valid. Interpretasi: proteksi aktif (rate-limit/fail2ban/blokir) memutus serangan otomatis SEBELUM auth diuji. Kredensial PG tidak tembus; temuan positif untuk kontrol kategori A/B.
- Verifikasi pasca-brute (26 Nov 2026 13:27): `nmap -p 5432 --reason` → **open (syn-ack)**; `psql` manual → tetap `FATAL: password authentication failed`. Port hidup normal untuk koneksi manual, tapi serangan otomatis Hydra diputus → konsisten dengan rate-limit/fail2ban aktif khusus pola brute-force. PostgreSQL RESMI benteng: kredensial diganti + proteksi otomatis + auth manual tetap dijaga.
- Run penutup (26 Nov 2026 13:47-13:52, evidence `2026-11-26_s2_hydra_pg_full.txt`): `hydra -l postgres -P top100.txt -t 1 -W 1` → 100 password terpopuler, 0 valid. Password postgres bukan password populer dunia; throttle tetap aktif (~20 tries/min). **S2 DITUTUP: PostgreSQL benteng terkonfirmasi penuh.**
- Tahap 2 Deteksi: [tunggu klip/alert dari Blue Team + SIEM]
- Tahap 3 Analisis: SSH pindah port + key-only → hardening kategori A bekerja; brute password mustahil di level protokol.
- Tahap 4 Respons: [keputusan Blue Team — tidak ada celah password yang perlu ditutup; pertahankan key-only]
- Tahap 5 Verifikasi: [ulangi brute pasca-mitigasi → tetap gagal]

## S3 — Web/API Thingsboard :80 + :9090
- Tahap 1 Serang (26 Nov 2026, evidence `2026-11-26_s3_nikto.txt`): `nikto -h :9090` & `:80` → X-Frame-Options absen + X-Content-Type-Options absen di KEDUA port (clickjacking & MIME-sniffing). Scan dihentikan manual setelah 2 temuan utama terkumpul. `curl -i` konfirmasi header respons tanpa kedua security header tsb.
- Tahap 2 Deteksi: [tunggu klip/alert dari Blue Team + SIEM]
- Tahap 3 Analisis: dua port sajikan app identik (reverse proxy :80→:9090 atau duplikat — konfirmasi Blue Team); missing security headers = temuan valid kategori C; uji kredensial default ThingsBoard + cek versi vs CVE-2025-34281 (< 4.2.1) berikutnya.
- Tahap 1 tambahan (26 Nov 2026, evidence `2026-11-26_s3_default_creds.txt`): kredensial default ThingsBoard (sysadmin/tenant/customer bawaan) diuji → respons `Invalid username or password`. Kredensial default sudah diganti — kategori A (hapus kredensial default) terverifikasi.
- Tahap 1 deep enum (26 Nov 2026 12:24-12:28, evidence `2026-11-26_s3_deep_enum.txt`): **NO TLS** — https handshake gagal, login dashboard plaintext HTTP (temuan kategori C terkuat). Swagger UI redirect `/swagger-ui.html → /swagger-ui/index.html` ter-expose (perlu konfirmasi isi). Security headers inkonsisten: ADA di `/api/*` (nosniff + X-Frame-Options DENY), TIDAK ADA di static frontend → scope mitigasi = layer static. Tomcat manager dimask SPA catch-all, HTTP method hanya GET/HEAD/OPTIONS, UDP top-50 tertutup. DoS attempt 1 batal (flag hping3 salah `--flood-i`), service tetap hidup.
- Tahap 1 Swagger enum (26 Nov 2026 12:32-12:35, evidence `2026-11-26_s3_swagger_enum.txt`): Swagger UI PUBlik di :9090 — **versi ThingsBoard 4.2.1.1 terbongkar** (>= 4.2.1 → CVE-2025-34281 gugur), peta API lengkap terekspos (admin/ai-model/alarm/asset/audit-log/auth/...). Endpoint publik `/api/noauth/*` (activate, resetPassword, userPasswordPolicy) + `/api/asset/types` deprecated. Semua endpoint sensitif terkunci JWT.
- Tahap 1 noauth policy (26 Nov 2026 12:43, evidence `2026-11-26_s3_noauth_policy.txt`): `/api/noauth/userPasswordPolicy` → 200 publik: **kebijakan password lemah** (min 6 karakter, tanpa wajib digit/uppercase/special, allowWhitespaces, tanpa expiration/reuse, forceReset=false). `/api/asset/types` → 401 (tetap terproteksi).
- Tahap 1 noauth + WS + actuator (26 Nov 2026 12:46-12:47, evidence `2026-11-26_s3_noauth_ws_actuator.txt`): `resetPasswordByEmail` → 200 kosong seragam (anti-enumeration email OK); activate/resetToken → 409 seragam (anti-enumeration token OK). WS `/api/ws/plugins/telemetry` handshake 101 tanpa auth (uji lanjutan lihat bawah). `/actuator` terekspos terbatas (hanya self+info; `/actuator/health` dimask SPA).
- Tahap 1 WS auth test (26 Nov 2026, evidence `2026-11-26_s3_ws_auth_test.txt`): klien WS sungguhan kirim subscribe telemetry tanpa AUTH → server tutup koneksi `1008 policy violation: Auth cmd is missing`. **Vektor WS TERTUTUP** — handshake terbuka tapi data terproteksi auth; temuan sebelumnya downgrade jadi aman.
- Tahap 4 Respons (26 Nov 2026, bukti `docs/mitigasi/`): Blue Team memasang mitigasi untuk keempat temuan:
  - **TLS**: Cloudflare Tunnel → hostname `thingsboard.cervosys.app` (foto `not-secure-web-access/01`, `03`). Akses via domain kini HTTPS; IP `100.87.196.33/login` masih HTTP not-secure (foto `02`) — sisa celah di jalur tailnet.
  - **Security headers**: nginx menambah `X-Frame-Options "DENY"` + `X-Content-Type-Options "nosniff"` always + rate-limit `10r/s` (foto `not-secure-web-access/04` + `open-header/02` — `curl -s -D -` menampilkan kedua header terpasang, HTTP/2 200 via Cloudflare).
  - **Swagger UI**: dinonaktifkan — `/swagger-ui/index.html` kini 404 `No static resource` (foto `swagger/02`; sebelum: foto `swagger/01`).
  - **Password policy**: minimum length 12, wajib 1 uppercase + 1 lowercase + 1 digit + 1 special (foto `password-policy-lemah/01`, halaman Security Settings sysadmin).
- Tahap 5 Verifikasi: [ulangi nikto/curl pasca-mitigasi → header muncul; Swagger 404; policy JSON berubah]
